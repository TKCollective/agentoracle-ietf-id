# draft-krausz-verification-state — IETF Internet-Draft

[![Status: -02 posted](https://img.shields.io/badge/status--02_posted-1f6feb)](https://datatracker.ietf.org/doc/draft-krausz-verification-state/)
[![Datatracker: I-D Exists](https://img.shields.io/badge/datatracker-I--D_Exists-success)](https://datatracker.ietf.org/doc/draft-krausz-verification-state/)
[![License: BSD-3-Clause](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](./LICENSE)

Public working repository for **`draft-krausz-verification-state`** — an IETF Internet-Draft specifying the `verification.*` constraint family: pre-action fail-closed gates for AI agent decisions, with signed receipts, version-bound mapping, and binary act/halt semantics.

## Status

**Current revision: `-02`, posted 23 September 2026.** Read it on the datatracker: **[datatracker.ietf.org/doc/draft-krausz-verification-state/](https://datatracker.ietf.org/doc/draft-krausz-verification-state/)**.

| Milestone | Date | Status |
|---|---|---|
| -00 outline drafted | 2026-05-29 | ✅ |
| External review (Beenz / @headlessoracle) | 2026-05-30 | ✅ — anchor citations and version-binding model incorporated (ADR-002) |
| -00 manuscript filed to IETF Datatracker | 2026-06-06 | ✅ |
| -01 published | 2026-06-12 | ✅ |
| -02 published | 2026-09-23 | ✅ — four-state vocabulary (`verified`/`contradicted`/`indeterminate`/`not_evaluated`), reason-code mechanism (including `instrument_failure`), admissibility gate, evidence pinning |

The full `-02` text is in this repo at [drafts/draft-krausz-verification-state-02.xml](./drafts/draft-krausz-verification-state-02.xml) (source), [.txt](./drafts/draft-krausz-verification-state-02.txt), and [.html](./drafts/draft-krausz-verification-state-02.html) (rendered), along with [drafts/CHANGES-01-to-02.md](./drafts/CHANGES-01-to-02.md) documenting exactly what changed since `-01`. Earlier revisions (`-00`, `-01`) are preserved in this repo's history for change tracking.

## What's in the draft

This Internet-Draft proposes the `verification.*` constraint family — a sibling family to `environment.*` (Verifiable Intent / `draft-borthwick-msebenzi-environment-state`) for pre-action fail-closed gates on AI agent decisions.

**Headline properties:**

- **Pre-action.** Verification happens before the action.
- **Fail-closed.** Anything ambiguous halts. "Impossible, not tedious" — Anthropic's framing for this design test `[ANTHROPIC-ZT]`.
- **Third-party recomputable.** Verifier never trusts the issuer's runtime; signature binds inputs, outputs, and mapping identifier together.
- **Forward-compatible.** Receipts signed under one ruleset stay verifiable as correct-under-that-ruleset forever — version-bound mapping makes this structural, not promised.

**As of `-02`:** a four-state verdict vocabulary (`verified`, `contradicted`, `indeterminate`, `not_evaluated`) replaces `-01`'s raw `supported`/`refuted`/`unverifiable`/`unknown` domain one-for-one, with a `v_reason_code` mechanism that separates a state's substance from a verifier's own instrument failure (`instrument_failure` is available under all four states), an admissibility gate applied before any state is assigned, and optional evidence pinning (`evidence_set`) so a verdict can be recomputed offline from the receipt and the pinned bytes alone.

**Receipt structure:** JWS envelope (RFC 7515) with canonical input fields, derived output, signed mapping identifier, and claim binding (`v_claim`). Verification protocol recomputes derivations locally from signed primitives.

## Related work

The draft references and aligns with:

| Standard / Draft | Anchor citation | Layer |
|---|---|---|
| RFC 9334 (RATS Architecture) | Evidence → Verifier → AR → RP vocabulary | Attestation flow |
| RFC 9942 (COSE Receipts) | Receipt structure | Signing & receipt envelope |
| RFC 9943 (Trustworthy and Transparent Digital Supply Chains) | Architecture | Attestation flow |
| `draft-borthwick-msebenzi-environment-state` | Companion draft (sibling family) | Environment-state attestation |
| Anthropic's Zero Trust for AI Agents framework `[ANTHROPIC-ZT]` | Phase 8, "Measure what matters" | Industry framework alignment |

## Reference implementations and ecosystem

- **Receipt spec v0.3** (binary-halt + canonical/derived/version-bound): [TKCollective/agentoracle-receipt-spec](https://github.com/TKCollective/agentoracle-receipt-spec/tree/v0.3-binary-halt)
- **Reference verifier** (TypeScript, npm `@agentoracle/receipt-verify`): [TKCollective/agentoracle-receipt-verify](https://github.com/TKCollective/agentoracle-receipt-verify)
- **Reference evaluation harness** (MIT; 57.6% AVeriTeC measured 28 May 2026 on the pre-migration pipeline): [TKCollective/agentoracle-eval-harness](https://github.com/TKCollective/agentoracle-eval-harness)
- **Cross-operator benchmark** (open methodology, open submissions): [TKCollective/agentoracle-benchmark](https://github.com/TKCollective/agentoracle-benchmark)
- **Production deployment**: [tanilo.io](https://tanilo.io)

## Reading the draft

**Current public artifact:** the full **`-02`** text, published on the IETF Datatracker at [datatracker.ietf.org/doc/draft-krausz-verification-state/](https://datatracker.ietf.org/doc/draft-krausz-verification-state/), and mirrored in this repo at [drafts/draft-krausz-verification-state-02.xml](./drafts/draft-krausz-verification-state-02.xml) / [.txt](./drafts/draft-krausz-verification-state-02.txt) / [.html](./drafts/draft-krausz-verification-state-02.html). [drafts/CHANGES-01-to-02.md](./drafts/CHANGES-01-to-02.md) documents every change from `-01`.

Earlier artifacts — the `-00` manuscript and structural outline — remain in [drafts/](./drafts/) for change tracking.

## Reviewers welcome

If you've shipped a verifier, a receipt format, or a pre-action gate primitive in any agent infrastructure stack — your read on `-02` is genuinely valuable. Open an issue or email joe@tanilo.io.

The draft is positioned to be a **community technical document**, not a single-vendor proposal. Co-authoring future revisions is open to operators with shipped, independent implementations — `-02`'s Open Issues section names where that stands today, including the from-text cold reconstruction of the evidence-pinning rules currently in progress.

## License

Outline and README content: BSD-3-Clause to align with IETF document conventions. Files individually marked otherwise are exempt.

## Author

Joe Krausz · TK Collective LLC. · `joe@tanilo.io` · [tanilo.io](https://tanilo.io)
