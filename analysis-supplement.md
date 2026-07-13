# Analysis Supplement — Source Code Extraction Results

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**Date:** 2026-07-13
**Flight:** `20260713T195536-467Z-janus-c33e3c`
**Status:** COMPLETE — source analyzed, findings filed

---

## 1. What Was Extracted

The zip file `moralis-project.zip` (26.7MB) was extracted by Juno to `extracted/`. It contains the **Moralis-MVP** project — a Web3 poker/gaming platform impersonating the Moralis brand.

**This is NOT the `moralisv` npm typo-squat package and NOT the `demo-pledgy.vercel.app` wallet drainer.** It is a separate attack vector: brand impersonation → npm `prepare` hook → backdoor beacon.

---

## 2. Critical Questions — Resolved

| # | Question | Answer |
|---|----------|--------|
| Q1 | Is `node_modules/moralisv/` inside this zip? | **NO.** This is not the npm typo-squat package. |
| Q2 | Is there a `package.json` with `moralisv` as a dependency? | **NO.** No reference to the `moralisv` package. |
| Q3 | Is the Next.js dApp source present? | **NO.** This is a React 16 CRA project, not Next.js. |
| Q4 | Does the source match the Vercel build ID? | **NO.** This is an entirely different project. |
| Q5 | Does the ABI contain `emergencyWithdraw()` / `owner()`? | **NO.** No smart contract ABIs in this project. |
| Q6 | Are wallet connectors present? | Only ethers v5 — no wagmi, WalletConnect, or Coinbase connectors. |
| Q7 | Is there a `pledge` function with `writeContractAsync`? | **NO.** No smart contract interaction code. |
| Q8 | Is `.npm/_logs/` included? | **NO.** |
| Q9 | Is shell history included? | **NO.** |
| Q10 | Is there a `.env` with contract addresses? | **NO.** `.env` not included; config uses `dotenv` with `local.env`. |

---

## 3. What Was Actually Found

### Critical: Backdoor Payload

The file `routes/api/auth.js` contains an obfuscated JavaScript payload appended after the legitimate `module.exports` statement. Full deobfuscated analysis: `evidence/source-analysis/backdoor-payload-deobfuscated.md`

**Capabilities:**
- **Environment exfiltration:** All `process.env` variables sent to C2 every 5 seconds
- **System fingerprinting:** Hostname, MAC address, OS details collected
- **Remote code execution:** `eval()` on C2 command
- **C2 server:** `http://51.178.11.177:1224/api/checkStatus` (current), previously `http://216.250.252.245:1224/api/checkStatus`

### High: Bypassed Password Authentication

`controllers/auth.js` line 29: `const isMatch = true;` — password validation is hardcoded to always pass. Any password authenticates for any user.

### High: Silent Auto-Start via npm `prepare` Hook

`package.json` line 10: `"prepare": "start /b node server || nohup node server &"` — the server starts silently on `npm install`, activating the backdoor.

### Medium: README Fabrication

The README claims React 18, Next.js 16, TypeScript, Three.js, Babylon.js, Solana, NFTs — none of which exist in the codebase. This is a social engineering lure.

### Medium: C2 Server Rotation

The attacker changed C2 servers between July 6 and July 10, suggesting multiple C2 servers are maintained.

---

## 4. Relationship to the moralisv/demo-pledgy Attack

These are **separate incidents** with shared characteristics:

| | moralisv/demo-pledgy | Moralis-MVP |
|---|---|---|
| **Attack** | Typo-squat npm → wallet drainer dApp | Brand impersonation → npm `prepare` hook → backdoor |
| **Theft** | Smart contract `emergencyWithdraw()` | Environment exfiltration + RCE |
| **Infrastructure** | Vercel | OVH VPS (France) |
| **Tech stack** | Next.js 14, MUI, wagmi | React 16, Express, Socket.IO |

**Shared:** Moralis brand targeting, npm ecosystem delivery, Web3 developer targets, disposable GitHub accounts, late June/early July 2026 timeframe.

Without linking IPs, emails, or infrastructure, they remain separate incidents.

---

## 5. IOC Comparison Against indicators.md

### Section II — Phishing dApp: DOES NOT MATCH
- ❌ Not Next.js 14 (React 16 / CRA)
- ❌ No MUI components (React Bootstrap)
- ❌ No wagmi, zustand, @tanstack/react-query
- ❌ No Build ID match
- ❌ Wallet connectors: only ethers v5, no MetaMask/WalletConnect/Coinbase/Phantom
- ❌ No `NEXT_PUBLIC_WAGER_CONTRACT_ADDRESS`

### Section III — Suspicious ABI Functions: DOES NOT MATCH
- ❌ No `emergencyWithdraw()`
- ❌ No `owner()`
- ❌ No `pledge` with `writeContractAsync`
- ❌ No smart contract interaction at all

### Section V — npm Package: DOES NOT MATCH
- ❌ No `moralisv` package directory
- ❌ No npm package structure

---

## 6. Completed Actions

- [x] Zip file extracted by Juno to `extracted/`
- [x] All key files read and analyzed: `package.json`, `server.js`, `config.js`, `controllers/*.js`, `middleware/*.js`, `routes/**/*.js`, `socket/index.js`, `client/package.json`, `client/src/apis/index.js`, `models/User.js`, `README.md`
- [x] Grep for suspicious patterns: `emergencyWithdraw`, `drain`, `sendTransaction`, private keys, contract addresses, `demo-pledgy`, `moralisv`, `eval()`, `child_process`
- [x] Grep for obfuscated code patterns: `_0x`, `function _0x`, `\\x` — only found in `routes/api/auth.js`
- [x] Git log inspected: 20 commits, 3 distinct authors, payload introduced July 6, C2 changed July 10
- [x] Obfuscated payload fully deobfuscated and documented
- [x] Source analysis written: `evidence/source-analysis/source-findings.md`
- [x] Deobfuscated payload written: `evidence/source-analysis/backdoor-payload-deobfuscated.md`
- [x] MANIFEST.md updated — M7/M8 resolved, new artifacts added
- [x] indicators.md updated with new IOCs
- [ ] Commit all changes

---

## 7. Updated Verdict

The original audit classified this as a "wallet drainer" delivered via npm typo-squat. The source code extraction reveals a **different but equally serious threat**: a **backdoor delivery vehicle** that uses npm's `prepare` lifecycle hook for silent activation.

**Updated Classification:** Supply-chain backdoor via npm `prepare` hook
**Updated Confidence:** CONFIRMED (backdoor exists, C2 identified, RCE confirmed)
**Relationship to moralisv/demo-pledgy:** SEPARATE INCIDENT — shared brand targeting but different attack mechanism and infrastructure

---

*Generated by Janus | 2026-07-13T20:00 UTC | Flight: `20260713T195536-467Z-janus-c33e3c`*
