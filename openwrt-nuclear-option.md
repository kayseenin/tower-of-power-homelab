# OpenWRT Homelab: The Nuclear Option 🚀
## A Hardened, Automated, Travel-Ready Network Lab

---

## 🎯 What You're Building Now

A **legitimate enterprise-grade network** that gives you:
- Complete control over your network stack (OpenWRT router)
- Automated captive portal login at hotels/airports
- VLANs for network segmentation (IoT, guests, lab, production)
- Ad blocking at multiple layers (Pi-hole + OpenWRT)
- VPN kill switches and policy-based routing
- Network-wide security hardening
- Travel router capabilities
- Full IDS/IPS with Suricata (optional but badass)

This is the difference between "I have a homelab" and "I understand networking."

---

## 🏗️ New Architecture

### Hardware Strategy

**Option 1: Dedicated OpenWRT Router (Recommended)**
- Get a cheap travel router that supports OpenWRT
- Examples: GL.iNet GL-AXT1800, GL-MT3000, Netgear R7800
- ~$50-150 depending on model
- Sits between your ISP modem and your network

**Option 2: Raspberry Pi as OpenWRT Router**
- Use your existing Raspberry Pi OR get a second one
- Add USB ethernet adapter for WAN port
- More limited but totally doable
- Great for learning, maybe not daily driver

**Option 3: Virtual OpenWRT (Advanced)**
- Run OpenWRT in VM on Mac Mini
- USB ethernet adapters for physical ports
- Most flexible for lab work
- Can be tricky with networking

**My recommendation: Start with dedicated hardware (Option 1) + keep Pi for Pi-hole**

### New Network Topology

```
Internet → ISP Modem → [OpenWRT Router] → Network Switch → Devices
                              ↓
                      ┌───────┴────────┐
                      │                │
                 [Pi-hole]        [Mac Mini]
                 DNS/DHCP         Portainer
                      │                │
                  Tailscale        Tailscale
```

**VLANs (Network Segmentation):**
```
VLAN 10: Management (OpenWRT, Pi-hole, Mac Mini)
VLAN 20: Trusted (MacBook, personal devices)
VLAN 30: IoT (smart home devices - isolated)
VLAN 40: Lab (testing new stuff - isolated)
VLAN 50: Guest (visitors - internet only)
```

---

## 📦 Updated Hardware Manifest

**In GeeekPi T1 Rack:**
- Mac Mini M4 (Portainer, containers)
- Raspberry Pi (Pi-hole DNS)

**Network Infrastructure:**
- OpenWRT router (new purchase)
- ISP modem/router (in bridge mode)

**Separate:**
- HP Omen 16 (game streaming host)
- MacBook M4 Air (client device)

**Optional Add-ons:**
- Managed switch (for VLANs) - TP-Link TL-SG108E (~$40)
- Second Raspberry Pi (for redundancy/learning)

---

# PHASE 0: HARDWARE SELECTION & PURCHASE

## Your Hardware (Already Owned)

**✅ GL.iNet GL-MT3000 (Beryl AX)**
- MediaTek MT7981B CPU (dual-core 1.3GHz)
- 512MB RAM / 256MB NAND
- WiFi 6 (AX3000) - 2.4GHz + 5GHz
- 2x Gigabit Ethernet ports + 1x USB 3.0
- Already runs OpenWRT 22.03+
- **Perfect for this project** - one of the best travel routers

**✅ MikroTik Managed Switch (which model?)**
- Enterprise-grade VLAN support
- RouterOS configuration
- Perfect for homelab

**✅ Nvidia Shield Pro 2019**
- **Primary F1 TV streaming device**
- Tegra X1+ processor
- 4K HDR, Dolby Vision/Atmos
- AI upscaling
- **THIS is your VPN bypass device** (needs direct connection)
- Wired ethernet strongly recommended for streaming

**In GeeekPi T1 Rack:**
- Mac Mini M4
- Raspberry Pi

**Separate:**
- HP Omen 16 (game streaming host)
- MacBook M4 Air (client device)

