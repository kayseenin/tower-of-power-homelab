# Monitoring Setup Guide

Complete guide to setting up comprehensive monitoring for your homelab using Uptime Kuma and external services.

---

## Overview

**The Monitoring Problem:**
If your Mac Mini goes down, any monitoring running ON it goes down too. Solution: Multi-layered monitoring.

**Monitoring Architecture:**
```
External (Cloud)
    ↓
Healthchecks.io (monitors Mac Mini heartbeat)
    ↓
Uptime Kuma (monitors all services)
    ↓
Your Services (Portainer, containers, etc.)
```

---

## Uptime Kuma Setup

### Deployment

Already deployed via docker-compose:

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - uptime-kuma:/app/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 3001:3001
    restart: unless-stopped

volumes:
  uptime-kuma:
```

Access at: `http://mac-mini-ip:3001`

### Initial Configuration

1. Create admin account (first visit)
2. Set strong password
3. Click "Create"

---

## Adding Monitors

### Monitor Types

| Type | Use Case | Example |
|------|----------|---------|
| HTTP(s) | Check web services | Portainer, web apps |
| Ping | Check device availability | Mac Mini, Raspberry Pi |
| Port | Check if port is open | SSH (22), DNS (53) |
| Docker Container | Monitor container health | Requires Docker socket |
| DNS | Check DNS resolution | Pi-hole functionality |
| TCP Port | Advanced port checking | Custom services |

### Monitor 1: Portainer (HTTP)

**Purpose:** Verify Portainer web interface is accessible

1. Click **"+ Add New Monitor"**
2. Configure:
   - **Monitor Type:** `HTTP(s)`
   - **Friendly Name:** `Portainer`
   - **URL:** `http://10.69.10.3:9000` (use your Mac Mini IP)
   - **Heartbeat Interval:** `60` seconds
   - **Retries:** `3`
   - **Heartbeat Retry Interval:** `60` seconds
   - **Accepted Status Codes:** `200-299`
3. Click **Save**

Status should turn green "Up" within seconds.

### Monitor 2: Uptime Kuma Itself (HTTP)

**Purpose:** Self-monitoring (useful for external checks)

1. **Add New Monitor**
2. Configure:
   - **Monitor Type:** `HTTP(s)`
   - **Friendly Name:** `Uptime Kuma`
   - **URL:** `http://10.69.10.3:3001`
   - **Heartbeat Interval:** `60`
3. **Save**

### Monitor 3: Mac Mini Availability (Ping)

**Purpose:** Check if Mac Mini is online

1. **Add New Monitor**
2. Configure:
   - **Monitor Type:** `Ping`
   - **Friendly Name:** `Mac Mini`
   - **Hostname:** `10.69.10.3`
   - **Heartbeat Interval:** `60`
3. **Save**

### Monitor 4: SSH Access (Port)

**Purpose:** Verify SSH is accessible

1. **Add New Monitor**
2. Configure:
   - **Monitor Type:** `Port`
   - **Friendly Name:** `Mac Mini SSH`
   - **Hostname:** `10.69.10.3`
   - **Port:** `22`
   - **Heartbeat Interval:** `60`
3. **Save**

---

## Organizing Monitors with Tags

### Creating Tags

1. Click **Settings** (gear icon, bottom left)
2. Click **Tags**
3. **Add New Tag:**
   - Name: `infrastructure`
   - Color: Blue
4. Repeat for:
   - `containers` (Green)
   - `network` (Orange)
   - `external` (Purple)

### Applying Tags to Monitors

1. Edit any monitor
2. Scroll to **Tags** section
3. Click "+ Add Tag"
4. Select tag
5. Save

**Suggested Tagging:**
- Mac Mini, Raspberry Pi → `infrastructure`
- Portainer, Uptime Kuma, Dozzle → `containers`
- OpenWRT, Pi-hole → `network`
- Google.com, F1TV → `external`

---

## External Monitoring with Healthchecks.io

