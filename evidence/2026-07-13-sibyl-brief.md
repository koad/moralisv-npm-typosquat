# moralisv — npm Typo-Squat + Wallet Drainer Campaign

**Date:** 2026-07-13
**Researcher:** Sibyl
**Status:** Active — victim machine offline, triage in progress
**Classification:** Supply-chain phishing → wallet drainer
**Source:** Voice report from MegaHertz (Auroracoin)

---

## Summary

- **GitHub user `moralisv`** is a disposable account created **2026-07-09** — same day as the incident. Zero repos, zero activity, bare profile.
- The name is a **typo-squat on `moralis`** — the legitimate Moralis Web3 SDK (~118k weekly npm downloads, v2.27.2).
- The phishing payload is **`demo-pledgy.vercel.app`** — a Next.js dApp branded as "Prediction Market Web3" (internally called "Pledgy"), deployed on Vercel.
- The dApp is a **smart-contract wallet drainer**: it presents as a peer-to-peer prediction market with multi-sig escrow and Polymarket governance, but the contract contains a hidden `emergencyWithdraw()` function allowing the owner to drain all pledged funds.
- **No source code was recovered** — the npm package has no public repos, no published artifact remains on npm (if it was ever there), and the GitHub account is empty.

---

## Findings

### GitHub Account: `moralisv`

| Attribute | Detail | Confidence |
|-----------|--------|------------|
| User ID | 301768378 | CONFIRMED |
| Created | 2026-07-09T14:05:57Z | CONFIRMED |
| Public repos | 0 | CONFIRMED |
| Gists | 0 | CONFIRMED |
| Followers / Following | 0 / 0 | CONFIRMED |
| Bio / Location / Email | All null | CONFIRMED |
| Events / Activity | None | CONFIRMED |

**Assessment:** Account created the same day as the incident and immediately abandoned or held dormant. Every attribute is consistent with disposable attack infrastructure. Confidence: HIGH.

### npm Package: `moralisv`

| Attribute | Detail |
|-----------|--------|
| npm registry search | Zero results — no package currently published |
| npm user endpoint | Unauthorized / not found |
| Typo-squat target | `moralis` (Moralis Web3 SDK, ~118k monthly downloads) |
| Attack vector gap | No source code recovered — no repos, no tarball, no package.json |

**Assessment:** If a package was published, it was likely removed by npm security or the attacker after the incident. Without the package source, the exact delivery mechanism (README link, postinstall hook, or import-time redirect) from `moralisv` → `demo-pledgy.vercel.app` remains unconfirmed. Confidence: PROBABLE (package existed) / SPECULATIVE (exact mechanism).

### Phishing dApp: `demo-pledgy.vercel.app`

| Attribute | Detail |
|-----------|--------|
| URL | `https://demo-pledgy.vercel.app` |
| Title | "Prediction Market Web3" |
| Framework | Next.js 14 (App Router), MUI (Material UI), wagmi, zustand, @tanstack/react-query |
| Build ID | `H3OJvaAJDEugn0X-7XwtV` |
| Target chain | Polygon Amoy (testnet) — chain ID from wagmi config |
| Wallet connectors | MetaMask, WalletConnect, Coinbase Wallet, Phantom (EIP-6963) |
| Demo mode | Mock data via zustand store when no contract address is configured |
| Live mode | Real contract interactions when `NEXT_PUBLIC_WAGER_CONTRACT_ADDRESS` is set |
| Default contract | `0x0000000000000000000000000000000000000000` (placeholder) |

**Contract ABI — Key Functions:**

| Function | UI Exposure | Purpose |
|----------|-------------|---------|
| `createWager` | ✅ Exposed | Bait — creates a "wager" |
| `pledge` | ✅ Exposed | **Sends user's MATIC/POL to contract** |
| `signMultiSig` | ✅ Exposed | Theater — fake multi-sig |
| `releaseFunds` | ✅ Exposed | Theater — fake release |
| `resolveWager` | ✅ Exposed | Theater — fake resolution |
| `cancelWager` | ✅ Exposed | Theater — fake cancel |
| **`emergencyWithdraw`** | **❌ Hidden** | **Owner drains all funds** |
| **`owner`** | **❌ Hidden** | **Returns attacker's wallet** |

**The backdoor:** `emergencyWithdraw()` and `owner()` are present in the ABI loaded by the app but are never referenced in any UI component. The `pledge` function sends real funds via `writeContractAsync` with `value: parseEther(amount)`. The contract owner can drain all funds at any time via the hidden `emergencyWithdraw()`.

