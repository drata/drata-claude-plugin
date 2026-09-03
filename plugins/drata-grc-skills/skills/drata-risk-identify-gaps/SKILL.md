---
name: drata-risk-identify-gaps
description: >
  The risk action queue: only risks where the record is incomplete or contradictory and someone
  owes an edit — untreated, unowned, unscored, no residual, no reduction, residual above
  inherent, overdue or undated treatments. Risks already assessed and treated are counted, not listed. Use for
  'which risks need attention', untreated or unowned risks, overdue treatments. Scores, heat map and
  breakdowns → drata-risk-report; writes → drata-risk-resolve-gaps. Read-only.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listRiskRegisters, Drata_searchRisks, Drata_searchControls"
metadata:
  area: Risk Management
  permission: read-only
---

**SCOPE IS THE RISK REGISTER — NEVER THE WORKSPACE. Resolve it first, before any other call.**

Risks live in **risk registers**, not workspaces. Call `Drata_listRiskRegisters()` as the first
action of every run and scope everything that follows to the register id you get back.

- **One register → select it silently.** Never ask, never mention that a choice existed. A picker
  with a single option teaches the user this skill does not know their account.
- **Several, and the user named one → use it and say nothing.** Other registers existing is not
  ambiguity.
- **Several, and the user named none → ask which register**, listing them by name. This is the only
  question this skill opens with, and it is about registers.
- **Never ask the user to pick a workspace, and never resolve one to reach risks.** Registers are
  not workspaces: a register may map to several, to none, or carry no workspace association at all.
  Asking for a workspace here sends the user to a scope that does not select risks, and picking one
  yourself silently narrows or misses the register they meant.
- **Never infer a register from a similarly-named workspace.** Register and workspace names are
  chosen independently; a name that looks like a match is a coincidence, not a mapping. If a place
  name cannot be resolved to a register, ask — do not pattern-match it.

