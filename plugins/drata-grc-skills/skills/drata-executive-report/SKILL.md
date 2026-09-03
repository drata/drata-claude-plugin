---
name: drata-executive-report
description: >
  The stakeholder briefing: board, executive, audit-committee, or QBR view of live posture —
  headline metrics, framework readiness, top risks — single workspace or org
  roll-up with per-workspace scorecards and repeated-issue detection. Point-in-time only: Drata
  exposes no history, so this reports where things stand today and never a trend or a change since
  last period. Use for 'board deck
  numbers', 'exec summary of our compliance posture', multi-workspace roll-ups. Today's queue →
  drata-all-identify-gaps. Read-only.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listWorkspaces, Drata_searchControls, Drata_listRequirements, Drata_searchMonitoringTests, Drata_searchRisks, Drata_listVendors, Drata_searchPersonnelCompliance"
metadata:
  area: Reporting & Stakeholder Communications
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
     .drata .ws{font-family:'Geist Mono',monospace;font-weight:600;text-transform:uppercase;letter-spacing:.09em;font-size:11px;color:var(--faint);display:flex;justify-content:space-between;border-bottom:1px solid var(--dust);padding-bottom:8px;margin-bottom:14px;}
     .drata .fw{margin-bottom:14px;}
     .drata .fwlab{display:flex;justify-content:space-between;align-items:baseline;font-size:13.5px;color:var(--muted);margin-bottom:5px;}
     .drata .fwlab .nm{color:var(--space);font-weight:500;} .drata .fwlab b{color:var(--space);font-weight:600;font-variant-numeric:tabular-nums;}
     .drata .track{height:22px;background:var(--dust);border-radius:4px;overflow:hidden;}
     .drata .fill{height:100%;background:var(--cobalt);border-radius:4px 0 0 4px;} .drata .fill.ember{background:var(--ember);}
     .drata .m5{display:grid;grid-template-columns:auto repeat(5,1fr);gap:2px;font-size:12px;}
     .drata .m5 .c{padding:10px 0;text-align:center;font-variant-numeric:tabular-nums;border-radius:2px;}
     .drata .m5 .ax{color:var(--faint);font-family:'Geist Mono',monospace;font-size:11px;padding:6px 8px;text-align:right;}
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

# Leadership & Board Briefing

## Purpose
One artifact: the stakeholder briefing, told in **charts first and prose last**. **Workspace scope
sets the shape** — a single-workspace briefing, or an
org roll-up across workspaces / business units / product lines. Keep the metric set small and
defensible, and label every number. Read-only.

## Key tools
- `Drata_listWorkspaces` — **called first, always**: it decides single vs roll-up.
- `Drata_searchControls` — readiness headline (`is_ready`; `pagination.totalCount` is always present in list mode — read it, never paginate to tally, and there is no `include_total_count` parameter on this tool); `Drata_listRequirements` — framework in-scope/ready status.
- `Drata_searchMonitoringTests` — failing automated checks (`check_result_status="FAILED"`).
- `Drata_searchRisks` — open, high-score risks (structured mode spans every accessible register and stamps `riskRegisterName`).
- `Drata_listVendors` — third-party posture (`action_required`; report `breakdown.current` as the vendor headline). **Account-scoped, not workspace-scoped — call it once per briefing, never once per workspace.**
- `Drata_searchPersonnelCompliance` — training / MFA / policy-acceptance posture (pass `slim=True` explicitly; it defaults False here, and pass `employment_status` explicitly or the population silently narrows to current employees + contractors).

## Workflow

