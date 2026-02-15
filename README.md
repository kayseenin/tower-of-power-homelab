# Tower of Power Homelab

Self-hosted infrastructure with Docker, monitoring, AI, and policy-based networking.

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
- Deploy private AI for learning and automation
- Practice Infrastructure as Code principles
- Gain hands-on experience with production-grade tools

---

## 🚧 Current Status: Systematic Build

Building from scratch with proper documentation and professional practices.

### Phase Progress

**✅ Phase 1: Router Foundation** - Complete
- GL-BE3600 configured with 10.X.0.0/16 network
- SSH key authentication implemented
- Essential packages installed
- Internet connectivity verified

**✅ Phase 1.5: Docker Services & AI** - Complete
- Ollama + Open WebUI deployed
- Local AI (Llama 3.2 3B) operational
- Accessible across LAN
- Monitoring configured

**⏳ Phase 2: Network Infrastructure** - Next
- MikroTik CRS310 switch configuration
- Physical topology establishment
- Layer 2 connectivity testing

**📋 Phase 3: DNS & Ad Blocking** - Planned
- Raspberry Pi setup with Pi-hole
- Network-wide ad blocking
- DNS architecture

**📋 Phase 4: Remote Access** - High Priority
- Tailscale mesh VPN deployment
- Subnet routing configuration
- MagicDNS setup
- Test access via cellular hotspot

**📋 Phase 5: Service Expansion** - Planned
- Homepage dashboard
- Additional Docker services
- Media server considerations

**📋 Phase 6: Advanced Features** - Future
- VPN policy routing for streaming
- VLAN implementation
- Automated monitoring & alerts

---

## 🏗️ Architecture

### Hardware

- **Mac Mini M4** - Primary server (Docker, AI, Portainer)
- **GL-BE3600** - WiFi 7 travel router (OpenWRT)
- **MikroTik CRS310** - 10-port managed switch (planned)
- **Raspberry Pi** - DNS/ad-blocking (planned)
- **Nvidia Shield Pro** - Primary streaming device
- **HP Omen 16** - Game streaming host (planned)

### Current Services

| Service | Purpose | Port | Status |
|---------|---------|------|--------|
| Portainer | Container Management | 9000 | ✅ Running |
| Uptime Kuma | Service Monitoring | 3001 | ✅ Running |
| Ollama | AI Engine | 11434 | ✅ Running |
| Open WebUI | AI Chat Interface | 3002 | ✅ Running |

---

## 📚 Documentation

- [Getting Started Guide](docs/getting-started.md) - Initial setup walkthrough
- [Docker Setup](docs/docker-setup.md) - Container configuration
- [Monitoring Setup](docs/monitoring-setup.md) - Uptime Kuma & Healthchecks
- [Lessons Learned](docs/lessons-learned.md) - Challenges and solutions

### Build Logs

- [Phase 1: Router Foundation](docs/build-log-phase1-router.md) - GL-BE3600 setup
- [Phase 1.5: Docker & AI](docs/build-log-phase1.5-docker-ai.md) - Ollama deployment

---

## 🎓 Skills Demonstrated

### DevOps & Infrastructure
- Container orchestration (Docker)
- Infrastructure as Code (docker-compose)
- Remote system administration (SSH)
- Service monitoring and alerting
- Network architecture design
- Credentials management

### AI & Machine Learning
- Local LLM deployment (Ollama)
- Model selection and optimization
- Resource-constrained AI hosting
- Private AI infrastructure

### Networking
- Professional subnet design (10.X.0.0/16)
- Router configuration (OpenWRT)
- Switch management (MikroTik)
- VPN mesh networking (Tailscale - planned)
- DNS architecture (Pi-hole - planned)

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
5. Deploy Ollama for local AI
6. Set up external monitoring (Healthchecks.io)

See [Getting Started Guide](docs/getting-started.md) for detailed instructions.

---

## 🔮 Next Up

**Tomorrow's priorities:**
- Tailscale deployment across all devices
- Test remote access via cellular hotspot
- Access AI from anywhere
- MikroTik switch integration (if time)

**Later:**
- Raspberry Pi with Pi-hole
- Homepage dashboard
- VLAN segmentation
- VPN policy routing

---

## 📝 Development Log

**February 14, 2025 - Phase 1.5 Complete**
- Deployed Ollama + Open WebUI in Docker
- Llama 3.2 3B model operational
- Local AI accessible across LAN
- Portainer credentials reset and secured
- Credentials management system created

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
