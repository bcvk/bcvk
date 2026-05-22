# Uzakiş — Supabase Veritabanı Şeması

PostgreSQL + RLS. `auth.users` ile `profiles.id` eşleşir.

---

## Enum tipleri

```sql
CREATE TYPE work_type AS ENUM ('remote', 'freelance', 'hybrid');
CREATE TYPE meetup_type AS ENUM (
  'lunch', 'evening', 'weekend', 'coworking_silent',
  'walk', 'sector_networking', 'skill_swap', 'event'
);
CREATE TYPE meetup_status AS ENUM ('scheduled', 'cancelled', 'completed');
CREATE TYPE noise_level AS ENUM ('quiet', 'moderate', 'loud');
CREATE TYPE visibility AS ENUM ('public', 'sector_only', 'connections_only');
CREATE TYPE report_reason AS ENUM (
  'flirt_pressure', 'harassment', 'spam', 'fake_profile', 'unsafe_venue', 'other'
);
CREATE TYPE connection_status AS ENUM ('pending', 'accepted', 'blocked');
```

---

## profiles

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username TEXT UNIQUE,
  display_name TEXT NOT NULL,
  avatar_url TEXT,
  cover_color TEXT DEFAULT '#1B4D5C',
  bio TEXT CHECK (char_length(bio) <= 160),
  city TEXT NOT NULL,
  district TEXT,
  work_type work_type NOT NULL,
  primary_sector TEXT NOT NULL,
  secondary_sector TEXT,
  locale TEXT NOT NULL DEFAULT 'tr' CHECK (locale IN ('tr', 'en')),
  availability_weekday_lunch BOOLEAN DEFAULT true,
  availability_weekday_evening BOOLEAN DEFAULT false,
  availability_weekend BOOLEAN DEFAULT false,
  availability_flexible BOOLEAN DEFAULT false,
  looking_for TEXT[] DEFAULT '{}',
  interests TEXT[] DEFAULT '{}',
  spotify_connected BOOLEAN DEFAULT false,
  github_url TEXT,
  linkedin_url TEXT,
  future_city TEXT,
  future_city_month DATE,
  profile_visibility visibility DEFAULT 'public',
  dm_policy TEXT DEFAULT 'connections_only' CHECK (dm_policy IN ('meetup_only', 'connections_only', 'sector')),
  trust_score INT DEFAULT 0,
  meetups_hosted INT DEFAULT 0,
  meetups_attended INT DEFAULT 0,
  onboarding_completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_profiles_city ON profiles(city);
