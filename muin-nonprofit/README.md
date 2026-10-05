# Muin - Non-Profit Back Office (Claude plugin)

Connect Claude (Cowork / Claude Code) to your non-profit's **Muin** system of
record. Claude can read your fundraising progress, fund balances, donor
stewardship signals, grant pipeline, and IRS Form 990 readiness; draft warm
donation receipts; and record contributions - always with you approving any
money or donor write.

> **Claude conducts the work. Muin keeps the authoritative record.**
> Claude for Small Business orchestrates the SaaS you already pay for, but it
> has no non-profit back office - no fund accounting, no 990, no donor
> stewardship, no restricted-fund compliance. This plugin fills that gap by
> giving Claude live, read-first access to Muin.

---

## What's inside

Nine skills, all backed by Muin's `nonprofit_*` MCP tools:

| Skill | What it does | Muin tool(s) |
|---|---|---|
| `muin-router` | Plain-English front door. Recommends one next action, says why in one sentence, confirms before acting, and names anything it can't reach. | (routes to the others) |
| `muin-onboard` | "Set me up." Tests the connection, lists available tools, captures your org context (NP type, fiscal year, funds), and sets a weekly cadence. | `nonprofit_fundraising_dashboard` |
| `nonprofit-pulse` | A weekly operating brief: cash & fund position, donor signals, grant pipeline, campaign progress. | dashboard · fund balances · donor summary · grants |
| `campaign-thermometer` | Per-campaign goal-vs-raised brief + a nudge plan. | `nonprofit_fundraising_dashboard` |
| `fund-compliance` | Restricted-fund balances vs. spend policy; flags overspend and over-policy spending. | `nonprofit_get_fund_balances` |
| `donor-lapse-watch` | Surfaces at-risk / lapsing donors and drafts re-engagement outreach. | `nonprofit_donor_summary` |
| `grant-match` | Grant pipeline status and age triage. | `nonprofit_list_grants` |
| `990-prep` | IRS Form 990 / 990-PF readiness snapshot and accountant hand-off (US orgs). | `nonprofit_990_summary` |
| `donation-receipt` | Drafts a warm, tax-compliant acknowledgment; can record a contribution **after you approve**. | `nonprofit_draft_donation_receipt` · `nonprofit_record_contribution` |

### Tool annotations

Each of the nine tools above is declared to Claude with MCP `annotations` -
a human-readable `title`, a `readOnlyHint`, and a `destructiveHint` - so Claude
can tell you what it is about to do before it does it.

| Tool | Title | `readOnlyHint` | `destructiveHint` |
|---|---|---|---|
| `nonprofit_990_summary` | Get Form 990 Summary | true | false |
| `nonprofit_donor_summary` | Get Donor Summary | true | false |
| `nonprofit_draft_donation_receipt` | Draft Donation Receipt | true | false |
| `nonprofit_fundraising_dashboard` | Get Fundraising Dashboard | true | false |
| `nonprofit_get_fund_balances` | Get Fund Balances | true | false |
| `nonprofit_list_contributions` | List Contributions | true | false |
| `nonprofit_list_grants` | List Grants | true | false |
| **`nonprofit_record_contribution`** | Record Contribution | **false** | false |
| `nonprofit_upcoming_events` | List Upcoming Events | true | false |

