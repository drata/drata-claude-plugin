---
name: drata-personnel-report
description: >
  The workforce compliance report: pass / fail / excluded per check as stacked bars, split by
  employee vs contractor and by department, plus onboarding time-to-compliance and offboarding
  ageing. Population-level only — no individual names. Use for 'how compliant is our workforce',
  training and policy coverage, onboarding and offboarding health. Read-only.
area: Personnel & Access Compliance
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_searchPersonnelCompliance, Drata_searchDeviceCompliance, Drata_listPersonnel, Drata_listDevices, Drata_listPersonnelGroups, Drata_lookupUserIdentity"
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
   - **No individual people.** This report is population-level: **never print a person's name, email, or a per-person row**, and never a list that identifies individuals by elimination. Counts, rates, medians and distributions only. A named remediation list is a different deliverable and is not produced here; personnel records are changed in the Drata UI. **Never print "population-level by design", "for the named list, use …", or any other sentence explaining what this report deliberately omits** — the reader did not ask what is missing, and a report that narrates its own scope decisions spends attention and returns nothing.
   - **Charts, not tables, for every distribution.** Each check, each population split, each cohort renders as a horizontal stacked bar. A table is only for values a bar cannot carry (medians in days). Bars use `#2E4DFF` pass · `#D53641` fail · `#D9DCDE` excluded — validated: worst adjacent pair separates at ΔE 37.1 normal vision, 29.2 protan; the grey sits under 3:1 contrast, so **the count labels beside each bar are mandatory**.
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

# Workforce Compliance Report

## Purpose
Show **where the workforce stands**: pass / fail / excluded for every compliance check, split by
employee vs contractor and by department, plus how fast new starters reach compliance and how
former staff clear offboarding. Population-level and chart-led — distributions and rates, never a
per-person roster or a queue of actions. Read-only.

## Key tools
- `Drata_listPersonnel` — roster slices (`employment_status`, `compliance_status`, `group`, `custom_field`, `facets`). **`slim=True` returns `failingChecks[]` per person in the same call** — it defaults True here.
- `Drata_searchPersonnelCompliance` — per-check status: `accepted_policies`, `bg_check`, `identity_mfa`, `security_training`, `hipaa_training`, `nist_ai_training`, `offboarding` (PASS|FAIL|MISCONFIGURED|EXCLUDED). `slim` defaults **False** — pass it explicitly.
- `Drata_searchDeviceCompliance` — `agent_installed`, `password_manager`, `disk_encryption`, `antivirus`, `auto_updates`, `lock_screen`. **Returns people, not devices**, and reflects manual evidence. `slim` defaults **False** — pass it explicitly.
- `Drata_listDevices` — a single **named person's** devices and per-monitor checks. There is no account-wide device list.
- `Drata_listPersonnelGroups` — teams/departments for the owning-team grouping.
- `Drata_lookupUserIdentity` — `expand=['roles']` is the only role/permission visibility in the server (access reviews, admins, service accounts).

## Workflow
1. **Fix the denominator first — this is not optional.** Omitting `employment_status` **silently
   restricts the population to CURRENT_EMPLOYEE + CURRENT_CONTRACTOR**. Read back
   `_appliedEmploymentStatus` from the response and label every count with the population it
   covers. For "everyone", or for any former-staff offboarding check, pass the list explicitly and
   include **SPECIAL_FORMER_EMPLOYEE** and **SPECIAL_FORMER_CONTRACTOR** or you will undercount:
   ```
   Drata_listPersonnel(employment_status=["CURRENT_EMPLOYEE","CURRENT_CONTRACTOR",
     "FORMER_EMPLOYEE","FORMER_CONTRACTOR","SPECIAL_FORMER_EMPLOYEE","SPECIAL_FORMER_CONTRACTOR"])
   ```
