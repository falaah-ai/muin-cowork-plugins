---
name: donation-receipt
description: Draft a warm, tax-compliant donation acknowledgment for a non-profit, and optionally record a contribution after explicit approval. Use when the user asks to thank a donor, draft a receipt or acknowledgment, or log/record a gift or contribution.
---

# Donation Receipt

Draft the warm thank-you a donor should get, and — only with explicit approval —
record a contribution in Muin. This is the one skill that can write, so the
guardrails matter.

## Two distinct actions

### 1. Draft an acknowledgment (read-only, safe)

`nonprofit_draft_donation_receipt` with a `contribution_id` returns **draft text
only** — donor name, amount, tax-deductible amount, and a warm receipt body. It
does **not** send anything and does **not** persist a receipt.

- Need the `contribution_id`? Get it from `nonprofit_list_contributions` first.
- Review/adjust the wording with the user. They send the final receipt through
  Muin's normal flow.

### 2. Record a contribution (WRITE — approval-gated)

`nonprofit_record_contribution` creates a new contribution. It is gated **twice**:

1. **You must get explicit user approval first.** Restate donor, amount, date,
   fund, and payment method, and wait for a clear "yes" before calling it.
2. **The connection must be allowed to write.** Muin refuses the call
   server-side otherwise, and what fixes it depends on the connection:
   - **OAuth (the default):** the refusal is about the signed-in person, not
     about a credential. An owner or admin adds the capability to their Muin
     role, or they re-approve the connection from
     **Settings > Integrations > Connect to Claude**. There is no key to mint.
   - **API key** (Claude Code, CI, a headless box): the key must carry the
     `write` scope. Tell the user to mint a read+write key on Muin's Connect
     page.

   Do not retry around the refusal in either case.

Required args: `donor_person_id`, `amount` (> 0), `contribution_date`
(YYYY-MM-DD). Optional: `fund_id`, `contribution_type` (default `donation`),
`payment_method` (default `card`), `currency` (default `USD`), `notes`.

## The one-warm-receipt rule (critical)

Muin issues **exactly one** warm receipt per donation through its own flow. This
plugin must **never** send a second acknowledgment email. The draft tool is for
*reviewing wording*; recording a contribution lets Muin's normal pipeline issue
the single receipt. Never promise the donor "I've emailed your receipt" — you
haven't, and you shouldn't.

## Gotchas

- **Draft ≠ send.** The draft tool has zero side effects. Don't imply an email
  went out.
- **Anonymous gifts** come back addressed to "Friend" — keep them anonymous.
- **Don't fabricate a donor or fund.** If you don't have a real `donor_person_id`
  / `fund_id`, look it up; never invent one to satisfy the call.
- **Confirm money before writing, every time.** No silent or assumed writes.

## Tunable thresholds

- **None.** This skill is about correctness and consent, not cutoffs.

## Output template (record path)

```
About to RECORD a contribution — please confirm:
  Donor: <id / name>
  Amount: <currency> <amount>
  Date: <YYYY-MM-DD>
  Fund: <fund or "unallocated">
  Method: <payment_method>
Reply "yes" to record this in Muin. (Muin will issue the single warm receipt.)
```

## Worked example

> **User:** "Log Sister Aisha's $500 gift to the Building fund from yesterday and
> thank her."
>
> **Skill:**
> About to RECORD a contribution — please confirm:
>   Donor: #A12 (Aisha R.)
>   Amount: USD 500.00
>   Date: 2026-06-23
>   Fund: Building fund
>   Method: card
> Reply "yes" to record this in Muin. Muin will issue the single warm receipt —
> I won't send a separate email. Want to see the draft acknowledgment wording
> first?
