# REMEDIATION — Moralis-MVP Backdoor Cleanup Guide

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**For:** Anyone who cloned, forked, or ran `npm install` in the Moralis-MVP repository
**Prerequisite:** Read `SELF-AUDIT.md` first — preserve evidence before cleaning up.

---

## ⚠️ First: Disconnect From the Internet

If the backdoor is still running (confirmed in `SELF-AUDIT.md` Section 1), **disconnect now** before proceeding:

- **Wi-Fi:** Turn off
- **Ethernet:** Unplug cable
- **Don't just close the laptop** — kill the connection

The backdoor beacons every 5 seconds. Every additional beacon is another data exfiltration event. Cut it off.

---

## Remediation by Platform

Choose your OS below. Commands are copy-paste ready.

---

## Windows 10 / 11

### Step 1: Kill the Backdoor Process

Find and kill the node.exe process running `server.js`:

```cmd
:: Find the PID of node processes
netstat -ano | findstr ":7777"
:: Example output: TCP  0.0.0.0:7777  0.0.0.0:0  LISTENING  1234
:: ↑ 1234 is the PID

:: Kill it
taskkill /PID 1234 /F

:: If you can't find it by port, find all node.exe processes
tasklist | findstr node.exe
:: Kill all of them
taskkill /IM node.exe /F
```

### Step 2: Check for Persistence

The backdoor's `start /b` command does NOT survive reboot on its own, but the attacker had RCE and could have installed persistence.

```cmd
:: === Startup folder ===
dir "%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup"
dir "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

:: Delete suspicious .bat, .ps1, .vbs, .js, or .exe files
:: Example:
:: del "%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\suspicious.bat"

:: === Registry Run keys ===
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run

:: Delete suspicious entries:
:: reg delete HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v SuspiciousKey /f

:: === Task Scheduler ===
schtasks /query /fo LIST /v | findstr /i "node server moralis npm js"

:: Delete suspicious tasks:
:: schtasks /delete /tn "SuspiciousTaskName" /f

:: === Services (less likely but check) ===
sc query | findstr /i "node moralis"
:: sc delete "SuspiciousService"
```

### Step 3: Remove the Project

```cmd
:: Delete the entire Moralis-MVP directory
rmdir /s C:\path\to\Moralis-MVP

:: Or if you're in it:
cd ..
rmdir /s Moralis-MVP
```

### Step 4: Clear npm Cache

```cmd
npm cache clean --force
```

### Step 5: Check for Dropped Files

The attacker could have written files anywhere via RCE. Key places to check:

```cmd
:: Temp directory
dir %TEMP% /od
dir %TMP% /od

:: Look for recently created .js, .bat, .ps1, .exe files
forfiles /p %TEMP% /m *.js /c "cmd /c echo @fdate @ftime @path" /d +07/01/2026
forfiles /p %TEMP% /m *.bat /c "cmd /c echo @fdate @ftime @path" /d +07/01/2026

:: User profile root
dir %USERPROFILE% /od | findstr /i "\.js \.bat \.ps1 \.vbs"

:: Check Downloads folder for unexpected files
dir %USERPROFILE%\Downloads /od
```

### Step 6: Check VS Code Terminal History

If you used VS Code to run `npm install`:

```cmd
:: VS Code stores shell history in its state database
:: Check recent paths opened in VS Code:
dir "%APPDATA%\Code\User\workspaceStorage" /s | findstr "workspace.json"

:: The terminal command history is in the shell's own history:
:: PowerShell: %USERPROFILE%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
:: CMD: No persistent history by default
```

---

## macOS

### Step 1: Kill the Backdoor Process

```bash
# Find the PID on port 7777
lsof -ti :7777 | xargs kill -9

# Find all node server processes
ps aux | grep "node server" | grep -v grep | awk '{print $2}' | xargs kill -9

# Kill any nohup'd node processes
ps aux | grep nohup | grep node | grep -v grep | awk '{print $2}' | xargs kill -9

# Verify nothing is left
lsof -i :7777
ps aux | grep "node server" | grep -v grep
```

