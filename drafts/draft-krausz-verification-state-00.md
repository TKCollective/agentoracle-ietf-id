%%%
title = "The verification.* Constraint Family: Pre-Action Fail-Closed Gates for AI Agent Decisions"
abbrev = "verification.* family"
ipr = "trust200902"
area = "General"
workgroup = ""
keyword = ["agent", "verification", "receipt", "JWS", "gate", "AI"]
docname = "draft-krausz-verification-state-00"
date = 2026-06-06

[seriesInfo]
stream = "IETF"
name = "Internet-Draft"
value = "draft-krausz-verification-state-00"
status = "informational"

[[author]]
initials = "J."
surname = "Krausz"
fullname = "Joseph Krausz"
organization = "TK Collective"
  [author.address]
  email = "Joe@agentoracle.co"
  uri = "https://agentoracle.co"
  [author.address.postal]
  city = "Los Angeles"
  region = "CA"
  country = "USA"
%%%

## Abstract

This document specifies the `verification.*` constraint family --- a pre-action, fail-closed gate primitive for AI agent decisions, sibling in shape to the `environment.*` family used in Verifiable Intent specifications. A `verification.*` receipt is a JWS-signed artifact ([@RFC7515]) carrying a canonical input, a derived binary `act`/`halt` output, and a versioned mapping identifier that binds them. A relying party recomputes the gate locally from signed primitives under the named mapping; the verifier never trusts the issuer's runtime. This shape provides decision explainability and traceability evidence aligned with EU AI Act Article 12 record-keeping obligations and with the Decision Explainability tier of Anthropic's Zero Trust for AI Agents framework [@ANTHROPIC-ZT]. The format is forward-compatible across mapping revisions: receipts signed under one mapping ID remain verifiable as correct-under-that-mapping after newer mappings ship.

{mainmatter}

## Introduction

AI agents increasingly take economic and operational actions on probabilistic input. A customer-service agent posts content; a procurement agent commits to a supplier; a research agent files a regulatory report. Each action is preceded by a factual claim the agent believes to be true. Today, the infrastructure verifying whether the claim is true *before* the agent acts on it is fragmented across operator logs (after-the-fact), confidence scores (soft signals), and ad-hoc validation pipelines (not interoperable).

This document specifies the `verification.*` constraint family: a pre-action, fail-closed gate primitive shaped identically to the `environment.*` family used in Verifiable Intent specifications [@VINTENT], but applied to probabilistic predicates rather than boolean state. A `verification.*` receipt asserts, for a specific claim under a specific ruleset at a specific moment, whether an agent SHOULD proceed or halt. The receipt is JWS-signed [@RFC7515] and structured such that a relying party can recompute the gate decision locally from signed primitives, without trusting the issuer's runtime.

The four properties this primitive needs to satisfy together:

1. **Pre-action.** The verification happens before the action.
2. **Fail-closed.** Anything ambiguous halts; "impossible, not tedious."
3. **Third-party recomputable.** The verifier never trusts the issuer's runtime; signature binds inputs, outputs, and mapping identifier together.
4. **Forward-compatible.** Receipts signed under one ruleset remain verifiable as correct-under-that-ruleset even after newer rulesets ship.

Property 3 is the load-bearing one and the one most existing primitives stop short of. SCITT [@SCITT] signs receipts (necessary but not sufficient). W3C Verifiable Credentials Confidence Method [@VC-CM] exposes confidence as a verifiable property (useful but not gating). RATS [@RFC9334] provides the Evidence->Verifier->AR->RP vocabulary (matched here). None fully recompose the gate verdict from signed inputs. Property 4 is the one that becomes a footgun if skipped: rule changes silently invalidate or mis-verify old artifacts.

This document is complementary to security frameworks for AI agent deployment, including the Zero Trust framework for AI Agents published by Anthropic in May 2026 [@ANTHROPIC-ZT], which names decision explainability as non-optional for regulated AI systems. The `verification.*` constraint family is one credible artifact shape for satisfying that explainability requirement.

### Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@RFC2119] [@RFC8174] when, and only when, they appear in all capitals, as shown here.

### Scope

In scope:

- Gate semantics for pre-action verification of factual claims.
- Receipt envelope format and required claims.
- Calibration anchor requirements and discipline.
- Multi-axis freshness model.
- Verification protocol for relying parties.

Out of scope:

- Signing key management policy (defers to JWKS standards).
- Specific verifier implementations or model architectures.
- Claim-extraction techniques (how a claim is identified from agent input/output).
- Adversarial-probe methodology specifics.

## Terminology

**Verifier** --- Entity issuing a verification receipt.

**Relying Party (RP)** --- Agent or downstream system consuming the receipt at a gate.

**Gate** --- The decision point in the RP's code path where `act` / `halt` is consumed.

**Calibration anchor** --- The dataset and seed defining confidence semantics for the verifier's pipeline.

**Multi-axis freshness** --- Signature, calibration, and evidence each carry independent validity periods.

**Verdict (raw)** --- One of `supported`, `refuted`, `unverifiable`, `unknown`. The underlying truth label.

**Recommendation (canonical)** --- One of `confident_supported`, `vulnerable_supported`, `weak_supported`, `refuted`, `unverifiable`, `error`. Derived from primitives under the named mapping.

**Gate (derived)** --- `act` or `halt`. Derived from recommendation under the named mapping.

**Mapping document** --- A published, immutable document specifying how recommendations are derived from primitives and how gates are derived from recommendations. Identified by a stable string identifier (e.g., `v0.3.0-2026-05-30`).

## The verification.* Constraint Family

### Sibling-not-Member Relationship to environment.*

| Property | `environment.*` | `verification.*` |
|---|---|---|
| Predicate | Boolean (state matches or does not) | Probabilistic in \[0,1\] |
| Threshold ownership | Oracle-defined, fixed-semantic | Calibration-anchored, mapping-versioned |
| Freshness | Single TTL | Multi-axis |
| Gate shape | Binary halt | Binary halt |

Both families produce a binary fail-closed gate primitive. They differ in predicate shape (boolean vs probabilistic) and threshold provenance (fixed vs calibration-anchored). This difference is sufficient to warrant a sibling family rather than a member entry under `environment.*`: probabilistic predicates require calibration discipline that boolean predicates do not, and the version-binding mechanism described in {{receipt-format}} is specific to mappings between probabilistic primitives and binary outputs.

### Composition Rules

**Conjunction:** Multiple `verification.*` constraints in a single mandate combine with AND (orthogonal+conjunctive). All constraints MUST resolve to `act` for the action to proceed.

**Ordering with environment.*:** `environment.*` constraints SHOULD short-circuit before `verification.*` constraints. Environment evaluation is typically cheaper (no oracle roundtrip) and a failed environment constraint moots the verification.

## Receipt Format {#receipt-format}

### JWS Envelope

Verification receipts MUST be issued as JSON Web Signatures (JWS) [@RFC7515] in compact serialization.

- `alg`: ES256 mandatory. Other algorithms optional and SHOULD follow IANA JOSE Algorithms registry.
- `kid`: MUST resolve to a key in the issuer's published JWKS [@RFC7517] at `/.well-known/jwks.json` unless an alternate JWKS location is published in the issuer's metadata.
- `typ`: `verification-receipt+jws`.

### Payload Claims --- Canonical, Derived, Mapping-Bound

The payload signs three structurally distinct field groups plus their bindings.

**Canonical input (signed primitives):**

- `v_verdict` (string, REQUIRED) --- One of `supported`, `refuted`, `unverifiable`, `unknown`.
- `v_confidence` (number, REQUIRED) --- Float in `[0, 1]`.
- `v_gate_threshold` (number, REQUIRED) --- Float in `[0, 1]`. The confidence floor used to derive the recommendation.
- `v_adversarial_result` (string, REQUIRED) --- One of `resilient`, `vulnerable`, `not_checked`.

