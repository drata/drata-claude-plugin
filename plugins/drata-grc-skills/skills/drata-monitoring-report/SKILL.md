---
name: drata-monitoring-report
description: >
  The monitoring dashboard for a workspace, chart-led with no tables and using Drata's own terms:
  test status (Enabled vs Disabled), test results (Passed, Failed, Error), days failing and failures
  by check type - all scoped to Production so the numbers match the Drata UI - plus a Production vs
  Code chart. Count calls plus one bounded pass over the failing set. Use for
  'monitoring coverage', 'how many tests are failing', how long tests have been failing, switched-off
  tests. Control-level causes -> drata-control-identify-gaps. Read-only.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_searchMonitoringTests"
metadata:
  area: Compliance & Audit Readiness
  permission: read-only
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Rendering — branded, always. This is the default; never ask the user to pick an output mode.** Read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` and render every substantive deliverable (dashboard, report, briefing, gap worklist) in Drata branding: a self-contained HTML document using its §3 `.drata` theme. **Deliver it as HTML, always.** If the host has an artifact tool, render it there. If it does not, **write the complete HTML to a `.html` file and send that file** — every environment this runs in can deliver a file. **Name the file after this skill's folder, exactly: `<skill-name>-<scope>-<YYYY-MM-DD>.html` (e.g. `drata-framework-report-soc-2-2026-08-04.html`) — never a shortened or re-worded variant of the skill name, and never the artifact's display title.** **There is no markdown fallback.** Emitting the report as chat markdown, a bare table, or `###` headings is a failure of the deliverable, not a graceful degradation, and "the host had no artifact tool" is not a reason to do it. The only exception is the explicit text-only opt-out in rule 2. Match effort to the ask — short factual answers stay inline per §4. Two elements of a styled deliverable are a binding contract, even if the brand kit could not be read:
   - **Chart colors: Drata palette only, set explicitly in every chart config — never a library default.** First or single series `#2E4DFF`; multi-series ramp `#BEDAFF` → `#2E4DFF` → `#0F161A`; status tones `#00779C` pass / `#F2C14F` at-risk / `#D53641` fail, only on values that truly pass or fail; one `#FF410C` highlight per view at most; axis and label text `#828B8F`. Every heat map or matrix (risk 5×5, inherent × residual, any coverage grid) uses one band scale: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, at most one worst cell `#FF410C`.
   - **Table hygiene:** one fact per cell — never a chip, code list and number together; never two categories slash-merged into one row. Numeric cells `class="num"`; codes `.code`, never wrapped. Chips mark real pass/fail only — a count like "2 of 5 mapped" stays neutral ink. No Status/severity/health column that only re-buckets a count. Commentary: last column, one sentence, only where it adds signal. **Caps: 6 columns, 15 rows.** Fold rank into the lead cell ("1 · Acme Corp") or a metric pair into `X.X (−Y.Y)`; drop the weakest column rather than cram. Past 15 rows show 15 and close with "13 more — full list on request". **Columns need the theme's `18px` right gutter** — override it to `padding:… 0` and a `.num` column collides with its neighbour, headers merging into `COLUMNACOLUMNB`. Cells are top-aligned. Wrap every table in `<div class="panel">`. **KPI tiles are uniform or they are wrong.** Every tile in a row carries its denominator in the figure — full-size numerator, then `of N` in a muted `<span class="den">`. Always the word `of` — `N of M`. **Never `/`, never `X/Y`, never `N/M`**, anywhere a ratio appears: KPI tiles, bar labels, table cells, body text and headings all use `of`. This is the house standard across every skill; a slash in one report and `of` in the next is the inconsistency this rule exists to prevent. **Never move the denominator into the label** (`Controls not ready (of M)`) — the label names what is counted and nothing else, phrased the same way on every tile. **Every tile in a row counts the same polarity**: choose healthy-of-total or needs-attention-of-total once and hold it across the row, so no reader has to work out that one figure is progress and its neighbour is a problem. A figure with no available denominator does not belong in the KPI row. Never invent a score scale Drata lacks — no 0–100 health score, no weighted total, no points column; rank on real Drata numbers.
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Header identity — the customer's logo, top-left, only when it can truly be inlined; else the company name as text.** Call `Drata_getCompany` once per run before rendering (account-scoped, no arguments, read-only; batch it with the run's other independent reads). It returns `name`, `legalName` and `logoUrl`. Resolve the header in this order and stop at the first that succeeds:
     1. **Inlined logo — gate first, then fetch, then verify.** Attempt this step only if `logoUrl` is non-empty **and** the host provides a tool that can actually download raw image bytes from an arbitrary URL. Many sandboxed hosts — including Claude's cloud / Cowork environments — forbid fetching arbitrary CDN URLs, and the Drata image CDN additionally refuses generic fetchers; **in those hosts this step fails immediately and silently, and falling through to the name is the designed outcome, not a degraded render.** Where a download is possible: fetch once (no retries, no proxies, no cache mirrors, never a route around a refusal), verify the bytes decode as a real image (image magic bytes, mime `image/*`, non-zero size), base64-encode **those downloaded bytes with a real encoder in this run**, and emit `<img class="cust" src="data:[mime];base64,[data]" alt="[company name]" onerror="this.style.display='none';this.nextElementSibling.style.display='block'"><div class="custname" style="display:none">[company name]</div>`. **Never type, reconstruct, or approximate base64 from memory — fabricated image data renders a broken or wrong mark exactly where the customer's identity belongs.** If any part of this step cannot be completed and verified, it did not succeed.
     2. **Company name as text.** `<div class="custname">[company name]</div>` — used whenever `logoUrl` is absent or empty, no permitted fetch path exists in this host, the fetch fails or is refused, the bytes are not a decodable image, or the base64 cannot be produced from real downloaded bytes. **This fallback is first-class: a report headed by the company's name in clean type is a correct header; a broken image, an empty header, or invented image data is the only failure.**
     **Never emit `<img src="https://…">`.** A remote reference is not an acceptable third option: artifact sandboxes block external images, and a blocked, expired or access-controlled URL renders a broken-image icon exactly where the customer's identity belongs. It is a verified inlined image or it is the name — nothing in between.
     **When the logo renders, the company name does not appear as visible text** — it lives in the `alt` attribute and in the hidden `onerror` fallback `<div>`, which stays invisible unless the image fails to decode; that hidden div is the safety net, not a second header.
     **Proportions: constrain the height, leave the width free.** `height:32px; width:auto; max-width:200px; object-fit:contain` — a wide wordmark and a square icon then share one baseline with no stretching, squashing or cropping. **Never set `height` and `width` together, never `width:100%`, never a fixed pixel width**, and never re-encode the image to a different aspect ratio. If a logo would exceed `max-width` at 32px tall, `object-fit:contain` shrinks it proportionally — that is correct, do not compensate.
     On the dark board/exec surface, an inlined dark-on-transparent logo disappears; use `<div class="custname" style="color:#fff">[company name]</div>` instead rather than shipping an invisible mark.
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace name>`) → hairline → KPI row → real `<table>` markup → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there. **The source line always spells the scope out in full** — the workspace's own name, or `All workspaces` for an org roll-up covering more than one. Never omit it, never abbreviate it, never substitute a workspace id or a slug, and never leave it to be inferred from the title.
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
     .drata .cust{height:32px;width:auto;max-width:200px;object-fit:contain;display:block;margin:0 0 14px;}
     .drata .custname{font-weight:600;font-size:15px;letter-spacing:-.01em;color:var(--space);margin:0 0 14px;}
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

# Monitoring Coverage Report

## Purpose
A chart-led picture of the automated monitoring tests in a workspace: how many are `Enabled`, how
many of those `Passed`, and — the question that matters most — **how many days the `Failed` ones
have been failing**. No tables, no test lists, no root-cause grouping. Read-only.

## Key tools
- `Drata_searchMonitoringTests` — the whole report. `pagination.totalCount` on a `size=1` call is
  authoritative; there are no facets, so one call per bucket.
  Filters: `check_status` (UNUSED, NEW, ENABLED, DISABLED, TESTING), `check_result_status`
  (READY, PASSED, FAILED, ERROR, PREAUDIT), `check_type` (10 connection categories),
  `test_source` (DRATA, CUSTOM, EXTERNAL, ACORN, DRATA_LIBRARY) — **`ACORN` is Code**, the only
  value that is not Production. **Never use `query`** — it ignores every filter.
- Row fields: `checkResultStatus`, `checkStatus`, `testSource`, `testId`, `lastPassedAt`,
  **`failedSince`**. `check_type` is filterable but **not returned on the row**, so its distribution
  needs one count call per value.

## Production is the population — every chart, every number
**The Drata monitoring list shows only Production tests. Every figure in this report is Production
unless a chart says otherwise.** Pool Production and Code and every number comes out higher than the
screen the reader is checking against, with no clue why.

`test_source` takes one value at a time, so there is no "everything except Code" filter — **derive
Production by subtraction on every single count**:
```
Production = <count> − <same count with test_source="ACORN">
```
That is two calls per figure, not one. It is the price of matching the UI and it is not optional.

Verified on a test workspace: the API total for a check type included Code-pool failures, and only the Production remainder matched the UI. Reporting the raw API total was wrong.

**Every chart heading states its own split, so the exclusion is never invisible.** Each section
carries a subtitle in one fixed shape:

```
Production [n]  ·  Code [n] not shown
```

Applied down the report — the numbers are that chart's own population, not the report total:

| Chart | Subtitle |
|---|---|
| Test status | `Production [N] · Code [M] not shown` |
| Test results | `Production [N] Enabled · Code [M] Enabled not shown` |
| Days failing | `Production [N] Failed · Code [M] Failed not shown` |
| Check type | `Production [N] Failed · Code [M] Failed not shown` |
| Test source | *(no subtitle — this chart is the split)* |

**Write it on every chart even when the Code figure is zero** (`Code 0 not shown`). A missing
subtitle reads as "nothing was excluded", which is a different claim from "nothing was there to
exclude", and only one of them is checkable.

**Code appears in the subtitles and in the final chart. Never in a bar, a segment or a headline
figure** anywhere else.

## Workflow
**Count calls for everything except ageing, one parallel batch. Each figure is a pair — the raw
count and its Code counterpart — so Production can be derived.**
```
Drata_searchMonitoringTests(size=1)                                  # all tests
  ... check_status="ENABLED"          ... check_result_status="PASSED"
  ... check_result_status="FAILED"    ... check_result_status="ERROR"
  ... check_result_status="FAILED", check_type=<each of 10>          # chart 4

