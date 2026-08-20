---
title: "SOAP Servislerini Modern REST API'lere Taşımak"
description: "Eski WCF servislerini adım adım REST'e çevirmek: strangler fig yaklaşımı, sözleşme dönüşümü ve üretimde çıkan tuzaklar."
pubDate: "Aug 20 2026"
heroImage: "/images/soap-rest-migration.svg"
---

Bir kurumsal projede elime 2013'ten kalma bir WSDL geçmişti. Açtım baktım, 4 bin küsur satır. İçinde 60'a yakın operasyon var ve yarısının adı `GetMusteriV2`, `GetMusteriV2Son`, `GetMusteriV2SonYeni` gibi şeyler. Açıkçası ilk tepkim "bunu kim yazdıysa artık burada çalışmıyordur" oldu. Doğruymuş da. O günden sonra bu tarz servisleri sıfırdan yazmak yerine yavaş yavaş devralmak gerektiğini öğrendim.

## Neden Hâlâ SOAP Var?

Yeni bir projede kimse SOAP seçmiyor, bu doğru. Ama bankacılıkta, sigortada, kamu entegrasyonlarında hâlâ bir sürü SOAP servisi dönüyor. Sebep basit: o servisler çalışıyor ve kimse dokunmaya cesaret edemiyor.

Bir de SOAP'ın gerçekten iyi yaptığı şeyler vardı. WS-Security ile mesajın kendisini imzalayıp şifreleyebiliyorsun, WS-AtomicTransaction ile iki farklı servis arasında dağıtık transaction kurabiliyorsun. REST'te bunların birebir karşılığı yok. Kurumsal tarafta bu maddeler bazen sözleşmede yazılı oluyor, "biz artık REST'e geçtik" deyip geçemiyorsun.

Sıfırdan yeniden yazma fikri kulağa hep güzel geliyor. Pratikte şöyle oluyor ama: 60 operasyonun 15'ini yazıyorsun, kalanı için "bunlar zaten kullanılmıyordur" diyorsun, canlıya çıkıyorsun, iki gün sonra muhasebeden telefon geliyor. Yaşanmış :)

## Terminoloji (REST Tarafından Gelenler İçin)

**WSDL**: Servisin sözleşmesi. OpenAPI'nin dedesi diyebiliriz. Hangi operasyonlar var, hangi tipleri alıp veriyor, hangi adrese gidiliyor — hepsi burada yazıyor. İyi tarafı makine tarafından okunabilir olması, `dotnet-svcutil` ile tek komutta istemci sınıflarını üretebiliyorsun. Kötü tarafı insan gözüyle okumanın işkence olması.

**Envelope**: Her SOAP mesajı bir zarfın içinde gider, `Header` ve `Body` diye iki kısmı vardır. Kimlik bilgisi, transaction bilgisi gibi şeyler `Header`'a, asıl veri `Body`'ye konur. REST'teki HTTP header + body ayrımının XML'e gömülmüş hali gibi düşünebilirsin.

**SOAP Fault**: SOAP'ın hata mekanizması. Burada kritik bir detay var: bir SOAP servisi hata verse bile HTTP 200 dönebiliyor. Yani status code'a bakıp "tamam, çalıştı" diyemezsin, gövdeyi parse edip içinde Fault var mı diye bakman lazım. REST'ten gelenlerin ilk çarpıldığı yer burasıdır.

**WS-Security**: Mesaj seviyesinde güvenlik. TLS taşıma katmanını şifreler, WS-Security mesajın kendisini şifreler. Aradaki fark şu: mesaj üç farklı aracıdan geçiyorsa TLS her durakta çözülür, WS-Security ile uçtan uca kapalı kalır.

**MTOM**: Büyük binary dosyaları XML'in içinde base64 olarak taşımak yerine ayrı bir parça halinde gönderme yöntemi. Base64 dosya boyutunu %33 civarı şişirdiği için, dosya taşıyan eski servislerde sık karşılaşılır.