**1. RESOLVE SCOPE FIRST.** Call `Drata_listWorkspaces()` before any domain read, and **page it to
completion** — an account may hold more workspaces than one response returns. **The source line
states how many were covered out of how many exist** (`Org roll-up: N of N workspaces`). Never
sample workspaces silently, and never let a roll-up quietly cover a subset.
- **One workspace** → single-workspace briefing. **Never mention the roll-up, multi-workspace comparison, or "other workspaces" at all** — to a single-workspace account that language is noise.
- **Two or more** → ask once: this workspace, a named subset, or the org roll-up. Default to the roll-up when the user's words are org-level ("across all", "by business unit", "portfolio", "per product line", MSP framing).
**Never ask who the audience is.** The four old modes pulled identical data and only reordered
prose; in a chart-led report there is almost no prose left to reorder, so the question spends the
user's attention for nothing. If they name an audience unprompted, honour it as emphasis. Ask only
about scope when two or more workspaces exist.

**1b. Sweep frameworks per workspace — cheap, do not skip it.** `Drata_listRequirements` filters
server-side, so both numbers come back as one-row responses:
```
# discovery + denominator, one call per tag per workspace, all in one parallel batch
Drata_listRequirements(workspace_id="<WS>", framework_tag=["<TAG>"], is_in_scope=true, size=1, include_total_count=true)
   → totalCount > 0 means the framework is in scope in that workspace; the count IS the denominator
# numerator, only for tags that came back non-zero
Drata_listRequirements(workspace_id="<WS>", framework_tag=["<TAG>"], is_in_scope=true, is_ready=true, size=1, include_total_count=true)
```
**`workspace_id` is mandatory on both, exactly as in step 2** — "per workspace" is what makes these
numbers mean anything, and a workspace-scoped call without it returns an `action_required` picker
rather than data on a multi-workspace account. A response with no `totalCount` is a picker, not a
zero: resend it with the workspace, never record it as "framework not in scope".
**Sweep every tag, every time.** The batch must cover the whole enum below — not a shortlist of
"likely" frameworks, not the ones you saw in another workspace. A tag you skip is a framework that
silently disappears from the report. The tag list — CUSTOM, SOC_2, ISO27001, ISO270012022, ISO27701, ISO277012025, ISO270172015, ISO270182019, ISO270182025, ISO420012023, CCPA, CCPA2026, GDPR, HIPAA, PCI, PCI4, SCF, NIST80053, NISTCSF, NISTCSF2, NISTAI, NIST800171, NIST800171R3, CMMC, MSSSPA, MSSSPA11, FFIEC, COBIT, SOX_ITGC, CCM, CYBER_ESSENTIALS, CYBER_ESSENTIALS_32, FEDRAMP, FEDRAMP20X, DRATA_ESSENTIALS, CIS8, HITRUST, DORA, NIS2, ESSENTIAL_EIGHT, NYDFS, TISAX, CPS230, CYFUN, AIUC_1.
**Use these tags exactly.** An unrecognised tag returns **HTTP 400, not an empty result**, so a sweep with a guessed tag errors mid-batch instead of skipping. `ISO42001` is wrong — it is `ISO420012023`; `ISO27018` is `ISO270182019`. If a 400 comes back, read the enum out of the error message and retry rather than dropping the framework. **The list is a starting set, not a closed enum — Drata adds frameworks, and a tag missing from it is not an error, just a framework that never gets probed.** If the user names a framework absent from the list, probe their term anyway and read the accepted values out of any 400 rather than reporting it out of scope. **Never paginate
the requirements catalogue to derive this** — the payload carries long descriptions and the filters
already do the counting. Readiness = ready ÷ in-scope, *Tool Calls* on both figures.

