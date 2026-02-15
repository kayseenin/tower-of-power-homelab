# Tower of Power Homelab

Self-hosted infrastructure with Docker, monitoring, and policy-based networking.

![Status](https://img.shields.io/badge/status-active-success)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat&logo=Raspberry%20Pi&logoColor=white)
![OpenWRT](https://img.shields.io/badge/OpenWRT-00B5E2?style=flat&logo=openwrt&logoColor=white)

---

## 🎯 Project Goals

- Learn container orchestration and infrastructure management
- Implement network-wide ad blocking and privacy protection
- Enable secure remote access without internet exposure
- Build automated monitoring and alerting systems
- Practice Infrastructure as Code principles
- Gain hands-on experience with production-grade tools

---

## 🚧 Current Status: Systematic Rebuild

Building from scratch with proper documentation and professional practices.

**Why rebuilding:**
- Fix Tailscale cross-platform connectivity issues
- Establish proper network foundation
- Document every decision and configuration
- Learn the "why" behind each component

### Phase Progress

**✅ Phase 1: Router Foundation** - Complete
- GL-BE3600 configured with 10.X.0.0/16 network
- SSH key authentication implemented
- Essential packages installed
- Internet connectivity verified

**⏳ Phase 2: Network Infrastructure** - Next
- MikroTik CRS310 switch configuration
- Physical topology establishment
- Layer 2 connectivity testing

**📋 Phase 3: DNS & Ad Blocking** - Planned
- Raspberry Pi setup with Pi-hole
- Network-wide ad blocking
- DNS architecture

**📋 Phase 4: Remote Access** - Planned
- Tailscale mesh VPN deployment
- Subnet routing configuration
- MagicDNS setup

**📋 Phase 5: Service Expansion** - Planned
- Docker service expansion
- Local AI deployment (LM Studio)
- Additional self-hosted applications

**📋 Phase 6: Advanced Features** - Future
- VPN policy routing for streaming
- VLAN implementation
- Automated monitoring & alerts

---

## 🏗️ Architecture

### Hardware

- **Mac Mini M4** - Primary server (Docker, Portainer)
- **GL-BE3600** - WiFi 7 travel router (OpenWRT)
- **MikroTik CRS310** - 10-port managed switch
- **Raspberry Pi** - DNS/ad-blocking (Pi-hole)
- **Nvidia Shield Pro** - Primary streaming device
- **HP Omen 16** - Game streaming host

### Current Services

| Service | Purpose | Port | Status |
|---------|---------|------|--------|
| Portainer | Container Management | 9000 | ✅ Running |
| Uptime Kuma | Service Monitoring | 3001 | ✅ Running |

---

## 📚 Documentation

- [Getting Started Guide](docs/getting-started.md) - Initial setup walkthrough
- [Docker Setup](docs/docker-setup.md) - Container configuration
- [Monitoring Setup](docs/monitoring-setup.md) - Uptime Kuma & Healthchecks
- [Lessons Learned](docs/lessons-learned.md) - Challenges and solutions

### Build Logs

- [Phase 1: Router Foundation](docs/build-log-phase1-router.md) - GL-BE3600 setup

---

## 🎓 Skills Demonstrated

### DevOps & Infrastructure
- Container orchestration (Docker)
- Infrastructure as Code (docker-compose)
- Remote system administration (SSH)
- Service monitoring and alerting
- Network architecture design

### Networking
- Professional subnet design (10.X.0.0/16)
- Router configuration (OpenWRT)
- Switch management (MikroTik)
- VPN mesh networking (Tailscale)
- DNS architecture (Pi-hole)

### Automation
- Cron job scheduling
- Automated health checks
- Dead man's switch monitoring
- Container lifecycle management

---

## 🚀 Quick Start

**Prerequisites:**
- Mac Mini or server with Docker
- Basic terminal/SSH knowledge

**Setup:**
1. Enable Remote Login on server
2. Install Docker via Homebrew
3. Deploy Portainer for container management
4. Configure monitoring

cat > README.md << 'EOF'
# Tower of Power Homelab

Self-hosted infrastructure with Docker, monitoring, and policy-based networking.

![Status](https://img.shields.io/badge/status-active-success)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat&logo=Raspberry%20Pi&logoColor=white)
![OpenWRT](https://img.shields.io/badge/OpenWRT-00B5E2?style=flat&logo=openwrt&logoColor=white)

---

## 🎯 Project Goals

- Learn container orchestration and infrastructure management
- Implement network-wide ad blocking and privacy protection
- Enable secure remote access without internet exposure
- Build automated monitoring and alerting systems
- Practice Infrastructure as Code principles
- Gain hands-on experience with production-grade tools

---

## 🚧 Current Status: Systematic Rebuild

Building from scratch with proper documentation and professional practices.

**Why rebuilding:**
- Fix Tailscale cross-platform connectivity issues
- Establish proper network foundation
- Document every decision and configuration
- Learn the "why" behind each component

### Phase Progress

**✅ Phase 1: Router Foundation** - Complete
- GL-BE3600 configured with 10.X.0.0/16 network
- SSH key authentication implemented
- Essential packages installed
- Internet connectivity verified

**⏳ Phase 2: Network Infrastructure** - Next
- MikroTik CRS310 switch configuration
- Physical topology establishment
- Layer 2 connectivity testing

**📋 Phase 3: DNS & Ad Blocking** - Planned
- Raspberry Pi setup with Pi-hole
- Network-wide ad blocking
- DNS architecture

**📋 Phase 4: Remote Access** - Planned
- Tailscale mesh VPN deployment
- Subnet routing configuration
- MagicDNS setup

**📋 Phase 5: Service Expansion** - Planned
- Docker service expansion
- Local AI deployment (LM Studio)
- Additional self-hosted applications

**📋 Phase 6: Advanced Features** - Future
- VPN policy routing for streaming
- VLAN implementation
- Automated monitoring & alerts

---

## 🏗️ Architecture

### Hardware

- **Mac Mini M4** - Primary server (Docker, Portainer)
- **GL-BE3600** - WiFi 7 travel router (OpenWRT)
- **MikroTik CRS310** - 10-port managed switch
- **Raspberry Pi** - DNS/ad-blocking (Pi-hole)
- **Nvidia Shield Pro** - Primary streaming device
- **HP Omen 16** - Game streaming host

### Current Services

| Service | Purpose | Port | Status |
|---------|---------|------|--------|
| Portainer | Container Management | 9000 | ✅ Running |
| Uptime Kuma | Service Monitoring | 3001 | ✅ Running |

---

## 📚 Documentation

- [Getting Started Guide](docs/getting-started.md) - Initial setup walkthrough
- [Docker Setup](docs/docker-setup.md) - Container configuration
- [Monitoring Setup](docs/monitoring-setup.md) - Uptime Kuma & Healthchecks
- [Lessons Learned](docs/lessons-learned.md) - Challenges and solutions

### Build Logs

- [Phase 1: Router Foundation](docs/build-log-phase1-router.md) - GL-BE3600 setup

---

## 🎓 Skills Demonstrated

### DevOps & Infrastructure
- Container orchestration (Docker)
- Infrastructure as Code (docker-compose)
- Remote system administration (SSH)
- Service monitoring and alerting
- Network architecture design

### Networking
- Professional subnet design (10.X.0.0/16)
- Router configuration (OpenWRT)
- Switch management (MikroTik)
- VPN mesh networking (Tailscale)
- DNS architecture (Pi-hole)

### Automation
- Cron job scheduling
- Automated health checks
- Dead man's switch monitoring
- Container lifecycle management

---

## 🚀 Quick Start

**Prerequisites:**
- Mac Mini or server with Docker
- Basic terminal/SSH knowledge

**Setup:**
1. Enable Remote Login on server
2. Install Docker via Homebrew
3. Deploy Portainer for container management
4. Configure monitoring with Uptime Kuma
5. Set up external monitoring (Healthchecks.io)

See [Getting Started Guide](docs/getting-started.md) for detailed instructions.

---

## 🔮 Future Enhancements

**Next up:**
- MikroTik switch integration
- Raspberry Pi with Pi-hole
- Tailscale remote access
- Homepage dashboard

**Later:**
- VLAN segmentation
- VPN policy routing
- Game streaming (Sunshine/Moonlight)
- Media server (Jellyfin)
- File sync (Nextcloud)
- Advanced monitoring (Grafana)

---

## 📝 Development Log

**February 14, 2025 - Phase 1 Complete**
- Configured GL-BE3600 router with professional subnet
- Implemented SSH key authentication
- Installed essential networking tools
- Established network foundation

**February 13, 2025 - Initial Setup**
- Mac Mini Docker and Portainer deployment
- Uptime Kuma monitoring configuration
- Healthchecks.io external monitoring
- Initial documentation created

---

## 🤝 Acknowledgments

Developed with guidance from Claude (Anthropic) for architecture planning, troubleshooting, and documentation.

---

## 📄 License

MIT License - Use as inspiration for your own homelab!

---

**Follow along:** Check `docs/build-log-phaseX-*.md` for detailed progress.