2. **Build the headline from facets before fetching any rows.** A couple of `size=1` faceted calls
   give per-check failure counts across the whole population:
   ```
   Drata_searchPersonnelCompliance(facets=["identity_mfa","accepted_policies","security_training",
     "bg_check"], size=1)
   Drata_searchDeviceCompliance(facets=["disk_encryption","antivirus","agent_installed",
     "auto_updates","lock_screen","password_manager"], size=1)
   ```
   Bounded facets are zero-filled, so "0 failing" is distinguishable from "not measured". Rank the
   roster's sections by these counts; read `pagination.totalCount`, never tally pages.

   **The `offboarding` facet is never part of that batch.** Offboarding applies to former staff, and
   these calls run against the active-only default — asking for it here returns all zeros and the
   zeros are an artefact of your own filter, not a finding. It gets its own former-scoped pair:
   ```
   Drata_listPersonnel(employment_status=["FORMER_EMPLOYEE","FORMER_CONTRACTOR",
     "SPECIAL_FORMER_EMPLOYEE","SPECIAL_FORMER_CONTRACTOR"], include_total_count=True, size=1)
   Drata_searchPersonnelCompliance(employment_status=["FORMER_EMPLOYEE","FORMER_CONTRACTOR",
     "SPECIAL_FORMER_EMPLOYEE","SPECIAL_FORMER_CONTRACTOR"], facets=["offboarding"], size=1)
   ```
   **A zero here means you forgot `employment_status`.** Any tenant with history has hundreds of
   former personnel. If either call returns 0, the filter is wrong — re-issue it with the four
   `FORMER_*` statuses spelled out before you write a single word about the result.

2a. **Personnel are account-wide. The workspace never filters them.** The MCP may still prompt you
   to pick a workspace; pick one and move on — the same personnel counts come back from every
   workspace in the account. **Never write, imply, or reason from "no former personnel in this
   workspace", "this workspace has no personnel", or any other workspace-scoping explanation for a
   personnel count.** That explanation is always false, and reaching for it means you are
   rationalising an empty result instead of fixing the filter that produced it. The workspace name
   belongs in the source line only, as provenance — never as the scope of a personnel figure.
3. **Fetch rows only for the clusters you will act on**, using the specialised reason filter — never
   a generic overall-status search:
   ```
   Drata_searchPersonnelCompliance(identity_mfa=["FAIL"], slim=True)
   Drata_searchDeviceCompliance(disk_encryption=["FAIL"], slim=True)
   ```
   Every check filter takes a **list of enum values** — `PASS | FAIL | MISCONFIGURED | EXCLUDED` —
   never a boolean. `identity_mfa=False` is not a valid filter.
   **`MISCONFIGURED` and `EXCLUDED` are distinct from `FAIL`.** An excluded person is a documented
   exception, not a failure — never fold them into the failure count; give them their own segment in
   the check chart (with the exclusion named) so the exception stays visible and reviewable. `MISCONFIGURED` means the
   check itself is broken — a tooling fix, not a person to chase. Neither is actionable from here —
   exclusions are managed in the Drata UI and MISCONFIGURED checks in the connected integration;
   say so rather than assigning a person.
4. **Group by root cause, then by owning team.** Resolve teams with `Drata_listPersonnelGroups` and
   slice each failure by `group`. One person can appear under several causes — expected.
5. **Single-person diagnosis — only when the user asks about a named person, answered inline, never
   in the report artifact.** `Drata_listPersonnel(email=…, slim=True)` returns `failingChecks[]`
   in one call — the fastest path to "why is this person non-compliant", with no second lookup.
   Only reach for `Drata_listDevices(person=…)` when you need per-device detail.
6. **Access / role review — on direct request only; not part of this report's output.**
   `Drata_lookupUserIdentity(expand=["roles"])` for who holds Admin and for
   non-personnel accounts; scope a team with `Drata_listPersonnelGroups` + `Drata_listPersonnel(group=…)`.
7. **Label the posture you are reporting** (`${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`):
   raw per-device check results are **not** the same as a person's effective compliance posture,
   which can include manual evidence or exclusions. Say which one the roster shows.

