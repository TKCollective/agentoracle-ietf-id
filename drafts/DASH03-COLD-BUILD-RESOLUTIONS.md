# -03 resolutions: babyblueviper1's evidence-set cold build

Branch `dash03-cold-build-resolutions`, off `main` at `29f7d6f`. It is a proposal for review. It has not been filed.

The source is babyblueviper1's cold implementation of `draft-krausz-verification-state-02` §5.3 and §5.4.1. He built it from the draft text alone, with no reference code: [preaction-governance-conformance@ecdbeb8](https://github.com/babyblueviper1/preaction-governance-conformance/tree/ecdbeb8/examples/evidence-set-cold), in `examples/evidence-set-cold/FINDINGS.md` and `tools/evidence_set_check.py`. Each finding was checked against the filed -02 text. None of them turned out to be a misreading. He also published an `evidence_root` (`fad3cd3f…`), and the rev 8 fixture generator reproduced it independently.

Files on this branch:

- `draft-krausz-verification-state-03.xml` is -02 with only §5.3.1, §5.3.2, one sentence of §5.3.3 and §5.4.1 changed. The `docName`/`seriesInfo` are set to -03. The date is still 2026-09-22 and needs setting at filing.
- `draft-krausz-verification-state-03.txt` and `draft-krausz-verification-state-03.html` are rendered with xml2rfc 3.x. Re-rendering -02 from its XML with the same toolchain reproduces the filed -02 .txt byte for byte.
- `dash03-cold-build-resolutions.diff` is the unified XML diff, -02 to -03.
- `DASH03-QUEUE.md` lists the -03 and v0.4 items that are queued but not applied here.

## Lettering: no existing letter moved

Every §5.4.1 paragraph from -02 keeps its letter, (a) through (g). The new rules are added after (g) as new paragraphs:

- (h): unsupported version
- (i): the definition of `resolved`
- (j): precedence
- (k): the explicit evaluation order

The only edits to existing paragraphs are made in place. The number of paragraphs does not change.

The whole of -02 was checked for lettered references to §5.4.1. There are three, and each still points at the same rule:

| Location in -02 | Reference | Still correct in -03 |
|---|---|---|
| §5.4.1(f) | "unknown resolution under (c) or (d)" | Yes. It is extended to "(c), (d), (h), or (i)", and the existing letters are unchanged. |
| §5.4.1 closing paragraph | "Steps (a) and (b) are checks on the receipt's internal consistency" | Yes. It is extended to "(a), (b), and (h)". |
| §5.4.1 closing paragraph | "Step (d) is a check against external content" | Yes, unchanged. |

§3.1's "(a) the subject … (b) a reason code" is an inline list in another section, not a §5.4.1 reference.

FINDINGS.md at `ecdbeb8` cites the following, and each still resolves to the same paragraph and wording in -03:

- (a): "the specific violated condition named in the rule it failed". This sentence is kept verbatim.
- (b): "begins 'If evidence_root is non-null'". This is kept verbatim.
- (c): "a partial set resolves unknown". This is unchanged.
- (d): "resolves unknown" and "a per-item reason for every entry in sources". Both are kept verbatim.

`tools/evidence_set_check.py` cites 5.4.1(a) and 5.4.1(b). Both are unchanged in position.

Adding the registry table changes one numbering: the existing Tables 2 and 3 become Tables 3 and 4. No prose in the draft cites a table number.

## Evaluation order and precedence

§5.4.1(k) sets the order:

1. (e) runs first. If there is no `evidence_set`, the step resolves `unknown` and evaluation stops.
2. The version-independent structural checks of (h) run next. These are `evidence_set_not_object` and `evidence_set_version_absent_or_not_string`.
3. The version check of (h) runs next. If the version is unsupported, the step resolves `unknown` and evaluation stops.
4. (a) runs, and every condition it finds is reported.
5. (b) runs, but only if (a) reported no condition.
6. (d) runs for every entry, but only if (b) reported no condition.
7. (c) and (i) then set the token, combined under (j).

