---
title: "LLM API'lar Nedir ve Nasıl Kullanılırlar?"
description: "Büyük dil modellerine API üzerinden erişim: temel kavramlar, kullanım alanları ve üretimde dikkat edilmesi gerekenler."
pubDate: "Sep 14 2025"
heroImage: "/images/llm-architecture.svg"
---

Son birkaç yılda yazılım geliştirme süreçlerimiz ciddi bir dönüşüm geçirdi. ChatGPT'nin çıkışıyla birlikte "acaba bunu da AI'a yaptırabilir miyim?" sorusu günlük rutinimizin parçası oldu. Ben de backend tarafında çalışırken bu modelleri projelerime nasıl entegre edebileceğimi araştırmaya başladım. Bu yazıda öğrendiklerimi ve deneyimlerimi paylaşmak istiyorum.

## Neden API Kullanıyoruz?

Aslında mantık basit: Bu modelleri çalıştırmak için ciddi donanım gerekiyor. GPU'lar pahalı, bakımı zahmetli ve her proje için ayrı bir altyapı kurmak pek mantıklı değil. API kullanarak sadece kullandığın kadar ödüyorsun, güncel modellere anında erişiyorsun ve ölçeklendirme derdini sağlayıcıya bırakıyorsun.

Tabii her şeyin bir bedeli var. Veri gizliliği konusunda dikkatli olmak gerekiyor, özellikle müşteri verisi işliyorsan. Token başına ücretlendirme de kontrolsüz bırakılırsa fatura sürprizleri yaşatabilir — bunu tecrübeyle öğrendim.

## Bilmeniz Gereken Terimler

İlk başta bu kavramlar kafa karıştırıcı gelebilir ama zamanla oturuyor:

**Token** denen şey aslında modelin "düşünme birimi". Kelimeler genellikle birden fazla token'a bölünüyor. Türkçe'de bu biraz daha karmaşık çünkü ekli dilimiz token sayısını artırıyor. Faturalandırma token üzerinden yapılıyor, o yüzden prompt'larınızı gereksiz uzatmamak önemli.

**Bağlam penceresi** modelin aynı anda "görebildiği" metin miktarı. 8K, 32K, 200K gibi rakamlar görürsünüz. Büyük belgelerle çalışıyorsanız bu sınır kritik. Ama şunu da bilin: pencere büyüdükçe maliyet de artıyor.

**Temperature** ve benzeri parametreler çıktının ne kadar "yaratıcı" olacağını belirliyor. Kod üretimi için düşük tutuyorum (0.1-0.3 arası), yaratıcı metin için biraz yükseltiyorum. Üretim sistemlerinde genellikle düşük tutmak daha güvenli.

**Streaming** kullanıcı deneyimi için çok önemli. Cevabın tamamını beklemek yerine parça parça göstermek, uygulamayı çok daha responsive hissettiriyor.

**Function/Tool calling** ise işlerin gerçekten ilginçleştiği nokta. Model bir fonksiyon çağırması gerektiğine karar veriyor, siz o fonksiyonu çalıştırıp sonucu geri veriyorsunuz. Ajan sistemlerinin temeli bu.

## Mimari Nasıl Görünüyor?

![LLM API mimarisi](/images/llm-architecture.svg)

Tipik bir akış şöyle işliyor: Kullanıcıdan gelen istek önce sizin uygulamanıza geliyor. Burada gerekli ön işlemleri yapıyorsunuz — belki RAG için vektör veritabanından ilgili belgeleri çekiyorsunuz, belki iş kurallarını uyguluyorsunuz. Sonra LLM API'sine istek atıyorsunuz, cevabı alıp işleyip kullanıcıya dönüyorsunuz. Arada loglama, önbellekleme ve hata yönetimi katmanları var tabii.

## Kimler Ne Sunuyor?

**OpenAI** hâlâ en yaygın tercih. GPT-4 ailesi güçlü, dokümantasyon iyi, topluluk geniş. Hızlı prototipleme için ideal.

**Azure OpenAI** kurumsal projeler için mantıklı. Azure'un güvenlik altyapısıyla (VNet, Key Vault, RBAC) entegre çalışıyor. Model güncellemeleri biraz geç gelebiliyor ama kurumsal müşteriler için bu genellikle artı.

**Anthropic'in Claude'u** uzun belgelerle çalışmada çok iyi. Özellikle hukuk, araştırma gibi metin yoğun alanlarda tercih ediliyor.

**Google Gemini** multimodal (görsel + metin) işlerde güçlü. GCP kullanıyorsanız entegrasyon kolay.

