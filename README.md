# Institutional Proof of Promise (PoP)

A production-grade GenVM primitive for trustlessly verifying milestones, grants, and public commitments on GenLayer.

## Overview
Traditional smart contracts rely on human escrow agents or DAO committees to manually verify if a funded team delivered their roadmap (e.g. "launched a beta" or "open-sourced the repo"). This is slow, subjective, and prone to corruption.

**Proof of Promise** leverages GenLayer's native Intelligent Contracts to autonomously resolve public commitments. It reads web evidence and evaluates it semantically against the initial promise obligations using a decentralized LLM jury.

## Why this Primitive is "Institutional Grade"

This contract explicitly addresses the security and trust-model vulnerabilities present in generic URL-fetching Oracles:

### 1. Strict Source-Authority Policy
* **Vulnerability:** Generic fact-checkers often allow callers to provide arbitrary URLs, meaning attackers can define the evidence trust model by linking to their own fake websites.
* **Solution:** At creation, the promise sponsor hardcodes a rigid `trusted_domains` whitelist (e.g., `['github.com', 'twitter.com']`). The `add_evidence` function utilizes Python's `urllib.parse` to aggressively reject any untrusted URL injections. The contract guarantees that the AI only consumes data from pre-approved authoritative sources.

### 2. Graceful Fail-Closed Handling
* **Vulnerability:** Unreachable URLs (404s, timeouts) crash naive GenVM consensus implementations.
* **Solution:** Web acquisition (`gl.nondet.web.render`) is tightly wrapped in `try/except` blocks. If a source is unreachable, the LLM is injected with a strict `ERROR_FETCHING_URL` flag, triggering a graceful degradation of the state to `UNVERIFIABLE` rather than panicking the consensus network.

### 3. Boolean Semantic Consensus
* **Vulnerability:** Forcing exact JSON matching on subjective evaluations fails due to LLM non-determinism (temperature variance). Naive AI contracts crash when nodes generate slightly different confidence scores.
* **Solution:** The contract implements a robust Boolean Semantic Consensus Strategy. Instead of strict numerical equivalence, the Validator extracts and compares only the core deterministic `verdict` (`FULFILLED` vs `UNVERIFIABLE`). The underlying confidence scores are preserved in state for UI rendering, but isolated from the consensus layer to guarantee high-availability execution.

### 4. Advanced Threat Protection (v0.2.16 Update)
* **Caller-Authorization (Evaluation Locks):** To prevent malicious third parties from triggering premature evaluations (before evidence collection is finalized), the `trigger_evaluation` function enforces strict role-based access control. Only the Creator (Funder) or the explicitly assigned Developer can invoke the evaluation process.
* **Prompt Injection Fencing:** Untrusted user inputs and dynamically scraped web content are tightly sandboxed within `<UNTRUSTED_SUBMISSION>` tags. The contract actively sanitizes and purges forged tags from evidence payloads, preventing prompt-breakout attacks designed to manipulate the LLM into rendering fraudulent `FULFILLED` verdicts.

## Reusability
This contract serves as a foundational "Lego block" (primitive) that can be plugged into:
- **Grants DAOs:** Automatically releasing Treasury funds when a milestone reaches `FULFILLED` status.
- **Decentralized Upwork/Escrow:** Adjudicating gig-economy disputes autonomously based on submitted PRs/Links.

## Deployment
- **Network:** GenLayer StudioNet
- **Dependencies:** `py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6`
