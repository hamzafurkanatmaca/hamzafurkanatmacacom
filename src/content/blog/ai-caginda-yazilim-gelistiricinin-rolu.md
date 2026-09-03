---
title: "AI Çağında Yazılım Geliştiricinin Rolü"
description: "Kod üretmek ucuzladı, doğrulamak aynı kaldı: değişen iş akışı, büyüyen inceleme yükü ve yolda öğrendiklerim."
pubDate: "Sep 3 2026"
heroImage: "/images/gelistirici-rolu.svg"
---

Geçen ay ekipten bir arkadaş 600 satırlık bir PR açtı. Açıklaması tertemizdi, testleri yeşildi, isimlendirme projenin geri kalanıyla tutarlıydı. Benim iki günde yazacağım işi bir öğleden sonra bitirmişti. İnceledim, birkaç ufak yorum bıraktım, birleştirdik.

İki hafta sonra bir müşteri, başka bir müşterinin sipariş özetini gördü.

Sebep tek satırdı. Önbellek anahtarı `siparis-ozet-{kullaniciId}` şeklinde kurulmuştu. Sistem çok kiracılı ve kullanıcı kimlikleri kiracı bazında üretiliyor — yani iki farklı şirkette aynı kimlik pekâlâ olabiliyor. Modelin bunu bilmesi mümkün değildi, çünkü o bilgi kodun hiçbir yerinde yazmıyordu; ekibin kafasındaydı. Biz de incelemede kaçırdık, çünkü o satır tek başına bakınca son derece makul görünüyordu.

O olaydan sonra bu işteki rolümün ne olduğu konusunda kafam netleşti.

## Ne Değişti?

Kod üretmek ucuzladı. Bu kadar net. Eskiden bir günde 200 satır yazardım ve o 200 satırın her birini niye yazdığımı bilirdim. Şimdi bir öğleden sonra 800 satır çıkıyor.

Ama şöyle bir detay var: üretmenin maliyeti düştü, **doğrulamanın maliyeti aynı kaldı**. Kodu okumak, anlamak, "burada ne ters gidebilir" diye düşünmek hâlâ eskisi kadar sürüyor. Aradaki oran bozulunca darboğaz yer değiştirdi. Artık gün içinde en çok vakit alan şey klavyeyle kod yazmak değil, önüme gelen kodun doğru olup olmadığına karar vermek.

İkinci değişiklik daha sinsi: yanlış kod artık **daha güzel** görünüyor. Eskiden aceleyle yazılmış kodun bir kokusu olurdu — tutarsız isimler, yarım bırakılmış yorumlar, kopyala-yapıştır izleri. İncelerken o kokuyu alır, oraya daha dikkatli bakardın. Şimdi hatalı kod da düzgün biçimlendirilmiş, güzel isimlendirilmiş ve özenli yorumlanmış halde geliyor. Yukarıdaki önbellek hatası tam olarak bu yüzden gözden kaçtı.

## Ne Değişmedi?

Sorumluluk. Canlıda bir şey patladığında "onu AI yazmıştı" diye bir savunma yok, olmayacak da. PR'ın altında senin adın yazıyor, birleştirme düğmesine sen bastın.

Bir de sistemin bütününü kafanda tutma işi değişmedi. Model önündeki dosyaları görüyor; senin üç ay önce başka bir serviste aldığın kararı, iş biriminin geçen hafta söylediği şeyi veya "o tabloya dokunmuyoruz çünkü rapor ekibi ondan besleniyor" bilgisini görmüyor. Bu bilgiyi taşıyan hâlâ sensin.

## Terminoloji

**Bağlam**: Modelin o an gördüğü her şey — açık dosyalar, verdiğin talimat, repodaki kurallar. Çıktı kalitesinin büyük kısmı burada belirleniyor. Kötü çıktı aldığımda ilk baktığım yer model seçimi değil, bağlamda ne eksik olduğu.

**Otomatik tamamlama ve agent farkı**: Otomatik tamamlama yazdığın satırı bitiriyor, kontrol tamamen sende. Agent ise dosyaları kendi okuyup kendi değiştiriyor, komut çalıştırabiliyor. İkisi aynı şey değil ve inceleme yükleri de aynı değil.

