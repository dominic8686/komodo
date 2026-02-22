# Komodo Docker Management - Agent Work Log

## Project Overview
Migrating media automation stack from 192.168.211.14 to 192.168.211.187 with automated version updates using GitHub, Renovate, and Komodo.

## Infrastructure Details

### Servers
- **Komodo Server**: 192.168.211.108:9120
  - Credentials: admin / TempPass123!
  - Status: ✅ Deployed and accessible
  
- **Source Docker Host**: 192.168.211.14
  - Credentials: root / M&Tallica
  - Docker Compose Location: /home/dock/.docker/compose/
  - Status: ✅ Running 11 containers
  
- **Target Docker Host**: 192.168.211.187
  - Credentials: root / M&Tallica
  - Docker Version: 29.2.1
  - Docker Compose Version: v5.0.2
  - Status: ✅ Docker installed, ready for migration
  
- **Proxmox Host**: 192.168.211.9
  - Credentials: root / M&Tallica
  - Purpose: Hypervisor hosting VMs

### GitHub Repository
- **Repo**: https://github.com/dominic8686/komodo
- **Purpose**: Store Docker Compose files for version control and automated updates

### Webhook Integration
- **Method**: Cloudflare Tunnel
- **Purpose**: Secure connection for GitHub webhooks to reach Komodo

## Containers to Migrate

### ✅ Core Media Automation Services (6-7 containers)
1. **prowlarr** - Indexer manager (centralized)
2. **radarr** - Movie automation
3. **sonarr** - TV show automation
4. **readarr** - Book/audiobook automation
5. **qbittorrent** - Torrent download client
6. **flaresolverr** - Cloudflare bypass solver
7. **jackett** (optional) - Legacy indexer proxy

### ❌ Containers to Skip
- **portainer** - ❌ Redundant (using Komodo instead)
- **watchtower** - ❌ Replaced by Renovate
- **tandoor** - ❌ Not needed

### Compose File Structure
```
/home/dock/.docker/compose/
├── docker-compose.yml (main file with all 11 services)
├── .env (main environment variables)
├── tandoor.env
└── other app-specific .env files
```

## Workflow Architecture

```
Docker Compose Files (GitHub)
         ↓
    Renovate (GitHub Actions)
    (Replaces Watchtower)
         ↓
    Opens PR when updates found
         ↓
    User reviews & merges PR
         ↓
    GitHub Webhook
         ↓
    Cloudflare Tunnel
         ↓
    Komodo (192.168.211.108)
         ↓
    Pulls latest & redeploys
         ↓
    Target Host (192.168.211.187)
```

## Migration Strategy

### Phase 1: Infrastructure Setup
1. Set up Cloudflare Tunnel for Komodo webhook access
2. Configure Komodo to manage 192.168.211.187
3. Set up GitHub repository structure
4. Configure Renovate for automated dependency scanning
5. Set up GitHub webhook to Komodo via Cloudflare Tunnel

### Phase 2: Container Migration (One at a time)
**Suggested Migration Order:**
1. **FlareSolverr** (supporting service, least dependencies)
2. **Prowlarr** (indexer manager, central config)
3. **qBittorrent** (download client)
4. **Radarr** (movies)
5. **Sonarr** (TV shows)
6. **Readarr** (books)
7. **Jackett** (optional, if still needed)

### Phase 3: Validation & Cleanup
1. Verify all services running on new host
2. Test Renovate PR creation
3. Test webhook automation
4. Decommission old host (192.168.211.14)

## Current Progress
- [x] Komodo password reset
- [x] GitHub repository created
- [x] Docker installed on target host (192.168.211.187)
- [x] Inventory of source containers (192.168.211.14)
- [x] Identified containers to migrate (6-7 core services)
- [ ] Set up Cloudflare Tunnel
- [ ] Add target host to Komodo
- [ ] Repository structure setup
- [ ] Renovate configuration
- [ ] GitHub webhook setup
- [ ] First container migration

## Next Steps
1. Set up Cloudflare Tunnel for webhook access
2. Add 192.168.211.187 to Komodo as managed host
3. Create GitHub repository structure for compose files
4. Configure Renovate
5. Extract first container's compose config (FlareSolverr)
6. Begin migration

## Important Notes
- **Portainer skipped** - Komodo replaces it
- **Tandoor skipped** - Not needed
- **Watchtower will be removed** - Renovate provides better control
- **Jackett optional** - Prowlarr can replace it
- **Environment variables must be secured** - Use .gitignore for .env files
- **Data persistence** - Ensure volume mappings are correct for data migration

---
*Last Updated: 2026-02-22 13:17*