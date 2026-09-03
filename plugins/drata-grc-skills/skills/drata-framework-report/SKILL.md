---
name: drata-framework-report
description: >
  Answer 'how ready is a given workspace for framework X?' — asks workspace first, then which
  in-scope framework (auto-selecting when only one of either exists), then reports framework readiness % with per-area pass rates,
  and the list of not-ready requirements for in-scope frameworks; directional coverage and gap list
  for one you're considering. States what Drata records — no predicted findings, no verdicts. Use for SOC 2 / ISO
  27001 / HIPAA readiness, audit prep, expansion sizing. Gap fixes route to the domain resolve-gaps
  skills. Read-only.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listWorkspaces, Drata_listRequirements, Drata_searchControls, Drata_searchMonitoringTests, Drata_listEvidence, Drata_searchRisks"
metadata:
  area: Compliance & Audit Readiness
  permission: read-only
---

**STOP — ASK BEFORE YOU READ ANYTHING.**

Your first tool call is `Drata_listWorkspaces()`. Nothing else. Not `Drata_listRequirements`, not a
count probe, not "just checking what's in scope".

- **More than one workspace → your next message asks which workspace, and asks only that.** Do not
  name a framework, do not offer frameworks, do not pull data. Wait for the answer.
- **After the workspace is chosen → sweep it for in-scope frameworks.** **Exactly one in scope →
  select it silently and report on it; ask nothing.** State it in the report's source line, not as a
  question — a one-option question is not a choice, it is a round trip that costs the user a turn.
  **Two or more → ask which one**, listing what you found with requirement counts. Wait for the
  answer.
- **Only in-scope frameworks are offerable.** If the user names one that is not in scope in the
  chosen workspace, say so in one line, show the in-scope list, and ask again. **Do not proceed into
  a directional report because they named it** — see the zero-requirements rule below.
- **Exactly one workspace → skip the first question silently** and go straight to the framework step.
- **One workspace and one in-scope framework → ask nothing at all** and go straight to the report.
- **The user named both up front → ask nothing.**

A framework named without a workspace is not enough on a multi-workspace account: the same tag can
be in scope in one workspace and absent in another, so the report would be built against a
denominator the user never chose. **Choosing a workspace yourself, or letting the MCP's own
workspace picker stand in for the question, does not satisfy this** — the user picks, explicitly,
before any read.

