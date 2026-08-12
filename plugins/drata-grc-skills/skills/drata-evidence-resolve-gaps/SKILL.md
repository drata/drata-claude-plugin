---
name: drata-evidence-resolve-gaps
description: >
  Guided evidence fixes from one flat menu with live counts — no artifact, automated failing,
  renewal overdue, no control mapped, key info missing — pick a gap, answer one question, confirm
  one previewed batch. Creates or retires items on request. Use for 'fix my evidence', 'clean up
  the evidence library', 'assign evidence owners', 'link evidence to DCF-15'. Write access.
area: Compliance & Audit Readiness
permission: write (create / update / delete evidence)
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listEvidence, Drata_searchControls, Drata_createEvidence, Drata_updateEvidence, Drata_deleteEvidence; Drata_searchMonitoringTests optional (automated-failing context). Requires create/update/delete:evidence scope + a permitting role."
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
   - **Header identity — the customer's logo, top-left, only when it can truly be inlined; else the company name as text.** Call `Drata_getCompany` once per run before rendering (account-scoped, no arguments, read-only; batch it with the run's other independent reads). It returns `name`, `legalName` and `logoUrl`. Resolve the header in this order and stop at the first that succeeds:
     1. **Inlined logo — gate first, then fetch, then verify.** Attempt this step only if `logoUrl` is non-empty **and** the host provides a tool that can actually download raw image bytes from an arbitrary URL. Many sandboxed hosts — including Claude's cloud / Cowork environments — forbid fetching arbitrary CDN URLs, and the Drata image CDN additionally refuses generic fetchers; **in those hosts this step fails immediately and silently, and falling through to the name is the designed outcome, not a degraded render.** Where a download is possible: fetch once (no retries, no proxies, no cache mirrors, never a route around a refusal), verify the bytes decode as a real image (image magic bytes, mime `image/*`, non-zero size), base64-encode **those downloaded bytes with a real encoder in this run**, and emit `<img class="cust" src="data:[mime];base64,[data]" alt="[company name]" onerror="this.style.display='none';this.nextElementSibling.style.display='block'"><div class="custname" style="display:none">[company name]</div>`. **Never type, reconstruct, or approximate base64 from memory — fabricated image data renders a broken or wrong mark exactly where the customer's identity belongs.** If any part of this step cannot be completed and verified, it did not succeed.
     2. **Company name as text.** `<div class="custname">[company name]</div>` — used whenever `logoUrl` is absent or empty, no permitted fetch path exists in this host, the fetch fails or is refused, the bytes are not a decodable image, or the base64 cannot be produced from real downloaded bytes. **This fallback is first-class: a report headed by the company's name in clean type is a correct header; a broken image, an empty header, or invented image data is the only failure.**
     **Never emit `<img src="https://…">`.** A remote reference is not an acceptable third option: artifact sandboxes block external images, and a blocked, expired or access-controlled URL renders a broken-image icon exactly where the customer's identity belongs. It is a verified inlined image or it is the name — nothing in between.
     **When the logo renders, the company name does not appear as visible text** — it lives in the `alt` attribute and in the hidden `onerror` fallback `<div>`, which stays invisible unless the image fails to decode; that hidden div is the safety net, not a second header.
     **Proportions: constrain the height, leave the width free.** `height:32px; width:auto; max-width:200px; object-fit:contain` — a wide wordmark and a square icon then share one baseline with no stretching, squashing or cropping. **Never set `height` and `width` together, never `width:100%`, never a fixed pixel width**, and never re-encode the image to a different aspect ratio. If a logo would exceed `max-width` at 32px tall, `object-fit:contain` shrinks it proportionally — that is correct, do not compensate.
     On the dark board/exec surface, an inlined dark-on-transparent logo disappears; use `<div class="custname" style="color:#fff">[company name]</div>` instead rather than shipping an invisible mark.
   Mini artifact structure: Title naming the operation (`Batch Preview — …` / `Write Receipt — …`)
   → scope line (workspace name — or `All workspaces` — plus register where the skill is register-scoped, and timestamp; always spelled out in full, never an id) → hairline → table(s) → footer
   `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else. Table hygiene
   is the shared contract's: ≤6 columns, one fact per cell, numeric cells `.num`, codes `.code`;
   chips only on true per-row outcomes (`chip ok` written / `chip gap` failed). Deliver like every
   other skill: via the host artifact tool if one exists, else write the HTML to a `.html` file
   named `<skill-name>-<scope>-<YYYY-MM-DD>.html` using this skill's exact folder name
   and send it. Conventions that hold everywhere: no emoji; the word DRATA is plain text; status
   words are exactly Ready / At-risk / Failing; every ratio uses the word `of` (`N of M`),
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
     .drata .cust{height:32px;width:auto;max-width:200px;object-fit:contain;display:block;margin:0 0 14px;}
     .drata .custname{font-weight:600;font-size:15px;letter-spacing:-.01em;color:var(--space);margin:0 0 14px;}
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

# Evidence Fix Menu

## Purpose
One flat menu of the most common evidence gaps, with live counts from a single sweep. Pick a gap;
the skill guides the fix: one question (answers suggested), one previewed batch, one confirmation,
one receipt — then the menu again with fresh numbers. **Follow
`${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md` for every write. Nothing is written until the user
confirms the exact previewed change.** If the user already named a specific change ("set the
pen-test evidence to renew quarterly"), skip the menu and run the fix directly: read → preview →
confirm → write.

## The menu
```
Evidence gaps right now — [workspace]:
  1. No artifact attached ....... [n]
  2. Automated evidence failing .. [n]   (its monitoring test fails)
  3. Renewal overdue ............ [n]   ([x] more due soon)
  4. No control mapped .......... [n]   (feeds no audit)
  5. Key info missing ........... [n]   (owner / description / renewal schedule)
Pick one to fix.
```

## Counting — one sweep, cached for the session
- One call reads the whole library: `Drata_listEvidence(size=500,
  expand=["controls","user","renewalSchemaAndVersions"], include_total_count=true)`, paged only
  past 500 items (narrate progress if it pages) — plus one
  `Drata_searchMonitoringTests(check_result_status="FAILED")` call for line 2, issued in the same
  parallel batch. Everything derives client-side:
  line 1 = no current version, or current type `NONE` · line 2 = current `TEST_RESULT` whose
  backing test is FAILED (matched by version `source` name — an unmatched TEST_RESULT item is
  reported as unmatched, never assumed healthy) · line 3 = `renewalDate` past (due soon = the
  `EXPIRING_SOON` window; spot-check dates — never trust `NEEDS_SOURCE`/`NEEDS_ARTIFACT`
  statuses) · line 4 = `controls` empty · line 5 = owner absent, description empty, or
  `renewalScheduleType` `NONE`.
- Lines 1–3: first match wins; lines 4–5 can overlap them — one footer sentence says so.
- Multi-workspace: pass the same `workspace_id` everywhere; a mismatch returns an empty library,
  not an error. If drata-evidence-identify-gaps ran this session, reuse its buckets — no sweep.

## The fixes
1. **No artifact** — **no file upload exists in the MCP, ever — never promise one.** Per item:
   link a `url` · link a ticketing `ticket_url` (regex-validated) · request the file from its owner
   (message drafted; delivered via the host's messaging tools, only on approval — say so if none
   exist) · or roll several file-bound items into one **upload
   checklist** for a single Drata-UI session (Evidence Library → item → New version).
2. **Automated failing** — never re-file over an automated item: the file isn't the problem, its
   test is. Show the failing test and days failing, offer to message the owner, route to
   drata-monitoring-report.
3. **Renewal overdue** — per item: *is the artifact still accurate?* Yes → re-file (a new
   `url`/`ticket_url` creates a new version, auto-sets `filed_at` to today and resets the renewal
   clock — disclose first; it requires a `renewal_schedule_type` — ask, never default). No →
   collect the replacement, or use line 1's choices. Cadence wrong → re-schedule.
4. **No control mapped** — top-5 candidate controls by name/topic from one narrowed
   `Drata_searchControls` call (*Calculated*). **`control_codes` REPLACES the whole mapping** —
   read current codes, send the union, name anything that would drop; an empty list clears all.
5. **Key info missing** — "Owner, description, or renewal schedule first?" Owner: one name per
   batch (candidates: owners of the controls each item feeds, *Calculated*; pass names verbatim,
   never fabricate an email — ambiguous names return candidates, ask once). Description: drafted
   from name + linked controls, approved before write. Schedule: map wording to a preset
   (monthly → ONE_MONTH … annual → ONE_YEAR, one-off → NONE); `CUSTOM` only with an explicit
   date — presets auto-calculate and **silently strip** an explicit `renewal_date` (report the
   `_note`).

## Flow rules
- Every fix is three beats: one question → one full-list preview (every row, before → after;
  more than 3 rows → mini-branded artifact per rule 1, otherwise chat) → one explicit
  confirmation. Then sequential writes, one parallel read-back pass scoped to the touched
  fields, a receipt (more than 3 rows written → mini-branded artifact per rule 1, otherwise one
  line of chat), and the menu with refreshed counts.
- Batches cap at 25 rows per confirmation; a failed write stops the batch and reports.
- Suggestions and drafts are labeled *Calculated* and applied only after approval. Never invent an
  owner, control code, URL, or description. `name` lookups are a **PREFIX** match.
- On request (not on the menu): create an item (needs `owner` + `renewal_schedule_type` — ask for
  both in one round; `url` **or** `ticket_url`, at most one, or neither for an evidence item) ·
  retire an item (two-call delete: `confirm=false` preview → explicit approval → `confirm=true` →
  verify absence; irreversible, removes every version and linkage, never batched) · message an
  owner (draft shown; delivered via the host's messaging tools, only on approval — say so if none
  exist).
- No write scope → say so and stop; offer drata-evidence-identify-gaps.

## Example invocations
- "Fix my evidence." / "What's broken in the evidence library?"
- "Assign owners to the evidence that has none."
- "Work through the expired evidence."
- "Link the unmapped evidence to the right controls."
- "Create evidence from this ticketing-system ticket, owner jane.doe@example.com, annual renewal." *(direct — no menu)*
