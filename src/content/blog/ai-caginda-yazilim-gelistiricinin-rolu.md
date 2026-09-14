---

title: "AI Çağında Yazılım Geliştiricinin Rolü"
description: "Kod üretmek ucuzladı, doğrulamak aynı kaldı: değişen iş akışı, büyüyen inceleme yükü ve yolda öğrendiklerim."
pubDate: "Sep 3 2026"
heroImage: "/images/gelistirici-rolu.svg"
-----------------------------------------

AI'ın yazılım geliştirme biçimimizi değiştirdiğini söylemek artık pek iddialı bir cümle değil. Birkaç yıl önce çoğumuz AI araçlarını daha çok meraktan kurcalıyorduk. Bugün ise gün içinde yazdığım kodun, yaptığım araştırmanın ve hatta bazı teknik kararların önemli bir bölümünde bir şekilde AI var.

Ama bence asıl değişiklik “AI kod yazıyor” kısmı değil.

Kod zaten yazılıyor. Hatta fazlasıyla yazılıyor.

Benim işimde değişen şey, kodu üretmekten çok **üretilen kodun doğru olup olmadığına karar vermek** oldu.

## Kod Yazmak Ucuzladı

Eskiden bir günde 200 satır kod yazdıysam, o 200 satırın büyük bölümünü neden yazdığımı bilirdim. Çünkü satır satır ben uğraşmıştım.

Şimdi öğleden sonra bir agent'a birkaç görev veriyorsun, bir bakıyorsun 800 satırlık değişiklik çıkmış.

İlk başta bu inanılmaz verimli hissettiriyor.

Sonra PR'ı açıyorsun.

Ve o 800 satırı yine sen okuyorsun.

İşte burada biraz garip bir durum oluştu. Kod üretmenin maliyeti ciddi şekilde düştü ama **kodu doğrulamanın maliyeti pek düşmedi**.

Kodun gerçekten ihtiyacı karşılayıp karşılamadığını anlamak, edge case düşünmek, sistemin başka bir yerini bozup bozmadığına bakmak hâlâ zaman alıyor.

Yani darboğaz yer değiştirdi.

Ben artık günün önemli bir kısmında kod yazmıyorum. Kod okuyorum.

Daha doğrusu, önüme gelen kod için sürekli şu soruyu soruyorum:

> Burada ne ters gidebilir?

Bir başka ilginç taraf da şu: yanlış kod artık eskisinden çok daha düzgün görünüyor.

Eskiden aceleyle yazılmış kodun genelde bir kokusu olurdu. İsimlendirme bozuktur, bir yerde copy-paste izi vardır, yorum yarım kalmıştır. Code review sırasında gözün otomatik olarak oraya giderdi.

AI'ın yazdığı hatalı kod ise gayet şık olabiliyor.

İsimler düzgün.

Format düzgün.

Yorumlar düzgün.

Kod yanlış.

Bence AI ile çalışırken alışılması gereken en tehlikeli şeylerden biri bu.

## Değişmeyen Kısım

Bir şey ise hiç değişmedi: sorumluluk.

Canlıda sistem patladığında kimseye “ama o kısmı AI yazmıştı” diyemiyorsun.

PR senin.

Merge düğmesine sen bastın.

Dolayısıyla kodu kimin ürettiğinden çok, kimin onayladığı önemli.

Bir de sistem bilgisinin tamamını modele aktarmak hâlâ kolay değil.

Model önündeki dosyaları okuyabiliyor. Repoda arama yapabiliyor. Hatta düzgün kurulmuşsa build alıp test bile çalıştırabiliyor.

Ama üç ay önce başka bir serviste neden garip bir karar aldığımızı bilmiyor.

Geçen hafta iş biriminin söylediği istisnayı bilmiyor.

Ya da ekipte herkesin bildiği ama hiçbir yerde yazmayan şu tarz bilgileri bilmiyor:

“Bu tabloya dokunmuyoruz çünkü eski raporlama sistemi buradan besleniyor.”

İşte o bilgi hâlâ geliştiricide.

Bence bugün deneyimli bir geliştiricinin değerli olduğu yerlerden biri tam olarak burası.

## Önce Birkaç Kavram

Yazının devamında kullanacağım birkaç kavram var. Çok akademik tanımlara girmeden ne kastettiğimi söyleyeyim.

**Bağlam**

Modelin o anda bildiği her şey.

Açık dosyalar, verdiğin prompt, repodaki kurallar, örnek kodlar, testler…

Bir süredir kötü çıktı aldığımda ilk suçladığım şey model değil. Önce “ben buna neyi söylemedim?” diye bakıyorum.

