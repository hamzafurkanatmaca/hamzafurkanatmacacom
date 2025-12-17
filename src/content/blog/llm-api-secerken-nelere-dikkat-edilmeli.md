---
title: "LLM API Seçerken Nelere Dikkat Edilmeli?"
description: "Sadece model kalitesi değil: veri gizliliği, maliyet, gecikme ve operasyonel gereksinimler de önemli."
pubDate: "Sep 14 2025"
heroImage: "/images/decision-guide.svg"
---

Herkes "en iyi LLM hangisi" diye soruyor ama aslında yanlış soru bu. Asıl sorulması gereken "benim işime en çok yarayan hangisi" olmalı. Çünkü bir chatbot için doğru olan seçim, hassas veri işleyen bir sağlık uygulaması için tamamen yanlış olabiliyor.

## Veri meselesi

İlk bakılacak yer burası. Gönderdiğin veriler nereye gidiyor, kim görüyor, ne kadar saklanıyor? Eğer sadece genel sorular sorulacaksa çok da önemli değil, ama müşteri bilgisi veya finansal veri işin içine girince durum değişiyor.

Azure OpenAI veya AWS Bedrock gibi kurumsal servisler bu konuda daha net sözleşmeler sunuyor. Tabii bedeli var ama KVKK/GDPR derdinden kurtuluyorsunuz.

## Hangi model?

İlk akla gelen genelde en büyük modeli kullanmak oluyor. Ama basit bir iş için en pahalı modeli çalıştırmak gereksiz. Küçük modelle başlayıp yetmezse büyüğüne geçmek daha mantıklı. Hatta bazı sistemlerde ikisini birden kullanıyoruz — kolay sorular küçük modele gidiyor, zor olanlar büyüğüne.

Bir de bağlam penceresi var. Kısa metinlerle çalışıyorsan 8K token yeter, ama uzun dökümanlar işin içine girince 128K lazım olabiliyor. Fazla token = fazla para, o yüzden gereksiz yere büyük pencere seçmeye gerek yok.

## Maliyet

LLM faturaları beklenmedik yerlere gidebiliyor. Özellikle cache kullanmadan çalışınca her istek para yazıyor. Token fiyatlandırmasını anlamak lazım: hem gönderdiğin hem aldığın token için ödüyorsun, üstelik çıkış tokenları genelde daha pahalı.

Türkçe kullanıyorsan bir de şu var: Türkçe ek yapısı yüzünden aynı cümle İngilizce'den daha fazla token tutuyor. Yani Türkçe prompt'lar biraz daha pahalıya geliyor.

Prompt'ları kısa tutmak, tekrar eden sorular için cache kullanmak, acil olmayan işleri toplu çalıştırmak faturayı düşürüyor.

## Hız

Kullanıcı 2 saniye beklemiyor. 200ms ile 2 saniye arasında ciddi fark var — biri fark edilmiyor, diğeri sinir bozuyor.

Büyük model ve uzun prompt yavaş yanıt demek. Streaming açmak iyi bir çözüm, en azından cevap yavaş yavaş geliyor ve kullanıcı bir şeyler olduğunu görüyor. Arka planda çalışan işler için (özet çıkarma, analiz) hız o kadar kritik değil tabii.

## Çıktı kontrolü

Model bazen saçmalıyor. Aynı soruya bir gün güzel cevap veriyor, ertesi gün alakasız bir şey söylüyor. Çıktıyı bir şekilde kontrol etmek lazım — JSON bekliyorsan şema validasyonu yapmak, hassas konularda filtreleme eklemek gibi.

## Log tutmak

Neyi loglamak lazım: istekler, yanıtlar, token sayıları, ne kadar sürdü, ne kadara mal oldu. Bunlar olmadan ne bütçe planlaması yapılıyor ne de sorun çıkınca debug.

## Hata durumu

Sağlayıcının limiti var, çok istek atınca 429 hatası geliyor. Retry mekanizması kurmak lazım. Bir de şunu düşünmek lazım: servis tamamen çökerse uygulama ne yapacak? Hata mesajı mı gösterecek yoksa tamamen mı duracak?

## Bağımlılık

Tek bir sağlayıcıya çok bağlanmamak iyi oluyor. Fiyatlar değişiyor, yeni modeller çıkıyor, bazen daha iyi alternatif oluyor. LLM çağrılarını bir katmanın arkasına koymak, gerekince sağlayıcı değiştirmeyi kolaylaştırıyor.

---

Sonuç olarak, uzun araştırma yapmak yerine küçük bir deneme yapmak daha iyi. Gerçek verilerle test edip sonuçlara bakmak, pazarlama yazılarını okumaktan daha güvenilir.
