# Lessons Learned

Real-world challenges encountered during homelab setup and their solutions.

---

## SSH and Remote Access

### Challenge: First Time SSH Setup Confusion

**Problem:** Didn't know where to enable SSH on macOS.

**Solution:** System Settings → General → Sharing → Remote Login. Simple once you know where it is.

**Takeaway:** macOS hides some server features in non-obvious places. Always start with System Settings → Sharing for network services.

---

## Docker Installation

### Challenge: Docker Commands Not Found After Installation

**Problem:** Installed Docker Desktop but `docker` command didn't work in terminal.

**Solution:** 
1. Make sure Docker Desktop is actually running (check menu bar icon)
2. Restart terminal to reload PATH
3. Verify: `eval "$(/opt/homebrew/bin/brew shellenv)"`

**Takeaway:** Installing an app doesn't always add CLI tools to PATH immediately. Restart terminal after installations.

---

### Challenge: Understanding Docker vs Docker Desktop

**Problem:** Confusion about what Docker Desktop actually is.

**Solution:** 
- **Docker Desktop** = GUI app that runs Docker daemon on macOS
- **Docker CLI** = Command-line tool (`docker` command)
- **Docker Engine** = The actual containerization engine (runs in a VM on macOS)

**Takeaway:** macOS can't run containers natively (Linux kernel feature). Docker Desktop creates a lightweight Linux VM to run Docker Engine.

---

## Portainer Setup

### Challenge: Port Already in Use

**Problem:** Tried to run Portainer on port 9000 but got "port already allocated" error.

**Solution:**
```bash
# Check what's using the port
lsof -i :9000

# Kill the process or use different port
docker run -p 9001:9000 ...
```

**Takeaway:** Always check if ports are available before deploying services. Common conflicting ports: 80, 443, 8080, 9000.

---

## Uptime Kuma Configuration

### Challenge: Docker Socket Permission Error

**Problem:** Uptime Kuma showed "connect ENOENT /var/run/docker.sock" when trying to monitor Docker containers.

**Solution:** Mount the Docker socket with read-only permissions:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock:ro
```

**Takeaway:** Docker socket access is required for container monitoring but presents security risks. Only give it to trusted containers and always use `:ro` (read-only) when possible.

---

### Challenge: Confusion About Monitor Types

**Problem:** Didn't understand when to use HTTP vs Ping vs Port monitors.

**Solution:**
- **HTTP(s)** - When you want to check if a web service is responding properly
- **Ping** - When you just want to know if a device is online
- **Port** - When you want to check if a specific port is open (SSH, DNS, etc.)
- **Docker Container** - When you want container-specific metrics (requires socket access)

**Takeaway:** Start with HTTP monitors for web services and Ping for devices. Those cover 90% of use cases.

---

### Challenge: The "Who Watches the Watchers" Problem

**Problem:** Uptime Kuma runs on Mac Mini. If Mac Mini dies, how do I know?

**Solution:** Multi-layered monitoring:
1. **Uptime Kuma** (internal) - Monitors services
2. **Healthchecks.io** (external) - Monitors Mac Mini via heartbeat
3. Eventually: Second Uptime Kuma on Raspberry Pi

**Takeaway:** Never rely solely on internal monitoring. Always have something external watching your infrastructure.

---

## Cron Job Setup

### Challenge: Editing Crontab with Vi

**Problem:** Opened `crontab -e` and got dropped into vi editor. Didn't know how to edit or save.

**Solution:**
1. Press `i` to enter insert mode
2. Type your cron job
3. Press `Esc` to exit insert mode
4. Type `:wq` and press Enter to save and quit

**Alternative:** Use nano instead:
```bash
export EDITOR=nano
crontab -e
```

**Takeaway:** Vi is powerful but unfriendly for beginners. Nano is more intuitive. Set your preferred editor with `EDITOR` environment variable.

---

### Challenge: Cron Job Syntax Confusion

**Problem:** Didn't understand what `*/5 * * * *` meant.

**Solution:** Cron format is: `minute hour day month weekday command`
- `*/5` = Every 5 minutes
- `*` = Every (hour/day/month/weekday)
- `0 0 * * *` = Midnight daily
- `0 9 * * 1` = 9 AM every Monday

**Helpful tool:** https://crontab.guru - visual cron schedule editor

**Takeaway:** Cron syntax is cryptic but follows a pattern. Use online tools to validate your schedules.

---

## Network and Connectivity

### Challenge: Finding Mac Mini's IP Address

**Problem:** Needed Mac Mini's IP for SSH but didn't know how to find it.

**Solution:**
- **On Mac Mini:** System Settings → Network → Details → TCP/IP
- **From MacBook:** `arp -a | grep -i apple` or check router
- **Best practice:** Set static IP or DHCP reservation

**Takeaway:** IP addresses can change with DHCP. For servers, use static IPs or reservations. Or use hostnames with mDNS (`mac-mini.local`).

---

### Challenge: Accessing Services from MacBook

**Problem:** Could access Portainer on Mac Mini at `localhost:9000` but not from MacBook.

**Solution:**
- `localhost` only works on the same machine
- From other devices, use actual IP: `http://192.168.1.50:9000`
- Or hostname: `http://mac-mini.local:9000`

