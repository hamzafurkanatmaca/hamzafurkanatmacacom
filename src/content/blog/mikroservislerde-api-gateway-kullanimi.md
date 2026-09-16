---

title: "Mikroservislerde API Gateway Kullanımı"
description: "Gateway ne işe yarıyor, nerede işleri kolaylaştırıyor ve yanlış kullanıldığında nasıl ayrı bir probleme dönüşüyor?"
pubDate: "Aug 31 2026"
heroImage: "/images/api-gateway-mimarisi.svg"
---------------------------------------------

API Gateway'i ilk kez teoriden değil, zorunluluktan kullanmaya başladım. Bir e-ticaret projesinde mobil taraftaki arkadaşlarla ana sayfa açılırken 11 ayrı API isteği yapıldığını tespit ettik. İlk başta sayı biraz abartılı geldi ama detaylı inceleyince gerçekten öyle olduğunu gördük. Kampanyalar bir servisten, kategoriler başka bir servisten, sepet bilgisi başka bir yerden, kullanıcı bilgileri ve öneriler ise başka servislerden geliyordu.

Asıl sıkıntı sadece istek sayısı da değildi. Bu servislerin adresleri mobil uygulamanın içinde tutuluyordu. Backend tarafında bir servisin adresini veya portunu değiştirdiğimizde mobil uygulamaya yeni sürüm çıkılması gereken durumlar oluyordu. Bir noktadan sonra “bunun önüne bir katman koymamız lazım” dedik ve gateway kullanmaya başladık.

Gateway birçok sorunu gerçekten çözdü ama zamanla başka bir şeyi de öğrendim: yanlış kullanılırsa kendisi yeni bir probleme dönüşebiliyor.

## Gateway'e Ne Zaman İhtiyaç Oluyor?

Üç dört servisten oluşan küçük bir sistemde gateway olmadan gayet rahat yaşayabilirsin. Hatta sırf yapı güzel görünsün diye araya gateway koymak gereksiz bir karmaşıklık bile yaratabilir.

Servis sayısı büyüdükçe durum değişiyor. Bir süre sonra mobil uygulama servis adreslerini biliyor, web uygulaması biliyor, dış entegrasyon yapan sistemler biliyor. Bir servisi taşımak veya ikiye bölmek istediğinde aslında servis tarafındaki değişikliğin yanında bütün istemcileri de düşünmek zorunda kalıyorsun.

Mikroservis mimarisinin amaçlarından biri servisleri birbirinden bağımsız hale getirmek ama bu sefer istemcileri servislerin iç yapısına bağımlı hale getirmiş oluyorsun. Gateway burada güzel bir sınır oluşturuyor. Dış dünya tek bir adres biliyor, içeride hangi servisin nerede çalıştığı ise istemciyi ilgilendirmiyor.

Bizde gateway ihtiyacını belirgin hale getiren başka bir konu da, tekrar eden kodlar olmuştu. Bir dönem yedi farklı servis vardı ve neredeyse hepsinde ayrı ayrı JWT doğrulama kodu bulunuyordu. Daha sonra yapılan bir kontrolde servislerden birinin token'ın `exp` alanını kontrol etmediğini fark ettik. Yani süresi geçmiş bir token o serviste hâlâ geçerli olabiliyordu.

İlginç olan, kodun review'dan geçmiş olmasıydı. Aynı kod birkaç serviste tekrarlandığı zaman insanın gözü de alışıyor. “Bu zaten diğer servistekiyle aynı” deyip daha yüzeysel bakmaya başlıyorsun.

Authentication, rate limit, CORS, logging ve benzeri servislerin asıl işi olmayan ama her serviste tekrar eden konuları mümkün olduğunca ortak bir noktada çözmenin değerini orada daha net gördüm.

## Gateway'in Sistemdeki Yeri

