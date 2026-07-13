# Source Code Analysis — Moralis-MVP Project

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**Date:** 2026-07-13
**Flight:** `20260713T195536-467Z-janus-c33e3c`

---

## 1. Identity: What This Project Actually Is

This is **Moralis-MVP** — a Web3 poker/gaming platform that **impersonates the real Moralis brand** (`moralis.io`). It is:

- A React 16 + Express + MongoDB + Socket.IO poker game application
- Uses ethers v5 for wallet connections
- Has a working poker game engine (`game/Table.js`, `game/Deck.js`, `game/Player.js`)
- The game uses virtual chips (not real cryptocurrency) — allocated via `config.INITIAL_CHIPS_AMOUNT` (100,000)

**It is NOT:**
- ❌ The `moralisv` npm typo-squat package (no `node_modules/moralisv/` directory exists)
- ❌ The `demo-pledgy.vercel.app` wallet drainer (completely different tech stack)
- ❌ A Next.js 14 / MUI / wagmi application (README claims this but code is React 16)

---

## 2. README Fabrication

The README is a wholesale fabrication. It claims technologies that do not exist in the codebase:

| README Claims | Reality |
|---------------|---------|
| React 18 / Next.js 16 / TypeScript | React 16 with `react-scripts` (CRA) / JavaScript |
| Three.js / Babylon.js | No 3D libraries in `package.json` |
| Solana Wallet Integration | No Solana dependencies; only ethers v5 |
| NFT Asset Ownership / Marketplace | No NFT contracts or marketplace code |
| AI-Powered Systems | No AI/ML dependencies |
| Docker Deployment Support | No Dockerfile in project |
| Redis database | No Redis dependencies |
| Polygon blockchain | Claimed but only ethers v5 for wallet connection |

The README is a **social engineering lure** — designed to make the project appear more legitimate and sophisticated than it actually is.

---

## 3. Critical Security Findings

### Finding 1: Hidden Backdoor / C2 Beacon (CRITICAL)

**Location:** `routes/api/auth.js`, line 18 (appended after `module.exports`)
**Introduced:** Commit `48daa08` by `Rechard Treslove <moralismd668@gmail.com>`, 2026-07-06

A heavily obfuscated JavaScript payload:
- Collects hostname, MAC address, OS details
- **Exfiltrates ALL environment variables** (`process.env`) — including MongoDB URI, JWT secrets, API keys
- Beacons to `http://51.178.11.177:1224/api/checkStatus` every 5 seconds
- Provides **full remote code execution** via `eval()` on C2 command

Full deobfuscated analysis: see `evidence/source-analysis/backdoor-payload-deobfuscated.md`

### Finding 2: Bypassed Authentication (HIGH)

**Location:** `controllers/auth.js`, line 29

```javascript
const isMatch = true;  // Password validation is HARDCODED to always pass
```

The bcrypt password comparison is replaced with a hardcoded `true`. **Any password** authenticates successfully for any user. This is likely intentional — it ensures the attacker can always log in to any account on any deployment of this code.

### Finding 3: Silent Auto-Start via `prepare` Hook (HIGH)

**Location:** `package.json`, line 10

```json
"prepare": "start /b node server || nohup node server &"
```

The `prepare` npm lifecycle hook runs automatically after `npm install`. This hook:
- Silently starts the Express server in the background
- On Windows: `start /b node server` (background process)
- On Unix: `nohup node server &` (detached background process)
- **Activates the backdoor beacon immediately** — the victim doesn't need to run the app

This is a **supply-chain attack pattern**: `npm install` → `prepare` hook fires → server starts → beacon activates → data exfiltration begins.

### Finding 4: C2 Server Change (MEDIUM)

The C2 server was changed between commits:
- **Original:** `http://216.250.252.245:1224/api/checkStatus` (commit `48daa08`, July 6)
- **Current:** `http://51.178.11.177:1224/api/checkStatus` (commits `5e96843`/`9f6c2b5`, July 10)

Both IPs are hosted on OVH (France). The change suggests the attacker maintains multiple C2 servers and rotates them.

---

## 4. Git History Analysis

### Timeline

```
2026-06-22 → 2026-06-28  MoralisMD <moralismd@gmail.com>
                          Initial project: poker game scaffolding, routes, models

2026-06-30 → 2026-07-06  Rechard Treslove <moralismd668@gmail.com>
                          README fabrication, backdoor injection (July 6)

2026-07-10                moralisv <hello@moralisus.com>
                          C2 server update, final settings
```

### Authors

| Author | Email | Period | Role |
|--------|-------|--------|------|
| MoralisMD | `moralismd@gmail.com` | Jun 22-28 | Original poker game developer (or initial persona) |
| Rechard Treslove | `moralismd668@gmail.com` | Jun 30 - Jul 6 | Backdoor injector; README fabricator |
| moralisv | `hello@moralisus.com` | Jul 10 | Final configuration; C2 update |

The domain `moralisus.com` in the latest author email (`hello@moralisus.com`) is itself a Moralis brand impersonation — the real Moralis is at `moralis.io`.

