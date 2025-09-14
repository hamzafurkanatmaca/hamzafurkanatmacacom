---
title: "LLM API'lar Nedir ve Nasıl Kullanılırlar?"
description: "Temeller, kullanım senaryoları, mimari örüntüler, maliyet/performans ve üretim için en iyi uygulamalar."
pubDate: "Sep 14 2025"
heroImage: "/images/llm-architecture.svg"
---

### Giriş
Büyük dil modelleri (LLM – Large Language Model) artık yazı yazmaktan kod üretmeye, veriden bilgi çıkarmaktan arama deneyimini geliştirmeye kadar birçok alanda kullanılıyor. LLM'lere doğrudan model barındırmadan erişmenin en yaygın yolu bir LLM API'ını çağırmaktır. Bu yazıda LLM API'larının ne olduğunu, hangi kavramlarla çalıştıklarını, gerçek dünyada nasıl konumlandıklarını ve üretim ortamında nasıl güvenli ve verimli kullanılabileceklerini özetliyorum.

### LLM API nedir?
LLM API; sağlayıcının barındırdığı bir modeli HTTP üzerinden çağırmanızı sağlayan arabirimdir. İstemci uygulama metin (veya çok modlu veriler) gönderir, model verilen bağlama göre bir çıktı üretir ve sonuç JSON olarak geri döner. Bu yaklaşımın avantajları:
- Bakım ve donanım maliyeti yoktur.
- En güncel modellere hızla erişirsiniz.
- Ölçeklenebilirlik ve SLA sağlayıcıya devredilir.

Dezavantajları:
- Veri gizliliği ve regülasyon gereksinimleri için ek önlem gerekir.
- Token bazlı maliyet kontrol edilmelidir.
- Sağlayıcıya bağımlılık (vendor lock‑in) riski bulunur.

### Temel kavramlar
- **Token**: Modelin hesaplama birimidir. Kelimelerin alt parçaları olabilir. Ücretlendirme ve sınırlar token üzerinden yapılır.
- **Bağlam penceresi (context window)**: Modelin bir seferde “hatırlayabildiği” bağlam miktarıdır. 8k, 32k, 200k+ gibi değerlerle ifade edilir.
- **Prompt**: Modele verdiğiniz talimat ve veriler. Chat tabanlı API’larda rol tabanlı mesajlardan oluşur (system, user, assistant, tool).
- **Sampling parametreleri**: `temperature`, `top_p`, `top_k` gibi ayarlar üretimin çeşitliliğini etkiler. Üretim sistemlerinde deterministiklik için genellikle düşürülür.
- **Max tokens**: Cevabın en fazla kaç token olacağını sınırlar.
- **Stop sequences**: Modelin belirli dizelerde üretimi durdurmasını sağlar.
- **Streaming**: Cevabın parça parça (chunk) gelmesini sağlayarak UI’ı daha akıcı kılar.
- **Function/Tool calling**: Modelin tanımlı şemalara göre fonksiyon çağrısı önermesi; arka planda gerçek fonksiyon çalıştırıp sonucu tekrar modele vermenizi sağlar (ajan/agent desenlerinin temeli).
- **Embeddings**: Metinleri vektör uzayına taşıyarak semantik arama, benzerlik skoru ve RAG (Retrieval‑Augmented Generation) için kullanılır.
- **RAG vs. Fine‑tuning**: Özel bilgi tabanı kullanımı için çoğu senaryoda RAG yeterlidir; kalıcı stil/terim uyumu ve dar görevlerde fine‑tuning tercih edilir.
- **Çok modluluk**: Görsel, ses veya dosya girdileriyle çalışabilen modeller (vision, audio, multimodal).

### Görsel: Yüksek Seviyeli Mimari
![LLM API mimarisi – istemci, API, uygulama katmanı, gözlemlenebilirlik](/images/llm-architecture.svg)

Yukarıdaki diyagramda tipik bir akış görülüyor: İstemci → API Gateway → Uygulama (RAG/iş kuralları) → LLM Sağlayıcı → Gözlemlenebilirlik ve önbellek katmanları.

