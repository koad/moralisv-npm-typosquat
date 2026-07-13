# Threat Analysis — moralisv npm Typo-Squat Wallet Drainer

**Auditor:** Janus
**Date:** 2026-07-13
**Confidence:** CONFIRMED (wallet drainer mechanism) / PROBABLE (npm delivery vector)

---

## 1. What Happened

On 2026-07-09, a GitHub account named `moralisv` was created — a one-character typo-squat on `moralis`, the legitimate Moralis Web3 SDK (~118k npm weekly downloads). The same day, MegaHertz (Auroracoin) fell victim to a wallet drainer deployed at `demo-pledgy.vercel.app`.

The dApp presented as a legitimate "Prediction Market Web3" platform with multi-sig escrow and Polymarket governance integration. Underneath, it contained a hidden `emergencyWithdraw()` contract function allowing the owner to drain all pledged funds. The UI was professional-grade: Next.js 14, MUI, wagmi wallet connectors (MetaMask, WalletConnect, Coinbase Wallet, Phantom), zustand state management, and a responsive dark-themed design.

The victim's machine is currently disconnected from the internet. Exact fund loss is unknown pending wallet inspection from a clean device.

## 2. Janus Perimeter View — What Should Have Tripped

### Signal 1: Typo-squatted GitHub account creation

| Attribute | Signal |
|-----------|--------|
| **What** | GitHub user `moralisv` created 2026-07-09 |
| **Why it's a signal** | One-character difference from a high-download npm package (`moralis`) |
| **Detection window** | ~0 hours — account was created same day as incident |
| **False positive risk** | MODERATE — many legitimate accounts have similar names |
| **What Janus would need** | A watch list of high-value npm package names + GitHub account creation monitoring for typo-squat variants |

**Verdict:** This would NOT have tripped with current Janus capabilities. Janus does not currently monitor GitHub account creation or compare new accounts against npm package names.

### Signal 2: npm package publication under typo-squat name

| Attribute | Signal |
|-----------|--------|
| **What** | Package `moralisv` possibly published to npm |
| **Why it's a signal** | Typo-squat on `moralis` (~118k weekly downloads) — classic supply-chain attack vector |
| **Detection window** | Unknown — package was removed by time of investigation |
| **False positive risk** | LOW — npm has existing typo-squat detection, but it's reactive |
| **What Janus would need** | npm registry RSS/API monitoring for packages matching typo-squat heuristics on watched package names |

**Verdict:** This would NOT have tripped. Janus does not monitor the npm registry.

### Signal 3: Vercel deployment with wallet drainer ABI pattern

| Attribute | Signal |
|-----------|--------|
| **What** | `demo-pledgy.vercel.app` deployed with Next.js app containing hidden `emergencyWithdraw()` + `owner()` ABI functions |
| **Why it's a signal** | `emergencyWithdraw()` is a known wallet drainer signature — present in ABI but never referenced in UI |
| **Detection window** | Days to weeks — the Vercel deployment likely predates or coincides with the GitHub account creation |
| **False positive risk** | LOW-MODERATE — `emergencyWithdraw` is a legitimate pattern in some contracts (e.g., upgradeable proxies), but combined with unreferenced `owner()` and typo-squat delivery, the signal sharpens |
| **What Janus would need** | Vercel deployment monitoring for dApps with suspicious ABI patterns, or passive JS chunk analysis for known wallet drainer signatures |

**Verdict:** This would NOT have tripped. Janus does not monitor Vercel deployments or analyze JS chunks for ABI patterns.

### Signal 4: Same-day creation + incident correlation

| Attribute | Signal |
|-----------|--------|
| **What** | GitHub account created same day victim was drained |
| **Why it's a signal** | Disposable infrastructure — create, attack, abandon |
| **Detection window** | Post-hoc only — requires correlating incident reports with account creation |  
**False positive risk** | N/A — this is a retrospective signal, not a preemptive one |
| **What Janus would need** | Real-time correlation between npm/GitHub account creation and incident reports |

**Verdict:** This is a forensic signal, not a detection signal. Useful for pattern confirmation, not prevention.

## 3. The Gap

Janus's current watch surface covers:
- Entity repo commit/PR/issue activity (GitHub atom feeds)
- Daemon MCP emission streams (flights, emissions)
- Public channel perimeter (Discord, Keybase)
- Session transcript archive mining

**None of these surfaces would have caught this attack.** The attack moved entirely outside Janus's watch perimeter:
- npm registry (unwatched)
- GitHub account creation (unwatched)
- Vercel deployments (unwatched)
- Wallet drainer ABI patterns (no detection rules)

