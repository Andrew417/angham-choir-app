```markdown
# ChoirNotes — Angham St. Mark's Choir

A mobile-first web app that centralises song data for the choir — replacing scattered WhatsApp messages with a single, always-accessible reference for all members.


## Features

### Library (public, no login required)
- View all songs in the repertoire
- Search songs by **name** or **key/scale**
- Song count displayed on dashboard
- Tap any song to open the full detail view

### Song Detail
Sections appear in this fixed order:
1. Key / BPM / Rhythm chips
2. Structure
3. Chord Sheets
4. Audio Tracks
5. Lyrics
6. Notes

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
create table songs (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  key text,
  bpm integer,
  rhythm text,
  structure jsonb,
  lyrics text,
  notes text,
  created_at timestamp default now()
);

create table audio_tracks (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  sort_order integer default 0,
  created_at timestamp default now()
);

create table chord_sheets (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  sort_order integer default 0,
  created_at timestamp default now()
);
```

### Row Level Security (RLS)

```sql
alter table songs enable row level security;
alter table audio_tracks enable row level security;
alter table chord_sheets enable row level security;

create policy "public read"  on songs for select using (true);
create policy "public write" on songs for all using (true);

create policy "public read"  on audio_tracks for select using (true);
create policy "public write" on audio_tracks for all using (true);

create policy "public read"  on chord_sheets for select using (true);
create policy "public write" on chord_sheets for all using (true);
```

### Storage Buckets

| Bucket   | Visibility | Purpose                  |
| -------- | ---------- | ------------------------ |
| `audio`  | Public     | MP3/M4A/WAV audio tracks |
| `chords` | Public     | Images and PDFs          |

```sql
create policy "public upload audio" on storage.objects for insert to anon with check (bucket_id = 'audio');
create policy "public read audio"   on storage.objects for select to anon using (bucket_id = 'audio');
create policy "public delete audio" on storage.objects for delete to anon using (bucket_id = 'audio');

create policy "public write chords"  on storage.objects for insert with check (bucket_id = 'chords');
create policy "public read chords"   on storage.objects for select using (bucket_id = 'chords');
create policy "public delete chords" on storage.objects for delete using (bucket_id = 'chords');
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

### Screens

| Screen ID          | Description                         |
| ------------------ | ----------------------------------- |
| `screen-dashboard` | Song list, search, total count      |
| `screen-detail`    | Full song info, audio, chord sheets |

### Modals

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

## Feature Reference

### Structure Map
- Block types: Intro (teal), Verse (blue), Chorus (pink), Bridge (purple), Break (orange), Outro (grey)
- Tap a block type button to add it to the sequence
- Each block shows its repeat count (e.g. `Verse ×2`) — default is ×2
- Tap any block to reveal +/− buttons to adjust repeat count; tap elsewhere to close
- Drag and drop to reorder blocks (desktop + mobile touch)
- Stored as JSONB array of `{type, repeat}` objects e.g. `[{"type":"verse","repeat":2}]`

### Lyrics
- Single plain-text field — paste full lyrics directly
- Displayed with whitespace preserved in the detail view
- Stored as plain `text` in Supabase

### Audio Tracks
- Upload MP3, M4A, or WAV files to the `audio` storage bucket
- In-browser audio player on the detail screen
- Drag and drop to reorder
- Inline rename via edit icon
- Delete with confirmation

### Chord Sheets
- Upload JPG, PNG, or PDF files to the `chords` storage bucket
- Images preview inline; PDFs open in a new tab
- Drag and drop to reorder
- Inline rename via edit icon
- Delete with confirmation

**Vercel (free tier)**
- 100 GB bandwidth/month
- Deployments from `main` only trigger auto-deploy
- Static files only — no server-side code (fine for you, everything is client-side)
- Individual file size limit: **25 MB** — keep your `index.html` well under that

**Supabase (free tier)**
- **500 MB** database storage
- **1 GB** file storage (audio + chord sheets combined) — this is your main constraint
- **2 GB** bandwidth/month
- Project **pauses after 1 week of inactivity** — first load after pause is slow (~5–10s), then normal
- Max **50 MB per file upload**
- 500 simultaneous connections max

**Most likely issues you'll hit:**
1. Storage fills up from large audio files — keep tracks under 5 MB each
2. Project pauses if choir doesn't use it for a week — first person to open it will see a slow load, then it wakes up
3. No backups automatically — export manually from Supabase dashboard occasionally