## Output format
```
## Workforce Compliance Report — [workspace] · [date]
Source line: `Pulled from Drata · [date] · [workspace] · Population: [_appliedEmploymentStatus,
spelled out]` — the population is a denominator disclosure and lives here, never in a tile.

### KPI row — plain cards, no mini bars
`Non-compliant [N] of [population]` leads and nothing displaces it, then the three largest failing
checks, each against the same population, all counting the same direction. **No mini progress bar inside a
tile in this report** — every distribution below is already a full-width stacked bar, and a 6px bar
in the tile duplicates the chart under it at a size nobody can read.

### Compliance by check — one stacked bar per check
**This replaces the old Excluded table and Failing-checks table; they are one chart now.** One bar
per check, ordered by fail count, every bar on the same population scale so checks compare directly.

```html
<div class="sb">
  <div class="sbl"><span class="nm">Policy acceptance</span>
    <span><b>[N]</b> fail · [M] pass · [X] excluded</span></div>
  <div class="stack">
    <i style="width:60%;background:#2E4DFF" title="Pass [M]"></i>
    <i style="width:39%;background:#D53641" title="Fail [N]"></i>
    <i style="width:1%;background:#D9DCDE"  title="Excluded [X]"></i>
  </div>
</div>
<div class="key">
  <span><span class="sw" style="background:#2E4DFF"></span>Pass</span>
  <span><span class="sw" style="background:#D53641"></span>Fail</span>
  <span><span class="sw" style="background:#D9DCDE"></span>Excluded — not in scope for this check</span>
</div>
```
**Excluded is a third segment, never folded into pass.** Rates quoted in the label are
`fail of (population − excluded)` — the in-scope denominator. A check whose fail equals the entire
in-scope population is unconfigured, not failing: render it greyed with the label
`no results recorded` and keep it out of the ordering.

### Employees vs contractors — the same bars, two panels
One panel per population, identical bar set and scale, so the two read side by side. Populations
that behave differently are the finding; the bars show it without a sentence.

### By department — one stacked bar per department
**Always rendered when a department-shaped group family exists.** Same `.sb` markup as the check
chart, same three segments, one bar per department, ordered by fail rate:

```html
<div class="sb">
  <div class="sbl"><span class="nm">dept-alpha</span>
    <span><b>[N]</b> fail · [M] pass · [T] people · [P]%</span></div>
  <div class="stack">
    <i style="width:65%;background:#2E4DFF"></i>
    <i style="width:35%;background:#D53641"></i>
  </div>
</div>
```
Bars are scaled to **each department's own headcount**, so a small team and a large one compare on
rate rather than size; the headcount sits in the label so nobody mistakes a full red bar for a big
problem when it is four people. Departments do not sum to the population — state that once beneath.

**Never render a bare count table here.** A count with no denominator cannot be read: a raw fail count per department says nothing until you know each department’s headcount. Every department carries `n fail · n pass · N people · n%` and the bar is drawn on the rate.

#### Finding the department family — do not hardcode a prefix
Group naming is per-tenant. `dept-*` is one convention among many, the account may carry several
thousand groups, and the `groupIds` facet truncates around 100 entries — so both "grep for `dept-`"
and "read the facet" quietly drop real departments. Discover the family instead:

**1 · Probe for a family name.** `listPersonnelGroups` takes a case-insensitive substring `q`, so a
`size=1, include_total_count=True` probe costs almost nothing. Fire these in one parallel batch and
read only `totalCount`:
```
q="dept"  q="department"  q="div"  q="org"  q="function"  q="team"  q="bu-"
```
Take the **first token in that order** whose count lands in **3–40 groups**. Above 40 it is an
access or project namespace, not an org chart; below 3 it is not a family. If nothing qualifies,
skip the section entirely — never substitute `app-*`, `oncall-*`, `vendor-*`, or `contractors-*`,
which are access and engagement groups, not departments.

**2 · Fetch the family and collapse duplicate prefixes.** `listPersonnelGroups(q=<token>, size=500)`.
Tenants routinely carry **the same departments under two prefixes** — an older `dept-<name>` and a newer `pg-dept-<name>` are one department, and charting both double-counts it and doubles
the row count. Strip everything up to and including the probe token from each name; where two names
reduce to the same remainder, keep only the variant whose group has the larger headcount and drop
the other. Chart the reduced names (`<name>`, not `pg-dept-<name>`).

**3 · One call per surviving department** — this is exact and beats the facet, which truncates:
```
Drata_listPersonnel(group="<name>", facets=["complianceStatus"], include_total_count=True, size=1)
```
`totalCount` is the headcount; the facet gives PASS / FAIL / EXCLUDED. A ≤40-group family is a
single parallel batch. **Never derive department numbers from the `groupIds` facet** — the small
departments fall off the end of it and vanish from the chart without a trace.

Cap the chart at the 15 largest departments by headcount and say how many were not drawn.

### Onboarding — two charts, both visual
**1 · Cohort bars.** One stacked bar per cohort — 0–30 / 31–60 / 61–90 days since `startedAt` —
segmented fully compliant vs still open, cohort size in the label. Reading down the three bars shows
whether people converge on compliant with tenure or simply never do.

**2 · Time-to-complete per check.** One bar per check, filled to the **share of the last 90 days'
starters who completed it**, with the median days as a label on the right (`MFA — [P]% · median [N] day(s)`).
A check that never completes renders as an empty track labelled `no completions · [n] open` — that
empty bar is the long pole and it should be the most visible row in the section.

### Offboarding — former personnel
**Always render this section as two charts. It is never a note, a sentence, or a "not applicable"
line.** If your numbers say this section is empty, the numbers are wrong — go back to step 2 and
re-pull with the four `FORMER_*` statuses. **Never explain an empty offboarding result**; fix it.

Scope is the **whole former-personnel population** — every
`FORMER_*` status, not a subset. Get the population from `listPersonnel` with
`include_total_count` and all four `FORMER_*` statuses; that `totalCount` is the subtitle figure
(`across [n] former personnel`) and the only baseline.

**The offboarding facet will usually sum to slightly less than the population.** A few people carry
no offboarding check record at all. Do not reconcile by shrinking the baseline to the facet sum, and
do not silently drop the difference: keep the population as the stated denominator and render the
remainder as a fourth `#F2C14F` segment labelled `no offboarding check recorded` whenever it is
non-zero. If the gap is larger than a couple of percent, re-check the status filter before shipping.