**Takeaway:** `localhost` = 127.0.0.1 = only this machine. Use actual IPs or hostnames for network access.

---

## Container Management

### Challenge: Container Keeps Restarting

**Problem:** Deployed container but it kept restarting every few seconds.

**Solution:**
```bash
# Check logs to see why it's failing
docker logs container-name

# Common issues:
# - Missing required environment variables
# - Port already in use
# - Volume permissions wrong
# - Command in container exits immediately
```

**Takeaway:** Logs are your friend. Always check logs first when containers misbehave.

---

### Challenge: Understanding Restart Policies

**Problem:** Set `restart: always` but container didn't restart after Mac Mini reboot.

**Solution:**
- `restart: always` works but only if Docker is running
- Mac Mini needs Docker Desktop to auto-start on boot
- Check: System Settings → General → Login Items

**Takeaway:** Container restart policies depend on Docker daemon being up. Ensure Docker Desktop starts automatically.

---

## Data Persistence

### Challenge: Lost Uptime Kuma Data After Recreating Container

**Problem:** Deleted and recreated container, lost all monitors and settings.

**Solution:** Use named volumes (which I did):

```yaml
volumes:
  - uptime-kuma:/app/data  # Named volume persists
```

Named volumes survive container deletion.

**Takeaway:** Always use named volumes for data you want to keep. They persist even when containers are removed.

---

## Security Concerns

### Challenge: Should I Expose Docker Socket?

**Problem:** Mounting `/var/run/docker.sock` felt dangerous.

**Solution:**
- **Mounting the socket gives full Docker control** to that container
- Only give to trusted containers (Portainer, Uptime Kuma, Watchtower)
- Always use `:ro` (read-only) when possible
- For production, use Docker TCP socket with TLS instead

**Takeaway:** Docker socket is powerful. Treat it like a root password. Only trusted containers should have access.

---

### Challenge: Setting Strong Passwords

**Problem:** Used simple passwords for Portainer/Uptime Kuma initially.

**Solution:**
- Use a password manager (1Password, Bitwarden, etc.)
- Generate random 20+ character passwords
- Never reuse passwords across services

**Takeaway:** Homelab is for learning, but still practice good security habits. They become muscle memory for real production work.

---

## Troubleshooting Methodology

### What I Learned About Debugging

**Approach that works:**

1. **Check if it's running:** `docker ps`
2. **Check the logs:** `docker logs container-name`
3. **Check connectivity:** Can you ping? Can you curl?
4. **Check permissions:** Volume mounts, file ownership
5. **Check resources:** Is disk full? Memory exhausted?
6. **Google the exact error message**

**Things that rarely help:**
- Restarting everything immediately
- Making multiple changes at once
- Assuming you know the problem without checking

**Takeaway:** Systematic debugging beats random changes. Change one thing, test, repeat.

