---
name: drata-help
description: >
  The catalog and router: explains the report / identify-gaps / resolve-gaps grammar, lists all 18 skills
  grouped by domain with write markers, and points to the best-fit skill for a goal — plus what
  needs no skill. Use for 'what can this plugin do', 'which skill
  should I use'. Reads no compliance data — its only Drata call is the account-scoped
  Drata_getCompany that supplies the artifact's header logo.
area: Start Here
permission: read-only (catalog only — one account read for the header)
compatibility: "Requires Drata MCP with tools: Drata_getCompany (header branding only)"
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
   - **Carve-out for this skill: it reads no tenant data, so three elements of that structure do not apply here.** This is a static catalog and router — it calls no MCP tool, so **never render a KPI row, never a `Pulled from Drata · <timestamp> · <workspace>` source line, and never a `Now · Next · Watch` block.** There are no counts, no workspace and no findings behind them, and a source line on a page that read nothing is a fabricated provenance claim. Use no source line at all, or a plain subtitle naming the plugin and the skill count (`Drata GRC Assistant · 18 skills`). Structure here is: Title → optional plain subtitle → hairline → the catalog rows → footer. **Everything else in rule 1 still binds** — branded self-contained HTML, the §3 `.drata` theme block, table hygiene and the 6-column / 15-row caps, the no-emoji / plain-`DRATA` / Ready · At-risk · Failing conventions, and the footer rule (the icon SVG and nothing else).
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

# Drata Help — Skill Catalog & Router

## Purpose
One answer to "what can this do, and what do I type?" Every skill is named
**`drata-<domain>-<action>`**. The **domain** is the thing you're working on (control, risk,
vendor, …); the **action** is one of just three actions — and the action word tells you exactly
what you get back.

## The whole system in one line
> **`report` to see it · `identify-gaps` to find what needs work · `resolve-gaps` to change it.**

| Action | The question it answers | What you get back | Changes Drata? |
|---|---|---|---|
| **`report`** | "Where do we stand?" | A snapshot / dashboard of the current state | No — read-only |
| **`identify-gaps`** | "What's wrong, and what first?" | A ranked worklist, each item with a next action | No — read-only |
| **`resolve-gaps`** ✎ | "Make the change." | The create / update / delete, previewed and confirmed | **Yes — previewed & confirmed first** |

**Look vs. act.** `report` is the standing view you read, share, or baseline. `identify-gaps` → `resolve-gaps`
is the action loop: identify-gaps safely finds and ranks the work, then resolve-gaps makes the change. Four
families now run the full arc — **risk** (`drata-risk-report` · `drata-risk-identify-gaps` · `drata-risk-resolve-gaps`),
**vendor** (`drata-vendor-report` · `drata-vendor-identify-gaps` · `drata-vendor-resolve-gaps`), **controls**
(`drata-control-report` · `drata-control-identify-gaps` · `drata-control-resolve-gaps`) and **evidence**
(`drata-evidence-report` · `drata-evidence-identify-gaps` · `drata-evidence-resolve-gaps`). **Personnel** is report-only for now.

**Finding a skill:** type a stem to narrow the list — `drata-` for everything, `drata-risk` for
the risk skills, `drata-control` for the control skills.

**Markers:** `✎` marks a skill that writes to Drata. `✎?` marks a read skill with **one optional,
explicitly-confirmed write** — today only `drata-vendor-identify-gaps`, which can escalate a vendor finding
into a risk-register entry if you ask it to. Everything unmarked never changes anything.

## How to use
1. Run the `${CLAUDE_PLUGIN_ROOT}/shared/output-mode.md` check first.
2. If the user named a goal ("are we ready for SOC 2?"), point them at the single best-fit
   skill AND offer to run it. Otherwise render the whole catalog below.
3. Keep it a catalog — do **not** call the Drata MCP from this skill.

## Output format
Render a **compact, scannable** catalog — not this whole file:
1. A title + the one-line grammar: **report** to see it · **identify-gaps** to find what needs work · **resolve-gaps** ✎ to change it.
2. The grouped skill list below, **one row per skill** — name · short phrase · `✎` if it writes — keeping the family groups.
3. If the user named a goal, put the single best-fit skill (with an offer to run it) above the catalog.

Keep everything else in this file — the grammar deep-dive, the router hints, the "Not a skill" notes — as your own routing knowledge; **do not render it into the artifact.**

Lay the catalog out as a **single column** (real `<table>` rows or `.item` rows) with generous row spacing and normal line height. Never a tight multi-column grid or fixed-height cells — cramped columns are what make text overlap; let a long name or phrase wrap, never on top of the next row.

## The catalog
17 skills + this one. Each line is the artifact you get back.

### ▸ Start Here
- **drata-all-identify-gaps** — the cross-domain "what needs attention now" briefing.
- **drata-help** — this catalog and router.

### ▸ drata-control-* — Controls
- **drata-control-identify-gaps** — control worklists: nothing mapped, evidence not current, plus missing owner or description.
- **drata-control-resolve-gaps** ✎ — create/update controls; set owners and requirement/policy/test maps.
- **drata-control-report** — control readiness, and why the rest is not ready.

