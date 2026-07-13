# Indicators of Compromise — moralisv npm Typo-Squat Wallet Drainer

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**Date:** 2026-07-13
**Updated:** 2026-07-13T20:00 UTC (source analysis complete)
**Extracted from:** Sibyl research brief (2026-07-13) + source code analysis (2026-07-13)

---

## I. GitHub Account (moralisv drainer campaign)

| Field | Value |
|-------|-------|
| **Username** | `moralisv` |
| **User ID** | `301768378` |
| **Created** | 2026-07-09T14:05:57Z |
| **Public repos** | 0 |
| **Gists** | 0 |
| **Followers** | 0 |
| **Following** | 0 |
| **Bio** | null |
| **Location** | null |
| **Email** | null |
| **Profile URL** | `https://github.com/moralisv` |
| **API endpoint** | `https://api.github.com/users/moralisv` |

**TTP:** Disposable account — created same day as attack, zero activity, zero personalization. Consistent with single-use attack infrastructure.

---

## II. Phishing dApp (Vercel) — drainer campaign

| Field | Value |
|-------|-------|
| **URL** | `https://demo-pledgy.vercel.app` |
| **Title** | "Prediction Market Web3" |
| **Internal name** | "Pledgy" |
| **Framework** | Next.js 14 (App Router) |
| **UI library** | MUI (Material UI) |
| **Web3 libraries** | wagmi, zustand, @tanstack/react-query |
| **Build ID** | `H3OJvaAJDEugn0X-7XwtV` |
| **Target chain** | Polygon Amoy (testnet) |
| **Wallet connectors** | MetaMask, WalletConnect, Coinbase Wallet, Phantom |
| **Default contract** | `0x0000000000000000000000000000000000000000` (placeholder) |
| **Live config env var** | `NEXT_PUBLIC_WAGER_CONTRACT_ADDRESS` |

### JS Chunks (Next.js build artifacts)

| Chunk | Path | Content |
|-------|------|---------|
| App logic | `/_next/static/chunks/app/%5B%5B...slug%5D%5D/page-73a68b4fae778511.js` | Full app logic, contract ABI, pledge function, wallet integration |
| Config | `/_next/static/chunks/215-ce00b19119147501.js` | wagmi config, wallet connectors, zustand store with localStorage |

---

## III. Smart Contract ABI — Suspicious Functions (drainer campaign)

### Hidden functions (present in ABI, never referenced in UI)

| Function | Signature (conceptual) | Purpose |
|----------|----------------------|---------|
| `emergencyWithdraw` | `emergencyWithdraw()` | Owner drains all contract funds |
| `owner` | `owner() view returns (address)` | Returns attacker's wallet address |

### Bait functions (exposed in UI)

| Function | Purpose |
|----------|---------|
| `createWager` | Creates a wager — bait |
| `pledge` | **Sends user's MATIC/POL to contract** — the drain vector |
| `signMultiSig` | Fake multi-signature — theater |
| `releaseFunds` | Fake fund release — theater |
| `resolveWager` | Fake resolution — theater |
| `cancelWager` | Fake cancellation — theater |

---

## IV. Typo-Squat Target (drainer campaign)

| Field | Value |
|-------|-------|
| **Target package** | `moralis` (Moralis Web3 SDK) |
| **npm URL** | `https://www.npmjs.com/package/moralis` |
| **Version** | v2.27.2 |
| **Weekly downloads** | ~118,000 |
| **Typo-squat variant** | `moralisv` (appended 'v') |
| **Levenshtein distance** | 1 |

---

## V. npm Package (drainer campaign — Unconfirmed)

| Field | Value |
|-------|-------|
| **Package name** | `moralisv` |
| **npm registry** | No results — package removed or never published |
| **npm user** | Not found / unauthorized |

---

## VI. Moralis-MVP Backdoor Campaign — NEW FINDINGS

### A. Backdoor Payload

