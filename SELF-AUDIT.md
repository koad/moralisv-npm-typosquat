# SELF-AUDIT — Moralis-MVP Backdoor Exposure Assessment

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**For:** Anyone who cloned, forked, or ran `npm install` in the Moralis-MVP repository
**Status:** If you ran `npm install`, assume compromise.

---

## Quick Check: Am I Still Infected?

Run the command for your OS **right now**. If you see output, the backdoor is still running.

### Windows (Command Prompt or PowerShell)
```cmd
netstat -ano | findstr :7777
```

### macOS / Linux
```bash
lsof -i :7777 2>/dev/null || ss -tlnp | grep 7777
```
Also check for the background process:
```bash
ps aux | grep "node server" | grep -v grep
```

If either returns results: **the backdoor is live. Disconnect from the internet immediately** (Wi-Fi off, ethernet unplugged) — then proceed to `REMEDIATION.md`.

---

## 1. What Was Stolen

### Confirmed: Full Environment Variable Dump (`process.env`)

Every time the backdoor beaconed (every 5 seconds), it sent **everything in `process.env`** to the attacker's server. This is a complete dump — no filtering, no selection. If it was in your environment, it was exfiltrated.

#### What's Commonly in a Web3 Developer's `.env`

| Category | Common Keys | Risk If Stolen |
|----------|-------------|----------------|
| **Wallet seed phrases** | `MNEMONIC`, `SEED_PHRASE`, `WALLET_MNEMONIC`, `PRIVATE_KEY`, `WALLET_PRIVATE_KEY` | **CRITICAL** — Full wallet control. All assets drainable. |
| **Wallet private keys** | `DEPLOYER_KEY`, `OWNER_KEY`, `DEV_WALLET`, `ACCOUNT_PRIVATE_KEY` | **CRITICAL** — Same as above. |
| **Exchange API keys** | `BINANCE_API_KEY`, `COINBASE_API_KEY`, `KRAKEN_KEY` | **HIGH** — Trading access, withdrawal risk if paired with secret. |
| **Exchange API secrets** | `BINANCE_SECRET`, `COINBASE_SECRET` | **CRITICAL** — Combined with key = full exchange API access. |
| **Node provider keys** | `INFURA_API_KEY`, `ALCHEMY_KEY`, `QUICKNODE_KEY`, `MORALIS_API_KEY` | **MEDIUM** — Rate-limit abuse, request inspection, allowlist bypass. |
| **RPC URLs** | `RPC_URL`, `MAINNET_RPC`, `POLYGON_RPC`, `INFURA_URL` | **LOW** — Reveals your providers; keys embedded in URLs = HIGH. |
| **Database credentials** | `MONGODB_URI`, `DATABASE_URL`, `REDIS_URL`, `MONGO_URI` | **CRITICAL** — Full database access. Contains user data if production. |
| **JWT / session secrets** | `JWT_SECRET`, `SESSION_SECRET`, `AUTH_SECRET`, `TOKEN_SECRET` | **HIGH** — Token forgery. User impersonation if production. |
| **API keys (general)** | `ETHERSCAN_API_KEY`, `OPENSEA_API_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | **MEDIUM-HIGH** — Service access, potential billing abuse. |
| **Cloud credentials** | `AWS_*`, `GCLOUD_*`, `AZURE_*`, `DIGITALOCEAN_*`, `VERCEL_TOKEN` | **CRITICAL** — Infrastructure takeover. |
| **Email / SMTP** | `SMTP_USER`, `SMTP_PASS`, `SENDGRID_API_KEY`, `MAILGUN_KEY` | **MEDIUM** — Spam relay, phishing from your domain. |
| **Environment metadata** | `NODE_ENV`, `APP_NAME`, `DOMAIN`, `CI`, `NPM_TOKEN` | **LOW** — Reconnaissance. |
| **Shell / path** | `PATH`, `HOME`, `USER`, `SHELL` | **LOW** — Fingerprinting. |

**Key question to ask yourself:** What `.env` file was in the project directory — or loaded into your shell — when `npm install` ran?

### Your `.env` Was Dumped If:
- It was in the project root when you ran `npm install` (loaded by `dotenv` via `config.js`)
- It was set in your shell environment (exported variables, shell profile)
- It was loaded by VS Code's integrated terminal (inherits workspace settings)
- It was present as `.env.local`, `.env.development`, or any dotenv variant

**The attacker got the snapshot of all of them at the moment `npm install` completed.**

---

## 2. What Else Could Have Been Deployed

The backdoor provided **full remote code execution** via `eval()`. The attacker could run **any JavaScript** on your machine with the same privileges as the `node` process. This means they could have deployed:

### Payloads the RCE Channel Could Deliver

| Payload Type | What It Does | How |
|--------------|-------------|-----|
| **Keylogger** | Captures every keystroke | `require('os')` + input hooks, or spawns a system keylogger binary |
| **Clipboard scraper** | Reads clipboard for wallet addresses, seed phrases, passwords | `require('child_process').exec('powershell Get-Clipboard')` or `pbpaste` |
| **Wallet file scanner** | Finds and exfiltrates MetaMask vaults, keystore files, wallet.dat | Crawls `%APPDATA%`, `~/Library`, `~/.ethereum`, `~/.bitcoin` |
| **Browser profile extraction** | Steals MetaMask extension data, saved passwords, cookies | Reads Chrome/Firefox profile directories |
| **SSH key theft** | Exfiltrates `~/.ssh/id_rsa`, `id_ed25519`, known_hosts | Simple file read via `require('fs')` |
| **Persistence mechanism** | Installs startup entries, scheduled tasks, crontab entries | `child_process.exec()` to modify OS startup |
| **Lateral movement** | Scans local network, attempts SSH with found keys | `require('net')` port scanning |
| **Cryptominer** | Deploys XMRig or similar | Downloads and executes binaries |
| **Ransomware** | Encrypts files | Any executable deployed via RCE |
| **Credential dumping** | Windows: LSASS dump, macOS: Keychain dump | OS-specific credential access |

**Important:** The attacker had this capability from the moment `npm install` completed. They did not need you to visit a website, open the app, or do anything else. The beacon started within seconds of install.

---

## 3. How to Check If the Backdoor Is Still Running

### Windows 10/11

```cmd
:: Check for the backdoor port
netstat -ano | findstr ":7777"