### Başlıca sağlayıcılar ve ekosistem
- **OpenAI / Azure OpenAI**: GPT‑4o/4.1 ve türevleri; kurumsal güvenlik, iyi dokümantasyon.
- **Anthropic**: Claude 3 ailesi; uzun bağlam ve güvenlik vurgusu.
- **Google**: Gemini ailesi; güçlü multimodal yetenekler.
- **Mistral, Cohere**: Hafif ve hızlı modeller; üretim odaklılık.
- **Açık kaynak**: `Ollama`, `vLLM`, `TGI` ile kendi altyapınızda barındırma. Veri egemenliği gereken senaryolarda ideal.

### Tipik kullanım senaryoları
- **Soru‑Cevap ve Yardım Masası**: Ürün dokümanları üzerinden RAG ile doğru ve izlenebilir yanıtlar.
- **Özetleme ve raporlama**: Toplantı notları, loglar, e‑postalar, sözleşmeler.
- **Bilgi çıkarımı (extraction)**: Fatura, sözleşme gibi dokümanlardan yapılandırılmış alanlar.
- **Kod asistanı ve PR yardımı**: Kod yorumlama, test önerisi, commit mesajları.
- **Semantik arama**: Embedding + vektör veritabanı (PGVector, Qdrant, Weaviate, Pinecone).
- **Ajanlar**: Takvim, e‑posta, CRM gibi araç çağrılarıyla uzun görevleri yönetme.

---

## Kod Örnekleri

### 1) .NET ile Basit Chat Completion çağrısı
```csharp
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

var http = new HttpClient();
http.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", Environment.GetEnvironmentVariable("LLM_API_KEY"));

var payload = new
{
    model = "gpt-4o-mini",
    messages = new object[]
    {
        new { role = "system", content = "Sen deneyimli bir .NET ve AI danışmanısın." },
        new { role = "user", content = "RAG mimarisini 3 maddeyle özetler misin?" }
    },
    temperature = 0.2,
    max_tokens = 300
};

var json = JsonSerializer.Serialize(payload);
var res = await http.PostAsync("https://api.openai.com/v1/chat/completions",
    new StringContent(json, Encoding.UTF8, "application/json"));
res.EnsureSuccessStatusCode();
var body = await res.Content.ReadAsStringAsync();
Console.WriteLine(body);
```

### 2) Streaming yanıtı okuma (SSE)
```csharp
var request = new HttpRequestMessage(HttpMethod.Post, "https://api.openai.com/v1/chat/completions");
request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", Environment.GetEnvironmentVariable("LLM_API_KEY"));
request.Content = new StringContent(JsonSerializer.Serialize(new
{
    model = "gpt-4o-mini",
    stream = true,
    messages = new[] { new { role = "user", content = "Bir paragraf üreterek yavaşça akıt." } }
}), Encoding.UTF8, "application/json");

using var client = new HttpClient();
using var response = await client.SendAsync(request, HttpCompletionOption.ResponseHeadersRead);
response.EnsureSuccessStatusCode();
await using var stream = await response.Content.ReadAsStreamAsync();
using var reader = new StreamReader(stream);
while (!reader.EndOfStream)
{
    var line = await reader.ReadLineAsync();
    if (string.IsNullOrWhiteSpace(line)) continue;
    if (line.StartsWith("data: "))
    {
        var jsonLine = line.Substring(6);
        if (jsonLine == "[DONE]") break;
        Console.Write(jsonLine);
    }
}
```

### 3) Embeddings + PGVector ile semantik arama
Önce PostgreSQL tarafı (pgvector eklentisi):
```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE IF NOT EXISTS documents (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  embedding VECTOR(1536) -- model vektör boyutu
);

-- En yakın komşu (cosine distance)
-- SELECT id, title FROM documents ORDER BY embedding <-> :query_embedding LIMIT 5;
```