Çoğu zaman problem oradan çıkıyor.

**Otomatik tamamlama ve agent**

Bunları aynı şeymiş gibi konuşuyoruz ama bence aralarında ciddi fark var.

Autocomplete sen kod yazarken yardımcı oluyor. Direksiyon hâlâ tamamen sende.

Agent ise repoyu gezebiliyor, dosya değiştirebiliyor, komut çalıştırabiliyor ve bazen senden çok daha fazla kod üretebiliyor.

Doğal olarak review yükü de aynı değil.

**Vibe coding**

Kodun nasıl çalıştığını tam anlamadan “çalıştı, tamamdır” deyip devam etmek.

Bir haftaya çöpe atılacak prototipte hiçbir problem görmüyorum.

Üretim kodunda ise işler biraz değişiyor.

Çünkü bugün anlamadığın kodu yarın debug etmek zorunda kalan kişi çoğu zaman yine sen oluyorsun.

**İnceleme borcu**

Bu ifadeyi son dönemde teknik borçtan daha çok düşünüyorum.

Teknik borçta genelde borcun nerede olduğunu bilirsin.

“Şu modülü kötü yazdık, sonra düzeltelim.”

İnceleme borcunda ise problem farklı.

Yeterince anlamadan merge ettiğin kodun **neresinin problemli olduğunu bilmiyorsun**.

Belki hiçbir sorun çıkmayacak.

Belki iki ay sonra production'da patlayacak.

Asıl tehlike de bu.

**Eval**

Ürünün içinde LLM kullanıyorsan, çıktının ne kadar iyi olduğunu ölçmek için oluşturduğun test seti.

Tam olarak unit test değil tabii ama zihinsel olarak ben biraz o şekilde düşünüyorum.

Deterministik olmayan bir sistem kullanıyorsan “bence iyi cevap veriyor” noktasından bir yerde çıkman gerekiyor.

**Kurallar dosyası**

Repoya koyduğun ve modele projedeki kuralları anlattığın dosya.

Açıkçası AI araçlarıyla çalışırken yaptığım şeyler arasında efor/getiri oranı en yüksek olanlardan biri bu oldu.

Birazdan örnek vereceğim.

## Benim İçin İş Akışı Nasıl Değişti?

![AI çağında geliştiricinin iş akışındaki rolü](/images/gelistirici-rolu.svg)

Kabaca akış şu hale geldi:

Problemi anla → bağlamı hazırla → modele işi ver → çıkan şeyi doğrula → sisteme entegre et.

Burada ilginç olan şu: AI aslında bu zincirin tamamını hızlandırmıyor.

En çok “üret” kısmını hızlandırıyor.

Problemin doğru tanımlanması hâlâ sende.

Bağlamın hazırlanması sende.

Doğrulama sende.

Sisteme gerçekten uyup uymadığına karar vermek yine sende.

Dolayısıyla üretim hızını on kat artırıp review tarafını aynı bırakırsan bir noktadan sonra verim değil, sadece daha büyük bir kuyruk üretmiş oluyorsun.

Bunu yaşayarak fark ettim.

## AI'ı Nasıl Kullanıyorum?

### Otomatik tamamlama

En basit kullanım şekli bu.

DTO mapping, basit guard'lar, test setup'ları, tekrar eden kodlar…

Bu tip işlerde hâlâ çok kullanışlı.

Risk de görece düşük çünkü kod sen yazarken küçük parçalar halinde geliyor.

Ama burada bile bir alışkanlık oluşuyor.

Öneri geliyor.

Tab.

Bir tane daha geliyor.

Tab.

Bir süre sonra fark ediyorsun ki aslında bazı satırları okumadan kabul etmeye başlamışsın.

Küçük ama bence önemli bir tuzak.

### Sohbet ederek

Bir problemi anlatıp çözüm tartışmak hâlâ en sevdiğim kullanım biçimlerinden biri.

Özellikle mimari kararlar için iyi çalışıyor.

“Bunu event ile mi çözsem yoksa doğrudan çağrı mı yapsam?”

“Burada idempotency nasıl kurulmalı?”

“Bu tasarımın sıkıntılı tarafı ne?”

Koddan önce fikir tartışmak bazen doğrudan kod üretmekten daha faydalı oluyor.

Dezavantajı şu: bağlamı sen taşıyorsun.

Repo modelin önünde değilse bir noktadan sonra olmayan şeyler uydurmaya başlayabiliyor.

Mesela projede `IKiraciBaglami` var ama model sana gayet rahat şekilde:

```csharp
TenantContext.Current
```

diye bir şey yazabiliyor.

