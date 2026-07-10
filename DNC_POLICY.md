# Do-Not-Call (DNC) Compliance Policy — Charging Systems Inc.

**Effective date:** Pending counsel review (drafted 2026-06-09)
**Status:** In development, pending approval. **Operational policy — NOT legal advice. Review with compliance counsel before the first outbound dial.**
**Company:** Charging Systems Inc.
**Address:** 4000 Pleasant Grove, Norman, OK 73072, US
**Contact:** admin@chargingsystemsinc.com
**Scope:** B2B outbound calling by Charging Systems Inc. sales reps to business
prospects in the United States. Does not cover SMS (see [`SMS_CONSENT.md`](SMS_CONSENT.md))
or buyer-message responses on marketplaces.

---

## 1. Purpose

This policy describes how Charging Systems Inc. honors federal and state
Do-Not-Call (DNC) obligations when its sales reps place outbound business-to-business
calls to prospects. It is the operational compliance gate for the
**200/day outbound program** built on the
[`account-db`](https://github.com/arcreactorco/account-db) schema, the
[`shared-contracts/dnc-scrub`](https://github.com/arcreactorco/shared-contracts) pipeline,
and the operator dashboards downstream.

The schema implementation is in
[`account-db/migrations/001_dnc_compliance.sql`](https://github.com/arcreactorco/account-db/blob/main/migrations/001_dnc_compliance.sql).
This policy describes the *rules around* that schema; the schema is *how* we
record what the rules require.

## 2. Hard rules (binding on every system, rep, and operator)

These are non-negotiable. Bypassing any of them is a violation of this policy
and may expose Charging Systems Inc. to TCPA, state telemarketing-law, or FCC
liability.

1. **No call without a pre-call scrub.** Every candidate phone number must be
   filtered against both:
   - the **federal DNC Registry** (last refresh ≤ 31 days old), AND
   - the **internal DNC list** (`account.accounts.dnc = TRUE`, plus any
     contact-level DNC additions once that table exists)
   before it reaches a rep's dialer queue. A call to an un-scrubbed number is
   prohibited. The scrub run timestamp + match counts are appended to
   `compliance.dnc_events` with `event_type = 'scrub_match'` /
   `'scrub_clear'`, per scrub run.

2. **Honor on-call DNC requests immediately.** If a target says "do not call
   me again" — in any form, by any channel — during or after a call, the rep
   ends the call promptly and:
   - flips `account.accounts.dnc = TRUE` on that account,
   - records `dnc_reason` (the verbatim request, paraphrased OK),
   - sets `dnc_added_at = now()`,
   - emits a `compliance.dnc_events` row with `event_type='added'`,
     `source='consumer_request'`, `reason` set, and the rep's user id in
     `actor_user_id`.

   The flip must land **before the rep's next dial.** Operator tooling enforces
   this; reps cannot proceed past the post-call form without flipping the flag
   when the request was made.

3. **Once DNC, always DNC unless documented removal.** A `dnc=TRUE` account
   may only flip back to `dnc=FALSE` via an explicit operator action with:
   - a documented reason (e.g., the target re-engaged via inbound channel
     and asked to be re-added),
   - a `compliance.dnc_events` row with `event_type='removed'`,
     `source='consumer_request'` / `'internal'` (case-dependent), and
   - the operator's user id.

   Default scrub runs do NOT remove DNC state — only an operator does.

4. **No call to a federally-listed number, ever.** Federal-registry matches
   override all internal state. If the federal scrub returns a match for a
   phone, that phone is unreachable for outbound until the federal registry
   itself removes it (which Charging Systems Inc. does not control).
   Re-scrub on the standard cadence (≤ 31 days).

5. **No call to a state-listed number from that state.** State-level DNC
   lists (e.g., Indiana, Wyoming, where applicable) are honored the same
   way. If a phone is on the state list AND the account address is in that
   state, the phone is unreachable for outbound.

6. **Calling hours: 8 AM – 9 PM in the target's local time.** The federal
   floor. Some states have tighter hours; if a state is tighter, the state
   wins. Reps' dialer tooling enforces by ZIP-to-timezone lookup.

7. **B2B exemption is not blanket.** The TCPA contains a B2B exemption from
   many DNC obligations, but it is narrow and fact-specific (business line vs.
   personal cell, established business relationship, etc.). Charging Systems
   Inc. **does not assume B2B exemption** for any phone unless a compliance
   review explicitly clears that phone. Default posture: treat every number
   as if the strictest applicable rules apply.

8. **Identification on every call.** Reps identify themselves by name and
   identify the call as being from Charging Systems Inc. before any sales
   content. No anonymous, ID-spoofed, or pretexted calls.

## 3. What we record (the audit trail)

For every DNC state change OR scrub match, a row lands in
`compliance.dnc_events`. The row is append-only — the table is the
canonical log of the company's DNC posture and history. Any system
or operator action that touches DNC state writes the audit row before
the underlying state change is considered final.

Minimum fields recorded:

- `occurred_at` (timestamptz) — when the event happened
- `event_type` — one of `added`, `removed`, `scrub_match`, `scrub_clear`,
  `manual_review`, `internal_request`
- `account_id` and/or `phone` — the subject
- `source` — federal registry, state registry, internal, consumer request,
  court order, inferred
- `reason` — free-form, required for `added` and `removed`
- `actor_user_id` or `actor_label` — who or what made the change
- `safe_harbor_doc` — pointer to evidence (URL or doc id) when available

A compliance reviewer or counsel can reconstruct the full DNC history of
any account or phone number from this table alone.

## 4. How an account gets onto the internal DNC list

Five paths, each with its own `event_type` and `source` per §3:

| Path | `event_type` | `source` | Required fields |
|---|---|---|---|
| Target asked us to stop calling (live call, email, letter) | `added` | `consumer_request` | `reason`, `actor_user_id` |
| Federal DNC Registry matched their phone (scrub run) | `scrub_match` | `federal_registry` | `phone`, `actor_label` (e.g., `scrub_2026-06-15`) |
| State DNC Registry matched | `scrub_match` | `state_registry` | `phone`, `actor_label` |
| Court order / settlement | `added` | `court_order` | `reason`, `safe_harbor_doc` |
| Operator judgment (internal review) | `added` | `internal` | `reason`, `actor_user_id` |

In all five cases, the `account.accounts.dnc` flag flips to TRUE alongside
the audit row when the event applies to an account (federal-registry
phone-only matches may not always tie to an account).

## 5. How an account gets removed from the internal DNC list

Three paths:

1. **Explicit consumer re-opt-in.** The target contacts us via an inbound
   channel and asks to be re-added. Operator records the request,
   verifies identity, emits `event_type='removed'`, `source='consumer_request'`,
   `reason` includes verification proof.
2. **Federal/state registry no longer matches AND there was no other reason
   for the internal DNC.** A scrub-only DNC can be cleared via
   `event_type='scrub_clear'` when the phone falls off the federal/state list.
   This does NOT flip an account that was DNC for other reasons (the audit
   trail makes this explicit).
3. **Erroneously added.** If a DNC was added by operator error, the
   correcting event is `event_type='removed'`, `source='internal'`, with the
   reason explicitly stating "added in error on YYYY-MM-DD; correcting."
   The original `added` event remains in the log (append-only); both events
   together describe the history.

## 6. Scrub cadence + safe-harbor posture

- **Federal DNC Registry scrub:** every 31 days or before the start of any
  new outbound list, whichever comes first. Per FCC rules a 31-day window
  is the maximum-stale tolerance for safe-harbor.
- **State scrubs:** monthly or per state-specific cadence (which we
  document per-state when adding states to the program).
- **Internal DNC scrub:** real-time. Every candidate is checked against
  `account.accounts.dnc` at queue-build time (in code, not human review).
- **Evidence retention:** scrub run results (matched-count, sampled
  phones with last 4 digits only, run timestamps) are retained for **five
  years** in `compliance.dnc_events`. Raw scrub-result files are stored
  outside git per CSI's PII policy.

## 7. Operator and rep responsibilities

- **Operators** (Dillon, Pete, future admins):
  - Approve scrub cadence and any deviations
  - Review `manual_review` rows quarterly
  - Sign off on any `removed` events from `internal` source (a rep cannot
    self-remove DNC; that requires operator approval)
  - Maintain this document; flag drift between policy and actual practice
- **Reps:**
  - Follow the dialer's prompts; do not bypass the post-call DNC capture
    form
  - Notify operators immediately if they believe a call was made to a DNC
    number (mistake or system bug)
  - Do not store phone numbers outside the system (no personal phone
    spreadsheets, no personal CRMs)

## 8. Incident response

If an outbound call is made to a number on any DNC list (federal, state,
internal):

1. Stop the rep's dialer immediately
2. Operator emits a `compliance.dnc_events` row with
   `event_type='manual_review'`, `source='internal'`,
   `reason` describing the incident
3. Operator reviews the scrub run that should have caught it — was it
   stale? Was internal data drift the cause? Was it operator error?
4. Document the root cause and the corrective action in the same row's
   `safe_harbor_doc` field (point at an internal incident doc)
5. If counsel review is warranted (e.g., target alleges a violation),
   escalate before any further outbound calling

A single accidental call is recoverable. A pattern is not. The audit log
is the basis for proving the difference.

## 9. What this policy does NOT cover

- **Inbound calls.** A target who calls us is not "outbound dialing."
  Standard call-handling applies.
- **Marketplace messages.** Buyer messages on eBay/Amazon are governed by
  marketplace policies + [`SMS_CONSENT.md`](SMS_CONSENT.md) where SMS
  alerts are involved.
- **Email outreach.** CAN-SPAM, not TCPA. Separate policy when we have one.
- **SMS.** [`SMS_CONSENT.md`](SMS_CONSENT.md) covers our existing internal
  operations-alert SMS. Customer-facing SMS outreach is not approved by this
  policy.
- **International calling.** US scope only today.
- **Voicemail / ringless voicemail.** Not approved by this policy at all.

## 10. Sources + references

- Federal DNC Registry — https://www.donotcall.gov
- FCC rules (47 CFR § 64.1200) — https://www.ecfr.gov/current/title-47/chapter-I/subchapter-B/part-64/subpart-L
- FTC Telemarketing Sales Rule — https://www.ftc.gov/legal-library/browse/rules/telemarketing-sales-rule
- State DNC lists — varies; consult per state before adding state to outbound program
- Charging Systems Inc. internal architecture —
  [`account-db/migrations/001_dnc_compliance.sql`](https://github.com/arcreactorco/account-db/blob/main/migrations/001_dnc_compliance.sql) (schema)
- Charging Systems Inc. compliance idea record —
  [`company-docs/ideas/2026-06-02-dnc-registry-scrub-and-internal-dnc-policy.md`](https://github.com/arcreactorco/company-docs/blob/main/ideas/2026-06-02-dnc-registry-scrub-and-internal-dnc-policy.md)

## 11. Open items before this policy goes Approved

These must be answered before the first outbound dial:

- [ ] Counsel review of this document end-to-end (the policy is
      operational, not legal advice; counsel needs to confirm the rules
      below match the strictest applicable obligations)
- [ ] Decision on federal-registry access path (DIY SAN account vs.
      managed-service vendor) — see the ideas doc Path A/B/C question
- [ ] Decision on which states the outbound program initially targets
      (state DNC rules vary; pick states the scrub vendor / SAN account
      covers)
- [ ] Operator dashboard built that captures the post-call DNC form
- [ ] Scrub run automation tested end-to-end against
      `compliance.dnc_events`
- [ ] Rep training on §2 + §7 obligations recorded as completed
