# Uzakiş — Onboarding

## Prensipler

- İlk feed: **≤ 90 saniye** (auth hariç imza ~3 sn)
- Zorunlu: **9 ekran** (0–8)
- Ton: samimi, **sen** dili
- Ekran 8: **basılı tut imza** — atlanamaz (erişilebilirlik: Onayla butonu)

---

## Ekran 0 — Tanıma

| Alan | Limit | TR | EN |
|------|-------|----|----|
| Başlık | 60 | Evden çalışıyorsun. İnsan yüzü de lazım. | You work from home. You still need real people. |
| Gövde | 120 | Slack yetmez bazen. Uzakiş: öğle, akşam, hafta sonu — gerçek buluşma. | Sometimes Slack isn't enough. Uzakiş: lunch, evening, weekend — real meetups. |
| Alt | 40 | Flört değil. İş sunumu değil. | Not dating. Not a pitch fest. |
| CTA | 20 | Devam et | Continue |

---

## Ekran 1 — Değer

| Alan | Limit | TR |
|------|-------|-----|
| Başlık | 50 | Yalnız çalışmıyorsun — yalnız mola veriyorsun. |
| Kart 1 | 40 | Öğle kahvesi |
| Kart 2 | 40 | Akşam buluşması |
| Kart 3 | 40 | Hafta sonu |
| CTA | 24 | Tamam, anladım |

---

## Ekran 2 — Giriş

| Alan | Limit | TR |
|------|-------|-----|
| Başlık | 30 | Seni içeri alalım. |
| Gövde | 80 | Apple, Google veya e-posta — şifre yok. |
| Buton 1 | 24 | Apple ile devam et |
| Buton 2 | 24 | Google ile devam et |
| Buton 3 | 28 | E-posta ile magic link |
| Yasal | 100 | Kayıt olarak Kullanım ve KVKK'yı kabul edersin. |

**Magic link akışı:** e-posta → "Link gönderdik" → deep link → oturum.

---

## Ekran 3 — Dil

| Alan | Limit |
|------|-------|
| Seçenek | Türkçe · English |
| Not | 60 | Sonra ayarlardan değiştirebilirsin. |

`profiles.locale` = `tr` | `en`

---

## Ekran 4 — Şehir

| Alan | Limit | TR |
|------|-------|-----|
| Başlık | 30 | Şu an neredesin? |
| Gövde | 90 | Buluşmalar bu şehre göre listelenir. |
| Placeholder | 20 | Şehir ara |
| CTA | 20 | Buradayım |
| Progress | 24 | 4/9 · Bir dakikaya az kaldı |

---

## Ekran 5 — Çalışma tipi

| Seçenek | TR |
|---------|-----|
| A | Remote — bir şirkete bağlıyım |
| B | Freelance — proje proje |
| C | Hybrid — ikisi de |

---

## Ekran 6 — Sektör

Tek ana sektör (+ opsiyonel ikinci v2).

Yazılım · Tasarım · Pazarlama · Ürün · Müşteri desteği / Ops · Finans · Eğitim · İçerik · Hukuk · Diğer

---

## Ekran 7 — Ad + müsaitlik + arzu

### Ad
| Alan | Limit |
|------|-------|
| Başlık | 24 | Sana ne diyelim? |
| Alan | display_name (zorunlu) |

### Müsaitlik (chip, çoklu)
- Hafta içi öğle (12–14)
- Hafta içi akşam (18–21)
- Hafta sonu
- Fırsat buldukça

### Ne arıyorsun (max 4)
Kahve · Akşam · Hafta sonu · Sessiz coworking · Yürüyüş · Sektör networking · Yeni şehirde arkadaş · Düzenli masa

---

## Ekran 8 — İmza (basılı tut)

### Davranış
- `LongPressGesture` minimum **1.8 sn**
- Arkaplan: `progress 0→1` → `backgroundWarm` → `brandCoral` → `brandDeep`
- Haptic: %50, %100
- Bırakınca progress sıfır (tamamlanmadıysa)

### Metin (progress'e göre)
| progress | TR | EN |
|----------|----|----|
| 0 | Basılı tut… | Hold… |
| 0.5 | Neredeyse… | Almost… |
| 1 | Hoş geldin. | Welcome. |

### Üst (sabit)
| TR | EN |
|----|-----|
| Uzakiş topluluğuna katılıyorum | I'm joining the Uzakiş community |
| Flört yok. İş pitch yok. Saygı var. | No dating. No pitches. Respect. |

### Bitiş animasyonu
- İmza çizgisi + `{display_name}` + tarih
- `community_pledges` kaydı

### Erişilebilirlik
- Buton: **Onayla** (VoiceOver) — aynı kayıt

### CTA final
| TR | EN |
|----|-----|
| Uzakiş'e gir | Enter Uzakiş |

---

## Katman 2 (feed sonrası)

| % | Tetik | Alan |
|---|--------|------|
| 30 | Feed | Bildirim izni |
| 40 | Profil | İlgi alanları (min 3) |
| 50 | | Foto (opsiyonel) |
| 60 | | Bio 160 char |
| 70 | | Mahalle |
| 80 | | Spotify / GitHub / LinkedIn |
| 90 | | Kapak rengi, görünürlük |

---

## Feed iniş sheet

| Alan | Limit | TR |
|------|-------|-----|
| Başlık | 40 | Hoş geldin, {name}. |
| Gövde | 120 | {city}'de az buluşma olabilir — ilk masayı sen aç. |
| CTA 1 | 28 | Öğle kahvesi aç |
| CTA 2 | 24 | Önce etrafa bak |
