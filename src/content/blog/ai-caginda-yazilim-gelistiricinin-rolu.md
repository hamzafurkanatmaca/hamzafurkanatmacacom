---

title: "AI Çağında Yazılım Geliştiricinin Rolü"
description: "Kod üretmek ucuzladı, doğrulamak aynı kaldı: değişen iş akışı, büyüyen inceleme yükü ve yolda öğrendiklerim."
pubDate: "Sep 3 2026"
heroImage: "/images/gelistirici-rolu.svg"
-----------------------------------------

AI'ın yazılım geliştirme biçimimizi değiştirdiğini söylemek artık pek iddialı bir cümle değil. Birkaç yıl önce çoğumuz bu araçları meraktan kurcalıyorduk. Bugün ise gün içinde yazdığım kodun, yaptığım araştırmanın ve hatta bazı teknik kararların bir yerinde mutlaka AI var.

Ama bana kalırsa asıl değişiklik “AI artık kod yazabiliyor” değil.

Kod zaten yazılıyor. Hatta eskisine göre çok daha hızlı yazılıyor. Değişen şey, geliştiricinin zamanını nerede harcadığı.

Eskiden günün büyük bölümü kod üretmekle geçiyordu. Şimdi giderek daha fazla zamanımı üretilen kodun gerçekten doğru olup olmadığını anlamaya harcıyorum.

## Kod Yazmak Ucuzladı

Eskiden bir günde 200 satır kod yazdıysam o 200 satırın büyük bölümünü neden yazdığımı bilirdim. Çünkü satır satır ben uğraşmıştım. Bir tasarım kararını neden aldığımı, hangi noktada zorlandığımı, hangi edge case yüzünden o `if`'i koyduğumu hatırlıyordum.

Şimdi bir agent'a birkaç görev veriyorsun, bir bakıyorsun birkaç yüz satırlık değişiklik çıkmış. İlk başta bu inanılmaz verimli hissettiriyor. Gerçekten de üretim tarafında büyük bir hız farkı var.

Sonra PR'ı açıyorsun.

Ve o birkaç yüz satırı yine sen okuyorsun.

Burada biraz garip bir dengesizlik oluşuyor. Kod üretmenin maliyeti ciddi şekilde düştü ama kodu doğrulamanın maliyeti aynı oranda düşmedi. Kodun gerçekten ihtiyacı karşılayıp karşılamadığını anlamak, sistemin başka bir yerini bozup bozmadığına bakmak, edge case düşünmek ve alınan kararların doğru olup olmadığını kontrol etmek hâlâ zaman alıyor.

Yani darboğaz aslında yer değiştirdi.

Ben artık günün önemli bir bölümünde kod yazmıyorum. Kod okuyorum. Daha doğrusu önüme gelen kod için sürekli “burada ne ters gidebilir?” diye düşünüyorum.

Bir başka ilginç değişiklik de hatalı kodun artık eskisinden çok daha düzgün görünmesi. Eskiden aceleyle yazılmış kodun genelde bir kokusu olurdu. İsimlendirme tutarsızdır, bir yerde copy-paste izi görürsün, yorum yarım kalmıştır. Review sırasında gözün ister istemez oralara giderdi.

AI'ın yazdığı yanlış kod ise gayet şık olabiliyor. İsimler düzgün, format düzgün, yorumlar düzgün, method isimleri yerinde.

Kod yine de yanlış.

Bence AI ile çalışırken alışılması gereken en önemli şeylerden biri bu. Kodun profesyonel görünmesi artık doğruluğu hakkında çok az şey söylüyor.

## Değişmeyen Kısım

Bir şey ise hiç değişmedi: sorumluluk.

Canlıda sistem patladığında kimseye “ama o kısmı AI yazmıştı” diyemiyorsun. PR senin. Merge düğmesine sen bastın. Dolayısıyla kodu kimin ürettiğinden çok kimin onayladığı önemli.

Sistemin bütününü kafada tutma işi de hâlâ büyük ölçüde geliştiricide.

Model repodaki dosyaları okuyabiliyor. Kodda arama yapabiliyor. Düzgün bir ortam verirsen build alıp test de çalıştırabiliyor. Ama üç ay önce başka bir serviste neden garip bir karar aldığımızı bilmiyor. Geçen hafta iş biriminin söylediği istisnayı bilmiyor. Ekibin yıllardır bildiği ama hiçbir yerde yazmayan “o tabloya dokunmuyoruz çünkü eski rapor sistemi buradan besleniyor” bilgisini bilmiyor.