**Binding**: Servisin hangi protokol ve güvenlik ayarlarıyla konuşacağını belirleyen yapılandırma. `basicHttpBinding`, `wsHttpBinding`, `netTcpBinding` gibi isimler görürsün. Bağlantı kurmaya çalışıp da sebebini bir türlü anlayamadığın hatalar alıyorsan, %80 ihtimalle sorun buradadır.

## Mimari

![SOAP'tan REST'e geçiş mimarisi](/images/soap-rest-migration.svg)

Akış şu: İstemci artık SOAP'a değil, senin yazdığın REST cephesine istek atıyor. Cephe isteği bir adapter katmanına veriyor, adapter JSON'u XML'e çevirip eski servise gönderiyor, dönen cevabı tekrar JSON'a çevirip geri veriyor.

Buradaki asıl kazanç istemcinin eski servisi hiç görmemesi. Arka tarafta o servisi parça parça yenisiyle değiştirdiğinde istemcinin haberi bile olmuyor. Zaten bütün mesele bu.

## Geçiş Stratejileri

**Big bang**: Hepsini bir seferde yaz, bir gece geçişi yap. Kağıt üzerinde en temiz görünen yöntem. Gerçekte ise geri dönüşü olmayan bir kumar. Ben hiç denemedim, denemek de istemem — test ortamında yakalayamadığın tek bir edge case bütün geçişi geri aldırabiliyor.

**Eksiler:** Rollback planı yapmak çok zor, risk tek bir geceye yığılıyor, ekip o gece uyumuyor.

**Strangler fig**: Şahsi favorim. Eski servisin önüne bir cephe koyuyorsun, bütün trafiği oradan geçiriyorsun. Sonra operasyonları teker teker yeni koda taşıyorsun, cephe hangi isteğin nereye gideceğine karar veriyor. En sonunda eski servise hiç trafik gitmez hale geliyor ve fişi çekiyorsun.

**Eksiler:** Geçiş dönemi uzun sürüyor, bazen yıllara yayılıyor. Bir süre iki sistemi birden bakmak zorundasın. Yönetime "ne zaman biter" sorusunu net cevaplamak da zor oluyor.

**Salt cephe**: Sadece adapter yazıyorsun, eski servise hiç dokunmuyorsun. İstemciler modern bir API görüyor, arkada her şey aynı kalıyor. Süre kısıtın varsa veya eski sisteme müdahale yetkin yoksa mantıklı.

**Eksiler:** Teknik borç olduğu yerde duruyor, sadece üstünü örtmüş oluyorsun. E bir de araya bir katman daha girdiği için gecikme artıyor tabi.

**Paralel çalıştırma**: Aynı isteği hem eskiye hem yeniye gönderip cevapları karşılaştırıyorsun. Kullanıcıya eskinin cevabını dönüyorsun, farklılıkları logluyorsun. Yeni kodun doğruluğundan emin olmak için harika bir yöntem.

**Eksiler:** Her istek iki kez çalıştığı için yük iki katına çıkıyor. Yan etkisi olan operasyonlarda (kayıt oluşturma, para transferi) kullanılamaz — yoksa aynı işi iki kere yaparsın.

## Sözleşme Dönüşümü

İşin en sinsi kısmı burası. XML'den JSON'a geçerken birebir karşılığı olmayan şeyler var ve bunlar genelde canlıda patlıyor.

**Boş değer meselesi**: XML'de bir alanın hiç olmaması, boş gelmesi (`<Ad></Ad>`) ve `xsi:nil="true"` gelmesi üç ayrı şeydir. JSON'da hepsi `null` veya `""` oluyor. Hangisinin hangisine karşılık geldiğine baştan karar verip yazılı hale getirmek lazım, yoksa herkes kendi yorumunu yapıyor.

**Tek elemanlı diziler**: XML'de bir liste tek eleman içeriyorsa, çoğu serializer onu dizi olarak değil tekil nesne olarak veriyor. JSON tarafında bir bakıyorsun bazen `{"kalem": {...}}` bazen `{"kalem": [{...}]}` geliyor. İstemci tarafında bunu handle etmek baya can sıkıcı, adapter'da normalize etmek en temizi.

