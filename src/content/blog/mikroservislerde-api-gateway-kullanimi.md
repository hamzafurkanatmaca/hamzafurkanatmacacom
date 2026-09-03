---
title: "Mikroservislerde API Gateway Kullanımı"
description: "Gateway ne işe yarar, ne işe yaramaz: yönlendirme, kimlik doğrulama, rate limit ve üretimde canımı yakan tuzaklar."
pubDate: "Aug 31 2026"
heroImage: "/images/api-gateway-mimarisi.svg"
---

Bir e-ticaret projesinde mobil ekibin lideri yanıma gelip "ana sayfa açılırken 11 ayrı isteğe çıkıyoruz, kötü bağlantıda uygulama açılmıyor" dedi. Baktım gerçekten öyle: kampanyalar bir servisten, kategoriler başka bir servisten, sepet sayısı bambaşka bir yerden geliyor. Üstelik her servisin adresi mobil uygulamanın içine gömülüydü. Bir servisin portu değişse market'e yeni sürüm çıkmamız gerekiyordu.

O gün "bize bir gateway lazım" dedik. Sonrasında öğrendiğim şey şu oldu: gateway koymak sorunu çözüyor ama yerine yeni bir sorun bırakıyor, ve o yeni sorunu baştan bilerek girmek lazım.

## Gateway Olmayınca Ne Oluyor?

Servis sayısı 3-4 iken kimse gateway'i düşünmüyor, haklı olarak. Sorun 8-10'u geçince başlıyor.

**İstemci bütün adresleri biliyor.** Mobil uygulama, web, bir de entegrasyon yapan iş ortağı — üçü de servislerin nerede durduğunu biliyor. Bir servisi bölmek istediğinde üç tarafı birden güncellemen gerekiyor. Pratikte bu "o servisi hiç bölmeyelim" demeye geliyor.

**Aynı kod her serviste tekrar ediyor.** Bizde 7 servis vardı ve 7 ayrı JWT doğrulama kodu. Sonradan fark ettik ki bir tanesi `exp` alanını hiç kontrol etmiyormuş — süresi geçmiş token'la istek atabiliyordun. Kod incelemesinde kimse görmemiş, çünkü herkes "bu zaten diğerinden kopyalandı" diye bakmış. Bu tarz kesişen ilgileri tek yerde toplamanın en büyük faydası burada.

**Her servis dışarı açık.** Yani her servis için ayrı sertifika, ayrı WAF kuralı, ayrı güvenlik duvarı istisnası. Güvenlik ekibi bunu sevmiyor, ki haklılar.

**İstemci gereksiz konuşkan.** Yukarıdaki 11 istek meselesi. Sunucular arasında bu istekler 2 milisaniye sürüyor, mobil şebekede her biri 200 milisaniye.

## Terminoloji

**Yönlendirme (Routing)**: Gelen isteğin yolu veya alan adına bakıp hangi servise gideceğine karar vermek. `/api/siparisler/*` → sipariş servisi gibi. Gateway'in en temel işi bu, gerisi üzerine eklenen şeyler.

**Kesişen ilgiler (Cross-cutting concerns)**: Her serviste tekrar eden işler — kimlik doğrulama, loglama, rate limit, CORS, sıkıştırma. Gateway'in asıl varlık sebebi bunları tek yere toplamak.

**BFF (Backend for Frontend)**: Her istemci tipi için ayrı bir cephe yazmak. Mobil az ve küçük veri ister, web ekranı daha zengin veri ister. Tek bir genel API ikisini de tam memnun edemiyor. BFF'i "gateway'in içine biraz uygulama mantığı koymanın meşru hali" diye düşünebilirsin.

**Toplama (Aggregation)**: Tek bir istemci isteğini birkaç servise dağıtıp cevapları birleştirmek. Kulağa hoş geliyor ama dikkat: burada gateway kolayca bir monolite dönüşebiliyor. Aşağıda tekrar değineceğim.

**Rate limit**: Belirli bir anahtar (kullanıcı, IP, API anahtarı) için birim zamandaki istek sayısını sınırlamak. Token bucket en yaygın yöntem — kova dolu başlıyor, her istek bir jeton yiyor, kova sabit hızda doluyor. Ani yükleri tolere ettiği için sabit pencereye göre daha adil.

**Devre kesici (Circuit breaker)**: Bir servis üst üste hata verdiğinde ona bir süre hiç istek göndermemek. "Zaten çökmüş servise istek yağdırıp iyice bitirmeyelim" mantığı. Çöken servis toparlanma fırsatı buluyor, sen de her istekte timeout süresi kadar beklemekten kurtuluyorsun.

