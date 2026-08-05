---
name: drata-control-report
description: >
  Three charts on control readiness for a workspace: the control library split in scope vs out of
  scope, what share of in-scope controls are ready, and a
  breakdown of the not-ready ones by cause - test failing, evidence mapped but not satisfying
  readiness, policy mapped but not satisfying readiness, or nothing mapped at all. Ten count
  calls, no rows fetched. Use for 'how many controls are ready', 'why are controls not ready',
  control readiness breakdown. The named controls behind each cause -> drata-control-identify-gaps;
  changes -> drata-control-resolve-gaps. Read-only.
area: Compliance & Audit Readiness
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_searchControls"
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

# Control Readiness

## Purpose
Three charts, nothing else: **the library in scope vs out**, **how much of the in-scope control
estate is ready**, and **why the rest is not**. Read-only; the named controls behind each cause are drata-control-identify-gaps.

## Key tools
- `Drata_searchControls` — the whole report. Boolean filters AND together, and `pagination.totalCount`
  on a `size=1` call is the authoritative count: `is_ready`, `is_monitored`, `has_passing_test`,
  `has_evidence`, `has_policy`, `is_enabled` (defaults True). No facets — one call per number.
  **Never use `query`**: it switches to a semantic endpoint that ignores every filter, so counts
  drawn from it are wrong with no error.

## Workflow
**Ten `size=1` calls, one parallel batch. Never fetch rows to count.** `is_enabled` defaults to True
and enabled is what in-scope means for a control — there is no `is_in_scope` parameter here.

```
T   searchControls(workspace_id=W)                                    # in scope (is_enabled defaults True)
X   ... is_enabled=False                                              # out of scope
N   ... is_ready=False                                                # not ready — chart 3 denominator

A   ... is_ready=False, is_monitored=True,  has_passing_test=False                        # test failing
B1  ... is_ready=False, is_monitored=False, has_evidence=True
B2  ... is_ready=False, is_monitored=True,  has_passing_test=True, has_evidence=True
C1  ... is_ready=False, is_monitored=False, has_evidence=False, has_policy=True
C2  ... is_ready=False, is_monitored=True,  has_passing_test=True, has_evidence=False, has_policy=True
D1  ... is_ready=False, is_monitored=False, has_evidence=False, has_policy=False
D2  ... is_ready=False, is_monitored=True,  has_passing_test=True, has_evidence=False, has_policy=False
```

**A monitored control with a passing test can still be not ready, and it is not a category of its
own.** Attribute it to whatever is actually blocking it: evidence if it has evidence mapped, policy
if it has a policy and no evidence, nothing-mapped if it has neither. That is what the `2` half of
each pair is for — **evidence = B1 + B2, policy = C1 + C2, nothing mapped = D1 + D2.** Never render
a "monitored and passing, blocked elsewhere" segment; it names a state, not a cause, and the reader
cannot act on it.

**`has_passing_test` is not strictly binary — `True` and `False` do not sum to the whole.** On a
real workspace, not-ready monitored controls are 45, of which 25 return for `has_passing_test=False`
and 19 for `True`: one record answers neither. **So never derive one half as `total − the other
half`** on this field, and never assume the pairs above cover the population.

**Reconcile, and disclose the remainder as a segment.** Compute
`R = N − (A + B1 + B2 + C1 + C2 + D1 + D2)`. If `R > 0`, render it as a final `#D9DCDE` segment
labelled `unclassified — [n]`. A bar that does not sum to its stated denominator is a defect; a
small honest grey sliver is not.

## Output format
The rendered report opens with the title header, then the three charts in order.

## Control Readiness — [workspace] · [date]
Title it for this skill's job (control readiness); never "automation".

### Chart 1 — The control library: in scope vs out of scope
**The first bar on the page.** One `.sb` stacked bar over the whole library:
**in scope `#2E4DFF`** · **out of scope `#BEDAFF`**, counts in the label and the in-scope share
beside them.