![Mikroservis mimarisinde API Gateway'in konumu](/images/api-gateway-mimarisi.svg)

Ben gateway'i sistemin giriş kapısı gibi düşünüyorum. İstemciden gelen istek önce buraya geliyor. Gateway gerekliyse kullanıcının kimliğini doğruluyor, rate limit kontrolünü yapıyor, isteğin hangi servise gitmesi gerektiğine karar veriyor ve sonra iç ağa yönlendiriyor.

İstemci böylece sistemin içerisindeki servis yapısını bilmek zorunda kalmıyor.

Buradaki en önemli konu bence gateway'in görevini fazla büyütmemek. Başlangıçta sadece routing yapıyor. Sonra authentication geliyor, ardından rate limit ekleniyor. Bunlar doğal.

Sorun genelde bir süre sonra “hazır bütün trafik buradan geçiyor, şu hesabı da burada yapalım” denmesiyle başlıyor. Ardından küçük bir `if`, bir mapping, birkaç iş kuralı derken gateway'in içinde ciddi miktarda uygulama kodu oluşuyor.

Bir süre sonra mikroservisleri birbirinden ayırmaya çalışırken bütün sistemin ortasına yeni bir monolit koymuş oluyorsun.

Bu yüzden benim kafamdaki sınır basit: gateway yönlendirir, doğrular ve sınırlar. Mümkün olduğunca domain hesabı yapmaz.

## Routing ve Ortak İşler

Gateway'in en temel yaptığı iş routing. Örneğin dışarıdan gelen:

```text
/api/siparisler/*
```

isteklerini sipariş servisine,

```text
/api/urunler/*
```

isteklerini ürün servisine gönderebilirsin.

Bu basit gibi görünse de önemli bir soyutlama sağlıyor. Sipariş servisini yarın başka bir sunucuya taşısan veya içerideki URL yapısını değiştirsen istemcinin bunu bilmesine gerek kalmıyor.

Gateway'in diğer önemli kullanım alanı ise birden fazla servisi ilgilendiren ortak işler. Authentication, authorization, rate limit, CORS ve request correlation bunun klasik örnekleri.

Tabii her şeyi gateway'e taşımak da doğru değil. Örneğin authorization'ın bir kısmını gateway seviyesinde yapmak mantıklı olabilir ama “bu kullanıcı bu siparişi gerçekten görebilir mi?” gibi domain'e bağlı bir kontrolün çoğu zaman ilgili serviste kalması gerekir.

Gateway'in her şeyi bilmesi gerektiğini düşünmeye başladığın anda sınırlar belirsizleşmeye başlıyor.

## BFF ve Aggregation

Gateway ile beraber en çok kullandığım yaklaşımlardan biri de BFF, yani Backend for Frontend oldu.

Mobil uygulamanın ihtiyacıyla web uygulamasının ihtiyacı her zaman aynı olmuyor. Mobil tarafta daha az veri, daha küçük response ve daha az network isteği önemliyken web tarafında aynı ekran çok daha fazla bilgi gösterebiliyor.

Bu durumda herkese tek bir “genel API” vermeye çalışmak bazen iki tarafı da mutsuz ediyor.

BFF burada özellikle ekran bazlı ihtiyaçlarda işe yarıyor. İstemci beş farklı servise ayrı ayrı gitmek yerine BFF'ye tek bir istek gönderiyor, BFF de arka tarafta gerekli servislerle konuşup istemcinin ihtiyacı olan sonucu oluşturuyor.

Ama burada da sınır önemli. BFF birkaç servisten veri toplayıp istemciye uygun hale getiriyorsa güzel. İş kurallarının büyük bölümü oraya taşınmaya başladıysa yine başka bir probleme gidiyoruz.

## Hangi Gateway?

.NET ağırlıklı çalışan bir ekipte bugün ilk bakacağım çözüm YARP olur.

YARP'ın sevdiğim tarafı ayrı bir teknoloji dünyası kurmak zorunda bırakmaması. Normal bir ASP.NET uygulamasına ekleniyor, routing yapılandırmasını konfigürasyondan okuyabiliyor ve özel bir davranış gerektiğinde doğrudan C# ile müdahale edebiliyorsun.

.NET bilen bir ekip için öğrenme maliyeti oldukça düşük.

Dezavantajı ise Kong gibi ürünlerde hazır bulunan birçok şeyin YARP'ta kutudan çıkmaması. Yönetim paneli, developer portal veya büyük bir plugin ekosistemi bekliyorsan bazı parçaları kendin geliştirmen gerekiyor.

Ocelot da .NET tarafında uzun süre çok kullanılan bir çözümdü. Hâlâ çalışan sistemlerde görmek mümkün ve sırf daha yeni bir alternatif çıktı diye mevcut çalışan sistemi sökmek bana çok mantıklı gelmiyor. Fakat sıfırdan bir proje açıyor olsam ben YARP tarafına giderdim.

Nginx ve Kong gibi çözümler ise özellikle farklı teknolojilerin aynı sistemde kullanıldığı yapılarda avantajlı hale geliyor. Bir servis .NET, biri Go, diğeri Node ise gateway'in uygulama framework'ünden bağımsız olması güzel. Kong'un plugin yapısı sayesinde rate limit, authentication veya request transformation gibi birçok özellik hazır geliyor.

Bunun karşılığında operasyon tarafında ayrı bir uzmanlık oluşabiliyor. Standart konfigürasyonun dışına çıktığında ekipte sistemi gerçekten bilen insan sayısı azalabiliyor. Kritik bir altyapının tek kişinin bildiği bir şeye dönüşmesi de benim çok sevdiğim bir durum değil.

Dış dünyaya ciddi anlamda API açılan sistemlerde Azure API Management veya AWS API Gateway gibi managed çözümler başka avantajlar sunuyor. Partner yönetimi, API key, kullanım raporu, kota, developer portal gibi ihtiyaçlar varsa hazır gelmeleri ciddi zaman kazandırıyor. Tabii bunun karşılığında maliyet ve bulut sağlayıcısına bağımlılık artıyor.

Envoy ise çok güçlü ama benim gözümde daha çok gerçekten o güce ihtiyacı olan sistemler için uygun. Trafiğin çok büyük olmadığı, birkaç servisten oluşan küçük bir sistemde sırf güçlü diye Envoy kullanmanın fazla olduğunu düşünüyorum.

## Kendi Gateway'ini Yazmak

İlk bakışta kendi gateway'ini yazmak oldukça kolay görünüyor. İsteği alıyorsun, `HttpClient` ile diğer servise gönderiyorsun, response'u geri dönüyorsun.

İlk sürüm gerçekten bir günde bile çıkabilir.

Sonra WebSocket ihtiyacı geliyor. Ardından streaming, büyük dosya upload'ları, timeout yönetimi, connection pooling ve `X-Forwarded-*` başlıkları çıkıyor.

Bir noktadan sonra fark ediyorsun ki kendi reverse proxy ürününü geliştirmeye başlamışsın.

Bu yüzden özel bir sebep yoksa hazır ve yıllardır kullanılan çözümler varken kendi gateway altyapımı yazmayı tercih etmem.

## YARP ile Basit Routing

Basit bir YARP yapılandırması şu şekilde olabilir:

```json
{
  "ReverseProxy": {
    "Routes": {
      "siparis-route": {
        "ClusterId": "siparis",
        "AuthorizationPolicy": "dogrulanmis",
        "Match": {
          "Path": "/api/siparisler/{**catch-all}"
        },
        "Transforms": [
          {
            "PathPattern": "/siparisler/{**catch-all}"
          }
        ]
      },
      "urun-route": {
        "ClusterId": "urun",
        "Match": {
          "Path": "/api/urunler/{**catch-all}"
        },
        "Transforms": [
          {
            "PathPattern": "/urunler/{**catch-all}"
          }
        ]
      }
    },
    "Clusters": {
      "siparis": {
        "LoadBalancingPolicy": "PowerOfTwoChoices",
        "Destinations": {
          "d1": {
            "Address": "http://siparis-servis:8080/"
          }
        },
        "HealthCheck": {
          "Active": {
            "Enabled": true,
            "Interval": "00:00:10",
            "Path": "/health"
          }
        }
      },
      "urun": {
        "Destinations": {
          "d1": {
            "Address": "http://urun-servis:8080/"
          }
        }
      }
    }
  }
}
```

Kod tarafında da çok fazla şey gerekmiyor:

```csharp
builder.Services
    .AddReverseProxy()
    .LoadFromConfig(
        builder.Configuration.GetSection("ReverseProxy"));

app.UseAuthentication();
app.UseAuthorization();

app.MapReverseProxy();
```

Burada ürün endpoint'ini bilerek authentication zorunluluğu olmadan bıraktım. Ürün listesi herkese açıkken sipariş servisinin açık olması doğal olarak istemeyeceğimiz bir durum.

Bu tarz ayrımları daha en başta belirgin hale getirmek önemli. Aksi halde sistem büyüdüğünde “bu endpoint public miydi?” sorusunun cevabını farklı dosyalardan araştırmaya başlıyorsun.

Health check de bence ilk günden düşünülmesi gereken şeylerden biri. Birden fazla instance varsa ve bir tanesi çökmüşse gateway'in hâlâ ona trafik göndermesi çok garip kullanıcı hatalarına yol açabiliyor. Bazı kullanıcılar sorunsuz çalışırken bazıları sürekli 500 alıyor.

Bu da “bende çalışıyor” probleminin dağıtık sistem versiyonu oluyor.

## Kullanıcı Bilgisini Servislere Taşımak

Gateway token'ı doğruladıktan sonra kullanıcı bilgisini arka servislere header üzerinden geçirmek kullanılan yöntemlerden biri.

Burada önemli bir güvenlik detayı var. İstemcinin gönderdiği aynı isimdeki header'a güvenmemek gerekiyor.

Örneğin:

```csharp
builder.Services
    .AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"))
    .AddTransforms(builderContext =>
    {
        builderContext.AddRequestTransform(ctx =>
        {
            ctx.ProxyRequest.Headers.Remove("X-Kullanici-Id");
            ctx.ProxyRequest.Headers.Remove("X-Kiraci-Id");

            var kullaniciId =
                ctx.HttpContext.User.FindFirst("sub")?.Value;

            if (!string.IsNullOrEmpty(kullaniciId))
            {
                ctx.ProxyRequest.Headers.Add(
                    "X-Kullanici-Id",
                    kullaniciId);
            }

            var correlationId =
                ctx.HttpContext.TraceIdentifier;

            ctx.ProxyRequest.Headers.Remove(
                "X-Correlation-Id");

            ctx.ProxyRequest.Headers.Add(
                "X-Correlation-Id",
                correlationId);

            return ValueTask.CompletedTask;
        });
    });
```

Bizde bir güvenlik testinde buna benzer bir konu yakalanmıştı. İstemciden `X-Kullanici-Id` göndererek başka bir kullanıcı gibi davranmayı denemişlerdi. O dönemde gateway gelen başlığı düzgün biçimde temizlemiyordu.

Neyse ki test ortamında fark edildi.

Bu yöntemi kullanırken sadece gateway koduna güvenmek de yeterli değil. Arka servislerin dış ağdan doğrudan erişilebilir olmaması gerekiyor. Aksi halde saldırgan gateway'i tamamen atlayıp servise doğrudan kendi `X-Kullanici-Id` header'ını gönderir.

Yani güvenlik burada uygulama koduyla ağ mimarisinin birlikte çözmesi gereken bir konu.

## Rate Limit

Rate limit gateway'e oldukça doğal oturan özelliklerden biri. Kullanıcı giriş yapmışsa kullanıcı kimliğine göre, anonim kullanıcıysa IP adresine göre limit uygulanabilir.

.NET tarafında yerleşik rate limiter ile bunu rahat şekilde yapmak mümkün:

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter =
        PartitionedRateLimiter.Create<HttpContext, string>(ctx =>
        {
            var anahtar =
                ctx.User.FindFirst("sub")?.Value
                ?? ctx.Connection.RemoteIpAddress?.ToString()
                ?? "anonim";

            return RateLimitPartition.GetTokenBucketLimiter(
                anahtar,
                _ => new TokenBucketRateLimiterOptions
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
            new { hata = "Çok fazla istek gönderdiniz." },
            ct);
    };

    options.RejectionStatusCode =
        StatusCodes.Status429TooManyRequests;
});