### Step 2: Check for Persistence

```bash
# === LaunchAgents (user) ===
ls -la ~/Library/LaunchAgents/
# Look for .plist files with node, server, moralis, or unfamiliar names
# Remove suspicious ones:
# launchctl unload ~/Library/LaunchAgents/com.suspicious.plist
# rm ~/Library/LaunchAgents/com.suspicious.plist

# === LaunchDaemons (system — requires sudo) ===
sudo ls -la /Library/LaunchDaemons/
# Same inspection — look for node, server, moralis

# === Shell profiles ===
cat ~/.zshrc
cat ~/.bash_profile
cat ~/.bashrc
cat ~/.profile

# Look for lines that start node, exec, nohup, or curl to unknown IPs
# Remove suspicious lines with a text editor

# === Crontab ===
crontab -l
# If there are suspicious entries:
# crontab -e  (then delete the lines)
```

### Step 3: Remove the Project

```bash
# Delete the entire Moralis-MVP directory
rm -rf /path/to/Moralis-MVP
```

### Step 4: Clear npm Cache

```bash
npm cache clean --force
```

### Step 5: Check for Dropped Files

```bash
# Recently modified files in home directory (last 30 days)
find ~ -type f -newermt "2026-07-01" -not -path "*/Library/*" -not -path "*/.npm/*" -not -path "*/.git/*" -ls 2>/dev/null | head -50

# Check /tmp
ls -la /tmp/ | grep -i -E "node|server|moralis|\.js$|\.sh$"

# Check Downloads
ls -la ~/Downloads/
```

### Step 6: Keychain Audit

```bash
# List all keychain entries (check for suspicious apps requesting keychain access)
security dump-keychain ~/Library/Keychains/login.keychain-db 2>/dev/null | grep -E "svce|acct" | head -40

# Check for MetaMask keychain entries
security find-generic-password -l "MetaMask" 2>/dev/null
```

---

## Linux

### Step 1: Kill the Backdoor Process

```bash
# Find the PID on port 7777
fuser -k 7777/tcp 2>/dev/null || lsof -ti :7777 | xargs kill -9

# Find all node server processes
ps aux | grep "node server" | grep -v grep | awk '{print $2}' | xargs kill -9

# Kill any nohup'd node processes
ps aux | grep nohup | grep node | grep -v grep | awk '{print $2}' | xargs kill -9

# Verify nothing is left
ss -tlnp | grep 7777
ps aux | grep "node server" | grep -v grep
```

### Step 2: Check for Persistence

```bash
# === Systemd user units ===
systemctl --user list-units --all | grep -i -E "node|server|moralis"
ls -la ~/.config/systemd/user/

# Disable and remove suspicious services:
# systemctl --user disable suspicious-service.service
# rm ~/.config/systemd/user/suspicious-service.service

# === Crontab ===
crontab -l
# If there are suspicious entries:
# crontab -e  (then delete the lines)

# === Shell profiles ===
cat ~/.bashrc
cat ~/.profile
cat ~/.bash_profile
cat ~/.zshrc

# Look for lines that start node, exec, nohup, or curl to unknown IPs
# Remove suspicious lines with a text editor

# === /etc/crontab, /etc/cron.* (requires sudo) ===
sudo cat /etc/crontab
sudo ls /etc/cron.d/
sudo ls /etc/cron.hourly/
sudo ls /etc/cron.daily/
```

### Step 3: Remove the Project

```bash
# Delete the entire Moralis-MVP directory
rm -rf /path/to/Moralis-MVP
```

### Step 4: Clear npm Cache

```bash
npm cache clean --force
```

### Step 5: Check for Dropped Files