§5.4.1(j) sets precedence when conditions of different kinds apply: malformed outranks `unknown`, and `unknown` outranks `resolved`. An `unknown` never masks a malformed condition that the order requires to be evaluated. This is the Phase 1 corpus rule 5 on [tsc#4](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5816886977): "a check that ran and failed outranks one that could not run."

An unsupported `evidence_set_version` stops every version-specific rule, meaning (a) through (d) and (i). After that point, only the two version-independent structural checks run.

## Normative changes: these change which receipts pass

1. **`retrieved_at` seconds = 60 is now malformed (F6).** RFC 3339 permits `:60`, and -02 permitted anything RFC 3339 does. -03 disallows it and reports `retrieved_at_not_canonical_form`. babyblueviper1's implementation allows `:60`, so this reverses his choice on this point.
2. **An unsupported `evidence_set_version` now resolves `unknown` (F7).** -02 said nothing about what an unsupported version means. -03 resolves it as `unknown` with reason `evidence_set_version_unsupported`, and version-specific rules are not evaluated. Only an absent or non-string value is malformed. His implementation chose malformed, so this reverses that choice.
3. **`pinned_count` must match exactly.** -02 said "MUST be less than or equal to `source_count`". -03 requires `pinned_count` to equal the actual number of pinned entries and reports `pinned_count_mismatch`. This adopts his implementation's tightening.
4. **The `snippet_sha256_absent_when_pinned` check is now unconditional (F3).** It moved out of (b), where it was gated on a non-null root, into the per-entry rule checked in (a). What changes in practice is narrower than it sounds. The case he found (a pinned entry with a null snippet and a null root) already halted in -02, because a null root with `pinned_count > 0` violates §5.3.1. What changes is that the snippet condition itself is now reported, and under report-all that changes the conformance-visible set of conditions.
5. **The NUL rule is new (F5).** -02 asserted that "No member may contain an octet 0x00", but never made it a check. -03 makes it an explicit per-entry rule, `member_contains_nul`, covering url, snippet_sha256, content_kind and retrieved_at on every entry, pinned or unpinned. It is checked before root construction relies on it.
6. **A digest on an unpinned entry is now malformed (F4).** A non-null `snippet_sha256` on an entry with `pinned: false` is now malformed and reports `snippet_sha256_present_when_unpinned`.

Two more outcome changes are not on the list above and needed a decision:

- **F10.** When the set-level `retrieved_at` is absent, it is now derived as the bytewise-least entry value instead of being left undefined. A receipt that some verifiers would have halted can now pass.
- **F12.** A fully pinned set that the verifier did not content-check now resolves `unknown` instead of an undefined result. This changes the step token, not whether the receipt halts, because `unknown` never halts under §5.4 step 7.

Decision, 2026-09-25: Joe Krausz approved both F10 and F12 as written above. (Note added 2026-09-27.)

## Clarifications: no change to which receipts pass

- **F1/F2.** Named conditions are now given for MUSTs that were already in -02 but had no name. The draft also gains a 30-row registry table at the end of §5.4.1.
- **Report-all (F1).** (a) no longer says "MAY name any one". A verifier MUST report every violated condition, and conformance compares the set of conditions, not their order. A type check that fails suppresses the checks that depend on that type, so the reported set is deterministic.
- **F6.** Calendar validity (a real month and day, and in-range hour and minute) is written out. RFC 3339 already excluded these values, so this is a clarification. Only the `:60` change above is normative.
- **F8.** `snippet_digest_present_for_full_resource` is renamed `resource_sha256_present_for_full_resource`. The -02 name is recorded inline.
- **F9.** `sources_not_array` is split out from `evidence_set_names_no_sources`. A non-array `sources` value already violated "sources (array)".
- **F10.** The rationale is added, "so that freshness is judged by the oldest evidence", along with the edge case where no entry has a valid value.
- **F11.** `content_matches` is added as the third per-item reason.
- **F12.** The resolved-definition (i) sets out the conjunction rule and states what an unchecked, fully pinned receipt still establishes.
- **Order and precedence.** (j) and (k) write down an order that -02 left implicit. -02's "any inconsistency is malformed" already let malformed dominate.
- **§5.3.3.** The NUL sentence now points to the §5.3.2 rule.

## Credit, per finding

Every finding below is babyblueviper1's, from the cold build at [`ecdbeb8`](https://github.com/babyblueviper1/preaction-governance-conformance/tree/ecdbeb8/examples/evidence-set-cold). Where -03 departs from the choice his implementation made, the departure is noted.

| # | Finding (babyblueviper1) | -03 resolution | Relation to his choice |
|---|---|---|---|
| F1 | Doubly malformed receipts: "MAY name any one" lets conformant verifiers disagree | Report every condition; compare the set | Adopted |
| F2 | About a dozen §5.3.1/§5.3.2 MUSTs have no named condition | Named in place and collected in a registry table | Adopted; his strings used wherever usable |
| F3 | `snippet_sha256_absent_when_pinned` sits inside (b) and never fires when the root is null | Checked per entry in (a), unconditionally | Adopted |
| F4 | A digest on an unpinned entry is unaddressed | `snippet_sha256_present_when_unpinned`, malformed | Adopted |
| F5 | The no-0x00 premise is asserted, not checked | NUL exclusion rule, `member_contains_nul` | Adopted, his name |
| F6 | Canonical form vs. a valid instant | Calendar validity required; `:60` disallowed | Calendar validity adopted; leap second reversed |
| F7 | Unrecognized `evidence_set_version` | `unknown`, `evidence_set_version_unsupported` | Reversed: his choice was malformed |
| F8 | `snippet_digest_present_for_full_resource` names the wrong member | Renamed `resource_sha256_present_for_full_resource` | Adopted (the finding is his; the new name is ours) |
| F9 | `sources` absent vs. not an array | Split into `evidence_set_names_no_sources` and `sources_not_array` | Split, where his code folds them |
| F10 | Set-level `retrieved_at` absent | Derived as the bytewise-least entry value, rationale stated | His derivation adopted as is |
| F11 | No reason token for a held, matching item | `content_matches` | Adopted, his name |
| F12 | Nothing defines `resolved` for a fully pinned set that was never content-checked, and nothing combines per-item results | (i): `resolved` only when fully pinned, the root recomputes, and every item is `content_matches` | His strict reading adopted |

The two closed Open Issues items can only be marked closed after this lands. They are item 1 (independent cold build pending) and item 2 (condition enumeration not checked name for name). That edit is queued, not applied, because this branch is §5.3/§5.4.1 only.

## Two more report-all suppression cases (babyblueviper1, 2026-09-25)

After the first round above landed on this branch, babyblueviper1 read the rendered -03 text and found the same class of nondeterminism the (a) fix was meant to close, in two places the fix's named examples (`sources` not an array; an entry not an object) didn't cover: [tsc#4, 2026-09-25](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5827285607).

1. **`pinned` not a boolean** (`pinned_absent_or_not_boolean`). `pinned` decides which per-entry branch rules apply (`snippet_sha256_absent_when_pinned` vs. `snippet_sha256_present_when_unpinned`, and the `content_kind` rules) and feeds `pinned_count_mismatch` and `fully_pinned_mismatch`. Without a general suppression rule, one conformant reader could report only `{pinned_absent_or_not_boolean}` for a malformed `pinned` value while another goes on to evaluate the branch and count rules against it and reports a larger set — both readings conformant to the -02 text, and disagreeing.
2. **An entry's `retrieved_at` not in canonical form** (`retrieved_at_not_canonical_form`). That value feeds `set_retrieved_at_not_bytewise_least` and is one of the fields `duplicate_bound_tuple` compares. Without the general rule, one reader could carry the malformed value into those comparisons and another could exclude it, again producing conformant but differing sets.

-03 closes both the same way it closed the two named cases: (a) now states the rule generally — "a condition is not evaluated if any member it takes as input failed its own type or form check; the failed member's own condition is reported instead" — rather than only the two instances -02's cold-build round named. This is a clarification, not a normative change: it doesn't change which receipts pass, only which set a report-all conformant verifier is required to produce for these two inputs.

| Case | Malformed input | Reported set under -03 |
|---|---|---|
| 1 | `pinned` present and not a boolean | `{pinned_absent_or_not_boolean}` |
| 2 | An entry's `retrieved_at` present but not canonical RFC 3339 form | `{retrieved_at_not_canonical_form}` |

Credit: babyblueviper1, both cases and the general fix.

## (k) closes a gap of its own (babyblueviper1, 2026-09-25)

The evaluation order in (k) named the version-independent structural checks of (h) as a step but didn't say what happens when one of them fails — silent on whether a failure there halts evaluation before the version check runs. babyblueviper1's suggested wording, adopted verbatim: "...the version-independent structural checks of (h); if either fails, the receipt is malformed, the step reports that condition alone, and evaluation stops; the version check of (h)...". Without it, one reader could stop at the structural failure and another could continue to the version check and beyond, producing different reported sets for the same malformed receipt. Credit: babyblueviper1.

## Security Considerations addition (babyblueviper1's F7 acceptance, 2026-09-25)

On accepting F7 (an unsupported `evidence_set_version` resolves `unknown`, reversing his cold-build implementation's `malformed` choice), babyblueviper1 noted the reversal's effect is the same as (e): an issuer who wants to avoid the malformed halt can already leave out `evidence_set` entirely, so naming an unsupported version gives it nothing that omission doesn't, and asked that this be said in Security Considerations so a reader doesn't take `evidence_set_version_unsupported` for an escape hatch rather than the same no-evidence `unknown` category. Added as a new paragraph in "Other Security Considerations". Credit: babyblueviper1.

## (a) states what whole-check suppression costs (babyblueviper1, 2026-09-25)

babyblueviper1 accepted the whole-check reading ([tsc#4, 2026-09-25](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5837288024)) and pointed out one consequence that needed saying so nobody reads it as a gap. His clause is added to (a), right after the whole-check sentence:

> Under this rule, a single malformed `retrieved_at` suppresses `set_retrieved_at_not_bytewise_least` and `duplicate_bound_tuple` for the whole set; this loses nothing, because the receipt already halts as malformed on `retrieved_at_not_canonical_form` --- the rule changes which set of conditions is reported, never the verdict.

This is a clarification. It doesn't change which receipts pass or which set is reported; it only states a consequence the whole-check sentence already has. Credit: babyblueviper1.

## Conformance vectors for the report-all and (k)-stop rules (babyblueviper1, 2026-09-25)

These are the five vectors babyblueviper1 said he'd add when he moves his cold checker to -03 ([tsc#4](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5837288024)). The expected sets follow from -03 §5.4.1(a) and (k) as written on this branch. Conformance compares the reported set, never its order.

| Vector | Input (what's malformed) | Rule | Expected reported set |
|---|---|---|---|
| case 1 | One entry with `pinned: "yes"`, a non-null `snippet_sha256`, `pinned_count: 1`, `fully_pinned: true` | (a) whole-check | `{pinned_absent_or_not_boolean}` |
| case 2 | Entry `retrieved_at` values `["2026-09-01T00:00:00.000Z", "2026-9-1"]`, set-level `retrieved_at` equal to the first | (a) whole-check | `{retrieved_at_not_canonical_form}` |
| (k) stop 1 | `evidence_set` present and not a JSON object | (k), structural check of (h) | `{evidence_set_not_object}` |
| (k) stop 2 | `evidence_set_version` absent | (k), structural check of (h) | `{evidence_set_version_absent_or_not_string}` |
| (k) stop 3 | `evidence_set_version` present and not a string | (k), structural check of (h) | `{evidence_set_version_absent_or_not_string}` |

Each one is malformed, so the gate decision is halt.

- **Case 1:** `pinned` failed its type check, so no rule that takes it as input is evaluated. That rules out the branch rules (`snippet_sha256_*`, `content_kind_*`) and `pinned_count_mismatch`. (Corrected 2026-09-28: this gloss also listed `fully_pinned_mismatch`. That check takes the declared counts as its operands, not `pinned`, so it is evaluated; in this vector the declared `pinned_count: 1` and the derived `source_count` of 1 agree with `fully_pinned: true`, so it reports nothing and the expected set is unchanged. See Roberto Locatelli's input 2 below.)
- **Case 2:** `"2026-9-1"` failed its form check. So `set_retrieved_at_not_bytewise_least` and `duplicate_bound_tuple` are not evaluated for the set, even though `"2026-9-1"` sorts before the set-level value bytewise.
- **The (k) stops:** version-specific rules never run. Stops 2 and 3 share one condition name, because -03 names absent and not-a-string as one condition.

Credit: babyblueviper1, both cases and all three stop vectors.

## Presence rules and the NUL rule under whole-check suppression (babyblueviper1, 2026-09-25 and 2026-09-26)

After moving the checker to -03 ([tsc#4, 2026-09-25](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5840546440)), babyblueviper1 named one reading -03 left implicit: does the whole-check rule in (a) suppress `snippet_sha256_present_when_unpinned` when the unpinned entry's `snippet_sha256` fails its own form check? His checker at `8870fa6` reported only `{snippet_sha256_not_lowercase_hex64}`. He asked the same question of `member_contains_nul`, which he reported alongside the member's other conditions.

Resolved on [tsc#4, 2026-09-26](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5842824265): `snippet_sha256_present_when_unpinned` is a presence check. It takes the member's presence as input, not its form, so it is evaluated whatever the form and both conditions are reported. It still takes `pinned` as input, so a malformed `pinned` value still suppresses it (case 1 above is unchanged). `member_contains_nul` is a content rule, not a type or form check: reported alongside the member's other conditions, suppressing nothing. -03 now says both in §5.3.2. His checker follows the same reading at [`edb864b`](https://github.com/babyblueviper1/preaction-governance-conformance/commit/edb864b), with these two vectors added; he reports 30 tests passing.

| Vector | Input (what's malformed) | Rule | Expected reported set |
|---|---|---|---|
| presence 1 | An unpinned entry (`pinned: false`, valid `unpinned_reason`) whose `snippet_sha256` is the non-null, non-hex string `"XYZ"` | §5.3.2 presence rule | `{snippet_sha256_not_lowercase_hex64, snippet_sha256_present_when_unpinned}` |
| presence 2 | An unpinned entry whose `url` contains U+0000, `snippet_sha256: null`, all other members well-formed | §5.3.2 NUL rule | `{member_contains_nul}` |

Both are malformed, so the gate decision is halt. In presence 1, fixing the form would not clear the presence defect, so the two conditions are independent and both are reported. In presence 2, `member_contains_nul` is the only condition because every other member is well-formed; the point of the vector is that it is not suppressed by, and does not suppress, anything.

Credit: babyblueviper1, both questions and both vectors.

## Roberto Locatelli's review inputs (tsc#4, 2026-09-28)

Roberto Locatelli ([tsc#4, 2026-09-28](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5865535036)), posting through an AI agent operating under his mandate, ran seven inputs through babyblueviper1's checker and a checker of his own written from the -03 text. He chose them in advance as places where the text seemed open, so the number of disagreements is not a rate. babyblueviper1 reproduced all seven on his checker and fixed two of its readings at [`c877213`](https://github.com/babyblueviper1/preaction-governance-conformance/commit/c877213) ([tsc#4](https://github.com/x402-foundation/tsc/issues/4#issuecomment-5867907111)).

| Vector | Input (what's malformed) | Rule | Expected result |
|---|---|---|---|
| R1 | A pinned `snippet` entry with `resource_sha256: null`, everything else well-formed; verifier holds no content | §5.3.2 `resource_sha256`: absent and null are equivalent | No condition; not malformed; step `unknown` (per-item `content_not_held`) |
| R2 | The only entry has `pinned: "yes"`; declared `source_count: 1`, `pinned_count: 1`, `fully_pinned: false` | §5.3.1 `fully_pinned` over the declared counts; §5.4.1(a) | `{pinned_absent_or_not_boolean, fully_pinned_mismatch}` |
| R2b | Two well-formed pinned entries; declared `source_count: 5`, `pinned_count: 2`, `fully_pinned: true` | §5.3.1 `fully_pinned` over the declared counts | `{source_count_mismatch, fully_pinned_mismatch}` |
| R3 | Two pinned entries with the same `url`, `retrieved_at` and `content_kind`, both `snippet_sha256: null`; `evidence_root: null`; `pinned_count: 2` | §5.3.2 entry distinctness; §5.4.1(a) suppresses only on a failed type or form check | `{snippet_sha256_absent_when_pinned, evidence_root_absent_with_pinned_items, duplicate_bound_tuple}` |

- **R1** is the one the -02/-03 text did not settle, and it changed the gate decision: one checker halted as malformed, the other resolved `unknown`, which never halts under §5.4 step 7. -03 now says in §5.3.2 that absent and `null` are equivalent for `resource_sha256`, as it already said for `content_kind`, and the `full_resource` rule reads "absent or `null`". The result is `unknown` here because the verifier holds no content, not because of `resource_sha256`: a `null` value carries no digest and never contributes to `resolved`, and it is not malformed. A non-null value that is not 64 lowercase hex characters still reports `resource_sha256_not_lowercase_hex64`.
- **R2 and R2b.** `fully_pinned` is defined over `pinned_count` and `source_count`, and the fallback in §5.3.1 substitutes derived values only when a count is declared absent. So the declared counts are the operands, and `fully_pinned` takes `pinned` as input only through an absent `pinned_count`. No text change: the text already said this.
- **R3.** `snippet_sha256` is typed "string or null", so a `null` digest on a pinned entry fails a consistency rule (`snippet_sha256_absent_when_pinned`), not a type or form check, and `duplicate_bound_tuple` is still evaluated. No text change.

Each result other than R1's is malformed, so the gate decision is halt.

Credit: Roberto Locatelli, all four vectors and the R1 finding. babyblueviper1 confirmed R2 and R3 against the text and fixed his checker.

