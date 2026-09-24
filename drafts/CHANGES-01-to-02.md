# Change list: `draft-krausz-verification-state-01` → `-02`

One line per change, grouped under the 8 confirmed brief items. Not submitted anywhere; for your review only.

## 1. Evidence pinning (rev 8 cut, assembly notes resolved, renumbered, Abstract updated, `evidence_set` name kept)

- Added new §5.3 "Evidence Pinning" with subsections: `evidence_set` member (§5.3.1), Source Entries (§5.3.2), Evidence Root Construction (§5.3.3), What an Evidence Set Does Not Establish (§5.3.4), Independence (§5.3.5) — synthesized from rev2→rev8 amendment files, cut at rev 8's 44-vector/15-condition state.
- `evidence_set_version` fixed at `"ao-evidence-set-v1"`; member name `evidence_set` preserved unchanged as instructed.
- Set-level `retrieved_at` = bytewise-least over **all** entries (pinned and unpinned), per rev8 Finding 36; malformed condition `set_retrieved_at_not_bytewise_least`.
- `source_count` REQUIRED and > 0 (rev4 F20); `pinned_count` ≤ `source_count`; `fully_pinned` computed boolean; all three carry an explicit derivation fallback when absent (rev8 F41/F43/F44) — flagged as an unresolved REQUIRED-vs-derivable tension in the new Open Issues section rather than silently reconciled.
- `pinned` REQUIRED and strictly boolean, closing the domain before any "all entries" rule runs (rev8 F40); condition `pinned_absent_or_not_boolean`.
- `content_kind`, `resource_sha256` (condition `snippet_digest_present_for_full_resource`, ranges over pinned **and** unpinned per rev4 F19 + rev8 F42), `unpinned_reason` (2-value domain, rev3) all specified.
- `retrieved_at` per-entry canonical form: RFC 3339, UTC, `Z`, exactly 3 fractional digits, required for every entry pinned or unpinned (rev6 F32 + rev8 F37); condition `retrieved_at_not_canonical_form`.
- Possession-not-composition rule (digest at receipt, rev2/rev3) and entry-distinctness rule ranging over all entries (rev5 F24 + rev8 F35; condition `duplicate_bound_tuple`) both restated in full.
- Evidence root construction: leaf preimage `"ao-evidence-leaf-v2"` (4-member, UTF-8/hex-as-string) — **string unchanged, per standing rule**; interior node `"ao-evidence-node-v1"` (raw-octet children) — **string unchanged, per standing rule**; single-item termination; odd-node promotion (no self-pairing); 4-term canonical sort order (rev5 F24c).
- Five explicit evidence-limits bullets (not-unchanged / not-completeness / not-the-time / not-unbiased-selection / not-why-absent), rev2 + rev5 F26.
- Declared-method-only `independence` block, no computation standard mandated.
- Verification Protocol gains new step 7 (evidence-set resolution) and a new §5.4.1 "Evidence-Set Resolution Step" defining `resolved`/`unknown` tokens (rev6 F29), root recomputation with `snippet_sha256_absent_when_pinned` halt (rev8 F46), `fully_pinned: false` → `unknown`, per-item reason MUST accompany every `unknown`, including trivial unpinned entries (rev4 F17 + rev8 F45); prior steps renumbered 7→8, 8→9.
- Abstract rewritten to describe the four-state vocabulary, evidence pinning, `retrieved_at` freshness, and that digest-anchored vectors are historical/unchanged — and rewritten again this pass to remove inline `[RFC7515]`/`[ANTHROPIC-ZT]` citations (idnits: abstracts may not carry references).

## 2. Four states + capture relationship + reason codes

