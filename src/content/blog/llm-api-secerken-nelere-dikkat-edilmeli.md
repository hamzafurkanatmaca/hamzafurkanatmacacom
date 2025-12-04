---
title: "LLM API Seçerken Nelere Dikkat Edilmeli?"
description: "Sadece model kalitesi değil: veri gizliliği, maliyet, gecikme ve operasyonel gereksinimler de önemli."
pubDate: "Sep 14 2025"
heroImage: "/images/decision-guide.svg"
---

"En iyi LLM hangisi?" sorusu yanlış soru. Doğrusu şu: "Benim için en uygun LLM hangisi?" Çünkü cevap tamamen bağlama göre değişiyor. Bir e-ticaret sitesinin chatbot'u ile bir sağlık uygulamasının AI asistanı farklı kriterlere göre değerlendirilmeli.

Bu yazıda, LLM API seçerken gerçekten önemli olan faktörleri ve kendi deneyimlerimden çıkardığım dersleri paylaşacağım.

## Veri Gizliliği: İlk ve En Önemli Soru

Her şeyden önce şunu netleştirin: Modelinize göndereceğiniz veriler nereye gidebilir?

Bazı durumlarda cevap basit — genel bilgi, hassas olmayan içerik, herhangi bir sağlayıcıya gönderilebilir. Ama müşteri verileri, sağlık bilgileri, finansal kayıtlar söz konusuysa işler karmaşıklaşıyor.

Sorulması gerekenler:
- Sağlayıcı verileri ne kadar süre saklıyor?
- Veri hangi ülkede/bölgede işleniyor?
- Model eğitimi için kullanılıyor mu?
- KVKK, GDPR, HIPAA gibi regülasyonlara uyumlu mu?

Azure OpenAI veya AWS Bedrock gibi kurumsal çözümler bu konuda daha şeffaf. VNet entegrasyonu, veri işleme sözleşmeleri, denetim logları sunuyorlar. Bedelini ödüyorsunuz ama güvenlik ekibiniz rahat ediyor.

## Model Kapasitesi: Daha Büyük Her Zaman Daha İyi Değil

İlk refleks genellikle "en güçlü modeli kullanayım" oluyor. Ama bu hem pahalı hem de çoğu zaman gereksiz.

Şöyle düşünün: Basit bir sentiment analizi için GPT-4 kullanmak, Formula 1 aracıyla market alışverişine gitmek gibi. İşi görür ama gereksiz pahalı ve overkill.

**Pratik yaklaşım:**
1. Önce en küçük modelle deneyin
2. Sonuçları değerlendirin
3. Yetersizse bir üst modele geçin
4. Veya görev bazlı routing yapın — basit işler küçük model, karmaşık işler büyük model

Bağlam penceresi de önemli. 8K token yeterli mi yoksa 128K mı lazım? Uzun belgelerle çalışıyorsanız büyük bağlam şart ama her fazla token maliyete yansıyor.

## Maliyet: Kontrol Edilmezse Sürpriz Yapar

LLM API faturaları kontrolsüz bırakılırsa hızla kababilir. Bunu yaşadım, öğrendim.

Token bazlı fiyatlandırmayı anlamak önemli:
- Giriş tokenları (prompt) + çıkış tokenları (cevap) = toplam maliyet
- Genellikle çıkış tokenları daha pahalı
- Türkçe gibi agglutinative dillerde token sayısı İngilizce'ye göre daha yüksek

**Maliyeti düşürmek için:**
- Prompt'ları kısa ve öz tutun
- Gereksiz context eklemeyin
- Sık sorulan sorular için önbellekleme yapın
- Gerçek zamanlı olması gerekmeyen işleri batch'leyin
- Aynı soruları deduplikasyonla filtreleyin

Bir de şunu hesaplayın: Aylık beklenen kullanımınız ne? 1000 istek mi, 100.000 mi, 10 milyon mu? Hacme göre fiyat kırılımları değişiyor, bazen kurumsal anlaşma mantıklı olabiliyor.

## Gecikme: Kullanıcı Deneyimini Belirliyor

Bir chatbot için 200ms ile 2000ms arasında fark var mı? Evet, çok var. Kullanıcı algısı açısından kritik.

**Gecikmeyi etkileyen faktörler:**
- Model büyüklüğü (büyük model = daha yavaş)
- Bağlam uzunluğu (uzun prompt = daha yavaş)
- Sağlayıcının altyapısı ve bölgesi
- Ağ gecikmesi