**Vibe coding**: Çıkan kodu tam anlamadan, "çalışıyor gibi" diye kabul etmek. Prototipte, atılacak bir demoda gayet serbest. Üretim koduna girdiği anda borca dönüşüyor.

**İnceleme borcu**: Düzgün okunmadan birleştirilmiş kod. Teknik borçtan farkı şu: teknik borcun nerede olduğunu bilirsin, "şu modülü bir gün düzelteceğiz" dersin. İnceleme borcunun nerede olduğunu bilmezsin. Kendini canlıda gösterene kadar öğrenemezsin.

**Değerlendirme (eval)**: Ürününde LLM kullanıyorsan çıktının kalitesini ölçen test kümesi. Birim testin bu alandaki karşılığı gibi düşün — deterministik olmayan bir şeyi ölçmenin tek makul yolu.

**Kurallar dosyası**: Repoya konan, projenin konvansiyonlarını modele anlatan dosya. Efor/getiri oranı en yüksek şey bu, aşağıda örneğini vereceğim.

## İş Akışı

![AI çağında geliştiricinin iş akışındaki rolü](/images/gelistirici-rolu.svg)

Akış şu: Problemi tanımlıyorsun, bağlamı kuruyorsun, model taslağı üretiyor, sen doğruluyorsun, sonra entegre ediyorsun.

Şemadaki asıl mesaj ortadaki kutunun darlığı. Model beş adımdan yalnızca birini hızlandırıyor. Diğer dördü olduğu yerde duruyor ve dördü de sende. Üretim adımını on kat hızlandırıp doğrulama adımına aynı özeni göstermeyince ne olduğunu yukarıda anlattım.

## AI'ı Kullanma Biçimleri

**Otomatik tamamlama**: Editörde satır veya blok tamamlama. Girmesi en kolay, riski en düşük yöntem. Sınır kodu, DTO eşlemeleri, tekrar eden test kurulumları gibi işlerde ciddi hız kazandırıyor.

**Eksiler:** Bağlamı dar, birkaç dosyaya yayılan işlerde yetersiz kalıyor. Bir de refleks meselesi var — öneri gelince Tab'a basmak alışkanlık haline geliyor ve okumadan kabul ettiğin satır sayısı sandığından fazla oluyor.

**Sohbet ederek**: Problemi anlatıp çıkan kodu projeye taşımak. Tasarım tartışmak, bir yaklaşımın artılarını eksilerini konuşmak için iyi çalışıyor.

**Eksiler:** Bağlamı elle sen taşıyorsun, uzun sürünce yoruyor. Model repoyu görmediği için var olmayan yardımcı metotlar uydurabiliyor — sende `IKiraciBaglami` varken `TenantContext` diye bir şey çağırıyor, sen de her seferinde elle düzeltiyorsun.

**Agent**: Repoda çalışan, dosyaları okuyup değiştiren, testi kendi çalıştıran kip. Verim farkının en çok hissedildiği yer burası. Çok dosyaya dokunan mekanik işlerde — bir arayüzü değiştirip tüm çağıranları güncellemek gibi — gerçekten iyi.

**Eksiler:** Değişiklik hacmi büyüyor, inceleme yükü onunla birlikte büyüyor. Küçük bir istek 12 dosyaya dokunmuş olarak geri gelebiliyor. Sınır koymazsan yoldan çıkıyor: kapsamı dar tut, build ve testi elinin altına ver, "şu klasörün dışına çıkma" gibi kısıtları baştan söyle.

**Uçtan uca otonom**: Issue'yu verip PR bekleme. İyi tanımlanmış, izole, testi zaten var olan işlerde çalışıyor — bağımlılık güncellemesi, tekrarlayan dönüşümler, ufak hata düzeltmeleri.

**Eksiler:** Belirsiz tanımlanmış işlerde soru sormak yerine varsayımda bulunuyor ve yanlış çözümü de doğru çözümle aynı özgüvenle sunuyor. En tehlikeli mod bu, çünkü hatanın maliyeti en geç fark edildiği yerde ortaya çıkıyor.

## Kod Örnekleri

### Modelin Bilemeyeceği Şey

Girişte anlattığım hatanın kendisi:

