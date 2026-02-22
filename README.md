# Komodo Docker Management

Automated Docker container management using Komodo, GitHub, and Renovate.

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

## Workflow

1. **Renovate** scans compose files daily for Docker image updates
2. When updates are found, Renovate creates a **Pull Request**
3. Review the PR and **merge** if acceptable
4. GitHub webhook triggers **Komodo** via Cloudflare Tunnel
5. Komodo **pulls latest changes** and **redeploys** affected containers

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

## Maintenance

- Review Renovate PRs regularly
- Test updates before merging to production
- Monitor Komodo dashboard for deployment status

---
*Managed by Komodo | Automated with Renovate | Secured with Cloudflare Tunnel*