**Once they have picked, pass that choice as `workspace_id` on every single call that accepts it** —
`Drata_listRequirements`, `Drata_searchControls`, `Drata_searchMonitoringTests`, `Drata_listEvidence`.
The rule above governs *who decides* the workspace, never *whether you send it*. Omitting it after
the user has chosen is the bug it exists to prevent, not compliance with it: on a multi-workspace
account a workspace-scoped call with no `workspace_id` comes back as an `action_required` **picker
object, not data** — no `data`, no `pagination.totalCount`. A sweep that tests `totalCount > 0` reads
that picker as a zero and concludes the framework is not in scope, which sends a perfectly healthy
tenant into the zero-requirements STOP below. If a call ever returns a picker, you failed to thread
the workspace — resend with `workspace_id`, do not interpret it as an empty result. Pass the
workspace **name** the user gave, verbatim; the server resolves it.

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
     .drata .area{margin-bottom:16px;}
     .drata .arow{display:flex;justify-content:space-between;align-items:baseline;font-size:13px;color:var(--muted);margin-bottom:6px;}
     .drata .arow .nm{color:var(--space);font-weight:500;font-family:'Geist Mono',monospace;font-size:12px;letter-spacing:.04em;}
     .drata .arow b{color:var(--space);font-weight:600;font-variant-numeric:tabular-nums;}
     .drata .grid{display:flex;flex-wrap:wrap;gap:3px;}
     .drata .cell{width:15px;height:15px;border-radius:2px;background:var(--dust);} .drata .cell.on{background:var(--cobalt);}
     .drata .rgrid{display:flex;flex-wrap:wrap;gap:4px;}
     .drata .rq{font-family:'Geist Mono',monospace;font-weight:600;font-size:10px;letter-spacing:.02em;
       line-height:1;padding:5px 6px;border-radius:3px;background:#fff;border:1px solid var(--dust);
       color:var(--muted);white-space:nowrap;}
     .drata .rq.on{background:var(--cobalt);border-color:var(--cobalt);color:#fff;}
     .drata .track{height:22px;background:var(--dust);border-radius:4px;overflow:hidden;}
     .drata .fill{height:100%;background:var(--cobalt);border-radius:4px 0 0 4px;} .drata .fill.ember{background:var(--ember);}
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

# Framework Readiness

## Purpose
One question, one artifact: **how ready is this workspace for framework X?** Workspace first,
then X — X may already be in scope
(the pre-audit question) or be one the tenant is weighing up (the expansion question). Both
produce one deliverable — a readiness scorecard: headline %, the gaps behind it, the plan to
close them. Framework scoping is not writable in Drata: this informs the decision, never makes
it.

## Key tools
- `Drata_listRequirements` — the denominator in both modes: `framework_tag[]`, `is_in_scope`, `is_ready`, `expand=controls`, `size` up to 500. Requirement `is_ready` gives readiness without touching controls.
- `Drata_searchControls` — control readiness and its drivers; in expansion mode, the existing set that may already cover the target.
- `Drata_searchMonitoringTests` — failing checks; `Drata_searchRisks` — open risks with their Drata scores; `Drata_listEvidence` — evidence status per control.

## Non-negotiables — check these before you start and before you send

1. **The deliverable is the branded artifact, not a chat answer — in every mode, including
   expansion and every degraded case.** The only thing that may be plain chat text is a *question*
   (which workspace, which framework) or a one-line "that framework is not in scope here". The
   moment you are presenting findings — tables, counts, headings, an assessment — it renders as the
   artifact. Unusual or partial data is not a licence to fall back to markdown. Build the self-contained HTML
   with the `.drata` theme, the KPI tile and the code-labelled requirement grid. **Never reply
   with markdown prose, a bare table, or a chat-formatted summary** — if you find yourself typing
   `### Overall posture` into the conversation, stop and render the artifact instead. A report that
   arrives as chat text has failed regardless of how good its numbers are.
2. **Never start until workspace and framework are both settled — settled includes silently auto-selected when there is only one option.** Step 1 is a blocking gate, not a
   preamble. Do not pull requirements, controls, evidence or anything else "to get going" while a
   question is outstanding.
3. **One framework per report.** If the user has not chosen one and more than one is in scope, ask — do not substitute an
   all-frameworks table. A cross-framework league table is a different deliverable; offer it, never
   assume it.
4. **Never narrate your own data gathering.** No "I have enough for the report", no "a full
   enumeration would need several more pages", no "let me capture the authoritative count", no
   pagination commentary. The reader wants the finding, not the fieldwork. If coverage is genuinely
   partial, that is one line inside the artifact stating the cap — nothing more.
5. **Never close with an offer.** No "want me to pull the full list into a spreadsheet?", no "or
   drill into a specific framework?". The report ends on its last content block.
6. **The KPI row is ONE tile: `[ready] of [in-scope]` with `· [N]%` on the label line.** Not four,
   not two. **No separate ready / not-ready tiles** (the denominator already carries both), **no
   standalone percentage tile** (it is on the label line), and **no invented aggregate such as
   "areas fully ready"** — that is a metric Drata does not keep, it is derived from a grouping that
   is itself inferred, and in the run that prompted this rule the tile implied near-completion while the report showed two areas, neither of them complete. If a figure is not `ready of in-scope`, it is not a
   KPI tile.
7. **Everything that can be reconciled, must be.** The area rows must sum to the ready and in-scope
   totals; any count of areas stated anywhere must equal the number of area rows rendered. Check
   both before sending — a report whose own sections disagree is worse than one that omits them.
8. **Requirements are the unit.** Control-level counts do not belong in the KPI tile *or* the
   headline sentence. Controls may appear only inside the gap sections, as the cause of an unready
   requirement.

## Workflow

**1. WORKSPACE FIRST, then framework — a blocking gate.** Call `Drata_listWorkspaces()` before
anything else, and make no other tool call until the scope is settled.
**One workspace → skip the question entirely** and never mention that others could exist. **Two or
more → ask which workspace, and ask nothing else in that message.** Only once it is chosen, sweep
that workspace for its in-scope frameworks — never ask the user to name a framework from memory, and
never offer one that is not in scope there:
```
Drata_listRequirements(workspace_id="<the workspace the user chose>",
                       framework_tag=["<TAG>"], is_in_scope=true, size=1, include_total_count=true)
```
one call per tag in a single parallel batch; `totalCount > 0` means it is in scope in that
workspace. **`workspace_id` is not optional here** — without it these probes return pickers rather
than counts and every framework reads as out of scope. A response carrying no `totalCount` is a
picker, not a zero. Then:

- **Exactly one in-scope framework → do not ask. Select it and run.** Name it in the title and
  source line so the user can see what was chosen. **Never present a picker with one option**, and
  never ask a yes/no confirmation of it either ("I found SOC 2 — shall I use it?" is the same wasted
  turn wearing a different hat). Same rule as the one-workspace case, and the same rule
  drata-risk-report applies to a single risk register.
- **Two or more in-scope → present them with their requirement counts and let the user pick.**
  Skipping this question and reporting on every framework at once is the failure this step exists to
  prevent.
- **Zero in-scope → the zero-requirements rule in step 1b applies:** say so and stop; do not
  auto-select something that is out of scope.

If the user already named both workspace and framework, ask nothing.

**1b. DETECT THE MODE.** With the `framework_tag` settled — valid tags are CUSTOM, SOC_2, ISO27001, ISO270012022, ISO27701, ISO277012025, ISO270172015, ISO270182019, ISO270182025, ISO420012023, CCPA, CCPA2026, GDPR, HIPAA, PCI, PCI4, SCF, NIST80053, NISTCSF, NISTCSF2, NISTAI, NIST800171, NIST800171R3, CMMC, MSSSPA, MSSSPA11, FFIEC, COBIT, SOX_ITGC, CCM, CYBER_ESSENTIALS, CYBER_ESSENTIALS_32, FEDRAMP, FEDRAMP20X, DRATA_ESSENTIALS, CIS8, HITRUST, DORA, NIS2, ESSENTIAL_EIGHT, NYDFS, TISAX, CPS230, CYFUN, AIUC_1

**Use these tags exactly.** An unrecognised tag returns **HTTP 400, not an empty result**, so a sweep with a guessed tag errors mid-batch instead of skipping. `ISO42001` is wrong — it is `ISO420012023`; `ISO27018` is `ISO270182019`. If a 400 comes back, read the enum out of the error message and retry rather than dropping the framework.

**This list goes stale — Drata adds frameworks every few releases, and the newest ones are exactly the ones a customer has just adopted.** A tag missing from the list above does **not** error; it is simply never probed, so the framework silently reads as not in scope and the zero-requirements rule below then refuses the report. So: if the user names a framework you do not recognise, **probe it anyway** rather than declaring it out of scope — pass their term as the tag, and if that 400s, read the accepted values out of the error and match against those. The error message is the authority on what exists; this list is only a starting set.

Probe scope:
`Drata_listRequirements(workspace_id="<chosen workspace>", framework_tag=["<X>"], is_in_scope=true, size=500)`. Rows →
**IN-SCOPE MODE**. None → re-run with `is_in_scope=false` (same `workspace_id`).

- Rows there → **EXPANSION MODE**, but only if the user has explicitly asked about adopting the
  framework. Otherwise stop and re-offer the in-scope list.
- **Zero rows in both states → STOP. Produce no report.** The framework is not loaded in this
  workspace at all, so there is nothing to measure against. **First rule out the two ways this
  conclusion is reached wrongly:** a missing `workspace_id` (the response is an `action_required`
  picker, not a zero) and a tag absent from the starting list above (never probed at all). Both look
  exactly like "not in scope" and neither is. Only after both are excluded is the zero real. Then
  say that in one line, list the
  frameworks that *are* in scope, and ask which one they want. **Never substitute
  framework-agnostic control posture** — total controls, % ready, controls without evidence, failing
  tests — as a stand-in. Those numbers describe the workspace's SOC 2 control library, not the named
  framework; presenting them under an ISO 27001 heading tells the reader something about ISO 27001
  that the data does not support. A workspace-wide control summary is drata-control-identify-gaps's job. **Name the mode and its denominator in
the first two lines of output** — they are different numbers and confusing them is the most
misleading error this skill can make.

| Mode | Denominator | Numerator | Label |
|---|---|---|---|
| In-scope | in-scope requirements for X | requirements with `is_ready=true` | *Tool Calls* — Drata's grading |
| Expansion | ALL of X's requirements (`is_in_scope=false`) | requirements a *satisfied* control covers | *Calculated* — directional |

Never present an expansion % as in-scope framework readiness, or in-scope readiness against the full
catalog. Two frameworks in one ask → run once each, scorecards separate.

**2. IN-SCOPE MODE.** Readiness % = ready ÷ in-scope requirements (*Tool Calls*); group by code
prefix (SOC 2 TSC, CSF function, 800-53 family) for per-category pass rates (*Calculated*).
For WHY, run **one** widest-gap probe — `Drata_searchControls(is_ready=false, has_evidence=false,
size=1)` — and report its `pagination.totalCount` as a **single line** ("[k] of [N] not-ready
controls have no evidence at all"). Stop there: the full four-dimension driver analysis, the
ready-vs-not-ready comparison is **drata-control-report's** deliverable; the ownership caveat is **drata-control-identify-gaps's** —
route the user there rather than reproducing them here. Structured filters only —
`query` switches to a semantic endpoint that ignores them. **`is_ready=false` is "failing";
`has_passing_test=false` is not the same thing.** Findings:
`Drata_searchMonitoringTests(check_result_status="FAILED")` (cite `testId`) and
`Drata_searchRisks(status=["ACTIVE"], inherent_score_gte=7)`.

**3. EXPANSION MODE.** Denominator = X's total requirements from the `is_in_scope=false` sweep.
Pull the existing set with `Drata_searchControls(size=50, expand="requirements")`, paginated.
**Covered** = a target requirement a *satisfied* control maps to — satisfied meaning current
evidence or a passing test, not merely mapped. Controls are rarely pre-mapped to an unadopted
framework's codes, so requirements shared with in-scope frameworks are the main signal.
Readiness % = covered ÷ total target requirements (*Calculated*, directional); the **gap list**
— each uncovered requirement with its closest control (or "none") — is
the defensible output, the % only the headline. **Fallback:** if X's requirements aren't
enumerable, say so, then map controls against what X shares with in-scope frameworks plus model
knowledge of its structure, labeled *Calculated*. **Never report a % without a real denominator.**

**4. Assemble.** One skeleton, both modes; render only what the mode populates. Counts are
*Tool Calls*; groupings, coverage and estimates *Calculated*. A mapped control or a single
passing test is **not** an attestation.

## Visuals

**Per-area readiness is a progress bar.** Each area renders as a `--dust` track with a `--cobalt`
fill at `ready ÷ total`, labelled **`N of M ready · P%`** — count, total, the word *ready*, then the
percentage. Never `N/M`, never a bare percentage with no bar, never a number without its
denominator. **The overall figure is not a bar** — it lives in the KPI tile and in the grid key.

**The fill is `#2E4DFF`. Never gold, amber, olive or any colour outside the Drata palette** — an
off-palette bar reads as a status signal the report never defined, and readiness is not a status
tone. The percentage may follow the count on the same label line; it never replaces it.

**Lead conservatively: the first two visuals never depend on grouping.** Drata stores no grouping
field, so any area breakdown is inferred — it must never be the thing the report opens with.

**No overall readiness bar.** The KPI tile already states `N of M · P%`; a full-width bar directly
beneath it repeats that and nothing else. The grid is the first visual.

**1 · The requirement grid, ungrouped — every tile carries its requirement code.** A blank coloured
square tells the reader only how many are ready, which the KPI tile already said; the same square
with `CC6.1` in it tells them *which*, and that is the whole point of the visual. **Never render an
unlabelled `.cell` swatch grid in this report.**

- One `.rq` tile per requirement, **its Drata code in 10px Geist Mono inside the tile**, in code
  order (longest-prefix sort, so `CC1.1` … `CC9.2`, then `A1.1`) — not ready-first. Code order means
  a reader can look up any requirement; state is carried by the fill, not by position.
- **Ready = solid `#2E4DFF` with white code. Not ready = white tile, `--dust` border, `--muted`
  code.** No status tones here — readiness is Drata's grade, not a pass/fail signal, and a wall of
  red is not the finding.
- Every tile keeps a `title` with the full code, requirement name and state, so hovering gives the
  name the tile has no room for.
- **The grid carries its own readiness figure** — a key beneath it reading `Ready N · P%` and
  `Not ready M · Q%`, so the grid stands on its own if it is the only thing a reader looks at.
- **Long codes: strip one shared leading prefix and say so in the key.** HIPAA `164.308(a)(3)` →
  `308(a)(3)` with `codes shown without the shared 164. prefix` in the key line. Never truncate
  mid-code with an ellipsis, and never abbreviate a prefix that is not shared by every tile.

This is the honest default: it shows every requirement individually, names it, and assumes nothing
about structure.

**Above 120 requirements the labelled grid stops fitting — drop the ready tiles, not the labels.**
Render only the not-ready requirements as `.rq` tiles (they are the actionable set), keep the key
stating both totals, cap at 120 tiles and close the key with `· N more`. A grid of unlabelled
squares is never the answer to a grid that got too big.

**2 · The per-area breakdown — only when grouping is trustworthy.** A *separate, secondary* section
below the ungrouped grid, never a replacement for it. Render it only when **both** hold: the
framework has a published grouping the codes map to cleanly (SOC 2's five categories, ISO 27001's
Annex A themes) **or** the codes carry a real family (NIST letter families, KSI, HIPAA sections);
**and** it passes the density check — areas under 60% of requirement count, at least two
requirements per area on average. Otherwise omit the section entirely and say nothing about areas.
Name the derivation in the section subtitle ("by Trust Services Category", "by code family") so the
reader knows where the grouping came from.

**Above ~250 requirements** even the not-ready tiles stop being scannable: keep the KPI tile, and
replace the grid with per-area bars **only if** the grouping test above passes. If it does not, show
the top areas by gap as a plain table instead — never a grid nobody can read.

Markup — labelled grid with its own key:
```html
<div class="rgrid">
  <span class="rq on" title="CC1.1 — Control environment · ready">CC1.1</span>
  <span class="rq" title="CC1.2 — Board independence · not ready">CC1.2</span>…
</div>
<div class="key">
  <span><span class="sw" style="background:#2E4DFF"></span>Ready <b>[N]</b> · [P]%</span>
  <span><span class="sw" style="background:#fff;border:1px solid #D9DCDE"></span>Not ready <b>[M]</b> · [Q]%</span>
</div>
```
Per-area row (secondary section only) — **never inside a `<table>`**:
```html
<div class="area">
  <div class="arow"><span class="nm">Availability</span><span><b>[N]</b> of [M] ready · [P]%</span></div>
  <div class="track"><div class="fill" style="width:50%"></div></div>
</div>
```

**A progress bar is never a table cell.** Putting a `.track` in a `<td>` next to its own numbers is
what produced `N of M` colliding with the bar: the cell has no room to give, so the text and the
fill overlap. The area block puts the label on its own line **above** a full-width track — label
row, then bar, stacked, never side by side. If you are reaching for a table to lay out areas, use
the `.area` blocks instead; a table is for the not-ready requirement list, which has no bars in it.

**No other charts.** No donut or gauge for the readiness % (the KPI and the bar both carry it), no
radar over areas, and no trend — the MCP exposes no history.

## Output format
```
## Framework Readiness — [Framework] · [date]
**Mode:** In-scope | Expansion (not yet adopted)
**Readiness [N]%**  Denominator: [X] [in-scope reqs | total target reqs]

### KPI — exactly one tile
`[ready] of [in-scope]` with `· [N]%` on the label line. **Nothing else belongs in the KPI row of
this report** — no control counts, no evidence %, no monitoring or test figures, no risk counts.
Those are other skills' headlines and they dilute the one number this report exists to give.
(Expansion mode: `[covered] of [target reqs]`, labelled *Calculated · directional*.)

### Requirement grid — see "Visuals" above
Ungrouped grid first; the per-area breakdown is a secondary section, only when grouping is
trustworthy. **Use the framework's own published grouping — the one an auditor and a reader
would name — not the raw code prefix.** `category` was null on every requirement observed, so
the grouping is derived from the code, but the derivation must land on real framework structure.

**SOC 2 — always the five Trust Services Categories, never the criteria series.** Grouping by raw
prefix produces ~15 buckets (CC1…CC9, A1, C1, PI1, P1…P8), which is not how SOC 2 is scoped or
discussed and reads as an error:

| Code starts with | Area |
|---|---|
| `CC` | Security (Common Criteria) |
| `A` | Availability |
| `C` (not `CC`) | Confidentiality |
| `PI` | Processing Integrity |
| `P` (not `PI`) | Privacy |

**Match longest-prefix first** — `CC` before `C`, `PI` before `P` — or Confidentiality will swallow
the Common Criteria and Privacy will swallow Processing Integrity. Only categories actually in scope
appear; a workspace scoped to Security alone shows one area, and that is correct.

**ISO 27001:2022** — Annex A themes: `A.5` Organizational · `A.6` People · `A.7` Physical · `A.8`
Technological, with clauses 4–10 as *Management system* if present.

**HIPAA** — codes arrive as `164.308(a)(3)`, `164.412` (no `§`). Group on the section number:
`164.308` Administrative Safeguards · `164.310` Physical Safeguards · `164.312` Technical Safeguards
· `164.314` Organizational Requirements · `164.316` Policies & Documentation · `164.4xx` Breach
Notification.

**NIST 800-53 / CMMC / NIST 800-171** — the letter family in the code is the grouping: `AC-2` →
**AC (Access Control)**, `AU-6` → **AU (Audit and Accountability)**. **NIST CSF** — the function
prefix: `ID.AM-1` → **Identify**, `PR`, `DE`, `RS`, `RC`. These are code-derived and safe.

**FedRAMP 20x** — the KSI family: `KSI-SVC-03` → **KSI-SVC**.

**Do not group when the codes carry no family.** GDPR requirements come back as bare article
numbers (`47`, `49`), CCPA as section numbers. Prefix logic on those produces one bucket per
requirement, which is noise wearing the costume of structure. **Run this check before rendering
groups: if the number of areas is 60% or more of the number of requirements, or any grouping would
average fewer than two requirements per area, drop the grouping and render one ungrouped block.**
Chapter or title roll-ups for these exist in the standards but not in Drata's data, and guessing
them silently is worse than not grouping.

**Sourcing.** `category` was null on every requirement observed, across every framework tested — so **the API supplies no grouping and never will resolve this for
you**. Code-derived families (NIST letter families, KSI, HIPAA sections) are read straight from the
data. The named roll-ups (SOC 2's five categories, ISO 27001's Annex A themes) come from the
published standard, not from Drata: use one **only** where the standard's grouping is unambiguous
and the code maps to it cleanly. Anywhere else, fall back to the code family, label the section
**"by code prefix"**, and **never invent area names**.

### Not-ready requirements
One row per not-ready requirement, straight from Drata: `[code] · [name] · [area]`. **No severity
label, no "critical" or "significant", no effort sizing, no predicted finding, no distinct-controls
count, no auditor-request list.** State which requirements Drata grades not ready; that is the fact
the report owns.

For the controls behind each not-ready requirement, run drata-control-identify-gaps; for evidence
currency, drata-evidence-identify-gaps.

**Nothing after this line.** No "Evidence & monitoring signal" section — evidence and failing-test
counts are drata-evidence-identify-gaps and drata-monitoring-report headlines, and repeating them here
re-introduces exactly the control/test figures the KPI rule bars. No effort or time-to-readiness
estimate: weeks-to-audit is a guess dressed as a finding, and the assumptions it rests on are not in
Drata. No board-ready summary — the whole artifact is the summary. No 30/60/90 roadmap: sequencing
work is gap identification, not reporting.
```

## Edge cases
| Situation | Handling |
|---|---|
| "What would it take to add X" but X is in scope | Already adopted — run in-scope mode and say so; the readiness % is the answer |
| Target requirements not enumerable, or tag unknown | Confirm the tag; if it doesn't exist, use the fallback (shared requirements + model knowledge) labeled *Calculated* — no fabricated % |
| Controls not mapped to the target's codes | Expected in expansion mode; infer via shared requirements and state the inference |
| Readiness % challenged | In-scope % is *Tool Calls* from requirement `is_ready`; expansion % is *Calculated* and directional — the gap list is the defensible output |
| Historical trend or "progress since last quarter" | **No snapshots** exist — point-in-time only; offer to baseline now and compare against a prior report the user supplies |
| A section's read errors, is empty, or won't group | Note it inline, collapse to one aggregate row, keep the rest |

## Example invocations
- "Drata, are we ready for our SOC 2 Type II audit?"
- "Give me an ISO 27001 readiness scorecard with per-domain pass rates."
- "Which HIPAA requirements are not ready in Drata?"
- "How close are we to ISO 42001 based on our current controls?"
- "We have SOC 2 and ISO 27001 — what would it take to add PCI DSS?"
- "Give me the gap list to expand to GDPR."
