# Açık Deniz — Proje Talimatları

## Proje Nedir
Tarayıcıda çalışan, tek dosyalık (`index.html`) gerçek zamanlı gemi savaşı oyunu.
Sunucu tarafı kodu yok. Vanilla JS + Three.js (3D sahne) + 2D canvas (HUD).

## Altyapı
- GitHub: `osmancnar/gemi-oyunum`. `main` dalına her commit Netlify'a otomatik yayınlanır.
- Modeller/görseller (.glb, .png) **index.html ile AYNI klasöre** yüklenmeli.
- Firebase: Auth, Firestore (`players/{uid}`, `chat`, `fleets`, `fleetRequests`, `auctions/{itemId}`, `tickets`), RTDB (`rooms/{roomId}`, harita bazlı).
- **Claude'un canlı Firebase/Firestore verisine erişimi YOK** — sadece statik dosyalar üzerinde çalışılıyor.
- Test akışı: kullanıcı localhost'ta Python `http.server` ile test ediyor, sonra GitHub'a push edip Netlify'a atıyor. Oyun sekmesi AÇIKKEN Firestore'da elle veri değiştirmek işe yaramaz.

## KRİTİK KURALLAR (defalarca hataya yol açtı)
1. **TDZ**: Yeni global `let`/`const` tanımları `initGame()`'den ÖNCE yapılmalı. Bu hata en az üç kez oyunun tamamen açılmamasına yol açtı.
2. **"Tanımsız = izin var" tuzağı**: Bir koşulda eşik/değer tanımsızken sessizce atlanması tehlikeli — kontrol et.
3. **Doğrulama**: Sözdizimi kontrolü (node --check) yetmez; gerçek RTDB/Firestore transaction akışıyla ve gerçek login/auth callback akışıyla uçtan uca test şart.
4. **Test mock tuzağı**: Sahte DOM fonksiyonları eksik/no-op bırakılırsa yanlış pozitif sonuç verir. `innerHTML` string ataması mock'ta gerçek çocuk düğüm oluşturmaz — `panel.children.length` ile veya gerçekçi innerHTML setter ile doğrula.
5. **HTML `disabled`**: Gerçek tarayıcıda click olayını hiç tetiklemez — engellemeyi JS içinde (mesajla) yap.
6. **Özel 3D modeller (Meshy)**: `mesh.rotation.y = -ship.angle` mesh'in rotation'ını siler — özel modeller MUTLAKA `THREE.Group()` içine alınmalı. Meshy dosyalarının çoğu 2-3 tasarım bir arada gelir, önce histogram/boşluk kontrolüyle doğrula.
7. **Göç (migration) gerekliliği**: Mevcut bir mekanik yeni sisteme geçirilirken eski kayıtlı kullanıcı verisi de uyumlu hale getirilmeli.
8. **Hafıza/iddia doğruluğu**: "X zaten entegre edildi" gibi büyük iddiaları asla varsaymadan önce dosyada grep ile doğrula.
9. **Kapsam disiplini**: Kullanıcı belirli bir alanı (örn. sadece Pazar kartı + hotbar) işaret ettiğinde, paylaşılan CSS sınıflarını (`.cannonHero`, `.marketProductImg` vb.) değiştirirken kapsam dışına taşma — class-scoped override kullan.

## Tasarım Sistemi (UI/UX)
`:root` CSS token bloğu: `--mmo-bg-black/navy/wood/bronze/gold/emerald/text`, `--mmo-font-display=Cinzel`, `--mmo-font-body=IBM Plex Mono`.
Ortak sınıflar: `.mmoCard` (kategoriye göre sol kenarlık), `.mmoBadge`, `.mmoProgressTrack/.mmoProgressFill`, `.mmoScroll`, animasyonlar (`mmoFadeIn/mmoSlideIn/mmoGlowPulse/mmoFloat`).
8 adımlık genel UI/UX geçişi TAMAMLANDI (Genel UI Sistemi, Genel Bakış, Seyir Defteri, Pazar, Görevler, Mojuo, Filo, Rütbe). Şu an ince ayar/gözden geçirme aşamasındayız.

## Yasaklar
- Canlı Firebase/Firestore verisine dokunma iddiasında bulunma (erişim yok).
- Filo sekmesinin Firebase'e bağlı dinamik HTML üretim mantığına (`renderFleetTab` vb.) dokunma — sadece paylaşılan CSS sınıfları stillendirilebilir.
- Paylaşılan bir CSS sınıfını değiştirirken önce o sınıfın başka nerelerde kullanıldığını grep ile kontrol et (sızıntı riski).