These nine are the non-profit slice. The same Muin MCP server also exposes
document, finance, HR, vendor and workflow tools; the full 27-tool inventory
with its annotations is in the [marketplace README](../README.md#tool-inventory).

### The approval model (read this)

- **Reads are free.** Every skill above reads from Muin without changing
  anything by default.
- **Writes are gated - twice.** Recording a contribution
  (`nonprofit_record_contribution`) requires (1) your explicit approval in
  Claude, and (2) a credential that carries the `write` scope. A read-only
  credential is rejected by Muin's server before the write runs.
- **Receipts never auto-send.** `donation-receipt` returns *draft* text only.
  Muin issues exactly one warm receipt per donation through its normal flow -
  this plugin never sends a second email.

---

## Setup

### 1. Install the plugin

```bash
# From the Muin marketplace (once published):
/plugin marketplace add falaah-ai/muin-cowork-plugins
/plugin install muin-nonprofit@muin-cowork-plugins

# Or test locally from a checkout of this repo:
/plugin marketplace add ./muin.devops/cowork-plugins
/plugin install muin-nonprofit@muin-cowork-plugins
```

### 2. Connect (OAuth - the default)

Nothing to mint and nothing to paste. The plugin ships pointed at Muin's MCP
endpoint with **no credential in the file**:

```json
{
  "mcpServers": {
    "muin": {
      "type": "http",
      "url": "https://muin-api.falaah.ai/api/v1/mcp"
    }
  }
}
```

That is `.mcp.json` in full - there is deliberately no `headers` block.

The first time Claude uses a Muin tool it gets a `401`, discovers Muin's
authorization server from
`https://muin-api.falaah.ai/.well-known/oauth-protected-resource`
(RFC 9728) and
`https://muin-api.falaah.ai/.well-known/oauth-authorization-server`
(RFC 8414), registers itself, and opens Muin's sign-in page in your browser.
You approve the connection there. The connection is then **yours**: it carries
your own Muin permissions, and you can revoke it without rotating anything.

PKCE is required and only `S256` is accepted; redirect URIs are matched
exactly, never by prefix.

Every Muin workspace is served from this one endpoint: signing in is what
selects your organization, so there is nothing to configure. The endpoint is
also shown in Muin at **Settings → Integrations → Connect to Claude**.

### 3. Claude Code only: connect with an API key

Use this **only** where there is no browser to sign in with: Claude Code, CI, a
headless box. In Muin go to **Settings → Integrations → Connect to Claude** and
generate a connection key. Minting one requires fresh multi-factor
authentication.

- Choose **read-only** for a look-but-don't-touch connection (every skill except
  recording a contribution).
- Choose **read + write** only if you want Claude to record contributions
  (still approval-gated in chat).

The secret is shown **once** - copy it immediately. You can revoke or rotate it
from the **API Keys** page at any time; revoking cuts Claude off at once.

Then add an `Authorization` header to `.mcp.json` and keep the key itself in an
environment variable:

```json
{
  "mcpServers": {
    "muin": {
      "type": "http",
      "url": "https://muin-api.falaah.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer ${MUIN_API_KEY}" }
    }
  }
}
```

```bash
export MUIN_API_KEY="<paste the key Muin showed you once>"
```

Never paste a key into a file you commit or a chat you share.

### 4. Run the onboarding skill

In Claude: invoke **`muin-onboard`** ("set me up"). It verifies the connection,
lists the Muin tools it can see, captures your org context, and proposes a
weekly cadence.

---

## How it connects

```
Claude (Cowork / Code)
   │  MCP (JSON-RPC over HTTP)
   │  Authorization: Bearer <OAuth token, or your MCP API key>
   ▼
https://muin-api.falaah.ai/api/v1/mcp
   │
   ▼
Muin backend  ──►  your organization's data
   (organization-scoped, rate-limited, audited)
```

The bearer is read from the `Authorization` header and nowhere else - Muin's
protected-resource document advertises `bearer_methods_supported: ["header"]`,
so a token is never accepted in a query string or a request body.

Every call is scoped to **your** organization (no cross-organization reads),
rate-limited, and written to Muin's audit log.

## Privacy & data egress

The `nonprofit_*` tools return operational fields (amounts, fund names, donor
totals, readiness flags). They do **not** return payment card data or
government identifiers. Donor records are returned by internal ID and name only.
All access is logged in Muin.

Privacy policy: <https://falaah.ai/legal/privacy/>

## Changelog

- **0.2.0** - OAuth is the default connect path; `.mcp.json` ships with no
  `Authorization` header and the API-key form is documented as the Claude Code
  path. The published MCP endpoint is corrected to
  `https://muin-api.falaah.ai/api/v1/mcp`. Tool annotations published.
- **0.1.0** - First internal build: nine skills over the `nonprofit_*` tools.

## License

Apache-2.0. See `LICENSE` at the marketplace root.
