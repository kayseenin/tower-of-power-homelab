cat > docs/build-log-phase1-router.md << 'EOF'
# Phase 1: Router Foundation

**Completed:** February 14, 2025  
**Duration:** ~2 hours  
**Status:** ✅ Complete

## Overview

Configured GL-BE3600 router as the network foundation for the homelab. Established professional subnet structure, implemented SSH key authentication, and verified connectivity.

## Goals

- Factory reset and initial configuration
- Migrate to 10.X.0.0/16 professional subnet
- Implement SSH key-based authentication
- Install essential networking tools
- Verify internet connectivity

## Configuration Steps

### 1. Factory Reset & Initial Access

**Reset procedure:**
- Held reset button for 10 seconds during power-on
- Router reverted to factory defaults
- Connected via default WiFi SSID

**Initial setup:**
- Web interface: http://192.168.8.1
- Set strong admin password
- Connected router to upstream network

### 2. Network Architecture

**Subnet design:**
```
10.X.0.0/16 - Homelab network
├── 10.X.0.1      - Router (BE-3600)
├── 10.X.10.0/24  - Infrastructure (static assignments)
│   ├── 10.X.10.2 - Raspberry Pi (planned)
│   ├── 10.X.10.3 - Mac Mini (planned)
│   └── 10.X.10.4 - MikroTik Switch (planned)
└── 10.X.20.0/24  - DHCP pool (dynamic clients)
    └── 10.X.20.100-250 - DHCP range
```

**Applied configuration:**
- Router IP: 10.X.0.1
- Subnet mask: 255.255.0.0 (/16)
- DHCP start: 10.X.20.100
- DHCP end: 10.X.20.250

### 3. SSH Authentication

**Key generation (MacBook):**
```bash
ssh-keygen -t ed25519 -f ~/.ssh/homelab_key -C "homelab-infrastructure"
cat ~/.ssh/homelab_key.pub  # Copy output
```

**Router configuration:**
- Added public key via GL.iNet web interface
- Verified key-based login works
- **Retained password authentication** for recovery access

**Test:**
```bash
ssh -i ~/.ssh/homelab_key root@10.X.0.1  # ✅ Success
```

### 4. System Packages

**Installation:**
```bash
opkg update
opkg install nano htop tcpdump curl wget-ssl iperf3
```

**Packages installed:**
- `nano` - Text editor
- `htop` - System resource monitor
- `tcpdump` - Network packet analyzer
- `curl` / `wget-ssl` - HTTP/HTTPS utilities
- `iperf3` - Network performance testing

### 5. System Configuration

**Timezone:**
```bash
uci set system.@system[0].timezone='EST5EDT,M3.2.0,M11.1.0'
uci set system.@system[0].zonename='America/New_York'
uci commit system
/etc/init.d/system restart
```

**Hardware offload:**
- Attempted to enable flow offloading
- Encountered nftables compatibility issue
- Reverted changes - router likely pre-optimized
- Performance remains excellent

### 6. Connectivity Verification

**Tests performed:**
```bash
nslookup google.com      # ✅ DNS resolution
ping -c 3 8.8.8.8        # ✅ Internet connectivity
curl -I https://google.com  # ✅ HTTPS working
```

## Technical Details

**Hardware:**
- Model: GL-BE3600 (Slate 7)
- WiFi: WiFi 7 (BE3600)
- Ethernet: 2x 2.5Gbps ports
- Firmware: OpenWRT 23.05 (GL.iNet)

## Issues & Solutions

### Hardware Offload Error

**Issue:** nftables compatibility error when enabling flow offloading

**Resolution:** Removed offload configuration; router performance unaffected

**Lesson:** Newer routers use different subsystems requiring adjusted configuration

## Security Posture

**Implemented:**
- ✅ SSH key authentication
- ✅ Strong admin password
- ✅ Non-standard subnet
- ✅ DHCP range separation

**Future hardening:**
- [ ] Disable password SSH auth
- [ ] VLAN firewall rules
- [ ] VPN policy routing
- [ ] Intrusion detection

## Next Phase

**Phase 2: MikroTik Switch Configuration**
- Configure CRS310 management IP
- Create bridge for ports
- Connect to router
- Test Layer 2 connectivity

## Lessons Learned

1. **GL.iNet interface stability:** Custom interface more stable than LuCI on BE-3600
2. **Recovery access:** Keeping password auth enabled prevents lockout scenarios
3. **Dev/prod separation:** Configuring independently allows testing without disruption
4. **Firmware differences:** Newer routers require adjusted configuration approaches

---

**Phase 1 complete. Router foundation established.**