| Field | Value |
|-------|-------|
| **File location** | `routes/api/auth.js` (appended after `module.exports`) |
| **Introduced** | Git commit `48daa08`, 2026-07-06 |
| **Obfuscation** | String array rotation + hex-indexed lookup + base64 C2 URL |
| **Beacon interval** | 5000ms (5 seconds) |
| **TID parameter** | `00:00:00:00:00:00` |
| **Hidden message** | `bm93IGl0IHRpbWUgdG8gZ2V0IGV2ZXJ5dGhpbmc=` → "now it time to get everything" |

### B. C2 Servers

| C2 IP | Port | Endpoint | First Seen | Last Seen | Hosting |
|-------|------|----------|------------|-----------|---------|
| `51.178.11.177` | 1224 | `/api/checkStatus` | 2026-07-10 | Current | OVH (France) |
| `216.250.252.245` | 1224 | `/api/checkStatus` | 2026-07-06 | ≤ 2026-07-10 | OVH (France) |

### C. Malicious npm Hook

| Field | Value |
|-------|-------|
| **Hook type** | `prepare` (npm lifecycle) |
| **Command** | `start /b node server \|\| nohup node server &` |
| **File** | `package.json` line 10 |
| **Effect** | Silently starts Express server on `npm install`, activating backdoor |

### D. Bypassed Authentication

| Field | Value |
|-------|-------|
| **File** | `controllers/auth.js` line 29 |
| **Code** | `const isMatch = true;` |
| **Effect** | Any password authenticates successfully for any user |

### E. Git Authors (Moralis-MVP repo)

