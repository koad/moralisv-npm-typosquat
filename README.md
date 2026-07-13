# moralisv — npm Supply-Chain Attack Audit

**Classification:** Supply-chain phishing → backdoor + RCE  
**Confidence:** CONFIRMED  
**Date:** 2026-07-09 through 2026-07-13  

---

## Summary

A threat actor impersonating the Moralis brand (`moralis.io`) distributed a malicious GitHub repository called **Moralis-MVP** — a Web3 poker/gaming platform. The repo contained an obfuscated backdoor that activated silently when the victim ran `npm install`.

The backdoor exfiltrated all environment variables (`process.env`), beaconed to a C2 server every 5 seconds, and provided full remote code execution via `eval()`. The attacker maintained the project under multiple sock-puppet GitHub accounts and rotated C2 infrastructure between attacks.

A separate campaign by the same actor used an npm typo-squat (`moralisv` → `moralis`) to direct victims to a wallet drainer dApp at `demo-pledgy.vercel.app`.

---

## Repo Contents

| File | Purpose |
|------|---------|
| `analysis.md` | Full threat assessment — what happened, what Janus would have caught, severity |
| `analysis-supplement.md` | Source code extraction results — the backdoor dissected |
| `indicators.md` | Complete IOC catalog — accounts, IPs, ABI signatures, file patterns, git authors |
| `recommendations.md` | Detection rules for Janus — what to watch going forward |
| `MANIFEST.md` | Artifact index, classification, status |
| **`SELF-AUDIT.md`** | **→ If you cloned this repo: open this first.** Exposure assessment and evidence preservation. |
| **`REMEDIATION.md`** | **→ Cleanup checklists for Windows, macOS, and Linux.** Credential rotation prioritized by blast radius. |
| `evidence/` | Sibyl's research brief, backdoor deobfuscation, source code analysis |
| `extracted/` | The Moralis-MVP source code from the victim's machine (contains the live backdoor) |

---

## Quick Facts

| Dimension | Detail |
|-----------|--------|
| **Delivery** | npm `prepare` hook → `"start /b node server \|\| nohup node server &"` |
| **Backdoor location** | `routes/api/auth.js` (line 18, after `module.exports`) |
| **Backdoor author** | Rechard Treslove (`moralismd668@gmail.com`), commit `48daa08`, July 6 2026 |
| **C2 server** | `51.178.11.177:1224/api/checkStatus` (OVH France) |
| **Previous C2** | `216.250.252.245:1224/api/checkStatus` (OVH France) |
| **Beacon interval** | 5 seconds |
| **Exfiltrated** | All `process.env` — MongoDB URI, JWT secrets, API keys, wallet keys if present |
| **RCE** | `eval()` on C2 command — full arbitrary code execution |
| **Auth bypass** | `const isMatch = true;` — any password authenticates |
| **Git authors** | MoralisMD, Rechard Treslove, moralisv (`hello@moralisus.com`) |
| **Wallet drainer** | Separate campaign — `demo-pledgy.vercel.app` with hidden `emergencyWithdraw()` |

---

## Attack Chain

```
1. Social engineering: "Fork this cool Moralis poker project"
2. Victim runs:  npm install
3. prepare hook:  "start /b node server || nohup node server &"
4. Server starts: Express on port 7777 (auto-increments if busy)
5. auth.js loads: Obfuscated payload executes
6. Beacon begins:  process.env exfiltrated to C2 every 5 seconds
7. C2 responds:   Deploys arbitrary code via eval()
8. Attacker has:  Full RCE + all environment variables
```

---

## If You Cloned This Repo

**Disconnect from the internet immediately.** Then read `SELF-AUDIT.md` → `REMEDIATION.md`.

Assume everything in your `.env` file was stolen. Assume the attacker deployed additional malware. Rotate wallet seed phrases first — credentials list in `REMEDIATION.md`.

---

## Reporting

- **C2 hosting:** `abuse@ovh.net` — both IPs hosted on OVH France
- **GitHub abuse:** Report `github.com/moralisv`
- **FBI IC3:** https://www.ic3.gov/ (US victims — cryptocurrency theft)

---

*Audit by Janus (koad:io) | 2026-07-13 | `janus-audit-2026-07-13-moralisv`*
