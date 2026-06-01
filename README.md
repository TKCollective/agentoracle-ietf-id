# draft-krausz-verification-state — IETF Internet-Draft

[![Status: -00 outline](https://img.shields.io/badge/status---00_outline-1f6feb)](./drafts/draft-krausz-verification-state-00-outline.md)
[![Target submission: June 2026](https://img.shields.io/badge/target_submission-June_2026-success)]()
[![License: BSD-3-Clause](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](./LICENSE)

Public working repository for **`draft-krausz-verification-state-00`** — an IETF Internet-Draft specifying the `verification.*` constraint family: pre-action fail-closed gates for AI agent decisions, with signed receipts, version-bound mapping, and binary act/halt semantics.

## Status

| Milestone | Date | Status |
|---|---|---|
| -00 outline drafted | 2026-05-29 | ✅ |
| External review (Beenz / @headlessoracle) | 2026-05-30 | ✅ — anchor citations and version-binding model incorporated (ADR-002) |
| -00 outline published in this repo | 2026-06-01 | ✅ |
| **-00 manuscript drafted (Markdown)** | **2026-06-01** | **✅** [drafts/draft-krausz-verification-state-00.md](./drafts/draft-krausz-verification-state-00.md) |
| -00 sent for second-round read (Beenz) | 2026-06-02 → 06-03 | 📋 |
| xml2rfc v3 conversion | 2026-06-04 → 06-05 | 📋 |
| Submission to datatracker.ietf.org/submit | 2026-06-06 (target) | 📋 |
| Working group submission email (RATS list) | 2026-06-06 | 📋 |

The outline is what's currently public; the -00 manuscript is in active drafting in xml2rfc v3 format and will be filed to the IETF Datatracker on or before the target date.

## What's in the draft

This Internet-Draft proposes the `verification.*` constraint family — a sibling family to `environment.*` (Mastercard Verifiable Intent / `draft-msebenzi-environment-state`) for pre-action fail-closed gates on AI agent decisions.

**Headline properties:**

- **Pre-action.** Verification happens before the action.
- **Fail-closed.** Anything ambiguous halts. "Impossible, not tedious."
- **Third-party recomputable.** Verifier never trusts the issuer's runtime; signature binds inputs, outputs, and mapping identifier together.
- **Forward-compatible.** Receipts signed under one ruleset stay verifiable as correct-under-that-ruleset forever — version-bound mapping makes this structural, not promised.

**Receipt structure:** JWS envelope (RFC 7515) with canonical input fields, derived output, signed mapping identifier. Verification protocol recomputes derivations locally from signed primitives.

## Related work

The draft references and aligns with:

| Standard / Draft | Anchor citation | Layer |
|---|---|---|
| RFC 9334 (RATS Architecture) | Evidence → Verifier → AR → RP vocabulary | Attestation flow |
| `draft-ietf-scitt-architecture` | SCITT WG architecture document | Signing & receipt envelope |
| W3C VC Confidence Method | Verifiable Credentials work item | Confidence-as-property |
| `draft-kamimura-vap-framework` | Individual I-D, version pinned at submission | Verifier abstraction |
| `draft-msebenzi-environment-state-00` | Companion draft (sibling family) | Environment-state attestation |

## Reference implementations and ecosystem

- **Receipt spec v0.3** (binary-halt + canonical/derived/version-bound): [TKCollective/agentoracle-receipt-spec](https://github.com/TKCollective/agentoracle-receipt-spec/tree/v0.3-binary-halt)
- **Open-source verifier client** (`@agentoracle/receipt-verify`): [TKCollective/agentoracle-receipt-verify](https://github.com/TKCollective/agentoracle-receipt-verify)
- **Reference evaluation harness** (MIT, reproducing 57.6% AVeriTeC): [TKCollective/agentoracle-eval-harness](https://github.com/TKCollective/agentoracle-eval-harness)
- **Cross-operator benchmark** (open methodology, open submissions): [TKCollective/agentoracle-benchmark](https://github.com/TKCollective/agentoracle-benchmark)
- **Live deployment**: [agentoracle.co](https://agentoracle.co)

## Reading the draft

**Current public artifact:** the full **-00 manuscript** at [drafts/draft-krausz-verification-state-00.md](./drafts/draft-krausz-verification-state-00.md). It contains all eleven sections, normative receipt-format definition, verification protocol, security considerations, IANA considerations, and references.

The earlier **structural outline** is at [drafts/draft-krausz-verification-state-00-outline.md](./drafts/draft-krausz-verification-state-00-outline.md) and is preserved for change tracking.

Next step: second-round read by Beenz / @headlessoracle (per his May 30 commitment), then xml2rfc v3 conversion, then submission to datatracker.ietf.org/submit. When filed, this README will be updated with the Datatracker URL.

## Reviewers welcome

If you've shipped a verifier, a receipt format, or a pre-action gate primitive in any agent infrastructure stack — your read on the -00 manuscript is genuinely valuable before submission. Open an issue or email Joe@agentoracle.co.

The draft is positioned to be a **community technical document**, not a single-vendor proposal. Co-authoring the v01+ revisions is open to operators with shipped, independent implementations.

## License

Outline content: BSD-3-Clause to align with IETF document conventions. Files individually marked otherwise are exempt.

## Author

Joseph Krausz · TKCollective · `Joe@agentoracle.co` · [agentoracle.co](https://agentoracle.co)
