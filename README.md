# Muin Cowork Plugins

A Claude Code / Cowork **plugin marketplace** that connects Claude to
[Muin](https://falaah.ai) - Falaah's connected operations platform for
mission-driven organizations.

> **Claude conducts the tools you already use. Muin is the system of record
> built for your mission.** Claude for Small Business orchestrates generic SaaS
> but has no non-profit back office. These plugins give Claude live, read-first,
> approval-gated access to Muin's non-profit system of record.

## Plugins

| Plugin | What it does |
|---|---|
| [`muin-nonprofit`](./muin-nonprofit) | Run a non-profit's back office from Claude: fundraising, fund balances, donor stewardship, grants, IRS Form 990 readiness, and approval-gated donation recording. |

More Muin plugins (finance, HR, vendors, contracts) will land here as additional
subdirectories under this same marketplace.

## Install

```bash
# From a local checkout (internal testing):
/plugin marketplace add ./muin.devops/cowork-plugins
/plugin install muin-nonprofit@muin-cowork-plugins

# From the published GitHub marketplace (once public):
/plugin marketplace add falaah-ai/muin-cowork-plugins
/plugin install muin-nonprofit@muin-cowork-plugins
```

## Connecting to Muin

There are two ways to authenticate, and the plugin ships configured for the
first one.

### OAuth (default, and the only path that needs no secret)

`muin-nonprofit/.mcp.json` ships exactly this, with no `Authorization` header:

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

Claude reads the MCP endpoint, gets a `401`, and follows the RFC 9728
protected-resource document at
`https://muin-api.falaah.ai/.well-known/oauth-protected-resource` to the
authorization server described at
`https://muin-api.falaah.ai/.well-known/oauth-authorization-server`
(RFC 8414). It then registers itself, opens Muin's sign-in page, and the
person approves the connection in their own browser. Nothing secret is ever
written into a config file, and the connection carries that person's own
permissions.

Properties a reviewer usually asks about, as the authorization server
advertises them:

| Property | Value |
|---|---|
| Token endpoint | `https://muin-api.falaah.ai/api/v1/oauth/token` |
| Dynamic client registration | open, per-IP rate limited (`/api/v1/oauth/register`) |
| PKCE | `S256` only; `plain` is refused |
| Redirect URI matching | exact string match, never a prefix |
| Bearer presentation | `Authorization` header only (`bearer_methods_supported: ["header"]`) |
| Revocation | `https://muin-api.falaah.ai/api/v1/oauth/revoke` |

Every Muin workspace is served from this one endpoint: signing in is what
selects your organization. The Connect page in Muin
(**Settings → Integrations → Connect to Claude**) shows the same endpoint and a
ready-to-copy `.mcp.json`.

### API key (Claude Code and other terminal use)

Where there is no browser to sign in with, mint an MCP-scoped Muin API key and
add an `Authorization` header. That path, the read-only / read+write scope
choice, and the approval model are documented in
[`muin-nonprofit/README.md`](./muin-nonprofit/README.md#3-claude-code-only-connect-with-an-api-key).

## Tool inventory

The Muin MCP server exposes **27 tools: 25 read-only and 2 writes.** Every tool
carries MCP `annotations` (`title`, `readOnlyHint`, `destructiveHint`) in
`tools/list`, so Claude can tell the person what it is about to do before it
does it.

| Tool | Title | `readOnlyHint` | `destructiveHint` |
|---|---|---|---|
| `documents_get` | Get Document | true | false |
| `documents_list_recent` | List Recent Documents | true | false |
| `documents_search` | Search Documents | true | false |
| `finance_get_invoice` | Get Invoice | true | false |
| `finance_get_summary` | Get Financial Summary | true | false |
| `finance_list_invoices` | List Invoices | true | false |
| `grounding_document_search` | Search Published Documents | true | false |
| `grounding_org_profile` | Get Organization Profile | true | false |
| `grounding_upcoming_events` | List Upcoming Public Events | true | false |
| `hr_compliance_score` | Get HR Compliance Score | true | false |
| `hr_expiring_certs` | List Expiring Certifications | true | false |
| `hr_policy_gaps` | List HR Policy Gaps | true | false |
| `nonprofit_990_summary` | Get Form 990 Summary | true | false |
| `nonprofit_donor_summary` | Get Donor Summary | true | false |
| `nonprofit_draft_donation_receipt` | Draft Donation Receipt | true | false |
| `nonprofit_fundraising_dashboard` | Get Fundraising Dashboard | true | false |
| `nonprofit_get_fund_balances` | Get Fund Balances | true | false |
| `nonprofit_list_contributions` | List Contributions | true | false |
| `nonprofit_list_grants` | List Grants | true | false |
| **`nonprofit_record_contribution`** | Record Contribution | **false** | false |
| `nonprofit_upcoming_events` | List Upcoming Events | true | false |
| `vendors_get_risk` | Get Vendor Risk Profile | true | false |
| `vendors_list` | List Vendors | true | false |
| `workflows_get_status` | Get Workflow Status | true | false |
| `workflows_list_templates` | List Workflow Templates | true | false |
| `workflows_pending_tasks` | List Pending Workflow Tasks | true | false |
| **`workflows_transition`** | Advance Workflow Instance | **false** | **true** |

The two writes are gated twice: the credential must carry the `write` scope, and
the person must approve the call in Claude. A read-only credential is refused by
Muin's server before the tool body runs.

`readOnlyHint` is declared per tool rather than derived from the scope check, so
the two can be asserted to agree. A test fails the build when they disagree
(`muin.server/tests/unit/services/mcp/test_744_tool_annotations.py`).

## Structure

```
cowork-plugins/
├── .claude-plugin/
│   └── marketplace.json        # lists every Muin plugin
├── CONTRIBUTING.md             # this tree is a mirror; where the source lives
├── LICENSE                     # Apache-2.0
├── README.md                   # this file
└── muin-nonprofit/
    ├── .claude-plugin/plugin.json
    ├── .mcp.json               # https://muin-api.falaah.ai/api/v1/mcp
    ├── README.md
    └── skills/                 # 9 skills (router, onboard, 7 NP skills)
```

## Privacy

The tools return operational fields (amounts, fund names, donor totals,
readiness flags). They do not return payment card data or government
identifiers. Every call is scoped to one organization, rate-limited, and written
to Muin's audit log.

Privacy policy: <https://falaah.ai/legal/privacy/>

## Contributing

This tree is a mirror. See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for where the
source lives and how a change reaches this repo.

## Changelog

- **0.2.0** - OAuth is the default connect path and `.mcp.json` ships with no
  `Authorization` header; the API-key form is documented as the Claude Code
  path. Published MCP endpoint corrected to
  `https://muin-api.falaah.ai/api/v1/mcp`; the previously documented host did
  not resolve and the `/mcp` path was missing the `/api/v1` mount prefix. Tool
  inventory published with its MCP annotations.
- **0.1.0** - First internal build: marketplace, `muin-nonprofit` plugin, nine
  skills.

## License

Apache-2.0. See [`LICENSE`](./LICENSE).