```csharp
// Üretilen hali — tek başına bakınca kusursuz görünüyor
public async Task<SiparisOzetDto> GetirAsync(int kullaniciId, CancellationToken ct)
{
    var anahtar = $"siparis-ozet-{kullaniciId}";

    if (_cache.TryGetValue<SiparisOzetDto>(anahtar, out var onbellekten))
        return onbellekten!;

    var ozet = await _repo.OzetGetirAsync(kullaniciId, ct);
    _cache.Set(anahtar, ozet, TimeSpan.FromMinutes(5));
    return ozet;
}
```

Düzeltilmiş hali:

```csharp
public async Task<SiparisOzetDto> GetirAsync(int kullaniciId, CancellationToken ct)
{
    // Kullanıcı kimlikleri kiracı bazında üretiliyor; kiracı katılmazsa
    // iki farklı şirketin aynı kimlikli kullanıcıları aynı kaydı paylaşır.
    var anahtar = $"siparis-ozet-{_kiraci.Id}-{kullaniciId}";

    if (_cache.TryGetValue<SiparisOzetDto>(anahtar, out var onbellekten))
        return onbellekten!;

    var ozet = await _repo.OzetGetirAsync(kullaniciId, ct);
    _cache.Set(anahtar, ozet, TimeSpan.FromMinutes(5));
    return ozet;
}
```

Düzeltmenin kendisi 30 saniyelik. Asıl kıymetli olan o yorum satırı. O bilgi kodda yazılı olmadığı sürece aynı hatayı başka bir dosyada tekrar yapacaktık — AI olsun ya da olmasın. Ekibe yeni katılan biri de aynı tuzağa düşerdi zaten.

### Testi Sen Yaz, Geçirmeyi Ona Bırak

Bu sırayı tersine çevirmek işe yarıyor:

```csharp
[Fact]
public async Task Ayni_Kullanici_Kimligi_Farkli_Kiracilarda_Onbellegi_Karistirmaz()
{
    var cache = new MemoryCache(new MemoryCacheOptions());

    var aServisi = OzetServisiKur(cache, kiraciId: "sirket-a", beklenenTutar: 100m);
    var bServisi = OzetServisiKur(cache, kiraciId: "sirket-b", beklenenTutar: 250m);

    // İki kiracıda da kullanıcı kimliği 42
    var a = await aServisi.GetirAsync(42, default);
    var b = await bServisi.GetirAsync(42, default);

    Assert.Equal(100m, a.ToplamTutar);
    Assert.Equal(250m, b.ToplamTutar);
}
```

Bu testi modele yazdırmazsın, çünkü testin var olma sebebi senin sistem bilgin. Ama test bir kez ortadayken "şunu geçir" demek gayet iyi çalışıyor — ve çıkan kodu doğrulaman da kolaylaşıyor, çünkü artık doğruluğun tanımı yazılı.

### Kuralları Repoya Yazmak

Kök dizine konan basit bir markdown dosyası. Bende şuna benzer bir şey duruyor:

```markdown
## Bu Repoda Geçerli Kurallar

- Sistem çok kiracılı. Önbellek anahtarı, dosya yolu veya arama sorgusu
  üreten her yerde `IKiraciBaglami.Id` mutlaka yer alır.
- Para alanları `decimal`. Hesaplamada `double` kullanılmaz.
- Dış servis çağrıları `IHttpClientFactory` üzerinden yapılır,
  doğrudan `new HttpClient()` yasak.
- Yeni endpoint `/api/v1` altına eklenir, yetki politikası açıkça yazılır.
- Veritabanı değişikliği migration ile yapılır, elle SQL çalıştırılmaz.
```

Bu dosyanın komik tarafı şu: aynı şeyleri ekibe yeni katılan insanlara da sözlü olarak anlatıyorduk, yıllarca hiç yazmamıştık. Model için yazdık, en çok yeni gelen arkadaşlar faydalandı. Bir ara "acaba bunu baştan yapsaydık" diye düşündüm, cevabı biliyorum: yapmazdık, çünkü ihtiyaç görünür değildi.

## Pratikte İşe Yarayanlar

1. **Küçük parça iste.** 600 satırlık bir PR'ı kimse hakkıyla incelemiyor, ben de incelemedim. 150 satırlık dört PR, 600 satırlık bir PR'dan çok daha güvenli. Model büyük parça üretebiliyor diye büyük parça istemek zorunda değilsin.