**Attribute mı element mi**: XML'de `<Musteri id="5">` ile `<Musteri><Id>5</Id></Musteri>` farklı şeyler ama JSON'da ikisi de `{"id": 5}` oluyor. Geri dönüş yaparken hangisinin hangisi olduğunu bilmen gerekiyor.

**Tarihler**: `xs:dateTime` saat dilimi taşıyabilir de taşımayabilir de. Eski servisler genelde taşımıyor ve sunucunun yerel saatini varsayıyor. Sunucu Türkiye'deyken sen UTC varsayarsan 3 saat kayıyorsun. Bu hatayı bir kez yaptım, raporlar bir gün boyunca yanlış çıktı.

**Ondalık sayılar**: Türkçe'de ondalık ayracı virgül olduğu için, kültür ayarı belirtmeden `decimal` parse edince `1.234` sayısı bazı makinelerde bin iki yüz otuz dört olarak okunuyor. `CultureInfo.InvariantCulture` yazmayı unutmayalım.

WSDL'den otomatik OpenAPI üreten araçlar var ama çıktıları genelde kullanışsız oluyor — operasyon isimleri `Process`, `ProcessResponse` gibi kalıyor. Sözleşmeyi elle tasarlamak daha iyi sonuç veriyor. REST'e geçiyorsan zaten kaynak bazlı düşünmen lazım, WSDL'deki operasyon listesini olduğu gibi endpoint'e çevirmek anlamsız.

## Kod Örnekleri

### WSDL'den İstemci Üretmek

```bash
dotnet tool install --global dotnet-svcutil
dotnet-svcutil http://legacy.sirket.local/MusteriServis.svc?wsdl --outputFile Connected/MusteriServis.cs
```

Bu komut sana `MusteriServisClient` diye bir sınıf üretiyor. Üretilen dosyayı açıp okumaya çalışma, 3-4 bin satır olabiliyor. Sadece kullan.

### Adapter: SOAP Çağrısını REST'e Sarmak

```csharp
app.MapGet("/api/v1/musteriler/{id:int}", async (int id, IMusteriAdapter adapter, CancellationToken ct) =>
{
    var musteri = await adapter.GetirAsync(id, ct);
    return musteri is null ? Results.NotFound() : Results.Ok(musteri);
});

public sealed class MusteriAdapter : IMusteriAdapter
{
    private readonly MusteriServisClient _client;

    public async Task<MusteriDto?> GetirAsync(int id, CancellationToken ct)
    {
        var cevap = await _client.GetMusteriV2Async(new GetMusteriRequest { MusteriId = id });

        // Eski servis "bulunamadı" durumunu null yerine boş nesne ile dönüyor
        if (string.IsNullOrWhiteSpace(cevap.Musteri?.Ad))
            return null;

        return new MusteriDto(
            Id: cevap.Musteri.MusteriId,
            Ad: cevap.Musteri.Ad.Trim(),
            // Servis tarihi saat dilimsiz gönderiyor, yerel kabul edip UTC'ye çeviriyoruz
            KayitTarihi: DateTime.SpecifyKind(cevap.Musteri.KayitTarihi, DateTimeKind.Local).ToUniversalTime()
        );
    }
}
```

`Ad` alanına bakıp "bulunamadı" kararı vermek çirkin, farkındayım. Ama eski servis öyle yazılmış, elimizden bir şey gelmiyor. Bu tarz durumlar geçiş projelerinde sürekli çıkıyor — adapter katmanının varlık sebebi de zaten bu pisliği tek bir yerde toplamak.

### SOAP Fault'u HTTP Durum Koduna Çevirmek

