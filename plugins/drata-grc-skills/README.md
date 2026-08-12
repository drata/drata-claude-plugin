## What's new in 3.8.12

- **Reports carry your company's identity.** The header now shows your organization's logo
  where it can be retrieved, and your company name in clean type otherwise — both are
  correct renderings, not a fallback and a failure.
- **Predictable file names.** When a report is delivered as a file, it is named for the
  skill, scope, and date (`drata-framework-report-soc-2-2026-08-04.html`), so a folder of
  exports stays sortable and self-describing.
- **Less to specify.** Framework reports select the framework or workspace automatically
  when only one is in scope, instead of asking.
- **Clearer evidence language.** Individual evidence records are now called "evidence
  items" throughout; "bucket" refers only to the four workable categories. Previously a
  library of 333 records could render as "333 buckets".
- **Command descriptions match what each skill does.** Two descriptions promised output
  their skills deliberately do not produce — a named per-person roster, and root-cause
  analysis — which could route you to the wrong skill before it even loaded.
- **Safer batch edits.** A batch write that fails partway now returns a receipt showing
  what was applied, what failed, and what was not attempted, and re-previews the remainder
  rather than retrying automatically.
- **Monitoring figures are Production-only, reliably.** The Production/Code split is now
  bound to a concrete value, so the subtraction cannot silently return the pooled figure.

# Drata GRC Assistant

**One Claude plugin. 17 job-to-be-done GRC skills + a built-in help index, over the Drata MCP. Bring your own agent — Claude, ChatGPT, Cursor, Copilot.**

GRC practitioners describe the job in plain language ("Drata, are we ready for our SOC 2 audit?"); the assistant routes to the right skill and calls the live Drata MCP. No Drata UI, no tool names to memorize.

## The skills, at a glance

| Area | Skills |
|---|---|
| **Start Here** | `drata-all-identify-gaps` · `drata-help` |
| **Compliance & Audit Readiness** | `drata-framework-report` · `drata-control-report` · `drata-control-identify-gaps` · `drata-control-resolve-gaps` ✎ · `drata-monitoring-report` · `drata-evidence-report` · `drata-evidence-identify-gaps` · `drata-evidence-resolve-gaps` ✎ |
| **Risk Management** | `drata-risk-report` · `drata-risk-identify-gaps` · `drata-risk-resolve-gaps` ✎ |
| **Third-Party & Vendor Risk** | `drata-vendor-report` · `drata-vendor-identify-gaps` ✎? · `drata-vendor-resolve-gaps` ✎ |
| **Personnel & Access Compliance** | `drata-personnel-report` |
| **Reporting & Stakeholder Comms** | `drata-executive-report` |

`✎` = writes to Drata (always diff-previewed and confirmed). `✎?` = read-only apart from one optional, explicitly-confirmed write. Every skill maps to one or more of Drata's live MCP tools.

## Install (Claude Code)

```bash
/plugin marketplace add drata/drata-grc-skills
/plugin install drata-grc-skills@drata
```

