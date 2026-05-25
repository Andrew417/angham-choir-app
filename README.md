# ChoirNotes — Angham St. Mark's Choir

A mobile-first web app that centralises song data for the choir — replacing scattered WhatsApp messages with a single, always-accessible reference for all members.

---

## Live URL

> **https://angham-stmarks-choir.vercel.app**

---

## Features

### Library (public, no login required)
- View all songs in the repertoire
- Search songs by **name** or **key/scale**
- Song count displayed on dashboard
- Tap any song to open the full detail view

### Song Detail
- Displays: Key, BPM, Rhythm, Structure, Lyrics, Director Notes
- **Audio Tracks** — upload and stream MP3/M4A/WAV files directly in-browser
- **Chord Sheets** — upload images (JPG/PNG) or PDFs; images preview inline

### Add / Edit / Delete Songs
- Tap **+** on the dashboard to add a new song
- Tap **edit** icon on any song detail to edit
- Tap **delete** icon to delete (with confirmation dialog)
- All changes persist instantly to Supabase

---

## Tech Stack

| Layer           | Technology                        |
| --------------- | --------------------------------- |
| Frontend        | Single `index.html` (HTML/CSS/JS) |
| Styling         | Tailwind CSS (CDN)                |
| Database        | Supabase (PostgreSQL)             |
| Storage         | Supabase Storage                  |
| Hosting         | Vercel (static)                   |
| Version Control | GitHub                            |

---

## Repository

```
https://github.com/Andrew417/angham-choir-app.git
```

Local path: `D:\projects\Angham_choir_App\index.html`

---

## Supabase Configuration

### Credentials (embedded in `index.html`)

```javascript
const SUPABASE_URL = 'https://buomipedlddbrutjxefm.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

### Database Schema

```sql
-- Songs
create table songs (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  key text,
  bpm integer,
  rhythm text,
  structure text,
  lyrics text,
  notes text,
  created_at timestamp default now()
);

-- Audio tracks (linked to songs)
create table audio_tracks (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  created_at timestamp default now()
);

-- Chord sheets (linked to songs)
create table chord_sheets (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  created_at timestamp default now()
);
```

### Row Level Security (RLS)

Applied to all three tables — public read and write (no auth required):

```sql
-- Repeat for audio_tracks and chord_sheets
alter table songs enable row level security;
create policy "public read"  on songs for select using (true);
create policy "public write" on songs for all    using (true);
```

### Storage Buckets

| Bucket   | Visibility | Purpose                  |
| -------- | ---------- | ------------------------ |
| `audio`  | Public     | MP3/M4A/WAV audio tracks |
| `chords` | Public     | Images and PDFs          |

Storage RLS policies (applied to both buckets):

```sql
-- Replace 'chords' with 'audio' and repeat
create policy "public write"  on storage.objects for insert using (bucket_id = 'chords');
create policy "public read"   on storage.objects for select using (bucket_id = 'chords');
create policy "public delete" on storage.objects for delete using (bucket_id = 'chords');
```

---

## Deployment

### Push to GitHub

```powershell
cd D:\projects\Angham_choir_App
git add .
git commit -m "your message"
git push
```

### Deploy to Vercel

1. Log into [vercel.com](https://vercel.com) via GitHub
2. Import repo `angham-choir-app`
3. Framework preset: **Other** (static site)
4. No build command needed
5. Deploy → get public URL

Vercel auto-deploys on every push to `main`.

---

## Project Structure

```
Angham_choir_App/
└── index.html       # Entire app — all screens, styles, and logic in one file
```

### Screens (inside `index.html`)

| Screen ID          | Description                         |
| ------------------ | ----------------------------------- |
| `screen-dashboard` | Song list, search, total count      |
| `screen-detail`    | Full song info, audio, chord sheets |

### Modals (overlays, no page change)

| Modal               | Triggered by                |
| ------------------- | --------------------------- |
| Add/Edit Song       | `+` button or Edit icon     |
| Upload Audio Track  | `+` in Audio Tracks section |
| Upload Chord Sheet  | `+` in Chord Sheets section |
| Delete Confirmation | Any delete action           |

---

## Known Limitations

- No user accounts — all access is public (read and write)
- No PIN protection on add/edit/delete (intentionally removed)
- Audio files should be kept under **5 MB** each (Supabase free tier: 1 GB total)
- Supabase anon key is exposed in client — acceptable for a private choir app with no sensitive data
- Backup via Supabase dashboard → Table Editor → Export

---

Here's the updated README section reflecting all current changes:

---

## Changelog

### Latest
- Full Supabase integration (songs, audio tracks, chord sheets)
- Single-file app (`index.html`) — no build step
- Dashboard with live search and song count
- Song detail screen with all metadata fields

**Structure Map**
- Visual colored block builder replacing plain-text structure field
- Block types: Intro (teal), Verse (blue), Chorus (pink), Bridge (purple), Break (orange), Outro (grey)
- Tap a block type to add it to the sequence
- Drag and drop to reorder blocks (desktop + mobile touch)
- Tap × on any block to remove it
- Structure stored as JSONB array in Supabase

**Lyrics**
- Structured sections replacing plain-text lyrics field
- Each section has a type dropdown (Verse, Chorus, Bridge, etc.) + lyrics textarea
- Section header automatically colored to match its block type
- Up/down arrows to reorder sections
- Lyrics stored as JSONB array of `{type, content}` objects in Supabase

**Audio & Chord Sheets**
- Audio upload to `audio` bucket + in-browser player
- Chord sheet upload to `chords` bucket + image preview inline
- Drag and drop to reorder tracks and sheets (desktop + mobile touch)
- Inline rename by tapping the edit icon
- Delete with confirmation dialog

**General**
- Add / Edit / Delete songs via bottom-sheet modal
- Toast notifications for all actions
- Storage RLS policies for both buckets (`audio` and `chords`)
- Mobile-first responsive design (max-width 768px)
- Removed unused Search and Settings nav items

### Schema changes required
```sql
-- Sort order for tracks and sheets
alter table audio_tracks add column sort_order integer default 0;
alter table chord_sheets  add column sort_order integer default 0;

-- Structure and lyrics converted to JSONB
alter table songs alter column structure type jsonb using null;
alter table songs alter column lyrics    type jsonb using null;

-- Chord sheets table (new)
create table chord_sheets (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  created_at timestamp default now()
);
alter table chord_sheets enable row level security;
create policy "public read"  on chord_sheets for select using (true);
create policy "public write" on chord_sheets for all    using (true);

-- Storage RLS for audio and chords buckets
create policy "public write"  on storage.objects for insert with check (bucket_id = 'chords');
create policy "public read"   on storage.objects for select using  (bucket_id = 'chords');
create policy "public delete" on storage.objects for delete using  (bucket_id = 'chords');
create policy "public write audio" on storage.objects for insert with check (bucket_id = 'audio');
create policy "public read audio"  on storage.objects for select using  (bucket_id = 'audio');
create policy "public delete audio" on storage.objects for delete using (bucket_id = 'audio');
```