**Mistral** ve **Cohere** maliyet-performans dengesinde iyi alternatifler. Her iş için GPT-4 gerekmiyor, bazen daha küçük modeller işi görüyor.

**Açık kaynak** (Llama, Mixtral vb.) veri egemenliği şartsa veya özel fine-tuning gerekiyorsa değerlendirmeye değer. Ama operasyonel yükü hafife almayın.

## Gerçek Hayatta Nerede Kullanılıyor?

- **Yardım masası ve chatbot**: Ürün dokümantasyonu üzerinden RAG ile müşteri sorularını yanıtlama
- **Özetleme**: Toplantı notları, uzun e-postalar, raporlar
- **Veri çıkarımı**: Faturalardan, sözleşmelerden yapılandırılmış veri elde etme
- **Kod asistanı**: PR inceleme, test önerisi, dokümantasyon
- **Semantik arama**: Klasik keyword aramanın ötesinde, anlamsal eşleşme

## Kod Örnekleri

### Basit bir Chat çağrısı (.NET)

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

### Streaming ile daha iyi UX

Kullanıcının cevabı beklerken ekranın donması hiç hoş değil. Streaming ile cevap parça parça geliyor:

```csharp
var request = new HttpRequestMessage(HttpMethod.Post, "https://api.openai.com/v1/chat/completions");
request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);
request.Content = new StringContent(JsonSerializer.Serialize(new
{
    model = "gpt-4o-mini",
    stream = true,
    messages = new[] { new { role = "user", content = "Mikroservis mimarisinin avantajlarını anlat." } }
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
        // Her chunk'ı işle ve UI'a gönder
    }
}
```

### Basit RAG Yapısı

RAG'ın mantığı şu: Modele soruyu sormadan önce, ilgili bilgileri vektör veritabanından çekip prompt'a ekliyorsun. Model böylece kendi bilmediği şeyleri de "biliyor" gibi cevap verebiliyor.

![RAG akışı](/images/rag-flow.svg)

```csharp
// 1. Kullanıcı sorusunu embedding'e çevir
// 2. Vektör DB'den en yakın 5 dokümanı getir
// 3. Bu dokümanları prompt'a ekle

var context = string.Join("\n\n", relevantDocs.Select(d => d.Content));

var prompt = $"""
Aşağıdaki bilgileri kullanarak soruyu yanıtla. 
Bilmediğin bir şey sorulursa "bu konuda bilgim yok" de.

Bağlam:
{context}

Soru: {userQuestion}
""";
```

## Maliyet Kontrolü

Bu konuda birkaç ders çıkardım:

1. **Doğru model seçimi**: Her iş için en güçlü model gerekmez. Basit sınıflandırma işleri için GPT-4 yerine GPT-3.5 veya daha küçük modeller yeterli.

2. **Prompt optimizasyonu**: Gereksiz uzun system prompt'lar maliyeti artırıyor. Net ve kısa tutun.

3. **Önbellekleme**: Aynı sorular tekrar tekrar soruluyorsa (FAQ gibi), cevapları cache'leyin.

4. **Batch işleme**: Gerçek zamanlı olması gerekmeyenler için toplu işlem yapın.

## Güvenlik Notları

- **Prompt injection**: Kullanıcı girdisini doğrudan prompt'a koymadan önce temizleyin. Kötü niyetli kullanıcılar modeli manipüle etmeye çalışabilir.
- **Hassas veri**: PII (kişisel tanımlayıcı bilgi) içeren verileri modele göndermeden önce maskeleyin.
- **Çıktı kontrolü**: Model bazen istenmeyen şeyler üretebilir. Kritik işlemlerde çıktıyı doğrulayın.

## Üretime Çıkmadan Önce

Kontrol listem şöyle:

- [ ] Prompt'lar versiyonlanmış mı?
- [ ] Rate limiting var mı?
- [ ] Retry mekanizması kuruldu mu?
- [ ] Loglar tutuluyor mu (istek, yanıt, token sayısı, maliyet)?
- [ ] Fallback senaryoları düşünüldü mü (API çökerse ne olacak)?
- [ ] Veri gizliliği politikası netleştirildi mi?

---

LLM API'ları doğru kullanıldığında gerçekten güçlü araçlar. Ama "AI her şeyi çözer" yaklaşımıyla değil, spesifik problemlere spesifik çözümler olarak düşünmek daha sağlıklı. Umarım bu yazı başlangıç noktası olarak işinize yarar.