```bash
# Recently modified files in home directory (last 30 days)
find ~ -type f -newermt "2026-07-01" -not -path "*/.cache/*" -not -path "*/.npm/*" -not -path "*/.git/*" -ls 2>/dev/null | head -50

# Check /tmp and /dev/shm
ls -la /tmp/ | grep -i -E "node|server|moralis|\.js$|\.sh$"

# Check Downloads
ls -la ~/Downloads/
```

### Step 6: SSH Key Audit

```bash
# List all SSH keys
ls -la ~/.ssh/

# Check for recently modified keys
find ~/.ssh -newermt "2026-07-01" -ls

# Check authorized_keys for unexpected entries
cat ~/.ssh/authorized_keys

# If you suspect compromise, rotate all SSH keys (see Credential Rotation below)
```

---

## All Platforms — Credential Rotation

**Rotate in this order.** If the attacker got your `.env`, they got everything at once. Prioritize by damage radius.

### Priority 1: Wallet Seed Phrases / Private Keys

If `MNEMONIC`, `SEED_PHRASE`, `PRIVATE_KEY`, or any wallet key was in your `.env`:

1. **Create new wallets immediately** — on a clean, air-gapped machine
2. **Transfer all assets** from compromised wallets to new wallets
3. **Do not reuse any seed phrase** that was on the compromised machine
4. If you used MetaMask on this machine, the attacker had potential access to the vault file — assume all MetaMask wallets are compromised

### Priority 2: MetaMask Password

The victim entered MetaMask passwords (wrong ones — but same passwords used elsewhere):

- Change your MetaMask password
- Change the password on **any service where you reused that password**
- This is why password reuse is dangerous: one wrong entry in a compromised terminal = the attacker knows a password you use elsewhere

### Priority 3: Exchange API Keys

If `BINANCE_API_KEY`, `COINBASE_API_KEY`, `KRAKEN_KEY` or similar were in `.env`:

1. Log into each exchange and **revoke all API keys immediately**
2. Check API key usage logs for unauthorized access (especially withdrawals)
3. Generate new API keys with restricted IP whitelisting
4. Enable withdrawal address whitelisting if available

### Priority 4: Database Credentials

If `MONGODB_URI`, `DATABASE_URL`, `MONGO_URI`, or similar were in `.env`:

1. Anything with credentials embedded in the URI — **rotate the password immediately**
2. Check database access logs for the C2 IPs (`51.178.11.177`, `216.250.252.245`)
3. If this was a production database, initiate incident response — the attacker had full read/write access

### Priority 5: Node Provider API Keys

If `INFURA_API_KEY`, `ALCHEMY_KEY`, `QUICKNODE_KEY`, `MORALIS_API_KEY` were in `.env`:

1. Rotate keys in each provider's dashboard
2. Check usage for unusual request patterns
3. If you had IP allowlisting, the attacker's IPs may appear in logs

### Priority 6: GitHub Tokens / SSH Keys

1. **GitHub:** Settings → Developer settings → Personal access tokens → revoke all
2. **GitHub:** Settings → SSH and GPG keys → delete all SSH keys
3. Generate new SSH keys **on a clean machine**: `ssh-keygen -t ed25519 -C "your@email.com"`
4. Check GitHub audit log (Settings → Archive → Security log) for unauthorized repo access, new PATs, or SSH key additions
5. If you had `NPM_TOKEN` — revoke and regenerate at npmjs.com

### Priority 7: Email Passwords