app.UseRateLimiter();
```

Burada benim daha önce düştüğüm önemli bir tuzak var. Bu sayaç local memory'de tutuluyorsa tek gateway instance'ında koyduğun limit ile dört gateway instance'ında koyduğun limit aynı anlama gelmiyor.

Dakikada 100 istek sınırı koyduğunu düşünüyorsun ama dört pod olduğunda kullanıcı farklı instance'lara dağılarak teorik olarak çok daha fazla istek geçirebiliyor.

Bizde bir dönem “limit koyduk ama neden tam tutmuyor?” diye bakarken bunu fark etmiştik.

Gerçekten global bir rate limit gerekiyorsa state'in ortak bir yerde tutulması gerekiyor. Redis bu iş için kullanılabilecek seçeneklerden biri.

`Retry-After` header'ını dönmek de küçük ama faydalı bir detay. Sadece 429 dönersen bazı istemciler hiçbir şey olmamış gibi hemen tekrar deneyebiliyor ve zaten yük altında olan sistemi daha fazla zorlayabiliyor.

## Retry ve Circuit Breaker

Retry mekanizması doğru yerde oldukça faydalı ama fazla kullanıldığında tam tersine sistemi daha kötü hale getirebiliyor.

Bir dönem ödeme servisinde buna benzer bir olay yaşadık. Gateway başarısız istekleri tekrar deniyordu, gateway'in çağırdığı servis kendi downstream çağrılarını tekrar deniyordu ve oradaki HTTP client'ın da ayrı retry politikası vardı.

Sonuçta kullanıcıdan gelen tek bir istek arka tarafta birçok çağrıya dönüşüyordu.

Servis zaten yavaşladığı için retry devreye giriyor, retry devreye girdikçe servis daha fazla yük alıyor ve daha da yavaşlıyordu.

Bu yüzden retry ayarını tek bir servise bakarak değil, çağrı zincirinin tamamını düşünerek yapmak gerekiyor.

Örneğin:

```csharp
builder.Services
    .AddHttpClient("ic-servisler")
    .AddStandardResilienceHandler(o =>
    {
        o.AttemptTimeout.Timeout =
            TimeSpan.FromSeconds(3);

        o.TotalRequestTimeout.Timeout =
            TimeSpan.FromSeconds(10);

        o.Retry.MaxRetryAttempts = 2;

        o.CircuitBreaker.FailureRatio = 0.5;

        o.CircuitBreaker.SamplingDuration =
            TimeSpan.FromSeconds(30);
    });