- New major section "Verification State Vocabulary" (§3) inserted after Terminology, before the `verification.*` Constraint Family section.
- Four states defined: `verified`, `contradicted`, `indeterminate`, `not_evaluated` — 1:1 renaming of `-01`'s `supported`/`refuted`/`unverifiable`/`unknown` (declaration order preserved).
- Capture relationship stated: a state MUST be captured together with its subject and (when required) reason code; a bare state without subject is to be treated as absent. This wording is the document's own formulation of a term used without further gloss in working-group discussion; per your review, the definition is kept and the Open Issues entry now invites comment on whether it matches that usage, rather than treating it as pending confirmation.
- `instrument_failure` reason code available under all four states (verifier's own failure, distinct from the substance of any state).
- Under `indeterminate`: `divergence` ("found unattributed divergence" — a real signal, subject unresolved) vs `absence` ("produced no signal" — the name is preserved unchanged, per your instruction). Sourced to the finding/absence split worked out with navigatorbuilds, babyblueviper1, and Marcus Still in the x402 TSC #4 thread; see Acknowledgments — Marcus Still's confirmation is recorded as received, not pending (see below).
- Payload claims: `v_verdict` keeps its wire name (see "Corrections applied after initial review" below — an initial pass renamed it to `v_state` and that rename was reverted as a breaking wire change); this revision changes `v_verdict`'s domain to the four-value vocabulary; added `v_reason_code` and `v_subject`.

## 3. Subject clause generalized

- New §3.2 "The Subject Clause": the subject requirement now attaches to **any state carrying an open attribution question**, not only `not_evaluated` as in earlier drafts — citing navigatorbuilds' Status/Check/Verdict roll-up example as the motivating case.

## 4. Melchiorre's clause + operative test

- New §3.1.1 "Enumerating the Negatives Behind an Indeterminate Count": before publishing a negative (an `absence`-side `indeterminate` count), an implementation MUST enumerate the status codes behind it.
- Sourced explicitly to Melchiorre Oliva's 16 September 2026 natural experiment (HTTP 429 across two hosts, 3,008 vs 0), credited in Acknowledgments and in the section text.

## 5. Admissibility as an explicit gate, not a state

- New §3.3 "Admissibility as a Gate, Not a State": admissibility is evaluated **before** a state is ever assigned; it is not a fifth member of the state domain. `recon_excluded`-type observations never reach state assignment.

## 6. Henri's boundary sentence

- New §9.2 "Relationship to Decision Records" carries your approved sentence **verbatim, unedited**: "A pre-action verification record, as specified here, establishes whether the factual claims an action rests on were checked, against which evidence, and with what result — which a decision record does not. See [I-D.sirkkavaara-vaara-receipt] for related work; this document makes no claim about what that specification establishes, and neither format depends normatively on the other."
- Cited informatively only, via a new `I-D.sirkkavaara-vaara-receipt` reference entry — real title confirmed against the IETF Datatracker: "The Vaara Receipt: A Recomputable Receipt Format for Decisions About Autonomous Actions," `draft-sirkkavaara-vaara-receipt-11` (2026-09-18, the current revision as of this build — the brief's `-10` had since been superseded).
- No normative dependency created either direction, as instructed. Courtesy copy to Henri is a separate action outside this file — not sent by this draft.

## 7. RFC 9942/9943, `retrieved_at` freshness axis, deprecations

- SCITT subsection (§9.1) now cites RFC 9943 (published full RFC, June 2026) in place of the individual/WG draft `draft-ietf-scitt-architecture` that `-01` cited.
- New subsection for RFC 9942 (COSE Receipts), stated as complementary and **not** a normative dependency of this document.
- Multi-Axis Freshness table's Evidence row changed from `v_evidence.valid_until` to `evidence_set.retrieved_at`, with fallback to per-source `retrieved_at` when the set-level member is absent.
- `v_evidence` and `v_sources_used` marked DEPRECATED in Payload Claims (kept for back-compat; absence is not a defect) and in IANA Considerations (registrable for `-01`-era receipts, deprecated for new issuance).
- References section: added `xi:include` entries for RFC 9942 and RFC 9943; removed the standalone `SCITT` reference entry (superseded by RFC 9943); the one remaining `[SCITT]`-style inline mention in the Differentiator Summary now points at `RFC9943`.

## 8. No AgentOracle in new prose; TK Collective LLC; historical digests unchanged

- Author organization is "TK Collective LLC." Contact `email` is now `joe@tanilo.io`; the `uri` element was removed entirely (see "Corrections applied after initial review" below — an initial pass had left `Joe@agentoracle.co` / `https://agentoracle.co`, flagged for confirmation; your review confirmed the protected-exception rule for digest-anchored vectors doesn't extend to author contact info, so it was updated).
- Searched the full document for "AgentOracle": the only two remaining instances are (a) the existing contact email/URI above, and (b) one pre-existing citation of an existing repository name (`TKCollective/agentoracle-eval-harness`) that was already in `-01` and names an existing, filed artifact rather than introducing new prose use of the brand. Both are flagged here rather than silently left in.
- All three protected cryptographic strings (`ao-evidence-leaf-v2`, `ao-evidence-node-v1`, and the existing `-01` vectors) are unchanged.
- Decision Table restated against `v_verdict`'s new four-value domain, mapped 1:1 in declaration order to `-01`'s raw-verdict values; the `v_recommendation`/`v_gate` vocabulary itself is left unrenamed this revision (flagged in Open Issues).

## Acknowledgments (item 10) — each credited by actual, sourced contribution

New unnumbered back-matter section crediting: Michael Msebenzi (evidence-pinning findings 1–46, cross-format reconstruction/verification), Pote/poteshniy of AgentTrust (3-of-5 pinned defect, `signed_only`→`bytes_attested` rename, rev7 signoff, v0.4 threat-model review — affiliation now cited to the source that resolves poteshniy as the AgentTrust operator), Pablo Play (Finding 35 alternate reading, ongoing cold-build reconstruction — tracked as an open item, no longer stated as a precondition for submission; see below), Melchiorre Oliva (429-census clause+test), navigatorbuilds (subject-clause example; 13-call-site absence-side audit), babyblueviper1 (endorsed the finding/absence split; no taxonomy conflict on their side), Marcus Still/stillmarcus24 (published `AGREE/DISAGREE/INDETERMINATE/NOT_EVALUATED` four-state taxonomy in x402 TSC #4 — confirmed 7 of 7 absence-side production call sites at commit `901ca6a`, then endorsed the four states plus reason-code mechanism on tsc#4 on 2026-09-20; recorded as received, not pending), Henri Sirkkavaara (boundary sentence).

## Item 11 — nothing overclaimed as built

- Citation resolver and MiniCheck are **not** described anywhere in this draft.
- New "Open Issues" section (§10, before Security Considerations) explicitly states: independent cold-build implementation is still pending (Pablo Play's reconstruction) and is tracked as an open issue rather than a precondition for submission; the malformed-condition enumeration has not been verified name-for-name against an independent source; the pinned/unpinned duplicate-identity pair has no conformance vector yet; the recommendation/gate vocabulary is unrenamed; the REQUIRED-vs-derivable tension on `source_count`/`pinned_count`/`fully_pinned` is named as unresolved; the "capture relationship" wording invites comment on whether it matches working-group usage; the `v_verdict`→`v_state` claim rename is deferred to a future revision (new item, see below).

## Corrections applied after initial review (2026-09-22)

Two rounds of review turned up issues in the first -02 pass; all are corrected in the package described here.

- **`v_verdict`→`v_state` wire rename reverted.** The first pass renamed the payload claim itself, which would have broken every existing `-01` verifier against new receipts. Reverted: the wire field stays `v_verdict`; only its value domain changes to the four-state vocabulary. A new Open Issue records the rename as deferred to a future revision rather than silently dropped.
- **Marcus Still acknowledgment corrected twice.** First correction: removed the word "independent" (his review was not independent of the working thread) and replaced "remained outstanding" with confirmed language once his sign-off came in. Second, final correction: "7 of 7 absence-side test cases" changed to "7 of 7 absence-side **production call sites**" — the more accurate description of what he verified. Final text: "He confirmed 7 of 7 absence-side production call sites at commit `901ca6a`, then endorsed the four states plus reason-code mechanism adopted in [The Four States] on x402 TSC issue #4 on 2026-09-20."
- **Author contact corrected.** Changed from `Joe@agentoracle.co` / `https://agentoracle.co` to `joe@tanilo.io`, with the `<uri>` element removed entirely. Organization remains "TK Collective LLC." "Los Angeles, CA" was confirmed already present in the filed `-01` draft (not invented here), so no change was needed there.
- **Capture relationship (§3.1) kept, not moved** — an intermediate suggestion to relocate the definition was reversed on final review. Open Issue 6 now reads "Comments are invited on whether it matches that usage" instead of flagging it as the author's unconfirmed interpretation.
- **Pote/AgentTrust affiliation sourced.** Added a new informative reference, `AGENTTRUST-IDENTITY`, citing `https://github.com/x402-foundation/x402/issues/2557#issuecomment-4634180929`, and cited it inline at the point Pote/poteshniy is credited.
- **Submission-gating language removed, twice.** Open Issue #1 (independent cold-build implementation of evidence pinning) no longer says the document "is not submitted while that gate is open" — it now reads that the missing cold build "remains an open issue tracked here rather than a precondition for submission." The Pablo Play acknowledgment no longer calls his reconstruction "the remaining gate before this document can be submitted" — it now "tracks as an open item." This revision is ready to file; filing timing is the author's decision, not gated on any pending review.
- **`draft-borthwick-msebenzi-environment-state-02` citation confirmed correct** against the IETF Datatracker (filed 2026-08-27) — no change was needed, the existing citation already pointed at the current revision.
- **Henri Sirkkavaara acknowledgment corrected.** Previously credited him with proposing "the boundary language, adopted verbatim in Section 9.2" — overstated, since he proposed the shape of the statement, not its exact wording. Now reads: "Henri Sirkkavaara proposed the shape of the boundary statement in Section 9.2 — that it state what a pre-action verification record establishes that a decision record does not, and stop there, without characterizing his own specification."
- **Author's address trimmed.** Removed "Los Angeles, CA" (city and region) from the postal address. Kept: Joe Krausz, TK Collective LLC, United States of America, `joe@tanilo.io`.

## Format / tooling

- Rendered via `xml2rfc` v3.34.1 (xml2rfc v3 vocabulary) to `.txt` and `.html`, after all corrections above.
- idnits 2.17.1 run locally against the final `.txt` (see `idnits-output-draft-krausz-verification-state-02.txt` in this package): **Summary: 0 errors (**), 0 flaws (~~), 0 warnings (==), 3 comments (--).** All 3 remaining items are informational comments, not errors or warnings: a code-comment heuristic suggesting `<CODE BEGINS>`/`<CODE ENDS>` markers around an illustrative snippet, and two false-positive "looks like a reference" flags on the digits "1" and "0" appearing together on line 505 (part of a version string, not a citation).
- Package: `draft-krausz-verification-state-02.xml`, `.txt`, `.html`, this change list, and the idnits output, zipped together. **Not submitted anywhere — filing remains your decision.**

## Round 3 corrections (2026-09-23) — your Part 1/Part 2 review

All ten Part 1 items applied, plus one Open Issue from Part 2. Nothing else touched.

1. **§5.4 step 7 (sourcecode block).** "Section 4.2" (twice) → "Section 5.4.1" — the evidence-set resolution step actually lives at §5.4.1, not §4.2 (Composition Rules).
2. **§11.1.** "the ordering specified in Section 3.2" → now an `<xref>` to §4.2 (Composition Rules), the section that actually states the environment.*/verification.* ordering rule.
3. **§5.4 step 8.** "subject to clock tolerance per Section 6" → "subject to clock tolerance." §6 (Binary-Halt Gate Semantics) doesn't define clock tolerance and no section does — dangling reference removed rather than pointed at nothing.
4. **§9.5 VAP Framework.** Pinned to `draft-kamimura-vap-framework-01` (21 July 2026, confirmed current via IETF Datatracker) with the correct title ("Verifiable AI Provenance Framework (VAP): An Architectural Framework for Evidentiary-Grade AI Decision Trails") and author (T. Kamimura). The prior citation had no revision pinned and carried a stale 2024 date/title. Editorial note removed since the pin is now done.
5. **§8.1.** "reproducing 57.6% on AVeriTeC" (present tense) → "reproduced 57.6% on AVeriTeC, measured 28 May 2026 on the pre-migration Sonar-backed pipeline" (past tense, dated, scoped to the pipeline that produced it — not a claim about the current post-migration pipeline).
6. **Hard-coded "Section N" audit.** Checked every instance. Two live inside a `<sourcecode>` CDATA pseudocode block (§5.4 steps 7–8, items 1 and 3 above) — XML markup can't be embedded there, so those got corrected numbers as plain text rather than `<xref>`. One in ordinary prose (§11.1, item 2 above) converted to a proper `<xref>`. No other hard-coded "Section N" prose references found outside these three.
7. **§3 intro.** "It resolves working-group discussion current as of 2026-09-22, including today's wg-identity #21 and tsc #4 threads" → "It reflects working-group discussion current as of 2026-09-22, including wg-identity #21 and tsc #4 threads." A filed draft doesn't resolve a live discussion thread, and "today's" reads wrong on any date after filing.
8. **`[AGENTTRUST-IDENTITY]` removed.** The acknowledgment now reads "Pote (poteshniy) identified the 3-of-5 pinned-evidence defect..." with no affiliation claim and no citation to the GitHub comment identifying him as AgentTrust's operator. The unused reference entry was deleted from §13. This matches the site correction already made for the same reason — Pote is a collaborator who reviews the work, not an arm's-length independent party, and the affiliation citation implied otherwise.
9. **babyblueviper1 acknowledgment.** Removed "once informed of the alternative" — the remaining text ("endorsed that split as the correct cut") states the fact without the editorializing qualifier.
10. **§9.2 heading text.** "Independent work on decision records for AI agent actions is in progress" → "Related work on decision records for AI agent actions is in progress." Matches the section's own title ("Relationship to Decision Records") and avoids an unearned "independent" adjacent to Henri's work, which the document elsewhere is careful not to characterize.

**Part 2 — one Open Issue added, nothing else.** Per your correction: no citation to `draft-clifford-testimony-record` anywhere in this filing, since you haven't engaged with Clifford yet and won't cite him before replying. The verified/attested framing is *not* credited or added — §5.4.1 and §5.3.4 already draw that line without naming it, and nothing is lost by leaving it there. If you adopt the explicit framing later, it belongs in `-03` with his name on it.

The one Open Issue actually added: **"The media-type registration request is incomplete"** — §12 names `verification-receipt+jws` as a media type to register but supplies neither the RFC 6838 registration template nor the top-level media type (`application/`) it registers under. Flagged as deferred to a future revision.

**Not touched:** the standing claim ("independent byte-identical second implementation") lives in tracker files and public-facing text, not in this XML — nothing in `-02`'s own body asserts AgentTrust's implementation is "independent" (the Acknowledgments section, corrected above, already avoids that word for Pote). Confirmed no other instance of "independent" mischaracterizing AgentTrust exists in this draft.

**Verification after this round:** rebuilt via `xml2rfc` (txt + html) with zero warnings. idnits re-run on the final text: **0 errors, 0 flaws, 0 warnings, 2 comments** (both false-positive "looks like a reference" flags on adjacent digits in a version string, not IETF-nit issues). No text touched outside the 11 items listed above.

---

## Round 4 (2026-09-23, post-round-3 full read)

Three fixes, nothing else:

1. **§5.2 `v_recommendation` enumeration** — added `un_probed_not_cleared`, in §2's order. §6.1's decision table already required this state; §5.2's field list had omitted it.
2. **§6.1 instrument_failure → error rule** — rewritten. The rule cannot be part of mapping `v0.3.0-2026-05-30` (it predates reason codes, and published mappings are immutable per §5.7). Restated as a requirement of this revision itself: an issuer reporting `instrument_failure` MUST derive `error` and halt. Applying the rule in practice requires a future mapping revision that encodes it — cross-referenced to Open Issue 4.
3. **§5.2 field list** — added `v_claim` (object, REQUIRED) to the "Binding (signed)" group, pointing to §5.5, since §5.5 already makes it mandatory.

Re-rendered with xml2rfc (text + HTML), idnits: 0 errors, 0 flaws, 0 warnings, 2 harmless comments (unchanged false-positive on the `[0, 1]` confidence range).