The transition from `MoralisMD` to `Rechard Treslove` to `moralisv` suggests either:
- Multiple personas used by one attacker
- A project that changed hands (sold/transferred on underground forums)

---

## 5. Answering the Key Questions

### Q: Is this a legitimate project with a sloppy `prepare` hook, or is there actual wallet-draining functionality hidden deeper?

**This is a backdoor delivery vehicle.** The poker game code is largely legitimate (the game engine works), but it serves as camouflage for:

1. **The `prepare` hook** — silent server activation on `npm install`
2. **The obfuscated backdoor** — environment exfiltration + RCE
3. **The bypassed auth** — guaranteed attacker access

There is NO wallet-draining functionality in the traditional sense (no `emergencyWithdraw()`, no smart contract interaction in the backend). The attack vector is different: **data exfiltration + remote access**, not direct crypto theft via smart contracts.

However, the backdoor's RCE capability means once a victim runs `npm install`, the attacker can deploy ANY payload — including a wallet drainer — at will.

### Q: What is the connection between this project and the `moralisv` / `demo-pledgy.vercel.app` drainer?

**These appear to be SEPARATE incidents** with shared characteristics:

| Dimension | moralisv/demo-pledgy drainer | Moralis-MVP backdoor |
|-----------|------------------------------|----------------------|
| **Attack vector** | Typo-squat npm package → wallet drainer dApp | Brand impersonation → npm `prepare` hook → backdoor |
| **Target** | Web3 developers via npm | Developers who clone/fork the GitHub repo or get it via npm |
| **Tech stack** | Next.js 14, MUI, wagmi | React 16, Express, Socket.IO |
| **Theft mechanism** | Smart contract `emergencyWithdraw()` | Environment exfiltration + RCE |
| **Infrastructure** | Vercel (`demo-pledgy.vercel.app`) | OVH VPS (`51.178.11.177`, `216.250.252.245`) |
| **Moralis brand** | Typo-squat on `moralis` (npm package name) | Full brand impersonation (name, README, domain `moralisus.com`) |

**Shared DNA:**
- Both target the Moralis brand
- Both use npm as the delivery ecosystem
- Both target Web3 developers specifically
- Both use disposable/sock-puppet GitHub accounts
- Both active in late June / early July 2026

**Possible explanations:**
1. **Same threat actor, different campaigns** — One group running parallel operations with different tools
2. **Same underground marketplace** — Both projects acquired from the same seller/forum
3. **Copycat** — One inspired by the other
4. **Coincidence** — Moralis is a large brand; multiple attackers target it

Without linking the IPs, emails, or infrastructure, these remain separate incidents in the audit.

---

## 6. The Prepare Hook: Supply-Chain Attack Pattern

The `prepare` script is the critical delivery mechanism:

```
Victim runs:  npm install
              ↓
npm triggers: prepare hook → "start /b node server || nohup node server &"
              ↓
Server starts: Express on port 7777 (or next available)
              ↓
auth.js loads: Obfuscated payload executes
              ↓
Beacon begins:  Every 5 seconds, exfiltrates process.env to C2
              ↓
C2 responds:   Can deploy arbitrary code via eval()
```

### Previous C2 Variant Analysis

The earlier commit (`48daa08`) had a different obfuscated payload with C2 at `216.250.252.245:1224`. The obfuscation structure is identical but the string array and C2 URL differ — confirming the payload was deliberately updated.

---

## 7. What Was NOT Found

- ❌ No `node_modules/moralisv/` directory — this is NOT the npm typo-squat package
- ❌ No Next.js code, Vercel deployment config, or `demo-pledgy` references
- ❌ No wallet drainer smart contract ABIs
- ❌ No `emergencyWithdraw()` or `owner()` patterns
- ❌ No reference to `NEXT_PUBLIC_WAGER_CONTRACT_ADDRESS`
- ❌ No `.npm/_logs/` or shell history files
- ❌ No `.env` file with secrets (the project uses `config.js` with `dotenv`)

---

## 8. New IOCs

See updated `indicators.md` for the complete list. Key additions:

| IOC | Type | Value |
|-----|------|-------|
| C2 Server (current) | Network | `51.178.11.177:1224` |
| C2 Server (previous) | Network | `216.250.252.245:1224` |
| C2 Endpoint | URL | `/api/checkStatus` |
| Beacon interval | Behavioral | 5000ms |
| TID value | Artifact | `00:00:00:00:00:00` |
| Attacker email 1 | Identity | `moralismd668@gmail.com` |
| Attacker email 2 | Identity | `hello@moralisus.com` |
| Attacker email 3 | Identity | `moralismd@gmail.com` |
| Attacker name 1 | Identity | Rechard Treslove |
| Attacker name 2 | Identity | MoralisMD |
| Impersonated domain | Infrastructure | `moralisus.com` |
| Malicious npm hook | Artifact | `"prepare": "start /b node server \|\| nohup node server &"` |

---

*Generated by Janus | 2026-07-13 | Flight: `20260713T195536-467Z-janus-c33e3c`*