# the same set again with test_source="ACORN", to subtract:
  ... test_source="ACORN"              ... test_source="ACORN", check_status="ENABLED"
  ... test_source="ACORN", check_result_status="PASSED" / "FAILED"
  ... test_source="ACORN", check_result_status="FAILED", check_type=<each of 10>
```
`Disabled = Production total − Production enabled`; never filter for it separately.
Then **one** row call for the ageing chart — bounded to the failing set:
```
Drata_searchMonitoringTests(check_result_status="FAILED", size=50)   # paginate if >50
```
`failedSince` is on every failing row; bucket it locally. That is the only row fetch in the report.

**`check_result_status="READY"` is broken — never use it.** On a test workspace it returns **0** while rows plainly carry `checkResultStatus: "READY"`. Those are the switched-off tests, and they
are already counted by `check_status="DISABLED"` — use that instead and never report a READY figure.

**Verify the arithmetic before shipping** — these identities catch a dropped filter instantly, and
they must hold **on the Production figures**, not the pooled ones:
- `Production Enabled + Production Disabled = Production total`
- `Production Passed + Failed + Error = Production Enabled`
- `sum of the ten check types = Production Failed`

On a test workspace the pooled totals included a sizeable Code pool; subtracting it per metric is what yields the Production figures. The report shows the Production
column.

## Output format
```
## Monitoring Coverage — [workspace] · [date]
Source line: `Pulled from Drata · [date] · [workspace] · [n] tests`

