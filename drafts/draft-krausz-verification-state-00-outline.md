# IETF Internet-Draft: Outline

**Filename:** `draft-krausz-verification-state-00.txt` / `.md`
**Target filing:** Early June 2026 (per Beenz pitch commitment)
**Working title:** *The verification.\* Constraint Family: Pre-Action Fail-Closed Gates for AI Agent Decisions*
**Companion draft:** `draft-msebenzi-environment-state-00` (Beenz / Headless Oracle)
**Submission path:** [datatracker.ietf.org/submit](https://datatracker.ietf.org/submit/)

---

## Front matter (boilerplate per RFC 7991 / xml2rfc)

- `<rfc>` element: `category="info"` (start informational, request standards track later)
- `ipr="trust200902"`, `submissionType="IETF"`
- `<workgroup>` blank for individual submission v00; target WG: **RATS** (relying-party / attestation results space) — alternate: **HTTPAPI** if framed as `/evaluate` profile.
- Authors: Joseph Krausz (AgentOracle / TK Collective). Co-author hold per ADR-001 until external builders demonstrate independent adoption.

---

## Abstract (~150 words)

Single paragraph. Mention:
- Pre-action verification primitive for AI agents
- Sibling to `environment.*` (verifiable intent / Mastercard) constraint family
- Binary fail-closed gate: `act` / `halt`
- JWS-signed receipts (RFC 7515) with multi-axis freshness
- W3C VC Confidence Method alignment (confidence as sidecar, not gate)
- Differentiates from SCITT (signing only), RATS AR (multi-step), VAP (framework), W3C VC CM (no halt) by being the **gate primitive itself**

---

## Status of This Memo / Copyright Notice

Boilerplate from xml2rfc template. Trust200902.

---

## Table of Contents

(auto-generated; ToC must list §§ 1–11)

---

## 1. Introduction

- AI agents take economic actions on probabilistic input → need fail-closed gate
- Today's options force consumers to write halt logic against confidence scores → fragmentation
- This document specifies the **verification.\* constraint family**: a binary, pre-action, fail-closed gate primitive
- Composable with: x402 (payment), Mastercard Verifiable Intent (`environment.*`), SCITT/RATS (attestation), W3C VC Confidence Method (sidecar metadata)

### 1.1 Requirements Language

Standard RFC 2119 / 8174 block.

### 1.2 Scope

In scope: gate semantics, receipt envelope, calibration discipline, freshness model.
Out of scope: signing key management policy (defers to JWKS standards), specific verifier implementations, claim-extraction techniques.

---

## 2. Terminology

Define:
- **Verifier** — entity issuing a verification receipt
- **Relying Party (RP)** — agent or downstream system consuming the receipt at a gate
- **Gate** — the decision point in the RP's code path where `act` / `halt` is consumed
- **Calibration anchor** — the dataset + seed that defines confidence semantics
- **Multi-axis freshness** — signature, calibration, and evidence each carry independent `valid_until`
- **Verdict (raw)** — `supported` / `refuted` / `unverifiable` — underlying truth label
- **Recommendation (gate)** — `act` / `halt` — the binary fail-closed signal

---

## 3. The `verification.*` Constraint Family

### 3.1 Sibling-not-member relationship to `environment.*`

| Property | `environment.*` | `verification.*` |
|---|---|---|
| Predicate | Boolean | Probabilistic in [0,1] |
| Threshold ownership | Oracle-defined | Calibration-anchored |
| Freshness | Single TTL | Multi-axis |
| Gate shape | Binary halt | Binary halt |

### 3.2 Why sibling and not member

- Predicate shape differs (probabilistic vs boolean)
- Threshold provenance differs (calibration-anchored vs fixed-semantic)
- Both fail closed → same gate primitive shape → sibling

### 3.3 Composition rules

- Multiple `verification.*` constraints in a single mandate combine with AND (orthogonal+conjunctive)
- Ordering: `environment.*` first SHOULD short-circuit before `verification.*` (cheaper, doesn't require oracle roundtrip)

---

## 4. Receipt Format

### 4.1 JWS envelope (RFC 7515)

- `alg`: ES256 mandatory, others optional
- `kid`: resolves via JWKS at issuer's `/.well-known/jwks.json`
- `typ`: `verification-receipt+jws`

### 4.2 Payload claims — canonical / derived / mapping-bound

Standard JWT (RFC 7519): `iss`, `sub`, `iat`, `exp`.

The receipt signs three structurally distinct fields plus their bindings:

**Canonical input (signed):**
- `v_verdict` — `supported|refuted|unverifiable` (signed primitive)
- `v_confidence` — `[0,1]` (signed primitive)
- `v_gate_threshold` — `[0,1]` (signed primitive)
- `v_adversarial_result` — `resilient|vulnerable|not_checked` (signed primitive)
- `v_recommendation` — derived from the above, also signed; enum: `confident_supported | vulnerable_supported | weak_supported | refuted | unverifiable | error`

**Derived output (signed):**
- `v_gate` — `act|halt`

**Binding (signed):**
- `v_gate_mapping` — stable identifier of the published mapping document (e.g. `v0.3.0-2026-05-30`). The mapping document is immutable after publication; future revisions ship as new identifiers.

**Provenance (signed, not gating):**
- `v_calibration` — anchor metadata
- `v_method` — verifier pipeline identifier
- `v_sources_used` — array of source labels
- `v_evidence` — optional URI pointer

### 4.3 Verification protocol

A verifier MUST execute the following sequence and treat any failure as a malformed receipt (halt):

```
1. Verify JWS signature (RFC 7515).
2. Resolve v_gate_mapping → fetch the named, immutable mapping document.
3. Recompute candidate_recommendation from
   (v_verdict, v_confidence, v_gate_threshold, v_adversarial_result)
   using the named mapping's rules.
4. Confirm candidate_recommendation == v_recommendation.
5. Compute candidate_gate = mapping(v_recommendation).
6. Confirm candidate_gate == v_gate.
```

The verifier never trusts the issuer runtime to have applied the mapping correctly. Signatures bind inputs, outputs, and mapping identifier together; the verifier recomputes locally.

### 4.4 Claim binding

- `v_claim` carries `text` OR `hash` (PII-aware redaction)
- Hash-only mode for sensitive content

### 4.5 Version-binding rationale (the "future rules change" case)

Gate-derivation rules will evolve. A receipt signed under mapping `v0.3.0-...` MUST remain verifiable as correct-under-`v0.3.0` after a newer mapping ships. The mapping identifier is the binding that makes this true: a verifier fetches the *same* mapping document the issuer used, regardless of newer revisions. Receipts never silently re-verify to a different gate.

---

## 5. Binary-Halt Gate Semantics

### 5.1 Decision table

| `v_verdict` | confidence ≥ threshold | adversarial flag | gate |
|---|---|---|---|
| supported | yes | resilient / not_checked | act |
| supported | yes | vulnerable | halt |
| supported | no | (any) | halt |
| refuted | (any) | (any) | halt |
| unverifiable | (any) | (any) | halt |
| (any error condition) | — | — | halt |

### 5.2 Threshold rules

- Default threshold: implementation-defined, MUST appear in receipt as `v_gate_threshold`
- Consumers MAY require higher; MUST NOT require lower (gate is the floor)

### 5.3 Fail-closed mandate

If receipt is missing, malformed, expired, or unverifiable → RP MUST treat as `halt`.

---

## 6. Multi-Axis Freshness

### 6.1 Three independent axes

- **Signature freshness**: `exp` claim (key rotation cadence)
- **Calibration freshness**: `v_calibration.valid_until`
- **Evidence freshness**: `v_evidence.valid_until`

### 6.2 Staleness semantics

- Stale signature → invalid receipt (halt)
- Stale calibration → SHOULD re-evaluate; MAY honor as soft-stale per policy
- Stale evidence → SHOULD re-evaluate; MAY honor for retrospective audit

### 6.3 Rationale

Each axis has a different remediation path → can't be collapsed into single TTL.

---

## 7. Calibration Anchor Requirements

### 7.1 Reproducibility mandate

- Verifier MUST publish anchor dataset, seed, version
- Verifier SHOULD publish reproduction harness (Apache or MIT)
- Reference implementation: AgentOracle eval-harness ([github](https://github.com/TKCollective/agentoracle-eval-harness)) reproducing 57.6% AVeriTeC

### 7.2 Drift detection

- Calibration `valid_until` SHOULD reflect known drift bounds
- Verifier SHOULD publish recalibration cadence

### 7.3 Open benchmarks

Recommended: AVeriTeC, FEVER 2.0 (post-2018 contamination concerns noted), domain-specific benchmarks as they emerge.

---

## 8. Relationship to Related Work

### 8.1 SCITT — anchor: `draft-ietf-scitt-architecture`

- The SCITT WG architecture document is the citation anchor (not the individual submissions clustered around it).
- Signing & receipt envelope only — no gate semantics.
- This draft: SCITT-compatible JWS profile; SCITT receipts could carry the `verification.*` payload defined here.

### 8.2 RATS — anchor: RFC 9334

- RFC 9334 is the architecture RFC. It defines the Evidence → Verifier → Attestation Result → Relying Party vocabulary this draft matches.
- This draft: a `verification.*` receipt is an Attestation Result whose gate field (`v_gate`) is the RP-consumable primitive.

### 8.3 W3C VC Confidence Method

- W3C Verifiable Credentials work item. Confidence-as-a-verifiable-property is the pattern this draft adopts for `v_confidence` (metadata, not gate).
- This draft: confidence is signed metadata; halt is the gate. Alignment, not overlap.

### 8.4 VAP framework (`draft-kamimura-vap-framework`)

- Individual I-D. Cite by draft name and pin the specific version read; individual drafts move and expire.
- TODO before -00 submission: read the current version (do not lean on secondhand description), then cite the exact `-NN` revision.
- This draft: a verifier profile under the VAP umbrella if the framework remains stable.

### 8.5 Mastercard Verifiable Intent (`environment.*`)

- Sibling family; binary halt semantics shared
- This draft: extends the family pattern to probabilistic predicates

### 8.6 Differentiator summary

> *Pre-action fail-closed gate. Not signing machinery. Not confidence quantification. The gate primitive itself.*

---

## 9. Security Considerations

- Key compromise → JWKS rotation; receipts pre-rotation remain verifiable until rotation horizon
- Replay attacks → `iat` + `exp` + claim-binding via `v_claim.hash`
- Confidence inflation attacks → calibration anchor + reproducible harness mitigates
- Selective disclosure → hash-only mode for PII
- Downgrade attacks (forcing back to non-binary gate) → MUST NOT honor pre-v0.3 spec receipts at v0.3+ gates

---

## 10. IANA Considerations

- Register `verification-receipt+jws` media type
- Register `v_*` JWT claim names (or coordinate `verification.*` namespace with `draft-msebenzi-environment-state`)
- JWKS well-known path: `/.well-known/jwks.json` (existing, RFC 7517 §4.7)

---

## 11. References

### 11.1 Normative
- RFC 2119, RFC 8174, RFC 7515 (JWS), RFC 7517 (JWK), RFC 7519 (JWT), RFC 7991 (xml2rfc)

### 11.2 Informative
- `draft-msebenzi-environment-state-00` (companion)
- `draft-ietf-scitt-architecture` (SCITT WG architecture; anchor citation)
- RFC 9334 (RATS architecture; Evidence → Verifier → AR → RP vocabulary)
- W3C Verifiable Credentials Confidence Method specification (confidence-as-verifiable-property pattern)
- `draft-kamimura-vap-framework-NN` (VAP — pin exact version before submission; read first)
- Mastercard Verifiable Intent specification
- AVeriTeC dataset (Schlichtkrull et al., 2024)
- AgentOracle receipt spec v0.3 + [ADR-001](https://github.com/TKCollective/agentoracle-receipt-spec/blob/v0.3-binary-halt/adr/ADR-001-binary-halt-gate.md) + [ADR-002](https://github.com/TKCollective/agentoracle-receipt-spec/blob/v0.3-binary-halt/adr/ADR-002-canonical-derived-version-binding.md)
- First published mapping: [mappings/v0.3.0-2026-05-30.md](https://github.com/TKCollective/agentoracle-receipt-spec/blob/v0.3-binary-halt/mappings/v0.3.0-2026-05-30.md)

---

## Authors' Addresses

```
Joseph Krausz
TK Collective
Los Angeles, CA, USA
Email: train@joekrausz.com
```

---

## Filing checklist (for early June)

- [ ] xml2rfc v3 source (`.xml`) — convert this outline
- [ ] Validate with `xml2rfc --v3 --text --html`
- [ ] Generate `.txt` + `.html` from same source
- [ ] Verify all references resolve in datatracker.ietf.org
- [ ] IPR declaration form ([datatracker.ietf.org/ipr](https://datatracker.ietf.org/ipr/))
- [ ] Submit via [datatracker.ietf.org/submit](https://datatracker.ietf.org/submit/)
- [ ] Email RATS WG list announcing the draft
- [ ] Cross-link from receipt-spec README + AgentOracle landing page

---

## Open decisions before filing

1. **WG submission**: RATS vs HTTPAPI vs individual. Recommendation: individual + cross-post to RATS list.
2. **Namespace**: `v_*` short-form vs `verification.*` long-form for JWT claims. Recommendation: `verification.*` for symmetry with `environment.*` family naming.
3. **Co-author**: Beenz invited as co-author? Per ADR-001 external review note, holding co-author slot until external builders independent. Default: solo v00; invite v01 after WG feedback.
4. **Category**: informational (recommended for v00) or standards track? Recommendation: informational; promote after WG adoption.