:: If port 7777 isn't found, check adjacent ports (server auto-increments)
netstat -ano | findstr ":777"
netstat -ano | findstr ":778"

:: Find node.exe processes
tasklist | findstr "node.exe"

:: Check the PID from netstat against tasklist
:: If PID 1234 is on :7777:
tasklist /fi "PID eq 1234"

:: Check for background node processes (start /b leaves no window)
wmic process where "name='node.exe'" get ProcessId,CommandLine
```

### macOS

```bash
# Check for the backdoor port
lsof -i :7777
# or
ss -tlnp | grep 7777

# Find node server processes
ps aux | grep "node server" | grep -v grep

# Check for nohup'd processes
ps aux | grep nohup | grep -v grep

# Find all node processes with listening ports
lsof -i -P | grep node | grep LISTEN
```

### Linux

```bash
# Check for the backdoor port
ss -tlnp | grep 7777
# or
lsof -i :7777 2>/dev/null

# Find node server processes
ps aux | grep "node server" | grep -v grep

# Check for nohup'd processes
ps aux | grep nohup | grep -v grep

# Check all node processes
ps aux | grep node | grep -v grep
```

---

## 4. Timeline Reconstruction

Understanding *when* the infection happened tells you what else might be compromised.

### Check npm Install Date

The `node_modules` directory timestamp shows when `npm install` last ran:

**Windows:**
```cmd
dir /tc node_modules
```

**macOS / Linux:**
```bash
ls -ld node_modules
stat node_modules
```

### Check npm Logs

npm keeps detailed logs of every install. These show exact timestamps and package versions:

```bash
# Find npm log directory
npm config get cache

# Look at logs (path varies by OS)
# macOS/Linux:
ls -lt ~/.npm/_logs/

# Windows:
dir /od %APPDATA%\npm-cache\_logs\

# Read the most recent log
# macOS/Linux:
cat ~/.npm/_logs/$(ls -t ~/.npm/_logs/ | head -1)