**Servis keşfi (Service discovery)**: Gateway'in servislerin güncel adreslerini nereden öğrendiği. Kubernetes'te bu iş DNS ile hallediliyor, dışarıda Consul falan kullanılıyor. Adresleri konfigürasyona elle yazmak da bir yöntem, sadece ölçek büyüyünce sıkıntı çıkarıyor.

**Service mesh farkı**: Gateway kuzey-güney trafiğiyle ilgilenir, yani dışarıdan içeri gelen istekler. Service mesh ise doğu-batı trafiğiyle, yani servislerin kendi aralarındaki konuşmalarla. İkisi rakip değil, farklı işler. "Mesh kurduk, gateway'e gerek yok" cümlesini duyarsan biri bir şeyi karıştırıyor demektir.

## Mimari

![Mikroservis mimarisinde API Gateway'in konumu](/images/api-gateway-mimarisi.svg)

Akış şöyle: İstemci tek bir adrese istek atıyor. Gateway token'ı doğruluyor, rate limit'e bakıyor, isteğin hangi servise gideceğine karar veriyor ve iç ağa yönlendiriyor. Servisler dışarıya hiç açılmıyor.

Dikkat edilecek nokta, gateway'in token'ı doğruladıktan sonra arka tarafa ne gönderdiği. Biz doğrulanmış kullanıcı kimliğini bir başlığa koyup öyle iletiyoruz, servisler token'ı tekrar açmıyor. Ama bunun bir şartı var: iç ağa dışarıdan doğrudan erişilememesi lazım. Aksi halde herkes kendine `X-Kullanici-Id: 1` başlığı ekleyip admin oluyor.

## Gateway Seçenekleri

**YARP**: Şahsi favorim. Microsoft'un yazdığı, .NET içine gömülü çalışan bir ters vekil kütüphanesi. Ayrı bir ürün kurup yönetmiyorsun, kendi ASP.NET projene ekliyorsun. Yönlendirmeyi `appsettings.json`'dan okuyor, özel bir şey lazım olunca da C# yazıp araya girebiliyorsun. .NET ekibi için giriş maliyeti neredeyse sıfır.

**Eksiler:** Kutudan çıkan hazır özellik sayısı Kong gibi olgun ürünlere göre az. Yönetim paneli yok, portal yok, hazır eklenti pazarı yok. İstediğin şeyin bir kısmını kendin yazıyorsun.

**Ocelot**: .NET dünyasında uzun süre standart buydu, hâlâ çalışan bir sürü kurulum var. Yeni bir projeye bugün koyar mıyım, açıkçası koymam — ekosistem YARP'a kaydı.

**Eksiler:** Gelişimi yavaşladı, YARP'a göre daha az bakım alıyor. Yüksek yükte performansı da aynı seviyede değil.

**Nginx / Kong**: Nginx zaten ters vekil olarak çok olgun, Kong da onun üstüne eklenti sistemi kurmuş. Rate limit, kimlik doğrulama, dönüşüm — çoğu şey hazır eklenti olarak var. Çok dilli bir ortamdaysan (bir kısmı .NET, bir kısmı Go, bir kısmı Node) dil bağımsız olması ciddi avantaj.

**Eksiler:** Konfigürasyon dili ayrı bir öğrenme eğrisi. Standart eklentilerin dışına çıkman gerektiğinde Lua yazmak zorunda kalabiliyorsun, ki bu ekipte genelde bir kişinin bildiği bir şey oluyor ve o kişi izne çıkınca sıkıntı.

**Azure API Management / AWS API Gateway**: Yönetilen servis. Sen kurmuyorsun, yamalamıyorsun, ölçeklemiyorsun. Geliştirici portalı, abonelik anahtarları, kullanım raporları hazır geliyor. Dışarıya API satıyorsan veya iş ortaklarına açıyorsan bu paket baya iş görüyor.

**Eksiler:** Pahalı, özellikle APIM'in geliştirici dışı katmanları. Yerelde geliştirme yapmak zor, politikaları test etmek için genelde gerçek ortama çıkman gerekiyor. Bir de bulut sağlayıcısına iyice yapışıyorsun.

**Envoy**: Yüksek performanslı, service mesh'lerin de altında çalışan yapı. Ölçek gerçekten büyükse ve trafiği ince ayar yapman gerekiyorsa doğru araç.

**Eksiler:** Konfigürasyonu yorucu. Ekipte bu işe ayrılmış birileri yoksa altından kalkmak zor.

Bir de klasik tuzak: **kendi gateway'ini yazmak**. İlk hafta çok eğlenceli oluyor, `HttpClient` ile isteği alıp öbür tarafa gönderiyorsun, çalışıyor. Sonra streaming lazım oluyor, WebSocket lazım oluyor, hop-by-hop başlıklarını doğru yönetmen gerekiyor, `X-Forwarded-*` derdi çıkıyor... Altı ay sonra kimsenin dokunmak istemediği bir kütüphanenin bakımcısı oluyorsun. Bunun için hazır çözümler var, ben tavsiye etmem.

## Kod Örnekleri

### YARP ile Temel Yönlendirme

Yönlendirme tamamen konfigürasyondan okunuyor, kod yazmadan başlıyorsun:

```json
{
  "ReverseProxy": {
    "Routes": {
      "siparis-route": {
        "ClusterId": "siparis",
        "AuthorizationPolicy": "dogrulanmis",
        "Match": { "Path": "/api/siparisler/{**catch-all}" },
        "Transforms": [
          { "PathPattern": "/siparisler/{**catch-all}" }
        ]
      },
      "urun-route": {
        "ClusterId": "urun",
        "Match": { "Path": "/api/urunler/{**catch-all}" },
        "Transforms": [
          { "PathPattern": "/urunler/{**catch-all}" }
        ]
      }
    },
    "Clusters": {
      "siparis": {
        "LoadBalancingPolicy": "PowerOfTwoChoices",
        "Destinations": {
          "d1": { "Address": "http://siparis-servis:8080/" }
        },
        "HealthCheck": {
          "Active": { "Enabled": true, "Interval": "00:00:10", "Path": "/health" }
        }
      },
      "urun": {
        "Destinations": {
          "d1": { "Address": "http://urun-servis:8080/" }
        }
      }
    }
  }
}
```

```csharp
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

app.UseAuthentication();
app.UseAuthorization();
app.MapReverseProxy();
```

`urun-route`'ta bilerek `AuthorizationPolicy` yazmadım — ürün listesi herkese açık. Buradaki ayrımı baştan netleştirmek lazım, yoksa "hangi endpoint açık, hangisi kapalı" sorusunun cevabı zamanla kimsenin bilmediği bir şeye dönüşüyor.

Sağlık kontrolünü de ilk günden aç. Kapalıyken çöken bir örneğe (instance) istek gitmeye devam ediyor ve kullanıcıların bir kısmı sebepsiz hata alıyor. Bunu canlıda öğrenmek can sıkıcı oluyor.

### Kullanıcı Kimliğini Arka Tarafa Taşımak

Gateway token'ı doğruladıktan sonra kullanıcı kimliğini başlığa koyuyor. Kritik nokta, istemciden gelen aynı isimli başlığı **önce silmek**:

```csharp
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"))
    .AddTransforms(builderContext =>
    {
        builderContext.AddRequestTransform(ctx =>
        {
            // İstemci bu başlığı kendi göndermiş olabilir, önce temizliyoruz
            ctx.ProxyRequest.Headers.Remove("X-Kullanici-Id");
            ctx.ProxyRequest.Headers.Remove("X-Kiracı-Id");

            var kullaniciId = ctx.HttpContext.User.FindFirst("sub")?.Value;
            if (!string.IsNullOrEmpty(kullaniciId))
                ctx.ProxyRequest.Headers.Add("X-Kullanici-Id", kullaniciId);

            // İzleme için korelasyon kimliği, yoksa üretiyoruz
            var izId = ctx.HttpContext.TraceIdentifier;
            ctx.ProxyRequest.Headers.Remove("X-Correlation-Id");
            ctx.ProxyRequest.Headers.Add("X-Correlation-Id", izId);

            return ValueTask.CompletedTask;
        });
    });
```

O iki `Remove` satırı boşuna değil. Bir güvenlik testinde `X-Kullanici-Id` başlığını elle set edip başkasının siparişlerini görmeyi denemişlerdi ve o gün gateway o başlığı temizlemiyordu. Neyse ki test ortamıydı.

### Rate Limit

.NET 8 ile gelen yerleşik rate limiter iş görüyor:

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(ctx =>
    {
        // Giriş yapmışsa kullanıcı bazlı, yapmamışsa IP bazlı sınırlıyoruz
        var anahtar = ctx.User.FindFirst("sub")?.Value
                      ?? ctx.Connection.RemoteIpAddress?.ToString()
                      ?? "anonim";

        return RateLimitPartition.GetTokenBucketLimiter(anahtar, _ => new TokenBucketRateLimiterOptions
        {
            TokenLimit = 100,
            TokensPerPeriod = 100,
            ReplenishmentPeriod = TimeSpan.FromMinutes(1),
            QueueLimit = 0,
            AutoReplenishment = true
        });
    });

    options.OnRejected = async (ctx, ct) =>
    {
        ctx.HttpContext.Response.Headers.RetryAfter = "60";
        await ctx.HttpContext.Response.WriteAsJsonAsync(
            new { hata = "Çok fazla istek gönderdiniz." }, ct);
    };

    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
});

