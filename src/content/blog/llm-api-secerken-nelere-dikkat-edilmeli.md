---
title: "LLM API Seçerken Nelere Dikkat Edilmeli?"
description: "Sadece model kalitesi değil: veri gizliliği, maliyet, gecikme ve operasyonel gereksinimler de önemli."
pubDate: "Sep 19 2025"
heroImage: "/images/llm.jpg"
---

Herkes "en iyi LLM hangisi" diye soruyor ama esasında bu soru yanlış. Asıl sorulması gereken "benim işime en çok yarayan hangisi" olmalı. Çünkü bir chatbot için doğru olan seçim, hassas veri işleyen bir sağlık uygulaması için tamamen yanlış olabiliyor.

## Veri meselesi

İlk bakılacak yer burası. Gönderdiğim veriler nereye gidiyor, kim görüyor, ne kadar saklanıyor? Eğer sadece genel sorular sorulacaksa çok da önemli değil, ama müşteri bilgisi veya finansal veri işin içine girince durumlar değişiyor.

Azure OpenAI veya AWS Bedrock gibi kurumsal servisler bu konuda daha net sözleşmeler sunuyor. Tabii bedeli var ama KVKK/GDPR derdinden kurtuluyoruz.

## Hangi model?

İlk akla gelen genelde en büyük modeli kullanmak oluyor. Ama basit bir iş için en pahalı modeli çalıştırmak gereksizdir. Küçük modelle başlayıp yetmezse büyüğüne geçmek daha mantıklı olur. Hatta bazı sistemlerde ikisini birden kullanıyoruz — kolay sorular küçük modele gidiyor, zor olanlar büyüğüne. Bu tarz bir route mantığı son derece işlevseldir.

Bir de bağlam penceresi var. Kısa metinlerle çalışıyorsan 8K token yeter, ama uzun dökümanlar işin içine girince 128K lazım olabiliyor. Fazla token = fazla para, o yüzden gereksiz yere büyük pencere seçmeye gerek yoktur.

## Maliyet

LLM faturaları beklenmedik yerlere gidebiliyor. Özellikle cache kullanmadan çalışınca her istek para yazıyor. Token fiyatlandırmasını anlamak lazım: hem gönderdiğin hem aldığın token için ödüyorsun, üstelik çıkış tokenları genelde daha pahalı.

Türkçe kullanıyorsan bir de şu var: Türkçe ek yapısı yüzünden aynı cümle İngilizce'den daha fazla token tutuyor. Yani Türkçe prompt'lar biraz daha pahalıya geliyor.

Prompt'ları kısa tutmak, tekrar eden sorular için cache kullanmak, acil olmayan işleri toplu çalıştırmak faturayı düşürebilir.

## Hız

Kullanıcı 2 saniye beklemek istemez. 200ms ile 2 saniye arasında ciddi fark vardır — biri fark edilmiyor, diğeri sinir bozuyor.

Büyük model ve uzun prompt yavaş yanıt demek. Streaming açmak iyi bir çözüm, en azından cevap yavaş yavaş geliyor ve kullanıcı bir şeyler olduğunu görüyor. Arka planda çalışan işler için (özet çıkarma, analiz) hız o kadar kritik değil tabi.

## Çıktı kontrolü

Model bazen saçmalıyor. Aynı soruya bir gün güzel cevap veriyor, ertesi gün alakasız bir şey söylüyor. Çıktıyı bir şekilde kontrol etmek lazım — JSON bekliyorsan şema validasyonu yapmak, hassas konularda filtreleme eklemek gibi.

## Log tutmak

Neyi loglamak lazım: istekler, yanıtlar, token sayıları, ne kadar sürdü, ne kadara mal oldu. Bunlar olmadan ne bütçe planlaması yapılır ne de sorun çıkınca debug.

## Hata durumu

Sağlayıcının limiti var, çok istek atınca 429 hatası geliyor. Retry mekanizması kurmak lazım. Bir de şunu düşünmek lazım: servis tamamen çökerse uygulama ne yapacak? Hata mesajı mı gösterecek yoksa tamamen mı duracak?

## Bağımlılık

Tek bir sağlayıcıya çok bağlanmamak iyi olur. Fiyatlar değişiyor, yeni modeller çıkıyor, bazen daha iyi alternatifler oluyor. LLM çağrılarını bir katmanın arkasına koymak, gerekince sağlayıcı değiştirmeyi kolaylaştırıyor.

---

Esasen, uzun araştırma yapmak yerine küçük bir deneme yapmak çoğu zaman daha iyidir. Gerçek verilerle test edip sonuçlara bakmak, pazarlama yazılarını okumaktan daha efektiftir.

Faydalı olması dileğiyle.
