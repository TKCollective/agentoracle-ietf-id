# IETF draft-krausz-verification-state-01 Backlog

**Target ship:** Week of June 15, 2026 (post-revenue work, after argentum-core PR lands).
**Source-of-truth file:** `/home/user/workspace/agentoracle-ietf-id/drafts/draft-krausz-verification-state-00.xml` — branch off `main` as `feat/v01-revisions`.

---

## Required edits for -01

### 1. §3 — Algorithm: ES256 → EdDSA reference
**Source:** Jacky Wang (agent.tech), June 12, 2026 — verified v0.3 fixtures locally.
**Issue:** §3 currently reads as if ES256 is the only/primary signing algorithm; v0.3 fixtures actually sign with EdDSA (Ed25519, kid `ao-fixture-2026-06-v03-ed25519`).
**Fix:** Acknowledge EdDSA (Ed25519) as the v0.3 reference algorithm. Keep ES256 valid for profiles that need it. Specifically: list both algorithms in §3 alg requirements; clarify that the reference fixture uses EdDSA and any conforming implementation MUST accept EdDSA.
**Acceptance:** Anyone reading §3 alongside the fixture pair sees zero apparent contradiction.

### 2. §6 — JWS serialization: compact vs JSON
**Source:** Jacky Wang (agent.tech), June 12, 2026.
**Issue:** §6 reads as if JWS compact serialization is expected; fixtures use JWS JSON serialization (flattened form).
**Fix:** Clarify that both serializations are conforming; v0.3 reference fixtures use JWS JSON because it preserves the protected/unprotected/signature object structure the verifier walks. Add a one-paragraph note in §6 explaining the choice and confirming compact remains valid for profiles that prefer it.
**Acceptance:** A reader trying to interpret the fixture pair against §6 finds explicit alignment.

### 3. [VINTENT] reference — dead Datatracker URL
**Source:** Mike (Beenz / environment-state author), June 12, 2026.
**Issue:** `[VINTENT]` reference resolves to a dead Datatracker URL (`.../html/v0.1-draft` — there is no draft by that name). `[ENV-STATE]` is correct.
**Fix:** Replace with the current working VI reference. If Mike sends a canonical reference string, use it directly. Otherwise regenerate from datatracker against the latest published VI document.
**Acceptance:** Every reference in -01 resolves to a live URL.

### 4. §1.2 — Sibling-framing language (cite back to environment-state)
**Source:** Mike (Beenz), June 12, 2026.
**Citation candidate:** *"deterministic read of the world vs. irreducibly probabilistic statement about a proposition"* — Mike's framing, with permission.
**Fix:** Add a one-paragraph note in §1.2 explicitly framing draft-krausz-verification-state-00 as the *proposition-side* sibling to environment-state's *world-state-read* family. Use Mike's distinction verbatim if he confirms.
**Acceptance:** Anyone reading §1.2 sees the two specs as complementary by design, with the environment-state cite as the family anchor.

---

## Defer to -02 (not in -01)

### 5. environment-state going host-neutral (VI / AP2 / x402 / TAP)
**Source:** Mike (Beenz), June 12, 2026 — heads-up, not an ask.
**Status:** environment-state's next revision reframes the family as referenceable by any agent-authorization mandate format (VI / AP2 / x402 / TAP), with VI kept as the named reference host.
**Action for -01:** None. Mike said "nothing for you to change on my timeline — file and revise on yours."
**Action for -02:** Update [ENV-STATE] cite framing to reference the host-neutral family pattern once Mike's revision is through review and he sends the new language.

---

## Workflow

1. Branch off `main` as `feat/v01-revisions` in `agentoracle-ietf-id` repo.
2. Edit `drafts/draft-krausz-verification-state-00.xml` → `drafts/draft-krausz-verification-state-01.xml`.
3. Local diff check: `xmllint --noout draft-krausz-verification-state-01.xml`.
4. Regenerate .txt and .md from the XML via the IETF tools script Joe used for -00.
5. PR review: get Mike + Jacky to re-read the changed sections only (don't re-litigate -00).
6. Submit to datatracker.

---

## Out-of-scope for -01

- §10 IANA registry — still deferred per the deliberate three-of-us decision (Joe / Mike / Douglas).
- New mapping documents — `v0.3.0-2026-05-30` stays as the reference mapping; any future mapping ships as its own immutable doc.
- ERC-8210 integration — lives in `docs/receipt-profiles.md` in agent.tech's repo, not in the IETF draft.

---

**Created:** June 12, 2026
**Owner:** Joe Krausz / TKCollective
**Next review:** When ready to ship -01, re-read this file first.
