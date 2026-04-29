# 📤 Telegram → GitHub File Upload Bot

<div align="center">

[![Persian](https://img.shields.io/badge/_Persian-README.fa.md-blue?style=for-the-badge)](README.fa.md) &nbsp; [![💰 Donate](https://img.shields.io/badge/💰_Support_the_Project-Donate-orange?style=for-the-badge)](#-support-the-project--donate)

</div>

> نسخه فارسی این راهنما موجود است: [README.fa.md](README_fa.md)

> **Why this was built** — 28 April 2025
>
> In Iran, frequent internet disruptions and the high cost of VPNs make accessing
> personal files unreliable. GitHub is one of the few services that often remains
> accessible without a VPN. This bot lets you push large files from Telegram
> (videos, audio, documents, …) directly into a GitHub repository, so you can
> download them from GitHub anytime — no VPN needed.
>
> All you need is a cheap hourly-billed VPS with a minimum 10 GB SSD, and this bot.

---

## Features

- **All file types** — documents, photos, videos, audio, voice, stickers, animations, video notes
- **Smart routing** — files under 50 MB go through the Contents API; larger files are stored via **Git LFS** automatically
- **Auto-cleanup** — when the repo exceeds a configured size limit, the oldest files are deleted to keep VPS disk usage low
- **Access control** — whitelist of allowed Telegram user IDs
- **Conflict resolution** — overwrite existing files or auto-version them (`file_1.ext`, `file_2.ext`, …)
- **Custom upload paths** — per-user `/setpath` command
- **Retry logic** — exponential back-off on GitHub secondary rate limits
- **Admin notifications** — errors forwarded to a configured admin user
- **Rotating log file** — 10 MB × 5 backups

---

## Project Structure

```
telegram-github-uploader/
├── bot/
│   ├── __init__.py
│   ├── __main__.py            # Entry point – run with `python -m bot`
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py        # All env-var configuration
│   │   └── logging_config.py  # Root logger setup
│   ├── handlers/
│   │   ├── __init__.py
│   │   ├── commands.py        # /start /help /setpath /clearpath /status
│   │   └── upload.py          # Main file-upload message handler
│   └── services/
│       ├── __init__.py
│       ├── file_service.py    # Telegram file extraction & download
│       └── github_service.py  # GitHub Contents API + Git LFS wrapper
├── logs/                      # Auto-created at runtime
├── .env                       # Your secrets (never commit this)
├── .env.example               # Template – copy and fill in
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Quick Start

### ⚡ One-line setup (recommended)

No cloning required. Run this on your VPS and the wizard handles everything:

```bash
bash <(curl -sL https://raw.githubusercontent.com/GonnaTox/tg-github-uploader/main/setup.sh)
```

The script will automatically:
1. Run `apt update` and install all required system packages
2. Clone this repository
3. Create a Python virtual environment and install dependencies
4. Walk you through every config value and write `.env`
5. Optionally configure Git LFS in your repository
6. Optionally install a `systemd` service so the bot survives reboots

> **Prerequisite:** `sudo` access on the VPS and an active internet connection.

---

### 🛠️ Manual setup

#### 1. Clone & install

```bash
sudo apt update
sudo apt install build-essential python3 python3-dev python3-venv python3-pip git git-lfs curl -y

git clone https://github.com/GonnaTox/tg-github-uploader
cd GitHub-uploader

python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

#### 2. Configure

```bash
cp .env.example .env
nano .env    # fill in the required values
```

#### 3. Enable Git LFS in your repository (for large files)

If you want to store files larger than 50 MB, you need to initialise Git LFS in your repo once — do this on your **local machine**, not the VPS:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

git lfs install
git lfs track "*"         # track all file types under LFS
git add .gitattributes
git commit -m "chore: enable Git LFS for all files"
git push
```

Install Git LFS if needed:
- Ubuntu/Debian: `sudo apt install git-lfs`
- macOS: `brew install git-lfs`
- Windows: download from [git-lfs.com](https://git-lfs.com)

> If you only upload files under 50 MB, this step is optional.

#### 4. Run the bot

```bash
screen -S tel2github          # keep the bot running in the background
source .venv/bin/activate
python -m bot
```

Detach from screen without stopping the bot: `Ctrl+A` then `D`.

---

## Environment Variables

#### Required

| Variable | Description |
|---|---|
| `TELEGRAM_BOT_TOKEN` | From [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_API_ID` | From [my.telegram.org](https://my.telegram.org) |
| `TELEGRAM_API_HASH` | From [my.telegram.org](https://my.telegram.org) |
| `GITHUB_TOKEN` | PAT with `repo` scope |
| `GITHUB_OWNER` | Your GitHub username or org |
| `GITHUB_REPO` | Target repository name |

#### Optional

| Variable | Default | Description |
|---|---|---|
| `GITHUB_BRANCH` | `main` | Target branch |
| `UPLOAD_BASE_PATH` | `uploads` | Root folder inside the repo |
| `FILE_CONFLICT_STRATEGY` | `version` | `overwrite` or `version` |
| `LFS_THRESHOLD_MB` | `50` | Files larger than this (MB) are routed through Git LFS |
| `CLEANUP_ENABLED` | `false` | Enable automatic deletion of old files |
| `CLEANUP_MAX_REPO_MB` | `2048` | Trigger cleanup when uploaded files exceed this size (MB) |
| `CLEANUP_KEEP_LATEST` | `10` | Number of most-recent files that are never deleted |
| `ALLOWED_USER_IDS` | *(empty = open)* | Comma-separated Telegram user IDs |
| `ADMIN_USER_ID` | *(none)* | Receives error notifications |
| `LOG_LEVEL` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `LOG_FILE` | `logs/bot.log` | Log file path |

---

## Getting Your Credentials

### Telegram API ID & Hash

Required by Pyrogram for the direct MTProto connection.

1. Go to [my.telegram.org](https://my.telegram.org) and sign in.
2. Click **"API development tools"**, fill in a short app name, click **"Create application"**.
3. Copy `App api_id` and `App api_hash`.

### Bot Token

Open [@BotFather](https://t.me/BotFather), send `/newbot`, follow the prompts, copy the token.

### GitHub Token

Go to **Settings → Developer settings → Personal access tokens → Tokens (classic)**, generate a new token with the **`repo`** scope, copy it immediately.

### Your Telegram User ID

Send `/start` to [@userinfobot](https://t.me/userinfobot) — it will reply with your numeric ID.

---

## Bot Commands

| Command | Description |
|---|---|
| `/start` | Welcome message with inline buttons |
| `/help` | Full help text |
| `/setpath <folder>` | Set a custom upload sub-folder |
| `/clearpath` | Reset to the default date-based path |
| `/status` | Show current configuration |

---

## File Size Limits & Smart Routing

| Layer | Maximum |
|---|---|
| Pyrogram / MTProto (download from Telegram) | **2 000 MB** |
| GitHub Contents API (files below threshold) | **100 MB** |
| Git LFS (files above threshold) | **2 GB** (free tier) |

The bot decides automatically:
- Files smaller than `LFS_THRESHOLD_MB` (default 50 MB) → **Contents API**
- Files larger → **Git LFS**

---

## Auto-Cleanup (for small VPS disks)

Enable this if you're on a VPS with limited SSD space. When total uploaded file size exceeds `CLEANUP_MAX_REPO_MB`, the bot deletes the oldest files while always keeping the `CLEANUP_KEEP_LATEST` most recent ones intact.

```env
CLEANUP_ENABLED=true
CLEANUP_MAX_REPO_MB=2048
CLEANUP_KEEP_LATEST=10
```

---

## Upload Path Format

```
{UPLOAD_BASE_PATH}/{folder}/{filename}

# Default (no /setpath):
uploads/2024-01-15/photo.jpg

# With /setpath project/images:
uploads/project/images/photo.jpg
```

---

## 💛 Support the Project — Donate

<div align="center">

This project was built out of necessity — to help people in Iran access their files freely, without relying on expensive VPNs.
If this tool has been useful to you, consider supporting its continued development. Every contribution, big or small, is deeply appreciated. 🙏

</div>

<br>

<div align="center">

|  | Network | Address |
|:---:|:---|:---|
| 💎 | **TON** | `UQDPxrimgBU6Mil0dhDn0Fc303RLRXKr9hGGDu7bTEBdGGqs` |
| 🔴 | **TRX (Tron)** | `TXix7uf6JPUKvWeUbA4A7wmQLVKnDbLRQU` |
| 🔷 | **ETH (Ethereum)** | `0x1FC907d3396460f1Cd94E3BC48564b1b46b70026` |

</div>

<br>

<div align="center">

> Thank you for keeping this project alive. ❤️

</div>
