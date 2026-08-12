---
name: drata-evidence-identify-gaps
description: >
  Read the evidence library once and split it into four workable buckets — needs artifact (by
  framework), automated evidence whose test is failing, renewal overdue, renewal due soon — plus
  callouts for buckets mapped to no control and buckets missing owner, renewal schedule, artifact
  or description. Derived from real payload fields, not the unreliable status filter. Use for
  'what's broken in our evidence library', audit-prep sweeps, renewal planning. Fixes →
  drata-evidence-resolve-gaps. Read-only.
area: Compliance & Audit Readiness
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listEvidence, Drata_searchControls, Drata_searchMonitoringTests"
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Rendering — branded, always. This is the default; never ask the user to pick an output mode.** Read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` and render every substantive deliverable (dashboard, report, briefing, gap worklist) in Drata branding: a self-contained HTML document using its §3 `.drata` theme. **Deliver it as HTML, always.** If the host has an artifact tool, render it there. If it does not, **write the complete HTML to a `.html` file and send that file** — every environment this runs in can deliver a file. **Name the file after this skill's folder, exactly: `<skill-name>-<scope>-<YYYY-MM-DD>.html` (e.g. `drata-framework-report-soc-2-2026-08-04.html`) — never a shortened or re-worded variant of the skill name, and never the artifact's display title.** **There is no markdown fallback.** Emitting the report as chat markdown, a bare table, or `###` headings is a failure of the deliverable, not a graceful degradation, and "the host had no artifact tool" is not a reason to do it. The only exception is the explicit text-only opt-out in rule 2. Match effort to the ask — short factual answers stay inline per §4. Two elements of a styled deliverable are a binding contract, even if the brand kit could not be read:
   - **Chart colors: Drata palette only, set explicitly in every chart config — never a library default.** First or single series `#2E4DFF`; multi-series ramp `#BEDAFF` → `#2E4DFF` → `#0F161A`; status tones `#00779C` pass / `#F2C14F` at-risk / `#D53641` fail, only on values that truly pass or fail; one `#FF410C` highlight per view at most; axis and label text `#828B8F`. Every heat map or matrix (risk 5×5, inherent × residual, any coverage grid) uses one band scale: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, at most one worst cell `#FF410C`.
   - **Table hygiene:** one fact per cell — never a chip, code list and number together; never two categories slash-merged into one row. Numeric cells `class="num"`; codes `.code`, never wrapped. Chips mark real pass/fail only — a count like "2 of 5 mapped" stays neutral ink. No Status/severity/health column that only re-buckets a count. Commentary: last column, one sentence, only where it adds signal. **Caps: 6 columns, 15 rows.** Fold rank into the lead cell ("1 · Acme Corp") or a metric pair into `X.X (−Y.Y)`; drop the weakest column rather than cram. Past 15 rows show 15 and close with "13 more — full list on request". **Columns need the theme's `18px` right gutter** — override it to `padding:… 0` and a `.num` column collides with its neighbour, headers merging into `COLUMNACOLUMNB`. Cells are top-aligned. Wrap every table in `<div class="panel">`. **KPI tiles are uniform or they are wrong.** Every tile in a row carries its denominator in the figure — full-size numerator, then `of N` in a muted `<span class="den">`. Always the word `of` — `N of M`. **Never `/`, never `X/Y`, never `N/M`**, anywhere a ratio appears: KPI tiles, bar labels, table cells, body text and headings all use `of`. This is the house standard across every skill; a slash in one report and `of` in the next is the inconsistency this rule exists to prevent. **Never move the denominator into the label** (`Controls not ready (of M)`) — the label names what is counted and nothing else, phrased the same way on every tile. **Every tile in a row counts the same polarity**: choose healthy-of-total or needs-attention-of-total once and hold it across the row, so no reader has to work out that one figure is progress and its neighbour is a problem. A figure with no available denominator does not belong in the KPI row. Never invent a score scale Drata lacks — no 0–100 health score, no weighted total, no points column; rank on real Drata numbers.
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. Every Now · Next · Watch pointer is exactly `<div class="item">…text… <span class="route">drata-x-identify-gaps</span></div>`: class `item`, no bullet element (the theme's `.item::before` paints it and pins it to line one); `item ember` for the single most-critical row only. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Header identity — the customer's logo, top-left, only when it can truly be inlined; else the company name as text.** Call `Drata_getCompany` once per run before rendering (account-scoped, no arguments, read-only; batch it with the run's other independent reads). It returns `name`, `legalName` and `logoUrl`. Resolve the header in this order and stop at the first that succeeds:
     1. **Inlined logo — gate first, then fetch, then verify.** Attempt this step only if `logoUrl` is non-empty **and** the host provides a tool that can actually download raw image bytes from an arbitrary URL. Many sandboxed hosts — including Claude's cloud / Cowork environments — forbid fetching arbitrary CDN URLs, and the Drata image CDN additionally refuses generic fetchers; **in those hosts this step fails immediately and silently, and falling through to the name is the designed outcome, not a degraded render.** Where a download is possible: fetch once (no retries, no proxies, no cache mirrors, never a route around a refusal), verify the bytes decode as a real image (image magic bytes, mime `image/*`, non-zero size), base64-encode **those downloaded bytes with a real encoder in this run**, and emit `<img class="cust" src="data:[mime];base64,[data]" alt="[company name]" onerror="this.style.display='none';this.nextElementSibling.style.display='block'"><div class="custname" style="display:none">[company name]</div>`. **Never type, reconstruct, or approximate base64 from memory — fabricated image data renders a broken or wrong mark exactly where the customer's identity belongs.** If any part of this step cannot be completed and verified, it did not succeed.
     2. **Company name as text.** `<div class="custname">[company name]</div>` — used whenever `logoUrl` is absent or empty, no permitted fetch path exists in this host, the fetch fails or is refused, the bytes are not a decodable image, or the base64 cannot be produced from real downloaded bytes. **This fallback is first-class: a report headed by the company's name in clean type is a correct header; a broken image, an empty header, or invented image data is the only failure.**
     **Never emit `<img src="https://…">`.** A remote reference is not an acceptable third option: artifact sandboxes block external images, and a blocked, expired or access-controlled URL renders a broken-image icon exactly where the customer's identity belongs. It is a verified inlined image or it is the name — nothing in between.
     **When the logo renders, the company name does not appear as visible text** — it lives in the `alt` attribute and in the hidden `onerror` fallback `<div>`, which stays invisible unless the image fails to decode; that hidden div is the safety net, not a second header.
     **Proportions: constrain the height, leave the width free.** `height:32px; width:auto; max-width:200px; object-fit:contain` — a wide wordmark and a square icon then share one baseline with no stretching, squashing or cropping. **Never set `height` and `width` together, never `width:100%`, never a fixed pixel width**, and never re-encode the image to a different aspect ratio. If a logo would exceed `max-width` at 32px tall, `object-fit:contain` shrinks it proportionally — that is correct, do not compensate.
     On the dark board/exec surface, an inlined dark-on-transparent logo disappears; use `<div class="custname" style="color:#fff">[company name]</div>` instead rather than shipping an invisible mark.
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace name>`) → hairline → KPI row → real `<table>` markup → Now · Next · Watch → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there. **The source line always spells the scope out in full** — the workspace's own name, or `All workspaces` for an org roll-up covering more than one. Never omit it, never abbreviate it, never substitute a workspace id or a slug, and never leave it to be inferred from the title.
   - **No opinions, no predictions, no verdicts.** Report what Drata records and what you counted from it. **Never predict what an auditor will ask for, flag, or accept**; never label a gap *critical*, *significant*, *likely finding*, or *high risk* on your own authority; never size effort (S/M/L, hours, weeks) or estimate a date; never declare anything *audit-ready*, *compliant*, *certification-ready*, or *passing*; never interpret what a regulation or clause requires. Drata's own fields — `is_ready`, statuses, scores, dates, counts — are reportable as-is; ordering rows by those real numbers is fine, and a derived figure is labelled *Calculated*. **A reader must be able to act on this report without inheriting a judgement you made up.** If a sentence would not survive an auditor asking "where in Drata does that come from?", cut it.
   - **The artifact title names this skill's job, and no other skill's.** Title it after what this skill produces — an executive report says `Executive Report`, a gap worklist names the gaps it covers. **Never borrow a generic label like `Compliance Briefing`**: two skills wearing one title leaves the reader unable to tell which one they ran, and it collides with any similarly named skill the user has installed. Scope and date follow the title; nothing else does.
   - **Never emit a section you did not populate.** No placeholder heading, no "not included in this run", no "ask and I'll add it" offer, no note explaining which figures were not pulled. Either pull the data and render the section, or leave the section out entirely — a heading whose body apologises for itself costs the reader attention and returns nothing. The only disclosure that stays is a domain the skill *tried* to read and could not (permission denied), which is reported as one line, not a section. **Never explain what Drata does not store.** No "Drata has no asset object", no "there is no review-date field", no "the API does not expose X" — the absence of a field is your constraint while building, never a sentence in the deliverable. Where a field genuinely does not exist, answer with the nearest real Drata data, name it for what it actually is, and label it *Calculated* if you derived it; say nothing about the field you wanted and did not find. The reader came for their compliance posture, not for a tour of the data model.
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

# Evidence Library Gap Identification

## Purpose
Read the whole evidence library once and split it into the buckets someone can actually work:
what has no artifact, what is automated and failing, what needs renewing, and what is
mis-configured. Fixes route to drata-evidence-resolve-gaps. Read-only.

## Which status filters to trust — verified, not assumed
`Drata_listEvidence(statuses=[…])` is **partly reliable**. Each was checked against the payload on a test tenant, so use them exactly this way:

| Status | Verdict | Evidence |
|---|---|---|
| `EXPIRED` | **Use it** | every sampled `renewalDate` in the past — coherent |
| `EXPIRING_SOON` | **Use it** | every `renewalDate` a short window out. **Drata's look-ahead window is not documented in the API and the observed window was narrow, so it is not confirmed to be 30 days.** Never print a day threshold you have not measured — say "Renewal soon" and give the observed range |
| `NEEDS_SOURCE` | **Do not use** | returns items that plainly have a source, including ones with a current `URL` version |
| `NEEDS_ARTIFACT` | **Cross-check, never quote alone** | returned 0; confirm against `versions[]` before reporting a zero |
| `READY` | **Do not use for arithmetic** | READY plus the non-READY statuses exceeds the library total, so the statuses do not partition it |

**Rule: renewal buckets come from `EXPIRED` / `EXPIRING_SOON` filters** (cheap and server-side).
**Everything else is derived from expanded fields**, never from a status. Never publish a count that
depends on `NEEDS_SOURCE` or on statuses summing to the library total.

## Key tools
- `Drata_listEvidence` — the single sweep, `size=500`, `expand=["controls","user","renewalSchemaAndVersions"]`, paged to completion. Source for the derived buckets; also the `EXPIRED` / `EXPIRING_SOON` pulls for renewal.
- `Drata_searchMonitoringTests(check_result_status="FAILED")` — to tell which TEST_RESULT-backed evidence is failing.
- `Drata_searchControls(expand=["frameworkTags"])` — to attach frameworks to the controls an item feeds.

**The fields that matter** (all from the one sweep):

| Field | What it tells you |
|---|---|
| `versions[]` → `type` | `NONE` = no artifact · `URL` / `S3_FILE` = manual · **`TEST_RESULT` = automated**, fed by a monitoring test |
| `versions[]` → `current` | which version counts; no current entry = the evidence item is empty |
| `renewalSchema.renewalDate` | past = overdue · in the `EXPIRING_SOON` window = renewal soon |
| `renewalSchema.renewalScheduleType` | `NONE` = never prompts again, so it goes stale silently |
| `controls[]` | empty = mapped to nothing, counts toward no audit |
| `user` | absent = nobody accountable |
| `description`, `implementationGuidance` | empty = nobody can tell what belongs in the evidence item |

## Workflow
1. **Sweep the library.** `Drata_listEvidence(size=500, expand=["controls","user","renewalSchemaAndVersions"], include_total_count=true)`, paging the cursor to completion. `totalCount` is the denominator. On a multi-workspace tenant pass `workspace_id` — the wrong one returns an empty set, not an error.
2. **Pull the renewal buckets and the joins** — all independent of step 1, so batch them together:
   ```
   Drata_listEvidence(statuses=["EXPIRED"],       size=500, expand=["controls","user"])
   Drata_listEvidence(statuses=["EXPIRING_SOON"], size=500, expand=["controls","user"])
   Drata_searchMonitoringTests(check_result_status="FAILED")
   Drata_searchControls(size=50, expand=["frameworkTags"])   # page to completion
   ```
   Spot-check two rows of each renewal pull against their `renewalDate` before reporting the counts — that is the check that caught `NEEDS_SOURCE`.
3. **Assign each item to one primary bucket (first match wins)** so the four buckets stay disjoint and sum:
   - no current version, or current version `type: "NONE"` → **Needs artifact**
   - current version `type: "TEST_RESULT"` **and** its backing test is FAILED → **Automated evidence failing**
   - in the `EXPIRED` set → **Renewal overdue**; in the `EXPIRING_SOON` set → **Renewal soon** (both confirmed against `renewalDate`). Nothing here is late yet — never label it "due" or "overdue", and never attach a `≤N days` threshold to the tile
   - otherwise → current, no action
4. **Match automated evidence to its test by name.** TEST_RESULT versions carry the test name in `source` (e.g. evidence "Cloud Network Segmentation" ← version source "Cloud Network Segmentation"). Match on that. **Say so in the artifact** — it is a name match, not an ID join, so an unmatched item is reported as unmatched rather than assumed healthy.
5. **Build the framework join once, use it everywhere.** Evidence carries no framework, so join through its controls: `controls[].code` → that control's `frameworkTags`, giving each bucket a framework set. Compute it once from the step-2 control pull and apply it to **any** bucket the user asks to split — Needs artifact gets the table by default. An item feeding two frameworks counts in both, so **a framework split never sums to the bucket total — print that caveat under any table that uses it.** An item with no mapped controls has no framework; it belongs to the no-control callout, not to a framework row.
6. **Compute the two callouts** (diagnostics, not queues — see below).
7. **Give every queue row an owner and one action**, then close Now · Next · Watch.

## Output format
```
## Evidence Library Gap Identification — [workspace] · [date]
**Library: [total] evidence items**

KPI row — four tiles, disjoint, summing with "current" to the library total:
[a] of [total] Needs artifact · [b] of [total] Automated failing · [c] of [total] Renewal overdue · [d] of [total] Renewal soon

### Needs artifact — [a] evidence items
By framework. An item feeding two frameworks appears under both.

| Framework | Evidence items | Controls fed | Next step |
|---|---:|---:|---|
| SOC 2 | [N] | [M] | Attach the artifact in the Drata UI (no MCP upload), or link a URL |
*Framework counts overlap and do not sum to [a] — an item can serve several frameworks.*

### Automated evidence failing — [b] evidence items
Backed by a monitoring test whose last result is FAILED — the evidence item is not stale, its source is broken.

| Evidence | Test (source name) | Controls fed | Next step |
|---|---|---:|---|
| Cloud Network Segmentation | Cloud Network Segmentation | [N] | Fix the integration → drata-monitoring-report |
*Matched on version source name, not a test ID. [n] TEST_RESULT evidence items could not be matched to a test — listed, not assumed healthy.*

### Renewal — [c] overdue · [d] renewal soon
| Evidence | Renewal date | Days | Owner | Next step |
|---|---|---:|---|---|
| [name] | [date] | [N] overdue | [owner / —] | Re-file and re-date → drata-evidence-resolve-gaps |

### Callout — evidence items mapped to no control
[n] evidence items feed no control, so they count toward no audit and no framework. Either map them
(→ drata-evidence-resolve-gaps) or retire them.

### Callout — key fields missing
Diagnostics, not a queue. **These overlap each other and the buckets above — they do not sum.** The
empty-item case is not repeated here; it is the Needs artifact bucket.

**Scope the guidance check to manual evidence items only.** A `TEST_RESULT`-backed item is refreshed by
its monitoring test, not by a person, so implementation guidance is not expected and its absence is
not a finding — counting automated items here inflates the number with items nobody should act on.
Exclude them, and say the count is manual-only. The other three checks apply to every evidence item.
> **No owner** — [n] evidence items · nobody accountable for refreshing them
> **No renewal schedule** (`NONE`) — [n] evidence items · will never prompt again, so they go stale silently
> **No description** — [n] evidence items · an auditor cannot tell what the item is meant to prove
> **No implementation guidance** — [n] **manual** evidence items · the next person collecting it has no instructions

### Now · Next · Watch
**Now** — automated failing evidence items under a not-ready control: the source is broken, not the paperwork. → drata-monitoring-report
**Next** — needs-artifact evidence items in the framework you are auditing against. → drata-evidence-resolve-gaps
**Watch** — buckets with no renewal schedule: nothing is wrong today and nothing will ever flag them.
```

## What drata-evidence-resolve-gaps must be able to fix
Every bucket here has a write path in **drata-evidence-resolve-gaps**, with two hard limits that belong in
both skills:

| Gap finding | Fix | Limit |
|---|---|---|
| Needs artifact | `url` or `ticket_url` on `Drata_updateEvidence` | **No file upload exists in the MCP** — an actual file must be attached in the Drata UI. Never promise an upload |
| Automated failing | none here — fix the integration | drata-evidence-resolve-gaps cannot re-run or repair a test |
| Renewal overdue / due | `renewal_schedule_type` (+ `renewal_date` for `CUSTOM`), `filed_at` | Adding a new artifact **requires** a renewal schedule — ask, never default it |
| No control mapped | `control_codes` | **`control_codes` REPLACES the whole mapping.** Read current codes via `Drata_listEvidence(expand=["controls"])` and send the union, or the other mappings are silently dropped |
| No owner | `owner` | Pass the name verbatim; never fabricate an email |
| No description / guidance | `description`, `implementation_guidance` | — |

## Edge cases
| Situation | Handling |
|---|---|
| Tempted to use `statuses` | Only `EXPIRED` and `EXPIRING_SOON` are verified. `NEEDS_SOURCE` is wrong, `NEEDS_ARTIFACT` needs a cross-check, and the statuses do not partition the library |
| Buckets do not sum to the library total | Recount before rendering; the four buckets plus "current" must equal `totalCount` |
| Augmented filter used (`control_code`, `control_id`, `owner`) | Returns **no `totalCount`** and scans ≤10 pages — never quote a count from it |
| "Find the SOC 2 pen-test evidence" | `name` is a case-insensitive **PREFIX** match, not substring — sweep and filter client-side |
| A TEST_RESULT bucket matches no failing test | Report it as unmatched in the automated section — never assume it is healthy |
| Framework split questioned | It overlaps by design; say so rather than forcing items into one framework |
| Another bucket needs a framework split | The join from step 5 is already computed — reuse it, same overlap caveat. Never re-pull controls per bucket |
| "Upload the missing artifact" | Impossible via MCP — Drata UI, Evidence Library → New. Offer to create the record or attach a URL instead |
| Multiple workspaces | Pass the same `workspace_id` to every call; a mismatch returns empty sets that look like a clean library |

## Example invocations
- "Drata, what's broken in our evidence library?"
- "Which evidence buckets have no artifact, by framework?"
- "Which automated evidence is failing?"
- "What evidence renews in the next 30 days?"
- "Which evidence buckets aren't mapped to any control?"
- "Which evidence has no owner or no renewal schedule?"