---

## Time Management

### Challenge: Scope Creep

**Problem:** Wanted to set up everything at once (Pi-hole, VLANs, VPN, monitoring, media server...).

**Solution:** Break it into phases:
- **Phase 1:** Basic Docker + monitoring (DONE ✅)
- **Phase 2:** Pi-hole on Raspberry Pi
- **Phase 3:** Network infrastructure (VLANs, OpenWRT)
- **Phase 4:** Advanced features (VPN routing, game streaming)

**Takeaway:** Build incrementally. Get one thing working before adding the next. Document as you go.

---

## Documentation Habits

### Challenge: Forgetting What I Did

**Problem:** Set something up, it worked, but couldn't remember exact commands a week later.

**Solution:**
- Document WHILE doing, not after
- Save all docker-compose files
- Take screenshots of config screens
- Write down IP addresses, passwords, URLs
- Use git to track changes

**Takeaway:** Future you will thank present you for writing things down. Documentation isn't overhead—it's insurance.

---

## What I'd Do Differently

### Things I Got Right

✅ Started with remote access (SSH) - foundation for everything else  
✅ Used Portainer instead of just CLI - visual feedback helps learning  
✅ Set up monitoring early - caught issues quickly  
✅ Used external monitoring (Healthchecks.io) - learned about redundancy  
✅ Documented while building - this document exists!

### Things I'd Change

🔄 Would have set static IP for Mac Mini earlier  
🔄 Would have used a password manager from day one  
🔄 Would have read entire guides before starting (instead of skimming)  
🔄 Would have tested backups (I haven't backed up volumes yet!)  
🔄 Would have organized docker-compose files in a git repo immediately

---

## Key Realizations

### About Docker

- Docker is simpler than it seems - containers are just processes
- docker-compose is way easier than manual `docker run` commands
- Named volumes are essential for persistence
- Logs tell you almost everything you need to know

### About Networking

- `localhost` only works on the same machine
- IP addresses matter (static vs DHCP)
- Ports can conflict - check before deploying
- Firewalls exist (though macOS is permissive by default)

### About Monitoring

- External monitoring is not optional - it's essential
- Simple HTTP checks are usually enough
- Alert fatigue is real - only alert on critical things
- Uptime percentages are oddly satisfying to watch

### About Learning

- Barney-style explanations (super detailed) work best for me
- Breaking things is part of learning (containers are easy to recreate)
- Having a real use case (F1 TV, game streaming) makes it stick
- GitHub documentation forces you to understand what you did

---

## Advice for Others Starting Out

### Do This

✅ Start small - get one thing working  
✅ Document as you go  
✅ Use docker-compose, not manual docker run  
✅ Set up monitoring early  
✅ Test backups (seriously, test them)  
✅ Ask "why" - understand what each command does  
✅ Join communities (r/homelab, r/selfhosted)

### Avoid This

❌ Trying to do everything at once  
❌ Skipping documentation "to save time"  
❌ Using weak passwords  
❌ Exposing services to internet without understanding security  
❌ Not backing up data  
❌ Comparing your homelab to others' (everyone started somewhere)

---

## Next Challenges

What I expect to struggle with next:

1. **VLANs** - Network segmentation seems complex
2. **MikroTik RouterOS** - Different from consumer routers
3. **Policy-based routing** - Routing by device/service is new to me
4. **Tailscale subnet routing** - Making whole network accessible remotely
5. **Pi-hole configuration** - DNS is intimidating

I'll document solutions to these as I encounter them.

---

## Conclusion

**Total time invested so far:** ~4 hours  
**Services running:** 2 (Portainer, Uptime Kuma)  
**Things broken and fixed:** 3 (Docker socket, crontab editing, monitor confusion)  
**Lessons learned:** Many (documented above)  
**Feeling:** Accomplished and ready for more

The biggest lesson: **Perfect is the enemy of done.** Ship it, document it, iterate.

This homelab isn't finished—it never will be. That's the point.

---

*Last updated: 2025-02-13*