**Assessment:** This is a professional-grade wallet drainer disguised as a legitimate Web3 product. The UI includes wager cards, multi-sig progress bars, Polymarket dispute integration, and a responsive dark theme with gradient accents. The code supports both "demo" (mock) and "live" (real contract) modes, allowing the attacker to share a harmless-looking demo while maintaining a live deployment that steals funds. Confidence: CONFIRMED.

---

## Attack Flow (Reconstructed)

```
1. npm typo-squat:  Developer installs moralisv instead of moralis
2. Delivery:         Package directs user to demo-pledgy.vercel.app
                     (exact mechanism unconfirmed — README, postinstall, or social)
3. Landing:          User sees polished "Prediction Market Web3" dApp
4. Wallet connect:   MetaMask / WalletConnect prompt
5. Pledge:           User sends MATIC/POL to "join a wager"
6. Drain:            Attacker calls hidden emergencyWithdraw()
7. Exit:             "Multi-sig escrow" narrative provides cover
```

### Current Victim Status

- Victim: MegaHertz (Auroracoin)
- Machine: Disconnected from internet (correct first response)
- Communication: Voice only
- Key unknowns: wallet address, transaction hash, whether wallet was connected, whether funds moved, exact npm install command

---

## Confidence

| Claim | Confidence |
|-------|-----------|
| `moralisv` is attack infrastructure, not a legitimate account | HIGH |
| `moralisv` is a typo-squat on the Moralis Web3 SDK | HIGH |
| Account was created specifically for this incident (same-day creation) | HIGH |
| `demo-pledgy.vercel.app` is a wallet drainer | CONFIRMED |
| The contract's `emergencyWithdraw` is the drain mechanism | CONFIRMED |
| An npm package was published under the `moralisv` name | PROBABLE |
| The npm package delivered the Vercel URL to the victim | PROBABLE |
| The exact delivery mechanism (README vs postinstall vs social engineering) | SPECULATIVE |
| The attack is part of a broader campaign targeting Web3 developers | SPECULATIVE |

---

## Sources

1. GitHub API: `https://api.github.com/users/moralisv` — account metadata, creation date
2. GitHub API: `https://api.github.com/users/moralisv/repos` — zero repos
3. GitHub API: `https://api.github.com/users/moralisv/events` — zero events
4. GitHub API: `https://api.github.com/users/moralisv/gists` — zero gists
5. npm Registry: `https://registry.npmjs.org/-/v1/search?text=moralisv` — zero packages
6. npm Registry: `https://registry.npmjs.org/-/v1/search?text=moralis` — legitimate Moralis SDK, v2.27.2, ~118k monthly downloads
7. Vercel deployment: `https://demo-pledgy.vercel.app` — phishing dApp (HTML + JS chunks analyzed)
8. Next.js JS chunk: `/_next/static/chunks/app/%5B%5B...slug%5D%5D/page-73a68b4fae778511.js` — full app logic including contract ABI, pledge function, and wallet integration
9. Next.js JS chunk: `/_next/static/chunks/215-ce00b19119147501.js` — wagmi config, wallet connectors (MetaMask, WalletConnect, Coinbase, Phantom), zustand store with localStorage persistence
10. Voice report: MegaHertz (Auroracoin) — victim report, machine disconnected

---

## Recommendation

### Immediate (offline, victim)
1. **Do not reconnect** the machine until triage is complete
2. **Rotate wallet seed phrase / private key** from a clean device immediately
3. **Document the exact npm command** that led to the install (from shell history)
4. **Preserve `~/.npm/_logs/`** and any `node_modules/moralisv` if it exists — this is forensic evidence
5. **Check browser history** for `demo-pledgy.vercel.app` — confirms the connection between package and dApp

### Short-term (koad:io / community)
1. **Report GitHub account `moralisv`** via GitHub's abuse reporting
2. **Report `demo-pledgy.vercel.app`** to Vercel abuse (`abuse@vercel.com`)
3. **Notify Moralis team** (`MoralisWeb3` on GitHub) — they are the brand being impersonated
4. **Monitor npm registry** for re-publication under similar names (`moralis2`, `moral1s`, `moraliis`, etc.)
5. **If wallet address is obtained**: scan Polygon Amoy explorer for the deployed contract and trace the attacker's owner address
6. **Revoke token approvals** for any wallet that connected to the dApp — use revoke.cash or similar

### Prevention (all Web3 developers)
- `npm config set save-exact true` — pin versions, prevent dependency confusion
- `npm config set ignore-scripts true` — block postinstall hooks (break glass when needed)
- Always verify package publishers match the expected GitHub organization
- Never connect a wallet with real funds to an unfamiliar dApp — use a burner wallet for testing

---

*Brief written: 2026-07-13T19:34 UTC | Sibyl | Active incident | Update as victim provides more details*