**Canonical derived (signed):**

- `v_recommendation` (string, REQUIRED) --- One of `confident_supported`, `vulnerable_supported`, `weak_supported`, `refuted`, `unverifiable`, `error`. Derived deterministically from the canonical input under the named mapping.

**Derived output (signed):**

- `v_gate` (string, REQUIRED) --- One of `act`, `halt`. Derived from `v_recommendation` under the named mapping.

**Binding (signed):**

- `v_gate_mapping` (string, REQUIRED) --- Stable identifier of the published mapping document used at issuance (e.g., `v0.3.0-2026-05-30`). The mapping document is immutable after publication; future revisions ship as new identifiers.

**Provenance (signed, not gating):**

- `v_method` (string, OPTIONAL) --- Self-describing verifier pipeline identifier.
- `v_calibration` (object, OPTIONAL) --- Calibration anchor metadata. See {{calibration-anchor-requirements}}.
- `v_sources_used` (array of strings, OPTIONAL) --- Source labels actually consulted in this evaluation.
- `v_evidence` (string, OPTIONAL) --- URI pointer to evidence corpus, if persisted.

**Standard JWT claims [@RFC7519]:**

- `iss` (REQUIRED), `sub` (RECOMMENDED), `iat` (REQUIRED), `exp` (REQUIRED), `nbf` (OPTIONAL).

### Verification Protocol

A relying party verifying a receipt MUST execute the following sequence and treat any failure as a malformed receipt with gate decision `halt`:

~~~ text
1. Verify JWS signature against issuer's published JWKS (RFC 7515).
2. Resolve v_gate_mapping -> fetch the named immutable mapping document.
3. Recompute candidate_recommendation from
   (v_verdict, v_confidence, v_gate_threshold, v_adversarial_result)
   using the mapping's rules.
4. Confirm candidate_recommendation == v_recommendation.
5. Compute candidate_gate = mapping(v_recommendation).
6. Confirm candidate_gate == v_gate.
7. Verify exp/nbf against current time, subject to clock tolerance
   per Section 6.
8. If all match -> receipt is valid AND internally consistent under
   the named mapping. Any mismatch -> malformed; gate decision = halt.
~~~

The relying party never trusts the issuer's runtime to have applied the mapping correctly. The signature binds inputs, outputs, and mapping identifier together; the verifier recomputes locally.

### Claim Binding

The verified claim MUST be bound to the receipt via `v_claim`:

~~~ json
"v_claim": {
  "text": "string (OPTIONAL when caller marks input as PII-sensitive)",
  "hash": "hex-encoded SHA-256 of canonicalized claim text (REQUIRED when text is omitted)"
}
~~~

Hash-only mode permits PII-sensitive verification while preserving the receipt's auditability --- the verifier can confirm a future caller's claim hash matches the issued receipt's bound hash without exposing the claim text.

### Version-Binding Rationale

Gate-derivation rules will evolve. A receipt issued under mapping `v0.3.0-...` MUST remain verifiable as *correct-under-`v0.3.0`* after a newer mapping ships. The mapping identifier in `v_gate_mapping` is the binding that makes this true: a verifier fetches the *same* mapping document the issuer used at issuance, regardless of newer revisions. Receipts never silently re-verify to a different gate.

Mapping document publishers MUST treat published mappings as immutable. New rules ship under new mapping identifiers. Errata (typographical corrections only) MAY be appended to a published mapping document's metadata, but the normative rule tables MUST NOT change after first publication.

## Binary-Halt Gate Semantics

### Decision Table

The reference mapping `v0.3.0-2026-05-30` defines the following decision table. Future mappings MAY tighten or extend this table; the structural shape (binary act/halt output derived from canonical recommendation) MUST be preserved.