2. **Doğruluğun tanımını önce sen koy.** Test, kabul kriteri, örnek girdi-çıktı — hangisi uygunsa. Bu adımı atlarsan "çalışıyor mu" sorusunun cevabı hisse kalıyor.

3. **Anlamadığın kodu birleştirme.** Kuralların en basiti, uygulaması en zor olanı. Özellikle akşam saatlerinde ve iş yetiştirme telaşındayken zorlaşıyor. Yine de bu kuralı bozduğum her seferde pişman oldum.

4. **"Neden böyle yaptın" diye sor.** Çıkan koda gerekçesini sorunca gerekçenin çürük olduğu epey durum çıkıyor. Üstelik bu soru sana da yarıyor, kendi varsayımını da test etmiş oluyorsun.

5. **Kuralları koda ve repoya yaz.** Kafandaki bilgi yazılı olmadıkça ne model biliyor ne yeni gelen arkadaş. İkisi de aynı problem.

6. **Doğrulama araçlarını eline ver.** Agent kullanıyorsan build, test ve linter komutlarını çalıştırabilmesi büyük fark yaratıyor. Kendi hatasını kendi görüp düzeltiyor, sana daha temiz bir şey geliyor.

## Tuzaklar

- **İnceleme borcu birikiyor.** Ekip olarak üretim hızını iki katına çıkarıp inceleme kapasitesini aynı bıraktıysanız, fark bir yerde birikiyor. Genelde canlıda ödeniyor.

- **Emin dil bir sinyal değil artık.** İnsan meslektaşında tereddüt bir bilgidir; "bundan pek emin değilim" cümlesi sana nereye bakacağını söyler. Modelde o sinyal yok, yanlış cevabı da doğru cevapla aynı tonda veriyor. Buna göre ayar yapmak lazım.

- **Öğrenme kasını körletmek.** Hata ayıklama becerisi ancak hata ayıklayarak gelişiyor. Her takıldığında hemen sormak kısa vadede hızlandırıyor, uzun vadede o kası zayıflatıyor. Bunu kendimde fark ettim; artık bazı hataları bilerek kendim kovalıyorum, bir tür idman gibi.

- **Junior meselesi.** "Basit işleri AI yapıyor, junior'a iş kalmadı" cümlesini çok duyuyorum. Bence yanlış okuma. İş bitmedi, giriş bandı yükseldi — kalan iş daha zor. Çözüm junior almayı kesmek değil, işe alım sonrası ilk öğretilen şeyi değiştirmek. Eskiden "şunu yazmayı öğren" derdik; artık "şunun doğru olup olmadığını nasıl anlarsın" diye başlamak gerekiyor.

- **Uydurulmuş paketler.** Var olmayan paket adları önerilebiliyor ve birileri o isimlerle gerçekten zararlı paket yayınlıyor. Yeni bir bağımlılık eklerken paketin gerçekten var olduğuna, indirilme sayısına ve deposuna bakmak yarım dakika sürüyor.

- **Nereye ne gönderdiğin.** Kapalı kod tabanını, müşteri verisini veya sırları hangi servise gönderdiğine dikkat. Bu konuya [LLM API seçerken](/blog/llm-api-secerken-nelere-dikkat-edilmeli/) yazısında girmiştim, orada anlattığım maddeler kendi kodun için de aynen geçerli.

- **Yanlış metrik.** "Ne kadar hızlı yazdık" ölçüsü aldatıcı. Daha dürüst bir ölçü: canlıya çıkan ve geri dönmeyen iş miktarı. Birinciyi iyileştirip ikinciyi bozmak mümkün, ki bizde bir dönem tam olarak bu oldu.

---

Esasen rolüm daralmadı, kaydı. Klavyede geçen süre azaldı; problemi doğru tanımlamakta, bağlamı kurmakta ve çıkanı doğrulamakta geçen süre arttı. Açıkçası işin bu tarafı bana daha keyifli geliyor, ama daha yorucu olduğunu da söylemem lazım — gün boyu karar vermek, gün boyu kod yazmaktan farklı bir yorgunluk.

Bir de şu var: bu araçları kullanmamak diye bir seçenek kaldığını düşünmüyorum. Ama "kullanmak" ile "körlemesine güvenmek" arasındaki farkı korumak tamamen bizim elimizde. Aradaki mesafe de zaten bu mesleğin kendisi.

Faydalı olması dileğiyle.