**1 · Offboarding status.** One `.sb` stacked bar, same markup as every other bar in this report:
**pass** `#2E4DFF` · **fail** `#D53641` · **excluded** `#D9DCDE`, with the raw counts in the label
(`[N] fail · [M] pass · [X] excluded`) and the in-scope rate — pass ÷ (pass + fail), with excluded
and unrecorded out of that denominator — as a single figure beside it. Say which denominator each
number uses in the label; never present the in-scope rate as a share of the population.

**2 · Ageing of the failures.** One stacked bar of the failing group only, segmented
**0–30 · 31–90 · 90+ days since `separatedAt`**, with the 90+ segment carrying the single ember.
Skip this bar — do not fake it — if `separatedAt` is absent for most of the failing group; say so
in one line instead.

Median days-to-offboard for those that completed goes in the section subtitle, not a table.

**Offboarding tickets are not available.** No personnel record or compliance check carries a ticket id, link, or status. **Never render a ticket column, chart, or count**;
if asked, say the field does not exist rather than substituting the offboarding check for it.

**Never render a data-quality or identity-sync panel** in this report — no synced vs not-synced bar,
no `lastCheckedAt` freshness section. It is instrumentation, not workforce compliance.
```

## Edge cases
| Situation | Handling |
|---|---|
| Asked for the list of who is failing | This report is population-level by design. Give the counts; say the named list is not part of this report |
| A check reads 100% fail | Unconfigured, not failing — grey it, label `no results recorded`, exclude from ordering and from the KPI row |
| Excluded folded into pass | Never. Third segment, and out of the denominator for any rate |
| `complianceChecks` from a list call | Capped at 10 of 15 — `OFFBOARDING` and the training checks are the ones dropped. Use `searchPersonnelCompliance` filters for counts |
| No department-shaped group family | Skip the department chart; do not break down by `app-*` access groups |
| Counts differ between calls | The personnel index is eventually consistent — pull every figure for one report in a single batch |

## Example invocations
- "How compliant is our workforce?"
- "Show training and policy coverage by department."
- "Employees vs contractors — compliance breakdown."
- "How healthy is our onboarding and offboarding?"
- "Workforce compliance report for the board deck."
