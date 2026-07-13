# Detection Recommendations — moralisv npm Typo-Squat Wallet Drainer

**Auditor:** Janus
**Date:** 2026-07-13
**Purpose:** Define new Janus watch rules based on this incident

---

## 1. New Watch Surfaces

This incident exposed three watch surfaces Janus does not currently cover. Each recommendation includes: the signal, the detection mechanism, the false-positive risk, and the implementation pathway.

---

### REC-01: npm Typo-Squat Monitoring

**Signal:** New npm packages whose names are within Levenshtein distance ≤ 2 of a high-value watchlist package.

**Watchlist candidates (seed list):**
| Package | Weekly Downloads | Why Watch |
|---------|-----------------|-----------|
| `moralis` | ~118k | Already targeted |
| `ethers` | ~2.3M | Web3 core |
| `web3` | ~350k | Web3 core |
| `@walletconnect/*` | ~500k | Wallet infrastructure |
| `wagmi` | ~350k | React Web3 hooks |
| `viem` | ~400k | Web3 core |
| `@metamask/*` | ~1M | Wallet infrastructure |
| `@solana/web3.js` | ~250k | Solana ecosystem |
| `@rainbow-me/rainbowkit` | ~150k | Wallet connection UI |

**Detection mechanism:**
- Poll npm registry search API for new packages matching typo-squat patterns on watchlist
- npm provides RSS feeds at `https://www.npmjs.com/package/<name>` — these can be monitored for new publications
- Alternative: `npm search` or registry API keyword monitoring

**False-positive risk:** HIGH for raw Levenshtein distance — many legitimate packages have similar names. Mitigation: require additional signals (low download count, no GitHub repo link, recent publish date, package.json anomalies).

**Implementation pathway:**
1. Build a watchlist of ~50 high-value Web3 npm packages
2. Write a typo-squat candidate generator (Levenshtein ≤ 2, common substitutions)
3. Poll npm registry search weekly for candidates
4. Flag any candidate with: < 100 weekly downloads AND (no GitHub repo OR GitHub repo created < 30 days ago)

**Priority:** HIGH — this is the direct recurrence path for this attack.

---

### REC-02: Vercel dApp Wallet Drainer Detection

**Signal:** `*.vercel.app` deployments that (a) request wallet connections and (b) contain suspicious ABI patterns.

**Detection mechanism:**
- Cannot monitor all Vercel deployments — too many, too noisy
- **Instead:** monitor GitHub repositories that deploy to Vercel and match typo-squat patterns
- For direct page monitoring: fetch landing pages of typo-squat-associated Vercel URLs and scan for wagmi/ethers wallet connector patterns + `emergencyWithdraw` ABI signatures

**Suspicious ABI pattern (regex-ready):**
```
emergencyWithdraw|emergencyWithdrawl|emergencyDrain|rugPull|adminWithdraw
```
Combined with: `owner|admin` function present in ABI but absent from UI component tree.

**False-positive risk:** MODERATE — `emergencyWithdraw` appears in legitimate upgradeable proxy contracts. Mitigation: require co-occurrence with typo-squat GitHub account or npm package.

**Implementation pathway:**
1. When an npm typo-squat candidate is flagged (REC-01), check if the package links to a Vercel deployment
2. If yes, fetch the landing page, extract JS chunks, scan for wallet connector patterns + suspicious ABI functions
3. Flag if both conditions are met

**Priority:** HIGH — this is the payload detection for the typo-squat vector.

---

### REC-03: GitHub Account Creation Monitoring for Typo-Squat Usernames

**Signal:** New GitHub accounts whose usernames match typo-squat patterns on high-value npm package names.

**Detection mechanism:**
- GitHub Events API provides a firehose of public events
- Filter for `CreateEvent` with `ref_type: repository` from new accounts
- Cross-reference usernames against npm package watchlist
- This is noisy — better used as a secondary signal when an npm typo-squat has already been flagged

**False-positive risk:** VERY HIGH — GitHub username space is crowded, many legitimate users have names similar to packages. Do NOT alert on this alone — use only as corroborating signal.

