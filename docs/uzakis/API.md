# Uzakiş — API Taslağı

Supabase Edge Functions + REST. Tüm Google çağrıları **sunucu tarafı**.

Base: `https://{project}.supabase.co/functions/v1`

Auth: `Authorization: Bearer {supabase_jwt}`

---

## 1. Google Places Proxy

**Neden proxy:** API key gizliliği, cache, attribution kontrolü, rate limit.

Env:
```
GOOGLE_PLACES_API_KEY=...
PLACES_CACHE_TTL_SECONDS=86400
PHOTO_CACHE_TTL_SECONDS=604800
```

---

### `GET /places/autocomplete`

Şehir sınırlı mekân araması.

**Query:**
| Param | Zorunlu | Açıklama |
|-------|---------|----------|
| q | evet | Arama metni (min 2 char) |
| city | evet | Örn. Istanbul |
| lat | hayır | Oturum konumu |
| lng | hayır | |
| type | hayır | `cafe` (default), `coworking` |

**Response 200:**
```json
{
  "predictions": [
    {
      "place_id": "ChIJ...",
      "name": "Petra Roasting Co.",
      "structured_formatting": {
        "main_text": "Petra Roasting Co.",
        "secondary_text": "Caferağa, Kadıköy, İstanbul"
      }
    }
  ],
  "attribution": "Powered by Google"
}
```

**Cache key:** `autocomplete:{city}:{hash(q)}` TTL 1h

---

### `POST /places/details`

Mekân detay + DB upsert (`venues`).

**Body:**
```json
{
  "place_id": "ChIJ...",
  "city": "Istanbul"
}
```

**İşlem:**
1. Google Place Details (fields: id, displayName, formattedAddress, location, rating, userRatingCount, priceLevel, regularOpeningHours, photos, types, googleMapsUri, businessStatus)
2. Upsert `venues`
3. İlk `venue_scores` boş veya tip-tahmini
4. Return

**Response 200:**
```json
{
  "venue": {
    "id": "uuid",
    "google_place_id": "ChIJ...",
    "name": "Petra Roasting Co.",
    "formatted_address": "...",
    "city": "Istanbul",
    "district": "Kadıköy",
    "lat": 40.987,
    "lng": 29.024,
    "google_rating": 4.6,
    "google_ratings_total": 1240,
    "price_level": 2,
    "business_status": "OPERATIONAL",
    "opening_hours": { "weekday_text": ["..."] },
    "primary_photo_ref": "places/.../photos/...",
    "photo_refs": ["...", "..."],
    "maps_url": "https://maps.google.com/...",
    "scores": {
      "work_score": null,
      "meet_score": null,
      "event_score": null,
      "signal_count": 0
    }
  },
  "attribution": "Powered by Google"
}
```

---

### `GET /places/photo`

Foto proxy (key sunucuda).

**Query:**
| Param | Zorunlu |
|-------|---------|
| ref | evet (photo name / ref) |
| max_width | hayır (default 1200) |