Test status ····· ONE stacked bar: Enabled vs Disabled            (check_status)
Test results ···· ONE stacked bar: Passed vs Failed vs Error      (check_result_status, of Enabled)
Days failing ···· bars, how long each failed test has been failing  ← the headline
Check type ······ bars, Failed tests per connection category       (check_type)
Test source ····· ONE stacked bar: Production vs Code              (of Failed)
```

### Chart 1 — Test status
**Use Drata's own vocabulary for every heading and segment label in this report.** The API calls
`check_status` the *test system status* and `check_result_status` the *test result status*, so the
two charts are **Test status** and **Test results**, and the segments are the enum values written
in sentence case — `Enabled`, `Disabled`, `Passed`, `Failed`, `Error`. **Never invent a synonym**:
no "estate", "health", "coverage score", "switched on", "green/red". A reader who filters the Drata
monitoring list should see the same words there as here, or the report cannot be checked against it.

One `.sb` / `.stack` stacked bar: `Enabled` `#2E4DFF` · `Disabled` `#D9DCDE`, over Production tests. It sets the denominator for the
next chart and exposes a large `Disabled` population that a pass rate alone would hide.
Subtitle: `Production [N] · Code [M] not shown`.

### Chart 2 — Test results
`Passed` `#2E4DFF` · `Failed` `#D53641` · `Error` `#F2C14F`, **over Production Enabled tests only**,
subtitle `Production [N] Enabled · Code [M] Enabled not shown`. A disabled
test is neither passing nor failing; including it in the denominator flatters the pass rate.
**Render the `Error` segment even at zero** in the legend — an errored test is broken tooling, not a
failing control, and its absence is a finding worth seeing.

