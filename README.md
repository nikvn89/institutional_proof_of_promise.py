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

### 3. Semantic Banding Consensus
* **Vulnerability:** Forcing exact JSON matching on subjective evaluations fails due to LLM non-determinism (temperature variance).
* **Solution:** The contract implements a Semantic Consensus Strategy. Instead of strict numerical equivalence, the Validator maps the `confidence_score` into deterministic bands (`0-34`, `35-79`, `80-100`). This ensures robust mathematical consensus on subjective material outcomes without failing on prose variance.

## Reusability
This contract serves as a foundational "Lego block" (primitive) that can be plugged into:
- **Grants DAOs:** Automatically releasing Treasury funds when a milestone reaches `FULFILLED` status.
- **Decentralized Upwork/Escrow:** Adjudicating gig-economy disputes autonomously based on submitted PRs/Links.

## Deployment
- **Network:** GenLayer StudioNet
- **Dependencies:** `py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6`