**Response:** `302` → Google photo URL veya stream image  
**Cache:** CDN / R2 optional TTL 7d (Google ToS'e uygun)

---

### `GET /places/nearby`

Keşif sekmesi (v2).

**Query:**
| Param | Default |
|-------|---------|
| city | zorunlu |
| lat, lng | zorunlu |
| radius_m | 2000 |
| filter | `work` \| `meet` \| `event` |
| limit | 20 |

**Response:**
```json
{
  "venues": [ { "...venue card shape..." } ],
  "attribution": "Powered by Google"
}
```

**Sıralama:** `venue_scores.{filter}_score` DESC, tie-break Google rating

---

### `POST /places/signals`

Host / anket mekân sinyali.

**Body:**
```json
{
  "venue_id": "uuid",
  "meetup_id": "uuid?",
  "has_wifi": true,
  "noise_level": "quiet",
  "has_outlets": false,
  "good_for_work": true,
  "good_for_meet": true,
  "good_for_event": false,
  "note": "Priz az, WiFi hızlı"
}
```

**Response:** `{ "ok": true, "scores": { "work_score": 78, ... } }`

Trigger: `recompute_venue_scores(venue_id)`

---

## 2. Auth

Supabase Auth doğrudan:
- `signInWithOAuth` Apple / Google
- `signInWithOtp` magic link

**POST /auth/complete-onboarding** (opsiyonel wrapper)

**Body:** onboarding payload (city, work_type, sector, availability, looking_for, pledge)

---

## 3. Meetups

### `GET /meetups`

**Query:** `city`, `date?`, `type?`, `sector?`, `tab?` (today|evening|weekend|sector)

### `POST /meetups`

**Body:**
```json
{
  "title": "Cuma akşam kahve",
  "meetup_type": "evening",
  "venue_id": "uuid",
  "starts_at": "2026-05-30T19:00:00+03:00",
  "max_participants": 6,
  "visibility": "public",
  "no_pitch": true,
  "sector_filter": null
}
```

**Validasyon:** `starts_at` < venue `closing_time` → warning veya 400

### `POST /meetups/{id}/join`
### `DELETE /meetups/{id}/leave`

---

## 4. Feedback

### `POST /meetups/{id}/feedback`

```json
{
  "rating": 5,
  "tags": ["good_chat", "would_return"],
  "venue_work_rating": 4,
  "venue_meet_rating": 5,
  "would_meet_again": true,
  "felt_unsafe": false
}
```

---

## 5. Connections

### `POST /connections/request`
### `POST /connections/{id}/accept`
### `POST /connections/{id}/block`

DM: `conversations` create only if `connections.status = accepted` OR shared meetup participant.

---

## 6. Messages

### `GET /conversations/{id}/messages?cursor=`
### `POST /conversations/{id}/messages`

```json
{ "body": "Merhaba, 19:00'da görüşürüz." }
```

Realtime: Supabase channel `conversation:{id}`

---

## 7. Reports

### `POST /reports`

```json
{
  "reported_user_id": "uuid",
  "meetup_id": "uuid?",
  "reason": "flirt_pressure",
  "detail": "..."
}
```

---

## 8. Profiles

### `GET /profiles/me`
### `PATCH /profiles/me`
### `GET /profiles/{username}`

### `GET /match/suggestions` (v2)

Query: `city`, returns ranked profiles with `match_reason` string:
*"Aynı sektör, Cuma akşamı müsait, 2 ortak ilgi"*

---

## 9. Hata kodları

| Code | Anlam |
|------|--------|
| 401 | JWT yok/geçersiz |
| 403 | RLS / DM policy |
| 404 | Mekân/buluşma yok |
| 409 | Dolu / zaten katılımcı |
| 422 | Validasyon |
| 429 | Rate limit (places) |
| 502 | Google upstream |

---

## 10. iOS entegrasyon notları

| Özellik | Endpoint |
|---------|----------|
| Mekân ara | autocomplete |
| Detay + foto | details + photo |
| Masa aç | meetups POST with venue_id |
| Harita keşif | nearby |

**Deep link:** `uzakis://meetup/{id}`, `uzakis://venue/{id}`

**Magic link:** `uzakis://auth/callback`

---

## 11. Maliyet kontrolü

| Kural | Uygulama |
|-------|----------|
| Autocomplete debounce | client 300ms + server rate limit user 30/min |
| Details bir kez | upsert venues, sonra DB'den |
| Photo lazy | liste: primary only; detay: gallery |
| Şehir seed | admin job top 100 cafe/şehir |

---

## 12. Google ToS checklist

- [ ] "Powered by Google" UI
- [ ] Harita/foto attribution
- [ ] Cache süreleri dokümante
- [ ] `google_maps_uri` kullanıcı yönlendirme için
- [ ] Veri saklama: place_id + senkron tarih; foto ref expiry policy