Bu bilgi bir yerde yazılı değilse modelin onu çıkarmasını beklemek pek gerçekçi değil.

Aslında bu sadece AI'ın problemi de değil. Ekibe yeni katılan bir geliştirici de aynı durumda.

AI burada biraz acı bir şeyi görünür hale getirdi: ekip içinde “herkes biliyor” dediğimiz birçok bilgi aslında sadece birkaç kişinin kafasında duruyor.

## Bağlamın Önemi

AI araçlarını daha fazla kullandıkça model seçiminden önce bağlama bakmaya başladım.

Kötü bir cevap geldiğinde eskiden “bu model beceremedi” diye düşünürdüm. Şimdi önce kendime şunu soruyorum: Ben buna neyi söylemedim?

Modelin o anda gördüğü açık dosyalar, verdiğin talimatlar, repo kuralları, testler, örnek kodlar ve mevcut mimari aslında cevabın büyük bölümünü belirliyor.

Mesela projede `IKiraciBaglami` diye bir yapı var ama sen sadece ilgili methodu kopyalayıp modele verirsen sana gayet rahat `TenantContext.Current` diye projede hiç var olmayan bir şey önerebilir.

Bunu hallucination olarak görebilirsin ama bir açıdan modelin yaptığı şey mantıklı. Elindeki boşluğu bildiği genel bir kalıpla dolduruyor.

Sorun, o boşluğu bizim bırakmış olmamız.

Bu yüzden bugün iyi AI kullanımı benim için doğru prompt yazmaktan çok doğru bağlam vermek anlamına geliyor.

## Otomatik Tamamlama ile Agent Aynı Şey Değil

AI destekli geliştirmeyi tek bir kullanım şekliymiş gibi konuşmak da bence yanıltıcı.

Autocomplete kullanırken kontrol hâlâ büyük ölçüde sende. Bir method yazıyorsun, birkaç satır öneriyor, kabul edip etmemeye sen karar veriyorsun. Değişiklik küçük parçalar halinde geliyor.

Agent tarafında ise durum farklı. Agent repoda gezebiliyor, onlarca dosyayı değiştirebiliyor, test çalıştırabiliyor ve bazen senden çok daha fazla kod üretebiliyor.

Doğal olarak review yükü de aynı değil.

Autocomplete kullanırken yanlış öneriyi birkaç saniyede fark edebilirsin. Agent yanlış varsayımla 15 dosyayı değiştirdiyse önce ne yaptığını anlaman gerekiyor.

Bu yüzden agent kullanırken artık görevi verirken sınırları da daha açık yazıyorum. “Şu klasörün dışına çıkma”, “public contract değiştirme”, “yeni dependency ekleme”, “önce mevcut testleri çalıştır” gibi kısıtlar gerçekten fark yaratıyor.

Bunları söylemediğim zaman bazen verdiğim problemi çözüyor ama yanında birkaç tane yeni problem bırakıyor.

## Vibe Coding Meselesi

Vibe coding kavramı biraz fazla kullanılıyor ama aslında arkasındaki problem gerçek.

Kodun nasıl çalıştığını tam anlamadan “çalıştı, tamamdır” diyerek devam etmek prototipte sorun olmayabilir. Bir haftaya çöpe atılacak bir demo yapıyorsan gayet makul.

Üretim kodunda ise durum farklı.

Bugün tam anlamadan merge ettiğin kodu yarın debug etmek zorunda kalan kişi büyük ihtimalle yine sen oluyorsun.

Bence buradaki asıl risk teknik borçtan bile biraz farklı. Teknik borçta genelde borcun nerede olduğunu biliyorsun. “Bu modülü kötü yazdık, sonra toparlarız” diyorsun.

AI ile beraber başka bir borç türü daha oluşmaya başladı: review borcu.

Yeterince anlamadan merge ettiğin kodun neresinde problem olduğunu bilmiyorsun. Belki hiçbir şey olmayacak. Belki iki ay sonra production'da çok garip bir edge case ile ortaya çıkacak.