| `v_verdict` | `v_confidence` vs threshold | `v_adversarial_result` | `v_recommendation` | `v_gate` |
|---|---|---|---|---|
| `supported` | >= threshold | `resilient` or `not_checked` | `confident_supported` | `act` |
| `supported` | (any) | `vulnerable` | `vulnerable_supported` | `halt` |
| `supported` | < threshold | `resilient` or `not_checked` | `weak_supported` | `halt` |
| `refuted` | (any) | (any) | `refuted` | `halt` |
| `unverifiable` or `unknown` | (any) | (any) | `unverifiable` | `halt` |
| (error condition) | N/A | N/A | `error` | `halt` |

### Threshold Rules

The `v_gate_threshold` value MUST be present in every receipt. Relying parties MAY require higher thresholds for their own gate policies; they MUST NOT lower the threshold below the issuer's signed value. The receipt's threshold is the floor.

### Fail-Closed Mandate

If a receipt is missing, malformed, expired, signature-invalid, or its mapping ID cannot be resolved, the relying party MUST treat the gate decision as `halt`. Implementations MUST NOT default to `act` under any error condition. This is the "impossible, not tedious" design principle: friction-based controls bypass under adversarial pressure; hard barriers do not.

## Multi-Axis Freshness

### Three Independent Axes

Verification receipts have three independent staleness axes, each with its own validity window:

| Axis | Field | Remediation |
|---|---|---|
| Signature | `exp` | Key rotation; verifier MUST reject expired signatures |
| Calibration | `v_calibration.valid_until` | Recalibrate verifier pipeline against current anchor |
| Evidence | `v_evidence.valid_until` (OPTIONAL) | Re-fetch evidence; usually shorter window than calibration |

### Staleness Semantics

- **Stale signature** --- Receipt is invalid; relying party MUST treat as `halt`.
- **Stale calibration** --- Receipt SHOULD be re-evaluated; relying party MAY honor as soft-stale per its own policy. Verifier issuers SHOULD publish recalibration cadence in metadata.
- **Stale evidence** --- Receipt SHOULD be re-evaluated. MAY be honored for retrospective audit purposes (e.g., post-incident review of a past action) but MUST NOT be honored as a gate signal for new actions.

Each axis has a different remediation path; collapsing them into a single TTL would force the most-restrictive cadence to govern all three.

## Calibration Anchor Requirements {#calibration-anchor-requirements}

### Reproducibility Mandate

Verifier issuers MUST publish:

- Anchor dataset name and version.
- Anchor seed (or seeded run identifier).
- Anchor scoring methodology.

Verifier issuers SHOULD publish a reference reproduction harness under a permissive open-source license (Apache 2.0, MIT, or BSD). A working reference implementation reproducing 57.6% on AVeriTeC [@AVERITEC] is available at https://github.com/TKCollective/agentoracle-eval-harness as one example of conformance.

### Drift Detection

Verifier issuers SHOULD publish a recalibration cadence in the JWKS metadata or the verifier's well-known endpoint. The `v_calibration.valid_until` field in receipts SHOULD reflect this cadence. Relying parties consuming receipts with stale calibration SHOULD log and surface the staleness rather than silently accept.

## Relationship to Related Work

### SCITT

The SCITT WG architecture document [@SCITT] defines the Supply Chain Integrity, Transparency, and Trust signing and receipt envelope. This document is SCITT-compatible: a `verification.*` receipt MAY be wrapped in a SCITT envelope, and SCITT receipts MAY carry `verification.*` payloads. SCITT specifies the signing infrastructure; this document specifies a gate primitive that may be transported via SCITT or via plain JWS.

### RATS

RFC 9334 [@RFC9334] defines the Remote Attestation Architecture's Evidence->Verifier->Attestation Result->Relying Party vocabulary. A `verification.*` receipt is an Attestation Result whose gate field (`v_gate`) is the RP-consumable primitive. This document is consistent with the RATS vocabulary and may be considered a verification-specific Attestation Result profile.

### W3C VC Confidence Method