The register name is what labels the output scope. Where the shared structure rule says
`<workspace>` in the source line, this skill writes the **register name(s)** instead.

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
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace name> · <register name>`) → hairline → KPI row → real `<table>` markup → Now · Next · Watch → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there. **The source line always spells the scope out in full** — the workspace's own name, or `All workspaces` for an org roll-up covering more than one. Never omit it, never abbreviate it, never substitute a workspace id or a slug, and never leave it to be inferred from the title.
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

# Risk Prioritization

## Purpose
Give a risk manager the short list of risks that **need work on the record right now**, and nothing
else. A risk is in the queue only when a field is missing, expired or self-contradictory — no
treatment plan, no owner, no score, no residual, a residual that shows no reduction, or a committed
date that has passed or was never set. Each row names the one edit that
clears it. **Staleness is not one of the flags** — there is no review-date field and "untouched for
months" is not a gap Drata records; see the rule below before you reach for last-edited.

**Risks that raise none of the eight flags do not appear, however large their score.** They are
counted in a single line so the reader can see the queue is the exception rather than the register
re-printed. Ranking, heat maps and score breakdowns belong to drata-risk-report.

`anticipated_completion_date` is the *treatment's* committed completion date, never a review date.
Read-only: it flags but never changes a risk (→ drata-risk-resolve-gaps for writes).

## Key tools
- `Drata_listRiskRegisters` — resolve the target register. **Never infer a register from a similarly-named workspace.**
- `Drata_searchRisks` — the ranking engine; use **structured filters** (`status[]` — the enum is `ACTIVE | ARCHIVED | CLOSED`; there is no `OPEN`, and passing it returns an empty set with no error — `inherent_score_gte`/`_lte`, `residual_score_gte`/`_lte`, `treatment_plan`, `owners`, `reviewers`, `vendor_names`, `anticipated_completion_date`, `type`, `categories`), `sort` + `sort_dir`, `expand` controls, `facets`.
- `Drata_searchControls` — linked-control readiness for the top-ranked risks.

## Workflow
1. **Resolve the register.** `Drata_listRiskRegisters()`; match the user's register explicitly and capture `risk_register_id`. **One register → never ask**; capture its id and proceed silently.
2. **Pull every open risk with the fields the flags need — paginate to the end.** The queue is
   computed from `owners`, `impact`, `likelihood`, `score`, `residualScore`, `treatmentPlan`,
   `anticipatedCompletionDate`, `completionDate` and `updatedAt`, so **`expand=["owners"]` is
   required** — without it every risk looks unowned. Structured filters, never semantic `query`:
   ```
   Drata_searchRisks(risk_register_id="…", status=["ACTIVE"],
     sort="inherentScore", sort_dir="desc", expand=["owners","controls"])
   ```
   Read `pagination.totalCount` for the open-risk denominator.

   **`searchRisks` sorts by `residualScore DESC` unless you say otherwise, and a page is not a
   population.** Two rules follow, and breaking either one produces numbers that look plausible and
   are wrong:

   - **Never count one field off a ranking by another.** Counting inherent-score rows out of a
     residual-sorted page drops exactly the risks whose treatment reduced residual, or whose
     residual was never scored (`residualScore: null` sorts to the bottom) — so a risk disappears
     from an inherent-severity count *because* its mitigation worked. If a figure is about inherent
     score, sort by `inherentScore` or filter with `inherent_score_gte` and read `totalCount`.
   - **Never count anything off a *ranked or truncated* page.** Every count that a server-side filter
     can produce — per score band, per treatment plan, per owner, overdue — comes from its own
     filtered call's `totalCount` or from a `facets` result. The ranked page exists to be *displayed*,
     never to be *tallied*.
   - **Some flags have no server-side filter, and those must be computed from rows — say so.**
     No filter exists for "no owner" (`owners` empty), "not scored" (`impact` or `likelihood` null),
     "residual equals inherent", "residual above inherent", or "no `anticipatedCompletionDate` at
     all". The only honest way to those figures is a **complete** pass over the active register —
     page the cursor to exhaustion, then count client-side — and the figure is labelled *Calculated*,
     never presented with `totalCount` authority. **A group count derived from one page is the defect
     this section exists to prevent:** if you cannot finish the pass, say how far you got and give no
     number. The rule above forbids tallying a *partial* page, not counting a *complete* set.

   The same page discipline applies to identity. When you write a row, copy its `riskId` from that
   row — **never from another row on the page**. Ids that share a prefix and sit near each other in
   a ranking are the easiest thing in this skill to cross-wire, and the result is a live link to the
   wrong risk, which reads as authoritative and is not.
3. **Pull control readiness for the top N — one call, then intersect locally.** `Drata_searchControls(is_ready=False, size=50)`, paginating on `pagination.totalCount`; then match each risk's `expand=["controls"]` codes against that set to see whether the mitigation is actually ready. Do **not** call `Drata_searchControls(query="<code>")` per control: `query` switches to a semantic endpoint that ignores every filter, so it returns wording-similar controls rather than the one you asked for.
4. **Flag the record defects — this is the queue.** Apply the eight detections in *What belongs in
   the queue* below to every open risk and keep only the risks carrying at least one. Count the
   clean ones; never list them. **A high score with a complete record is not a queue item.**
4a. **Order by Drata's own scores within each flag group. Do not invent a severity vocabulary.** `impact`, `likelihood`,
   `score` (inherent) and `residualScore` are real fields — rank on them and say so. **Never label a
   risk `Critical`, `High`, `Medium`, `Low`, `severe`, or `material`**: Drata stores no severity band
   on a risk, so any such word is your judgement wearing the register's authority, and a reader who
   escalates on it is acting on something no Drata field supports. Head sections by the real threshold instead — `Inherent score ≥ [N]`, `Residual [N]` — and label any derived figure
   *Calculated* per `${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`. The same applies in
   prose: describe what the record says (`no residual reduction recorded`, `committed date passed`),
   never what you think it means for the business (`carries regulatory exposure`, `customer-data
   exposure`, `material finding`).
5. **Flag separately:** **untreated** (`treatment_plan` is the enum value `UNTREATED` — it is never
   empty or null, so filter on the enum) and **treatment past its committed completion date**. These
   jump the queue regardless of score. `anticipated_completion_date` is the treatment's committed
   completion date — **not** a review date; the MCP has no risk review-date field, so never label it
   one.

   **Both counts come from their own filtered call with `totalCount` — never from tallying the rows
   you happened to fetch.** The overdue set in particular is far larger than the high-score set, so
   counting it off the ranked page undercounts it badly:
   ```
   Drata_searchRisks(risk_register_id=…, status=["ACTIVE"], treatment_plan=["UNTREATED"], size=1)
   Drata_searchRisks(risk_register_id=…, status=["ACTIVE"],
     anticipated_completion_date_lte="<today>", size=50, sort="anticipatedCompletionDate")
   ```
   **`anticipated_completion_date_lte` alone is not "overdue"** — it is wrong on three counts, and the
   call's raw `totalCount` must never be published as the overdue figure.
   (1) A risk can be ACTIVE, carry a past committed date, and already have a `completionDate` recorded
   — the treatment landed, the record was never closed. **Drop every row with a non-null
   `completionDate`** before you count.
   (2) **The call carries no `treatment_plan` filter, so its count includes ACCEPT and UNTREATED
   rows** that the overdue flag explicitly excludes: an accepted risk has no committed delivery date
   to miss, and an untreated one is counted under *Needs a decision* instead. Keep only
   MITIGATE / TRANSFER / AVOID.
   (3) `_lte` includes items committed for **today**, which are not yet past their date. Use `_lt`, or
   pass yesterday.
   State the overdue figure as the count that survives all three. Paginate the call to the end; the
   default page will not hold the whole set.
5a. **Owner concentration comes from the `ownerEmails` facet, never from counting names you can see.**
   `facets=["ownerEmails","treatmentPlan","status"]` with `size=1` returns the whole register's
   distribution in one call. The rows in your queue are the top of the register, not the register —
   counting owners across them understates every owner and silently omits anyone whose risks all sit
   below your cut. Report the facet's own numbers, and name the top owners *by that facet's order*.
5b. **Never transcribe a risk id you did not read from the API response.** `riskId` is the reader's
   only handle on the record — a wrong code sends them to a risk that does not exist or, worse, to a
   different one. Copy it verbatim from the row you are describing, and copy the title from `title`
   rather than paraphrasing it into something unsearchable. **Never append a bare `*`, `†`, or any
   other marker to an id**: some Drata titles legitimately contain an asterisk, so a stray one reads
   as part of the record. If a row needs a caveat, write the caveat in a cell — a footnote symbol
   with no footnote is a defect.
6. **Mark vendor-carried rows in place — never break them into their own section.** A vendor risk
   that needs work is already in one of the three groups; listing it again under a second heading
   is the same row twice, and a queue whose rows repeat is a breakdown again. Add the vendor name as
   a marker on the row instead, so it is visible where the work is.
   **Identify vendors by the `vendorNames` facet or `type=["EXTERNAL"]`, never by a `VR-` prefix on
   the riskId.** The prefix is a naming convention and does not track the field: on some registers the two disagree, so prefix-matching both misses vendor risks and mislabels internal ones.
7. **Apply the persona mode** (below) to shape the output.

## Output format
```
## Risk Prioritization — [register]
**Pulled:** [timestamp]   **Open risks:** [totalCount]   **risk_register_id:** [id]

### Needs a decision        [n] risks
### Needs a number          [n] risks
### Needs a date            [n] risks
### No flags                [n] risks — counted, not listed

### Now · Next · Watch
**Now** — the Needs-a-decision rows. → drata-risk-resolve-gaps
**Next** — Needs-a-number, then Needs-a-date. → drata-risk-resolve-gaps
**Watch** — flagged risks whose linked controls are not ready → drata-control-identify-gaps · vendor-carried rows → drata-vendor-identify-gaps.
```

## What belongs in the queue
**A queue lists risks where a named person owes a specific edit to the record. Nothing else.**

**A high inherent score is not a queue item.** A risk that is scored, owned, has a treatment plan, a
recorded residual and a committed date in the future raises none of these flags — the organisation
looked at it and recorded a decision. Listing it again because the number is large tells the risk manager something they
already know and gives them nothing to do. **Never rank the queue by score, and never open with a
"top risks by score" table** — that is drata-risk-report's job, and duplicating it here is what turns
a queue into a breakdown. Score is a *sort key within* a flag group, never a reason to appear.

**The eight flags — each is a missing or contradictory field with one obvious fix:**

| Flag | Detection | What is owed |
|---|---|---|
| Untreated | `treatmentPlan == "UNTREATED"` | Choose a treatment plan |
| No owner | `owners.totalCount == 0` | Assign an owner |
| Not scored | `impact` or `likelihood` null | Assess impact × likelihood |
| No residual | `treatmentPlan == "MITIGATE"` and `residualScore` null | Record the residual |
| No reduction | `treatmentPlan == "MITIGATE"` and `residualScore == score` | Confirm the treatment works, or re-score |
| Residual above inherent | `residualScore > score` | A data error — one of the two scores is wrong |
| Overdue | `treatmentPlan` in MITIGATE/TRANSFER/AVOID, `anticipatedCompletionDate` past **and** `completionDate` null | Re-commit to a date, or close it |
| No target date | `treatmentPlan == "MITIGATE"` and no `anticipatedCompletionDate` | Set a committed date |


Group them under the three headings by what the fix actually is — **Needs a decision** (untreated,
no owner, residual above inherent), **Needs a number** (not scored, no residual, no reduction),
**Needs a date** (overdue, no target date).

**Most flagged risks carry more than one flag, so group assignment must be deterministic**: a risk
goes to *Needs a decision* if it carries any decision flag, else *Needs a number* if it carries any
number flag, else *Needs a date*. Decision beats number beats date, always. It appears **once**, in
that group only, and its row lists every flag it carries so nothing is hidden by the grouping.

**There is no "stale" or "overdue review" flag, and never invent one.** Drata stores no risk
review-date field and no review cycle — `updatedAt` is only *when the record was last edited*, by
anyone, for any reason. A row saying a risk is "past review" or "stale" asserts a review schedule
the register does not hold, which is exactly the kind of made-up status a reader would act on.

If the user asks about review currency, answer with `updatedAt` and call it what it is — **last
edited on <date>** — say that Drata carries no review-date field, and stop there. **Never turn
last-edited into a flag, a KPI tile, or a queue row.** Registers are commonly reviewed in batches,
so an edit-recency cutoff mostly measures where the last batch fell, not neglect: on a register
reviewed each quarter, a six-month cutoff flags roughly half the rows the day before the next batch
and none the day after, with nothing about the risks having changed.

**Every flag is scoped by treatment plan, and the scope is the rule — not a refinement of it.**
An ACCEPT risk is a decision to carry the risk as it stands, so on ACCEPT rows: a residual equal to
inherent is the correct record, a missing residual is expected, and a past completion date is not an
overdue treatment because no treatment was ever committed. Applying the MITIGATE-shaped checks to
ACCEPT rows fills the queue with risks whose records are exactly right, which is the fastest way to
teach a risk manager to stop reading it. **Only `untreated`, `no owner`, `not scored` and
`residual above inherent` apply to every plan** — the rest are scoped in the table above, and the
scope is not optional.

**Sort within each group by inherent score, highest first**, so the flagged risks that matter most
sit at the top of their group. That is the only place score enters the queue.

**Close with the unflagged count** — `N of M open risks carry no flag` — and do not list them.
That line is what tells the reader the queue is the exception rather than the register re-printed,
and it is the number that shrinks as they work.

**Say "carries no flag", never "outstanding", "clean", "healthy", "complete" or "done".**
`outstanding` is not a Drata field and not a Drata concept — it reads as a status the register holds
and it does not exist. Every word in this report either names a real field (`treatmentPlan`,
`residualScore`, `anticipatedCompletionDate`) or names one of the eight flags defined here. A risk
with no flag is exactly that: **no flag was raised by these eight checks** — which is not a statement
that the risk is well managed, and must never be written as one.

### The two units, and the arithmetic that must hold
**A risk is not a flag hit. Never put both in one row, one chart, or one KPI tile without labelling
which is which** — this is the single easiest way to make a correct report look broken.

- **Risks** partition. Every open risk lands in exactly one group, or in the no-flag count:
  `decision + number + date + no-flag = open risks`. **Compute that sum and check it equals
  `totalCount` before shipping.** If it does not, a risk was double-counted or dropped.
- **Flag hits** do not partition and always exceed the risk count, because most flagged risks carry
  several flags at once. A per-flag breakdown is a legitimate second view — but it is measured in
  flag hits, its total is larger than the number of risks, and **it must say so in its own subtitle**
  (`N flags across M risks — most risks carry more than one`).

Never present the per-flag numbers and the group numbers adjacent without that sentence. A reader seeing `[N] overdue` beside `[M] needs a date` reasonably concludes the report is broken; it is not,
they are counting different things, and it is your job to say which.

**If every risk is clean, say so in one line and render no table.** An empty queue is a valid,
useful answer; do not pad it with a score ranking to fill the page.

**The KPI row is exactly the three groups**, each `n of [open risks]`: Needs a decision · Needs a
number · Needs a date. **No total-flagged tile** — it is the sum of the other three and spends a
quarter of the row restating them. **No "highest inherent score" tile** — it names no action and
belongs to drata-risk-report. **No tile for a single flag**: the three groups are disjoint and a
per-flag tile beside them mixes risks with flag hits, which is the confusion the units rule exists
to prevent.

**Every KPI tile must reconcile with the section beneath it, and you must check that before you
ship.** If a tile says four risks need a decision, exactly four rows sit under that heading. The
*Needs a date* group tile is the overdue-and-undated group, not a per-flag tile — its figure is the
`totalCount` of the overdue call **after** the three corrections in step 3, not the raw response
count and not the number of rows you chose to display; where the table shows fewer, say how many were
not drawn. **A tile whose figure contradicts its own
table is the single most damaging defect this skill can ship** — it is the first number a reader
quotes onward and the last one they check. Re-derive every tile from the data you rendered, not from
memory of an earlier call.

**Every tile carries a denominator**, per the shared KPI rule, and every tile in the row counts the
same polarity. The open-risk population is the source line's job (`Open risks: [N]`), not a bare
figure in the KPI row — a tile reading `[N] / Open risks` next to three problem tiles makes the
reader work out which figures are problems and which is the base.

**Table hygiene, applied to this skill's recurring cells:** the treatment cell carries the
`treatment_plan` enum and nothing else — commentary about it goes in its own column or is cut.
The commit-date cell carries the date; whether it has passed is the flag column's job, not a `· past`
suffix welded to the date. Owners are named in full or the cell says how many there are —
**never `John Doe +2`**; a `+N` hides exactly the people a reader needs to contact.
**Persona modes:** *Risk Manager* = full prioritization (default) · *Vendor reviewer* = vendor-scoped rollup only. For a one-screen exec summary (counts, top risks, untreated count) use drata-risk-report's KPI cards, or drata-executive-report for a stakeholder deliverable.

## Edge cases
| Situation | Handling |
|---|---|
| Natural-language `query` mode | It is single-register and ignores filters — use structured filters for gap identification and say so |
| Register vs workspace name | Resolve via `Drata_listRiskRegisters`; never infer from a workspace name |
| No treatment plan on a top risk | Surface in regardless of score |
| Register has zero open risks | **Corroborate before reporting it as good news.** Structured `Drata_searchRisks` is OpenSearch-backed, so a register whose index was never backfilled returns zero for every filter with no error. Re-check with an unfiltered `Drata_searchRisks(risk_register_id=…, status=["ACTIVE"], size=1)` count and confirm the register resolved via `Drata_listRiskRegisters`. Only then say "No open risks in [register]" and offer a closed/accepted review; if the register resolves but everything is zero, say it returned no indexed risks — **never report an index gap as a clean register** |
| **"Show me the trend" / "how has risk moved since last quarter"** | **No historical snapshots exist anywhere in the MCP.** Say so, offer to baseline today's counts as the first data point, or to diff against a prior report the user pastes in. **Never fabricate a trend.** |
| "Show me our vendor risks" | Ambiguous across three skills — register entries tagged to a vendor are here; the aggregate share per third party is drata-risk-report; vendors needing a review or a decision is drata-vendor-identify-gaps. Ask once which they mean |
| "Which risks are due for review?" | There is no risk review-date field in the MCP. Answer with treatments past or near their committed completion date (`anticipated_completion_date`) and name the substitution explicitly |

## Example invocations
- "Drata, prioritize the open risks in the primary register — worst first."
- "Which Drata risks have no treatment plan, or a treatment past its committed completion date?"
- "Which open risks in Drata are missing an owner or a residual score?"
- "Show me the vendor risks in Drata rolled up by vendor."
- "Which flagged risks in Drata have linked controls that are not ready?"
