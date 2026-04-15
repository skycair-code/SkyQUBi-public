# S7 SkyQUB*i* — Deployment Guide

> *"Love is the architecture."*
> Civilian Use Only — CWS-BSL-1.1

## Requirements

- **Linux** — Fedora 44+, RHEL 9+, Debian 12+, Ubuntu 22.04+, or Arch
- **Podman 5.x+** (rootless)
- **envsubst** (from gettext)
- **Python 3**
- **4GB+ RAM** (8GB recommended)
- **10GB+ free disk**

## Quick Start

```bash
# 1. Clone
git clone https://github.com/skycair-code/SkyQUBi-public.git
cd SkyQUBi-public

# 2. Get the admin image
# Download s7-skyqubi-admin-v2.6.tar from the releases page
# Place it in this directory

# 3. Configure secrets
cp .env.example .env.secrets
chmod 600 .env.secrets
# Edit .env.secrets — change ALL "CHANGE_ME" values
# Generate passwords: python3 -c "import secrets; print(secrets.token_urlsafe(32))"

# 4. Pre-audit (checks your system before deploying)
./start-pod.sh --check

# 5. Deploy
./start-pod.sh

# 6. Open Command Center
xdg-open http://127.0.0.1:57080
```

## Pre-Deployment Audit

The deploy script runs a full system audit before deploying. It checks:

| Check | What it validates |
|---|---|
| OS | Linux distribution detected |
| Packages | podman, envsubst, python3, curl, git installed |
| User | Not running as root |
| Podman | Rootless mode, subuid/subgid, socket active |
| SELinux | Status + required booleans |
| Disk | Minimum 10GB free |
| /var/tmp | Minimum 512MB (for image operations) |
| Secrets | .env.secrets exists, permissions 600, no CHANGE_ME values |
| Image | Admin image tar or loaded image present |
| SQL | Init scripts present |
| Config | Pod YAML present |
| Ports | No conflicts on 57080/57086/57090 |

Run `./start-pod.sh --check` to audit without deploying.

If any check fails, the script tells you exactly how to fix it.

## What Gets Deployed

| Container | Purpose | Port |
|---|---|---|
| s7-admin | Command Center + CWS Engine + SkyAV*i* | 127.0.0.1:57080 |
| s7-postgres | CWS Engine database (pgvector) | 127.0.0.1:57090 |
| s7-mysql | Admin UI database | pod-internal |
| s7-redis | Session cache | pod-internal |
| s7-qdrant | Vector memory | 127.0.0.1:57086 |

All ports bind to **127.0.0.1 only** — nothing exposed to your network.

## Post-Deploy

The script automatically:
- Waits for all services to boot
- Configures Ollama connection (if running on host)
- Fixes service storage paths for your system
- Binds all app ports to localhost
- Marks built-in services (SkyAV*i*, Qdrant, Ollama)
- Verifies 7 core services and reports status

## AI Chat (Ollama)

Optional — install Ollama for AI chat:

```bash
curl -fsSL https://ollama.com/install.sh | sh
OLLAMA_HOST=0.0.0.0:57081 ollama serve &
ollama pull qwen3:0.6b
```

The `0.0.0.0` binding is required so the pod can reach Ollama on the host.

## Image Signing (Optional)

If the image tar has a `.sig` file and a `s7-image-signing.pub` key is present, the deploy script verifies the signature before loading. Unsigned or tampered images are rejected.

## Stop / Restart

```bash
./start-pod.sh --down    # Stop
./start-pod.sh           # Start (re-runs pre-audit)
./s7-manager.sh          # Interactive menu
```

## Security

- All ports: 127.0.0.1 only
- CWS API: Bearer token required
- Containers: rootless, no privilege escalation
- Secrets: 600 permissions, never committed to git
- Image: signature verification (when signed)
- License: Civilian use only

## Platform Support

| Distro | Package Manager | Install Command |
|---|---|---|
| Fedora/RHEL | dnf | `sudo dnf install podman gettext python3 curl git` |
| Debian/Ubuntu | apt | `sudo apt-get install podman gettext-base python3 curl git` |
| Arch | pacman | `sudo pacman -S podman gettext python curl git` |

---

*S7 SkyQUB*i* — AI + Humanity. Built on Trust.*
*Patent Pending: TPP99606 — 123Tech / 2XR, LLC*