Sen de gidip düzeltiyorsun.

Üçüncü seferde insanın siniri bozuluyor.

### Agent

Verim farkını en net burada hissediyorum.

Bir interface değişti, 14 yerde kullanılıyor.

Eskiden tek tek açıp düzeltecektim.

Şimdi agent hepsini bulup değiştirebiliyor, build alabiliyor, hataları görüp tekrar düzeltebiliyor.

Bu gerçekten ciddi zaman kazandırıyor.

Ama başka bir problem getiriyor:

**değişiklik hacmi.**

Basit sandığın bir istek bir anda 12 dosyalık diff olarak geri gelebiliyor.

Bu yüzden agent'a görev verirken artık özellikle sınır koyuyorum.

“Şu klasörün dışına çıkma.”

“Public contract değiştirme.”

“Yeni dependency ekleme.”

“Önce mevcut testi çalıştır.”

Bunları söylemediğimde bazen çözmesi gereken problemi çözüyor ama yanında üç tane yeni problem bırakıyor.

### Uçtan uca otonom işler

Issue'yu verip PR beklemek kulağa çok güzel geliyor.

Bazı işlerde gerçekten çalışıyor.

Dependency güncellemesi.

Basit refactor.

Net tanımlanmış bug.

Testi olan küçük işler.

Ama belirsizlik arttıkça güvenim hızla düşüyor.

Çünkü insan geliştirici bazen durup:

“Burada tam olarak ne istiyorsunuz?”

diye soruyor.

Model ise çoğu zaman bir varsayım yapıp devam ediyor.

Daha kötüsü, yanlış varsayımla ürettiği çözümü de son derece kendinden emin sunuyor.

O yüzden otonomi arttıkça benim review seviyem de artıyor.

## Modelin Bilemeyeceği Bir Şey

Basit bir örnek vereyim.

Model şöyle bir kod üretsin:

```csharp
// Üretilen hali — tek başına bakınca kusursuz görünüyor

public async Task<SiparisOzetDto> GetirAsync(
    int kullaniciId,
    CancellationToken ct)
{
    var anahtar = $"siparis-ozet-{kullaniciId}";

    if (_cache.TryGetValue<SiparisOzetDto>(anahtar, out var onbellekten))
        return onbellekten!;

    var ozet = await _repo.OzetGetirAsync(kullaniciId, ct);

    _cache.Set(anahtar, ozet, TimeSpan.FromMinutes(5));

    return ozet;
}
```

Tek başına baktığımda ben de buna “gayet iyi” diyebilirim.

Ama sistem çok kiracılıysa ve kullanıcı kimlikleri tenant bazında üretiliyorsa burada ciddi bir açık var.

Doğru hali şöyle:

```csharp
public async Task<SiparisOzetDto> GetirAsync(
    int kullaniciId,
    CancellationToken ct)
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

Burada düzeltme 30 saniye.

Asıl önemli şey o yorum.

Çünkü hata C# bilgisi eksikliğinden kaynaklanmıyor.

Sistem bilgisinin eksikliğinden kaynaklanıyor.

O bilgiyi bir yere yazmazsak yarın başka bir geliştirici de aynı hatayı yapabilir. AI da yapabilir.

Aslında AI burada yeni bir problem yaratmadı.

Zaten var olan örtük bilgiyi görünür hale getirdi.

Bu ayrımı önemli buluyorum.

## Testi Ben Yazıp Kodu Ona Bıraktığım Oluyor

Son dönemde hoşuma giden bir çalışma şekli şu:

Önce doğruluğun ne olduğunu ben tarif ediyorum.

Örneğin:

```csharp
[Fact]
public async Task Ayni_Kullanici_Kimligi_Farkli_Kiracilarda_Onbellegi_Karistirmaz()
{
    var cache = new MemoryCache(new MemoryCacheOptions());

    var aServisi = OzetServisiKur(
        cache,
        kiraciId: "sirket-a",
        beklenenTutar: 100m);

    var bServisi = OzetServisiKur(
        cache,
        kiraciId: "sirket-b",
        beklenenTutar: 250m);

    // İki kiracıda da kullanıcı kimliği 42
    var a = await aServisi.GetirAsync(42, default);
    var b = await bServisi.GetirAsync(42, default);

    Assert.Equal(100m, a.ToplamTutar);
    Assert.Equal(250m, b.ToplamTutar);
}
```

Bu testin var olması gerektiğini modelin bilmesini beklemiyorum.

Çünkü testin arkasındaki bilgi benim sistem bilgim.

Ama testi yazdıktan sonra:

“Bu testi geçir.”

demek gayet iyi çalışıyor.

Üstelik çıkan kodu incelemek de daha kolay oluyor.

Çünkü artık “bu doğru mu?” diye tamamen soyut bir soru sormuyorum.

Doğruluğun bir kısmını önceden yazmış oluyorum.

## Kuralları Repoya Yazınca İlginç Bir Şey Oldu

Repo kökünde şöyle bir markdown dosyası tutuyorum:

```markdown
## Bu Repoda Geçerli Kurallar