# Windows:
type %APPDATA%\npm-cache\_logs\<latest-file>
```

### Check Git Log (If Cloned)

```bash
git log --oneline --all --date=iso
git reflog
```

The malicious commits are:
- `48daa08` — backdoor introduced (July 6, 2026)
- `5e96843` / `9f6c2b5` — C2 server updated (July 10, 2026)

### Check Network Connection History

**Windows:**
```cmd
:: Recent outbound connections to known C2 IPs
netstat -ano | findstr "51.178.11.177"
netstat -ano | findstr "216.250.252.245"
netstat -ano | findstr ":1224"
```

**macOS / Linux:**
```bash
# Check if connections to C2 IPs are active
ss -t | grep -E "51\.178\.11\.177|216\.250\.252\.245"
lsof -i | grep -E "51\.178\.11\.177|216\.250\.252\.245"
```

### Windows-Specific: Check for Persistence

The `start /b` command on Windows launches a background process with no visible window, but it does NOT survive logout/reboot. However, the attacker could have deployed additional persistence via RCE:

```cmd
:: Check Startup folder
dir "%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup"
dir "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

:: Check registry Run keys
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run

:: Check Task Scheduler for suspicious tasks
schtasks /query /fo LIST /v | findstr /i "node server moralis npm"

:: Check for WMI event subscriptions (advanced persistence)
wmic /namespace:\\root\subscription PATH __EventFilter GET __PATH
```

---

## 5. Evidence Preservation (Before Cleanup)

**Do this before removing anything.** You may need it for law enforcement, insurance, or your own investigation.

### What to Preserve

```bash
# 1. Save npm logs
# macOS/Linux:
cp -r ~/.npm/_logs ~/Desktop/moralisv-forensics/npm-logs/

# Windows:
xcopy %APPDATA%\npm-cache\_logs %USERPROFILE%\Desktop\moralisv-forensics\npm-logs\ /E

# 2. Save node_modules timestamps
# macOS/Linux:
ls -laR node_modules > ~/Desktop/moralisv-forensics/node_modules-listing.txt

# Windows:
dir /s node_modules > %USERPROFILE%\Desktop\moralisv-forensics\node_modules-listing.txt

# 3. Capture running process list
# macOS/Linux:
ps aux > ~/Desktop/moralisv-forensics/process-list.txt

# Windows:
tasklist /v > %USERPROFILE%\Desktop\moralisv-forensics\process-list.txt

# 4. Capture network state
# macOS/Linux:
netstat -an > ~/Desktop/moralisv-forensics/netstat.txt
lsof -i > ~/Desktop/moralisv-forensics/lsof.txt

# Windows:
netstat -ano > %USERPROFILE%\Desktop\moralisv-forensics\netstat.txt

# 5. Save git log
git log --all --oneline --graph --date=iso > ~/Desktop/moralisv-forensics/git-log.txt
git log -p > ~/Desktop/moralisv-forensics/git-log-full.txt

# 6. Save the malicious source (zip entire project directory)
# macOS/Linux:
zip -r ~/Desktop/moralisv-forensics/project-source.zip . -x node_modules/

# Windows (PowerShell):
Compress-Archive -Path * -DestinationPath $env:USERPROFILE\Desktop\moralisv-forensics\project-source.zip

# 7. Save browser history (if you visited the GitHub repo)
# Chrome: chrome://history → search "moralis" → screenshot or export
# Check download history for any unexpected files
```

---

## 6. What We Still Don't Know

The attacker had RCE on your machine. Without forensic analysis, we cannot determine:

- Whether they deployed additional payloads beyond the env dump
- Whether they accessed files outside the project directory
- Whether they established persistence beyond the node process
- Whether they moved laterally on your network
- Who else they may have compromised through you (GitHub repos you have push access to, npm packages you publish, etc.)

**Operate under the assumption that everything on this machine was accessible to the attacker.**

---

## Next Steps

1. **Isolate** — Disconnect from the internet until remediation is complete
2. **Preserve** — Save forensic evidence (Section 5)
3. **Remediate** — Follow `REMEDIATION.md`
4. **Rotate** — Change every credential that was in any `.env` on this machine
5. **Report** — File with GitHub, npm (if applicable), and local cybercrime authorities

---

*Generated by Janus | 2026-07-13 | Flight: `20260713T200858-398Z-janus-a75926`*
