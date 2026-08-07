# 🏛️ Institutional Proof of Promise — Escrow with AI Verification

**Contract (GenVM StudioNet):** `0xa315f1d65D476B1b667feb80f763666fd8a75505`
**Explorer:** https://explorer-studio.genlayer.com/address/0xa315f1d65D476B1b667feb80f763666fd8a75505
**GitHub:** https://github.com/nikvn89/institutional_proof_of_promise


---

## Overview

An on-chain escrow contract for institutional accountability. Promises are funded with a bounty, and GenLayer's LLM validators independently verify whether the promise was fulfilled by scraping web evidence from trusted domains.

---

## V3 Fixes (Joaquin's Feedback)

| Issue | V2 (old) | V3 (fixed) |
|---|---|---|
| **Developer assignment** | First `add_evidence` caller becomes dev — anyone can self-assign | Creator explicitly assigns `dev_address` at `create_promise()` |
| **Evidence submission** | Any account can add evidence | Only assigned developer can submit |
| **Evaluation timing** | Can trigger at any time, before deadline | `current_ts >= deadline` enforced; rejects premature evaluation |
| **Payout/Refund** | Transfer was commented out — no money moved | Full lifecycle: FULFILLED→dev, BROKEN→refund, PARTIAL→50/50 split |
| **Self-dealing** | Creator could be their own developer | `dev_address != creator` enforced |

---

## Security Architecture

| Property | Implementation |
|---|---|
| **Creator-Assigned Developer** | `dev_address` set at `create_promise()` by creator only |
| **Evidence Restricted** | Only `dev_address` can call `add_evidence()` |
| **Deadline Enforcement** | `current_ts >= deadline` required before evaluation |
| **Caller Authorization** | Only creator or dev can trigger evaluation |
| **Full Payout Lifecycle** | FULFILLED→100% dev, PARTIAL→50/50, BROKEN/UNVERIFIABLE→100% refund |
| **Re-entrancy Protection** | Bounty zeroed before any `gl.transfer()` |
| **Prompt Injection Fencing** | Evidence wrapped in `<UNTRUSTED>` blocks |
| **Bounded Schema Validation** | Verdict validated, score clamped 0-100, reason ≤ 280 chars |
| **Fail-Closed** | Errors default to UNVERIFIABLE → refund to creator |

---

## Contract Methods

| Method | Who | Description |
|---|---|---|
| `create_promise(id, statement, deadline_ts, domains, dev_address)` | Creator (payable) | Fund a promise + assign developer |
| `add_evidence(id, url)` | Developer only | Submit evidence URL (domain-whitelisted) |
| `trigger_evaluation(id, current_ts)` | Creator or Developer | AI evaluates after deadline |
| `get_promise(id)` | Anyone | View promise state + evidence |
| `get_all_promises()` | Anyone | View all promises |

---

## Promise Lifecycle

```
create_promise (ACTIVE, bounty locked, dev assigned)
       ↓
add_evidence (dev submits URLs)
       ↓
trigger_evaluation (after deadline)
       ↓
   ┌──────────────┬──────────────────┬──────────────┐
FULFILLED      PARTIALLY_FULFILLED  BROKEN/UNVERIFIABLE
100% → dev     50% dev / 50% creator   100% → creator refund
```

---

## Security Properties Verified

- ✅ Developer assigned by creator at creation — not self-assignable
- ✅ Only assigned developer can submit evidence
- ✅ Evaluation blocked before deadline
- ✅ Only creator or developer can trigger evaluation
- ✅ Complete payout/refund for all verdict outcomes
- ✅ Re-entrancy protected (bounty zeroed before transfer)
- ✅ Self-dealing prevented (creator ≠ developer)
- ✅ Bounded schema validation on AI results
- ✅ Fail-closed design (errors → UNVERIFIABLE → refund)
