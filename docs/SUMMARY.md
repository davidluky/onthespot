# OnTheSpot Repository Summary

## Top-Level Structure
```
.api/        # Main FastAPI application (Python)
.companion/  # Spotify Connect ZeroConf companion
.docs/       # Documentation (INSTALLATION.md, USAGE.md)
.ui/         # React + Vite frontend
.api/src/    # Python source code
.companion/src/  # Companion source code
.ui/src/     # React source code
.compose.yml # Docker compose
.Dockerfile  # Docker build
```

## Project Purpose
**OnTheSpot** is an easy-to-use music downloader written in Python with a FastAPI backend and React frontend. It supports downloading from multiple music services and manages a local music library with metadata, cover art, and M3U playlists.

## API (Python Backend)

### Core Architecture
- **FastAPI** app with `lifespan` managing background workers
- **Configuration**: `otsconfig.py` — persistent JSON config + Fernet-encrypted credential store (`credentials.py`)
- **Shared state**: `runtimedata.py` — queues, pools, locks, logging infrastructure
- **Download workers**: `services_middleware.py` — handles all service integrations

### Supported Services (9)
- **Spotify**, **Tidal**, **Apple Music**, **Deezer**, **Qobuz**, **SoundCloud**, **YouTube Music**, **Bandcamp**, **Crunchyroll**

### Key Files
| File | Purpose |
|------|---------|
| `main.py` | FastAPI app with all ~30 API endpoints, lifespan, workers |
| `otsconfig.py` | Configuration manager with persistent JSON + encrypted creds |
| `runtimedata.py` | Shared mutable state (queues, pools, locks, logging) |
| `accounts.py` | Account pool management & token retrieval |
| `library.py` | Music library indexing, metadata, file operations |
| `services_middleware.py` | Download logic for all services (yt-dlp + librespot) |
| `utils.py` | Path formatting, caching, audio processing, thumbnails, M3U |
| `parse_item.py` | URL parsing & parsing worker thread |
| `statistics.py` | Download history & statistics persistence |
| `updater.py` | GitHub release checking & update notifications |
| `credentials.py` | Fernet encrypted credential store |
| `constants.py` | `ItemStatus` enum and HTTP timeout config |
| `resources/regexes.py` | URL pattern matching for all supported services |

### API Endpoints (key categories)
- **`/profiles`** — download profile management
- **`/queue/*`** — download queue actions (pause, resume, retry, cancel, delete, reorder, batch)
- **`/config/*`** — configuration get/set/save/export/import/reset
- **`/accounts/*`** — YouTube auth, account add/remove/get/health/reconnect
- **`/backup/*`** — backup export/import of settings, queue, library metadata
- **`/diagnostics`** — system health, rate limit state, disk usage
- **`/logs`** — log file retrieval
- **`/sse/{user_id}`** — Server-Sent Events for real-time queue updates
- **`/api/sse/{user_id}`** — SSE endpoint used by the React frontend

### Workers (threaded)
- **ParsingWorker** — drains URL parsing queue, fans items into pending
- **DownloadWorker** — processes items from download queue
- **RetryWorker** — retries failed/cancelled items (configurable)
- **AccountPoolLoader** — authenticates all active accounts on startup

### Credential Security
- Sensitive values (`accounts`, `spotify_webapi_override_client_secret`, `playlist_automation_client_secret`) stored in encrypted file `credentials.enc` beside config
- Key stored in `credentials.key` (file permissions 0600)
- Config JSON remains readable/hand-editable

### FFmpeg Integration
- Used for audio format conversion, metadata embedding, cover art embedding, video muxing
- User-defined `ffmpeg_args` configurable in settings

## Companion (Spotify Connect)

### Purpose
Runs on user's LAN to handle Spotify Connect ZeroConf discovery and forwarding of short-lived pairing payloads to the remote OnTheSpot API.

### Key Features
- Auto-detects LAN interface and chooses available port
- Validates server URL (rejects plain HTTP to remote hosts, supports Tailscale)
- Forward Spotify login blob to `/accounts/spotify/companion/complete`
- Cleanup of temporary companion folder after successful pairing
- One-shot pairing process

### Entry Point: `companion/run.py`
- Starts `ZeroconfServer` for Spotify Connect discovery
- Waits for valid session, reads login payload from state file
- POSTs to OnTheSpot API to complete pairing
- Supports `--allow-insecure`, `--cleanup`, `--state-file`, `--port`, `--interface`

## User Interface (React + Vite)

### Structure
- **`ui/src/components/`** — 15+ React components
  - `AccountsManager.tsx`, `DownloadQueue.tsx`, `SettingsPage.tsx`, `SearchDashboard.tsx`, `StatisticsPanel.tsx`, etc.
- **`ui/src/lib/`** — API client, catalog services, i18n, notifications
- **`ui/package.json`** — React 19, Vite, Tailwind CSS, lucide-react icons

### Key Features
- **SSE connections** via `/api/sse/{user_id}` for real-time queue updates
- **Dark theme** with Tailwind CSS
- **Internationalization** (i18n files: en_US, de_DE, ja_JP, pt_PT)
- **Download queue management** with per-item actions
- **Settings page** for all configuration options
- **Diagnostics panel** showing system health, rate limits, disk usage
- **Library browser** with search, filters, metadata editing
- **Notification system** with toast-like banners
- **SSE-powered status changes** propagate from worker threads to frontend

### Build
- `npm run dev` — Vite dev server on port 3000
- `npm run build` — Vite production build into `ui/dist`
- FastAPI serves `ui/dist` static files when `ONTHESPOT_WEBUI_DIST` is set

## Documentation
- `docs/INSTALLATION.md` — Setup and installation instructions
- `docs/USAGE.md` — How to use the application

## Technology Stack
- **Backend**: FastAPI (Python 3.12+), yt-dlp, librespot, cryptography, ffmpeg
- **Frontend**: React 19, Vite, Tailwind CSS, lucide-react icons
- **Container**: Docker-based deployment with `compose.yml`
- **Credentials**: Fernet encrypted store separate from config JSON
- **Threading**: Extensive use of threading with locks for shared state

## Key Design Decisions
1. **Config + Encrypted Credentials split**: Keeps config hand-editable while protecting secrets
2. **Thread-safe shared state**: All mutable global state in `runtimedata.py` with proper locks
3. **Per-host API rate limiting**: Single global rate-limit with per-host locks in `utils.py`
4. **Caching strategy**: Public catalogue metadata cached; account-specific data never persisted
5. **SSE for real-time updates**: Rather than WebSockets or polling for queue status changes
6. **Modular service integrations**: Each service has its own module in `api/` with consistent interfaces
7. **Path formatters**: Configurable path templates for organizing downloaded files
8. **Library index**: Separate `.onthespot-library.json` index on top of actual music files for searchability