**İyileştirme taktikleri:**
- Streaming kullanın — cevap parça parça gelsin
- Daha küçük model seçin (mümkünse)
- Prompt'u optimize edin
- Sağlayıcıya yakın bölgede konuşlanın

Gerçek zamanlı chat uygulamaları için gecikme kritik. Arka plan işleme için (özetleme, analiz) tolere edilebilir.

## Prompt ve Çıktı Kontrolü

Bu kısım genellikle hafife alınıyor ama üretim kalitesini doğrudan etkiliyor.

**Bakılması gerekenler:**
- **Function/Tool calling**: Model harici fonksiyon çağırabilmeli (ajan sistemleri için şart)
- **Structured output**: JSON schema ile çıktı formatını zorlayabilmeli
- **System prompt**: Modelin davranışını tutarlı şekilde yönlendirebilmeli
- **Stop sequences**: Modelin nerede durması gerektiğini belirleyebilmeli

Guardrail'ler de önemli. Model bazen beklenmedik şeyler üretebiliyor. Kritik sistemlerde:
- Çıktıyı şema doğrulamasından geçirin
- Gerekirse ikinci bir modelle kontrol edin
- Hassas içerik filtreleri ekleyin

## Gözlemlenebilirlik: Göremediğinizi Yönetemezsiniz

Üretimde mutlaka şunları loglayın:
- Her istek/yanıt (veya örnekleme yapın)
- Token sayıları
- Gecikme süreleri
- Hata oranları ve türleri
- Maliyet (token başına hesaplama)

Bu veriler olmadan:
- Bütçe planlaması yapamaz
- Performans sorunlarını tespit edemez
- Kalite değerlendirmesi yapamaz
- Optimizasyon fırsatlarını göremezsiniz

Langfuse, LangSmith, Helicone gibi araçlar bu işi kolaylaştırıyor. Veya kendi loglama altyapınızı kurabilirsiniz.

## Rate Limiting ve Hata Yönetimi

Sağlayıcıların rate limit'leri var. Bunları aşarsanız istek reddedilir. Üretimde buna hazırlıklı olmalısınız:

- Exponential backoff ile retry
- Kuyruk mekanizması ile istekleri dengeleme
- Fallback sağlayıcı (A çökerse B'ye git)
- Graceful degradation (AI olmadan da çalışabilmeli)

Bu son madde önemli: AI servisi çökerse uygulamanız ne yapacak? "Şu an bu özellik kullanılamıyor" demek mi, tamamen çökmek mi? Bunu önceden planlamak lazım.

## Ekosistem ve Entegrasyon

Günlük hayatta şunlar önemli:
- SDK kalitesi (C#, Python, Node.js için)
- Dokümantasyon (örnekler, best practices)
- Topluluk (sorun yaşayınca yardım bulabilmek)
- Üçüncü parti entegrasyonlar (LangChain, Semantic Kernel, vb.)

Vektör veritabanları (PGVector, Pinecone, Weaviate) ile entegrasyon RAG için kritik. İş akışı araçları (Temporal, Durable Functions) ile entegrasyon ajan sistemleri için önemli.

## Vendor Lock-in: Gelecekteki Esneklik

Tek bir sağlayıcıya sıkı sıkıya bağlanmak risikli. Fiyatlar değişebilir, hizmet kesilebilir, daha iyi alternatifler çıkabilir.

**Bağımlılığı azaltmak için:**
- LLM çağrılarını bir abstraction katmanının arkasına koyun
- Prompt'ları versiyonlayın ve sağlayıcıdan bağımsız tutun
- Birden fazla sağlayıcıyı test edin
- Kritik sistemlerde fallback sağlayıcı belirleyin

LiteLLM, Portkey gibi araçlar bu abstraction'ı kolaylaştırıyor.

---

## Sonuç: Pilot Proje ile Başlayın

Uzun uzun araştırma yapmak yerine, küçük bir pilot proje ile başlamak en sağlıklısı. Gerçek verilerinizle, gerçek kullanım senaryolarınızla test edin.

Ölçün:
- Yanıt kalitesi (doğruluk, tutarlılık)
- Gecikme (p50, p95, p99)
- Maliyet (token başına, istek başına)
- Hata oranları

Bu verilerle karar vermek, pazarlama materyallerine güvenmekten çok daha güvenilir.

Ve unutmayın: Bugün doğru olan seçim yarın değişebilir. Bu alan çok hızlı gelişiyor. Esnek kalmak, tek bir çözüme bağlanmamak uzun vadede sizi kurtarır.