**2. Collect posture with documented filters and denominators.** Structured filters only —
`query` on `Drata_searchControls` switches to a semantic endpoint that ignores every filter, and
an AI `query` on `Drata_searchRisks` silently drops filters too. Read `pagination.totalCount`
for every headline count:
```
Drata_searchControls(is_ready=false)   # + unfiltered total
Drata_listRequirements(is_in_scope=true, is_ready=false)
Drata_searchMonitoringTests(check_result_status="FAILED")
Drata_searchRisks(status=["ACTIVE"], inherent_score_gte=7, sort="inherentScore", sort_dir="desc")
Drata_listVendors(action_required=true)   # ACCOUNT-SCOPED — run ONCE, never per workspace
Drata_searchPersonnelCompliance(security_training=["FAIL"], slim=True,   # check filters take a LIST of PASS|FAIL|MISCONFIGURED|EXCLUDED
  employment_status=[…])                 # pass explicitly — see step 2b
```
**In roll-up mode, run this same set once per workspace with `workspace_id` — except
`Drata_listVendors`, which is account-scoped.** Vendors (and policies) are not workspace
objects: running the vendor call per workspace returns the same account-wide list N times and
multiplies the "vendors needing action" figure by the workspace count. Pull vendors **once**,
report them as a single **org-level** row, and never give them a per-workspace column. Normalize
each workspace to the same scorecard on the workspace-scoped metrics, and **tag every value with
its workspace — never merge similarly-named objects across workspaces**; a control or risk with
the same name in two workspaces is two objects.

**2b. Name the personnel population — never let it default.** Omitting `employment_status` on the
personnel tools **silently filters to CURRENT_EMPLOYEE + CURRENT_CONTRACTOR**. A board or
audit-committee deck must not print a "personnel compliant" figure over an undisclosed
denominator. Pass `employment_status` explicitly (for a whole-org figure include
`SPECIAL_FORMER_EMPLOYEE` / `SPECIAL_FORMER_CONTRACTOR`), read back `_appliedEmploymentStatus`
from the response, and label the population directly under the personnel figure in the output.

**3. Roll up (multi-workspace only).** Produce per-workspace scorecards (readiness, risk,
monitoring, personnel — **not vendors; they are account-scoped and reported once at org level**),
an org summary table, **repeated issues** — the same failing
test `testId`, risk theme, or gap appearing in two or more workspaces, which is the highest-value
finding in this mode.

**4. Separate fact from inference.** Facts (*Tool Calls*) · derived rates (*Calculated*) ·
inferences · unknowns. **Assert no trend without a baseline the user
supplied** — see the edge case below. `is_ready=false` is "failing"; `has_passing_test=false` is
not the same thing.

**5. Lead with the charts.** The two visuals below carry the report; prose exists only to name
the finding. Cap the whole artifact at **six sentences outside chart labels and
tables**. Disclose any workspace or domain skipped for missing permission. A mapped control or a passing test is
**not** an attestation.

## Visuals — the report is charts first

Two charts carry this report. All inline HTML/CSS, no chart library. **Every value is labelled in
text beside its mark**, so nothing is encoded by colour alone and a low-contrast fill never hides a
number. Text stays in ink tokens — never colour text with a series colour.

**Palette (validated, do not re-pick).** Two-state bars use `#2E4DFF` ready/current and `#D53641`
not-ready/expired — that pair passes every check including contrast (ΔE 39.0 normal vision, 29.2
protan). Three-state bars add `#F2C14F` for expiring-soon; it separates cleanly (worst pair ΔE 31.2)
but sits under 3:1 contrast on white, which is exactly why the labels are mandatory, not optional.

**1 · Workspace → frameworks → readiness.** The lead visual. One white `.panel` per workspace, its
`.ws` header row carrying the workspace name and framework count; inside, **one progress bar per in-scope
framework — every one of them, never a top-N**. The header count and the number of bars in that
panel **must be the same number**; check it before rendering, because `N FRAMEWORKS` above three
bars is the report contradicting itself. If a panel genuinely has to be shortened, the header says
so (`N frameworks · M shown`) — but the default is show them all, **all on the same 0–100% scale** so frameworks compare across workspaces. Sort frameworks
**worst readiness first** — the gap is the point. Each bar is an `.fw` block — its `.fwlab` label
row over a `--dust` `.track` with a `--cobalt` `.fill`; a single series needs no legend because the label line carries `N of M · P%` in ink beside
the framework name. Give the **single lowest-readiness framework in the whole report** the `.ember`
fill — that is the one ember, and it should be the number the room talks about.