| Name | Email | Period | Role |
|------|-------|--------|------|
| MoralisMD | `moralismd@gmail.com` | Jun 22-28 | Original developer (or persona #1) |
| Rechard Treslove | `moralismd668@gmail.com` | Jun 30 - Jul 6 | Backdoor injector (or persona #2) |
| moralisv | `hello@moralisus.com` | Jul 10 | C2 updater (or persona #3) |

### F. Impersonated Infrastructure

| Field | Value |
|-------|-------|
| **Domain** | `moralisus.com` (impersonates `moralis.io`) |
| **Email** | `hello@moralisus.com` |
| **Project name** | "Moralis-MVP" / "Moralis Blockchain Ecosystem" |

### G. README Fabrication (Social Engineering)

The README claims React 18, Next.js 16, TypeScript, Three.js, Babylon.js, Solana, NFTs, Docker, Redis — **none present in actual codebase**. Real stack: React 16, Express, MongoDB, Socket.IO.

---

## VII. Attack Lifecycle Timeline (Updated)

```
2026-06-22 → 2026-06-28  MoralisMD develops poker game scaffolding
2026-06-30 → 2026-07-06  Rechard Treslove fabricates README, injects backdoor (July 6)
2026-07-06  22:07 UTC     Backdoor payload committed (C2: 216.250.252.245:1224)
2026-07-09  14:05 UTC     GitHub account "moralisv" created (drainer campaign)
2026-07-09                 demo-pledgy.vercel.app deployed or activated
2026-07-09                 Victim encounters drainer, connects wallet, funds drained
2026-07-10                 C2 server updated to 51.178.11.177:1224
2026-07-10                 Final commits by "moralisv" <hello@moralisus.com>
2026-07-09+                Victim disconnects machine, reports via voice
2026-07-13                 Sibyl investigates, files research brief
2026-07-13  19:41 UTC      Janus opens audit
2026-07-13  19:49 UTC      Source code zip arrives from victim via Juno
2026-07-13  19:55 UTC      Juno extracts zip; Janus analyzes source
2026-07-13  20:00 UTC      Backdoor confirmed, deobfuscated, IOCs expanded
```

---

## VIII. Windows-Specific IOCs

### A. Process Indicators

| IOC | Type | Value |
|-----|------|-------|
| Process name | Artifact | `node.exe` (child process hosting `server.js`) |
| Launch command | Artifact | `start /b node server` (background, no console window) |
| Parent process | Behavioral | `cmd.exe` → `node.exe` (when launched from Command Prompt) |
| VS Code variant | Behavioral | VS Code terminal spawns `node.exe` via integrated shell |
| Working directory | Artifact | `C:\Users\<victim>\...\Moralis-MVP\` |

### B. Network Indicators

| IOC | Type | Value |
|-----|------|-------|
| Listening port | Network | TCP `7777` (base port; auto-increments to `7778`, `7779`, etc. if occupied) |
| Port range | Network | `7777-7785` (likely range if multiple instances or port conflicts) |
| Outbound C2 connection | Network | TCP to `51.178.11.177:1224` or `216.250.252.245:1224` |
| Beacon interval | Behavioral | Outbound HTTP GET every 5 seconds |

### C. Persistence Paths (If RCE-Deployed)

The `start /b` command does NOT survive reboot. However, the attacker had RCE and could have deployed persistence via `child_process.exec()`:

| Location | Path | Persistence Type |
|----------|------|------------------|
| User Startup folder | `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\` | Executes on user login |
| System Startup folder | `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\` | Executes on any user login |
| Registry Run (HKCU) | `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Executes on user login |
| Registry Run (HKLM) | `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | Executes on any user login (requires admin) |
| Task Scheduler | `schtasks` entries with triggers: at logon, at startup, hourly, daily | Scheduled execution |
| WMI Event Subscription | `ROOT\Subscription:__EventFilter` + `CommandLineEventConsumer` | Fileless persistence, runs on system events |
| Service | `sc create` service entries | Runs as SYSTEM, starts at boot |

### D. Dropped File Locations

| Location | Path | Risk |
|----------|------|------|
| Temp directory | `%TEMP%\` or `%TMP%\` | Common dropper destination |
| AppData Local | `%LOCALAPPDATA%\` | Hidden from casual inspection |
| AppData Roaming | `%APPDATA%\` | Roams with domain profile |
| ProgramData | `C:\ProgramData\` | System-wide, less frequently checked |
| Downloads | `%USERPROFILE%\Downloads\` | Blends with normal download activity |
| Project directory | `C:\...\Moralis-MVP\node_modules\` | Hides among legitimate dependencies |

### E. VS Code Terminal Artifacts

| IOC | Location |
|-----|----------|
| Terminal command history (PowerShell) | `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` |
| Workspace state | `%APPDATA%\Code\User\workspaceStorage\` — contains recently opened folders |
| Terminal shell integration | VS Code terminal inherits workspace `.env` — env vars visible in terminal session |

### F. File Indicators

| IOC | Type | Value |
|-----|------|-------|
| Suspicious `.bat` files | Artifact | Any `.bat` file referencing `node`, `server`, `nohup`, or `moralis` |
| Suspicious `.ps1` files | Artifact | PowerShell scripts in unusual locations |
| Suspicious `.vbs` files | Artifact | VBScript in startup folders or temp |
| Modified `.env` file | Artifact | Recently modified `local.env` or `.env` in project root |
| npm log files | Artifact | `%APPDATA%\npm-cache\_logs\` — contains install timestamps |

---

## IX. Unresolved IOCs

| IOC | Status | Needed For |
|-----|--------|------------|
| Victim wallet address | UNKNOWN | Transaction tracing, fund recovery |
| Drain transaction hash | UNKNOWN | Attacker wallet identification |
| Deployed contract address (Polygon Amoy) | UNKNOWN | On-chain forensics |
| Attacker owner wallet | UNKNOWN | Attribution, blacklisting |
| `npm install` exact command | UNKNOWN | Drainer delivery vector confirmation |
| `~/.npm/_logs/` contents | UNKNOWN | Package metadata, install timestamp |
| `node_modules/moralisv/` contents | UNKNOWN | Drainer source code analysis |
| Browser history entries | UNKNOWN | Confirms dApp visit |
| Relationship between two campaigns | UNKNOWN | Attribution — same actor or separate? |
| C2 server operator identity | UNKNOWN | Law enforcement referral |

---

*Generated by Janus | 2026-07-13T19:41 UTC | Updated 2026-07-13T20:30 UTC | Flights: `20260713T195536-467Z-janus-c33e3c`, `20260713T200858-398Z-janus-a75926`*
