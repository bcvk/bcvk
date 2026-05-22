# Uzakiş — Mekân UX (Wireframe + Karakter Limitleri)

Google Places beslemeli, görsel ağırlıklı mekân deneyimi.

---

## 1. Mekân kartı — Liste / arama sonucu

**Boyut:** tam genişlik, hero **16:9** (min yükseklik 180pt)

| Bölge | Limit | Örnek |
|-------|-------|--------|
| Hero alt başlık (gradient üst) | 40 | Starbucks Kadıköy |
| Satır 2 | 30 | ⭐ 4.6 · ₺₺ · 1.2 km |
| Skor şeridi | 3×18 | Çalışma 82 · Buluşma 91 · Etkinlik 65 |
| Chip (max 3) | 12/chip | WiFi · Sessiz · Priz |
| Rozet | 20 | 3 Uzakiş buluşması bu ay |

**Boş foto:** gradient + harf monogram (mekân adı ilk harf)

**Etkileşim:** tap → Mekân detay; long press → hızlı "Bu mekânda masa aç"

---

## 2. Mekân detay — Tam ekran

### 2.1 Hero galeri
- Yükseklik: **42% ekran**
- Yatay `TabView` / pager, 1–10 foto (Google `photo_refs`)
- Sayfa göstergesi: noktalar
- Sağ üst: ✕ kapat
- Sol alt (gradient): mekân adı **max 50 char**

### 2.2 Özet şerit (hero altı, yapışkan değil)

| Alan | Limit | Örnek |
|------|-------|--------|
| Google puan | 20 | ⭐ 4.6 (1.240) |
| Fiyat | 8 | ₺₺ |
| Durum | 24 | Açık · Kapanış 22:00 |
| Adres | 80 | Caferağa Mah. … (tek satır, truncate) |

### 2.3 Uygunluk — 3 sekme

| Sekme | Başlık limit | İçerik |
|-------|--------------|--------|
| Çalışma | 12 | Skor büyük (0–100), 4 satır: WiFi, Gürültü, Priz, Masa |
| Buluşma | 12 | Sohbet, Oturma, Işık, Kalabalık |
| Etkinlik | 12 | Kapasite tahmini, Grup masası, Rezervasyon notu |

Her satır: ikon + label **20 char** + değer **30 char** + mini bar

**Alt not (topluluk):** max **100 char**  
*"Son Uzakiş notu: WiFi hızlı, priz az."*

### 2.4 Harita bloğu
- Yükseklik: 120pt
- Static map veya MapKit snapshot
- Attribution: "Google Maps" **zorunlu**
- Butonlar:
  - Yol tarifi **20 char** → `comgooglemaps://` veya web
  - Google'da aç **22 char**

### 2.5 Sosyal kanıt

| Alan | Limit |
|------|-------|
| Başlık | 28 | Bu mekânda Uzakiş |
| Satır | 60 | Bu hafta 2 buluşma · Son: Dün akşam kahve |

### 2.6 CTA (yapışkan alt)

| Buton | Limit | Aksiyon |
|-------|-------|---------|
| Birincil | 28 | Bu mekânda masa aç |
| İkincil | 24 | Mekânı kaydet |

---

## 3. Buluşma oluştur — Mekân adımı

**Adım sırası:** Tür → Mekân → Zaman → Detay → Yayınla

### 3.1 Tür (önceki adım)
Seçime göre filtre varsayılanı:
- Sessiz coworking → Çalışma skoru ≥ 70
- Akşam kahve → Buluşma ≥ 70
- Sektör etkinliği → Etkinlik ≥ 60

### 3.2 Arama

| Alan | Limit |
|------|-------|
| Başlık | 30 | Mekân seç |
| Placeholder | 28 | Kafe, coworking ara… |
| Hint | 80 | Şehir: {city}. Google'dan foto ve saatler gelir. |

**Autocomplete:** debounce 300ms → `GET /places/autocomplete`

### 3.3 Seçim önizleme kartı
Liste kartı ile aynı + uyarı satırı:

| Uyarı | Limit | Örnek |
|-------|-------|--------|
| Kapanış | 50 | ⚠️ 21:30'da kapanıyor — saatini ayarla |
| Düşük skor | 60 | Bu mekân sessiz çalışmaya uygun görünmüyor |

### 3.4 Host mekân sorusu (ilk kez bu place_id)

Tek sheet, 3 soru — sonraki ziyaretlerde atla:

| Soru | Tip |
|------|-----|
| WiFi var mı? | Evet / Kısmen / Hayır |
| Gürültü | Sessiz / Orta / Gürültülü |
| Priz yeterli mi? | Evet / Hayır |

→ `venue_signals` upsert

---

## 4. Buluşma detay — Mekân hero

- Arka plan: mekân hero foto **blur 20 + overlay 0.5**
- Ön plan: buluşma başlığı **max 60 char**
- Alt: mekân adı tıklanabilir → Mekân detay
- Chip: tür + saat + katılımcı **4/6**

---

## 5. Keşif — Mekânlar sekmesi (v2)

### 5.1 Üst filtre chip
`Çalışmaya uygun` · `Akşam` · `Hafta sonu` · `Yakınımda` · `Sektörüm`

### 5.2 Harita modu
- Pin renk: yeşil çalışma / mavi buluşma / turuncu etkinlik
- Cluster: sayı badge
- Tap pin → mini kart (foto + ad + skor)

### 5.3 Liste modu
Mekân kartı + sıralama: Uzakiş popülerliği · Google puan · Mesafe

---

## 6. Uygunluk skoru — Gösterim kuralları

| Skor | Renk | Label |
|------|------|-------|
| 80–100 | Yeşil | Çok uygun |
| 60–79 | Sarı | Uygun |
| 40–59 | Turuncu | Orta |
| 0–39 | Gri | Az veri |

**Az veri:** < 3 sinyal → "Henüz yetersiz — ilk buluşmada puanla"

### Hesaplama (özet)
```
work_score = f(wifi, noise, outlets, work_votes)
meet_score = f(noise_inverse, seating, meet_votes)
event_score = f(capacity_hint, group_table, event_votes)
```

Ağırlıklar buluşma türüne göre feed sıralamasında kullanılır.

---

## 7. Anket → Mekân güncelleme

Buluşma anketi ek soruları:

| Soru | Tip |
|------|-----|
| Mekân çalışmaya uygun muydu? | 1–5 |
| Mekân buluşmaya uygun muydu? | 1–5 |
| Etiket | WiFi sorunu · Çok gürültülü · Harika mekân |

→ `venue_signals` + `meetup_feedback` aggregate

---

## 8. Erişilebilirlik & i18n

- Tüm foto `accessibilityLabel`: "{name} foto {n}/{total}"
- Skorlar: VoiceOver "Çalışma uygunluğu yüzde 82"
- TR/EN: [ONBOARDING.md](./ONBOARDING.md) locale anahtarları `venue.*`

---

## 9. Tasarım token (öneri)

| Token | Light | Dark |
|-------|-------|------|
| backgroundPrimary | #FAF8F5 | #121212 |
| brandCoral | #E85D4C | #E85D4C |
| brandDeep | #1B4D5C | #2A6B7C |
| cardRadius | 16pt | 16pt |
| heroGradient | top clear → bottom rgba(0,0,0,0.6) | aynı |