This replaces the old per-workspace control-readiness bar and the Org summary table. Readiness here
is **requirement-level** (`Drata_listRequirements` `is_ready`), which is what an auditor grades
against. **If a control-level readiness figure also appears anywhere in the report, label both** —
they use different denominators and will not match (on one workspace the control-based and requirement-based figures differed by more than twofold on the same framework).

**2 · Open risks by residual score — the 5×5 matrix.** Grid of `residualImpact` (rows 5→1) ×
`residualLikelihood` (columns 1→5), built with the theme's `.m5` grid classes, each cell holding the
count of open risks at that pair. Fill by
count using the brand band scale only: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, and at most the
single worst cell `#FF410C` — the one ember in this chart. Empty cells stay `--mist`.
`impact`, `likelihood`, `score` and `residualScore` are real Drata fields, so the matrix invents
nothing. **Never label the bands Critical / High / Medium / Low** — Drata gives numbers, not names.

**This matrix needs its own pull — the step-2 top-risks call cannot populate it.** That call carries
`inherent_score_gte=7` and is sorted and capped for a *list*; cells built from it are silently a
high-inherent subset drawn from one page, presented as "open risks". Two independent errors in the
same grid: a filtered population and a truncated one. So fetch the matrix separately —
`Drata_searchRisks(risk_register_id=<each register>, status=["ACTIVE"], expand=["registers"], size=50)`,
**paginated to the end**, with no score filter — and bucket `residualImpact` × `residualLikelihood`
client-side. Label the count *Calculated*, state the population under the grid (`across [n] active
risks in [registers]`), and reconcile: **the cells plus the risks carrying no residual pair must equal
that population.** Risks with no residual scores are not a cell — say how many sat outside the grid
rather than dropping them. If you cannot page the register, render no matrix; a partial grid reads as
a complete one.

**Two charts, no more.** Evidence-currency and policy-publication belong to drata-evidence-identify-gaps, not to a briefing. **No other charts.** No trend or quarter-over-quarter line — the MCP exposes no history, and a
fabricated trend is the worst thing this report could do. No gauge or donut for a percentage a bar
already carries. No composite score.

## Frameworks — deliberately not listed
**Never print a framework list or a per-item framework column.** A control maps to several
frameworks at once, and the report cannot prove it has enumerated them all for a workspace, so any
list reads as complete when it is not. Framework-level readiness is a different question with its
own skill: route it to **drata-framework-report**. (For the record: `Drata_listRequirements`
does expose `frameworkName` per requirement, so the set is derivable by paging the whole requirements
catalogue — it is simply too heavy, and too easy to under-report, to belong in an exec briefing.)