```html
<div class="sb">
  <div class="sbl"><span class="nm">Control library</span>
    <span><b>404</b> in scope · 384 out of scope · 51% in scope</span></div>
  <div class="stack">
    <i style="width:51.3%;background:#2E4DFF" title="In scope 404"></i>
    <i style="width:48.7%;background:#BEDAFF" title="Out of scope 384"></i>
  </div>
</div>
```

**In scope means `is_enabled=True`, out of scope `is_enabled=False`** — that is the only scope
notion `searchControls` carries; there is no `is_in_scope` parameter. The library total is the sum
of the two counts, so the bar reconciles by construction; **never issue a third call for a total**
and never assume some other figure is the library size.

This bar exists to frame the two below it: every number in charts 2 and 3 is drawn from the blue
segment only. **Out-of-scope controls never appear again after this bar** — they carry no readiness
obligation, and folding them into any later denominator would understate readiness.

### Chart 2 — Ready vs not ready
One `.sb` stacked bar over in-scope controls: **ready `#2E4DFF`** · **not ready `#D53641`**, counts
and the ready percentage in the label. This is the headline and the denominator for chart 3.

### Chart 3 — Why the rest is not ready
One `.sb` stacked bar over the **not-ready** controls only. Say that denominator in the subtitle so
nobody reads it against the estate.

| Segment | Count | Colour |
|---|---|---|
| Test failing | `A` | `#D53641` |
| Evidence mapped, not satisfying readiness | `B1 + B2` | `#F2C14F` |
| Policy mapped, not satisfying readiness | `C1 + C2` | `#BEDAFF` |
| **Nothing mapped** | `D1 + D2` | `#0F161A` |
| Unclassified — only when `R > 0` | `R` | `#D9DCDE` |

**The segments are ordered and mutually exclusive by construction**: a failing test wins, then
evidence, then policy only where there is no evidence, then neither. A control with both evidence
and policy lands in evidence and appears once. **Never drop `has_evidence=False` from the policy
filters** — the two would overlap and the bar would over-count.

**The first three mean something is mapped and it is not carrying the control; the black segment
means nothing is mapped at all.** That is the distinction the chart exists to draw, and the black
segment is the one to read first — work not started rather than work not finished.

**Do not assert why** the mapped thing is failing — "evidence is stale", "policy unpublished". The
API exposes no such field. Say what is true: it is mapped and readiness is not satisfied.

**Verify before shipping:** the segments plus `R` equal `N`, and `ready + N = T`.

## Edge cases
| Situation | Handling |
|---|---|
| Segments do not sum to the not-ready total | Disclose the shortfall as the grey `unclassified` segment — never silently rescale, and never drop it |
| Monitored control with a passing test but not ready | Attribute it to evidence, policy or nothing-mapped — never give it its own segment |
| Asked which controls are in a segment | That is drata-control-identify-gaps; this report counts causes, it does not list controls |
| `has_passing_test=False` read as "failing control" | It includes controls with no test at all — that is why segment 1 also requires `is_monitored=True` |
| Disabled controls | They are the out-of-scope segment of chart 1 and appear nowhere else |
| Several workspaces | Ask once, reuse for the run. Counts are not comparable across workspaces |
| Asked for framework or topic breakdown | Not in this report — framework readiness is drata-framework-report |
| Asked whether failing controls have remediation tickets | **Not reported here.** `ticket_status=IN_PROGRESS` returns every control that is not ARCHIVED — including the hundreds with no ticket at all — so any ticket-coverage figure built from it is wrong in the flattering direction. Say the ticket data is not reliable enough to chart and stop; do not substitute a row-counted approximation |
| Out-of-scope controls in a readiness figure | Never — they appear only in chart 1; readiness denominators are in-scope only |

## Example invocations
- "How many of our controls are ready?"
- "Show me why our controls are not ready."
- "Break down the not-ready controls by cause."
- "Control readiness chart for the leadership deck."
