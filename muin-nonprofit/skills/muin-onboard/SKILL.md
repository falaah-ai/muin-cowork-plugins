---
name: muin-onboard
description: First-run setup for the Muin non-profit plugin. Use when the user says "set me up", "get started", "connect Muin", or is using the plugin for the first time. Verifies the connection, lists the Muin tools available, captures the organization's context (NP type, fiscal year, funds), and proposes a weekly cadence.
---

# Muin Onboarding (non-profit)

Get a non-profit connected and oriented in one pass. Run this once, the first
time someone uses the plugin.

## Steps (in order)

1. **Verify the connection.** Call `nonprofit_fundraising_dashboard`. A clean
   result means the connection, its permissions, and tenant routing all work.
   - On an auth/permission error, say which connection they are on before
     proposing a fix.
     **OAuth (the default):** the approval was revoked or has expired, or the
     signed-in person's Muin role does not carry the capability. They re-approve
     the connection, or an owner or admin widens their role, both from
     **Settings > Integrations > Connect to Claude**. There is no key involved.
     **API key** (Claude Code, CI, a headless box): the `MUIN_API_KEY` is
     missing, expired, or revoked, so mint a fresh key on the same page.
2. **Report what you can see.** Confirm the org is connected and summarize the
   live numbers (total raised, donor count, active campaigns) as proof.
3. **Capture org context.** Ask for and note: organization type (e.g. masjid,
   foundation, food bank), fiscal year start month, and whether they manage
   restricted/endowment funds. Use `nonprofit_get_fund_balances` to show the
   funds Muin already knows about so they don't re-type them.
4. **Check the country gate for 990.** If the org is US-based, mention that
   `990-prep` is available; if not, say the 990 readiness check won't apply.
5. **Set a cadence.** Propose a weekly `nonprofit-pulse` brief (e.g. Monday
   morning) and confirm the day that suits them.

## Gotchas

- **Whether Claude may write.** On OAuth this is decided by the signed-in
  person's own Muin role, so there is nothing to configure in the plugin; every
  other skill runs either way. On an API key it is decided by the key's scope,
  so recording gifts needs a read+write key. Don't push write access on an org
  that only wants visibility.
- **Empty org is normal.** A brand-new Muin tenant may return zero campaigns or
  funds — that's a clean state, not an error. Frame it as "nothing recorded
  yet" and offer to walk them through their first campaign or fund in Muin.
- **No secrets in chat.** On OAuth there is no secret to hand over at all: the
  browser approval *is* the connection, so never ask for one. If they connected
  with an API key instead, never ask them to paste it into the conversation
  either; it belongs in the `MUIN_API_KEY` environment variable.

## Tunable thresholds

- **Cadence:** default weekly. Offer daily for orgs in an active campaign push,
  monthly for very small orgs.

## Output template

```
✅ Connected to Muin — <org name / "your organization">
   Raised: <amount>   Donors: <n>   Active campaigns: <n>   Funds: <n>
Context captured: type=<…>, fiscal year starts <month>, restricted funds=<yes/no>
990 readiness: <available (US) | not applicable>
Proposed cadence: weekly pulse on <day>. Sound good?
```

## Worked example

> **User:** "Set me up."
>
> **Onboard:**
> ✅ Connected to Muin — Masjid Al-Noor
>    Raised: $48,200   Donors: 312   Active campaigns: 2   Funds: 5
> I can see a Zakat fund and a Building fund among your restricted funds.
> A couple of quick questions: when does your fiscal year start, and do you want
> me to be able to *record* gifts, or just keep an eye on things?
> I'd suggest a weekly pulse every Monday — want me to default to that?
