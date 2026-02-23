# Komodo Docker Management - Agent Work Log

## Project Overview
Migrating media automation stack from 192.168.211.14 to 192.168.211.187 with automated version updates using GitHub, Renovate, and Komodo.

## Infrastructure Details

### Servers
- **Komodo Server**: 192.168.211.108:9120
  - Public URL: https://komodo.dezznuts.me
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
- **URL**: https://komodo.dezznuts.me
- **Status**: ✅ Configured
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
    Cloudflare Tunnel (komodo.dezznuts.me)
         ↓
    Komodo (192.168.211.108)
         ↓
    Pulls latest & redeploys
         ↓
    Target Host (192.168.211.187)
```

## Migration Strategy

### Phase 1: Infrastructure Setup
1. ✅ Set up Cloudflare Tunnel (komodo.dezznuts.me)
2. Configure Komodo to manage 192.168.211.187
3. Set up GitHub repository structure
4. Configure Renovate for automated dependency scanning
5. Set up GitHub webhook to Komodo via Cloudflare Tunnel

### Phase 2: Container Migration (One at a time)
**Migration Order:**
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
- [x] Cloudflare Tunnel configured (komodo.dezznuts.me)
- [x] Add target host to Komodo (192.168.211.187 configured as managed server)
- [x] Repository structure setup (stacks/flaresolverr, .github/workflows, renovate.json)
- [x] Renovate configuration (GitHub Actions workflow, runs daily at 2 AM UTC)
- [x] GitHub PAT created and added as RENOVATE_TOKEN secret
- [x] Komodo Git Provider configured with GitHub credentials
- [x] Komodo Periphery installed on 192.168.211.187
- [x] SSH key authentication setup (C:\Users\domin\.ssh\proxmox_key)
- [x] GitHub webhook setup (webhook ID: 597477246)
- [x] First container migration: FlareSolverr v3.3.21 → v3.4.6 successfully
- [x] Komodo auto-deploy procedure created and tested
- [x] Renovate workflow tested end-to-end (PR #2 merged, auto-deployed)
- [x] Complete automation pipeline working
- [ ] Migrate remaining containers (Prowlarr, qBittorrent, Radarr, Sonarr, Readarr, Jackett)

## Next Steps
1. Create Komodo Procedure for auto-deploy (Pull Repo + Deploy Stack stages)
   - Navigate to https://komodo.dezznuts.me/procedures
   - Create new procedure: "Auto Deploy on Git Push"
   - Stage 1: Pull Repo → komodo-stacks
   - Stage 2: Deploy Stack → flaresolverr (with "if changed" flag)
2. Configure webhook in Komodo to trigger procedure on push events
3. Test end-to-end workflow:
   - Make a test commit to FlareSolverr stack
   - Verify webhook triggers procedure
   - Verify automatic redeployment
4. Test Renovate:
   - Wait for daily run or manually trigger workflow
   - Verify PR creation for image updates
   - Merge PR and verify webhook triggers deployment
5. Migrate remaining containers: Prowlarr, qBittorrent, Radarr, Sonarr, Readarr, Jackett

## Important Notes
- **Portainer skipped** - Komodo replaces it
- **Tandoor skipped** - Not needed
- **Watchtower will be removed** - Renovate provides better control
- **Jackett optional** - Prowlarr can replace it
- **Environment variables must be secured** - Use .gitignore for .env files
- **Data persistence** - Ensure volume mappings are correct for data migration

## Technical Details

### SSH Authentication
- **Key Location**: C:\Users\domin\.ssh\proxmox_key (ED25519)
- **Command Format**: `ssh -i "$env:USERPROFILE\.ssh\proxmox_key" root@<host>`
- **Deployed to**: All hosts (192.168.211.187, 192.168.211.108, 192.168.211.14, 192.168.211.9)

### GitHub Configuration
- **Webhook ID**: 597477246
- **Webhook URL**: https://komodo.dezznuts.me/listener/github/procedure/699b374cad1a1820b92fda2f/main
- **Webhook Secret**: GBmP5jUl1OUAluqaeixBZ4KTJLdv
- **Webhook Events**: push
- **Webhook Status**: ✅ Working (returns 200 OK)
- **Authentication**: GitHub CLI authenticated as dominic8686
- **Renovate Token**: Stored in GitHub Secrets as RENOVATE_TOKEN

### Komodo Configuration
- **Host Setting**: KOMODO_HOST=https://komodo.dezznuts.me
- **Webhook Secret**: GBmP5jUl1OUAluqaeixBZ4KTJLdv (global default)
- **Procedure ID**: 699b374cad1a1820b92fda2f
- **Procedure Name**: Auto Deploy on Git Push
- **Procedure Stages**:
  1. Pull Repo (komodo-stacks)
  2. Deploy Stack (flaresolverr, if changed)

### Komodo Periphery
- **Container**: komodo-periphery on 192.168.211.187
- **Repo Path (host)**: /opt/komodo-periphery/repos/
- **Repo Path (container)**: /etc/komodo/repos/github/
- **Clone Path**: Empty (uses default git provider structure)
- **Passkey**: securerandompasskey123

### FlareSolverr Stack
- **Stack ID**: 699b0d11ad1a1820b92fd47c
- **Run Directory**: /etc/komodo/repos/github/stacks/flaresolverr
- **Compose File**: docker-compose.yml
- **Initial Version**: v3.3.21
- **Current Version**: v3.4.6 (auto-updated via Renovate PR #2)
- **Status**: ✅ Deployed and automated

## ✅ Completed Milestones

### Infrastructure Setup (Complete)
- ✅ Cloudflare Tunnel configured for webhook access
- ✅ Komodo managing Docker on 192.168.211.187
- ✅ SSH key authentication deployed to all hosts
- ✅ GitHub repository with proper structure

### Automation Pipeline (Complete)
- ✅ Renovate scanning docker-compose.yml files daily
- ✅ GitHub webhook triggering Komodo procedure
- ✅ Komodo procedure pulling repo and deploying stacks
- ✅ End-to-end tested: PR merge → webhook → deploy → v3.4.6 running

### First Container Migration (Complete)
- ✅ FlareSolverr migrated from 192.168.211.14 to 192.168.211.187
- ✅ Version pinned in docker-compose.yml
- ✅ Renovate detected update (v3.3.21 → v3.4.6)
- ✅ PR created, merged, and automatically deployed

## 🎯 Troubleshooting Notes

### Issues Resolved
1. **Webhook 400 Error**: Fixed by setting correct KOMODO_HOST environment variable
2. **Repo Pull Failing**: Fixed by clearing Clone Path (use git provider default structure)
3. **Procedure ID Mismatch**: Fixed by using correct procedure ID from Komodo UI
4. **SSH Password Issues**: Fixed by creating ED25519 key pair
5. **Token Expired**: Created new GitHub PAT with all permissions

### Key Learnings
- Komodo uses `/etc/komodo/repos/{git_provider_domain}/` structure
- Clone Path should be empty to use default structure
- KOMODO_HOST must match your actual domain for webhooks to work
- Container restart doesn't reload env vars - need to recreate with `docker compose up -d --force-recreate`
- Procedure webhook URL format: `https://{host}/listener/github/procedure/{id}/{branch}`

## 📚 Documentation

- **README.md**: User-facing documentation for the automation
- **agent.md**: Technical implementation log (this file)
- **renovate.json**: Renovate configuration
- **.github/workflows/renovate.yml**: GitHub Actions workflow

---
*Last Updated: 2026-02-23 07:13*
*Status: ✅ Complete automation pipeline operational*