Embeddings üretip kaydetme:
```csharp
var embPayload = new { model = "text-embedding-3-small", input = new[] { "Mikroservislerde devre kesici deseni" } };
var embRes = await http.PostAsync("https://api.openai.com/v1/embeddings",
    new StringContent(JsonSerializer.Serialize(embPayload), Encoding.UTF8, "application/json"));
var embJson = await embRes.Content.ReadAsStringAsync();
// embJson'i parse edip vektörü (float[]) çıkarın ve documents.embedding alanına yazın.
```

Sorgulama:
```sql
-- :q_embedding = kullanıcı sorgusunun embedding'i
SELECT id, title, content
FROM documents
ORDER BY embedding <-> :q_embedding
LIMIT 5;
```

### 4) Basit RAG iskeleti
```csharp
// 1) Kullanıcı sorgusu → embedding
// 2) Vektör DB'den en ilgili 5 dokümanı çek
// 3) Doküman parçalarını prompt'a ekle ve modeli çağır
var context = string.Join("\n\n", topDocs.Select(d => $"[Kaynak:{d.id}] {d.content}"));
var prompt = $"Aşağıdaki bağlamdan faydalanarak yanıt ver. Kaynak id'lerini belirt.\n\nBağlam:\n{context}\n\nSoru: {userQuestion}";
```

---

### Görsel: RAG akışına genel bakış
![RAG akışı – indeksleme, retrieval, prompt birleştirme, yanıt](/images/rag-flow.svg)

### Maliyet ve performans
- **Doğru model**: “En büyük” model yerine iş için yeterli en küçük model. Gerektiğinde kademeli seçim (router).
- **Prompt optimizasyonu**: Gereksiz bağlamı atın; talimatı netleştirin; çıktı şemasını sınırlayın.
- **Token ekonomisi**: Toplam bağlam = giriş + sistem + çıktı. Ücretlendirme iki yöne de uygulanır.
- **Batch/parallel**: Bağımsız istekleri toplu veya paralel çalıştırın; kuyrukla dengeleyin.
- **Önbellek ve deduplikasyon**: Aynı prompt’lar için yanıtları paylaşın.

### Güvenlik ve gizlilik
- **Prompt injection**: Modelin talimatlarını bozabilecek içeriğe karşı filtreleme ve çıktı denetimi.
- **Veri sızıntısı**: PII maskeleme, gizli alanların prompt’a girmesini engelleme.
- **Sağlayıcı politikaları**: Veri saklama (retention) ayarlarını ve bölge (region) gereksinimlerini inceleyin.
- **Guardrails**: Şema doğrulama, moderasyon, kural tabanlı kontroller ve ikinci model ile doğrulama.

### Değerlendirme (eval) ve kalite
- **Offline eval**: Etiketli örnek seti; otomatik skorlama metrikleri (doğruluk, kapsama, tutarlılık).
- **LLM‑as‑a‑judge**: Hakem modelle dikkatli karşılaştırma; insan doğrulamasıyla destekleyin.
- **Online ölçüm**: A/B test, kullanıcı geri bildirimi, üretim telemetrisi.

### Üretime alma kontrol listesi
- Versiyonlanmış prompt’lar ve değişiklik günlüğü
- Oran sınırlama, retry ve idempotency anahtarları
- Zorunlu gözlemlenebilirlik (istek/yanıt, token, maliyet)
- Kaçış yolları (fallback model, zaman aşımı, boşta güvenli davranış)
- Gizlilik politikası ve veri saklama beyanı

### Sonuç
LLM API’ları; içerik üretimi, arama, otomasyon ve yardım masası gibi çok geniş bir yelpazede hızlı değer üretiyor. Doğru model seçimi, iyi tasarlanmış prompt’lar, ölçülebilirlik ve güvenlik ilkeleri ile birleştirildiğinde, kurumsal ölçekte sürdürülebilir çözümler kurmak mümkün. Bu yazıdaki kontrol listelerini başlangıç noktası olarak kullanabilir, uygulamanızın ihtiyaçlarına göre derinleştirebilirsiniz.