This audit itself is the mechanism for expanding Janus's perimeter. The recommendations in `recommendations.md` define the new watch rules.

## 4. What Janus Would Have Caught

Honest answer: **nothing.** This attack was below Janus's detection horizon.

The closest pre-existing signal would have been a daemon emission or entity brief from a developer reporting the typo-squat — but that requires someone to notice first. Janus is the first-line filter, and there was no line to filter.

## 5. Severity Assessment

| Dimension | Rating | Rationale |
|-----------|--------|-----------|
| **Financial impact** | UNKNOWN | Victim machine offline; funds not yet audited |
| **Attack sophistication** | HIGH | Professional-grade dApp, multi-chain wallet support, demo/live mode switch, plausible deniability via "multi-sig escrow" narrative |
| **Detection difficulty** | HIGH | No source code, disposable infrastructure, npm package removed, same-day creation and execution |
| **Likelihood of recurrence** | HIGH | The template is reproducible: typo-squat a popular Web3 package → deploy a polished dApp with hidden drain function → target developers. The attacker infrastructure cost is near-zero. |
| **Blast radius** | MODERATE | Targeted at Web3 developers specifically. Not a mass-consumer attack, but the victim pool (developers who `npm install` Web3 packages) is large and trusted. |

## 6. Immediate-Past Lessons

1. **Typo-squatting is the cheapest attack vector in Web3.** No exploit required — just a convincing UI and a hidden contract function. The victim authorizes the transaction themselves via MetaMask.
2. **"Demo mode" is a red flag.** A dApp that works in demo mode without a contract but switches to "live mode" with a configured address should trigger scrutiny. This is not a normal dApp pattern — legitimate dApps deploy to testnets for demos.
3. **Vercel is the attacker's deployment platform of choice.** Zero-cost deployment, automatic SSL, and `*.vercel.app` domains provide legitimacy. Janus should treat `*.vercel.app` dApps requesting wallet connections with elevated skepticism.
4. **The Moralis brand was chosen deliberately.** It's a Web3 development SDK — the target audience is developers who hold crypto and connect wallets to dApps. The typo-squat exploits the developer workflow, not the end-user workflow.

---

---

## 7. Postscript — Source Code Extraction Results (2026-07-13T20:00 UTC)

After the initial analysis, Juno obtained a source code archive (`moralis-project.zip`, 26.7MB) from the victim's machine and extracted it to `extracted/`. Full analysis: `analysis-supplement.md` and `evidence/source-analysis/`.

**The source archive contains a DIFFERENT attack from the `moralisv`/`demo-pledgy` drainer.**

### Summary of Moralis-MVP Backdoor

- **Project:** "Moralis-MVP" — a React 16 / Express / MongoDB / Socket.IO poker game
- **README:** Fabricated — claims React 18, Next.js 16, TypeScript, Three.js, Solana, NFTs (none present)
- **Backdoor:** Obfuscated JavaScript payload in `routes/api/auth.js` — exfiltrates `process.env`, beacons every 5s to C2, accepts `eval()` RCE
- **C2:** `http://51.178.11.177:1224/api/checkStatus` (was `http://216.250.252.245:1224/api/checkStatus`)
- **Delivery:** npm `prepare` hook silently starts server on `npm install`
- **Auth bypass:** `const isMatch = true;` in `controllers/auth.js` — any password works
- **Git authors:** MoralisMD (`moralismd@gmail.com`), Rechard Treslove (`moralismd668@gmail.com`), moralisv (`hello@moralisus.com`)

### Relationship Between Campaigns

These are **separate incidents** with shared targeting of the Moralis brand. Different attack mechanisms (smart contract drain vs. backdoor/RCE), different infrastructure (Vercel vs. OVH VPS), different tech stacks (Next.js 14 vs. React 16). Whether they are the same threat actor or separate operators targeting the same brand remains unresolved.

### Updated Assessment

The discovery of the Moralis-MVP backdoor does not change the drainer analysis in sections 1-6 but **expands the threat surface**. The attacker(s) targeting the Moralis brand are running multiple parallel campaigns:
1. Typo-squat npm package → wallet drainer dApp (sections 1-6)
2. Brand impersonation → npm `prepare` hook → backdoor beacon (this addendum)

Both target Web3 developers through the npm ecosystem. Janus's detection recommendations in `recommendations.md` should now cover both patterns: npm `prepare`/`postinstall` hooks AND typo-squat package monitoring.

---

*Generated by Janus | 2026-07-13T19:41 UTC | Updated 2026-07-13T20:00 UTC | Flight: `20260713T194115-706Z-janus-4653f9` / `20260713T195536-467Z-janus-c33e3c`*