1. Change email passwords (the attacker may have them from `.env` or browser profile access)
2. Enable 2FA if not already enabled
3. Check email account for:
   - Forwarding rules (attacker may be CC'd on all email)
   - Recovery email/phone changes
   - App-specific passwords

### Priority 8: All Other .env Credentials

Go through your `.env` file line by line. Every key-value pair that was in there was exfiltrated. Rotate all of it.

### Priority 9: Any .env Files on the Machine

The attacker had RCE. They could have read any file on the filesystem. If you have other projects with `.env` files on this machine, rotate those credentials too.

---

## Network-Level Blocking

### Block C2 IPs at Your Firewall

| IP | Port | Hosting | First Seen |
|----|------|---------|------------|
| `51.178.11.177` | 1224 | OVH (France) | July 10, 2026 |
| `216.250.252.245` | 1224 | OVH (France) | July 6, 2026 |

**Windows Firewall:**
```cmd
netsh advfirewall firewall add rule name="Block C2 51.178.11.177" dir=out remoteip=51.178.11.177 protocol=tcp remoteport=1224 action=block
netsh advfirewall firewall add rule name="Block C2 216.250.252.245" dir=out remoteip=216.250.252.245 protocol=tcp remoteport=1224 action=block
```

**macOS (pf):**
```bash
# Add to /etc/pf.conf (requires sudo):
# block drop out proto tcp from any to 51.178.11.177 port 1224
# block drop out proto tcp from any to 216.250.252.245 port 1224
sudo pfctl -f /etc/pf.conf
```

**Linux (iptables):**
```bash
sudo iptables -A OUTPUT -d 51.178.11.177 -p tcp --dport 1224 -j DROP
sudo iptables -A OUTPUT -d 216.250.252.245 -p tcp --dport 1224 -j DROP
# Persist:
sudo iptables-save > /etc/iptables/rules.v4  # Debian/Ubuntu
```

**Router-level:** Block these IPs at your router's firewall if possible — this protects all devices on your network.

---

## Reporting

### Hosting Provider Abuse

Both C2 IPs are hosted on OVH (France):
- **OVH Abuse:** `abuse@ovh.net` | https://www.ovh.com/abuse/
- Include both IPs, the malicious endpoint `/api/checkStatus`, and this audit reference

### GitHub Abuse

- Report the `moralisv` GitHub account: https://github.com/moralisv
- Report the Moralis-MVP repository (if still online)
- GitHub abuse form: https://support.github.com/contact/report-abuse

### Law Enforcement

- **FBI IC3:** https://www.ic3.gov/ (US victims — cryptocurrency theft)
- **France:** https://www.cybermalveillance.gouv.fr/ (OVH-hosted C2)
- Preserve all forensic evidence collected in `SELF-AUDIT.md` Section 5

---

## Verification: Confirm Cleanup

After completing all steps above:

```bash
# All platforms — verify no backdoor processes
ps aux | grep "node server" | grep -v grep   # Unix
tasklist | findstr node.exe                    # Windows

# Verify no C2 connections
netstat -ano | findstr ":1224"                 # Windows
ss -t | grep -E "51\.178\.11\.177|216\.250\.252\.245"  # Linux
lsof -i | grep -E "51\.178\.11\.177|216\.250\.252\.245"  # macOS

# Verify project is gone
ls /path/to/Moralis-MVP  # should return "No such file or directory"
```

If all return empty: **cleanup complete.** Now focus on credential rotation — that's where the real damage control happens.

---

## Checklist Summary

```
[ ] Disconnect from internet
[ ] Preserve forensic evidence (SELF-AUDIT.md Section 5)
[ ] Kill backdoor process
[ ] Check for persistence (startup, registry, crontab, LaunchAgents)
[ ] Check for dropped files
[ ] Remove project directory
[ ] Clear npm cache
[ ] Block C2 IPs at firewall
[ ] Rotate wallet seed phrases / private keys       ← DO FIRST
[ ] Rotate exchange API keys
[ ] Rotate database credentials
[ ] Rotate API keys (Infura, Alchemy, etc.)
[ ] Rotate GitHub tokens / SSH keys
[ ] Rotate email passwords
[ ] Rotate all other .env credentials
[ ] Rotate credentials from all other .env files on machine
[ ] Check all accounts for unauthorized activity
[ ] File abuse reports
[ ] Verify cleanup
[ ] Reconnect to internet
```

---

*Generated by Janus | 2026-07-13 | Flight: `20260713T200858-398Z-janus-a75926`*
