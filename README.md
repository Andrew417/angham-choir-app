Here is the rewritten documentation as a **project description for AI agents** – a concise, structured spec that gives an AI all the information needed to continue development, deployment, or maintenance.

---

# Project: Choir Song Manager (Angham St. Mark’s Choir)

## Purpose
A web app that centralises song data (scale, BPM, structure, lyrics, audio tracks) for a choir, replacing scattered WhatsApp messages. Members can search and view songs; admins can add/edit/delete songs and upload audio. Data must persist across sessions.

## Tech Stack
- **Frontend**: Single HTML/CSS/JS file (self‑contained, no build step)
- **Backend / DB**: Supabase (PostgreSQL + Storage)
- **Deployment**: Vercel (static hosting)
- **Version control**: GitHub (public repo)

## Repository & URLs
- **GitHub repo**: `https://github.com/Andrew417/angham-choir-app.git`
- **Vercel deployment URL**: `https://angham-stmarks-choir.vercel.app` (or custom name)
- **Local path**: `D:\projects\Angham_choir_App\` → file must be named `index.html`

## Supabase Configuration (Critical)

### Credentials (must be embedded in `index.html`)
```javascript
const SUPABASE_URL = 'https://buomipedlddbrutjxefm.supabase.co';  // from user's project
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImJ1b21pcGVkbGRkYnJ1dGp4ZWZtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Nzk3MTYzODksImV4cCI6MjA5NTI5MjM4OX0.OiE5lLQ5N939H2dlgZQIvPjjgQg7UQUwS-mcHwVn2fw';
```

### Database Schema (already applied)
```sql
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

create table audio_tracks (
  id uuid default gen_random_uuid() primary key,
  song_id uuid references songs(id) on delete cascade,
  name text,
  url text,
  created_at timestamp default now()
);

-- Row Level Security policies (public read/write for simplicity)
alter table songs enable row level security;
alter table audio_tracks enable row level security;
create policy "public read" on songs for select using (true);
create policy "public write" on songs for all using (true);
create policy "public read" on audio_tracks for select using (true);
create policy "public write" on audio_tracks for all using (true);
```

### Storage Bucket
- **Name**: `audio`
- **Visibility**: Public
- **Purpose**: Store uploaded audio files (MP3, etc.)

## App Features & Behavior

### Library Tab (public)
- Search songs by `name` or `key` (scale)
- Display list of songs; click to see details: key, BPM, rhythm, structure, lyrics, notes, audio tracks
- Built‑in audio player for each track (no download required)

### Admin Tab (PIN‑protected)
- Default PIN: `1234` (stored in browser’s localStorage; resets to 1234 if cleared)
- After PIN entry: add song (form with all fields), edit song, delete song
- Upload multiple audio files per song (stored in Supabase `audio` bucket)
- Audio files are linked via `url` column in `audio_tracks` table

## Git & Deployment Steps (for AI to execute)

1. **Initialize local repo** (if not already):
   ```powershell
   cd D:\projects\Angham_choir_App
   git init
   ```

2. **Add remote origin**:
   ```powershell
   git remote add origin https://github.com/Andrew417/angham-choir-app.git
   git branch -M main
   ```

3. **Ensure `index.html` contains correct Supabase credentials** (replace placeholders with actual values above).

4. **Commit and push**:
   ```powershell
   git add .
   git commit -m "Initial commit - Choir Song Manager"
   git push -u origin main
   ```

5. **Deploy to Vercel**:
   - Log into Vercel (via GitHub OAuth)
   - Import the GitHub repo `angham-choir-app`
   - Framework preset: "Other" (static)
   - No build command needed
   - Deploy → obtain public URL

## Known Limitations & Constraints
- Audio files should be <5 MB each (free tier storage 1 GB)
- PIN is not secure; only meant to deter casual edits
- No user accounts – read access is public
- All data is in Supabase; backup via Supabase dashboard export

## Current Status (as of last user message)
- Supabase project created with tables and public `audio` bucket.
- Local folder exists: `D:\projects\Angham_choir_App\`
- `git init` has been run, but remote connection not yet completed (previous `git remote add` failed because `.git` was missing – now fixed).
- `index.html` may or may not exist locally. If missing, AI must regenerate the full app code (the complete self‑contained HTML/JS/CSS that uses the above Supabase credentials).

## Next Actions for AI Agent
1. If `index.html` is missing, generate the complete app code (based on earlier build) and instruct user to save it as `D:\projects\Angham_choir_App\index.html`.
2. Then execute the Git commands above to push to GitHub.
3. Guide user through Vercel deployment (or automate via Vercel CLI if possible).
4. After deployment, provide the public URL to the user.

## Additional Notes for AI
- The app uses the **Supabase JS client** (v2). Include it via CDN.
- The audio player should be an `<audio>` element with `controls` and dynamic `src` from the stored URL.
- The interface must be mobile‑friendly (choir members use phones).
- All data fetching and mutations use the Supabase client with the provided anon key.

---

**This description is complete. An AI agent can now continue the work without asking for missing context.**