### Why External Monitoring?

**Problem:** If Mac Mini dies, Uptime Kuma dies with it.

**Solution:** Cloud service monitors Mac Mini from outside.

### Setting Up Healthchecks.io

1. **Sign up:** https://healthchecks.io
2. **Create Check:**
   - Name: `Mac Mini Heartbeat`
   - Period: `5 minutes`
   - Grace Period: `2 minutes`
   - Tags: `homelab`, `critical`
3. **Copy Ping URL:** `https://hc-ping.com/YOUR-UNIQUE-ID`

### Configuring Cron Heartbeat

**On Mac Mini:**

```bash
# Open crontab
crontab -e

# Press 'i' for insert mode
# Add this line (replace with YOUR URL):
*/5 * * * * curl -fsS --retry 3 https://hc-ping.com/YOUR-UNIQUE-ID > /dev/null

# Press Esc, type :wq, press Enter
```

**What this does:**
- Every 5 minutes, curl pings Healthchecks.io
- If pings stop = Mac Mini is down
- You get email alert

### Testing the Heartbeat

```bash
# Test manually
curl -fsS https://hc-ping.com/YOUR-UNIQUE-ID

# Check Healthchecks.io dashboard
# Should show "Last Ping: just now"
```

### Verify Cron Job is Running

```bash
# List cron jobs
crontab -l

# Check cron is active
ps aux | grep cron

# Monitor cron execution (wait a few minutes)
grep CRON /var/log/system.log | tail -20
```

---

## Configuring Notifications

### Uptime Kuma Notifications

**Email (SMTP):**

1. **Settings** → **Notifications** → **Setup Notification**
2. Choose **Email (SMTP)**
3. Configure:
   - **Friendly Name:** `Email Alerts`
   - **Hostname:** `smtp.gmail.com` (for Gmail)
   - **Port:** `587`
   - **Security:** `TLS`
   - **Username:** your-email@gmail.com
   - **Password:** app-specific password
   - **From Email:** your-email@gmail.com
   - **To Email:** your-email@gmail.com
4. **Test** → Should receive test email
5. **Save**

**Apply to Monitors:**

1. Edit any monitor
2. Scroll to **Notifications**
3. Enable **Email Alerts**
4. **Save**

**Other Notification Options:**
- Discord (webhook)
- Telegram (bot)
- Slack (webhook)
- Ntfy (self-hosted push)
- Webhook (custom)

### Healthchecks.io Notifications

1. **Account Settings** → **Integrations**
2. Add:
   - Email (default)
   - SMS (paid plans)
   - Slack, Discord, Telegram
   - PagerDuty (for serious ops)

---

## Advanced Monitoring

### Monitor HTTP Response Time

Track how fast services respond:

1. Edit HTTP monitor
2. **Advanced** section:
   - Enable **Response Time**
3. View graphs showing response time trends

### Monitor Specific HTTP Content

Verify page contains expected text:

1. Edit HTTP monitor
2. **Advanced** → **Keyword**
3. Enter keyword to search for (e.g., "Portainer")
4. If keyword missing → monitor fails

### Certificate Expiration Monitoring

For HTTPS sites, monitor SSL certificate expiry:

1. Monitor shows cert expiration automatically
2. Get alerts before certs expire
3. Useful when you add HTTPS later

---

## Creating a Status Page (Optional)

Share homelab status publicly or privately:

1. **Status Pages** (left menu)
2. **Add New Status Page**
3. Configure:
   - **Title:** "Homelab Status"
   - **Description:** "Personal infrastructure monitoring"
   - **Theme:** Dark/Light
4. **Add Groups:**
   - Infrastructure
   - Containers
   - Network
5. **Add Monitors** to each group
6. **Save**

Access at: `http://mac-mini-ip:3001/status/slug`

**Security:**
- Make it password-protected
- Or keep it private (no sharing)
- Useful for showing family "is it down or is it just me?"

---

## Monitoring Best Practices

