---
name: drata-control-resolve-gaps
description: >
  Guided control fixes from one flat menu with live counts — nothing mapped, missing a policy,
  evidence not current, tests failing, key info missing — pick a gap, answer one question, confirm
  one previewed batch (mapping lists are replacements). Creates controls and maps
  requirements/policies/tests on request. Use for 'fix my controls', 'assign an owner', 'map this
  control to CC6.1'. Read-only diagnosis lives in drata-control-identify-gaps. Write access.
area: Compliance & Audit Readiness
permission: write (create / update control)
compatibility: "Requires Drata MCP with tools: Drata_searchControls, Drata_listRequirements, Drata_listPolicies, Drata_createControl, Drata_updateControl; optional for the evidence and monitor fixes: Drata_listEvidence, Drata_createEvidence, Drata_updateEvidence, Drata_searchMonitoringTests (absent → context-carrying handoff to drata-evidence-resolve-gaps). Requires create:control / update:control scope + a permitting role."
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Output — chat for dialogue, mini-branded HTML for batch artifacts.** This skill is
   transactional: menus, questions, the confirmation prompt, and every single-object or ≤3-row
   preview and receipt render as plain chat text — no artifact, no styling pass; the confirm
   loop always stays in chat. Exactly two moments render branded HTML using the mini theme below:
   - **A batch preview of more than 3 rows** — the exact before → after diff as a real `<table>`
     inside a `.panel`, rendered alongside (never instead of) the chat confirmation prompt; the
     confirmation question itself is never inside the artifact.
   - **The receipt of a batch write of more than 3 rows** — what changed, per-row outcome,
     read-back verification.
   Mini artifact structure: Title naming the operation (`Batch Preview — …` / `Write Receipt — …`)
   → scope line (workspace/register + timestamp) → hairline → table(s) → footer
   `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else. Table hygiene
   is the shared contract's: ≤6 columns, one fact per cell, numeric cells `.num`, codes `.code`;
   chips only on true per-row outcomes (`chip ok` written / `chip gap` failed). Deliver like every
   other skill: via the host artifact tool if one exists, else write the HTML to a `.html` file
   and send it. Conventions that hold everywhere: no emoji; the word DRATA is plain text; status
   words are exactly Ready / At-risk / Failing; every ratio uses the word `of` (`55 of 241`),
   never a slash; never invent a severity, score, or verdict Drata does not store.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Mini theme — a strict subset of the shared `.drata` base theme (same selectors, same values, nothing new); embed once per artifact:**

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
     .drata .panel{background:#fff;border:1px solid var(--dust);border-radius:4px;padding:6px 18px;margin:14px 0;}
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
     .drata .logo{display:flex;align-items:center;gap:6px;font-family:'Geist Mono',monospace;font-weight:600;
       font-size:15px;letter-spacing:.04em;color:var(--space);}
     .drata .logo svg{color:var(--space);height:16px;width:auto;}
     .drata .foot{display:flex;justify-content:flex-start;border-top:1px solid var(--dust);
       margin-top:22px;padding-top:12px;}
     </style>
     ```
2. Source labelling (`Calculated` / `Tool Calls`): `${CLAUDE_PLUGIN_ROOT}/shared/accuracy-and-sources.md`
3. **Data pulls — batch independent queries in parallel.** When this skill's workflow lists multiple MCP calls whose inputs do not depend on another call's results — count probes, per-facet or per-flag `size=1` calls, separate FAILED vs ERROR pulls, per-framework or per-workspace probes — issue them together as one parallel batch instead of one at a time; this is the single biggest speed win for report and identify-gaps runs. Keep sequential only what is genuinely dependent: any call whose filter, ID, or scope comes from a prior result (resolve the workspace or register first, expansions of found rows), and the entire preview → confirm → write → read-back chain in resolve-gaps skills. Never parallelize writes, and never let batching change a query's filters. In batch mode, the after-write read-backs may run as one parallel batch once the final write lands — writes themselves never parallelize.
4. Write safety (mandatory for every create / update / delete): `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md`

# Control Fix Menu

## Purpose
One flat menu of the most common control gaps, with live counts. Pick a gap; the skill guides the
fix: one question (answers suggested), one previewed batch, one confirmation, one receipt — then
the menu again with fresh numbers. **Follow `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md` for
every write. Nothing is written until the user confirms the exact previewed change.** If the user
already named a specific change ("put Alice on DCF-15"), skip the menu and run the fix directly:
read → preview → confirm → write.

## The menu
```
Control gaps right now — [workspace]:
  1. Nothing mapped ............ [n]   (no evidence, no policy, no monitor)
  2. Missing a policy .......... [n]
  3. Evidence not current ...... [n]
  4. Tests failing ............. [n]
  5. Key info missing .......... count on select   (owner / description)
Pick one to fix.
```