Then connect the Drata MCP via OAuth. The plugin ships **one** MCP server that defaults to the US host; pick your region with a single environment variable so the agent connects to exactly one endpoint (no phantom auth prompts for regions you aren't in):

| Region | Set `DRATA_MCP_URL` to |
|---|---|
| **US** (default) | *unset* — defaults to `https://mcp.drata.com/mcp/` |
| **EU** | `https://mcp-euc1.drata.com/mcp/` |
| **APAC** | `https://mcp-apse2.drata.com/mcp/` |

Your access is the intersection of your granted OAuth scopes and your Drata role — write skills degrade gracefully to read-only when a scope is absent.

## Design principles

- **Jobs, not tools.** Areas mirror GRC workflow stages; a practitioner never needs to know a tool name.
- **Named by the job — `report` · `identify-gaps` · `resolve-gaps`.** Every skill is `drata-<domain>-<action>`: `report` to see it, `identify-gaps` to find what needs work, `resolve-gaps` to change it.
- **One skill, exactly one output contract.** A `report` returns a snapshot; an `identify-gaps` returns a ranked worklist; a `resolve-gaps` returns a confirmed write receipt. If a job would need two "Output format" blocks, it is two skills — so a read job and a write job are never the same skill, and reads and writes keep separate safety postures.
- **Safety is an invariant.** Every write follows the [write-safety protocol](shared/write-safety.md): preview the exact diff → confirm → write → read back → verify. Deletes are two-step. Relationship fields *replace* rather than append, so the full intended set is previewed every time.
- **Defensible by default.** Claims are labeled by source; a mapped control or a passing test is not, by itself, an attestation. There are no historical snapshots in the MCP, so no skill claims a trend. See [accuracy-and-sources](shared/accuracy-and-sources.md).
- **Shared protocols resolve from the plugin root.** Skills reference `${CLAUDE_PLUGIN_ROOT}/shared/...` — never a bare relative `shared/...` — so output mode, brand kit, source labelling and write safety load correctly after a marketplace install, wherever the plugin lands on disk.
- **Output mode is a setting, not a question.** `userConfig.output_mode` (`styled` | `plain`) is set once in plugin settings and persists across sessions: `styled` renders deliverables as a designed Drata artifact, `plain` answers in chat markdown. Hosts without `AskUserQuestion` degrade gracefully to the configured default instead of blocking.
- **Frameworks are references, not skills.** SOC 2, ISO 27001, NIST, HIPAA, PCI, ISO 42001, etc. are reference packs; the job is the same, the framework is a parameter.
- **One look across all of them.** Every skill inherits the [Drata brand kit](shared/drata-brand-kit.md) — palette, Geist type, source chips (`Calculated` / `Tool Calls`), and a drop-in CSS theme — so outputs read as one Drata product.

## What changed in 3.8.7

**The header logo now falls back to the company name reliably. No naming, data, or output-contract changes beyond the header rule.**

- **Root cause fixed.** 3.8.6 told every skill to fetch `logoUrl` and inline it as base64, falling back to the name "if the fetch fails" — but the hosts this plugin most runs in (Claude's cloud / Cowork sandboxes) forbid downloading from arbitrary CDNs, and Drata's image CDN additionally refuses generic fetchers via `robots.txt`. Step 1 could never legitimately succeed there, and a model pressed to "fetch and encode" with no fetch path could emit fabricated base64 or a remote URL instead of taking the fallback — a broken image exactly where the customer's identity belongs.
- **The inline step is now gated, then verified.** The logo is attempted only when `logoUrl` is non-empty **and** the host can actually download raw image bytes; hosts that cannot fail the step immediately, by design. Where a download is possible: one fetch (no retries, no proxies, no routing around a refusal), the bytes must decode as a real image, and the base64 must come from a real encoder run over those downloaded bytes — **never typed, reconstructed, or approximated from memory.**
- **The name fallback is first-class.** `logoUrl` absent, no fetch path, fetch refused, bytes not an image, or base64 unverifiable → `<div class="custname">[company name]</div>`. A report headed by the company's name in clean type is a correct header; a broken image, an empty header, or invented image data is the only failure.
- **A safety net even on success.** The inlined `<img>` now carries `onerror` plus a hidden `.custname` div with the company name, so a data URI that still fails to decode in some viewer degrades to the name instead of a broken glyph. The hidden div stays invisible unless the image errors — the visible header remains logo *or* name, never both.
- All 18 skills, the brand kit (§3), and `output-mode.md` were updated together; everything else is identical to 3.8.6.

## What changed in 3.8.6

**Every artifact now carries the customer's own logo, resolved from Drata.**

- **`Drata_getCompany` supplies the header.** One account-scoped, read-only call per run — batched with the run's other independent reads — returns `name`, `legalName` and `logoUrl`. All 18 skills now declare it in `compatibility`.
- **Logo top-left, inlined as base64.** The logo is fetched and embedded as a `data:` URI so the artifact stays self-contained — it survives being saved, emailed, and opened offline.
- **Two outcomes, never three.** If there is no `logoUrl`, the fetch fails, or the bytes don't decode, the header falls back to the company name as text (`.custname`). **A remote `<img src="https://…">` is never emitted** — a blocked or access-controlled URL would put a broken-image icon exactly where the customer's identity belongs.
- **The logo replaces the company name.** When the logo renders, the name appears only in `alt`, where it still reaches screen readers and any export that drops images.
- **Proportions are guaranteed.** `height:32px; width:auto; max-width:200px; object-fit:contain` — height is fixed, width is free. Wordmarks, square icons and tall crests all sit on one baseline with no stretching or cropping; a logo wider than 6.25:1 is scaled down proportionally rather than distorted. Setting height and width together, `width:100%`, or a fixed pixel width is prohibited. On the dark board surface a dark-on-transparent logo is replaced by the name in white rather than shipped invisible.
- **The scope is always spelled out.** Every source line ends with the workspace's own name, or `All workspaces` for an org roll-up — never omitted, abbreviated, or reduced to an id. The two risk skills name the register alongside the workspace.
- **New theme classes `.cust` / `.custname`**, added to all 14 full themes and all 4 resolve-gaps mini themes, so batch previews and write receipts carry the same header.
- **`drata-help` frontmatter corrected.** It no longer claims to make no MCP calls — it makes exactly one, `Drata_getCompany`, for the header, and reads no compliance data.

## What changed in 3.8.5

**Personal data and customer-identifying detail removed from every skill. No naming, branding, behavior or output-contract changes.**

- **Personal data removed.** Every example person is now `John Doe` / `Jane Doe`, and every example address is an `@example.com` placeholder.
- **Third-party brand names removed from example data.** Named vendors and integrations in illustrative prompts, tables and sample rows are now placeholders (`Acme Corp`, `Northwind`, `Globex`) or generic descriptions ("the ticketing system", "the cloud provider"). Framework names, Drata tool names, API fields and enum values are untouched.
- **Measured tenant figures replaced with the lesson they taught.** Rules previously justified with production numbers now state the finding instead — "the API and the UI disagreed by a single control", "the majority of current vendors carry `impactLevel: UNSCORED`", "most failures were older than 90 days, with a long tail past a year". Every rule keeps its force; none of them reveals the size or health of a real estate.
- **Sample markup uses placeholders.** KPI tiles, chart labels, table rows and bar widths in the render contracts are now `[N]` / `[M]` / `[P]%` with round proportions, instead of counts and percentages carried over from real runs.
- **Sample scopes and labels genericized.** The sample department, register and category are now `dept-alpha`, `Register A` and `Category A`, the example workspace scope is `acme-corp`, and department-group examples use `dept-<name>` rather than a real department name.
- **Internal source reference removed** from this README's 2.0 notes.

Naming, the Drata brand kit, the palette, type, logo, voice, MCP wiring, every skill's logic, data sources and write-safety posture are all identical to 3.8.4.

## What changed in 3.8.4

**`drata-framework-report` readability and prompt-cost fixes, plus a house filename rule.**

- **The requirement grid is now labelled.** Every tile carries its requirement code in 10px Geist Mono (ready = solid Cobalt, white code; not ready = white tile, dust border, muted code), rendered in code order so a reader can look up any requirement instead of counting anonymous squares. Above 120 requirements the ready tiles drop out and only the not-ready codes render — never a fallback to blank swatches. New `.rgrid` / `.rq` classes in the skill's theme, and a brand-kit rule that any per-record grid must name the record.
- **No one-option pickers.** With exactly one in-scope framework in the workspace, the skill selects it and runs — no question, no yes/no confirmation. Same rule already applied to a single workspace and to `drata-risk-report`'s single register; the command stub and the skill's blocking gate now agree.
- **Delivered `.html` files are named after the skill's folder**, exactly: `<skill-name>-<scope>-<YYYY-MM-DD>.html` (e.g. `drata-framework-report-soc-2-2026-08-04.html`). Shortened, re-worded, or display-title filenames are out; the rule lives in `output-mode.md` and in all 18 skills' render contracts and command stubs.

## What changed in 3.8.3

**The four `resolve-gaps` skills gain a mini-branded contract for batch artifacts.**

- A batch preview or write receipt of **more than 3 rows** now renders as a mini-branded HTML artifact (title → scope line → hairline → table(s) → footer icon); the preview renders alongside the chat confirmation prompt, never instead of it. Dialogue, menus, confirmations, and small (≤3-row) previews and receipts remain plain chat.
- The mini theme embedded in each of the four skills is a verified strict subset of the shared `.drata` base theme — same selectors, same values, nothing new — so resolve-gaps artifacts cannot drift from the report look.
- The four command stubs, the brand kit's theme-copy note, `write-safety.md`, and `output-mode.md` were aligned with the new contract; every write-safety step is unchanged — only the medium of large previews and receipts changed.

## What changed in 3.8.2

**Consistency fixes across the skill families — no new skills, no logic changes.**

- The four `resolve-gaps` command stubs no longer demand a branded HTML artifact; they now match their skills' plain-chat contract (menus, previews, confirmations, receipts in chat).
- `drata-help` catalog cleanup: removed the empty `drata-instance-*` section, completed the truncated policy-coverage sentence (a direct `Drata_listPolicies` / `Drata_searchPolicies` read), corrected the skill count to 18, and fixed two routes that pointed at a nonexistent `drata-monitoring-identify-gaps` (now `drata-monitoring-report`).
- Shared-protocol contradictions resolved: on hosts without an artifact tool the fallback is a delivered `.html` file, never markdown (`output-mode.md` now agrees with the skills); the brand kit's theme-copy count and chart-library wording were corrected, and `Now · Next · Watch` is scoped to identify-gaps/worklist artifacts.
- Metadata corrections: `plugin.json` bumped to 3.8.2, and plugin/marketplace descriptions no longer claim policy or instance health & maturity domains that have no skill.

## What changed in 3.7.9

**`drata-control-report` now titles its report "Control Readiness."** Its rendered HTML previously carried "Control Automation Opportunity" — a leftover from the skill's pre-2.0 `control-automation-triage` name that matched neither the skill nor its output. The artifact title now names the job (control readiness), consistent with every other skill titling its HTML after its own name. No logic or data changes; a review confirmed the remaining skills' artifact titles already match their names.

## What changed in 3.7.8

**`drata-evidence-report` now renders card-forward.** Its output format is section cards only - stacked-bar and KPI-tile cards with one-word `Live` / `Sample` section chips - with the caption paragraphs, the routing sentence, and any Now/Next/Watch prose removed, matching the trimmed sample report. Bucket logic, data sources, and read-only posture are unchanged.

## What changed in 3.7.7

**Added `drata-evidence-report`** — a read-only evidence-freshness snapshot answering "how fresh is the data?": the share of the library that is Valid versus Needs-artifact, Expiring soon, and Expired, plus why evidence decays (renewal cadence and collection source). It completes the evidence family (`report` + `identify-gaps` + `resolve-gaps`), matching controls, risk, and vendor. The renewal buckets come from the verified `EXPIRED` / `EXPIRING_SOON` filters; everything else is derived from expanded payload fields, never from the status counts (which do not partition the library). The named items behind each bucket stay in `drata-evidence-identify-gaps`; the fixes stay in `drata-evidence-resolve-gaps`. The library is now 17 job-to-be-done skills plus the `drata-help` index.

## What changed in 3.7.6

**Renamed `drata-framework-readiness-report` → `drata-framework-report`** so it follows the same `drata-<domain>-report` shape as every other reporting skill (control, risk, vendor, monitoring, personnel, executive). Only the name changed — it still produces the same framework readiness scorecard, and all routing references (its command, the `drata-help` catalog, and cross-skill pointers from `drata-control-report` and `drata-executive-report`) were updated to match.

## What changed in 3.7.5

**Renamed the two action verbs for clarity.** `triage` and `manage` name an action but not its object, so out of context they read as jargon — the exact feedback this release answers. Both now say what they act on. `report` is unchanged (it already reads plainly).

- `triage` → **`identify-gaps`** — every `drata-<domain>-triage` skill is now `drata-<domain>-identify-gaps`: controls, evidence, risk, vendor, and the cross-domain `drata-all-identify-gaps`.
- `manage` → **`resolve-gaps`** — every `drata-<domain>-manage` skill is now `drata-<domain>-resolve-gaps`: controls, evidence, risk, vendor.

The grammar is now **`report`** to see it · **`identify-gaps`** to find what needs work · **`resolve-gaps`** ✎ to change it. Only the names changed — every skill's behavior, output contract, and write-safety posture is identical to 3.7.4. Commands, the `drata-help` catalog, and all cross-skill routing references were updated to match.

## What changed in 2.0

**Renamed**

- `drata-posture-report` → **`drata-all-triage`** — it ranks cross-domain failures and routes each one onward, which is triage, not a snapshot. "All" is breadth across controls, monitoring, risk, evidence, vendors **and personnel** (added with the rename, so the name isn't an overclaim) — not the union of the domain queues. The rendered artifact is still titled *Compliance Briefing*.

**Removed**
- `drata-instance-report` — dropped. Its KPI cards with inline bars moved to `drata-executive-report`; its control field-quality scorecard moved to `drata-control-triage`.
- `drata-framework-overlap-triage` — dropped from the package.

- `drata-questionnaire-report` — no backing in the source material. It is the one job with no underlying Drata object: the MCP exposes no questionnaire to read, answer, or submit, so the skill was pure synthesis over data other skills already return. Documented in `drata-help` under "Not a skill — just ask", pointing at the readiness, control and evidence skills for the substance.
- `drata-policy-report` — policy Q&A is now a direct `Drata_searchPolicies` read documented in `drata-help`, not a skill. It has no backing in the source material, and `searchPolicies` only reaches the *caller's own assigned policies* and requires OAuth — which serves the employee-self-service persona, not the GRC persona this plugin targets. Policy coverage moved into `drata-instance-report`.
24 skills became 21, and the naming became a rule rather than a habit: every skill is `drata-<domain>-<action>` with exactly one output contract.

**Merged**
- `drata-instance-maturity-report` + `drata-control-quality-report` → **`drata-instance-report`** — three skills were answering one question ("how well do we run Drata?") off the same tool calls; field quality is a dimension of maturity, not a separate report.
- `drata-framework-expansion-report` → a mode of **`drata-framework-readiness-report`** — "ready for our SOC 2?" and "how close are we to ISO 42001?" produce the same scorecard; in-scope versus prospective is a parameter.
- `drata-portfolio-report` → the multi-workspace mode of **`drata-executive-report`** — one briefing deliverable, with workspace scope setting the shape.

**Split**
- Evidence was one skill doing two jobs. Diagnosis moved to **`drata-evidence-triage`** (ranked worklist, read-only) and the write stayed in **`drata-evidence-manage`** — a read job and a write job cannot share an output contract.
- Vendor gained **`drata-vendor-report`** for the portfolio dashboard, leaving **`drata-vendor-triage`** as the queue-and-decision skill. Risk and vendor are now the two families with all three verbs.

**Renamed** (the output was already a ranked worklist; the name now says so)
- `drata-control-automation-triage` → **`drata-control-report`** — controls now has one triage, one report, one manage, matching risk and vendor
- `drata-personnel-report` → **`drata-personnel-report`**

**Removed**
- `drata-monitoring-manage` — deferred. The Drata MCP has no test re-run, trigger, or enable/disable, so the skill could not honor its own write contract.

**Also new**
- `drata-help` gained a **"Not a skill — just ask"** section (single-object lookups are direct MCP reads, not skills). The UI-only limits — no file upload, no policy authoring, no framework scoping, no Audit Hub, no export, no task creation, no user/role management, no historical snapshots — are stated inline in the skills they affect.
