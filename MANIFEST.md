# MANIFEST — moralisv npm Typo-Squat Wallet Drainer

**Audit ID:** `janus-audit-2026-07-13-moralisv`
**Auditor:** Janus
**Opened:** 2026-07-13T19:41 UTC
**Updated:** 2026-07-13T20:30 UTC (victim self-audit + remediation guides)
**Classification:** ~~Supply-chain phishing → wallet drainer~~ → **Two separate incidents discovered:**
  1. Supply-chain phishing → wallet drainer (`moralisv` / `demo-pledgy.vercel.app`)
  2. Supply-chain backdoor via npm `prepare` hook (Moralis-MVP — this source archive)
**Confidence:** CONFIRMED (backdoor in Moralis-MVP) / PROBABLE (npm delivery for moralisv drainer)
**Status:** ACTIVE — source analysis complete, IOCs expanded

---

## Artifact Index

| # | Artifact | Path | Type | Source | Classification | Confidence |
|---|----------|------|------|--------|----------------|------------|
| 1 | Sibyl's Research Brief | `evidence/2026-07-13-sibyl-brief.md` | Analysis report | Sibyl (koad:io) | — | — |
| 2 | Janus Threat Analysis | `analysis.md` | Threat assessment | Janus (this audit) | — | — |
| 3 | Indicators of Compromise | `indicators.md` | IOC catalog | Derived from Sibyl brief + source analysis | — | — |
| 4 | Detection Recommendations | `recommendations.md` | Detection rules | Janus (this audit) | — | — |
| 5 | Evidence README | `evidence/README.md` | Gap analysis | Janus (this audit) | — | — |
| 6 | MegaHertz Source Archive | `moralis-project.zip` (26.7MB) | Source code zip | MegaHertz (victim) via Juno | BACKDOOR CONFIRMED | HIGH |
| 7 | Analysis Supplement | `analysis-supplement.md` | Extraction results | Janus (this flight) | — | — |
| 8 | Deobfuscated Backdoor Payload | `evidence/source-analysis/backdoor-payload-deobfuscated.md` | Malware analysis | Janus (this flight) | BACKDOOR | CRITICAL |
| 9 | Full Source Findings | `evidence/source-analysis/source-findings.md` | Source audit | Janus (this flight) | — | — |
| 10 | Victim Self-Audit Guide | `SELF-AUDIT.md` | Exposure assessment + evidence preservation | Janus (this flight) | — | — |
| 11 | Remediation Checklist | `REMEDIATION.md` | Per-platform cleanup + credential rotation | Janus (this flight) | — | — |

## Missing Artifacts

| # | Artifact | Why Missing |
|---|----------|-------------|
| M1 | Raw GitHub API response (`/users/moralisv`) | Not archived by Sibyl; response values quoted inline in brief |
| M2 | Raw npm registry search response | Not archived by Sibyl; summary values quoted |
| M3 | `demo-pledgy.vercel.app` page HTML capture | Janus bond lacks network fetch capability. Page may still be live — capture when possible. |
| M4 | Next.js JS chunk: `app/%5B%5B...slug%5D%5D/page-73a68b4fae778511.js` | Analyzed by Sibyl; raw chunk not archived |
| M5 | Next.js JS chunk: `215-ce00b19119147501.js` | Analyzed by Sibyl; raw chunk not archived |
| M6 | Victim wallet address / transaction hash | Victim machine offline; voice-only communication |
| M7 | ~~`node_modules/moralisv` directory~~ | ✅ RESOLVED: Not present in extracted source — this is a different attack |
| M8 | ~~Victim shell history~~ | ✅ RESOLVED: Not present in extracted source |
| M9 | GitHub abuse report confirmation | Not yet filed |
| M10 | Vercel abuse report confirmation | Not yet filed |
| M11 | C2 server takedown (51.178.11.177) | Not yet filed |
| M12 | C2 server takedown (216.250.252.245) | Not yet filed |
| M13 | domain `moralisus.com` abuse report | Not yet filed |

## Key Discovery: Two Separate Attacks

The source code extraction revealed that the zip archive contains a **different attack** from the `moralisv`/`demo-pledgy` drainer:

| | moralisv/demo-pledgy Drainer | Moralis-MVP Backdoor |
|---|---|---|
| **Vector** | Typo-squat npm package → Vercel dApp | Brand impersonation → npm `prepare` hook |
| **Mechanism** | Smart contract `emergencyWithdraw()` | Environment exfiltration + RCE backdoor |
| **C2** | Vercel deployment | OVH VPS: `51.178.11.177:1224` (was `216.250.252.245:1224`) |
| **Tech** | Next.js 14 / MUI / wagmi | React 16 / Express / Socket.IO |
| **GitHub** | `moralisv` account (created July 9) | `moralisv` + `MoralisMD` + `Rechard Treslove` |

The victim's machine contained the Moralis-MVP backdoor project — not the demo-pledgy drainer source.

## Audit Template Version

This is Janus's first audit. The structure established here serves as the template:

```
audits/<slug>/
  MANIFEST.md           — artifact index, classification, confidence, status
  analysis.md           — Janus threat assessment: what was missed, what would trip
  analysis-supplement.md — source code extraction results (this file)
  indicators.md         — IOCs in structured format
  recommendations.md    — detection rules for Janus going forward
  evidence/
    README.md           — what's here, what's missing, why
    source-analysis/    — source code analysis artifacts
    <artifacts>         — archived copies of source material, API responses, captures
```

## Status Transitions

```
ACTIVE → STABILIZED (victim secured, triage complete)
       → CLOSED (all actions complete, recommendations filed)
       → SUPERSEDED (replaced by later audit of same campaign)
```

---

*Generated by Janus | 2026-07-13T19:41 UTC | Updated 2026-07-13T20:30 UTC | Flights: `20260713T195536-467Z-janus-c33e3c`, `20260713T200858-398Z-janus-a75926`*