```

Burada retry sayısının iki olması herhangi bir evrensel kural değil. Asıl mesele zincirde başka hangi katmanların tekrar denediğini bilmek.

Circuit breaker da bu noktada faydalı oluyor. Bir servis sürekli hata veriyorsa her kullanıcı isteğinde aynı servise tekrar tekrar gitmek yerine bir süre çağrıyı kesmek hem kullanıcıyı gereksiz timeout beklemekten kurtarıyor hem de sorun yaşayan servise toparlanma şansı veriyor.

## Mobil Uygulamadaki 11 İstek

Başta anlattığım mobil ana sayfa problemine dönelim.

Burada BFF yaklaşımı kullandık. Mobil uygulama artık kampanyalar, kategoriler ve öneriler için ayrı ayrı istek yapmak yerine tek endpoint'e gidiyordu. BFF arka tarafta gerekli servis çağrılarını paralel olarak yapıyordu.

Örneğin:

```csharp
app.MapGet(
    "/bff/anasayfa",
    async (
        IHttpClientFactory fabrika,
        ILogger<Program> log,
        CancellationToken ct) =>
    {
        var http =
            fabrika.CreateClient("ic-servisler");

        var kampanyalar = Dayanikli(
            http.GetFromJsonAsync<List<KampanyaDto>>(
                "/kampanyalar/aktif",
                ct),
            new List<KampanyaDto>(),
            log);

        var kategoriler = Dayanikli(
            http.GetFromJsonAsync<List<KategoriDto>>(
                "/kategoriler",
                ct),
            new List<KategoriDto>(),
            log);

        var oneriler = Dayanikli(
            http.GetFromJsonAsync<List<UrunDto>>(
                "/oneriler",
                ct),
            new List<UrunDto>(),
            log);

        await Task.WhenAll(
            kampanyalar,
            kategoriler,
            oneriler);

        return Results.Ok(new
        {
            Kampanyalar = await kampanyalar,
            Kategoriler = await kategoriler,
            Oneriler = await oneriler
        });
    });