**This hardware combination is excellent for the project.**
- Enterprise-grade VLAN support
- WAY better than TP-Link consumer stuff
- RouterOS configuration (we'll use this)
- Perfect for homelab

**This is actually BETTER gear than the budget recommendation. MikroTik switches are what pros use.**

---

# PHASE 1: OPENWRT SETUP

## GL-MT3000 Specific Notes

**Your MT3000 is excellent for this project. Here's what makes it great:**

- **Dual-core MediaTek MT7981B** - Plenty of power for VPN routing
- **Hardware NAT offloading** - Full gigabit speeds even with VLANs
- **512MB RAM** - Enough for OpenWRT + packages
- **WiFi 6 (AX3000)** - 574Mbps @ 2.4GHz + 2402Mbps @ 5GHz
- **USB 3.0 port** - Can run Pi-hole on USB drive if needed
- **OpenWRT 22.03+** - Pre-installed, just needs configuration

**Default Access:**
- IP: `192.168.8.1`
- Username: `root`
- Password: (set on first boot)
- WiFi SSID: `GL-MT3000-XXX`

**Performance Expectations:**
- VPN throughput: ~300-500 Mbps (WireGuard)
- Routing capacity: ~940 Mbps (NAT offloaded)
- Max clients: 50+ devices easily
- Power draw: ~5W (USB-C powered, great for travel)

## Step 1.1: Get OpenWRT Running

### If Using GL.iNet MT3000 (Your Router - Easy Mode)

**1. Unbox and power on:**
- Use included USB-C power adapter (5V/3A)
- Or power from USB-C port on Mac Mini (travel setup)

**2. Connect to it:**
- **WiFi:** Look for "GL-MT3000-XXXX" network
  - Password: `goodlife` (default)
- **OR** plug ethernet from MacBook to **LAN port**
  - WAN port has icon showing "internet" - don't use for setup

**3. Access web interface:**
- Open browser to: `http://192.168.8.1`
- **First time setup:**
  - Choose language: English
  - Set admin password (**WRITE IT DOWN**)
  - Time zone: Select yours
  - Click "Apply"

**4. Check firmware version:**
- System → Upgrade
- Should be on OpenWRT 22.03 or later
- If update available, install it now
- **DO NOT** use GL.iNet's auto-update - can break custom config

**5. Switch to full OpenWRT interface:**

The GL.iNet has a custom UI, but we want raw OpenWRT for full control:

- Click "More Settings" (bottom left)
- Scroll down to "Advanced"
- Toggle "Enable OpenWRT Web Interface"
- OR just go directly to: `http://192.168.8.1/cgi-bin/luci`

**6. Login to LuCI (OpenWRT web interface):**
- Username: `root`
- Password: (the one you just set)

You now have full OpenWRT access!

**7. Important MT3000 Settings:**

Network → Interfaces → LAN

```
Protocol: Static address
IPv4 address: 192.168.8.1  (change this later to 10.69.0.1)
IPv4 netmask: 255.255.255.0
IPv4 gateway: (leave blank for now)
```

**8. Enable hardware NAT offloading (critical for performance):**

Network → Firewall → Custom Rules

Add at the bottom:
```bash
# Enable hardware flow offloading
echo 1 > /sys/kernel/debug/hnat/hnat_entry
```

Or via SSH:
```bash
ssh root@192.168.8.1

# Enable hardware offloading permanently
uci set firewall.@defaults[0].flow_offloading='1'
uci set firewall.@defaults[0].flow_offloading_hw='1'
uci commit firewall
/etc/init.d/firewall restart
```

This gives you full gigabit speeds even with VLANs!

**9. Configure WiFi (optional optimization):**

Network → Wireless

**2.4GHz radio:**
- Mode: N
- Width: 40MHz
- Channel: 1, 6, or 11 (avoid auto)

**5GHz radio:**
- Mode: AC+AX
- Width: 80MHz
- Channel: 36, 44, 149, or 157
- **For travel:** Use 36-48 (universally allowed)

**10. USB 3.0 port (advanced option):**

The MT3000 has a USB port. You can:
- Plug in USB storage for extra space
- Run Pi-hole on USB drive (if not using separate Pi)
- USB tethering for failover internet

**11. Test internet connection:**

- Plug **WAN port** into your modem
- Plug **LAN port** to your MacBook
- Can you browse? If yes, basic setup is done!

You now have OpenWRT running perfectly!

### If Using Other Hardware

**1. Flash OpenWRT:**
- Find your router at: https://openwrt.org/toh/start
- Download firmware image
- Flash via manufacturer's web interface
- Wait 5-10 minutes
- Router reboots

**2. First access:**
- Connect ethernet to LAN port
- Go to: `http://192.168.1.1`
- No password set initially
- Click "Login"

**3. Set root password:**
- System → Administration
- Set password (WRITE IT DOWN)
- Save & Apply

---

## Step 1.2: Initial Configuration

**1. Connect to internet:**
- Network → Interfaces
- Click "Edit" on WAN interface
- Protocol: DHCP (or PPPoE if your ISP requires)
- Save & Apply
- Plug WAN port into your modem

**2. Update package lists:**

SSH into router:
```bash
ssh root@192.168.8.1
# Enter password you just set

# Update packages
opkg update
```

**3. Install essential packages:**

```bash
# Web interface improvements
opkg install luci-ssl luci-app-statistics

# Network tools
opkg install tcpdump iperf3 netcat

# VLAN support
opkg install luci-app-vlan

# AdBlock (OpenWRT-level blocking)
opkg install luci-app-adblock

# VPN support
opkg install wireguard-tools luci-app-wireguard

# DNS improvements
opkg install dnsmasq-full --force-overwrite
```

**4. Reboot:**
```bash
reboot
```

---

## Step 1.3: Basic Security Hardening

**1. Disable WAN SSH access:**

```bash
# Edit firewall rules
uci set firewall.@zone[1].input='REJECT'
uci commit firewall
/etc/init.d/firewall restart
```

**2. Set up SSH keys (no more passwords):**

On your MacBook:
```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to OpenWRT
ssh-copy-id root@192.168.8.1
```

**3. Disable password authentication:**

```bash
# On OpenWRT
uci set dropbear.@dropbear[0].PasswordAuth='off'
uci set dropbear.@dropbear[0].RootPasswordAuth='off'
uci commit dropbear
/etc/init.d/dropbear restart
```

**4. Change default LAN subnet (optional security through obscurity):**

```bash
uci set network.lan.ipaddr='10.69.0.1'
uci commit network
/etc/init.d/network restart
```

Now your router is at `10.69.0.1` instead of `192.168.x.x`.

---

# PHASE 2: VLAN CONFIGURATION

## Step 2.1: Plan Your VLANs

```
VLAN 10 (10.69.10.0/24): Management
  - OpenWRT (10.69.10.1)
  - Pi-hole (10.69.10.2)
  - Mac Mini (10.69.10.3)
  - Managed switch (10.69.10.4)

VLAN 20 (10.69.20.0/24): Trusted Devices
  - Your MacBook
  - Your phone
  - Trusted computers

VLAN 30 (10.69.30.0/24): IoT
  - Smart home devices
  - Cameras
  - Can't talk to other VLANs

VLAN 40 (10.69.40.0/24): Lab
  - Test VMs
  - Experimental stuff
  - Isolated from production

VLAN 50 (10.69.50.0/24): Guest
  - Visitors
  - Internet only
  - Can't see anything else
```

## Step 2.2: Create VLANs in OpenWRT

**1. In OpenWRT web interface:**

Network → Switch → Add VLAN

For each VLAN:
- VLAN ID: 10, 20, 30, 40, 50
- CPU port: tagged
- LAN ports: Configure based on your needs

**2. Create interfaces:**

Network → Interfaces → Add new interface

**Management VLAN (10):**
```
Name: mgmt
Protocol: Static address
IP: 10.69.10.1
Netmask: 255.255.255.0
DHCP: Ignore interface (Pi-hole will handle)
```

**Trusted VLAN (20):**
```
Name: trusted
Protocol: Static address
IP: 10.69.20.1
Netmask: 255.255.255.0
DHCP Server: Enabled
  Start: 10.69.20.100
  Limit: 150
  Lease time: 12h
DNS: 10.69.10.2 (Pi-hole)
```

Repeat for VLANs 30, 40, 50 with their respective IPs.

---

## Step 2.2b: Configure MikroTik Switch for VLANs

**IMPORTANT:** MikroTik switches are way more powerful than consumer gear, but configuration is different. Here's how to set it up properly.

### Access Your MikroTik Switch

**Option 1: Web Interface (WebFig)**
1. Connect to switch via ethernet
2. Open browser to `http://192.168.88.1` (default MikroTik IP)
3. Login: admin / (no password by default)

**Option 2: WinBox (Recommended)**
1. Download WinBox from mikrotik.com/download
2. Open WinBox, click "Neighbors"
3. Select your switch, connect

**Option 3: SSH/Terminal**
```bash
ssh admin@192.168.88.1
```

### Initial Setup

**1. Set admin password (CRITICAL):**

```bash
/user set admin password=YourStrongPassword
```

**2. Set switch IP to fit your network:**

```bash
/ip address remove [find]
/ip address add address=10.69.10.4/24 interface=bridge
/ip route add gateway=10.69.10.1
```

Now access at: `http://10.69.10.4`

### VLAN Configuration on MikroTik

**What model MikroTik do you have?** This matters because:
- **CRS3xx series** - Use bridge VLAN filtering (modern method)
- **CRS1xx/2xx series** - May need switch chip configuration
- **CSS series** - SwOS only (limited features)

I'll show you the **bridge VLAN filtering** method (works on most models):

### Method 1: Bridge VLAN Filtering (Modern - CRS3xx, RB5009, etc.)

**1. Create bridge:**

```bash
/interface bridge
add name=bridge1 vlan-filtering=no
```

**2. Add all ports to bridge:**

```bash
/interface bridge port
add bridge=bridge1 interface=ether1 pvid=10
add bridge=bridge1 interface=ether2 pvid=20
add bridge=bridge1 interface=ether3 pvid=20
add bridge=bridge1 interface=ether4 pvid=30
add bridge=bridge1 interface=ether5 pvid=40
add bridge=bridge1 interface=ether6 pvid=50
add bridge=bridge1 interface=sfp1 pvid=10
```

Port assignment example:
- ether1: Management VLAN (trunk to OpenWRT)
- ether2-3: Trusted devices
- ether4: IoT devices
- ether5: Lab devices
- ether6: Guest WiFi AP
- sfp1: Uplink to OpenWRT (trunk all VLANs)

**3. Configure VLAN table:**

```bash
/interface bridge vlan
add bridge=bridge1 tagged=ether1,sfp1 untagged=ether1 vlan-ids=10
add bridge=bridge1 tagged=ether1,sfp1 untagged=ether2,ether3 vlan-ids=20
add bridge=bridge1 tagged=ether1,sfp1 untagged=ether4 vlan-ids=30
add bridge=bridge1 tagged=ether1,sfp1 untagged=ether5 vlan-ids=40
add bridge=bridge1 tagged=ether1,sfp1 untagged=ether6 vlan-ids=50
```

**4. Enable VLAN filtering (THE MOMENT OF TRUTH):**

```bash
/interface bridge set bridge1 vlan-filtering=yes
```

**⚠️ WARNING:** Test this carefully! If you lock yourself out:
- Enable Safe Mode first: Ctrl+X in terminal
- Or use the switch's reset button

**5. Set ingress filtering:**

```bash
/interface bridge port
set [find interface=ether1] frame-types=admit-only-vlan-tagged
set [find interface=ether2] frame-types=admit-only-untagged-and-priority-tagged
set [find interface=ether3] frame-types=admit-only-untagged-and-priority-tagged
set [find interface=ether4] frame-types=admit-only-untagged-and-priority-tagged
set [find interface=ether5] frame-types=admit-only-untagged-and-priority-tagged
set [find interface=ether6] frame-types=admit-only-untagged-and-priority-tagged
set [find interface=sfp1] frame-types=admit-only-vlan-tagged
```

**6. Configure trunk port to OpenWRT:**

```bash
/interface bridge port
set [find interface=sfp1] pvid=10
```

### Method 2: Hardware Switch Configuration (Older CRS1xx/2xx)

**If your switch has `/interface ethernet switch` in the CLI:**

**1. Enable VLAN mode:**

```bash
/interface ethernet switch
set switch1 vlan-mode=secure
```

**2. Configure VLANs:**

```bash
/interface ethernet switch vlan
add ports=ether1,ether2,ether3,switch1-cpu vlan-id=10
add ports=ether1,ether4,ether5,switch1-cpu vlan-id=20
add ports=ether1,ether6,switch1-cpu vlan-id=30
add ports=ether1,ether7,switch1-cpu vlan-id=40
add ports=ether1,ether8,switch1-cpu vlan-id=50
```

**3. Set port PVID:**

```bash
/interface ethernet switch port
set ether2 vlan-mode=secure vlan-header=add-if-missing default-vlan-id=10
set ether3 vlan-mode=secure vlan-header=add-if-missing default-vlan-id=10
set ether4 vlan-mode=secure vlan-header=add-if-missing default-vlan-id=20
# etc...
```

### Visual Configuration via WebFig

**1. Bridge → VLANs:**

| VLAN ID | Tagged Ports | Untagged Ports |
|---------|-------------|----------------|
| 10 | sfp1 | ether1 |
| 20 | sfp1 | ether2, ether3 |
| 30 | sfp1 | ether4 |
| 40 | sfp1 | ether5 |
| 50 | sfp1 | ether6 |

**2. Bridge → Ports:**

| Port | PVID | Frame Types |
|------|------|-------------|
| sfp1 | 10 | admit-only-vlan-tagged |
| ether1 | 10 | admit-only-untagged |
| ether2 | 20 | admit-only-untagged |
| ether3 | 20 | admit-only-untagged |
| ether4 | 30 | admit-only-untagged |
| ether5 | 40 | admit-only-untagged |
| ether6 | 50 | admit-only-untagged |

### Testing VLANs on MikroTik

**1. Check VLAN table:**

```bash
/interface bridge vlan print
```

**2. Monitor traffic:**

```bash
/interface bridge monitor-traffic bridge1
```

**3. Test connectivity:**

Plug a device into ether2 (VLAN 20):
- Should get IP from 10.69.20.x range
- Should be able to ping 10.69.20.1 (OpenWRT)
- Should NOT be able to ping 10.69.30.x (IoT VLAN)

### Physical Cabling

```
ISP Modem
    ↓
OpenWRT Router (GL.iNet)
    ↓ (SFP or Ethernet - TRUNK all VLANs)
MikroTik Switch (port sfp1 or ether1)
    ↓
    ├─ ether2 → Mac Mini (VLAN 10 Management)
    ├─ ether3 → Raspberry Pi (VLAN 10 Management)
    ├─ ether4 → MacBook (VLAN 20 Trusted)
    ├─ ether5 → Smart TV (VLAN 30 IoT)
    ├─ ether6 → Lab Machine (VLAN 40 Lab)
    └─ ether7 → Guest AP (VLAN 50 Guest)
```

### Backup Your MikroTik Config

**After configuration:**

```bash
/export file=mikrotik-vlan-config
```

Download via WebFig: Files → mikrotik-vlan-config.rsc

**To restore:**

```bash
/import file=mikrotik-vlan-config.rsc
```

### Common MikroTik Issues & Fixes

**Problem: Locked yourself out after enabling VLAN filtering**

**Solution:**
1. Hold reset button 5-10 seconds (resets to defaults)
2. OR: Use Netinstall to recover
3. OR: Enable Safe Mode before making changes (Ctrl+X in terminal)

**Problem: Devices not getting IPs**

**Solution:**
```bash
# Check bridge VLAN filtering is working
/interface bridge vlan print

# Verify PVID is set
/interface bridge port print

# Check if DHCP is reaching devices
/tool sniffer quick interface=bridge1
```

**Problem: Inter-VLAN routing not working**

**Solution:** This is controlled by OpenWRT firewall, not the switch. Switch only does VLAN separation. Check OpenWRT firewall rules.

### MikroTik Management VLAN Best Practice

**Create management VLAN interface:**

```bash
/interface vlan
add interface=bridge1 vlan-id=10 name=vlan10-mgmt

/ip address
add address=10.69.10.4/24 interface=vlan10-mgmt

/ip route
add gateway=10.69.10.1
```

Now switch is ONLY accessible on VLAN 10 (more secure).

### Advanced: MikroTik Port Mirroring (For Debugging)

**Mirror traffic to analyze:**

```bash
# Mirror ether2 traffic to ether8 (for packet capture)
/interface ethernet switch
set switch1 mirror-source=ether2 mirror-target=ether8
```

Plug laptop into ether8, run Wireshark, see all traffic from ether2.

---

## Step 2.3: Configure Firewall Rules

**1. Basic inter-VLAN rules:**

Network → Firewall → General Settings

**Zones:**
```
Management (VLAN 10):
  Input: ACCEPT
  Output: ACCEPT
  Forward: ACCEPT

Trusted (VLAN 20):
  Input: ACCEPT
  Output: ACCEPT
  Forward: ACCEPT

IoT (VLAN 30):
  Input: REJECT
  Output: ACCEPT
  Forward: REJECT

Lab (VLAN 40):
  Input: REJECT
  Output: ACCEPT
  Forward: REJECT

Guest (VLAN 50):
  Input: REJECT
  Output: ACCEPT
  Forward: REJECT
```

**2. Allow specific access:**

Network → Firewall → Traffic Rules

**Allow Trusted → Management:**
```
Source zone: trusted
Destination zone: mgmt
Action: ACCEPT
```

**Allow IoT → Internet only:**
```
Source zone: iot
Destination zone: wan
Action: ACCEPT
```

**Block IoT → Everything else:**
```
Source zone: iot
Destination zone: mgmt, trusted, lab, guest
Action: REJECT
```

---

# PHASE 3: PI-HOLE INTEGRATION

## Step 3.1: Reconfigure Pi-hole for VLANs

**1. Give Pi-hole static IP in Management VLAN:**

On Raspberry Pi:
```bash
sudo nano /etc/dhcpcd.conf
```

Add:
```
interface eth0
static ip_address=10.69.10.2/24
static routers=10.69.10.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

Reboot Pi.

**2. Configure Pi-hole to listen on all interfaces:**

```bash
pihole -a -i all
```

**3. Set up Pi-hole as DHCP server (replaces OpenWRT DHCP):**

Pi-hole web interface → Settings → DHCP

**For each VLAN:**
```
VLAN 20 (Trusted):
  Range: 10.69.20.100 - 10.69.20.250
  Router: 10.69.20.1
  DNS: 10.69.10.2 (itself)
```

**4. Disable DHCP on OpenWRT:**

For each VLAN interface:
- Network → Interfaces → Edit
- DHCP Server tab
- Click "Disable DHCP for this interface"

---

# PHASE 4: CAPTIVE PORTAL AUTOMATION

## The Hotel WiFi Problem

When you're traveling, hotel WiFi sucks because:
1. Captive portals require browser login
2. Can't connect homelab devices easily
3. Your devices expect your home network

**Solution: Automated captive portal bypass + travel router mode**

## Step 4.1: Create Travel Mode Script

**1. Create the script on OpenWRT:**

```bash
ssh root@192.168.8.1
nano /root/hotel-wifi.sh
```

**2. Add this script:**

```bash
#!/bin/sh

# Hotel WiFi Auto-Login Script
# Detects captive portals and attempts automated login

HOTEL_WIFI="$1"  # SSID of hotel WiFi
LOGFILE="/tmp/hotel-wifi.log"

log() {
    echo "[$(date)] $1" >> $LOGFILE
}

# Connect to hotel WiFi
log "Connecting to $HOTEL_WIFI..."
uci set wireless.@wifi-iface[0].ssid="$HOTEL_WIFI"
uci set wireless.@wifi-iface[0].encryption='none'
uci commit wireless
wifi reload

sleep 10

# Test for captive portal
PORTAL_TEST=$(curl -s -I http://captive.apple.com | grep "HTTP")
log "Portal test result: $PORTAL_TEST"

if echo "$PORTAL_TEST" | grep -q "302\|301"; then
    log "Captive portal detected! Attempting auto-login..."
    
    # Get redirect URL
    PORTAL_URL=$(curl -s -I http://captive.apple.com | grep -i "Location:" | cut -d' ' -f2 | tr -d '\r')
    log "Portal URL: $PORTAL_URL"
    
    # Common hotel portal types
    
    # Hilton
    if echo "$PORTAL_URL" | grep -q "hilton"; then
        log "Detected Hilton portal"
        curl -X POST "$PORTAL_URL" \
            -d "roomNumber=&lastName=&acceptTerms=on" \
            -H "Content-Type: application/x-www-form-urlencoded"
    fi
    
    # Marriott
    if echo "$PORTAL_URL" | grep -q "marriott"; then
        log "Detected Marriott portal"
        curl -X POST "$PORTAL_URL" \
            -d "accept=true&provider=marriott" \
            -H "Content-Type: application/x-www-form-urlencoded"
    fi
    
    # Generic "Accept Terms" portal
    if echo "$PORTAL_URL" | grep -q "terms\|accept\|agree"; then
        log "Attempting generic portal bypass"
        curl -X POST "$PORTAL_URL" \
            -d "accept=true&terms=on&agree=yes" \
            -H "Content-Type: application/x-www-form-urlencoded"
    fi
    
    # Last resort: open in browser
    log "Manual intervention may be required"
    echo "Open browser to: $PORTAL_URL"
    
else
    log "No captive portal detected, connection successful!"
fi

# Verify internet connectivity
if ping -c 3 8.8.8.8 > /dev/null 2>&1; then
    log "Internet connectivity verified!"
    
    # Start Tailscale to connect home
    /etc/init.d/tailscale start
    log "Tailscale started, homelab should be accessible"
else
    log "ERROR: No internet connectivity"
fi
```

**3. Make it executable:**

```bash
chmod +x /root/hotel-wifi.sh
```

**4. Usage when traveling:**

```bash
ssh root@192.168.8.1
/root/hotel-wifi.sh "Marriott_Bonvoy_Guest"
```

---

## Step 4.2: Advanced: Policy-Based VPN Routing with F1 TV Bypass

**The Real Problem:** 
- Travel router = ONE device to hotel (bypasses device limits already!)
- F1 TV has aggressive VPN/geolocation detection
- Some devices need VPN (privacy), others need direct connection (streaming)

**The Solution:** Policy-based routing with selective VPN tunneling

### Architecture Overview

```
Hotel WiFi → OpenWRT Router → Your Devices
                    ↓
            ┌───────┴────────┐
            │                │
      [VPN Tunnel]    [Direct Connection]
    (Most traffic)      (F1 TV only)
            │                │
        Mullvad/          F1 TV
        ProtonVPN         Streaming
```

### Part A: Install VPN Client on OpenWRT

You have several VPN options. I recommend **Mullvad** or **ProtonVPN** for streaming:
- Fast servers
- Port forwarding support
- Good with streaming services when needed

**1. Install WireGuard (modern, fast):**

```bash
ssh root@192.168.8.1

opkg update
opkg install wireguard-tools luci-app-wireguard luci-proto-wireguard kmod-wireguard
```

**2. Install OpenVPN (fallback if WireGuard detected):**

```bash
opkg install openvpn-openssl luci-app-openvpn
```

**3. For this guide, we'll use WireGuard (faster for streaming):**

Get your VPN provider's WireGuard config:
- **Mullvad**: Account → WireGuard configuration → Download
- **ProtonVPN**: Downloads → WireGuard configuration
- **NordVPN**: Manual setup → WireGuard

---

### Part B: Configure WireGuard Interface

**1. Create WireGuard interface in OpenWRT:**

Network → Interfaces → Add new interface

```
Name: vpn
Protocol: WireGuard VPN
```

**2. Configure the interface:**

General Settings:
```
Private Key: [from your VPN provider config]
Listen Port: 51820
IP Address: [from VPN config, usually 10.x.x.x/32]
```

**3. Add VPN peer:**

Peers tab → Add peer

```
Public Key: [VPN server public key]
Allowed IPs: 0.0.0.0/0, ::/0
Endpoint Host: [VPN server address]
Endpoint Port: 51820
Persistent Keep Alive: 25
Route Allowed IPs: checked
```

**4. Firewall settings:**

Assign to firewall zone: `wan` (so it's treated as internet)

Save & Apply.

**5. Test VPN connection:**

```bash
# Check if interface is up
ifconfig wg0

# Test connectivity
ping -I wg0 1.1.1.1

# Check public IP
curl --interface wg0 ifconfig.me
# Should show VPN server IP
```

---

### Part C: Policy-Based Routing - The Magic Part

**This is where it gets cool:** Route specific devices or traffic through VPN, everything else direct.

**1. Install policy routing package:**

```bash
opkg update
opkg install ip-full ipset luci-app-mwan3
```

**2. Create routing tables:**

```bash
# Edit routing table names
echo "200 vpn" >> /etc/iproute2/rt_tables
echo "201 direct" >> /etc/iproute2/rt_tables
```

**3. Create firewall marks for traffic classification:**

Network → Firewall → Custom Rules

```bash
# Mark traffic destined for F1 TV (bypass VPN)
ipset create f1tv hash:ip timeout 0
ipset add f1tv 151.101.0.0/16    # F1 TV CDN
ipset add f1tv 104.16.0.0/13     # Cloudflare (F1 uses)

# Route F1 TV traffic directly
iptables -t mangle -A PREROUTING -m set --match-set f1tv dst -j MARK --set-mark 201

# All other traffic through VPN
iptables -t mangle -A PREROUTING -m mark --mark 0 -j MARK --set-mark 200
```

**4. Create routing rules:**

```bash
# VPN traffic
ip rule add fwmark 200 table vpn
ip route add default dev wg0 table vpn

# Direct traffic  
ip rule add fwmark 201 table direct
ip route add default via 192.168.8.1 table direct
```

---

### Part D: Device-Based Routing (Even Better)

**Route by device instead of destination:**

**Option 1: By IP Address**

Assign static IPs to devices:
- MacBook: 10.69.20.10 → VPN
- **Nvidia Shield: 10.69.20.15 → Direct (F1 TV)**
- iPad: 10.69.20.20 → Direct or VPN (your choice)
- Phone: 10.69.20.30 → VPN

```bash
# MacBook through VPN
iptables -t mangle -A PREROUTING -s 10.69.20.10 -j MARK --set-mark 200

# Nvidia Shield direct (F1 TV streaming device)
iptables -t mangle -A PREROUTING -s 10.69.20.15 -j MARK --set-mark 201

# iPad - your choice based on usage
iptables -t mangle -A PREROUTING -s 10.69.20.20 -j MARK --set-mark 201

# Phone through VPN
iptables -t mangle -A PREROUTING -s 10.69.20.30 -j MARK --set-mark 200
```

**Option 2: By MAC Address (more flexible)**

```bash
# MacBook (by MAC) through VPN
iptables -t mangle -A PREROUTING -m mac --mac-source AA:BB:CC:DD:EE:FF -j MARK --set-mark 200

# Nvidia Shield (by MAC) direct - GET MAC FROM SHIELD SETTINGS
iptables -t mangle -A PREROUTING -m mac --mac-source 11:22:33:44:55:66 -j MARK --set-mark 201
```

**To find Shield's MAC address:**
1. On Shield: Settings → Device Preferences → About → Status
2. Look for "WiFi MAC address" or "Ethernet MAC address"
3. Use the **ethernet** MAC if Shield is wired (recommended for streaming)

---

### Part D.5: Nvidia Shield Specific Optimizations

**Your Shield Pro is the perfect F1 TV device. Let's optimize it.**

**1. Reserve static IP for Shield in OpenWRT:**

Network → DHCP and DNS → Static Leases

```
Hostname: shield
MAC Address: [Shield's MAC]
IPv4 Address: 10.69.20.15
```

**2. Prioritize Shield traffic (QoS):**

```bash
# Mark Shield traffic as high priority
iptables -t mangle -A POSTROUTING -s 10.69.20.15 -j DSCP --set-dscp 46

# Or use Shield's MAC
iptables -t mangle -A POSTROUTING -m mac --mac-source [SHIELD_MAC] -j DSCP --set-dscp 46
```

**3. Disable IPv6 for Shield (prevents geo-location issues):**

```bash
# Block IPv6 traffic from Shield
ip6tables -A FORWARD -s 10.69.20.15 -j DROP
```

Or in OpenWRT web interface:
- Network → Firewall → Custom Rules
- Add: `ip6tables -A FORWARD -s 10.69.20.15 -j DROP`

**4. Force specific DNS for F1 TV (on Shield):**

Create DNS override specifically for Shield:

```bash
# Edit /etc/config/dhcp
config dhcp-option
    option name 'shield-dns'
    option value '6,1.1.1.1,8.8.8.8'

# Apply to Shield's lease
config host
    option name 'shield'
    option mac '[SHIELD_MAC]'
    option ip '10.69.20.15'
    option dhcp_option 'shield-dns'
```

This ensures Shield uses Cloudflare/Google DNS directly (not Pi-hole) for F1 TV.

**5. Ethernet strongly recommended:**

Shield Pro has gigabit ethernet - USE IT for F1 TV:
- WiFi: Great for most apps
- **Ethernet: Required for 4K/50fps F1 TV streams**

Connect Shield to MikroTik switch port assigned to VLAN 20 (Trusted).

**6. Shield-specific firewall rules:**

Allow Shield unrestricted access (it's on direct connection anyway):

```bash
# Allow all Shield traffic out
iptables -A FORWARD -s 10.69.20.15 -j ACCEPT

# No restrictions
iptables -A FORWARD -d 10.69.20.15 -j ACCEPT
```

**7. Test Shield streaming before race day:**

```bash
# On OpenWRT, verify Shield is taking direct path:
tcpdump -i br-lan host 10.69.20.15

# Should NOT see traffic on wg0 (VPN interface)
tcpdump -i wg0 host 10.69.20.15
# (Should show nothing)
```

**8. Alternative: Create dedicated "Streaming" VLAN:**

If you want to isolate all streaming devices:

```
VLAN 25: Streaming (Shield, Apple TV, etc.)
  - Always direct connection
  - QoS priority
  - No VPN ever
  - Optimized DNS
```

In `/root/vpn-policy.sh`, add:

```bash
# VLAN 25 - Streaming devices (always direct)
STREAMING_VLAN="10.69.25.0/24"
iptables -t mangle -A PREROUTING -s $STREAMING_VLAN -j MARK --set-mark 201
```

---

### Part E: Automated Setup Script

**Create a master VPN control script:**

```bash
nano /root/vpn-policy.sh
```

```bash
#!/bin/sh

# VPN Policy Routing Controller
# Manages which devices/services use VPN vs direct connection

MODE="$1"  # travel, home, f1-race

log() {
    logger -t vpn-policy "$1"
    echo "[$(date)] $1"
}

setup_routing_tables() {
    # Ensure routing tables exist
    grep -q "vpn" /etc/iproute2/rt_tables || echo "200 vpn" >> /etc/iproute2/rt_tables
    grep -q "direct" /etc/iproute2/rt_tables || echo "201 direct" >> /etc/iproute2/rt_tables
    
    # Set up routes
    ip route flush table vpn 2>/dev/null
    ip route add default dev wg0 table vpn
    
    ip route flush table direct 2>/dev/null  
    ip route add default via $(ip route | grep default | awk '{print $3}') table direct
    
    # Set up rules
    ip rule del fwmark 200 2>/dev/null
    ip rule add fwmark 200 table vpn
    
    ip rule del fwmark 201 2>/dev/null
    ip rule add fwmark 201 table direct
}

setup_f1tv_bypass() {
    log "Setting up F1 TV bypass..."
    
    # Create ipset for F1 TV IPs
    ipset create f1tv hash:ip timeout 0 2>/dev/null
    ipset flush f1tv
    
    # F1 TV known IPs/ranges
    ipset add f1tv 151.101.0.0/16    # Fastly CDN
    ipset add f1tv 104.16.0.0/13     # Cloudflare
    ipset add f1tv 13.224.0.0/14     # AWS CloudFront
    
    # Route F1 TV direct
    iptables -t mangle -D PREROUTING -m set --match-set f1tv dst -j MARK --set-mark 201 2>/dev/null
    iptables -t mangle -A PREROUTING -m set --match-set f1tv dst -j MARK --set-mark 201
    
    log "F1 TV will bypass VPN"
}

setup_device_routing() {
    log "Setting up device-based routing..."
    
    # Clear existing rules
    iptables -t mangle -F PREROUTING
    
    # MacBook - Through VPN (privacy)
    MACBOOK_IP="10.69.20.10"
    iptables -t mangle -A PREROUTING -s $MACBOOK_IP -j MARK --set-mark 200
    log "MacBook ($MACBOOK_IP) → VPN"
    
    # Nvidia Shield - Direct (F1 TV streaming)
    SHIELD_IP="10.69.20.15"
    iptables -t mangle -A PREROUTING -s $SHIELD_IP -j MARK --set-mark 201
    log "Nvidia Shield ($SHIELD_IP) → Direct (F1 TV optimized)"
    
    # Phone - Through VPN (privacy)
    PHONE_IP="10.69.20.30"
    iptables -t mangle -A PREROUTING -s $PHONE_IP -j MARK --set-mark 200
    log "Phone ($PHONE_IP) → VPN"
    
    # HP Omen - Configurable (gaming vs streaming)
    OMEN_IP="10.69.20.40"
    if [ "$GAMING_MODE" = "true" ]; then
        # Gaming: Direct connection for lower latency
        iptables -t mangle -A PREROUTING -s $OMEN_IP -j MARK --set-mark 201
        log "HP Omen ($OMEN_IP) → Direct (Gaming Mode)"
    else
        # Normal: Through VPN
        iptables -t mangle -A PREROUTING -s $OMEN_IP -j MARK --set-mark 200
        log "HP Omen ($OMEN_IP) → VPN"
    fi
    
    # Default - Everything else through VPN
    iptables -t mangle -A PREROUTING -m mark --mark 0 -j MARK --set-mark 200
}

enable_vpn() {
    log "Enabling VPN interface..."
    ifup vpn
    sleep 5
    
    # Verify VPN is up
    if ifconfig wg0 &>/dev/null; then
        VPN_IP=$(curl --interface wg0 -s ifconfig.me)
        log "VPN connected! Public IP: $VPN_IP"
        return 0
    else
        log "ERROR: VPN failed to connect"
        return 1
    fi
}

disable_vpn() {
    log "Disabling VPN..."
    ifdown vpn
    
    # Clear routing rules
    iptables -t mangle -F PREROUTING
    ip rule flush
    
    log "VPN disabled, all traffic direct"
}

# Main logic
case $MODE in
    travel)
        log "=== TRAVEL MODE ==="
        enable_vpn || exit 1
        setup_routing_tables
        setup_f1tv_bypass
        setup_device_routing
        log "Travel mode active: VPN with F1 TV bypass"
        ;;
        
    f1-race)
        log "=== F1 RACE MODE ==="
        log "Disabling VPN for optimal F1 TV streaming"
        disable_vpn
        ;;
        
    home)
        log "=== HOME MODE ==="
        disable_vpn
        log "Using home network (Tailscale to homelab)"
        ;;
        
    status)
        echo "=== VPN Status ==="
        echo "WireGuard interface:"
        ifconfig wg0 2>/dev/null || echo "  Not connected"
        echo ""
        echo "Public IP:"
        curl -s ifconfig.me
        echo ""
        echo "Routing rules:"
        ip rule show
        echo ""
        echo "F1 TV bypass IPs:"
        ipset list f1tv 2>/dev/null || echo "  Not configured"
        ;;
        
    *)
        echo "Usage: $0 {travel|f1-race|home|status}"
        echo ""
        echo "  travel   - Enable VPN with F1 TV bypass"
        echo "  f1-race  - Disable VPN for race day"  
        echo "  home     - Disable VPN (use Tailscale)"
        echo "  status   - Show current configuration"
        exit 1
        ;;
