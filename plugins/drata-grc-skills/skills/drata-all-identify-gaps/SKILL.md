---
name: drata-all-identify-gaps
description: >
  The 30-second cross-domain briefing: everything needing attention now — failing controls and
  tests, high open risks, expired evidence, overdue vendor reviews, non-compliant personnel — as
  counts plus Now/Next/Watch pointers routed to the right identify-gaps skill. Use for 'what needs
  attention today', 'what's broken in Drata', or a daily check-in. Breadth only; each domain
  identify-gaps skill owns the ranked deep dive. Read-only.
area: Start Here
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_searchControls, Drata_searchMonitoringTests, Drata_searchRisks, Drata_listVendors, Drata_listEvidence, Drata_searchPersonnelCompliance, Drata_listWorkspaces"
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Rendering — branded, always. This is the default; never ask the user to pick an output mode.** Read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` and render every substantive deliverable (dashboard, report, briefing, gap worklist) in Drata branding: a self-contained HTML document using its §3 `.drata` theme. **Deliver it as HTML, always.** If the host has an artifact tool, render it there. If it does not, **write the complete HTML to a `.html` file and send that file** — every environment this runs in can deliver a file. **There is no markdown fallback.** Emitting the report as chat markdown, a bare table, or `###` headings is a failure of the deliverable, not a graceful degradation, and "the host had no artifact tool" is not a reason to do it. The only exception is the explicit text-only opt-out in rule 2. Match effort to the ask — short factual answers stay inline per §4. Two elements of a styled deliverable are a binding contract, even if the brand kit could not be read:
   - **Chart colors: Drata palette only, set explicitly in every chart config — never a library default.** First or single series `#2E4DFF`; multi-series ramp `#BEDAFF` → `#2E4DFF` → `#0F161A`; status tones `#00779C` pass / `#F2C14F` at-risk / `#D53641` fail, only on values that truly pass or fail; one `#FF410C` highlight per view at most; axis and label text `#828B8F`. Every heat map or matrix (risk 5×5, inherent × residual, any coverage grid) uses one band scale: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, at most one worst cell `#FF410C`.
   - **Table hygiene:** one fact per cell — never a chip, code list and number together; never two categories slash-merged into one row. Numeric cells `class="num"`; codes `.code`, never wrapped. Chips mark real pass/fail only — a count like "2 of 5 mapped" stays neutral ink. No Status/severity/health column that only re-buckets a count. Commentary: last column, one sentence, only where it adds signal. **Caps: 6 columns, 15 rows.** Fold rank into the lead cell ("1 · Acme") or a metric pair into `9.1 (−7.3)`; drop the weakest column rather than cram. Past 15 rows show 15 and close with "13 more — full list on request". **Columns need the theme's `18px` right gutter** — override it to `padding:… 0` and a `.num` column collides with its neighbour, headers merging into `INTEGRATIONCONTROLSCODES`. Cells are top-aligned. Wrap every table in `<div class="panel">`. **KPI tiles are uniform or they are wrong.** Every tile in a row carries its denominator in the figure — full-size numerator, then `of N` in a muted `<span class="den">`. Always the word `of` — `55 of 241`. **Never `/`, never `X/Y`, never `55/241`**, anywhere a ratio appears: KPI tiles, bar labels, table cells, body text and headings all use `of`. This is the house standard across every skill; a slash in one report and `of` in the next is the inconsistency this rule exists to prevent. **Never move the denominator into the label** (`Controls not ready (of 622)`) — the label names what is counted and nothing else, phrased the same way on every tile. **Every tile in a row counts the same polarity**: choose healthy-of-total or needs-attention-of-total once and hold it across the row, so no reader has to work out that one figure is progress and its neighbour is a problem. A figure with no available denominator does not belong in the KPI row. Never invent a score scale Drata lacks — no 0–100 health score, no weighted total, no points column; rank on real Drata numbers.
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. Every Now · Next · Watch pointer is exactly `<div class="item">…text… <span class="route">drata-x-identify-gaps</span></div>`: class `item`, no bullet element (the theme's `.item::before` paints it and pins it to line one); `item ember` for the single most-critical row only. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace>`) → hairline → KPI row → real `<table>` markup → Now · Next · Watch → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there.
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

# All-Domain Gap Identification

## Purpose
**The cross-domain gap scan** — the one place to see everything needing attention now across
controls, monitoring, risk, evidence, vendors and personnel. It **ranks and routes; it does not
treat.** One prompt replaces the daily "log into Drata and click around" habit and returns
(1) a posture snapshot, (2) everything **already past a date someone committed to**, and (3) a
short *Now · Next · Watch* **pointer** list. Read-only. A 30-second briefing, not an audit: it
counts, it does not paginate (sole exception: the step-4 `hasTicket` scan).

**Every pointer carries a named next action and an owner — and here the owner is the domain that
owns the depth**, not a person. That satisfies the identify-gaps contract: the line says what to do next
and hands it to the domain identify-gaps skill holding the per-item owners, days-overdue detail and
ranked queue. Max three pointers per bucket, one line each; owner tables and ranked queues belong
to those skills, not here.

## Key tools
- `Drata_searchControls` — not-ready controls (`is_ready=false`); and `ticket_status='IN_PROGRESS'` for work already in flight.
- `Drata_searchMonitoringTests` — failed checks (`check_result_status="FAILED"`).
- `Drata_searchRisks` — open high-severity risks, and treatments past `anticipated_completion_date`.
- `Drata_listVendors` — Drata's own `action_required` and `next_review_deadline` flags.
- `Drata_listEvidence` — `EXPIRED` / `EXPIRING_SOON` items; `size` up to 500, so one call sweeps the library.
- `Drata_searchPersonnelCompliance` — per-check failure counts (`identity_mfa`, `accepted_policies`, `security_training`, `bg_check`, `offboarding`) via `facets` + `size=1`, so no rows are fetched. **Pass `employment_status` explicitly**; read back `_appliedEmploymentStatus`.
- `Drata_listWorkspaces` — scope; if several exist, brief the primary and note the rest.

## Workflow
1. **Resolve scope.** `Drata_listWorkspaces()`. One workspace → use it. Several → default to the primary, note the others at the bottom; if the user named one, use it. Vendors are account-scoped — never ask which workspace for them.
2. **Gather current failures in parallel** (structured filters, never semantic `query`, so counts are exact):
   ```
   Drata_searchControls(is_ready=false, size=50, expand=["flags"])   # expand flags → hasTicket per control (the only reliable ticket signal); page 1 of the step-4 scan — continue its cursor there, never re-issue the pull
   Drata_searchMonitoringTests(check_result_status="FAILED", size=25)
   Drata_searchRisks(status=["ACTIVE"], inherent_score_gte=7, sort="inherentScore",
     sort_dir="desc", size=15)
   Drata_searchPersonnelCompliance(facets=["identity_mfa","accepted_policies",
     "security_training","bg_check","offboarding"], size=1,
     employment_status=["CURRENT_EMPLOYEE","CURRENT_CONTRACTOR"])   # pass explicitly — see 2a
   ```
   **2a. Never let the personnel population default.** Omitting `employment_status` **silently
   restricts to CURRENT_EMPLOYEE + CURRENT_CONTRACTOR**. Pass the list explicitly (add
   `SPECIAL_FORMER_EMPLOYEE` / `SPECIAL_FORMER_CONTRACTOR` for an offboarding read), read back
   `_appliedEmploymentStatus`, and label that population under the figure. Facets are zero-filled,
   so "0 failing" is distinguishable from "not measured"; count each person once, not per check.
   **The under-table note states only the population denominator — never append a per-check
   breakdown (MFA / policies / training / BG) here.** Those per-check counts overlap (a person
   fails several checks, so they will not sum to the headline "N of M") and the per-check depth is
   drata-personnel-report's deliverable, not this snapshot's.
3. **Gather commitments by date — server-side filters, same parallel batch.** Split into **overdue** (already past a committed date) and **due soon** (a committed date approaching but not yet passed), and keep them in separate groups — due-soon items are NOT overdue. Do not compute a combined "overdue commitments" total or headline number:
   ```
   Drata_searchRisks(anticipated_completion_date_lte="<today, YYYY-MM-DD>",
     status=["ACTIVE"], size=25)        # treatments past their committed date
   Drata_listVendors(action_required=True, size=50)
   Drata_listVendors(next_review_deadline=["OVERDUE","DUE_SOON"], size=50)
   Drata_listEvidence(statuses=["EXPIRED","EXPIRING_SOON"], size=500, include_total_count=true)
   # No ticket_status filter — it does NOT filter to ticketed controls in this MCP (returns ~all controls, e.g. 401/402). Ticket status = per-control flags.hasTicket, from the step-2 expand=["flags"] fetch.
   ```
   Union the two vendor calls and dedupe by vendor ID before counting vendors once.
4. **Surface not-ready controls with no ticket yet — the actionable net-new work.** The only trustworthy ticket signal is the per-control `flags.hasTicket` boolean (from `expand=["flags"]`); **the `ticket_status` filter is broken in this MCP — it returns nearly all controls regardless of tickets (verified: 401 of 402), so never derive a count from it.** Page the `is_ready=false` set with `expand=["flags"]`, `size=50`, following the cursor **until pagination is exhausted** (for ~167 not-ready controls that is ~4 pages — a partial scan is not acceptable), and count `hasTicket=false` = the exact number of not-ready controls with **no remediation ticket yet** (call it **M**). Show **M as a single exact count** — never a ratio, never the word "sampled," never a "(sampled)" suffix. **If M = 0 (every not-ready control already has a ticket — 100% coverage), omit the row entirely: there is no gap to flag.** Only if the not-ready set exceeds 300 controls may you stop early — then show `≥M (first 300 scanned)`. When ranking Now/Next, deprioritise controls whose `flags.hasTicket=true` (already ticketed) in favour of the un-ticketed ones — never re-recommend ticketed work.
5. **Normalize the rest.** A failed test causing a not-ready control is one issue, not two — link them. Expired evidence under a not-ready control likewise. A person failing three checks is one person, not three.
6. **Read counts, don't paginate** (sole exception: the step-4 `hasTicket` scan). Headlines come from `pagination.totalCount`, or the facet buckets for personnel. Detail rows are capped: 50 controls, 25 tests, 15 risks, 50 vendors per flag, 500 evidence items; **personnel is counts only — `size=1`, no rows at all**. **State the caps in the output** so nobody mistakes a truncated list for the whole picture.
7. **Rank, then cut to three per bucket.** Order by: overdue-by-days → framework/audit impact → security sensitivity (access, encryption, incident response) → breadth → actionability. Anything past a committed date outranks the same item without one. Keep **at most three** in Now, three in Next, three in Watch — a pointer list, not a queue. No per-item owner or days-overdue table; that is the identify-gaps skills' deliverable.
8. **Emit the briefing + pointers** below. Label counts from `totalCount` / facets as *Tool Calls*; anything inferred (linkage, ranking) as *Calculated*. **Do NOT render an "Overdue commitments" aggregate or KPI card — there is no summed overdue number anywhere. Overdue items are listed individually under the "Overdue — past a committed date" group and due-soon items under "Due soon — not yet overdue." KPI cards, if used, show only the primary counts — controls not ready, monitoring tests failing, open high-severity risks, personnel failing 1+ check — never an overdue or due-soon total.**
9. **Route depth onward.** This skill never drills in. Every pointer line ends with the domain skill that owns the depth — the domain identify-gaps skill where one exists; for personnel and monitoring, drata-personnel-report / drata-monitoring-report — and the briefing closes with the routing line.

## Output format
```
## All-Domain Gap Identification — [workspace] · [date]
**Workspace:** [name]   **Pulled:** [timestamp]   **Caps:** counts complete (totalCount / facets);
rows sampled for ranking = first 50 controls / 25 tests / 15 risks / 50 vendors per flag /
500 evidence items; personnel = faceted counts only, no rows. Max 3 pointers per bucket.

### Posture at a glance
| Signal | Count |
|---|---|
| Controls not ready | [N] |
| Monitoring tests failing | [N] |
| Open high-severity risks (≥7) | [N] |
| Personnel failing 1+ compliance check | [N] |
| **Overdue — past a committed date** |  |
| — Risk treatments past completion date | [N] |
| — Vendor reviews overdue | [N] |
| — Evidence expired | [N] |
| **Due soon — not yet overdue** |  |
| — Vendor reviews due soon | [N] |
| — Evidence expiring soon | [N] |
| Not-ready controls with no ticket yet | [M] |

*Personnel figure covers [N] of [M] [_appliedEmploymentStatus, spelled out] — the population behind that count.*

### Where to look   (pointers — max 3 per bucket, one line each)
**Now (today)**
> [what, in plain language] — [count] · why it's first → **[domain owner skill]**

**Next (this week)**
> [what] — [count] · why it matters → **[domain owner skill]**

**Watch (no action yet)**
> [what] — [count] · the date it turns into work → **[domain owner skill]**
```

## Edge cases
| Situation | Handling |
|---|---|
| Nothing failing in a section | "All clear" and skip it — but still run the four overdue filters; overdue work hides behind a green board |
| API returns empty / errors | Note inline, keep the rest of the briefing |
| Multiple workspaces | Brief primary; list others at the bottom; offer the multi-workspace view (→ drata-executive-report) |
| User asks for a formal / board report | Route to drata-executive-report |
| User wants the full list, not the top N | Say the cap, then route to the matching identify-gaps skill — that is what they are for |
| Tempted to emit a ranked worklist here | Don't. This skill is the 30-second orientation: snapshot + at most three one-line pointers per bucket. Per-item owners, days-overdue and next actions are the identify-gaps skills' deliverable |
| No `employment_status` passed to the personnel call | Silently limited to CURRENT_EMPLOYEE + CURRENT_CONTRACTOR — pass it explicitly, read back `_appliedEmploymentStatus`, and never print the personnel count without its population |
| Same item in two dimensions (expired evidence under a not-ready control) | Count once, show under the more urgent dimension, cross-reference the other |
| Every not-ready control already ticketed (M = 0) | Omit the "no ticket yet" row entirely — 100% ticket coverage is not a gap |
| Control has an open ticket | Excluded from Now/Next by design — list it under *Already in flight* with the count, never re-recommend it |
| "How long has this been overdue / what's the trend?" | Days overdue are derivable from the date fields; **trends are not** — no historical snapshots exist in the MCP. Say so rather than inventing one |

## Example invocations
- "Drata, give me my compliance status."
- "What's broken in Drata today and what should I fix first?"
- "Morning GRC briefing."
- "What's overdue in Drata right now?"
- "Which risk treatments blew past their completion date, and which vendor reviews are late?"
- "Is anyone out of compliance on training or MFA?"
- "Where should the compliance team focus this week — skip anything already being worked on?"