app.UseRateLimiter();
```

Burada büyük bir uyarı var: bu sayaç **bellekte** tutuluyor. Gateway'i 4 pod ile çalıştırıyorsan gerçek sınırın dakikada 100 değil 400 oluyor. Biz bunu fark etmemiştik, "sınır neden tutmuyor" diye epey aradık. Gerçekten sıkı bir sınır lazımsa sayacı Redis'e taşımak gerekiyor.

`Retry-After` başlığını da koymayı unutma. Koymazsan istemciler ne zaman tekrar deneyeceklerini bilemiyor ve genelde hemen tekrar deniyorlar, bu da işi büsbütün kötüleştiriyor.

### Dayanıklılık: Timeout, Retry, Devre Kesici

```csharp
builder.Services.AddHttpClient("ic-servisler")
    .AddStandardResilienceHandler(o =>
    {
        o.AttemptTimeout.Timeout = TimeSpan.FromSeconds(3);
        o.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(10);
        o.Retry.MaxRetryAttempts = 2;
        o.CircuitBreaker.FailureRatio = 0.5;
        o.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(30);
    });
```

Retry sayısını 2'de bırakmamın sebebi aşağıdaki tuzaklar bölümünde. Kısaca: katman katman retry çarpılıyor.

### Toplama (Aggregation)

Mobilin 11 isteği meselesini böyle çözmüştük — tek endpoint, arkada paralel çağrılar:

```csharp
app.MapGet("/bff/anasayfa", async (IHttpClientFactory fabrika, ILogger<Program> log, CancellationToken ct) =>
{
    var http = fabrika.CreateClient("ic-servisler");

    var kampanyalar = Dayanikli(http.GetFromJsonAsync<List<KampanyaDto>>("/kampanyalar/aktif", ct), new List<KampanyaDto>(), log);
    var kategoriler = Dayanikli(http.GetFromJsonAsync<List<KategoriDto>>("/kategoriler", ct), new List<KategoriDto>(), log);
    var oneriler     = Dayanikli(http.GetFromJsonAsync<List<UrunDto>>("/oneriler", ct), new List<UrunDto>(), log);

    await Task.WhenAll(kampanyalar, kategoriler, oneriler);

    return Results.Ok(new
    {
        Kampanyalar = await kampanyalar,
        Kategoriler = await kategoriler,
        Oneriler    = await oneriler
    });
});