## Output format
```
## Executive Report — [workspace | N workspaces] · [date]
**Scope:** [workspace name | N workspaces rolled up] · **Prepared:** [timestamp] · point-in-time

### Readiness by framework
**KPI row — every tile `X of Y`, same polarity across the row.** Match the bars below and count the
healthy side: requirements ready of in-scope · controls ready of in-scope · monitoring tests **not
failing** of enabled Production · personnel **passing security training** of population. Never mix
"ready" and "not ready" tiles in one row, and never park a denominator in a tile label.

**Every tile must name exactly what its call measured — no tile may claim more than its filter.**
Three of these are easy to overstate, and all three are wrong in a way a customer will catch:

- **Monitoring.** The only pull is `check_result_status="FAILED"`, so the honest figure is *not
  failing*, never *passing*: the non-FAILED remainder also contains `ERROR`, `DISABLED` and `UNUSED`
  tests, and an errored test is broken tooling, not a passing control. The denominator is **enabled
  Production** tests, which means the Code subtraction (`total − test_source=code`) applies here too,
  on both the numerator and the denominator — the product's monitoring list shows Production only, so
  a pooled figure visibly exceeds the customer's screen. If you are not prepared to run the paired
  Code calls, drop the tile; do not ship a pooled one. Depth belongs to drata-monitoring-report.
- **Personnel.** The only pull is `security_training=["FAIL"]`. That yields *passing security
  training*, not *compliant* — compliance spans MFA, policy acceptance, background checks and
  devices, and labelling one check as all of them is the single most misleading tile this report can
  carry. Either title the tile for the one check, or facet the others and title it for what you
  actually faceted.
- **Controls.** The denominator is **in-scope** controls (`is_enabled` default True), not "total";
  out-of-scope controls belong in no readiness denominator, and drata-control-report uses in-scope,
  so "total" here would put the same tile at two different values in two skills.

Where a tile cannot be sourced honestly, **omit the tile** — a four-tile row is not a requirement,
and a mislabelled tile is worse than a missing one.

Then **chart 1 — one panel per workspace, one bar per framework, worst first**. At most two
sentences of prose; the bars carry it.
Personnel passing security training [N] of [M] · *Population: [_appliedEmploymentStatus, spelled out]*
**Org-level (account-scoped, not per workspace):** vendors needing action [N] of
[breakdown.current] — one figure for the whole account, counted once; a body line, never a tile
in the KPI row.

### Risk exposure
**Chart 2 — the 5×5 residual matrix.** One sentence naming the worst cell and its count. No table.

### Repeated issues across workspaces        [roll-up only]
- [theme] appears in [WS-A, WS-C] — …

### Per-workspace detail                     [roll-up only]
Only where a workspace needs comment the bars do not already make. Skip it otherwise.
   (no vendor line — vendors are account-scoped and appear once, above)

### Top risks & gaps
| Item | Type | Owner | Residual score | Workspace |
No framework column — see the frameworks rule.
```

## Edge cases
| Situation | Handling |
|---|---|
| Account has one workspace | Single-workspace briefing; never mention roll-up, portfolio, or cross-workspace comparison |
| **"Progress over the last 6 months" / any trend or QoQ ask** | Historical snapshots aren't yet available through the MCP, so a trend can't be computed here. Say so plainly, offer to baseline today's numbers, and offer to diff against a prior report the user pastes in. **Never fabricate a trend line.** |
| Tempted to add a "changes this period" or "what's new" section | Don't. The MCP has no history, so a changes heading has to disclaim itself in its own first line, and what fills it is current state already covered by Top risks & gaps. A report is a long-term snapshot: say `point-in-time` on the scope line and stop there. Creation timestamps could support an additions list, but that is deliberately out of scope here |
| Tempted to ask who the briefing is for | Don't — never prompt for audience. Produce the one report; honour an audience only if the user named it unprompted |
| Tempted to list frameworks | Don't — see the frameworks rule. Controls map to several frameworks each and the report cannot prove the list is complete |
| Similarly-named objects across workspaces | Never merge; always tag with the workspace |
| **Vendors or policies in a roll-up** | Both are **account-scoped, not workspace-scoped**. Pull them once and report one org-level figure; running them per workspace returns the same records N times and multiplies the count by the workspace count |
| **Personnel figure in a board / audit-committee deck** | Never print it without the population. Pass `employment_status` explicitly, read back `_appliedEmploymentStatus`, and label the denominator under the figure — the default silently drops to current employees + contractors |
| A workspace or domain read lacks permission | Disclose which were included and skipped; brief on what is available |
| "What should I work on today" | Route to drata-all-identify-gaps (the operational queue); for control field quality or owner distribution, drata-control-identify-gaps |
| Formal exported report or PDF requested | No PDF or deck export is produced here — the deliverable is the branded HTML artifact or `.html` file |

## Example invocations
- "Drata, prepare a board briefing on our compliance posture for Q3."
- "Give me an executive weekly update from Drata for the CISO."
- "Draft an audit-committee readout on SOC 2 readiness and open findings."
- "Build a QBR compliance summary for the leadership team."
- "Show me readiness and open risks by business unit in Drata."
- "Give me an org-level roll-up across every workspace — where do issues repeat?"
