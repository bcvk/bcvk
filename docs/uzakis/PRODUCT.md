# Uzakiş — Ürün Özeti

## Tek cümle

Türkiye'de remote ve freelance çalışanların **iş dışı sosyal ağı**: öğle, akşam, hafta sonu buluşmaları; flört değil.

## Kim

- Remote (şirket) · Freelance · Hybrid
- 81 il (yoğunluk şehir bazlı; feed şehre göre)

## Çekirdek özellikler

| Özellik | Açıklama |
|---------|----------|
| Buluşma (etkinlik) | Oluştur / katıl; tür: öğle, akşam, hafta sonu, coworking, skill swap, sektör |
| Mekân (Google) | Fotoğraflı kartlar; çalışma / buluşma / etkinlik uygunluk |
| Mesajlaşma | Buluşma grubu; DM karşılıklı Bağlan sonrası |
| Yakınlık | Şehir + mahalle; opsiyonel konum |
| Raporlama | Profil, mesaj, buluşma |
| Eşleşme | Sektör + müsaitlik + ilgi (skor v2) |
| Profil | Sektör, ilgi, Spotify, GitHub/LinkedIn link, özelleştirme |
| Anket | Buluşma sonrası 1–5 + etiketler |
| i18n | Türkçe + English |
| Karanlık mod | Sistem + uygulama |

## Kaldırılan / yok

- AI stack (Claude / Cursor / Codex) profil göstergesi
- E-posta + şifre
- Dating (swipe, kim baktı, flirt modu)

## Onboarding

9 zorunlu ekran (~2 dk) → feed. Ekran 8: **basılı tut imza** (topluluk sözleşmesi). Detay: [ONBOARDING.md](./ONBOARDING.md)

## Mekân katmanı

Google Places + Uzakiş uygunluk skorları. Detay: [VENUES-UX.md](./VENUES-UX.md), [API.md](./API.md)

## Öne çıkan fikirler (v1.2+)

1. **Masa Daveti** — paylaşılabilir davet kartı
2. **Güven rozeti** — anket + rapor yok
3. **Sessiz Saat** — yan yana çalışma buluşması
4. **Skill Swap günü** — aylık ritüel
5. **Sektör masası** — sektör bazlı etkinlikler
6. **Tekrar buluşalım mı?** — anket sonrası
7. **Şehir değişimi** — gelecek ay şehir

## Fazlar

| Faz | Kapsam |
|-----|--------|
| MVP | Auth · Onboarding+imza · TR · Feed · Buluşma CRUD · Sektör profil · Rapor · Grup chat |
| v1.1 | EN · Karanlık mod · Buluşma anketi · DM (Bağlan) |
| v1.2 | Google mekân · Uygunluk skoru · Skill swap · Sektör sekmesi · Masa Daveti |
| v2 | Güven rozeti · Harita keşif · Topluluk mekân oyları · Akıllı eşleşme |
| v2+ | Coworking B2B · Düzenli masa · Spotify |

## Nomadtable farkı

| Nomadtable | Uzakiş |
|------------|--------|
| Solo travel | Yerleşik remote/freelance |
| Gezi, parti | Öğle, akşam, hafta sonu |
| Global traveler | Türkiye, TR/EN |