```csharp
try
{
    return await _client.GetMusteriV2Async(istek);
}
catch (FaultException fault)
{
    var kod = fault.Code?.Name ?? "Bilinmeyen";

    // Eski servisin fault kodlarını HTTP karşılıklarına eşliyoruz
    var durum = kod switch
    {
        "MusteriBulunamadi" => StatusCodes.Status404NotFound,
        "YetkisizErisim"    => StatusCodes.Status403Forbidden,
        "GecersizParametre" => StatusCodes.Status400BadRequest,
        _                   => StatusCodes.Status502BadGateway
    };

    throw new UpstreamException(durum, fault.Message);
}
catch (CommunicationException)
{
    // Servise hiç ulaşılamadı, 502 uygun
    throw new UpstreamException(StatusCodes.Status502BadGateway, "Legacy servise ulaşılamadı");
}
```

Buradaki `switch` zamanla büyüyor, sonunda 30-40 satırlık bir eşleme tablosuna dönüşüyor. Ben en son bunu ayrı bir konfigürasyon dosyasına taşımıştım, kod içinde tutmaktan daha rahat oldu.

Production'da tabii bu kadarı yetmez — Polly ile retry, timeout ve circuit breaker eklemek gerekir. Ama mantık bu.

## Performans

1. **XML parse etmek pahalı**: JSON'a göre ciddi şekilde. Küçük mesajlarda fark etmiyor ama 5 MB'lık bir XML cevabı işlerken CPU'nun tepeye vurduğunu görürsün. Mümkünse sayfalama ekle.

2. **Payload boyutu**: Aynı veri XML'de JSON'un 2-3 katı yer tutuyor, namespace tekrarları yüzünden. Response compression açmak (gzip/brotli) burada baya işe yarıyor.

3. **Channel yönetimi**: WCF istemcisini her istekte `new`'lemek pahalı bir hata. Channel factory'yi tekrar kullan. Bir projede sadece bunu düzelterek ortalama yanıt süresi 400ms'den 120ms'ye inmişti.

4. **Timeout**: Eski servislerin varsayılan timeout'u genelde çok uzun oluyor, 1 dakika falan. Sen cephede daha kısa bir sınır koy, yoksa arka taraftaki bir yavaşlama gelip bütün istekleri kilitliyor.

5. **Cache**: Değişmeyen referans verileri (il listesi, ürün kategorileri) için cache koymak en kolay kazanç. Eski servise giden trafiğin yarısı bu tarz sorgular olabiliyor.

## Tuzaklar

- **WS-Security'nin birebir karşılığı yok.** OAuth2 + mTLS çoğu senaryoyu karşılıyor ama mesaj seviyesinde imzalama şartsa ek iş yapman lazım.
- **Dağıtık transaction kayboluyor.** WS-AtomicTransaction ile yapılan şeyleri REST tarafında saga veya outbox pattern ile çözmek gerekiyor. Bu ayrı bir makale konusu olmakla beraber, geçiş planında mutlaka hesaba katılmalı.
- **Eski istemciler bir süre daha SOAP konuşacak.** Hepsini aynı anda güncelleyemezsin, iki arayüzü paralel yaşatmayı baştan planla.
- **Sürüm politikası.** REST tarafında `/api/v1` ile başlayın. "Sonra eklerim" demeyin, sonra çok zor oluyor.
- **Log ve karşılaştırma.** Geçiş sırasında eski ve yeni cevapları loglayıp karşılaştırmak, sonradan çıkacak "bu alan neden değişti" sorularının tek cevabı oluyor.

---

Esasen bu işin sırrı sabır. Bir seferde her şeyi çevirmeye çalışan projelerin çoğu yarıda kalıyor, en az riskli operasyonu seçip oradan başlayanlar ise genelde bitiyor. Küçük bir operasyonla başla, cepheyi kur, canlıda bir hafta izle, sonra bir sonrakine geç.

Bir de şunu ekleyeyim: geçiş bitince eski servisi kapatmayı unutmayın. Kimse dokunmadığı için yıllarca ayakta kalan, artık neye yaradığını kimsenin bilmediği servisler var — o listeye bir yenisini eklemeyelim.

Faydalı olması dileğiyle.