### What to Monitor

**Essential:**
- ✅ Service HTTP endpoints (Portainer, etc.)
- ✅ Device availability (Ping)
- ✅ Critical ports (SSH, DNS)
- ✅ External heartbeat (Healthchecks.io)

**Nice to Have:**
- ⭐ Response times
- ⭐ Certificate expiration
- ⭐ Disk space (via custom scripts)
- ⭐ Bandwidth usage

**Avoid:**
- ❌ Monitoring too frequently (causes load)
- ❌ Monitoring non-critical services
- ❌ Alert fatigue (too many notifications)

### Setting Appropriate Intervals

| Service Type | Interval | Reasoning |
|--------------|----------|-----------|
| Critical (SSH, DNS) | 60s | Quick detection |
| Web Services | 60-120s | Balance load/detection |
| Heartbeat | 5 min | Cron-friendly |
| External Sites | 5 min | Avoid rate limits |

### Dealing with False Positives

**If monitor flaps (up/down repeatedly):**

1. Increase retry count (3-5 retries)
2. Increase heartbeat retry interval
3. Check if service is actually unstable
4. Verify network isn't the issue

---

## Monitoring Dashboard

### Recommended View Setup

**Main Dashboard:**
- Group by tags
- Show only critical monitors
- Pin important services to top

**Status Page:**
- Create for external sharing
- Show uptime percentages
- Display current incidents

---

## Sample Monitor List

Here's a complete starter configuration:

| Monitor | Type | Target | Interval | Tags |
|---------|------|--------|----------|------|
| Portainer | HTTP(s) | http://10.69.10.3:9000 | 60s | containers |
| Uptime Kuma | HTTP(s) | http://10.69.10.3:3001 | 60s | containers |
| Mac Mini | Ping | 10.69.10.3 | 60s | infrastructure |
| Mac Mini SSH | Port | 10.69.10.3:22 | 60s | infrastructure |
| Google | HTTP(s) | https://www.google.com | 300s | external |
| Cloudflare DNS | Ping | 1.1.1.1 | 300s | external |

**As you expand, add:**
- Pi-hole (HTTP, DNS)
- Raspberry Pi (Ping, SSH)
- OpenWRT (HTTP, Ping)
- Tailscale (monitor Tailscale IPs)

---

## Troubleshooting

### Monitor Shows "Down" but Service Works

**Check:**
1. URL is exactly correct (http vs https, port, path)
2. Uptime Kuma container can reach service
3. Firewall isn't blocking Uptime Kuma
4. Try from Mac Mini itself: `curl http://localhost:9000`

### Docker Monitors Not Working

**Issue:** "connect ENOENT /var/run/docker.sock"

**Fix:** Mount Docker socket:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
```

Redeploy Uptime Kuma container.

### Healthchecks.io Not Receiving Pings

**Check:**
1. Cron job is correct: `crontab -l`
2. Curl command works manually
3. Mac Mini has internet access
4. Check logs: `grep CRON /var/log/system.log`

### Too Many Notifications

**Solutions:**
1. Increase retry counts
2. Lengthen intervals
3. Only alert on critical services
4. Use "notification groups" - alert after X failures

---

## Monitoring Checklist

### Daily
- [ ] Check Uptime Kuma dashboard
- [ ] Verify all monitors green
- [ ] Review any down alerts

### Weekly
- [ ] Check response time trends
- [ ] Verify Healthchecks.io is pinging
- [ ] Review notification settings

### Monthly
- [ ] Review and remove unnecessary monitors
- [ ] Update monitor intervals if needed
- [ ] Check certificate expirations
- [ ] Test notification delivery

---

## Next Steps

- Add more services and monitor them
- Set up notifications for critical alerts
- Create status page for family/friends
- Deploy additional monitoring tools (Glances, Netdata)
- Integrate with Prometheus/Grafana (advanced)

See [Lessons Learned](lessons-learned.md) for common issues and solutions encountered during setup.