// Kritik olmayan parçalar hata verirse sayfayı komple düşürmüyoruz
static async Task<T> Dayanikli<T>(Task<T?> gorev, T varsayilan, ILogger log) where T : class
{
    try
    {
        return await gorev ?? varsayilan;
    }
    catch (Exception ex)
    {
        log.LogWarning(ex, "Ana sayfa parçası alınamadı, varsayılan dönülüyor");
        return varsayilan;
    }
}
```

Buradaki asıl fikir `Task.WhenAll` değil, o `Dayanikli` sarmalayıcısı. Öneri servisi çöktüğünde ana sayfanın komple 500 dönmesi yerine öneriler bölümü boş geliyor, kullanıcı alışverişine devam edebiliyor. Kampanya servisi için aynı şeyi yapar mıydım — ona iş tarafıyla konuşmak lazım, her parça eşit kritiklikte değil.

## Performans

1. **Ek bir atlama ekliyorsun.** Gateway araya girdiği için her isteğe birkaç milisaniye biniyor. Sağlıklı bir kurulumda 1-3 ms civarı, kabul edilebilir. Ama gateway'de senkron bir iş yapıyorsan (mesela her istekte veritabanına bakıp yetki sorgulamak) bu sayı 30 ms'ye çıkabiliyor ve fark ediliyor.

2. **Bağlantıları tekrar kullan.** Gateway arka tarafa `HttpClient` ile gidiyorsa her istekte yeni `HttpClient` üretmek klasik hata. `IHttpClientFactory` kullan, mümkünse HTTP/2 aç — çoklama (multiplexing) sayesinde bağlantı sayısı ciddi düşüyor.

3. **Gövdeyi tamponlama.** Büyük dosyaları vekillerken gateway cevabın tamamını belleğe alırsa hem gecikme artıyor hem bellek şişiyor. Akış (streaming) modunda geçir. Dosya indirmeyi gateway üzerinden hiç geçirmemek de bir seçenek — imzalı URL verip istemciyi doğrudan depolamaya yönlendirmek daha ucuz.

4. **Cache'i doğru yere koy.** Değişmeyen veriler (kategoriler, il listesi, kur bilgisi) için gateway seviyesinde kısa süreli cache en ucuz kazanç. Bizde kategori isteklerinin %90'ı 60 saniyelik cache ile arka tarafa hiç gitmiyordu.

5. **Gateway durumsuz kalsın.** Oturum bilgisi, sayaç, geçici veri gibi şeyleri gateway'in belleğinde tutarsan yatay ölçekleyemezsin. Ne tutman gerekiyorsa Redis'e koy.

6. **Sıkıştırmayı tek yerde yap.** Her servis kendi cevabını sıkıştırıp gateway açıp tekrar sıkıştırıyorsa boşa CPU yakıyorsun. Bir yerde karar ver.

## Tuzaklar

- **Gateway'e iş mantığı sızıyor.** En sinsi tuzak bu. Bir gün "şu alanı da burada hesaplayıversek" diye başlıyor, iki yıl sonra gateway'de kimsenin dokunmaya cesaret edemediği 4 bin satır kod oluyor. Dağıtık monolit dediğimiz şey tam olarak bu. Kural basit: gateway yönlendirir, doğrular, sınırlar. Hesaplamaz.

- **Tek nokta arıza.** Gateway çökerse her şey çöker. En az iki örnek çalıştır, sağlık kontrolü koy, dağıtımı kademeli yap. Bunun bedelini bir kez ödeyip öğrendik.

- **Retry çarpanı.** Bir gece ödeme servisi yavaşladı. Gateway 3 kez deniyordu, gateway'in çağırdığı servis de kendi içinde 3 kez deniyordu. Tek bir kullanıcı isteği arka tarafta 9 çağrıya dönüştü ve zaten zorlanan servis tamamen kapandı. Retry'ı sadece bir katmanda yap, üstüne mutlaka jitter ekle.

- **Timeout hiyerarşisi.** İstemcinin timeout'u gateway'inkinden uzun, gateway'inki arka servisinkinden uzun olmalı. Ters kurarsan istemci bağlantıyı kesiyor ama arka tarafta iş çalışmaya devam ediyor — kaynak yiyor, üstelik sonucu kimseye ulaşmıyor.

- **429'u izlemiyorsun.** Rate limit koyup metriğini toplamamak sık yapılan bir şey. Bir istemci sürekli sınıra takılıyorsa ya kötü niyetli ya da senin sınırın yanlış. İkisini de sadece grafikte görebilirsin.

- **Korelasyon kimliği yok.** Bir istek 4 servise dağılıyorsa, ortak bir kimlik olmadan sorunu ayıklamak imkânsıza yakın. Gateway bu kimliği üretip iletmenin en doğru yeri, çünkü isteği ilk gören o.

- **Sürümleme.** `/api/v1` ile başla. "Sonra ekleriz" deyip başlamayanlar sonra hiç ekleyemiyor.

- **Dağıtım korkusu.** Gateway'in dağıtımı bütün sistemi etkiliyor, bu yüzden ekip zamanla ona dokunmaktan çekinmeye başlıyor. Panzehiri, konfigürasyonu koddan ayırmak: yönlendirme değişikliği için yeni sürüm çıkmak zorunda kalmıyorsun.

---

Esasen gateway'i "her şeyi çözecek katman" olarak değil, "tekrar eden işleri toplayan ince bir kabuk" olarak düşünmek gerekiyor. İnce kaldığı sürece hayat kolay; kalınlaştıkça kaçtığın monolite geri dönüyorsun.

Servis sayınız az ve ekip küçükse acele etmeyin, gateway'in de bir bakım maliyeti var. Ama aynı kimlik doğrulama kodunu üçüncü kez kopyala-yapıştır yaptığınızı fark ettiyseniz, vakit gelmiş demektir.

Faydalı olması dileğiyle.
