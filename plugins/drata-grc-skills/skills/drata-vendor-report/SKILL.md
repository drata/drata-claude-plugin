---
name: drata-vendor-report
description: >
  The vendor portfolio dashboard (account-scoped), chart-led and high-level: portfolio composition
  (current / prospective / archived), assessment coverage, an inherent x residual grid, a category
  donut, exposure, and a single intake bar for prospective vendors. Built entirely from count calls,
  with no per-vendor loops. Use for 'vendor risk overview', TPRM posture, unassessed vendors, vendor
  intake. Review currency and the action queue -> drata-vendor-identify-gaps; changes ->
  drata-vendor-resolve-gaps. Read-only.
area: Third-Party & Vendor Risk
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_listVendors"
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
     .drata .hm{display:inline-grid;grid-template-columns:auto repeat(5,46px);grid-auto-rows:46px;gap:6px;align-items:center;margin:4px 0 2px;}
     .drata .hm .c{width:46px;height:46px;display:flex;align-items:center;justify-content:center;
       border-radius:4px;font-weight:600;font-size:14px;font-variant-numeric:tabular-nums;color:var(--space);}
     .drata .hm .c.on{color:#fff;}
     .drata .hm .ax{font-family:'Geist Mono',monospace;font-size:11px;color:var(--faint);letter-spacing:.06em;}
     .drata .hm .ax.y{text-align:right;padding-right:6px;} .drata .hm .ax.x{text-align:center;height:auto;}
     .drata .dn{display:flex;align-items:center;gap:26px;margin:6px 0 2px;}
     .drata .dnr{width:150px;height:150px;border-radius:50%;flex:none;
       -webkit-mask:radial-gradient(circle,transparent 58px,#000 59px);mask:radial-gradient(circle,transparent 58px,#000 59px);}
     .drata .dnl{font-size:13px;color:var(--muted);}
     .drata .dnl div{margin-bottom:6px;} .drata .dnl b{color:var(--space);font-weight:600;font-variant-numeric:tabular-nums;}
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

# Vendor Portfolio Dashboard

## Purpose
Answer **"where does our third-party portfolio stand?"** with one artifact: an account-scoped
**Vendor Portfolio dashboard** — how many vendors we actually have, how they spread across
residual risk and inherent impact, what they are (category, type, sub-processor), and who owns
them. Read-only; every tile is backed by an exact
structured `Drata_listVendors` query, so the numbers match the Drata UI.

## Key tools
- `Drata_listVendors` — the whole dashboard. With **no `status` filter and no cursor** it attaches `breakdown = {current, total, prospective, archived}`. Filters: `risk`, `impact_level`, `category[]`, `type[]`, `status[]`, `sub_processor`, `owner` (accepts a **list**, OR'd in one call), `renewal_date`, `next_review_deadline[]`, `action_required`. `expand`: `lastQuestionnaire`, `latestSecurityReviews`, `documents`, `integrations`.
- `Drata_listVendorDocuments`, `Drata_listVendorSecurityReviews`, `Drata_getVendor` — single-vendor drill-downs; this skill does not call them (see drata-vendor-identify-gaps).

## Workflow
1. **Scope.** Vendors are **account-scoped** — do **not** ask which workspace.
2. **Headline count — this call first, exactly like this:**
   ```
   Drata_listVendors()          # NO status filter, NO cursor → attaches `breakdown`
   ```
   Report **`breakdown.current`** as the headline, with `prospective` and `archived` beside it.
   **`current` is NOT the same as `status=ACTIVE`** — never substitute one for the other.
   Reporting raw `pagination.totalCount` as "our vendors" **contradicts the number the customer
   sees in the Drata UI**: it counts prospective and archived records too. Name the denominator
   on every tile.
3. **Every number on this dashboard comes from a `size=1` count call. Never fetch rows to count.**
   `Drata_listVendors(size=1, <one filter>)` returns `pagination.totalCount` and, whenever no
   `status` filter is applied, a server-computed `breakdown` of `{current, total, prospective,
   archived}` **that respects the filter you just passed**. That is the whole dashboard: about
   sixteen tiny calls, issued as **one parallel batch**, no pagination, no expands.

   **Read `breakdown.current`, never `pagination.totalCount`, for anything describing the portfolio
   you actually run.** The gap is large and always in the alarming direction, because archived
   vendors keep their old risk scores and lapsed review dates forever. On a real register:
   `risk=HIGH` returns 41 total but **16 current**; `next_review_deadline=OVERDUE` returns 62 total
   but **14 current**. Publishing the totals overstates the problem three- to four-fold and the
   error is invisible — both numbers are real, only one answers the question.

   **`status` filters suppress the breakdown.** When you filter by `status` the response carries no
   `breakdown`, so the figure is a raw total across every lifecycle state. Use `status` only for the
   intake funnel, where that is what you want, and never mix a status-filtered count into a
   current-portfolio chart.

4. **Never loop over vendors.** `Drata_listVendorDocuments` and `Drata_listVendorSecurityReviews`
   take a single `vendor_id`, so using them for portfolio figures means one call per vendor —
   hundreds of round trips, minutes of wall clock, and a response far past the size limit. **They
   are single-vendor drill-down tools and this skill does not call them at all.** The same applies
   to `expand=["documents","reviews","latestSecurityReviews"]` on a list call: heavy subcollections
   across hundreds of vendors is the other way this dashboard becomes slow. Attestation and
   questionnaire detail for one named vendor is drata-vendor-identify-gaps's job.

5. **Risk tiers — name the mapping.** Vendors carry two independent scores, and this is the most
   commonly mislabeled pair in vendor reporting:
   - **`risk`** (`NONE|LOW|MODERATE|HIGH`) = **residual** risk in the Drata UI.
   - **`impact_level`** (`INSIGNIFICANT…CRITICAL|UNSCORED`) = **inherent** risk in the Drata UI.
   Label each axis with both the API field and the UI word. Vendors have **no `facets` support**,
   unlike risks and personnel — hence the one-call-per-bucket pattern.

## Output format
**Chart-led. Two audiences, two questions, in this order.** A prospective vendor and a vendor you
already run need different things done, so they never share a chart: intake asks *where is this in
approval and what is blocking it*, the portfolio asks *is our assessment of what we already use
still valid*. Every block below is a bar, a grid or a funnel — tables only where a bar cannot carry
the value.

```
## Vendor Portfolio — [date]   (account-scoped)
Source line: `Pulled from Drata · [date] · account-scoped`

Portfolio ······ ONE stacked bar: current · prospective · archived   (sums to total)
KPI row ········ two tiles, both `n of [current]`, both needs-attention polarity:
                 inherent unscored · high residual

— CURRENT VENDORS ([breakdown.current]) —
Assessment coverage ··· ONE stacked bar: inherent scored vs UNSCORED
Inherent × residual ··· `.hm` grid, 5 rows x 4 cols, cell = vendor count  (Calculated)
                        row totals ARE the inherent distribution, column totals the residual one
By category ··········· donut, top 6 + Other + Uncategorised           (one call per category)
Exposure ·············· ONE stacked bar: sub-processor vs not

— PROSPECTIVE / INTAKE ([breakdown.prospective]) —
Intake ················ ONE stacked bar: under review vs not yet picked up, then decided
                        (status-filtered counts — raw totals, labelled as such)
```

### The inherent x residual grid replaces both risk bar charts
**Never render a separate inherent-risk bar chart and residual-risk bar chart alongside the grid.**
The grid is their cross-tabulation: its **row totals are the inherent distribution and its column
totals are the residual distribution**, so two bar charts beside it re-draw the grid's own margins
and spend three blocks saying what one says. Print the margins as a total row and a total column on
the grid itself and the bars have no remaining job. The grid also answers what neither bar can —
*which* vendors were scored high inherent and successfully brought down, versus high on both.

**Rows are the five real inherent levels, columns the four residual levels — a 5 x 4 grid, not
5 x 5.** Set `grid-template-columns:auto repeat(4,46px)` for this skill; the shared `.hm` CSS
defaults to five columns for the risk 5x5 and will leave an empty column here if you do not.

**UNSCORED is not a row.** It is the absence of a score, not a level, and on a real portfolio it
holds most of the population — as a row it would dwarf every real cell and make the grid unreadable.
The grid's denominator is **scored current vendors only**, stated in the subtitle
(`117 of 431 current vendors are scored on both axes`), and the unscored remainder is already the
headline of the assessment-coverage bar above it. Never let a reader infer the grid covers the
portfolio.

### Category donut
One donut over `category`, **top 6 slices plus `Other` plus `Uncategorised`**, counts on the labels.
Ordered largest first, Uncategorised always last and always in `#D9DCDE` whatever its size.
Built with the theme's `.dn` classes — a `conic-gradient` ring, legend beside it, never a canvas or
chart library:
```html
<div class="dn">
  <div class="dnr" style="background:conic-gradient(#2E4DFF 0 22.7%,#0F161A 22.7% 38%,
       #BEDAFF 38% 49%,#2E4DFF 49% 57%,#0F161A 57% 63%,#BEDAFF 63% 68%,#D9DCDE 68% 100%)"></div>
  <div class="dnl"><div><i class="sw" style="background:#2E4DFF"></i>Engineering <b>98</b></div>…</div>
</div>
```
Slice colours cycle the multi-series ramp anchors `#2E4DFF` · `#0F161A` · `#BEDAFF` — palette
values only, no interpolated hues — so the donut reads as one measure split
into parts, not seven unrelated series; the closing `#D9DCDE` neutral carries the remainder
(`Other` + `Uncategorised`). **Never a categorical rainbow**, and never a status colour
here: a category is not pass or fail.

**Drop every category whose `breakdown.current` is zero** rather than drawing a zero-width slice —
the enum carries values this account does not use, and an empty slice in the legend reads as a
category with vendors in it. **Uncategorised is mandatory when non-zero**: category is frequently
unset, so a donut of only the populated categories silently rebases the percentages onto a subset
and overstates every slice. The denominator is `breakdown.current`, named beneath the donut.

A donut is only legible to about eight slices — that is why the cap is six plus two. **Never render
one slice per enum value**, and never a second donut for type or sub-processor; those are bars.

### Intake — one stacked bar, not a funnel
**One `.sb` stacked bar across the prospective population**, segmented by where each vendor sits:
**under review** `#2E4DFF` · **not yet picked up** `#F2C14F` · **decided** (approved, rejected, on
hold) `#D9DCDE`. That single bar answers the only question intake has — *how much of the queue is
someone actually working* — in one line.

**Never three separate bars or a stepped funnel.** A funnel implies vendors flow left to right in
fixed proportion and that earlier stages contain later ones; these are mutually exclusive states at
a point in time, so the funnel shape asserts a progression the data does not describe. Three bars
each scaled to the same denominator force the reader to re-stack them mentally — the same defect as
the treatment-status chart.

**Status-filtered counts carry no `breakdown`**, so these are raw totals across every lifecycle
state. Say so in the subtitle and never mix them into a current-portfolio figure.

### The unscored row is a data-quality finding, not just a gap
A vendor can carry a **residual** score with no **inherent** score — the two fields are set
independently. Since UNSCORED is excluded from the grid, that population would otherwise vanish, so
report it as a single line beneath the grid: how many unscored-inherent vendors nonetheless hold a
residual of Low, Moderate or High. Those are vendors someone has judged without recording what they
were judging against, and a reader chasing the assessment backlog needs them separated from vendors
that were never touched at all.

**Lead with coverage, not composition.** The first two bars answer *how much of this portfolio have
we actually assessed*, and on a real register that is the finding: **314 of 431 current vendors
carry `impactLevel: UNSCORED`**. A category or type
breakdown drawn above those describes a portfolio nobody has evaluated yet, which reads as far more
control than exists. Composition charts are optional; coverage is not.

### Portfolio composition — the prospective / current split
**One stacked bar, not tiles:** `current` · `prospective` · `archived`, straight from the unfiltered
`breakdown`, summing to `breakdown.total`. It is the first visual on the page and it sets the scope
for everything under it.

**It is a bar rather than a KPI row on purpose.** Current and prospective are parts of one whole
measured in one unit, which is what a stacked bar is for; as tiles they would sit beside the
needs-attention tiles at a different denominator and different polarity, and the shared KPI rule
forbids a row that mixes a population figure with a problem figure. Keep the KPI row to the two
`of current` tiles and let this bar carry the split.

**Review currency belongs to drata-vendor-identify-gaps, not here.** No review-overdue tile, no
no-review-schedule tile, no review-coverage bar on this dashboard: "which vendors are due" is a
worklist someone acts on, and a report that opens with it is a queue wearing a dashboard's clothes.
This skill answers *what does the portfolio look like and how much of it have we assessed*; the
`next_review_deadline` counts live in the identify-gaps skill. Keep `Drata_listVendors(size=1,
next_review_deadline=…)` out of the call batch entirely.

**The sixteen calls, as one parallel batch** — this is the entire data layer:
```
Drata_listVendors(size=1)                                    # breakdown: current/prospective/archived
Drata_listVendors(size=1, impact_level=<each of 6>)          # inherent, read breakdown.current
Drata_listVendors(size=1, risk=<each of 4>)                  # residual, read breakdown.current
Drata_listVendors(size=1, sub_processor=True)                # read breakdown.current
Drata_listVendors(size=1, category=<each>)                   # donut, read breakdown.current
Drata_listVendors(size=1, status=<intake states>)            # intake only — raw totals
```

**Do not add a security-review-status chart.** `listVendors` exposes no review-status filter, so a
portfolio-wide breakdown of it can only be assembled by calling `listVendorSecurityReviews` once per
vendor — the per-vendor loop this skill exists to avoid. Review *currency* is
drata-vendor-identify-gaps's job (`next_review_deadline`); so is review *status* for a named
vendor.

**Do not chart `type`.** The VENDOR/SUPPLIER/CONTRACTOR/PARTNER enum is optional and typically left
unset, so a type chart mostly measures data entry. Check its counts before considering it, and drop
it whenever the populated values cover a small share of `breakdown.current`.
Nothing else. If a run is taking more than a few seconds, it is fetching rows or looping vendors —
stop and go back to counts.

**Prospective and current never share a bar, a tile or a denominator.** Every current-portfolio
figure is `of breakdown.current`; every intake figure is `of breakdown.prospective`. A tile mixing
them answers no question either audience has.

## Edge cases
| Situation | Handling |
|---|---|
| Tempted to report `totalCount` as the vendor count | Use `breakdown.current`; `totalCount` counts prospective + archived and contradicts the Drata UI |
| A `status` filter or cursor was passed | `breakdown` is **not** attached — re-call with neither for the headline |
| "High-risk vendors" requested | `risk=HIGH` is residual — say so, and offer `impact_level` (inherent) alongside |
| User asks "which workspace?" | Vendors are account-scoped — no workspace prompt needed |
| Trend or quarter-over-quarter asked | No historical snapshots exist in Drata — say so, or baseline against a prior report |
| `impact_level=UNSCORED` is large | Show it as its own bar and flag it as a scoring gap, not as low risk → score them via drata-vendor-resolve-gaps |
| "Which vendors need security reviews this quarter?" | That is a queue question — route to drata-vendor-identify-gaps |

## Example invocations
- "Drata, show me all high-risk vendors with expired SOC 2 reports."
- "Drata, how many vendors do we actually have — and how do they break down by risk?"
- "Give me a third-party portfolio dashboard for the board."
- "Drata, break our vendors down by category and owner."
- "Which of our vendors are sub-processors?"