- Sistem çok kiracılı. Önbellek anahtarı, dosya yolu veya arama sorgusu
  üreten her yerde `IKiraciBaglami.Id` mutlaka yer alır.

- Para alanları `decimal`. Hesaplamada `double` kullanılmaz.

- Dış servis çağrıları `IHttpClientFactory` üzerinden yapılır.
  Doğrudan `new HttpClient()` yasak.

- Yeni endpoint `/api/v1` altına eklenir, yetki politikası açıkça yazılır.

- Veritabanı değişikliği migration ile yapılır, elle SQL çalıştırılmaz.
```

Bunu ilk başta tamamen model için yazmıştım.

Sonra fark ettim ki ekipte yeni başlayan biri için de gayet faydalı bir dokümana dönüşmüş.

Biraz komik aslında.

Yıllardır yeni gelen insanlara aynı şeyleri sözlü olarak anlatıyoruz.

Ama yazmıyoruz.

AI gelince:

“Model bunu nereden bilecek?”

deyip yazmaya başladık.

Sonra insanlara da faydası oldu.

Muhtemelen baştan yazmamız gerekiyordu.

Ama yazmadık.

Çünkü ihtiyaç yeterince görünür değildi.

## Şu Ana Kadar En Çok İşime Yarayan Şeyler

### Küçük parça iste

Bir model 600 satır kod üretebiliyor diye 600 satır istemek zorunda değilsin.

Bunu birkaç kez yaptım ve sonrasında PR'ın yarısına geldiğimde kendimi sadece aşağı doğru kaydırırken buldum.

600 satırlık bir değişikliği hakkıyla incelemek zor.

150 satırlık dört değişiklik çok daha rahat.

AI'ın üretim kapasitesi arttıkça işi küçültmek bence daha da önemli hale geliyor.

### Doğruluğu mümkün olduğunca baştan tanımla

Test olabilir.

Acceptance criteria olabilir.

Örnek input-output olabilir.

Ama “doğru çözüm neye benziyor?” sorusuna mümkün olduğunca erken cevap vermek lazım.

Yoksa iş sonunda:

“Çalışıyor gibi.”

noktasına geliyor.

“Çalışıyor gibi” üretim sistemleri için pek iyi bir kriter değil.

### Anlamadığım kodu merge etmemeye çalışıyorum

Bunu yazmak kolay.

Uygulamak daha zor.

Özellikle akşam olmuşsa, release bekliyorsa ve önünde 400 satırlık gayet temiz görünen bir diff varsa insanın:

“Testler de geçiyor zaten.”

deyesi geliyor.

Ben bunu yaptığım birkaç durumda sonradan pişman oldum.

O yüzden kendime koyduğum en basit kural hâlâ bu:

Anlamıyorsam merge etmiyorum.

### “Neden?” diye sor

Modelin ürettiği çözüm mantıklı görünse bile bazen:

“Bunu neden böyle yaptın?”

diye soruyorum.

Şaşırtıcı derecede faydalı.

Bazen kod doğru görünüyor ama gerekçesini anlatmaya başladığında aslında yanlış bir varsayımdan yola çıktığı ortaya çıkıyor.

Daha güzeli, aynı soru bana da yarıyor.

Çünkü bazen modelin değil, benim başlangıç varsayımım yanlış çıkıyor.

### Kafandaki kuralları yaz

Bir bilgi sadece senin kafandaysa iki kişi bilmiyor:

AI ve ekibe yeni katılan geliştirici.

Bu yüzden mümkün olan şeyleri teste, dokümana, tipe, linter kuralına veya repo talimatına çevirmek ciddi fark yaratıyor.

### Agent'a test ve build çalıştırma imkânı ver

Agent sadece dosya değiştiriyorsa hatasının önemli bir kısmını sana bırakıyor.

Build ve test çalıştırabiliyorsa en azından basit hatalarını kendi görebiliyor.

Bu bence agent kullanımındaki en büyük kalite farklarından biri.

## En Çok Dikkat Ettiğim Tuzaklar

### Review borcu

Bir ekip eskiden haftada 20 birim iş üretiyorsa ve şimdi AI sayesinde 40 birim üretiyorsa güzel.

Ama review kapasitesi hâlâ 20 ise kalan 20 birim bir yere gidiyor.

Yok olmuyor.

Bir yerde hızlı geçiliyor.

Bir yerde yeterince okunmuyor.

Bir yerde “test geçiyor” denilip merge ediliyor.

Sonra borcu production'da ödüyorsun.

### Kendinden emin konuşması hiçbir şey ifade etmiyor

Bir insan ekip arkadaşın:

“Buradan çok emin değilim.”

dediğinde bu sana bilgi verir.

Oraya biraz daha dikkatli bakarsın.

LLM'de bu sinyal güvenilir değil.

Yanlış cevabı da son derece düzgün, net ve profesyonel bir tonla verebiliyor.

Buna alışmak gerekiyor.

### Debug kasını kaybetmek

Bunu kendimde fark ettiğim bir dönem oldu.

Bir hata geliyor.

İlk refleks:

Logu kopyala → modele at.

Sonra başka hata geliyor.

Yine aynı şey.

Kısa vadede çok hızlısın.

Ama bir süre sonra kendi başına hata kovalamak için kullandığın reflekslerin zayıflamaya başladığını hissediyorsun.

O yüzden artık bazı hatalarda bilerek hemen sormuyorum.

Önce kendim bakıyorum.

Biraz idman gibi.

### Junior konusu

“AI basit işleri yapıyor, artık junior'a ihtiyaç yok.”

cümlesini sık duyuyorum.

Ben buna pek katılmıyorum.

Bence junior'ın işi bitmedi.

Ama giriş seviyesi değişiyor.

Eskiden ilk öğrettiğimiz şey:

“Bunu nasıl yazarsın?”

oluyordu.

Şimdi giderek daha önemli hale gelen soru şu:

“Bunun doğru olduğunu nasıl anlarsın?”

Bu daha zor bir beceri.

Dolayısıyla junior ihtiyacı ortadan kalkmıyor ama yetiştirme şeklinin değişmesi gerekiyor.

### Uydurulmuş paketler

Bu hâlâ dikkat edilmesi gereken bir konu.

Model gayet gerçekçi görünen ama aslında var olmayan bir paket önerebiliyor.

Daha kötüsü, biri gidip o isimle kötü niyetli bir paket yayınlayabiliyor.

Yeni bir dependency eklerken paketin gerçekten ne olduğuna bakmak 30 saniye sürüyor.

Repo nerede?

Kim yayınlıyor?

İndirilme sayısı ne?

Bakmaya değer.

### Nereye ne gönderdiğin

Kod tabanı kapalıysa, müşteri bilgisi varsa veya secret'larla çalışıyorsan kullandığın AI servisinin neye eriştiği önemli.

Bu konuya [LLM API seçerken](/blog/llm-api-secerken-nelere-dikkat-edilmeli/) yazısında daha detaylı değinmiştim.

Kod için de aynı mantık geçerli.

“Hızlı olsun” diye ne gönderdiğimizi unutmak kolay.

### Yanlış şeyi ölçmek

“AI ile artık iki kat hızlı kod yazıyoruz.”

Güzel.

Ama bence daha anlamlı soru şu:

**İki kat daha fazla çalışan yazılım mı çıkarıyoruz?**

Çünkü üretilen kod miktarı ile üretilen değer aynı şey değil.

Bir dönem bizde de üretim hızı bariz artmıştı ama geri dönen iş sayısı da artmıştı.

O zaman aslında hızlanmadığımızı fark ediyorsun.

Sadece daha hızlı kod üretmiş oluyorsun.

---

Sonuçta kendi adıma yazılım geliştiricinin rolünün küçüldüğünü düşünmüyorum.

Ama yeri değişiyor.

Klavyede geçirdiğim süre eskisine göre daha az.

Problemi anlamaya, doğru bağlamı toplamaya, sistemi düşünmeye ve çıkan şeyi doğrulamaya ayırdığım süre ise daha fazla.

Açıkçası ben işin bu tarafını daha çok seviyorum.

Ama daha yorucu olduğunu da kabul etmek lazım.

Sekiz saat kod yazmanın yorgunluğu başka.

Sekiz saat boyunca sürekli karar vermenin yorgunluğu başka.

Bir de artık şu noktadayım:

Bu araçları hiç kullanmamak pek gerçekçi gelmiyor.

Ama kullanmakla güvenmek aynı şey değil.

AI'ın yazdığı kodu kullanabilirim.

Kararımı AI'a bırakamam.

Sanırım geliştiricinin rolünün değiştiği yer de tam olarak burası.
