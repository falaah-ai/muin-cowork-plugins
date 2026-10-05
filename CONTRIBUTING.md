# Contributing

**This repository is a mirror. Do not open a pull request against it.**

The canonical source is the `cowork-plugins/` directory of Falaah AI's internal
`muin.devops` repository:

```
muin.devops/cowork-plugins/
├── .claude-plugin/marketplace.json
├── CONTRIBUTING.md                        # this file
├── LICENSE
├── README.md
└── muin-nonprofit/
    ├── .claude-plugin/plugin.json
    ├── .mcp.json
    ├── README.md
    └── skills/
```

Everything published here is copied from that tree verbatim. A change made
directly in the mirror is overwritten by the next sync, so it is lost rather
than merged.

## Where a change belongs

| Change | Where it is made |
|---|---|
| A skill's instructions (`skills/*/SKILL.md`) | `muin.devops/cowork-plugins/muin-nonprofit/skills/` |
| Plugin or marketplace manifest, README, license | `muin.devops/cowork-plugins/` |
| The MCP endpoint URL, or anything about how the plugin connects | `muin.devops/cowork-plugins/muin-nonprofit/.mcp.json` |
| A tool's behavior, name, input schema, or annotations | Muin's backend: `muin.server/src/muin_server/services/mcp/tools/` |
| Tool permissions and write scoping | `muin.server/src/muin_server/services/chat/tool_permissions.py` |

The tools this plugin calls are not defined in this repository at all. A skill
here can only use a tool that Muin's MCP server already exposes, so a request to
change what a tool returns is a backend change, not a plugin change.

## How to propose something

Open an issue on this mirror, or write to <developer@falaah.ai>. Please include:

- the skill or tool involved, by name;
- what you asked Claude to do, and what it did instead;
- your plugin version (`.claude-plugin/plugin.json` → `version`).

Do not include API keys, access tokens, donor records, or any other Muin data in
an issue. If you need to show a response, redact it first.

## Reporting a security issue

Do not open a public issue. Follow the disclosure policy published at
<https://muin-api.falaah.ai/.well-known/security.txt>.

## License

Contributions accepted through the upstream repository are licensed under
Apache-2.0, the same license as this tree. See [`LICENSE`](./LICENSE).