### ▸ drata-framework-* — Frameworks
- **drata-framework-report** — audit-readiness scorecard for one framework.

### ▸ drata-monitoring-* — Monitoring tests
- **drata-monitoring-report** — test status, results, and how many days tests have been failing.

### ▸ drata-evidence-* — Evidence
- **drata-evidence-report** — evidence freshness: the share of the library that is valid vs expiring, expired, or missing an artifact.
- **drata-evidence-identify-gaps** — the evidence library as one ranked worklist.
- **drata-evidence-resolve-gaps** ✎ — create/link/re-own/re-schedule/retire one item.

### ▸ drata-risk-* — Risk
- **drata-risk-report** — risk posture dashboard: heat map, treatment analytics.
- **drata-risk-identify-gaps** — open risks ranked into an action queue.
- **drata-risk-resolve-gaps** ✎ — log / assess / treat / accept / close a risk.

### ▸ drata-vendor-* — Third-party / vendor risk
- **drata-vendor-report** — vendor portfolio dashboard.
- **drata-vendor-identify-gaps** ✎? — the vendor review queue, or a single-vendor decision.
- **drata-vendor-resolve-gaps** ✎ — onboard / update / retire a vendor record.

### ▸ drata-personnel-* — Personnel & access
- **drata-personnel-report** — who fails which check, and which team fixes it.

### ▸ Reporting & stakeholder comms
- **drata-executive-report** — board / exec / QBR briefing, single or rolled-up.

## Router hints (goal → skill)
- "What's broken / what should I work on?" → **drata-all-identify-gaps**, then **drata-control-identify-gaps**
- "What's overdue?" / "what slipped?" / "my morning briefing" → **drata-all-identify-gaps**
- "Are we ready for our SOC 2 / ISO 27001 audit?" → **drata-framework-report**
- "How close are we to ISO 42001?" → **drata-framework-report**
- "What do these failing tests mean?" → **drata-monitoring-report**
- "How fresh is our evidence? / what share is valid vs expired?" → **drata-evidence-report**
- "Which evidence is expired?" → **drata-evidence-identify-gaps**, then **drata-evidence-resolve-gaps** ✎ to fix an item
- "How many controls are ready, and why not?" → **drata-control-report**
- "Top risks / what to tackle first?" → **drata-risk-identify-gaps**
- "Show me the risk heat map" → **drata-risk-report**
- "Log / treat / close a risk" → **drata-risk-resolve-gaps** ✎
- "Show me high-risk vendors with expired SOC 2" → **drata-vendor-report**
- "Which vendors need review this quarter?" → **drata-vendor-identify-gaps**
- "Should we approve this vendor?" → **drata-vendor-identify-gaps**
- "Who's missing MFA / training?" → **drata-personnel-report**
- "Update the board" → **drata-executive-report**
- "Roll up all our workspaces / business units" → **drata-executive-report**

## Not a skill — just ask

**Answering an inbound security questionnaire, RFP/DDQ section, or auditor question is not a skill either.** Drata has no questionnaire objects, so there is nothing to read or submit. Pull the substance yourself — `drata-framework-report` for framework status, `drata-control-identify-gaps` for control state, `drata-evidence-identify-gaps` for what backs it — then draft in whatever tool the questionnaire lives in. Never present a mapped control or a passing test as an attestation.

**Policy questions ("am I allowed to use personal cloud storage?") are a direct read, not a skill.** `Drata_searchPolicies` returns the matching policy text; answer from it and cite the policy by name. Two limits to state every time: it searches **only the policies assigned to whoever is asking** — not the catalog, not someone else's — and it **requires OAuth** (it fails outright with an API key). Drata exposes no policy owner, approval date, or version, so never name an owner or a freshness date — tell the person to escalate to their policy owner. For policy *coverage* across the tenant, ask directly — `Drata_listPolicies` / `Drata_searchPolicies` are direct reads, not a skill.
Skills are for jobs with a shaped deliverable. A **single-object lookup is not a job** — it's one
MCP read, and reaching for an 18-skill library to answer it wastes the user's time. Answer these
directly, no skill:

- "List our workspaces."
- "What frameworks are in scope?" — there is no list-frameworks tool; I have to probe per framework tag with `Drata_listRequirements(framework_tag=…, is_in_scope=true)`, so tell me which you run.
- "Show me risk CR-08." · "What's the residual score on that risk?"
- "Who is Jane Doe / what's her start date?"
- "What does vendor X's record say?" · "When was vendor X last reviewed?"
- "What policies am I assigned?"

Two rules still apply to direct answers. A direct **read** still follows
`${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md` — label what came from a tool call versus
what you calculated, and never present a mapping as an attestation. A direct **write** still follows
`${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md` in full — preview the exact diff, confirm, write,
read back, verify. Skipping the skill never skips the protocol.

## Maintenance
This catalog is curated and must be updated whenever `skills/` changes — a skill added, renamed,
retired, or re-scoped. Each skill's folder name must equal its `name:` frontmatter exactly. Keep
the family grouping, the router hints, and the `✎` / `✎?` write markers in sync with the folders.