### Chart 3 — Days failing
**This is the headline of the report.** Bars over the Production `Failed` tests, subtitle
`Production [N] Failed · Code [M] Failed not shown`, bucketed on `failedSince`:
**0–7 · 8–30 · 31–90 · 91–180 · 180+ days**, with the median in the subtitle.

A count of failing tests says almost nothing on its own — a set of failures that all broke this week is an incident; the same count from months ago is a backlog, and they call for completely different responses.
On a test workspace **most failures were older than 90 days, with a long tail past a year** — which the raw failing count entirely conceals.

Colour the buckets on the band scale `#BEDAFF` → `#F2C14F` → `#D53641` across the buckets so age reads as severity left to right.
**Never label a bucket with a judgement** — no "stale", "abandoned", "critical". The bucket is a
number of days; what it means is the reader's call.

### Chart 4 — Check type
Bars over the Production `Failed` tests, one per connection category, longest first, subtitle
`Production [N] Failed · Code [M] Failed not shown`. It answers *which
integration is generating the failures* — and the distribution is usually lopsided enough to be actionable on its own:

```
Infrastructure    [a]      ([A] in the API − [B] Code)
Version control   [b]
Policy            [c]
Agent             [d]
In Drata          [e]
Observability     [f]
Identity          [g]
                  ──
                  [N]      = Production Failed
```

**The categories partition the failures exactly**, so `sum of all ten = Production Failed`. Compute
that and check it before shipping — cheapest correctness test in the report, and it catches both a
mistyped enum and a forgotten Code subtraction.

**Subtract Code per category, not once at the end.** On the test workspace every Code failure happened to fall in one check type, so a single lump subtraction would have looked right by luck; on any other
distribution it silently misassigns failures between categories.

**`check_type` is not returned on the row**, so this costs one `size=1` count call per value, twice
over — ten for the raw counts and ten more with the Code filter to subtract. Twenty tiny calls in
the same parallel batch. There is no way to derive it from the failing rows you already fetched.

**Drop the zero categories from the bars, keep them in the identity check.** On the test workspace several check types were zero; drawing empty bars for them wastes three rows and
implies a gap where there is simply no test of that kind. The sum still has to reach `Failed`.

**Use the enum values, sentence-cased, exactly as the API spells them** — `In Drata`, `Version
control`, `Agent`, `HRIS`. **Never rename a category to something friendlier**: `Agent` is not
"devices", `In Drata` is not "internal". Same rule as the status labels — the reader has to be able
to set this filter in Drata and see the same number.

Single-hue ramp `#2E4DFF → #BEDAFF`, largest first. This chart counts tests, not severity, so
**never colour a category red** — how bad a category is depends on the days-failing chart above,
not on how many tests it holds.

### Chart 5 — Test source
One stacked bar showing Production and Code side by side across the `Failed` tests — the one place
Code appears. Keep it last: it is a fact about `test_source`, not a
ranking, and nothing in this report should imply one population matters more.

**No tables anywhere in this report.** No test names, no testIds, no owner columns, no
failing-longest list. If the reader needs the individual tests, that is a different deliverable —
say so in one line and stop.

## Edge cases
| Situation | Handling |
|---|---|
| `check_result_status="READY"` returns 0 | It is broken. Those tests are the DISABLED population — count them with `check_status` and never report a READY figure |
| Pass rate quoted against all tests | Wrong denominator — a `Disabled` test is neither passing nor failing. Test results are always of `Enabled` |
| A figure does not match the Drata UI | Almost always pooled Production and Code. Subtract Code and re-check before assuming the UI is wrong |
| Code subtracted once at the end | Wrong — subtract per figure and per check type, or failures get misassigned between categories |
| Chart with no split subtitle | Incomplete — every chart states `Production [n] · Code [n] not shown`, including when Code is 0 |
| Asked which tests are failing | Not this report — it has no tables. Say so in one line and stop |
| Asked to group failures by root cause | Not this report. It counts and ages; it does not diagnose |
| `check_type` breakdown | Ten count calls — it is not on the row. Drop zero categories from the bars, but the ten still have to sum to `Failed` |
| Tempted to rename a check type | Never — `Agent` is not "devices", `In Drata` is not "internal". Use the enum spelling so the reader can set the same filter in Drata |
| ERROR count is zero | Still show the segment in the legend; a zero there is a real reading |
| Several workspaces | Ask once, reuse. Tests are workspace-scoped |
| Tempted to label an age bucket | Never — no "stale", "abandoned", "critical". The bucket is a number of days |

## Example invocations
- "Show me our monitoring coverage."
- "How many monitoring tests are switched off?"
- "How long have our failing tests been failing?"
- "What share of enabled tests are passing?"
- "Monitoring status chart for the leadership deck."
