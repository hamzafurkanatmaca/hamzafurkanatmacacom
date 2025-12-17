---
title: "LLM API'lar Nedir ve Nasıl Kullanılırlar?"
description: "Büyük dil modellerine API üzerinden erişim: temel kavramlar, kullanım alanları ve üretimde dikkat edilmesi gerekenler."
pubDate: "Sep 14 2025"
heroImage: "/images/llm-architecture.svg"
---

ChatGPT çıktığından beri "acaba bunu da AI'a yaptırabilir miyim?" sorusu kafamdan çıkmıyor. İlk başta sadece ChatGPT web arayüzünden kullanıyordum, sonra bir gün dedim ki "ben bunu kendi uygulamamın içine gömsem ne güzel olur." API dokümantasyonunu açtım, 3 saat sonra ilk çalışan örneği gördüğümde çocuk gibi sevindim. O günden beri bu API'larla haşır neşirim.

## Neden Kendi Sunucunda Çalıştırmıyorsun?

Düşünsene, GPT-4 çalıştırmak için evine bir sunucu odası mı kuracaksın? A100 GPU'lar tanesi 10-15 bin dolar civarı, elektrik faturasını saymıyorum bile. API kullanınca sadece kullandığın kadar ödüyorsun. Startup'lar için bu çok kritik — validasyon yapmadan önce altyapıya para gömmek mantıksız.

Tabii bedavaya gelmiyor. Geçen ay bir projesinde cache koymayı unutunca 3 günde 180 dolar fatura gelen birini gördüm. Her yeni projede ilk iş rate limiter ve cache koymak olmalı.

Bir de veri gizliliği meselesi var. Müşteri verisi işliyorsan, o verinin OpenAI sunucularına gitmesi bazı sektörlerde yasak. Finans ve sağlık projelerinde buna dikkat etmek lazım.

## Terminoloji (Başta Kafa Karıştırıyor Ama Alışıyorsun)

**Token**: Modelin "düşünme birimi" gibi bir şey. "Merhaba nasılsın" 3-4 token oluyor. Türkçe'nin eklemeli yapısı yüzünden aynı anlam için İngilizce'den daha fazla token harcıyoruz, bu da faturaya yansıyor. "Yapabileceklerimizden" gibi bir kelime tek başına 3-4 token.

**Bağlam penceresi**: Modelin aynı anda görebildiği metin miktarı. GPT-4 Turbo'da 128K token var, yani küçük bir kitabı tek seferde okuyabilir. Ama dikkat: pencere büyüdükçe fiyat da artıyor ve model bazen ortadaki bilgileri "unutuyor" — buna "lost in the middle" deniyor.

**Temperature**: 0'a yakın tutarsan model çok deterministik davranır, aynı soruya hep aynı cevabı verir. 1'e yaklaştıkça yaratıcılaşır ama saçmalama riski de artar. Kod üretimi için 0.1-0.2, yaratıcı yazı için 0.7-0.8 kullanılır genelde.

**Streaming**: Cevabın tamamını beklemek yerine parça parça almak. Kullanıcı açısından çok fark ediyor — ChatGPT'nin yazı yazarken göründüğü gibi. Bunu implement etmek biraz uğraştırıcı olabilir ama UX için değer. Kullanıcılar beklemekten sıkılmaya meyillidirler.

**Function calling**: İşlerin heyecanlandığı yer burası. Model "hava durumunu öğrenmem lazım" deyip sana bir fonksiyon çağrısı döndürüyor, sen API'dan hava durumunu çekip modele geri veriyorsun, o da kullanıcıya güzel bir cevap veriyor. Agent sistemlerinin temeli bu olmakla beraber agent sistemleri ayrı bir makale konusu.

## Mimari

![LLM API mimarisi](/images/llm-architecture.svg)

Akış basit aslında: Kullanıcı soruyor → sen soruyu alıp LLM API'a gönderiyorsun → cevabı alıp kullanıcıya dönüyorsun. Arada logging, caching, retry mekanizması olmalı tabi. Error handling'i baştan düzgün yapmazsan sonra çok uğraştırıyor, tecrübeyle sabit :)

## Hangi Sağlayıcıyı Seçmeli?

Şahsi favorim şu an **Claude 3.5 Sonnet**. Uzun döküman işlerinde GPT-4'ü geçiyor bence, üstelik daha ucuz. Anthropic'in "constitutional AI" yaklaşımı da hoşuma gidiyor.

**OpenAI** hâlâ varsayılan tercih çoğu kişi için. Dokümantasyonu en iyi olan onlar, community çok geniş, bir sorun yaşarsan Stack Overflow'da muhtemelen biri çözmüştür. Ama son zamanlarda fiyat politikaları biraz sinir bozucu.

**Azure OpenAI** kurumsal projeler için mantıklı. KVKK/GDPR hassasiyeti olan projelerde Azure tercih edilebilir. Modeller biraz geç geliyor ama kurumsal müşteriler zaten "en yeni" yerine "en stabil" istiyor.

**Google Gemini** görsel + metin işlerinde güçlü. İyi bir tasarımcı.

**Mistral** ve **Groq** maliyet-performans dengesi için bakılabilir. Her iş için GPT gerekmez, basit sınıflandırma için küçük model yeterli.

