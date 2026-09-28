# <div align="center"><img src="https://raw.githubusercontent.com/ots-downloader/onthespot/main/assets/logos/onthespot_icon.png" width="64" height="64" alt="OnTheSpot Logo" /><br/>OnTheSpot</div>

<div align="center">

[![GitHub Stars](https://img.shields.io/github/stars/ots-downloader/onthespot?style=for-the-badge&label=Stars&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/stargazers)
[![GitHub Forks](https://img.shields.io/github/forks/ots-downloader/onthespot?style=for-the-badge&label=Forks&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/network)
[![GitHub Issues](https://img.shields.io/github/issues/ots-downloader/onthespot?style=for-the-badge&label=Issues&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/issues)
[![GitHub License](https://img.shields.io/github/license/justin025/onthespot?style=for-the-badge&label=License&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/blob/main/LICENSE)
[![Python Version](https://img.shields.io/pypi/pyversion/onthespot?style=for-the-badge&label=Python%203.12%2B&labelColor=001224&color=ff6b6b)](https://www.python.org/download/releases/3.12/)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/ots-downloader/onthespot/ci.yml?style=for-the-badge&label=CI%20Status&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/actions)
[![CodeFactor](https://img.shields.io/codefactor/gh/ots-downloader/onthespot?style=for-the-badge&label=Code%20Factor&labelColor=001224&color=1DB954)](https://www.codefactor.io/repository/github/ots-downloader/onthespot/)
[![Contributors](https://img.shields.io/github/contributors/ots-downloader/onthespot?style=for-the-badge&label=Contributors&labelColor=001224&color=1DB954)](https://github.com/ots-downloader/onthespot/graphs/contributors)

</div>

<br/>

<div align="center">

## 🎵 A modern, easy-to-use music downloader written in Python

**OnTheSpot** is a feature-rich music downloader that supports multiple streaming services, manages a local music library with metadata and cover art, and provides a beautiful web interface for control and monitoring.

| Category | Details |
|----------|---------|
| **🔊 Supported Services** | Spotify, Tidal, Apple Music, Deezer, Qobuz, SoundCloud, YouTube Music, Bandcamp, Crunchyroll |
| **⚙️ Workers** | ParsingWorker, DownloadWorker, RetryWorker, AccountPoolLoader (all threaded) |
| **🌐 Frontend** | React 19 + Vite + Tailwind CSS (served by FastAPI) |
| **🗂️ Library** | Indexed metadata, cover art, M3U playlists, search & filtering |
| **🔧 Configuration** | JSON config + encrypted credential store (Fernet) |
| **📦 Deployment** | Docker, Unraid, standalone Python, Companion for Spotify Connect |

</div>

<br/>

<div align="center">

## 🚀 Quick Start

### Docker Compose (recommended)

```bash
git clone --branch fastapi-dev --single-branch https://github.com/ots-downloader/onthespot.git
cd onthespot
cp .env.example .env
docker compose up -d --build
```

Open `http://127.0.0.1:6767`, or the mapped address of the Docker/Unraid host.

### Standalone (Python)

```bash
# Clone and install
git clone --branch fastapi-dev --single-branch https://github.com/ots-downloader/onthespot.git
cd onthespot
pip install -e api/
# Or: pip install -r api/requirements.txt

# Start the application
cd api
python -m onthespot.main
# Or: uvicorn src.onthespot.main:app --host 127.0.0.1 --port 8000
```

Open `http://127.0.0.1:8000` to access the web interface.

</div>

<br/>

<div align="center">

## 📦 Features

### Music Download & Services

| Feature | Description |
|---------|-------------|
| **Multi-service search** | Search across Spotify, Tidal, Apple Music, Deezer, Qobuz, SoundCloud, YouTube Music, Bandcamp, Crunchyroll |
| **Download profiles** | Configure format, bitrate, download path per profile |
| **Path formatters** | Configurable file organization (Tracks, Albums, Movies, Shows, Podcasts) |
| **M3U playlist generation** | Auto-generate playlists with embedded metadata |
| **Video support** | Crunchyroll video downloads with chapters and subtitles |
| **YouTube Music** | Audio extraction with browser/cookie authentication |

### Library Management

| Feature | Description |
|---------|-------------|
| **Local library indexing** | Scan and index music files with full metadata |
| **Metadata editing** | Edit ID3/Vorbis/MP4 tags via ffmpeg |
| **Cover art** | Download and embed album artwork |
| **Duplicate detection** | Find and manage duplicate files |
| **File verification** | Verify downloaded file integrity |
| **Backup/export** | Export settings, queue, library, and history |

### Web Interface

| Feature | Description |
|---------|-------------|
| **Real-time queue** | SSE connections for live status updates |
| **Download controls** | Pause, resume, retry, cancel, delete, reorder, batch actions |
| **Settings page** | Full configuration of all options |
| **Diagnostics** | System health, rate limits, disk usage, worker status |
| **Log viewer** | Retrieve and download application logs |
| **Statistics** | Download history, success rates, format analytics |

### Spotify Connect Companion

| Feature | Description |
|---------|-------------|
| **ZeroConf discovery** | LAN-based Spotify Connect without Tailscale extension |
| **Pairing flow** | Short-lived credential payload to remote server |
| **Tailscale support** | Works across NATs and remote networks |
| **One-shot cleanup** | Auto-remove temporary companion folder after pairing |

</div>

<br/>

<div align="center">

## 🏗️ Architecture

```mermaid
graph TD
    subgraph "Frontend (React + Vite)"
        A[Web UI] -->|SSE| B[FastAPI /sse/{user_id}]
    end
    
    subgraph "FastAPI Backend"
        B -->|API Routes| C[main.py]
        C -->|Workers| D[ParsingWorker]
        C -->|Workers| E[DownloadWorker]
        C -->|Workers| F[RetryWorker]
        C -->|Workers| G[AccountPoolLoader]
    end
    
    subgraph "Shared State"
        D -->|parsing queue| H[runtimedata.py]
        E -->|download queue| H
        F -->|retry queue| H
        G -->|account pool| H
    end
    
    subgraph "Service Integrations"
        H -->|yt-dlp| I[services_middleware.py]
        I -->|librespot| J[Spotify]
        I -->|yt-dlp| K[Tidal/YouTube Music/SoundCloud]
        I -->|custom| L[Apple Music/Deezer/Qobuz/Bandcamp/Crunchyroll]
    end
    
    subgraph "Persistent Storage"
        H -->|JSON config| M[otsconfig.json]
        H -->|encrypted| N[credentials.enc]
        H -->|history| O[download-history.json]
        H -->|library index| P[.onthespot-library.json]
    end
```

</div>

<br/>

<div align="center">

## 📡 API Overview

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/profiles` | GET/POST | Manage download profiles |
| `/queue/downloads` | GET | Query download queue |
| `/queue/downloads/state` | GET | Download state & speed |
| `/queue/pending` | GET | Query pending queue |
| `/queue/parsing` | GET | Query parsing queue |
| `/config/get` | GET | Get current configuration |
| `/config/set` | PATCH | Update configuration settings |
| `/config/save` | POST | Save configuration |
| `/config/export` | GET | Export configuration |
| `/config/import` | POST | Import configuration |
| `/config/reset` | POST | Reset to defaults |
| `/accounts/youtube-auth/status` | GET | YouTube auth status |
| `/accounts/add` | POST | Add account for service |
| `/accounts/remove` | POST | Remove account |
| `/accounts/get` | GET | Get all accounts |
| `/accounts/health` | GET | Account health check |
| `/backup/export` | GET | Export backup (settings + queue + library) |
| `/diagnostics` | GET | System diagnostics |
| `/logs` | GET | Retrieve logs |
| `/api/sse/{user_id}` | GET | Server-Sent Events |

### Search & Queue

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/search` | POST | Search all configured services |
| `/query/url` | POST | URL-based search |
| `/queue/downloads/action` | POST | Action on specific queue item |
| `/queue/downloads/batch` | POST | Batch action on queue items |
| `/queue/downloads/verify` | POST | Verify download integrity |
| `/queue/pending/action` | POST | Action on pending item |

</div>

<br/>

<div align="center">

## 📚 Documentation

| Guide | Description |
|-------|-------------|
| [Installation Guide](docs/INSTALLATION.md) | Setup, Docker, dependencies, initial configuration |
| [Usage Guide](docs/USAGE.md) | Basic operations, download profiles, library management |
| [Feature Matrix](docs/FEATURE_MATRIX.md) | Complete feature overview per service |
| [Spotify Companion](companion/README.md) | ZeroConf Spotify Connect pairing |
| [API Reference](api/src/onthespot/main.py) | All FastAPI endpoints |

</div>

<br/>

<div align="center">

## 🛠️ Technology Stack

| Layer | Technology |
|-------|------------|
| **Backend** | FastAPI (Python 3.12+), yt-dlp, librespot, cryptography, ffmpeg |
| **Configuration** | JSON config + Fernet-encrypted credential store |
| **Frontend** | React 19, Vite, Tailwind CSS, lucide-react icons |
| **Container** | Docker, Docker Compose, Unraid |
| **Authentication** | OAuth2, browser cookies, API tokens |
| **Rate Limiting** | Per-host request locks with shared cooldown |
| **Caching** | Disk-based HTTP response cache with TTL |
| **Logging** | Rotating file handlers + stdout, structured JSON |

</div>

<br/>

<div align="center">

## 👥 Contributing

### Ways to Help

- **🐛 Report bugs** — Open an issue with reproduction steps
- **💡 Request features** — Describe the desired functionality
- **🔧 Submit PRs** — Fix bugs, add features, improve documentation
- **🌐 Translate** — Help translate the UI to new languages
- **📊 Test** — Report on new platforms, test beta features
- **💬 Spread the word** — Star the repo, share with friends

### Development Setup

```bash
# 1. Fork and clone
git clone --branch fastapi-dev --single-branch https://github.com/ots-downloader/onthespot.git
cd onthespot

# 2. Install dependencies
cd api
pip install -e .
cd ../ui
npm install

# 3. Start development
# Terminal 1: FastAPI with auto-reload
cd api
python -m onthespot.main

# Terminal 2: React dev server
cd ui
npm run dev
# UI will be at http://localhost:3000
# API will be at http://localhost:8000
```

### Coding Standards

- Follow [Black](https://black.readthedocs.io/) formatting (Python)
- TypeScript strict mode for frontend
- Add tests for new functionality
- Update documentation for any new features
- respect the encrypted credential store pattern

</div>

<br/>

<div align="center">

## 📜 License

[MIT](https://github.com/ots-downloader/onthespot/blob/main/LICENSE) © 2024 OnTheSpot Contributors

</div>

<br/>

<div align="center">

<!-- GitHub bottom section -->
<div style="display: inline-block; margin: 20px 0;">
  <a href="https://github.com/ots-downloader/onthespot/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=ots-downloader/onthespot" alt="contributors" />
  </a>
</div>

<div style="display: inline-block; margin: 20px 0;">
  <a href="https://github.com/ots-downloader/onthespot/stargazers">
    <img src="https://img.shields.io/github/stars/ots-downloader/onthespot?style=for-the-badge&label=Stars&labelColor=001224&color=1DB954" alt="Stars" />
  </a>
</div>
</div>

<br/>

<div align="center">
<p>Made with ❤️ by the OnTheSpot community</p>
<!-- GitHub star growth graph -->
![GitHub Star Growth](https://github-readme-stats.vercel.app/api?username=ots-downloader/onthespot&show_icons=true&theme=radical&count_private=false&include_all_commits=true&locale=en)
</div>
</div>

<!-- Badge references -->
[issues-shield]: https://img.shields.io/github/issues/ots-downloader/onthespot?style=flat&label=Issues&labelColor=001224&color=1DB954
[issues-url]: https://github.com/ots-downloader/onthespot/issues
[stars-shield]: https://img.shields.io/github/stars/ots-downloader/onthespot?style=flat&label=Stars&labelColor=001224&color=1DB954
[stars-url]: https://github.com/ots-downloader/onthespot/stargazers
[license-shield]: https://img.shields.io/github/license/justin025/onthespot?style=flat&label=License&labelColor=001224&color=1DB954
[downloads-shield]: https://img.shields.io/github/downloads/ots-downloader/onthespot/total.svg?style=flat&label=Downloads&labelColor=001224&color=1DB954
[downloads-url]: https://github.com/ots-downloader/onthespot/releases/