CREATE INDEX idx_profiles_sector ON profiles(primary_sector);
```

---

## community_pledges

```sql
CREATE TABLE community_pledges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  pledge_version TEXT NOT NULL DEFAULT '2026-05-v1',
  signed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  signature_method TEXT NOT NULL CHECK (signature_method IN ('long_press', 'button_confirm')),
  locale TEXT NOT NULL,
  UNIQUE (user_id, pledge_version)
);
```

---

## venues

Google Places ile senkron. `place_id` = Google place id.

```sql
CREATE TABLE venues (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  google_place_id TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  formatted_address TEXT,
  city TEXT NOT NULL,
  district TEXT,
  lat DOUBLE PRECISION NOT NULL,
  lng DOUBLE PRECISION NOT NULL,
  google_rating NUMERIC(2,1),
  google_ratings_total INT,
  price_level INT CHECK (price_level BETWEEN 0 AND 4),
  business_status TEXT,
  opening_hours JSONB,
  primary_photo_ref TEXT,
  photo_refs JSONB DEFAULT '[]',
  types TEXT[] DEFAULT '{}',
  maps_url TEXT,
  last_synced_at TIMESTAMPTZ DEFAULT now(),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_venues_city ON venues(city);
CREATE INDEX idx_venues_geo ON venues USING gist (
  ll_to_earth(lat, lng)
);
```

---

## venue_scores

Topluluk + host sinyalleri ile hesaplanan skorlar.

```sql
CREATE TABLE venue_scores (
  venue_id UUID PRIMARY KEY REFERENCES venues(id) ON DELETE CASCADE,
  work_score SMALLINT CHECK (work_score BETWEEN 0 AND 100),
  meet_score SMALLINT CHECK (meet_score BETWEEN 0 AND 100),
  event_score SMALLINT CHECK (event_score BETWEEN 0 AND 100),
  signal_count INT DEFAULT 0,
  last_computed_at TIMESTAMPTZ DEFAULT now()
);
```

---

## venue_signals

Tekil oy / host bildirimi.

```sql
CREATE TABLE venue_signals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  venue_id UUID NOT NULL REFERENCES venues(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  meetup_id UUID,
  has_wifi BOOLEAN,
  noise_level noise_level,
  has_outlets BOOLEAN,
  good_for_work BOOLEAN,
  good_for_meet BOOLEAN,
  good_for_event BOOLEAN,
  note TEXT CHECK (char_length(note) <= 100),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_venue_signals_venue ON venue_signals(venue_id);
```

---

## saved_venues

```sql
CREATE TABLE saved_venues (
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  venue_id UUID REFERENCES venues(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT now(),
  PRIMARY KEY (user_id, venue_id)
);
```

---

## meetups

```sql
CREATE TABLE meetups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  host_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  venue_id UUID REFERENCES venues(id) ON DELETE SET NULL,
  venue_name_fallback TEXT,
  title TEXT NOT NULL CHECK (char_length(title) <= 60),
  description TEXT CHECK (char_length(description) <= 300),
  meetup_type meetup_type NOT NULL,
  sector_filter TEXT,
  city TEXT NOT NULL,
  district TEXT,
  starts_at TIMESTAMPTZ NOT NULL,
  ends_at TIMESTAMPTZ,
  max_participants INT NOT NULL DEFAULT 6 CHECK (max_participants BETWEEN 2 AND 20),
  visibility visibility DEFAULT 'public',
  no_pitch BOOLEAN DEFAULT true,
  status meetup_status DEFAULT 'scheduled',
  participant_count INT DEFAULT 1,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_meetups_city_starts ON meetups(city, starts_at) WHERE status = 'scheduled';
CREATE INDEX idx_meetups_host ON meetups(host_id);
```

---

## meetup_participants

```sql
CREATE TABLE meetup_participants (
  meetup_id UUID REFERENCES meetups(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'participant' CHECK (role IN ('host', 'participant')),
  joined_at TIMESTAMPTZ DEFAULT now(),
  checked_in_at TIMESTAMPTZ,
  PRIMARY KEY (meetup_id, user_id)
);
```

---

## meetup_feedback

```sql
CREATE TABLE meetup_feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meetup_id UUID NOT NULL REFERENCES meetups(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  rating SMALLINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  tags TEXT[] DEFAULT '{}',
  note TEXT CHECK (char_length(note) <= 140),
  venue_work_rating SMALLINT CHECK (venue_work_rating BETWEEN 1 AND 5),
  venue_meet_rating SMALLINT CHECK (venue_meet_rating BETWEEN 1 AND 5),
  felt_unsafe BOOLEAN DEFAULT false,
  would_meet_again BOOLEAN,
  created_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE (meetup_id, user_id)
);
```

---

## connections

```sql
CREATE TABLE connections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  requester_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  addressee_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  status connection_status DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE (requester_id, addressee_id),
  CHECK (requester_id <> addressee_id)
);
```

---

## conversations & messages

```sql
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meetup_id UUID REFERENCES meetups(id) ON DELETE CASCADE,
  is_dm BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE conversation_members (
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  last_read_at TIMESTAMPTZ,
  PRIMARY KEY (conversation_id, user_id)
);

CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  sender_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  body TEXT NOT NULL CHECK (char_length(body) <= 2000),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_messages_conversation ON messages(conversation_id, created_at DESC);
```

---

## reports

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  reported_user_id UUID REFERENCES profiles(id) ON DELETE SET NULL,
  meetup_id UUID REFERENCES meetups(id) ON DELETE SET NULL,
  message_id UUID REFERENCES messages(id) ON DELETE SET NULL,
  reason report_reason NOT NULL,
  detail TEXT,
  status TEXT DEFAULT 'open' CHECK (status IN ('open', 'reviewing', 'resolved', 'dismissed')),
  created_at TIMESTAMPTZ DEFAULT now()
);
```

---

## trust_badges (v2)

```sql
CREATE TABLE trust_badges (
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  badge_type TEXT NOT NULL CHECK (badge_type IN ('trusted_host', 'trusted_guest', 'first_table')),
  earned_at TIMESTAMPTZ DEFAULT now(),
  PRIMARY KEY (user_id, badge_type)
);
```

---

## RLS özeti

| Tablo | Okuma | Yazma |
|-------|-------|-------|
| profiles | public alanlar herkese | sadece own |
| meetups | public + şehir | host create/update |
| meetup_participants | meetup görünürlüğü | self join/leave |
| messages | conversation member | member insert |
| venues | herkes | sadece service role sync |
| venue_signals | herkes | authenticated insert |
| reports | sadece admin | reporter insert |

Edge Functions: `places-*` service role ile `venues` upsert.

---

## Skor güncelleme (cron veya trigger)

`recompute_venue_scores(venue_id)`:
- Son 90 gün `venue_signals` + `meetup_feedback` aggregate
- Gürültü: quiet=+work, loud=-work,+meet (config)

---

## Realtime

- `messages` → conversation subscribe
- `meetup_participants` → katılımcı sayısı live