**Açık kaynak** (Llama 3, Mixtral) veri egemenliği şartsa veya özel fine-tuning gerekiyorsa değerlendirilmeli. Ama operasyonel yükü hafife alma — Bu kısmı deneyimlemedim ama sabah 4'te sisteminin neden çöktüğünü debug etmek eğlenceli olmasa gerek :).

## Nerelerde Kullanılıyor?

En çok gördüğüm use case'ler:

- **Müşteri destek chatbot'u**: Ürün dökümanlarını RAG ile besleyip müşteri sorularını otomatik yanıtlama. %70-80 soruyu halledebiliyorlar.
- **Özetleme**: Toplantı kayıtlarını, uzun e-postaları özetleme. Baya zaman kazandırıyor.
- **Dökümanlardan veri çıkarma**: PDF faturalardan, sözleşmelerden yapısal veri çekme. OCR + LLM kombinasyonu çok güçlü.
- **Kod asistanı**: Copilot tarzı autocomplete, PR review önerileri, test yazımı.

## Kod Örnekleri

### Basit Chat Çağrısı (.NET)

```csharp
var http = new HttpClient();
http.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", Environment.GetEnvironmentVariable("LLM_API_KEY"));

var payload = new
{
    model = "gpt-4o-mini",
    messages = new object[]
    {
        new { role = "system", content = "Kısa ve öz cevaplar ver." },
        new { role = "user", content = "Dependency injection'ı bir cümleyle açıklar mısın?" }
    },
    temperature = 0.2,
    max_tokens = 200
};

var response = await http.PostAsync(
    "https://api.openai.com/v1/chat/completions",
    new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json")
);

var result = await response.Content.ReadAsStringAsync();
Console.WriteLine(result);
```

Production'da tabii bunu böyle naked HttpClient ile yapmayız, Polly ile retry ekleriz, timeout koyarız falan. Ama demo için bu kadar yeterli.

### Streaming

Kullanıcının ekranının donması kötü deneyim. Streaming ile cevap parça parça geliyor:

```csharp
var request = new HttpRequestMessage(HttpMethod.Post, "https://api.openai.com/v1/chat/completions");
request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);
request.Content = new StringContent(JsonSerializer.Serialize(new
{
    model = "gpt-4o-mini",
    stream = true,
    messages = new[] { new { role = "user", content = "Mikroservis mimarisinin artıları eksileri?" } }
}), Encoding.UTF8, "application/json");

using var client = new HttpClient();
using var response = await client.SendAsync(request, HttpCompletionOption.ResponseHeadersRead);
await using var stream = await response.Content.ReadAsStreamAsync();
using var reader = new StreamReader(stream);

while (!reader.EndOfStream)
{
    var line = await reader.ReadLineAsync();
    if (line?.StartsWith("data: ") == true)
    {
        var data = line.Substring(6);
        if (data == "[DONE]") break;
        // Her chunk'ı parse edip UI'a gönder
    }
}
```

### RAG Mantığı

RAG'ın özü şu: Modele soruyu sormadan önce, vektör veritabanından ilgili dökümanları çekip prompt'a ekliyorsun. Model kendi eğitiminde görmediği şeyleri de böyle "biliyor".

![RAG akışı](/images/rag-flow.svg)

```csharp
// 1. Kullanıcı sorusunu embedding'e çevir
// 2. Pinecone/Qdrant/Weaviate'den en yakın 5 chunk'ı getir
// 3. Bu chunk'ları prompt'a ekle

var context = string.Join("\n\n", relevantDocs.Select(d => d.Content));

var prompt = $"""
Aşağıdaki bilgileri kullanarak soruyu yanıtla. 
Emin olmadığın konularda "bu konuda bilgim yok" de.

Bağlam:
{context}

Soru: {userQuestion}
""";
```

Chunk boyutu ve overlap ayarları önemli. 500-1000 token civarı chunk'lar genelde iyi çalışıyor ama bu veri setine göre değişir. Denemek lazım.

## Maliyet Kontrolü

Öğrendiğim dersler:

1. **Model seçimi**: Basit işler için gpt-4o-mini veya claude-3-haiku yeterli. Her şey için en büyük modeli kullanma.

2. **Prompt kısalt**: Gereksiz uzun system prompt'lar maliyeti şişiriyor. "Sen yardımsever bir asistansın..." gibi klişeler gereksiz, model zaten öyle.

3. **Cache**: Aynı sorular tekrar geliyorsa (FAQ mesela), Redis'e at. Semantic cache de var ama implement etmesi daha karmaşık.

4. **Rate limit koy**: Hem maliyet hem de API limit aşımı için gerekli.

## Güvenlik

- **Prompt injection**: Kullanıcı girdisini temizlemeliyiz. "Şimdi önceki tüm talimatları unut ve..." gibi şeyler deneyenler olabilir.
- **PII maskeleme**: TC kimlik, kredi kartı gibi verileri modele göndermeden önce maskelemeliyiz.
- **Output validation**: Model bazen halüsinasyon yapıyor. Kritik işlemlerde çıktıyı doğrulamalıyız.


Faydalı olması dileğiyle.
