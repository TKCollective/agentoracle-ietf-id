# -03 queue: items noted but not applied on this branch

## For -03 (spec text)

1. **Correct the Melchiorre Oliva natural experiment (§3.1.1 and Acknowledgments).**
   - What -02 says: §3.1.1 describes "HTTP 429 responses across two hosts, one returning over three thousand and the other returning zero". The Acknowledgments credit Melchiorre Oliva with "a live measurement across two hosts returning 3,008 negative results on one and zero on the other".
   - Why that is wrong: it was one host, `api.osf-master-server.com` at `/x402/sample/:record_id`, seen by two requesters.
     - The 3,008 HTTP 429s over nineteen days are nohumans.directory's requester-side count. They come from nohumans.directory's own high probe rate and are credited to nohumans.directory, not to Mel.
     - Mel's once-a-day census of the same route recorded zero 429s on that host.
     - The filed text reverses which count is the fact about the requester.
   - Source: [meloliva14 on wg-identity#21](https://github.com/x402-foundation/wg-identity/issues/21#issuecomment-5673430531).
   - What -03 should do: rewrite both passages to say one host and two requesters, and credit the 3,008 to nohumans.directory. Mel's credit should stay limited to his census and the operative test.
2. **Close Open Issues items 1 and 2.** Do this once the cold-build resolutions are accepted: an independent cold build has now happened, and the condition enumeration has been checked name for name.
3. **stillmarcus24's three-status point.** tanilo-receipt-verify emits valid/invalid/indeterminate, not the four-state vocabulary. This is a verifier gap and stays open.

## For receipt-spec v0.4, session history (from microsoft/autogen#7353)

These come from babyblueviper1's session-chain vectors at `bddcafc` and our reply:

1. **The sequence number must be inside the hashed content.** It has to be bound by the head hash rather than carried beside it, or entries can be reordered or renumbered without changing any head.
2. **A PASS needs a lower scope bound as well as an upper one.** The first `prev_head_hash` in a presented range is taken as given. A verifier must state the lowest entry its check actually covers, and must not present a range check as covering history before it.
3. The earlier caveats carry over:
   - The external-channel guarantee is "detectable by anyone who retained a later head", not "detectable" outright.
   - The chain detects equivocation. It does not prevent it.

## Separate (tanilo.io, not the draft)

- The incident note for 2026-09-24 calls `instrument_failure` a "verdict value". It is a reason code (`v_reason_code`, -02 §3.1). The same fix is needed in the wording of trust page §04.
