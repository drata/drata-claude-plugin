---
name: drata-vendor-identify-gaps
description: >
  The vendor action queue. Seven checks over current vendors - security review never started, past
  its deadline, or last completed over a year ago; next review overdue; no business unit; no
  security owner; status on hold - plus prospective vendors whose review has not started. Grouped by
  what the fix is, with single-vendor decision memos. Use for 'which vendors need review', overdue
  or missing security reviews, unowned vendors, vendor decisions. Portfolio -> drata-vendor-report.
  Mostly read-only; one gated write.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listVendors, Drata_getVendor, Drata_listVendorSecurityReviews, Drata_listVendorDocuments, Drata_searchRisks. Escalation also needs Drata_createRisk + create:risk scope."
metadata:
  area: Third-Party & Vendor Risk
  permission: "write (optional: create a risk on explicit confirmation)"
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
     .drata .lvl{display:inline-flex;align-items:center;gap:7px;font-size:12.5px;white-space:nowrap;}
     .drata .lvl i{width:9px;height:9px;border-radius:2px;display:inline-block;flex:none;}
     .drata tbody tr:hover{background:#FAFBFC;}
     .drata .flag{display:inline-block;font-size:11px;font-weight:500;padding:2px 9px;border-radius:999px;
       background:#FDE9EA;color:#A32A32;white-space:nowrap;}
     .drata .flag.warn{background:#FDF3DC;color:#8A6410;} .drata .flag.info{background:#E8EDFF;color:#2039E2;}
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
5. Write safety (mandatory for every create / update / delete): `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md`

# Vendor Action Queue & Decision

## Purpose
Turn third-party risk into work someone can do today. Two action-shaped answers under one
contract: **the queue** — every vendor Drata already flags as needing action, ranked, each row
with a named next action and an owner — and **the decision** — on one named vendor, an approve /
approve-with-conditions / defer / reject / reassess memo. Read-only, apart from the optional
risk escalation under `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md`.

## Key tools
- `Drata_listVendors` — builds the queue from two **server-side flags**: **`action_required: bool`** and **`next_review_deadline[]` = `OVERDUE|DUE_SOON|NO_RENEWAL`**. Narrow with `risk`, `impact_level`, `sub_processor`, `owner`; `expand`: `latestSecurityReviews`, `lastQuestionnaire`.
- `Drata_getVendor` — the full record for the decision; accepts an **exact name** or the numeric ID. `expand` allowlist: `customFields, documents, integrations, lastQuestionnaire, latestSecurityReviews, reviews, vendorUser, vendorRelationshipContact, dataAccessedOrProcessed, scheduleConfiguration, customVendorType`. There is **no `owner` or `risk` expand** — `risk` (residual), `impact_level` (inherent) and the owner are plain fields already on the record.
- `Drata_listVendorSecurityReviews` — review history (`status`, `decision`, `created_at_from/to`).
- `Drata_listVendorDocuments` — attestations, SOC 2 reports, bridge letters, DPAs.
- `Drata_searchRisks(vendor_names=[…])` — risks already logged against the vendor.
- `Drata_createRisk` — optional confirmation-gated escalation only.

## Workflow
1. **Scope & mode.** Vendors are **account-scoped** — do **not** ask which workspace. A named
   vendor → decision; no named vendor → queue; both if asked for both.
2. **Build the queue from the flags Drata already computes** — never reconstruct "overdue" from
   renewal dates:
   ```
   EXPAND = ["latestSecurityReviews"]
   Drata_listVendors(action_required=True, expand=EXPAND)
   Drata_listVendors(next_review_deadline=["OVERDUE","DUE_SOON"], expand=EXPAND)
   ```
   Union both sets, dedupe by vendor ID, and read `pagination.totalCount` per call for the
   denominators — never paginate to tally. These calls supply the flag denominators; the row data
   for all seven checks comes from the single paginated sweep in *How to gather it* below.

   **The two flags do not cover the same population, so never present their union as one number.**
   `action_required` is already narrowed server-side to *current* vendors **that have an owner** whose
   review is due soon or overdue — so it silently excludes every **unowned** vendor, which is exactly
   the population check 6 exists to find. `next_review_deadline` applies no owner filter and is not
   current-scoped at all. Consequences: the union is not "vendors needing action", `action_required`
   can never surface an unowned overdue vendor, and a count taken off it will sit below the review backlog the Drata UI shows. Use `next_review_deadline` for the review-currency counts, get unowned
   vendors from the swept rows, and if you cite `action_required` at all, say what it covers.
3. **Rank (Calculated), then name the work.** Rank by `next_review_deadline` bucket (OVERDUE
   ahead of DUE_SOON), then residual `risk` level, then
   `impact_level`; show those fields beside the rank. **There is no "days overdue" to rank on.**
   `next_review_deadline` is a three-value bucket (`OVERDUE|DUE_SOON|NO_RENEWAL`), not a date, and
   this skill deliberately does not reconstruct one from renewal dates — so an exact age does not
   exist in the data you have. Rank within a bucket on risk, never on an invented number of days. Every row gets a **named next action** — refreshed SOC 2, chase the questionnaire,
   schedule the review, get a bridge letter, escalate, retire — and an **owner** from the
   vendor's `owner`; if empty the row reads **Unowned — assign an owner first**. Only owner,
   tier, status, renewal cadence and retire are writable (drata-vendor-resolve-gaps); document,
   questionnaire and security-review actions are external — label them "outside Drata" on the row.
4. **Single-vendor decision.** Resolve the vendor, then gather in parallel:
   ```
   Drata_getVendor(vendor_id=…, expand=["latestSecurityReviews","lastQuestionnaire","documents","vendorUser"])
   Drata_listVendorSecurityReviews(vendor_id=…)
   Drata_listVendorDocuments(vendor_id=…)         # presigned links only at use
   Drata_searchRisks(vendor_names=["<vendor>"])   # already-logged risks
   ```
5. **Issue the memo.** Weigh inherent (`impact_level`) vs residual (`risk`), review currency,
   document gaps, and open risks into one decision, with conditions, owner, and next review
   date. Thin evidence is a **Gap** pointing to *defer* or *reassess*, never a silent approve.
   Label conclusions per `${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`. The decision
   verdict is this skill's explicit, user-requested deliverable — the sole exception to the
   no-verdicts rule; label it *Calculated*.
6. **Optional escalation.** For a finding that warrants a register entry, follow
   `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md`: resolve the register, preview the exact
   `Drata_createRisk(title=…, description=…, risk_register_id=…)` payload, confirm, write, read
   back. Title and description come from the finding — never fabricated.

## The seven checks — every run covers all of them
**A vendor enters the queue only if it fails one of these. Never ship a run that silently skips a
check** — if a check yields zero, print the heading with `0` rather than dropping it, so the reader
knows it was tested.

| # | Check | Detection | What is owed |
|---|---|---|---|
| 1 | Security review never started | `latestSecurityReviews` is `[]`, or its `status == "NOT_YET_STARTED"` | Start the review |
| 2 | Security review past its deadline | latest review `status != "COMPLETED"` and `reviewDeadlineAt` in the past | Finish or re-deadline it |
| 3 | Last completed review over a year old | newest review with `status == "COMPLETED"` is older than 12 months | Re-review |
| 4 | Next review date overdue | `next_review_deadline=["OVERDUE"]` | Re-review or re-schedule |
| 5 | No business unit | `category` is null **or** the string `"NONE"` | Assign a business unit |
| 6 | No security owner | `vendorUser` expand returns `user: null` | Assign an owner |
| 7 | Status on hold | `status=["ON_HOLD"]` | Resolve or release the hold |

Plus one prospective-only check: **prospective vendors whose security review has not started** —
`status=["PROSPECTIVE"]` intersected with check 1. Those are vendors waiting on the business, not
on a supplier, and they are the fastest thing on this page to clear.

**Unassigned business unit is decided on the row, never by a filter.** `category` accepts only its 11
named values (ENGINEERING, PRODUCT, MARKETING, CS, SALES, FINANCE, HR, ADMINISTRATIVE, SECURITY,
LEGAL, INFORMATION_TECHNOLOGY) — **there is no null or `"NONE"` filter to select unassigned vendors**,
so this check is evaluated client-side off the swept rows. Treat a `category` that is absent, null or
the literal string `"NONE"` as unassigned; all three occur depending on how the record was created.
The trap worth remembering: `risk: "NONE"` **is** a real residual level, so a rule that reads `"NONE"`
as absence on `risk` the way it does on `category` will quietly mis-sort vendors. Same string,
opposite meaning, two different fields.

**Empty `latestSecurityReviews` is "never reviewed", not "review passed".** An absent array is the
most common state on an unmanaged portfolio and it must never fold into a completed or not-required
bucket. `NOT_REQUIRED` is a deliberate decision someone recorded and is **not** a queue item.

### How to gather it — one pass, two expands, nothing else
Checks 4 and 7 have server-side filters and are cheap. **Checks 1, 2, 3, 5 and 6 have none**, so
they are computed from rows:
```
Drata_listVendors(size=50, expand=["vendorUser","latestSecurityReviews"], cursor=…)
```
Paginate that once over current vendors and evaluate all seven client-side. **Those two expands and
no others** — adding `documents`, `reviews` or `integrations` multiplies the payload across hundreds
of vendors and is what made the old dashboard slow. Never call `listVendorDocuments` or
`listVendorSecurityReviews` per vendor to build the queue; they are drill-down tools for one named
vendor in a decision memo.

Scope every count to current vendors. Archived records keep lapsed deadlines and stale reviews
forever, so an unscoped pass reports a backlog several times its real size.

### Grouping — by what the fix is, one row per vendor
Three sections for current vendors, one for prospective. **A vendor appears once**, under its first
matching group, with its other flags listed in its row:

- **Needs a review** — checks 1, 2, 3, 4, **narrowed to material vendors only**:
  `risk` in **MODERATE, HIGH** *and* `impact_level` in **MODERATE, MAJOR, CRITICAL**.
  Both conditions must hold — a vendor that is high residual but insignificant inherent, or critical
  inherent but no residual, does not enter this group. Review effort is finite and this is where it
  belongs.
- **Needs a record field** — checks 5, 6
- **On hold** — check 7. **Name the section after the status, not after an abstraction.**
  `On hold` is exactly what `status=["ON_HOLD"]` returns and exactly what the reader sees in Drata;
  `Needs a decision` is a euphemism for one enum value and forces the reader to guess which. The
  same applies to the chip in the row — it reads `On hold`.
- **Prospective — review not started**

**The risk narrowing hides vendors, so say how many.** Two populations fall out of *Needs a review*
and neither is safe to drop silently:
- vendors below the thresholds — genuinely lower priority, and a one-line count is enough;
- **vendors that are `UNSCORED` on inherent** — these fail the `impact_level` test not because they
  are immaterial but because nobody has judged them yet. On an unassessed portfolio this is the
  larger group by far, and letting the filter swallow it turns "we reviewed what mattered" into a
  circular claim.

Close the group with one line: `[n] more vendors have a review gap but fall below the risk
thresholds, of which [n] are unscored on inherent risk.` The unscored population's real home is the
assessment-coverage bar in drata-vendor-report — point there rather than re-listing it.

The other three groups apply the checks to **all current vendors**, unnarrowed: a missing owner or
business unit is a record defect at any risk level, and it is the cheapest thing on the page to fix.

A single table with seven flag columns would breach the six-column cap and put a mostly-empty grid
in front of the reader; the grouped form carries the same information and sorts by what someone
actually has to go do. Close with the count carrying no flag, unlisted. The three group counts plus the no-flag count sum
to `breakdown.current` — check that before shipping, even though only two of the groups have a
tile.

## Output format
One contract — the queue; the memo is the drill-down on a row or a named vendor.

**Review currency is this skill's headline, and it lives here rather than in drata-vendor-report.**
"Which vendors are due" is a worklist someone works, so the three `next_review_deadline` counts open
this queue; the report deliberately carries none of them. Scope every tile to
`breakdown.current` — filtered totals include archived vendors, which keep their lapsed review dates
forever and inflate the overdue figure several-fold (`OVERDUE` returns a total far above the current-vendor subset). Never publish the raw total.

```
## Vendor Action Queue — [date]   (account-scoped)
Source line: `Pulled from Drata · [date] · account-scoped`

KPI row — two tiles, both `n of [breakdown.current]`, both needs-attention polarity:
          **needs a review** · **needs a record field**
          (**no On hold tile** — it is one status count, and it is already the whole
           On hold section below; a tile that restates a section spends the row on nothing)

| Vendor | Residual | Inherent | Flag | Owner |
|---|---|---|---|---|
| Acme Corp | <span class="lvl"><i style="background:#D53641"></i>High</span> | <span class="lvl"><i style="background:#D53641"></i>Critical</span> | <span class="flag">Review overdue</span> | owner@example.com |

**Five columns, and this is the shape:**
- **Vendor name only — never the numeric id**, and never `Acme Corp (1234)`. The id is plumbing; it means
  nothing to the person doing the work and it is the widest low-value thing in the row. Same for a
  rank number: rows are already in priority order, so a `#` column re-states the row position and
  costs a column against the six-column cap.
- **Residual and inherent render as a colour pip plus the word**, using the `.lvl` class — pip
  `#D53641` High/Critical · `#F2C14F` Moderate/Major · `#BEDAFF` Low/Minor · `#D9DCDE` None/Unscored.
  The word stays; the pip is what lets someone scan the column in one pass. **Never colour the whole
  row or the cell background** — a table of red rows reads as one undifferentiated alarm.
- **One `.flag` chip per row** naming the failing check — the bucket word, never a day count
  (`Review overdue`, `Review due soon`, `Never reviewed`, `No owner`). **Never write `Review [N]d overdue` or any other age:** `next_review_deadline` is a bucket, so a number of days there is
  fabricated. Additional flags go in the same cell as
  further chips — `.flag` red for overdue and never-reviewed, `.flag warn` amber for record gaps,
  `.flag info` for on-hold.
- Owner is the email or **`—` for unassigned**; never leave the cell blank, since blank reads as a
  rendering fault rather than a finding.

### Decision — [vendor]
**Decision:** Approve with conditions   **Inherent:** Critical   **Residual:** High
**Reviewed:** [N] security reviews · SOC 2 Type II (ended [date]) · [M] open risk(s) (Tool Calls)
**Gaps:** No penetration test · questionnaire [N] months old (Gap)
**Conditions:** DPA countersigned; SOC 2 by [date]
**Owner:** [name/email]   **Next review:** [date]

### Now · Next · Watch
**Now** — overdue reviews on high-residual or sub-processor vendors. → record changes drata-vendor-resolve-gaps; the review itself happens in the Drata UI
**Next** — due-soon reviews, and vendors with no owner to chase them. → drata-vendor-resolve-gaps
**Watch** — UNSCORED vendors (a scoring gap, not low risk) and attestations lapsing next quarter.
```
Record changes (owner, status, tier, renewal cadence, retire) route to drata-vendor-resolve-gaps;
document, questionnaire and security-review work happens in the Drata UI.

## Edge cases
| Situation | Handling |
|---|---|
| Document link needed | `expand=['downloadUrl']` returns a **~60-second presigned URL** — use it immediately, never cache or persist it |
| Vendor identified by name | `Drata_getVendor` / `listVendorDocuments` / `listVendorSecurityReviews` accept an **exact name**; `updateVendor` / `deleteVendor` need the **numeric ID** — resolve it before any handoff. Ambiguous name: ask once, never guess |
| Queue empty, or a portfolio breakdown wanted | Say the queue is empty; the breakdown is the dashboard → drata-vendor-report |
| Escalation requested | Follow `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md`; degrade to read-only if `create:risk` scope is absent and say so |

## Example invocations
- "Drata, which vendors need action right now — worst first?"
- "Which vendor reviews are overdue or due soon, and who owns them?"
- "Which vendors need security reviews this quarter?"
- "Drata, run due diligence on Acme Corp and give me an approve/reject recommendation."
- "Should we renew Northwind? Pull their reviews, documents, and open risks."
- "Drata, log the missing-pen-test finding on Acme Corp as a risk."
