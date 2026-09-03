# Shared Protocol — Output Mode

**Branded is the default, always, in every skill. Never ask the user to choose an output mode.**

## Default: branded

Every substantive deliverable (dashboard, report, briefing, gap worklist) renders in Drata branding, sized to the ask — a one-line factual answer stays a styled inline reply per §4, never a forced artifact. (Resolve-gaps skills keep dialogue and small writes in plain chat; only their batch previews and write receipts of more than 3 rows render mini-branded HTML, per each skill's rule 1.)

- **Host has an artifact tool** (`create_artifact`, `show_widget`, or equivalent): read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` §3 and emit a self-contained HTML artifact with its `.drata` theme — customer identity (the inlined company logo from `Drata_getCompany` only where the host can truly download and verify it, else the company name as text — never a remote image URL, never fabricated base64) → Title → source line (always naming the workspace in full, or `All workspaces`) → hairline → KPI row → real `<table>` markup / charts → Now · Next · Watch (identify-gaps/worklist artifacts only — reports route to an identify-gaps skill in one line of body text instead) → footer (the Drata icon alone, bottom-left — the inline SVG from the skill's render contract, and nothing else: no wordmark, tagline, product name, `read-only` label, timestamp, or chrome text). The final content block runs straight into the footer icon — never a trailing routing recap, methodology, caps, or source-label paragraph between the last block and the footer.
- **No artifact tool** (Cursor, ChatGPT, plain chat surfaces): write the complete HTML to a `.html` file and send that file — there is no markdown fallback for a substantive deliverable. Brand-kit **§4** markdown — bracketed `[EYEBROW]`, the word `DRATA` never a glyph, status words not colors, no emoji — applies only to short inline answers and the text-only opt-out below.

This is a capability check, not a preference. There is nothing to ask, resolve, store, or remember.

## File naming — the skill's own folder name, exactly

When a deliverable is written to a file (or an artifact host asks for a slug/title id), the name is:

```
<skill-name>-<scope>-<YYYY-MM-DD>.html
```

- **`<skill-name>` is the skill's directory name, character for character** — `drata-control-report`, `drata-framework-report`, `drata-vendor-identify-gaps`. **Never a shortened, re-worded or prettified variant**: not `control-report.html`, not `drata-controls-report.html`, not `Control Readiness Report.html`, not the artifact's display title turned into a filename. If the user later greps their Downloads folder for the skill that produced a file, the filename has to match what they ran.
- **`<scope>`** is the workspace, framework, or register the run was scoped to, lowercased and hyphenated (`acme-corp`, `soc-2`). Omit it if the run had no scope; never invent one.
- **Date is the run date**, ISO, no times.
- Examples: `drata-framework-report-soc-2-2026-08-04.html` · `drata-control-report-acme-corp-2026-08-04.html` · `drata-help-2026-08-04.html`.
- One deliverable, one file. Never emit a second copy under a different name.

## The only exception: the user wants text-only

Switch to unbranded plain markdown when — and only when — one of these is true:

1. **The user asks in this session** ("text only", "plain text", "no styling", "switch Drata output to plain"). Honor it immediately for the rest of the session. If file tools exist, also write `~/.drata/output-prefs.json` → `{ "output_mode": "plain" }` so later sessions on this machine remember; if they don't exist, conversation memory is enough.
2. **The plugin setting says so.** Each Drata SKILL.md shows the substituted value of the `output_mode` setting. Exactly `plain` → text-only. Any other value — `styled`, empty, or a literal `${…}` placeholder — means branded.
3. **A saved preference says so.** If file tools are available and `~/.drata/output-prefs.json` parses with `"output_mode": "plain"`, honor it. A missing or unreadable file is not an error — branded.

To return to branded: *"switch Drata output to styled"* — honor immediately and update the prefs file if one exists.

## Failure rule

Never ask, never block, never let this step delay a deliverable. When any part of this is uncertain or errors, render branded and continue.
