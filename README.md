# Komodo Docker Management

Automated Docker container management using Komodo, GitHub, and Renovate.

## 🎯 What This Does

This repository provides **fully automated Docker container updates** with GitOps workflows:
- ✅ Renovate automatically detects new Docker image versions
- ✅ Creates Pull Requests with changelogs for review
- ✅ After merge, webhooks trigger automatic redeployment
- ✅ No manual intervention required (but you control what updates)
- ✅ Full audit trail in Git history

## Architecture

```
Docker Compose Files (GitHub)
         ↓
    Renovate (GitHub Actions)
         ↓
    Opens PR when updates found
         ↓
    User reviews & merges PR
         ↓
    GitHub Webhook → Cloudflare Tunnel
         ↓
    Komodo (https://komodo.dezznuts.me)
         ↓
    Deploys to Docker Host (192.168.211.187)
```

## Infrastructure

- **Komodo Server**: 192.168.211.108 (https://komodo.dezznuts.me)
- **Docker Host**: 192.168.211.187
- **Source Host**: 192.168.211.14 (containers being migrated from here)

## Services

### Core Media Automation Stack
- **prowlarr** - Indexer manager
- **radarr** - Movie automation
- **sonarr** - TV show automation
- **readarr** - Book automation
- **qbittorrent** - Download client
- **flaresolverr** - Cloudflare solver

## Directory Structure

```
├── .github/
│   └── workflows/
│       └── renovate.yml    # Renovate automation workflow
├── stacks/                 # Docker compose files for each service
│   ├── flaresolverr/
│   ├── prowlarr/
│   ├── radarr/
│   ├── sonarr/
│   ├── readarr/
│   └── qbittorrent/
├── renovate.json           # Renovate configuration
├── .gitignore             # Exclude sensitive files
└── README.md              # This file
```

## 🔄 How The Automation Works

### Daily Update Checks (Renovate)
1. **Renovate runs** at 2 AM UTC daily (or on-demand)
2. **Scans docker-compose.yml files** for pinned image versions
3. **Checks for newer versions** on Docker registries
4. **Creates Pull Requests** with:
   - Updated version numbers
   - Release notes and changelogs
   - Automatic assignment and labels

### Automatic Deployment (After PR Merge)
1. **You merge** the Renovate PR on GitHub
2. **GitHub webhook** fires to Komodo (via Cloudflare Tunnel)
3. **Komodo Procedure** runs automatically:
   - Pulls latest code from GitHub
   - Detects changed stacks
   - Redeploys only changed containers
4. **Container updated** with zero downtime

### Example: FlareSolverr Update
```
Day 1: FlareSolverr v3.3.21 running
  ↓
Day 2: Renovate detects v3.4.6 available
  ↓
Day 2: PR #2 created automatically
  ↓
Day 3: You review changelog and merge PR
  ↓
Day 3: Webhook triggers → Komodo pulls → Redeploys
  ↓
Day 3: FlareSolverr v3.4.6 now running ✅
```

## Adding New Services

1. Create a new directory under `stacks/` with the service name
2. Add `docker-compose.yml` with **pinned image versions**
3. Add any environment variables to `.env` (this file is gitignored)
4. Create a Stack in Komodo pointing to this directory
5. Deploy via Komodo

## Security Notes

- `.env` files are excluded from git (see `.gitignore`)
- Sensitive data should be managed via Komodo's environment variables
- All image versions should be pinned (not `:latest`)

## 📊 Monitoring & Maintenance

### Daily Tasks
- Check for Renovate PRs on GitHub
- Review changelogs and decide to merge

### Weekly Tasks  
- Monitor Komodo dashboard for deployment status
- Verify containers are running expected versions

### Troubleshooting
- **Webhook not triggering?** Check GitHub webhook deliveries for errors
- **Repo not pulling?** Verify Git Provider credentials in Komodo Settings
- **Stack not deploying?** Check Komodo Procedure execution logs

## 🔧 Configuration Files Explained

### `renovate.json`
```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "docker-compose": {
    "fileMatch": ["(^|/)docker-compose[^/]*\\.ya?ml$"]
  }
}
```
- Scans all docker-compose.yml files
- Uses recommended Renovate presets
- Automatically assigns PRs and adds labels

### `.github/workflows/renovate.yml`
- Runs Renovate via GitHub Actions
- Schedule: Daily at 2 AM UTC
- Uses RENOVATE_TOKEN secret for authentication

## 🔐 Security Configuration

### Secrets & Credentials
1. **GitHub PAT** (`RENOVATE_TOKEN`): Stored in GitHub Secrets
2. **Webhook Secret**: Configured in Komodo and GitHub webhook
3. **Komodo Passkey**: Set in Komodo Periphery environment
4. **SSH Key**: ED25519 key for host authentication

**Note**: Actual credential values are not stored in this repository for security reasons.

### What's NOT in Git
- `.env` files (use Komodo environment variables instead)
- Passwords and API keys
- Private certificates

## 🚀 Quick Start Guide

### For New Containers
1. Create directory: `stacks/[service-name]/`
2. Add `docker-compose.yml` with **pinned versions**:
   ```yaml
   image: ghcr.io/service:v1.2.3  # ✅ Pinned
   # NOT: ghcr.io/service:latest  # ❌ Don't use :latest
   ```
3. Create Stack in Komodo UI
4. Deploy manually first time
5. Future updates will be automatic via Renovate!

## 📝 Maintenance

---
*Managed by Komodo | Automated with Renovate | Secured with Cloudflare Tunnel*