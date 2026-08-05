---
name: drata-evidence-report
description: >
  A read-only snapshot of how fresh the evidence library is for a workspace: the share that is Valid
  versus Needs-artifact, Expiring soon, and Expired, plus why evidence goes stale - renewal cadence
  (how much never prompts again) and collection source (automated versus manual). Counts and charts,
  not a worklist; the renewal buckets come from the verified EXPIRED / EXPIRING_SOON filters,
  everything else from expanded payload fields. Use for 'how fresh is our evidence', 'what share is
  valid or expired', evidence freshness dashboard. Named items behind each bucket ->
  drata-evidence-identify-gaps; changes -> drata-evidence-resolve-gaps. Read-only.
area: Compliance & Audit Readiness
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_listEvidence; optional Drata_searchMonitoringTests (automated-failing cut)"
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Rendering — branded, always. This is the default; never ask the user to pick an output mode.** Read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` and render every substantive deliverable (dashboard, report, briefing, gap worklist) in Drata branding: a self-contained HTML document using its §3 `.drata` theme. **Deliver it as HTML, always.** If the host has an artifact tool, render it there. If it does not, **write the complete HTML to a `.html` file and send that file** — every environment this runs in can deliver a file. **There is no markdown fallback.** Emitting the report as chat markdown, a bare table, or `###` headings is a failure of the deliverable, not a graceful degradation, and "the host had no artifact tool" is not a reason to do it. The only exception is the explicit text-only opt-out in rule 2. Match effort to the ask — short factual answers stay inline per §4. Two elements of a styled deliverable are a binding contract, even if the brand kit could not be read:
   - **Chart colors: Drata palette only, set explicitly in every chart config — never a library default.** First or single series `#2E4DFF`; multi-series ramp `#BEDAFF` → `#2E4DFF` → `#0F161A`; status tones `#00779C` pass / `#F2C14F` at-risk / `#D53641` fail, only on values that truly pass or fail; one `#FF410C` highlight per view at most; axis and label text `#828B8F`. Every heat map or matrix (risk 5×5, inherent × residual, any coverage grid) uses one band scale: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, at most one worst cell `#FF410C`.
   - **Table hygiene:** one fact per cell — never a chip, code list and number together; never two categories slash-merged into one row. Numeric cells `class="num"`; codes `.code`, never wrapped. Chips mark real pass/fail only — a count like "2 of 5 mapped" stays neutral ink. No Status/severity/health column that only re-buckets a count. Commentary: last column, one sentence, only where it adds signal. **Caps: 6 columns, 15 rows.** Fold rank into the lead cell ("1 · Acme") or a metric pair into `9.1 (−7.3)`; drop the weakest column rather than cram. Past 15 rows show 15 and close with "13 more — full list on request". **Columns need the theme's `18px` right gutter** — override it to `padding:… 0` and a `.num` column collides with its neighbour, headers merging into `INTEGRATIONCONTROLSCODES`. Cells are top-aligned. Wrap every table in `<div class="panel">`. **KPI tiles are uniform or they are wrong.** Every tile in a row carries its denominator in the figure — full-size numerator, then `of N` in a muted `<span class="den">`. Always the word `of` — `55 of 241`. **Never `/`, never `X/Y`, never `55/241`**, anywhere a ratio appears: KPI tiles, bar labels, table cells, body text and headings all use `of`. This is the house standard across every skill; a slash in one report and `of` in the next is the inconsistency this rule exists to prevent. **Never move the denominator into the label** (`Controls not ready (of 622)`) — the label names what is counted and nothing else, phrased the same way on every tile. **Every tile in a row counts the same polarity**: choose healthy-of-total or needs-attention-of-total once and hold it across the row, so no reader has to work out that one figure is progress and its neighbour is a problem. A figure with no available denominator does not belong in the KPI row. Never invent a score scale Drata lacks — no 0–100 health score, no weighted total, no points column; rank on real Drata numbers.
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace>`) → hairline → KPI row → real `<table>` markup → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there.
   - **No opinions, no predictions, no verdicts.** Report what Drata records and what you counted from it. **Never predict what an auditor will ask for, flag, or accept**; never label a gap *critical*, *significant*, *likely finding*, or *high risk* on your own authority; never size effort (S/M/L, hours, weeks) or estimate a date; never declare anything *audit-ready*, *compliant*, *certification-ready*, or *passing*; never interpret what a regulation or clause requires. Drata's own fields — `is_ready`, statuses, scores, dates, counts — are reportable as-is; ordering rows by those real numbers is fine, and a derived figure is labelled *Calculated*. **A reader must be able to act on this report without inheriting a judgement you made up.** If a sentence would not survive an auditor asking "where in Drata does that come from?", cut it.
   - **The artifact title names this skill's job, and no other skill's.** Title it after what this skill produces — an executive report says `Executive Report`, a gap worklist names the gaps it covers. **Never borrow a generic label like `Compliance Briefing`**: two skills wearing one title leaves the reader unable to tell which one they ran, and it collides with any similarly named skill the user has installed. Scope and date follow the title; nothing else does.
   - **Never emit a section you did not populate.** No placeholder heading, no "not included in this run", no "ask and I'll add it" offer, no note explaining which figures were not pulled. Either pull the data and render the section, or leave the section out entirely — a heading whose body apologises for itself costs the reader attention and returns nothing. The only disclosure that stays is a domain the skill *tried* to read and could not (permission denied), which is reported as one line, not a section. **Never explain what Drata does not store.** No "Drata has no asset object", no "there is no review-date field", no "the API does not expose X" — the absence of a field is your constraint while building, never a sentence in the deliverable. Where a field genuinely does not exist, answer with the nearest real Drata data, name it for what it actually is, and label it *Calculated* if you derived it; say nothing about the field you wanted and did not find. The reader came for their compliance posture, not for a tour of the data model.
   - **Reports do not carry Now · Next · Watch, and never a Decisions list.** Those are worklist patterns: an identify-gaps skill hands someone a queue to work, a report tells them where things stand. A report ends on its last content block. **Never emit a `Now · Next · Watch` block, a "Decisions required" list, recommended actions, or owner-and-option prose in this skill** — if the reader needs a worklist, route them to the matching identify-gaps skill by name in one line of body text.
   - **Chat / markdown answers:** no emoji anywhere; the word `DRATA` is always plain text, never a glyph or logo substitute; status words are exactly Ready / At-risk / Failing.
   - **Drop-in theme — embed this block once in every styled HTML artifact and use its classes** (identical to brand-kit §3, carried here so the artifact never depends on reading another file):

     ```html
     <style>
     @import url('https://fonts.googleapis.com/css2?family=Geist:wght@400;500;600&family=Geist+Mono:wght@600&display=swap');
     .drata{--space:#0F161A;--slate:#1C262B;--mist:#F5F6F7;--dust:#D9DCDE;--cobalt:#2E4DFF;
       --cobalt-700:#2039E2;--cobalt-200:#BEDAFF;--ember:#FF410C;--muted:#596064;--faint:#828B8F;
       --positive:#00779C;--warning:#F2C14F;--negative:#D53641;
       background:var(--mist);color:#0F161A;font-family:'Geist',system-ui,sans-serif;line-height:1.4;
       border:1px solid var(--dust);border-radius:4px;padding:28px 30px;}
     .drata .eyebrow{font-family:'Geist Mono',monospace;font-weight:600;text-transform:uppercase;
       letter-spacing:.10em;font-size:12px;color:var(--muted);}
     .drata h1,.drata .head{font-weight:600;font-size:30px;letter-spacing:-.02em;margin:6px 0;}
     .drata .rule{height:1px;background:var(--dust);margin:18px 0;}
     .drata .kpi{background:#fff;border:1px solid var(--dust);border-radius:4px;padding:14px 16px;}
     .drata .panel{background:#fff;border:1px solid var(--dust);border-radius:4px;padding:6px 18px;margin:14px 0;}
     .drata .sb{margin-bottom:14px;}
     .drata .sbl{display:flex;justify-content:space-between;align-items:baseline;font-size:13px;color:var(--muted);margin-bottom:5px;}
     .drata .sbl .nm{color:var(--space);font-weight:500;} .drata .sbl b{color:var(--space);font-weight:600;font-variant-numeric:tabular-nums;}
     .drata .stack{display:flex;gap:2px;height:24px;}
     .drata .stack i{display:block;height:100%;font-style:normal;}
     .drata .stack i:first-child{border-radius:4px 0 0 4px;} .drata .stack i:last-child{border-radius:0 4px 4px 0;}
     .drata .key{display:flex;gap:20px;font-size:12.5px;color:var(--muted);margin-top:8px;}
     .drata .sw{width:10px;height:10px;border-radius:2px;display:inline-block;margin-right:7px;vertical-align:-1px;}
     .drata .kpi .fig{font-size:34px;font-weight:600;letter-spacing:-.02em;line-height:1;}
     .drata .kpi .lab{font-size:13px;color:var(--muted);margin-top:6px;} .drata .kpi .fig .den{font-size:16px;font-weight:500;color:var(--muted);letter-spacing:0;}
     .drata .dot{display:inline-block;width:8px;height:8px;border-radius:999px;margin-right:6px;}
     .drata table{width:100%;border-collapse:collapse;}
     .drata th{font-family:'Geist Mono',monospace;font-weight:600;text-transform:uppercase;
       letter-spacing:.08em;font-size:11px;color:var(--faint);text-align:left;padding:0 18px 8px 0;}
     .drata th:last-child,.drata td:last-child{padding-right:0;}
     .drata td{font-size:13.5px;padding:9px 18px 9px 0;border-top:1px solid var(--dust);vertical-align:top;}
     .drata td.num,.drata th.num{text-align:right;font-variant-numeric:tabular-nums;white-space:nowrap;padding-right:18px;}
     .drata .code{font-family:'Geist Mono',monospace;font-weight:600;font-size:12px;white-space:nowrap;}
     .drata .chip{display:inline-block;font-size:11px;padding:2px 8px;border-radius:999px;
       border:1px solid var(--dust);color:var(--muted);background:#fff;}
     .drata .chip.ok{color:#005D79;border-color:#B1E3F2;} .drata .chip.gap{color:#C83039;border-color:#FDCFD7;}
     .drata .bullet{width:9px;height:9px;background:var(--cobalt);display:inline-block;} .drata .bullet.ember{background:var(--ember);}
     .drata .item{position:relative;padding:8px 0 8px 20px;border-top:1px solid var(--dust);} .drata .item:first-child{border-top:none;} .drata .item::before{content:"";position:absolute;left:0;top:13px;width:9px;height:9px;border-radius:2px;background:var(--cobalt);} .drata .item.ember::before{background:var(--ember);} .drata .item .bullet{display:none;} .drata .route{font-family:'Geist Mono',monospace;font-weight:600;font-size:11.5px;color:var(--cobalt-700);}
     .drata .logo{display:flex;align-items:center;gap:6px;font-family:'Geist Mono',monospace;font-weight:600;
       font-size:15px;letter-spacing:.04em;color:var(--space);}
     .drata .logo svg{color:var(--space);height:16px;width:auto;}
     .drata .foot{display:flex;justify-content:flex-start;border-top:1px solid var(--dust);
       margin-top:22px;padding-top:12px;}
     .drata .chrome{font-family:'Geist Mono',monospace;font-weight:600;text-transform:uppercase;
       letter-spacing:.09em;font-size:11px;color:var(--faint);}
     </style>
     ```
2. **Text-only opt-out — the only exception.** Use unbranded plain text only if the user asked for it this session ("text only", "plain text", "no styling"), or the plugin's `output_mode` setting resolves to exactly `plain` here: "${user_config.output_mode}" (any other value, including a literal `${…}` placeholder, means branded). Details and the return phrase: `${CLAUDE_PLUGIN_ROOT}/shared/output-mode.md`.
3. Source labelling (`Calculated` / `Tool Calls`): `${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`
4. **Data pulls — batch independent queries in parallel.** When this skill's workflow lists multiple MCP calls whose inputs do not depend on another call's results — count probes, per-facet or per-flag `size=1` calls, separate FAILED vs ERROR pulls, per-framework or per-workspace probes — issue them together as one parallel batch instead of one at a time; this is the single biggest speed win for report and identify-gaps runs. Keep sequential only what is genuinely dependent: any call whose filter, ID, or scope comes from a prior result (resolve the workspace or register first, expansions of found rows), and the entire preview → confirm → write → read-back chain in resolve-gaps skills. Never parallelize writes, and never let batching change a query's filters.

# Evidence Freshness

## Purpose
The read-only snapshot of the evidence library's freshness — the counts and composition, not the
queue. Answers "how fresh is the data?": what share of the library is **Valid** versus **Needs
artifact**, **Expiring soon**, and **Expired**, and *why* evidence decays — renewal cadence and
collection source. The named items behind any bucket are drata-evidence-identify-gaps; the writes
that fix them are drata-evidence-resolve-gaps. Read-only.

## Which status filters to trust — verified, not assumed
`Drata_listEvidence(statuses=[...])` is **partly reliable** — each was checked against the payload in
a live tenant. Use it exactly this way:

| Status | Verdict | Why |
|---|---|---|
| `EXPIRED` | **Use it** | every sampled `renewalDate` in the past — coherent, server-side count |
| `EXPIRING_SOON` | **Use it** | every sampled `renewalDate` a few days out. **The look-ahead window is not documented in the API and was not confirmed to be 30 days — never print a day threshold you have not measured; say "Expiring soon" and give the observed renewal-date range** |
| `NEEDS_SOURCE` | **Do not use** | returns items that plainly have a source |
| `NEEDS_ARTIFACT` | **Cross-check, never quote alone** | can return 0 even when items lack an artifact — confirm against `versions[]` |
| `READY` | **Do not use for arithmetic** | READY plus the non-READY statuses exceeds the library total, so the statuses do not partition it |

**Rule: the two renewal buckets (Expired, Expiring soon) come from the `EXPIRED` / `EXPIRING_SOON`
filters. Valid, Needs artifact, cadence and source are derived from expanded fields — never from a
status, and never build `%Valid` from status arithmetic.**

## Key tools
- `Drata_listEvidence` — the whole report.
  - **One sweep:** `size=500, expand=["controls","user","renewalSchemaAndVersions"], include_total_count=true`, paging the cursor to completion. `totalCount` is the denominator. On a multi-workspace tenant pass `workspace_id` — the wrong one returns an empty set, not an error.
  - **Two renewal counts:** `size=1` calls reading `pagination.totalCount` — `statuses=["EXPIRED"]` and `statuses=["EXPIRING_SOON"]`. Batch them with the sweep.
- `Drata_searchMonitoringTests(check_result_status="FAILED")` — optional, only for the automated-evidence-failing cut (match a `TEST_RESULT` version to its test by `source` name).

**The fields that matter** (all from the one sweep):

| Field | What it tells you |
|---|---|
| `versions[]` -> `type` | `NONE` = no artifact · `URL` / `S3_FILE` = manual · `TEST_RESULT` = automated |
| `versions[]` -> `current` | which version counts; no current entry = no artifact |
| `renewalSchema.renewalDate` | past = expired · near-term = expiring soon |
| `renewalSchema.renewalScheduleType` | `NONE` = never prompts again, so it goes stale silently |
| `controls[]` | empty = mapped to nothing, counts toward no audit |
| `user` | absent = nobody accountable |

## The freshness buckets — one state per item, and they partition
Assign each item to exactly one freshness state, **first match wins**, so the buckets sum to
`totalCount`:

1. no current version, or current version `type: "NONE"` -> **Needs artifact**
2. in the `EXPIRED` set (`renewalDate` in the past) -> **Expired**
3. in the `EXPIRING_SOON` set -> **Expiring soon**
4. otherwise (a current artifact, renewal not due) -> **Valid**

`%Valid = Valid of totalCount`, labelled *Calculated* — the one headline number, the share of the
library carrying current evidence. **Verify the four buckets sum to `totalCount` before shipping.**

## Workflow
1. **Resolve the workspace once** (multi-workspace -> ask, then reuse it for every call).
2. **One parallel batch:** the sweep, the two renewal counts, and — only if you will show the
   automated cut — the FAILED monitoring-tests pull. All are independent.
3. **Bucket every item** from the sweep payload per the rule above; take Expired / Expiring soon
   from the server-side counts and reconcile — Needs artifact wins only when there is no current
   version at all, so an item is never counted in two buckets.
4. **Compute the two "why it decays" splits** from the same payload: renewal cadence
   (`renewalScheduleType` = `NONE` versus scheduled) and collection source (`TEST_RESULT` versus
   `URL` / `S3_FILE` versus none).

## Output format
**Card-forward, minimal prose.** The report is section cards only — a header, then stacked-bar and
KPI-tile cards (bars use the theme's `.sb` / `.stack` markup). No caption paragraphs, no explanatory notes, no routing sentence, no Now · Next ·
Watch block: every card carries its own labels and keys, and where a section needs a word of
context it uses a one-word chip on the section header, never a sentence. It ends on its last card.

```
## Evidence Freshness — [workspace] · [date]
eyebrow: EVIDENCE LIBRARY · FRESHNESS     source line: Pulled from Drata · [date] · [workspace]

Section 1 — Library freshness
  Headline stacked bar: Valid | Needs artifact | Expiring soon | Expired
    (partitions the library; the bar label calls out %Valid;
     Valid #2E4DFF · Needs artifact #0F161A · Expiring soon #F2C14F · Expired #D53641)
  KPI row (needs-attention of total, one polarity):
    Needs artifact [a] of [N]  ·  Expiring soon [b] of [N]  ·  Expired [c] of [N]

Section 2 — Why evidence goes stale
  Renewal cadence bar:   on a schedule   vs   no schedule (NONE — never prompts again)
  Collection source bar: automated (test-backed)   vs   manual (URL / file)   vs   no artifact
```

**Section chips carry what the prose used to.** When a card's numbers are all live from the sweep,
mark that section `Live`; every figure must be live from the sweep — never illustrative. One chip
per section header — never a caption paragraph.

**Never attach a day threshold to "Expiring soon"** unless you measured the window in this tenant.
**Never call Expired evidence a compliance failure or predict an audit finding** — report the count
and the renewal date. The named items behind each bucket are drata-evidence-identify-gaps and the
fixes are drata-evidence-resolve-gaps; the reader runs those directly — this report prints no
routing line.

## Edge cases
| Situation | Handling |
|---|---|
| Asked *which* items are expired or missing an artifact | That is drata-evidence-identify-gaps — this report counts buckets, it does not list items |
| Buckets do not sum to the library total | Recount before rendering; the four states must equal `totalCount` |
| Tempted to build `%Valid` from `READY` | Never — statuses do not partition the library; derive Valid from the payload |
| Augmented filter used (`control_code`, `owner`) | Returns **no `totalCount`** and scans <=10 pages — never quote a count from it |
| `EXPIRING_SOON` window | Not documented in the API — give the observed renewal-date range, never a fixed "30 days" |
| Several workspaces | Ask once, pass the same `workspace_id` to every call; counts are not comparable across workspaces |
| Asked to *fix* stale evidence | That is a write — drata-evidence-resolve-gaps, under the write-safety protocol |

## Example invocations
- "How fresh is our evidence library?"
- "What share of our evidence is valid versus expired?"
- "How much evidence is expiring soon?"
- "Evidence freshness dashboard for the audit steering meeting."
- "How automated is our evidence collection?"