Tehlikeli tarafı da bu zaten. Borcun olduğunu biliyorsun ama nerede olduğunu bilmiyorsun.

## Benim İş Akışım Nasıl Değişti?

![AI çağında geliştiricinin iş akışındaki rolü](/images/gelistirici-rolu.svg)

Benim için süreç kabaca problemi anlamak, gerekli bağlamı toplamak, modeli kullanarak bir çözüm üretmek, çıkan şeyi doğrulamak ve sisteme entegre etmek şeklinde ilerliyor.

Burada ilginç olan, AI'ın zincirin tamamını aynı oranda hızlandırmaması.

En çok kod üretme aşamasını hızlandırıyor.

Problemin gerçekten ne olduğunu anlamak hâlâ sende. Modelin hangi bilgiye ihtiyacı olduğunu belirlemek sende. Çıkan çözümün doğru olup olmadığını kontrol etmek sende. Mevcut sisteme gerçekten uyup uymadığına karar vermek yine sende.

Bu yüzden üretim tarafını on kat hızlandırıp review tarafını aynı bıraktığında bir noktadan sonra verim üretmiyorsun. Sadece daha büyük bir inceleme kuyruğu oluşturuyorsun.

Bunu özellikle agent kullanmaya başladıktan sonra daha net hissettim.

Kod yazma kapasitesi artık eskisi kadar sınırlayıcı değil. Dikkat kapasitesi sınırlayıcı olmaya başladı.

## AI'ı En Çok Nerelerde Kullanıyorum?

Otomatik tamamlama hâlâ en risksiz kullanım biçimlerinden biri. DTO mapping, test setup'ları, tekrar eden validation kodları veya çok tahmin edilebilir boilerplate işlerde ciddi zaman kazandırıyor.

Buradaki tehlike daha çok alışkanlık tarafında. Öneri geliyor, `Tab`. Bir tane daha geliyor, `Tab`. Bir süre sonra fark ediyorsun ki aslında bazı satırları gerçekten okumadan kabul etmişsin.

Küçük bir şey gibi görünüyor ama bu refleks zamanla “AI önerdiyse büyük ihtimalle doğrudur” rahatlığına dönüşebiliyor.

Sohbet ederek kullanmayı ise özellikle tasarım kararlarında seviyorum. Bir yaklaşımın artılarını eksilerini tartışmak, bir mimari kararın zayıf noktalarını aramak veya aklımdaki çözümü eleştirtmek bana çoğu zaman doğrudan kod üretmekten daha faydalı geliyor.

“Bunu event ile mi çözmek daha mantıklı, doğrudan servis çağrısı mı?”, “Burada idempotency nerede olmalı?”, “Bu tasarımın ileride problem çıkarabilecek tarafı ne?” gibi sorularda iyi bir tartışma aracı.

Agent tarafında ise asıl verim çok dosyalı mekanik işlerde ortaya çıkıyor. Bir interface değişti ve 14 farklı yerde kullanılıyorsa eskiden tek tek açıp düzeltmek zorundaydım. Şimdi agent bütün kullanımları bulup değiştirebiliyor, build alıp hataları görüp tekrar düzeltebiliyor.

Burada gerçekten ciddi zaman kazanılıyor.

Ama aynı anda review yükü de büyüyor.

Küçük sandığın bir değişiklik 12 dosyalık diff olarak geri geldiğinde, agent'ın hızlı olması senin o diff'i hızlı okuyabileceğin anlamına gelmiyor.

Uçtan uca otonom kullanımda ise daha temkinliyim. Dependency güncellemesi, küçük refactor, testi zaten olan ve iyi tanımlanmış bug gibi işler gayet uygun.

Belirsizlik arttıkça güvenim azalıyor.

Çünkü insan geliştirici bazen durup “burada tam olarak ne istiyorsunuz?” diye soruyor. Model ise çoğu zaman bir varsayım yapıp devam ediyor.

Daha kötüsü, yanlış varsayımla ürettiği çözümü de doğru çözüm kadar kendinden emin sunuyor.

## Modelin Bilemeyeceği Bir Şey

Basit bir örnek üzerinden anlatayım.

Modelin şu kodu ürettiğini düşünelim:

```csharp
public async Task<SiparisOzetDto> GetirAsync(
    int kullaniciId,
    CancellationToken ct)
{
    var anahtar = $"siparis-ozet-{kullaniciId}";

    if (_cache.TryGetValue<SiparisOzetDto>(
            anahtar,
            out var onbellekten))
    {
        return onbellekten!;
    }

    var ozet = await _repo.OzetGetirAsync(
        kullaniciId,
        ct);

    _cache.Set(
        anahtar,
        ozet,
        TimeSpan.FromMinutes(5));

    return ozet;
}
```

Tek başına baktığında gayet temiz.

İsimlendirme düzgün. Cache kullanımı düzgün. Async yapı doğru görünüyor.

Ama sistem çok kiracılıysa ve kullanıcı kimlikleri tenant bazında üretiliyorsa burada ciddi bir problem var.

Aynı `kullaniciId` iki farklı şirkette bulunabiliyorsa cache anahtarı yanlış.

Doğru hali şöyle olmalı:

```csharp
public async Task<SiparisOzetDto> GetirAsync(
    int kullaniciId,
    CancellationToken ct)
{
    // Kullanıcı kimlikleri kiracı bazında üretiliyor.
    // Kiracı anahtara katılmazsa iki farklı şirketin
    // aynı kimlikli kullanıcıları aynı cache kaydını paylaşabilir.

    var anahtar =
        $"siparis-ozet-{_kiraci.Id}-{kullaniciId}";

    if (_cache.TryGetValue<SiparisOzetDto>(
            anahtar,
            out var onbellekten))
    {
        return onbellekten!;
    }

    var ozet = await _repo.OzetGetirAsync(
        kullaniciId,
        ct);

    _cache.Set(
        anahtar,
        ozet,
        TimeSpan.FromMinutes(5));

    return ozet;
}
```

Buradaki düzeltme otuz saniyelik.

Asıl önemli şey kod değişikliği değil, yorum satırındaki bilgi.

Hata C# bilinmediği için oluşmuyor. Sistem bilinmediği için oluşuyor.

Bu bilgi kodda veya dokümantasyonda yoksa modeli suçlamak biraz kolaycılık oluyor. Aynı hatayı projeye yeni giren bir geliştirici de yapabilir.

Aslında AI burada yeni bir sorun yaratmadı. Zaten var olan örtük bilgiyi görünür hale getirdi.

## Önce Testi Yazmak

Son dönemde hoşuma giden çalışma şekillerinden biri, çözümü modele bırakmadan önce doğruluğun ne olduğunu mümkün olduğunca benim tanımlamam.

Örneğin yukarıdaki cache problemi için önce şu testi yazabilirim:

```csharp
[Fact]
public async Task
    Ayni_Kullanici_Kimligi_Farkli_Kiracilarda_Onbellegi_Karistirmaz()
{
    var cache =
        new MemoryCache(new MemoryCacheOptions());

    var aServisi = OzetServisiKur(
        cache,
        kiraciId: "sirket-a",
        beklenenTutar: 100m);

    var bServisi = OzetServisiKur(
        cache,
        kiraciId: "sirket-b",
        beklenenTutar: 250m);

    var a = await aServisi.GetirAsync(42, default);
    var b = await bServisi.GetirAsync(42, default);

    Assert.Equal(100m, a.ToplamTutar);
    Assert.Equal(250m, b.ToplamTutar);
}
```

Bu testin neden var olması gerektiğini modelin kendi kendine çıkarmasını beklemiyorum. Testin arkasındaki bilgi benim sistem bilgim.

Ama testi yazdıktan sonra “bu testi geçir” demek oldukça iyi çalışıyor.

Üstelik çıkan çözümü incelemek de kolaylaşıyor. Artık tamamen soyut bir şekilde “bu doğru mu?” diye bakmıyorum. Doğruluğun en azından bir bölümünü önceden tanımlamış oluyorum.

Bence AI ile geliştirmede giderek daha değerli hale gelen alışkanlıklardan biri bu.

Kod üretimini devredebilirsin.

Doğruluk tanımını devretmemek gerekiyor.

## Repo Kurallarını Yazıya Dökmek

AI araçlarını kullanmaya başladıktan sonra yaptığım en faydalı şeylerden biri, daha önce ekip içinde sözlü olarak dolaşan bazı kuralları repo i
