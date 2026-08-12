---
name: drata-risk-report
description: >
  The visual risk dashboard: KPI cards, 5×5 likelihood × impact heat map, treatment status as a
  stacked bar per register, risk categories showing what treatment removed and what remains, plus
  owner concentration and vendor exposure. Use for risk heat map, risk posture visuals, risk by
  category. Ranked queue → drata-risk-identify-gaps; changes → drata-risk-resolve-gaps. Read-only.
area: Risk Management
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_searchRisks, Drata_listRiskRegisters, Drata_searchControls"
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
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Header identity — the customer's logo, top-left, only when it can truly be inlined; else the company name as text.** Call `Drata_getCompany` once per run before rendering (account-scoped, no arguments, read-only; batch it with the run's other independent reads). It returns `name`, `legalName` and `logoUrl`. Resolve the header in this order and stop at the first that succeeds:
     1. **Inlined logo — gate first, then fetch, then verify.** Attempt this step only if `logoUrl` is non-empty **and** the host provides a tool that can actually download raw image bytes from an arbitrary URL. Many sandboxed hosts — including Claude's cloud / Cowork environments — forbid fetching arbitrary CDN URLs, and the Drata image CDN additionally refuses generic fetchers; **in those hosts this step fails immediately and silently, and falling through to the name is the designed outcome, not a degraded render.** Where a download is possible: fetch once (no retries, no proxies, no cache mirrors, never a route around a refusal), verify the bytes decode as a real image (image magic bytes, mime `image/*`, non-zero size), base64-encode **those downloaded bytes with a real encoder in this run**, and emit `<img class="cust" src="data:[mime];base64,[data]" alt="[company name]" onerror="this.style.display='none';this.nextElementSibling.style.display='block'"><div class="custname" style="display:none">[company name]</div>`. **Never type, reconstruct, or approximate base64 from memory — fabricated image data renders a broken or wrong mark exactly where the customer's identity belongs.** If any part of this step cannot be completed and verified, it did not succeed.
     2. **Company name as text.** `<div class="custname">[company name]</div>` — used whenever `logoUrl` is absent or empty, no permitted fetch path exists in this host, the fetch fails or is refused, the bytes are not a decodable image, or the base64 cannot be produced from real downloaded bytes. **This fallback is first-class: a report headed by the company's name in clean type is a correct header; a broken image, an empty header, or invented image data is the only failure.**
     **Never emit `<img src="https://…">`.** A remote reference is not an acceptable third option: artifact sandboxes block external images, and a blocked, expired or access-controlled URL renders a broken-image icon exactly where the customer's identity belongs. It is a verified inlined image or it is the name — nothing in between.
     **When the logo renders, the company name does not appear as visible text** — it lives in the `alt` attribute and in the hidden `onerror` fallback `<div>`, which stays invisible unless the image fails to decode; that hidden div is the safety net, not a second header.
     **Proportions: constrain the height, leave the width free.** `height:32px; width:auto; max-width:200px; object-fit:contain` — a wide wordmark and a square icon then share one baseline with no stretching, squashing or cropping. **Never set `height` and `width` together, never `width:100%`, never a fixed pixel width**, and never re-encode the image to a different aspect ratio. If a logo would exceed `max-width` at 32px tall, `object-fit:contain` shrinks it proportionally — that is correct, do not compensate.
     On the dark board/exec surface, an inlined dark-on-transparent logo disappears; use `<div class="custname" style="color:#fff">[company name]</div>` instead rather than shipping an invisible mark.
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace name> · <register name>`) → hairline → KPI row → real `<table>` markup → footer. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there. **The source line always spells the scope out in full** — the workspace's own name, or `All workspaces` for an org roll-up covering more than one. Never omit it, never abbreviate it, never substitute a workspace id or a slug, and never leave it to be inferred from the title.
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
     .drata .hm{display:inline-grid;grid-template-columns:auto repeat(5,46px);grid-auto-rows:46px;gap:6px;align-items:center;margin:4px 0 2px;}
     .drata .hm .c{width:46px;height:46px;display:flex;align-items:center;justify-content:center;
       border-radius:4px;font-weight:600;font-size:14px;font-variant-numeric:tabular-nums;color:var(--space);}
     .drata .hm .c.on{color:#fff;}
     .drata .hm .ax{font-family:'Geist Mono',monospace;font-size:11px;color:var(--faint);letter-spacing:.06em;}
     .drata .hm .ax.y{text-align:right;padding-right:6px;} .drata .hm .ax.x{text-align:center;height:auto;}
     .drata .hmwrap{display:flex;align-items:center;gap:10px;}
     .drata .hmy{font-family:'Geist Mono',monospace;font-size:11px;color:var(--faint);letter-spacing:.08em;
       writing-mode:vertical-rl;transform:rotate(180deg);}
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

# Risk Posture Dashboard

## Purpose
An at-a-glance view of the Drata risk register: where risk concentrates (heat map), how much
treatment reduces it, how it is being handled (treatment status), and how much risk each
**category** still carries after treatment.
Read-only; every number comes from a structured `Drata_searchRisks` query. Charts render
client-side; cap chart width at ~600px.

## Key tools
- `Drata_listRiskRegisters` — resolve and label the registers in scope.
- `Drata_searchRisks` — the data source for every element. Use **structured mode** (omit `query`): it searches **across all accessible registers** and stamps `riskRegisterName` on each row. Filters: `status[]` (`ACTIVE | ARCHIVED | CLOSED` — there is no `OPEN`; passing it returns an empty set with no error, so "open risks" is always `status=["ACTIVE"]`), `treatment_plan[]`, `type[]`, `categories[]`, `controls[]`, `owners`, `reviewers`, `vendor_names[]`, `inherent_score_gte/lte`, `residual_score_gte/lte`. `facets[]` = status, treatmentPlan, type, **categories**, controls, **ownerEmails**, reviewerEmails, **vendorNames**. `expand=['controls']` returns each risk's mapped controls.
- `Drata_searchControls` — readiness of those mapped controls. **"Not ready" is `is_ready=False`** — never substitute `has_passing_test=False`.

## Workflow
1. **Resolve scope.** `Drata_listRiskRegisters()`. Structured mode already spans every register, so report register-by-register from the `riskRegisterName` stamp rather than looping one register at a time.
2. **Facet-first, rows later.** One `size=1` call per dimension gives the whole aggregate layer with zero row fetching:
   ```
   Drata_searchRisks(status=["ACTIVE"], facets=["categories","treatmentPlan","status","type"], size=1)
   Drata_searchRisks(status=["ACTIVE"], facets=["ownerEmails","vendorNames"], size=1)
   Drata_searchRisks(status=["ACTIVE"], residual_score_gte=15, size=1)   # KPI: residual score ≥ 15
   ```
   Read `pagination.totalCount` for the denominator. Bounded facets are zero-filled, so "no risks in this bucket" is distinguishable from "bucket not present" — but **neither is distinguishable from "the search index holds nothing for this tenant"**. Structured `Drata_searchRisks` is OpenSearch-backed: on a tenant whose risk index was never backfilled, every structured query returns a full set of confident zeros. **Before publishing an all-zero or near-empty dashboard, sanity-check that the register is genuinely empty** — an unfiltered `Drata_searchRisks(risk_register_id=…, status=["ACTIVE"], size=1)` count, plus confirmation that the register resolved from `Drata_listRiskRegisters`. If the register resolves and every count is still zero, **say the register returned no indexed risks and stop** — never render a dashboard of zeros, which reads as a clean risk posture, and never present an index gap as a compliance finding.
3. **Pull scored rows for the grid.** There is no impact/likelihood facet — fetch rows (`size` ≤ 50, paginate) with `expand=["controls","categories"]` and bucket `impact` × `likelihood` yourself into the 5×5. Same rows give inherent-vs-residual deltas. All bucketing is *Calculated*.
4. **Category aggregation** (charted section below). From the same rows — no second sweep — per category, compute mean **inherent** and mean **residual** **across risks carrying both scores**, plus that paired count and the category's total risk count. A risk with several categories counts under each.
5. **Concentration.** From the `ownerEmails` facet, order owners by how many open risks they hold; from `vendorNames`, show which third parties carry register risk. A facet is **one dimension of counts and nothing else** — an untreated-per-owner column is a second dimension and needs its own call per row, so either ship the two-column table or budget those calls. **No share or percentage column in either table**, and no mean residual anywhere — see the rules below.
6. **Render** the dashboard below and label the aggregation per `${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`.

**Never use the AI `query` mode for a reporting task.** It is single-register and **silently drops every filter**, so the numbers will be wrong without any error. `q` is a keyword filter that composes with filters; `query` does not.

## Output format
A dashboard **spec** — each element plus the query behind it:
```
## Risk Posture — [registers in scope]   **Open risks:** [totalCount]   **Pulled:** [timestamp]

KPI cards ······ Total open · Residual score ≥ 15 · Untreated
                 (totalCount + facets=["treatmentPlan"] + one residual_score_gte=15 count)
Treatment status ······ ONE stacked bar per register over treatment_plan — see below
                 (facets=["treatmentPlan"])
Heat map ······· 5×5 likelihood × impact grid, cell = risk count, colour by severity
                 (rows bucketed client-side — Calculated)
Risk categories ······· one bar per category: what remains vs what treatment removed — see below
                 (facets=["categories"] + expand=["categories"] — Calculated)

### Risk-ownership concentration        (facets=["ownerEmails"])
| Owner | Open risks | Untreated — optional, one count call per rendered row (Tool Calls) |
### Vendor-carried risk                 (facets=["vendorNames"])
| Vendor | Open risks | Untreated — optional, one count call per rendered row (Tool Calls) |

High-risk table ··· top 5 by residual, read-only context for the heat map — the ranked queue with next actions is drata-risk-identify-gaps
```
### Heat map — use the `.hm` grid, do not improvise one
**The cells are squares on a single uniform gap.** Build it with the theme's `.hm` classes and
nothing else. Hand-rolled grids come out with wide rectangles, a large gap between columns and a
small one between rows — a matrix whose axes are not visually symmetric reads as if the two
dimensions are not comparable, which is the one thing a 5×5 exists to show.

Three rules the CSS already enforces — do not override them:
- `display:inline-grid`, so the grid sizes to its cells instead of stretching to the panel width.
  **Never `repeat(5,1fr)`** — `1fr` is what turns squares into wide rectangles on a wide panel.
- One `gap:6px` for both axes. **Never separate `column-gap` / `row-gap`,** and never a margin on
  the cell.
- Fixed `46px` cells with a matching `grid-auto-rows`, so every cell is identical whatever its count.

```html
<div class="hmwrap">
  <span class="hmy">Impact →</span>
  <div>
    <div class="hm">
      <span class="ax y">I5</span><span class="c" style="background:#BEDAFF">1</span>…
      <span class="ax y">I1</span><span class="c" style="background:#BEDAFF">2</span>…
      <span></span><span class="ax x">L1</span><span class="ax x">L2</span>…
    </div>
    <div class="key">…band legend…</div>
  </div>
</div>
```
Row order runs **I5 at the top down to I1**, columns L1→L5 left to right, so the worst cell is the
top-right corner.

**The band is the cell's own score — impact × likelihood — not the count inside it.** That score is
fixed by the cell's position, so the same matrix is coloured identically in every run and across
registers, which is what makes two heat maps comparable. Colouring by count instead makes a busy
low-severity cell look worse than a lone catastrophic one. Bands: **1–6 `#BEDAFF` · 8–12 `#F2C14F` ·
15–25 `#D53641`**, with `class="c on"` for white numerals on the red band only.

**Two exceptions, and only these two:**
- **An empty cell drops to `#BEDAFF` and shows `0`.** A cell with no risks in it is not a hot spot
  whatever its coordinates — leaving I5×L5 blazing red with a zero in it invents exposure that does
  not exist. Show the `0` rather than blanking the cell, so empty regions still read as measured.
- **The single highest-scoring *occupied* cell takes `#FF410C`.** Occupied is the operative word,
  and if several tie on score, the one holding the most risks takes it. Exactly one cell per
  matrix, never zero and never two.

### Treatment status — one stacked bar per register
**Position is fixed: directly beneath the KPI cards, above the heat map.** It is the second thing
on the page and nothing goes between it and the KPI row.

**Exactly one stacked bar per register, never one track per treatment plan.** Five bars reading `Mitigate N of M`, `Accept N of M`, `Untreated N of M`… spends five rows to say what one bar
says, repeats the same denominator five times, and — because every track is scaled to that same
denominator — forces the reader to mentally re-stack them to see the mix. **The segments of one bar
sum to the register's active total**; that is the whole point of the form. One track per plan means
the wrong chart got drawn. A donut is likewise out: it makes the small slices (untreated, avoid)
unreadable, which are the ones worth seeing.

Segment order is fixed, untreated first: **untreated → mitigate → accept → transfer → avoid**.

```html
<div class="sb">
  <div class="sbl"><span class="nm">Register A</span>
    <span><b>[A]</b> untreated · [B] mitigate · [C] accept · [D] avoid · [T] active</span></div>
  <div class="stack">
    <i style="width:10%;background:#D53641"  title="Untreated [A]"></i>
    <i style="width:65%;background:#2E4DFF" title="Mitigate [B]"></i>
    <i style="width:20%;background:#00779C" title="Accept [C]"></i>
    <i style="width:5%;background:#D9DCDE"  title="Avoid [D]"></i>
  </div>
</div>
<div class="key">
  <span><i class="sw" style="background:#D53641"></i>Untreated</span>
  <span><i class="sw" style="background:#2E4DFF"></i>Mitigate</span>
  <span><i class="sw" style="background:#00779C"></i>Accept</span>
  <span><i class="sw" style="background:#BEDAFF"></i>Transfer</span>
  <span><i class="sw" style="background:#D9DCDE"></i>Avoid</span>
</div>
```
Fixed colours: **untreated `#D53641` · mitigate `#2E4DFF` · accept `#00779C` · transfer `#BEDAFF` ·
avoid `#D9DCDE`**. The `treatmentPlan` facet is zero-filled across the enum — drop a zero-count
segment from the bar rather than drawing a sliver, but keep every plan in the legend so absence
reads as measured-and-empty. Scope the facet to `status=["ACTIVE"]`; unfiltered it counts archived
and closed risks and the bar stops summing to the open population.

### Risk categories — what remains vs what treatment removed
Mostly visual: **one horizontal bar per category**, every bar on a common **0–25** track so
categories compare directly. Two segments — **residual `#2E4DFF`** (what is still carried) then
**reduction `#BEDAFF`** (mean inherent minus mean residual, what treatment took off). Reading down
the column, a long dark run is risk still held; a long pale run is treatment working.

**Order by mean residual, highest first** — the question this chart answers is *where is the most
risk still sitting*, not which category is biggest.

```html
<div class="sb">
  <div class="sbl"><span class="nm">Category A</span>
    <span>residual <b>[R]</b> · from [I] inherent · [N] of [M] scored</span></div>
  <div class="stack" style="width:100%">
    <i style="width:60%;background:#2E4DFF" title="Residual [R]"></i>
    <i style="width:20%;background:#BEDAFF" title="Reduced [D]"></i>
  </div>
</div>
```
Segment widths are `mean ÷ 25`, so bars stop short of full width — that empty tail is the headroom
and it is meaningful; **do not normalise each bar to 100%**, which would make every category look
identically risky.

**Both means must come from the same set of risks.** A large share of risks carry an inherent score
and no residual score, and averaging inherent over all of them while averaging residual over only
the scored ones invents a reduction that nobody achieved. **Compute both means over risks that carry both scores**, and
put that base in the label (`N of M scored`). A category where no risk carries both renders as an
empty track labelled `not scored` — never as a zero reduction.

**Clamp the reduction segment at zero and say why it was clamped.** `residualScore > score` — residual
above inherent — is a real recurring data error, and a negative `(mean inherent − mean residual) ÷ 25`
renders as no segment at all, printing an implied *reduction 0* for what is actually a data-quality
problem. If the category's mean reduction is negative, or any risk in it carries residual above
inherent, **do not draw the bar as if treatment achieved nothing**: clamp the width to 0 and state the
count of risks with residual above inherent as a data-quality line beneath the chart.

Cap at 12 categories and say how many were not drawn. **Categories do not sum to the population** —
a risk can carry several and some carry none — so state that once beneath the chart and never
present the category counts as a breakdown of the register.

**Never emit a share or percentage over overlapping dimensions.** No *share of tags*, *share of
categories*, *% of tags*, or any figure dividing one category, owner or vendor count by the sum of
all of them. Every one of those dimensions overlaps: a risk carries several categories, several
owners, and may be counted under a vendor as well — so the denominator is a count of *assignments*,
not of risks, and a "34% share" answers a question nobody asked and no reader can act on. Give the raw count against the register's open-risk total (`N of M`) when a proportion genuinely helps, and
otherwise just the count. This applies to the category chart, the ownership table and the vendor
table alike.

### Concentration tables — a facet is one dimension of counts
**A facet returns single-dimension count buckets and nothing more: no cross-tabs, no means.**
`facets=["ownerEmails"]` gives owner → open-risk count; it cannot also tell you how many of that
owner's risks are untreated, and it never returns an average of anything. An `Untreated` column is a
second dimension and a `Mean residual` column is an aggregate over row data — neither has a source
in the facet response, and this skill forbids tallying rows to invent one.

Two honest options, in this order:
- **Drop the column.** Owner or vendor plus the open-risk count is a complete, correct table.
- **Buy it with calls.** One extra filtered `size=1` count per rendered row (`owners=[…]` or
  `vendor_names=[…]` plus `treatment_plan` = untreated), bounded to the top N you actually display,
  and labelled *Tool Calls*.

**A mean residual needs row data and is out of scope here — remove the column, never approximate
it.** A two-column table that is right beats a four-column table with two invented columns.

## Edge cases
| Situation | Handling |
|---|---|
| User asks for an asset view or asset register | Answer with the risk-category chart, labelled *Calculated*, and name it as categories — do not call it an asset view |
| Risk carries several categories | Count it under each; categories overlap by design and never sum to the population |
| Tempted to use the AI `query` mode | Never for reporting — single-register and silently drops every filter; use structured filters + `facets`, or `q` for keyword |
| Risk has no category | Group as *Uncategorised* and show its size — it is a tagging gap, not zero risk |
| Category's controls unresolved | Show "M mapped, readiness unknown for K" rather than implying they are ready |
| Multiple registers | Structured mode spans all of them; split or facet by `riskRegisterName` and label the scope |
| Sparse residual data | Show inherent-only for those rows and mark residual **Unknown** |
| Asked for trend or quarter-over-quarter | No historical snapshots exist — say so unless the user supplies a prior report |

## Example invocations
- "Which risk categories carry the most residual risk in Drata?"
- "Show me how much our treatments have actually reduced risk, by category."
- "Build me a risk heat map and inherent-vs-residual view from Drata."
- "Who owns the most open risk in Drata, and which vendors carry register risk?"
- "Give me a risk dashboard — KPIs, heat map, treatment breakdown — from Drata."