## Counting — parallel probes, never a loop
**The menu must never wait on a paged crawl.** Four of the five lines have server-side filters,
so their counts come from **one parallel batch of four `size=1` probes** — no expands, read
`pagination.totalCount` from the first page:
line 1 `has_evidence=False, has_policy=False, is_monitored=False` · line 2 `has_policy=False` ·
line 3 `is_ready=False, has_evidence=True, is_monitored=False` · line 4 `is_monitored=True,
has_passing_test=False`. Issue all four together — never one at a time — and render the menu the
moment they return (one round trip, 4 calls).
- **Line 5 has no server-side filter and shows no number** — it reads `Key info missing — count
  on select`. Only if picked does it cost anything: one paged pass (`size=50, expand=["flags"]`),
  **narrating progress per page** ("page 3 of ~8…"), stopping past ~300 rows with `≥N`.
- A control can sit in more than one line (line 2 includes line 1's controls) — one footer
  sentence says so; the numbers are each filter's own `totalCount`, not a partition.
- On a pick of lines 1–4, fetch only the **first page** of rows (`size=15`, the same server
  filter, no expands). Cursor pagination is sequential by nature — that is exactly why the menu
  is probe-based; any flow that must page narrates progress and offers to stop early.
- Cache everything for the session; refresh only lines a write touched. If drata-control-identify-gaps
  ran this session, reuse its rows — skip even the probes. **Never use `query` for any of
  this** — semantic mode ignores every filter.

## The fixes
1. **Nothing mapped** — one question: attach a *policy*, a *monitoring test*, or *evidence*? Then
   top-5 candidates by name/topic from one call (`Drata_listPolicies` /
   `Drata_searchMonitoringTests` / `Drata_listEvidence`, *Calculated*) → batch preview → confirm →
   write (`policy_names` / `test_ids` via `Drata_updateControl`; evidence links via
   `Drata_updateEvidence(control_codes=union)` — the link lives on the evidence record).
2. **Missing a policy** — "Which policy?" with the closest name matches. Batch map via
   `policy_names`.
3. **Evidence not current** — show each control's mapped evidence and renewal dates
   (`Drata_listEvidence(control_code=…)`); per item: re-file a new URL/ticket (a new artifact
   resets `filed_at` and the renewal clock — disclose first, and it requires a renewal schedule —
   ask, never default) · re-schedule · or message the evidence owner. Evidence tools absent → hand
   off to drata-evidence-resolve-gaps naming the control and items.
4. **Tests failing** — **no test re-run or repair exists in the MCP; never pretend otherwise.**
   Show which tests fail and for how long (`Drata_searchMonitoringTests(check_result_status=
   "FAILED")` matched to the picked controls), offer to message the control owner, route diagnosis
   to drata-monitoring-report.
5. **Key info missing** — "Owners or descriptions first?" Owners: one name for the batch
   (candidates: who owns similar controls in the same framework, *Calculated*; pass names
   verbatim, never fabricate an email). Descriptions: drafted from the control's own
   name/question/activity, shown for edit/approval, never written unedited.

## Flow rules
- Every fix is three beats: one question (answers suggested) → one full-list preview (every row,
  before → after — never `…and 14 more`; more than 3 rows → mini-branded artifact per rule 1,
  otherwise chat) → one explicit confirmation. Then sequential writes, one parallel read-back
  pass scoped to the touched fields, a receipt (more than 3 rows written → mini-branded artifact
  per rule 1, otherwise one line of chat), and the menu again with refreshed counts.
- Batches cap at 25 rows per confirmation; a failed write stops the batch and reports.
- Mapping lists (owners, policies, requirements, tests, reports) **REPLACE**: read current
  members, send the union, name anything that would drop. "Add X" is never sent as `[X]` alone.
- Suggestions and drafts are labeled *Calculated* and applied only after approval. Never invent an
  owner, code, URL, or description.
- Resolve write targets by exact `code` through structured retrieval — **never `query`**, for the
  read-back either (a semantic match can hit the wrong control and report a false MATCH).
- On request (not on the menu): create a control (name/code/description from the user — never
  invented), map requirements (`Drata_listRequirements`), edit custom fields, "how many…"
  (answered from the cached probes or pass), message an owner (draft shown; delivered via the host's messaging tools, only on
  approval — say so if none exist). No
  delete-control tool exists — deletion is out of scope.
- No `update:control` scope → say so and stop; offer drata-control-identify-gaps.

## Example invocations
- "Fix my controls." / "What's missing on our controls?"
- "Assign owners to the controls that have none."
- "Map policies to the controls missing one."
- "Which tests are failing on my controls, and who do I chase?"
- "Put Alice on DCF-15." *(direct fix — no menu)*
