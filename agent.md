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

## Current Stack on 192.168.211.14

### Media Automation Services (11 containers)
1. **prowlarr** - Indexer manager (centralized)
2. **radarr** - Movie automation
3. **sonarr** - TV show automation
4. **readarr** - Book/audiobook automation
5. **jackett** - Indexer proxy (legacy, can be replaced by Prowlarr)
6. **qbittorrent** - Torrent download client
7. **flaresolverr** - Cloudflare bypass solver
8. **portainer** - Container management UI
9. **watchtower** - Auto-container updater (⚠️ will be replaced by Renovate)
10. **tandoor** - Recipe manager

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
1. ✅ Install Docker on target host (192.168.211.187)
2. Set up Cloudflare Tunnel for Komodo webhook access
3. Configure Komodo to manage 192.168.211.187
4. Set up GitHub repository structure
5. Configure Renovate for automated dependency scanning
6. Set up GitHub webhook to Komodo via Cloudflare Tunnel

### Phase 2: Container Migration (One at a time)
Migrate containers individually to minimize downtime:

**Suggested Migration Order:**
1. **Portainer** (management UI, least critical)
2. **FlareSolverr** (supporting service)
3. **Prowlarr** (indexer manager, central config)
4. **qBittorrent** (download client)
5. **Radarr** (movies)
6. **Sonarr** (TV shows)
7. **Readarr** (books)
8. **Tandoor** (recipe manager)
9. **Remove Watchtower** (replaced by Renovate)
10. **Remove Jackett** (if using Prowlarr exclusively)

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
5. Extract first container's compose config
6. Begin migration

## Important Notes
- **Watchtower will be removed** - Renovate provides better control
- **Jackett may be redundant** - Prowlarr can replace it
- **Environment variables must be secured** - Use .gitignore for .env files
- **Data persistence** - Ensure volume mappings are correct for data migration

---
*Last Updated: 2026-02-22 13:10*