esac
```

**Make executable:**

```bash
chmod +x /root/vpn-policy.sh
```

---

### Part F: Usage

**When traveling (hotel):**

```bash
ssh root@192.168.8.1
/root/vpn-policy.sh travel
```

Now:
- Your MacBook, phone → encrypted through VPN
- F1 TV traffic → direct (no geo-blocking issues)
- Gaming traffic → you choose (can add to direct for lower latency)

**Race day (maximum performance):**

```bash
/root/vpn-policy.sh f1-race
```

All VPN disabled, everything direct for best streaming quality.

**Back home:**

```bash
/root/vpn-policy.sh home
```

VPN off, use Tailscale to access homelab.

**Check status anytime:**

```bash
/root/vpn-policy.sh status
```

---

### Part G: Web UI Control Panel

**Create simple web interface:**

```bash
nano /www/vpn-control.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>VPN Policy Control</title>
    <style>
        body { 
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
            max-width: 800px; 
            margin: 50px auto;
            background: #1a1a1a;
            color: #fff;
        }
        .card {
            background: #2a2a2a;
            border-radius: 10px;
            padding: 20px;
            margin: 20px 0;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        button {
            padding: 15px 30px;
            margin: 10px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
        }
        .btn-travel { background: #4CAF50; color: white; }
        .btn-travel:hover { background: #45a049; }
        .btn-race { background: #ff4444; color: white; }
        .btn-race:hover { background: #cc0000; }
        .btn-home { background: #2196F3; color: white; }
        .btn-home:hover { background: #0b7dda; }
        .btn-status { background: #ffa500; color: white; }
        .btn-status:hover { background: #ff8c00; }
        .status-box {
            background: #000;
            padding: 15px;
            border-radius: 5px;
            font-family: 'Courier New', monospace;
            color: #0f0;
            white-space: pre-wrap;
            max-height: 400px;
            overflow-y: auto;
        }
        h1 { color: #4CAF50; }
        .mode-badge {
            display: inline-block;
            padding: 5px 15px;
            border-radius: 20px;
            font-size: 14px;
            margin-left: 10px;
        }
        .badge-active { background: #4CAF50; }
        .badge-inactive { background: #666; }
    </style>
</head>
<body>
    <h1>🛡️ VPN Policy Controller</h1>
    
    <div class="card">
        <h2>Current Mode: <span id="current-mode" class="mode-badge badge-inactive">Unknown</span></h2>
        <p>Control VPN routing policy for travel and streaming optimization</p>
    </div>
    
    <div class="card">
        <h3>Quick Actions</h3>
        <button class="btn-travel" onclick="setMode('travel')">
            ✈️ Travel Mode<br>
            <small>VPN + F1 TV Bypass</small>
        </button>
        
        <button class="btn-race" onclick="setMode('f1-race')">
            🏎️ Race Day Mode<br>
            <small>All Direct (Best Streaming)</small>
        </button>
        
        <button class="btn-home" onclick="setMode('home')">
            🏠 Home Mode<br>
            <small>VPN Off (Tailscale Only)</small>
        </button>
        
        <button class="btn-status" onclick="checkStatus()">
            📊 Check Status
        </button>
    </div>
    
    <div class="card">
        <h3>Device Routing</h3>
        <div class="status-box" id="device-status">
            Loading device configuration...
        </div>
    </div>
    
    <div class="card">
        <h3>System Output</h3>
        <div class="status-box" id="output">
            Click a mode button to get started.
        </div>
    </div>
    
    <script>
        function setMode(mode) {
            showOutput('Activating ' + mode + ' mode...');
            
            fetch('/cgi-bin/vpn-control.sh?mode=' + mode)
                .then(r => r.text())
                .then(text => {
                    showOutput(text);
                    updateCurrentMode(mode);
                    setTimeout(checkStatus, 2000);
                });
        }
        
        function checkStatus() {
            fetch('/cgi-bin/vpn-control.sh?mode=status')
                .then(r => r.text())
                .then(showOutput);
        }
        
        function showOutput(text) {
            document.getElementById('output').textContent = text;
        }
        
        function updateCurrentMode(mode) {
            const badge = document.getElementById('current-mode');
            badge.textContent = mode.toUpperCase();
            badge.className = 'mode-badge badge-active';
        }
        
        // Auto-refresh status every 30 seconds
        setInterval(checkStatus, 30000);
        
        // Check status on load
        window.onload = checkStatus;
    </script>
</body>
</html>
```

**Create CGI handler:**

```bash
nano /www/cgi-bin/vpn-control.sh
```

```bash
#!/bin/sh

echo "Content-type: text/plain"
echo ""

MODE=$(echo "$QUERY_STRING" | sed 's/mode=//')

/root/vpn-policy.sh "$MODE"
```

```bash
chmod +x /www/cgi-bin/vpn-control.sh
```

**Access at:** `http://192.168.8.1/vpn-control.html`

Now you have a beautiful dashboard to control VPN routing!

---

### Part H: F1 TV Specific Optimizations

**1. DNS Override for F1 TV:**

Some streaming services check DNS. Force F1 TV to use specific DNS:

```bash
# Add to /etc/dnsmasq.conf
server=/formula1.com/1.1.1.1
server=/f1tv.formula1.com/1.1.1.1
```

**2. Disable IPv6 for F1 TV (can cause geo-location issues):**

```bash
# Block IPv6 for F1 TV device
ip6tables -A FORWARD -s 10.69.20.20 -j DROP
```

**3. QoS Priority for Streaming:**

```bash
# Prioritize F1 TV traffic
iptables -t mangle -A POSTROUTING -s 10.69.20.20 -j DSCP --set-dscp 46
```

---

### Part I: Kill Switch (Advanced)

Prevent traffic leaks if VPN drops:

```bash
nano /root/vpn-killswitch.sh
```

```bash
#!/bin/sh

# VPN Kill Switch
# Blocks ALL internet if VPN is down (for privacy-critical devices)

CRITICAL_DEVICES="10.69.20.10 10.69.20.30"  # MacBook, Phone

# Check if VPN is up
if ! ifconfig wg0 &>/dev/null; then
    logger -t vpn-killswitch "VPN DOWN! Blocking critical devices"
    
    # Block internet for critical devices
    for IP in $CRITICAL_DEVICES; do
        iptables -I FORWARD -s $IP -j DROP
        logger -t vpn-killswitch "Blocked: $IP"
    done
else
    # VPN is up, remove blocks
    for IP in $CRITICAL_DEVICES; do
        iptables -D FORWARD -s $IP -j DROP 2>/dev/null
    done
fi
```

**Run every minute:**

```bash
crontab -e
```

```
* * * * * /root/vpn-killswitch.sh
```

Now if VPN drops, MacBook and phone lose internet (no leaks!), but F1 TV device keeps working.

---

### Part J: Multiple VPN Providers (Pro Setup)

Have backup VPNs for when one is blocked:

```bash
# Primary: Mullvad
# Backup 1: ProtonVPN  
# Backup 2: IVPN

nano /root/vpn-failover.sh
```

```bash
#!/bin/sh

# Try Mullvad
if curl --interface wg0 -s --connect-timeout 5 ifconfig.me; then
    echo "Mullvad working"
    exit 0
fi

# Mullvad failed, try ProtonVPN
ifdown vpn
uci set network.vpn.proto='wireguard-proton'
ifup vpn
sleep 5

if curl --interface wg0 -s --connect-timeout 5 ifconfig.me; then
    echo "Switched to ProtonVPN"
    exit 0
fi

# Both failed
echo "All VPNs failed!"
exit 1
```

---

### GitHub Portfolio Addition

```markdown
## Advanced VPN Policy Routing

### Travel Router with Selective Tunneling
- WireGuard VPN integration on OpenWRT
- Policy-based routing for device/service-specific tunneling
- Automated F1 TV bypass (direct connection for streaming)
- Kill switch for privacy-critical devices
- Web dashboard for mode switching
- Multi-VPN failover capability

**Use Cases:**
- Hotel WiFi: VPN for privacy, direct for streaming
- F1 Race Days: All direct for optimal quality
- Home: VPN disabled, Tailscale for homelab access

**Technical Implementation:**
- IPSet for destination-based routing
- IPTables marking for traffic classification
- Custom routing tables (VPN/Direct)
- Device-specific policy enforcement
- Automated failover between VPN providers
```

This is WAY more practical than MAC randomization, and solves your actual use case perfectly!



---

# PHASE 5: TAILSCALE INTEGRATION

## Step 5.1: Install Tailscale on OpenWRT

**1. Install package:**

```bash
ssh root@192.168.8.1

opkg update
opkg install tailscale luci-app-tailscale
```

**2. Start and enable:**

```bash
/etc/init.d/tailscale start
/etc/init.d/tailscale enable

tailscale up
```

**3. Authenticate via URL in browser**

**4. Enable subnet routing (KEY FEATURE):**

```bash
tailscale up --advertise-routes=10.69.0.0/16 --accept-routes
```

Now your ENTIRE home network is accessible via Tailscale!

From anywhere in the world:
- Access Pi-hole at `10.69.10.2`
- Access Mac Mini at `10.69.10.3`
- Access any device on any VLAN

---

## Step 5.2: Exit Node Configuration

Make your homelab a Tailscale exit node (route ALL traffic through home):

```bash
tailscale up --advertise-exit-node
```

On Tailscale admin console:
- Enable exit node
- Approve subnet routes

Now on your MacBook when traveling:
```bash
tailscale up --exit-node=openwrt
```

ALL your traffic goes through your home network!

---

# PHASE 6: ADVANCED FEATURES

## DNS over HTTPS (DoH)

**1. Install package:**

```bash
opkg install https-dns-proxy luci-app-https-dns-proxy
```

**2. Configure:**

Services → HTTPS DNS Proxy
- Provider: Cloudflare
- Listen port: 5053

**3. Point dnsmasq to it:**

```bash
uci add_list dhcp.@dnsmasq[0].server='127.0.0.1#5053'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

Now all DNS queries are encrypted!

---

## Network-Wide Ad Blocking (Layer 2)

Pi-hole is layer 1. Add OpenWRT blocking as backup:

**1. Configure AdBlock:**

Services → Adblock
- Enable
- Blocklist sources: Select all
- DNS backend: dnsmasq

**2. This gives you:**
- Pi-hole blocks at DNS
- OpenWRT blocks as fallback
- Double protection

---

## VPN Kill Switch

Prevent leaks when VPN drops:

**1. Create firewall rule:**

Network → Firewall → Custom Rules

```bash
# Only allow internet if Tailscale is up
iptables -A FORWARD -o tailscale+ -j ACCEPT
iptables -A FORWARD -o wan -j REJECT
```

Now if Tailscale drops, no internet access (prevents leaks).

---

## SQM QoS (Quality of Service)

Prioritize traffic types:

**1. Install:**

```bash
opkg install luci-app-sqm
```

**2. Configure:**

Network → SQM QoS
- Interface: wan
- Download/Upload speeds: 90% of your actual speeds
- Enable

This prevents bufferbloat and makes video calls smooth.

---

## Intrusion Detection (Advanced)

**1. Install Suricata:**

```bash
opkg install suricata luci-app-suricata
```

**2. Configure:**

Services → Suricata
- Interface: wan
- Rules: Enable ET Open rules

**3. View alerts:**

Status → Suricata → Alerts

This logs suspicious network activity.

---

# PHASE 7: MONITORING & MAINTENANCE

## Step 7.1: Statistics & Graphs

**1. Install collectd:**

```bash
opkg install collectd collectd-mod-cpu collectd-mod-memory \
    collectd-mod-network collectd-mod-interface luci-app-statistics
```

**2. View:**

Statistics → Graphs

See real-time:
- CPU usage
- Memory usage
- Network traffic
- Device counts

---

## Step 7.2: Logging

**1. Set up remote logging to Mac Mini:**

System → System → Logging

```
Log remote host: 10.69.10.3
Remote port: 514
```

**2. On Mac Mini, collect logs:**

```bash
# Install syslog server
brew install syslog-ng
```

Now all OpenWRT logs go to Mac Mini for analysis.

---

## Step 7.3: Automated Backups

**1. Create backup script:**

```bash
nano /root/backup.sh
```

```bash
#!/bin/sh

BACKUP_DIR="/tmp/backups"
DATE=$(date +%Y%m%d-%H%M%S)
FILENAME="openwrt-backup-$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# Create backup
sysupgrade -b "$BACKUP_DIR/$FILENAME"

# Copy to Mac Mini
scp "$BACKUP_DIR/$FILENAME" user@10.69.10.3:/backups/openwrt/

# Keep only last 5 backups
cd $BACKUP_DIR
ls -t | tail -n +6 | xargs rm -f
```

**2. Schedule with cron:**

```bash
crontab -e
```

```
# Daily backup at 3 AM
0 3 * * * /root/backup.sh
```

---

# PHASE 8: TRAVEL ROUTER MODE

## Quick Travel Setup

**1. Create travel profile:**

```bash
nano /root/travel-mode.sh
```

```bash
#!/bin/sh

echo "Enabling travel mode..."

# Disable all VLANs
uci delete network.@switch_vlan[1]
uci delete network.@switch_vlan[2]
uci delete network.@switch_vlan[3]
uci delete network.@switch_vlan[4]

# Single flat network
uci set network.lan.ipaddr='192.168.8.1'
uci commit network

# Disable Pi-hole integration
uci set dhcp.lan.dhcp_option='6,1.1.1.1'

# Enable WiFi client mode
uci set wireless.sta.disabled='0'
uci commit wireless

/etc/init.d/network restart
wifi reload

echo "Travel mode enabled! Connect to hotel WiFi and run hotel-wifi.sh"
```

**2. Restore home mode:**

```bash
/etc/init.d/network restart
reboot
```

---

# GITHUB PORTFOLIO INTEGRATION

## New Sections to Add

### docs/openwrt/

```
openwrt/
├── README.md              # Why OpenWRT, what it does
├── initial-setup.md       # Basic configuration
├── vlan-configuration.md  # Network segmentation
├── travel-mode.md         # Captive portal automation
├── security-hardening.md  # Firewall, SSH keys, etc.
└── scripts/
    ├── hotel-wifi.sh
    ├── backup.sh
    └── travel-mode.sh
```

### Updated Main README

```markdown
## Network Infrastructure

### OpenWRT Router
- Custom firmware for complete network control
- VLAN-based network segmentation (Management, Trusted, IoT, Lab, Guest)
- Automated captive portal bypass for travel
- Tailscale integration for remote access
- IDS/IPS with Suricata
- DNS over HTTPS (DoH)
- Network-wide ad blocking
- QoS traffic shaping

### Skills Demonstrated
- Advanced networking (VLANs, routing, firewalls)
- Network security hardening
- Automation scripting (bash, UCI)
- Policy-based routing
- VPN configuration
- DNS architecture
- Traffic analysis
```

---

# COMPLETE ARCHITECTURE DIAGRAM

```
                          INTERNET
                              |
                              |
                    ┌─────────┴──────────┐
                    │   ISP Modem        │
                    │   (Bridge Mode)    │
                    └─────────┬──────────┘
                              |
                              |
                    ┌─────────┴──────────┐
                    │  OpenWRT Router    │
                    │  - VLANs           │
                    │  - Firewall        │
                    │  - Tailscale       │
                    │  - DoH             │
                    │  - SQM QoS         │
                    └─────────┬──────────┘
                              |
                    ┌─────────┴──────────┐
                    │  Managed Switch    │
                    │  (VLAN-aware)      │
                    └─────────┬──────────┘
                              |
            ┌─────────────────┼─────────────────┐
            │                 │                 │
     ┌──────┴─────┐    ┌──────┴─────┐   ┌──────┴─────┐
     │ VLAN 10    │    │ VLAN 20    │   │ VLAN 30    │
     │ Management │    │ Trusted    │   │ IoT        │
     └──────┬─────┘    └──────┬─────┘   └──────┬─────┘
            │                 │                 │
      ┌─────┴─────┐     ┌─────┴─────┐    ┌─────┴─────┐
      │ Pi-hole   │     │ MacBook   │    │ Smart TV  │
      │ Mac Mini  │     │ Phone     │    │ Cameras   │
      └───────────┘     └───────────┘    └───────────┘

                    TAILSCALE MESH
                    ┌─────────────────┐
                    │   Everywhere    │
                    │   100.x.x.x/32  │
                    │                 │
                    │ - OpenWRT       │
                    │ - Mac Mini      │
                    │ - Pi-hole       │
                    │ - HP Omen       │
                    │ - MacBook       │
                    └─────────────────┘
```

---

# SKILLS YOU'LL LEARN

## Networking
- ✅ VLANs and network segmentation
- ✅ Routing and switching
- ✅ Firewall configuration
- ✅ NAT and port forwarding
- ✅ QoS and traffic shaping
- ✅ DNS architecture (DoH, DoT)
- ✅ DHCP server configuration

## Security
- ✅ Network security hardening
- ✅ IDS/IPS implementation
- ✅ VPN configuration (WireGuard/Tailscale)
- ✅ Firewall rule creation
- ✅ SSH key authentication
- ✅ Traffic analysis and monitoring

## Automation
- ✅ Bash scripting
- ✅ UCI (OpenWRT config) scripting
- ✅ Cron job scheduling
- ✅ Automated captive portal bypass
- ✅ Network monitoring and alerting

## DevOps
- ✅ Infrastructure as Code
- ✅ Configuration management
- ✅ Automated backups
- ✅ Remote system administration
- ✅ Documentation and runbooks

---

# FINAL EQUIPMENT LIST

## Hardware You Already Have ✅

**OpenWRT Router:**
- GL.iNet router (already owned)

**VLAN-Capable Switch:**
- MikroTik managed switch (already owned) **← EXCELLENT CHOICE**

**Existing Homelab:**
- Mac Mini M4
- Raspberry Pi
- HP Omen 16
- MacBook M4 Air

**Total New Investment: $0** (just VPN subscription ~$5-10/month)

## Optional Add-ons

- Second Raspberry Pi for redundancy (~$50)
- Uninterruptible Power Supply (~$100)
- Longer ethernet cables (~$20)

---

## MikroTik Switch Configuration Notes

Your MikroTik switch uses **RouterOS**, not OpenWRT. This is actually MORE powerful but has different configuration. I'll add a MikroTik-specific VLAN setup section below.

---

# LEARNING RESOURCES

## Essential Reading

**OpenWRT Docs:**
- https://openwrt.org/docs/start
- https://openwrt.org/docs/guide-user/network/vlan/switch_configuration

**Networking:**
- "TCP/IP Illustrated" by Richard Stevens
- r/openwrt subreddit
- r/homelab subreddit

**Security:**
- OWASP IoT Security
- CIS Benchmarks for network devices

## Video Tutorials

- NetworkChuck (YouTube) - OpenWRT basics
- Crosstalk Solutions - VLAN configuration
- Lawrence Systems - pfSense/OpenWRT comparisons

---

# INTERVIEW TALKING POINTS

When discussing this in interviews:

**"I built a segmented home network with OpenWRT"**
- Implemented VLANs for network isolation
- Configured firewall rules for inter-VLAN communication
- Set up Tailscale for secure remote access
- Automated captive portal login for travel scenarios

**Technical depth:**
- "I used UCI scripting to automate network configuration"
- "Implemented DNS over HTTPS for privacy"
- "Set up SQM QoS to prevent bufferbloat"
- "Configured subnet routing through Tailscale"

**Problem-solving:**
- "Built automation to bypass hotel captive portals"
- "Isolated IoT devices to prevent lateral movement"
- "Implemented IDS/IPS to monitor network traffic"

This is **way** beyond most people's homelabs. You'll stand out.

---

# YOUR NEXT STEPS

**You already have the hardware! You're ahead of the game.**

1. **Identify your GL.iNet model** (check bottom label or web interface)
2. **Identify your MikroTik switch model** (check label - CRS3xx? CRS1xx?)
3. **Start with Phase 1** (OpenWRT basic setup - probably already done)
4. **Configure MikroTik VLANs** (use the appropriate method for your model)
5. **Set up OpenWRT interfaces** for each VLAN
6. **Integrate Pi-hole** with VLAN structure
7. **Add VPN policy routing** for travel
8. **Test everything** before deploying
9. **Document your setup** for GitHub portfolio

**Estimated Setup Time:**
- Basic VLANs: 2-3 hours
- VPN routing: 1-2 hours  
- Testing & refinement: 2-4 hours
- **Total: One weekend project**

**This is a legitimate, enterprise-grade network setup. You're building real skills.**

Ready to start? Let me know your GL.iNet and MikroTik models and I can give you the exact commands!