The W3C Verifiable Credentials Confidence Method [@VC-CM] defines confidence-as-a-verifiable-property within Verifiable Credentials. This document adopts the same pattern for `v_confidence` (signed metadata, not gating). Confidence is provenance; the gate is the contract. Alignment, not overlap.

### VAP Framework

The Verifier Abstraction Pattern framework [@VAP-FW] is an individual Internet-Draft providing verifier abstraction conventions. A `verification.*` receipt issuer MAY be considered a verifier profile under the VAP umbrella. (Editorial note: this section's citation MUST be pinned to a specific VAP version at submission time; individual drafts move and expire.)

### Mastercard Verifiable Intent (environment.*)

Mastercard's Verifiable Intent specification [@VINTENT] defines the `environment.*` constraint family for pre-action attestation of environment state. This document extends the constraint-family pattern to probabilistic predicates with calibration discipline, as a sibling family rather than a member entry. See {{the-verification.-constraint-family}}.

### Differentiator Summary

Pre-action fail-closed gate. Not signing machinery (covered by SCITT). Not confidence quantification (covered by W3C VC CM). Not just an Attestation Result (covered by RATS). The gate primitive itself, with the version-binding necessary to outlive ruleset evolution.

## Security Considerations

**Key compromise.** Verifier issuers MUST rotate JWKS keys on a published cadence. Receipts signed under a compromised key remain verifiable against historical key state during the rotation horizon. RPs MUST honor key revocation lists where published.

**Replay attacks.** Receipts include `iat` and `exp`. RPs MUST verify both against current time, subject to clock tolerance. Claim-binding via `v_claim.hash` provides resistance to receipt-reuse against a different claim payload.

**Confidence inflation attacks.** A verifier issuer who inflates `v_confidence` to push more receipts to `act` would diverge from their published calibration anchor. The reproducibility mandate ({{calibration-anchor-requirements}}) makes this detectable. RPs SHOULD periodically sample receipts against the issuer's published harness.

**Selective disclosure.** Hash-only claim binding ({{claim-binding}}) permits PII-sensitive verification without exposing claim text in the receipt envelope.

**Downgrade attacks.** A future relying party MUST NOT accept a v0.3-spec receipt against a v0.4-spec gate. The receipt format version is implicitly bound by `v_gate_mapping`; mismatches between expected and signed mapping IDs are malformed-receipt conditions per {{verification-protocol}}.

**Mapping document tampering.** Mapping documents MUST be hosted at URLs whose integrity is verifiable (e.g., git tag with content-addressable hash). Verifier implementations SHOULD cache mapping documents by ID and verify cached content matches the publisher's signed manifest, if published.

## IANA Considerations

This document requests the following IANA registrations:

**Media type:** `verification-receipt+jws` (to be registered in the IANA Media Types registry following the procedures of RFC 6838).

**JWT claim names:** `v_verdict`, `v_confidence`, `v_gate_threshold`, `v_adversarial_result`, `v_recommendation`, `v_gate`, `v_gate_mapping`, `v_method`, `v_calibration`, `v_sources_used`, `v_evidence`, `v_claim` (to be registered in the JSON Web Token Claims registry).

**Well-known URI:** Verification issuers using HTTPS SHOULD publish their JWKS at `/.well-known/jwks.json` per existing [@RFC7517] Section 4.7 conventions. No new well-known URI is requested.

The author requests coordination with `draft-msebenzi-environment-state` authors regarding namespace use; if both documents progress to RFC, the `verification.*` and `environment.*` namespaces should be registered jointly under a constraint-family registry to be defined.

{backmatter}

<reference anchor="ANTHROPIC-ZT">
  <front>
    <title>Zero Trust for AI Agents: A security framework for deploying autonomous AI agents in the enterprise</title>
    <author>
      <organization>Anthropic</organization>
    </author>
    <date year="2026" month="5"/>
  </front>
  <refcontent>PDF, 36 pages</refcontent>
  <target>https://www.anthropic.com</target>
</reference>

<reference anchor="AVERITEC">
  <front>
    <title>AVeriTeC: A Dataset for Real-World Claim Verification with Evidence from the Web</title>
    <author initials="M." surname="Schlichtkrull" fullname="Michael Schlichtkrull"/>
    <date year="2024"/>
  </front>
</reference>

<reference anchor="EU-AI-ACT">
  <front>
    <title>Regulation (EU) 2024/1689 of the European Parliament and of the Council on AI (AI Act)</title>
    <author>
      <organization>European Union</organization>
    </author>
    <date year="2024"/>
  </front>
  <refcontent>Article 12 (Record-keeping). High-risk AI system obligations applicable from August 2, 2026.</refcontent>
</reference>

<reference anchor="SCITT">
  <front>
    <title>An Architecture for Trustworthy and Transparent Digital Supply Chains</title>
    <author>
      <organization>IETF SCITT Working Group</organization>
    </author>
    <date year="2024"/>
  </front>
  <seriesInfo name="Internet-Draft" value="draft-ietf-scitt-architecture"/>
</reference>

<reference anchor="VAP-FW">
  <front>
    <title>Verifier Abstraction Pattern Framework</title>
    <author initials="K." surname="Kamimura" fullname="K. Kamimura"/>
    <date year="2024"/>
  </front>
  <seriesInfo name="Internet-Draft" value="draft-kamimura-vap-framework"/>
</reference>

<reference anchor="VC-CM">
  <front>
    <title>Verifiable Credentials Confidence Method</title>
    <author>
      <organization>World Wide Web Consortium</organization>
    </author>
    <date year="2024"/>
  </front>
  <refcontent>W3C Working Draft</refcontent>
</reference>

<reference anchor="VINTENT">
  <front>
    <title>Verifiable Intent Specification</title>
    <author>
      <organization>Mastercard</organization>
    </author>
    <date year="2024"/>
  </front>
  <seriesInfo name="Internet-Draft" value="draft-msebenzi-environment-state"/>
</reference>

<reference anchor="AO-RECEIPT-SPEC">
  <front>
    <title>AgentOracle Verification Receipt Format, v0.3</title>
    <author initials="J." surname="Krausz" fullname="Joseph Krausz"/>
    <date year="2026"/>
  </front>
  <target>https://github.com/TKCollective/agentoracle-receipt-spec/tree/v0.3-binary-halt</target>
</reference>

<reference anchor="AO-MAPPING-v0.3.0">
  <front>
    <title>AgentOracle Mapping Document v0.3.0-2026-05-30</title>
    <author initials="J." surname="Krausz" fullname="Joseph Krausz"/>
    <date year="2026"/>
  </front>
  <target>https://github.com/TKCollective/agentoracle-receipt-spec/blob/v0.3-binary-halt/mappings/v0.3.0-2026-05-30.md</target>
</reference>

<reference anchor="AO-EVAL">
  <front>
    <title>AgentOracle Evaluation Harness (MIT-licensed, reproduces 57.6% AVeriTeC)</title>
    <author>
      <organization>TKCollective</organization>
    </author>
    <date year="2026"/>
  </front>
  <target>https://github.com/TKCollective/agentoracle-eval-harness</target>
</reference>

<reference anchor="AO-VERIFY">
  <front>
    <title>AgentOracle Receipt Verifier (open-source TypeScript)</title>
    <author>
      <organization>TKCollective</organization>
    </author>
    <date year="2026"/>
  </front>
  <target>https://github.com/TKCollective/agentoracle-receipt-verify</target>
</reference>

<reference anchor="AO-BENCH">
  <front>
    <title>AgentOracle Cross-Operator Benchmark v0.1</title>
    <author>
      <organization>TKCollective</organization>
    </author>
    <date year="2026"/>
  </front>
  <target>https://github.com/TKCollective/agentoracle-benchmark</target>
</reference>