```

Burada bence önemli olan `Task.WhenAll` kullanmak değil. Asıl önemli karar, arka servislerden biri çalışmadığında ne olacağı.

Öneri servisi çöktü diye kullanıcının bütün ana sayfasını 500 ile düşürmek çoğu durumda mantıklı değil. Öneriler boş gelir, kullanıcı kategorileri görmeye ve alışveriş yapmaya devam eder.

Ama fiyat servisi için aynı yaklaşımı kullanamayabilirsin.

Her downstream servis aynı kritiklikte değil. Bunun cevabı da çoğu zaman teknik ekip tarafından tek başına verilemez; iş tarafıyla birlikte karar vermek gerekir.

## Performans Tarafı

Gateway her request'e doğal olarak bir network adımı daha ekliyor. Düzgün çalışan bir reverse proxy'nin eklediği birkaç milisaniye çoğu sistemde kabul edilebilir.

Sorun gateway'in içinde ağır işler yapmaya başladığında çıkıyor.

Örneğin her istek geldiğinde veritabanına gidip authorization bilgisi okumaya başlarsan artık yalnızca reverse proxy çalıştırmıyorsun. Trafiğin tamamının geçtiği yeni bir backend servisi oluşturmuş oluyorsun.

Bu nedenle gateway tarafında mümkün olduğunca I/O maliyetlerini düşük tutmaya çalışıyorum.

Connection reuse da önemli. Her request'te yeni `HttpClient` oluşturmak klasik bir hata ama gateway gibi bütün sistem trafiğini taşıyan bir uygulamada etkisi çok daha hızlı hissediliyor. `IHttpClientFactory`, connection pooling ve mümkün olduğunda HTTP/2 kullanımı burada faydalı.

Büyük dosyalarda buffering konusu da gözden kaçabiliyor. Gateway 500 MB'lık dosyanın tamamını belleğe alıp ardından istemciye göndermeye çalışırsa trafik arttığında memory kullanımı çok hızlı büyüyebilir. Streaming kullanmak veya bazı senaryolarda istemciyi signed URL ile doğrudan object storage'a yönlendirmek daha doğru olabilir.

Cache tarafında ise özellikle çok sık değişmeyen ve çok okunan verilerde basit çözümler ciddi trafik azaltabiliyor. Kategori listesi gibi saniye saniye değişmeyen bir veriyi kısa süre cache'lemek backend'e giden çağrıların büyük bölümünü ortadan kaldırabiliyor.

Yine de gateway'in local belleğinde çok fazla state tutmamak gerekiyor. Sayaçlar, session bilgileri veya instance'a özel state yatay ölçeklemede baş ağrıtabiliyor. Gateway mümkün olduğunca stateless kaldığında ölçeklemek çok daha kolay oluyor.

## Production'da En Çok Sorun Çıkaran Yerler

Benim gördüğüm en büyük tehlike gateway'in zamanla uygulama mantığını içine çekmesi. İlk değişiklik çok masum oluyor. Küçük bir mapping ekleniyor. Daha sonra bir hesaplama geliyor. Ardından başka bir ekip “zaten veri buradan geçiyor” diyerek kendi ihtiyacını ekliyor.

Bir süre sonra gateway'in içinde birkaç bin satırlık iş kuralı oluşabiliyor ve bütün ekip oraya dokunmaktan çekinmeye başlıyor.

Gateway bütün sistemin giriş noktası olduğu için deployment tarafı da hassas. Gateway çökerse sağlıklı çalışan arka servislerin hiçbir anlamı kalmıyor. Bu nedenle tek instance ile çalıştırmak, health check olmadan deployment yapmak veya rollback planını düşünmemek riskli.

Timeout ayarları da bütün sistem boyunca beraber düşünülmeli. İstemci beş saniyede bağlantıyı kapatıyor ama gateway otuz saniye bekliyor, arka servis ise bir dakika çalışmaya devam ediyorsa kullanıcı çoktan gitmiş olmasına rağmen sistem gereksiz yere kaynak tüketmeye devam eder.

Benzer şekilde rate limit koymak ama 429 metriklerini toplamamak da sık gördüğüm bir durum. Bir istemci sürekli limite takılıyorsa bunun sebebi saldırı olabilir, hatalı yazılmış bir mobil uygulama olabilir veya gerçekten senin limitin yanlış olabilir. Metric yoksa hangisi olduğunu ancak kullanıcı şikâyet ettiğinde fark ediyorsun.

Dağıtık bir yapıda correlation ID de neredeyse zorunlu hale geliyor. Gateway'den başlayan bir istek birkaç farklı servise dağıldığında ortak bir kimlik yoksa logların arasından aynı request'i takip etmek oldukça zorlaşıyor. İsteğin sisteme ilk girdiği yer gateway olduğu için correlation ID üretmek veya var olan kimliği taşımak için doğal bir nokta.

## Sonuç

API Gateway'i mikroservis mimarisinin zorunlu bir parçası olarak görmüyorum. Sistem küçükse, birkaç servis varsa ve istemcilerle servislerin ilişkisi basitse gateway eklemek gereksiz bir operasyon maliyeti getirebilir.

Ama servis sayısı arttıkça bazı belirtiler ortaya çıkıyor. İstemciler artık içeride hangi servisin nerede olduğunu bilmeye başlıyor, authentication kodları servisler arasında kopyalanıyor, CORS ve logging ayarları her yerde tekrar ediyor, mobil uygulama tek ekran için birçok backend isteği yapıyor veya dışarı açılan servis sayısı arttıkça güvenlik tarafı yönetilmesi zor hale geliyor.

Bu noktadan sonra gateway gerçekten fayda sağlamaya başlıyor.

Benim için kritik nokta ise mümkün olduğunca ince kalması.

Gateway sistemin giriş kapısı olabilir.

Ama sistemin kendisi olmamalı.

Çünkü içine sürekli biraz daha iş mantığı koymaya başladığın anda, mikroservislere geçerken kaçmaya çalıştığın monoliti bu kez sistemin tam ortasında yeniden oluşturmuş oluyorsun.