**Implementation pathway:**
1. When an npm typo-squat candidate is flagged (REC-01), query GitHub API for the matching username
2. Check account age, repo count, activity level
3. Flag if: account < 30 days old AND repos = 0 AND activity = 0

**Priority:** MEDIUM — useful as confirmation signal, not as primary detection.

---

### REC-04: "Demo Mode" dApp Pattern Detection

**Signal:** dApps that operate in a "demo mode" with mock data but switch to real contract interactions when an address is configured.

**Why this is a signal:** Legitimate dApps deploy to testnets for demos. A "demo mode" that masks real contract interaction capability is a deception pattern — it allows the attacker to share a harmless-looking demo while maintaining a live drain.

**Detection mechanism:**
- Scan JS chunks for patterns like `NEXT_PUBLIC_WAGER_CONTRACT_ADDRESS` or similar env-var-configured contract addresses
- Look for conditional logic: `if (contractAddress) { /* real */ } else { /* mock */ }`
- This is a code-review pattern, not an automated detection pattern — file under "manual review triggers"

**False-positive risk:** MODERATE — some legitimate dApps use feature flags. Mitigation: require co-occurrence with typo-squat or suspicious ABI.

**Implementation pathway:**
1. When a flagged dApp is being manually reviewed, grep JS chunks for `NEXT_PUBLIC_.*CONTRACT.*ADDRESS` patterns
2. Check if the conditional branch leads to `writeContractAsync` or `sendTransaction`
3. Flag if demo mode masks real transaction capability

**Priority:** LOW — this is a review heuristic, not an automated detection rule.

---

## 2. General Heuristics (No Implementation Required)

These are pattern-recognition rules for Janus to apply when reviewing any flagged signal:

### Heuristic A: "Too Polished to Be Fake" Is the Tell
Professional-grade UI (Next.js, MUI, responsive design, gradient accents) combined with typo-squat delivery is a HIGH-confidence signal. Attackers invest in UI because it converts victims. A janky dApp is less dangerous than a beautiful one.

### Heuristic B: Multi-Sig Theater Is a Drainer Signature
Any dApp that normalizes "funds held in escrow" while exposing a `pledge`/`deposit` function with real value transfer should trigger scrutiny. Multi-sig is legitimate, but multi-sig combined with hidden admin withdrawal is the drainer pattern.

### Heuristic C: Same-Day Infrastructure = Attack Infrastructure
GitHub account created same day as npm publication or dApp deployment = disposable attack infrastructure. Legitimate projects accumulate history before deployment.

### Heuristic D: Vercel + Wallet Connect + Typo-Squat = HIGH
Any `.vercel.app` domain that (a) requests wallet connection, (b) is linked from a typo-squat package, and (c) has a GitHub account with zero history is attack infrastructure. Confidence: HIGH.

---

## 3. Implementation Priority Matrix

| Rec | What | Priority | Cost | False+ Risk | Janus Can Do Now? |
|-----|------|----------|------|-------------|-------------------|
| REC-01 | npm typo-squat monitoring | HIGH | MEDIUM | HIGH (needs filtering) | No — needs new watch infrastructure |
| REC-02 | Vercel drainer detection | HIGH | MEDIUM | MODERATE | No — needs REC-01 as trigger |
| REC-03 | GitHub account monitoring | MEDIUM | LOW | VERY HIGH | No — too noisy standalone |
| REC-04 | Demo mode detection | LOW | LOW | MODERATE | No — review heuristic only |

---

## 4. Immediate Actions (Humans / Operators)

These are actions Janus cannot perform but should recommend:

1. **Report `moralisv` GitHub account** via https://github.com/contact/report-abuse
2. **Report `demo-pledgy.vercel.app`** to abuse@vercel.com with this audit attached
3. **Notify Moralis team** — they are the brand being impersonated; they may want to issue a security advisory
4. **Request shell/network capability** for Janus — the inability to `curl` external APIs is a structural gap for perimeter defense work
5. **Expand Janus bond** to include `koadio_commands: [browse, shell]` for evidence capture during audits

---

*Generated by Janus | 2026-07-13T19:41 UTC | Flight: `20260713T194115-706Z-janus-4653f9`*
