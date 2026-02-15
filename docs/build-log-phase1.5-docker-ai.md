# Phase 1.5: Docker Services & Local AI

**Completed:** February 14, 2025  
**Duration:** ~3 hours  
**Status:** ✅ Complete

## Overview

Expanded Docker infrastructure with AI capabilities. Deployed Ollama with Open WebUI for completely private, self-hosted AI accessible across the network.

## Goals

- Deploy local AI solution on Mac Mini
- Containerize AI for portability and easy management
- Enable remote access to AI services
- Add monitoring for new services
- Maintain secure credentials management

## Services Deployed

### Ollama + Open WebUI Stack

**Architecture:**
```
Open WebUI (Port 3002)
    ↓
Ollama Engine (Port 11434)
    ↓
Llama 3.2 3B Model (~2GB)
```

**Deployment via Portainer:**
```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama:/root/.ollama
    restart: unless-stopped

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3002:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama
    restart: unless-stopped

volumes:
  ollama:
```

**Model selected:** Llama 3.2 3B
- Smaller footprint (2GB vs 4.7GB for 8B)
- Fits in available RAM (~3-4GB needed)
- Fast inference on M4
- Still highly capable for homelab tasks

**Access:** `http://mac-mini-ip:3002`

## Configuration Steps

### 1. Portainer Password Reset

**Issue:** Forgot Portainer admin credentials

**Resolution:**
- Stopped and removed Portainer container
- Deleted volume to clear stored credentials
- Redeployed fresh Portainer instance
- Reconnected to local Docker environment
- All existing containers remained running (Docker manages them, not Portainer)

**Lesson:** Portainer is just a UI - containers are independent

### 2. Credentials Management System

**Created secure credentials tracking:**
- File: `~/Documents/.homelab-secure/credentials.md`
- Permissions: `600` (owner read/write only)
- Added to `.gitignore` to prevent accidental commits
- Contains all service logins, SSH keys, API URLs

**Security considerations:**
- Stored outside git repository
- Restricted file permissions
- Separate from public documentation
- Backed up separately

### 3. Ollama Deployment

**Process:**
1. Created stack in Portainer named `local-ai`
2. Deployed via web editor (paste YAML)
3. Initial image pulls (~6-10GB total):
   - `ollama/ollama` - AI engine
   - `ghcr.io/open-webui/open-webui:main` - Web interface
4. Pull time: ~5-10 minutes (depends on connection)

### 4. Model Selection & Download

**Initial attempt:** Llama 3.1 8B
- Error: `model requires more system memory (4.8 GiB) than is available (4.1 GiB)`
- macOS + other services using ~12GB of 16GB total

**Solution:** Llama 3.2 3B
```bash
docker exec -it ollama ollama pull llama3.2:3b
```
- Download size: ~2GB
- RAM usage: ~3-4GB
- Successfully loads and runs

### 5. Open WebUI Configuration

**First-time setup:**
- Created local admin account (email: `admin@local`)
- No internet account required (fully local)
- Selected llama3.2:3b model from dropdown
- Tested with homelab-related queries

### 6. Monitoring Integration

**Added to Uptime Kuma:**
- Monitor type: HTTP(s)
- URL: `http://mac-mini-ip:3002`
- Heartbeat: 60 seconds
- Status: ✅ Up

## Technical Details

**Hardware considerations:**
- M4 Neural Engine handles inference efficiently
- 16GB RAM total, ~4GB available after OS/services
- 3B model sweet spot for available resources
- SSD storage handles model loading quickly

**Container resource usage:**
- ollama: ~100MB idle, ~4GB when generating
- open-webui: ~50MB
- Total stack overhead: Minimal when idle

**Network accessibility:**
- Accessible on LAN via Mac Mini IP
- Ready for Tailscale remote access (Phase 4)
- No port forwarding needed (internal only)

## Issues & Solutions

### RAM Limitations

**Issue:** 8B model requires more RAM than available

**Root cause:**
- macOS baseline usage: ~6-8GB
- Other services (Docker, Portainer, Uptime Kuma): ~2-4GB
- Available for AI: ~4GB
- 8B model needs: ~8GB

**Solutions attempted:**
1. Tried loading 8B anyway → Failed
2. Considered closing other apps → Would impact workflow
3. **Selected smaller 3B model** → Success ✅

**Lesson:** Right-size models to available resources. 3B is plenty capable for homelab use cases.

### LM Studio Server Configuration

**Issue:** Couldn't enable server mode in LM Studio 0.4.2

**Attempts:**
- Located Developer settings
- Found "Local LLM Service" option
- Enabled but server wouldn't start
- Port 1234 never opened

**Resolution:** Switched to Ollama in Docker instead

**Benefits of Docker approach:**
- More reliable server startup
- Containerized (portable, reproducible)
- Easier remote access configuration
- Integrates with existing Docker stack
- Better resource management

## Use Cases Validated

**Tested AI capabilities:**
- Technical explanations (Docker, networking)
- Code generation (bash scripts)
- Homelab troubleshooting assistance
- Documentation help
- Learning new concepts

**Performance:**
- Response latency: 1-2 seconds to start
- Generation speed: ~15-25 tokens/second
- Quality: Excellent for technical queries
- Context retention: Good for multi-turn conversations

## Security Posture

**Current:**
- ✅ Local network only (not exposed to internet)
- ✅ Account required (local authentication)
- ✅ Credentials documented securely
- ✅ Container isolation

**Future (with Tailscale):**
- Encrypted access over mesh VPN
- No public exposure
- Access from anywhere securely

## Next Phase

**Phase 2: Network Infrastructure**
- MikroTik CRS310 switch configuration
- Physical topology establishment
- Wire Mac Mini to switch

**Phase 4: Remote Access (Priority)**
- Tailscale mesh VPN deployment
- Subnet routing configuration
- MagicDNS setup
- Test AI access via cellular hotspot

## Lessons Learned

1. **Model sizing matters:** Don't assume biggest = best. Right-size to resources.

2. **Docker > Native apps for servers:** Containerization provides better control, portability, and reliability.

3. **Portainer recovery is simple:** UI state is separate from actual containers. Reset doesn't affect running services.

4. **Credentials management from day 1:** Created system proactively instead of reactively after lockout.

5. **RAM is the bottleneck:** For AI workloads on 16GB system, plan ~4GB available for models after OS/services.

## Resources

- [Ollama Documentation](https://ollama.ai)
- [Open WebUI GitHub](https://github.com/open-webui/open-webui)
- [Llama 3.2 Model Card](https://ollama.ai/library/llama3.2)

---

**Phase 1.5 complete. Local AI operational and ready for remote access.**
