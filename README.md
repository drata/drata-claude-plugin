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

### Relationship to Drata's internal marketplace

Each plugin here is an export of the version reviewed internally in `drata/ai-plugins`, at the same
`version`. The export is byte-identical **except** for `plugins/*/skills/*/evals/` — Drata's internal
eval specifications — which are deliberately excluded. They describe how Drata tests these skills in
its own CI and are of no use to an installer.

If you are syncing a new release, that exclusion is the only permitted difference. Anything else
diverging means the export is stale.

The Drata MCP connector and the API behind it run on Drata infrastructure and are not part of
this repository. Access is authorized per-user via OAuth at connect time and is bounded by the
intersection of granted scopes and the user's Drata role.

## Security

Report security issues to [security@drata.com](mailto:security@drata.com). Do not open a public
issue for a suspected vulnerability.

## License

[Apache-2.0](LICENSE) — Copyright 2026 Drata, Inc.

<!-- no-op: PR pipeline verification (FACE-150) -->
