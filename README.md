# Drata Claude Plugin

Drata's official plugin marketplace for Claude and other MCP-compatible agents.

## Plugins

| Plugin | What it does |
|---|---|
| [`drata-grc-skills`](plugins/drata-grc-skills) | 17 job-to-be-done GRC skills plus a built-in help index over the Drata MCP — audit readiness, controls, evidence, risk, third-party/vendor risk, workforce compliance, and stakeholder reporting |

## Install

```bash
/plugin marketplace add drata/drata-claude-plugin
/plugin install drata-grc-skills@drata
```

Then connect the Drata MCP via OAuth. See the [plugin README](plugins/drata-grc-skills/README.md)
for regional endpoints and configuration.

## What this repository contains

Prompts and a reference to Drata's public MCP endpoint — nothing else. Specifically:

- `.claude-plugin/marketplace.json` — marketplace metadata
- `plugins/*/.claude-plugin/plugin.json` — plugin metadata
- `plugins/*/skills/*/SKILL.md` — natural-language workflow instructions
- `plugins/*/commands/*.md` — slash-command entry points
- `plugins/*/shared/*.md` — shared protocols (output, source labelling, write safety)
- `plugins/*/.mcp.json` — a reference to `https://mcp.drata.com/mcp/`; **no credentials**

The Drata MCP connector and the API behind it run on Drata infrastructure and are not part of
this repository. Access is authorized per-user via OAuth at connect time and is bounded by the
intersection of granted scopes and the user's Drata role.

## Security

Report security issues to [security@drata.com](mailto:security@drata.com). Do not open a public
issue for a suspected vulnerability.

## License

[Apache-2.0](LICENSE) — Copyright 2026 Drata, Inc.
