---
name: drata-vendor-resolve-gaps
description: >
  Guided vendor fixes from one flat menu with live counts — key info missing (owner/business
  unit), review overdue (chase), on hold, no renewal schedule — pick a gap, answer one question,
  confirm one previewed batch. Onboards, imports CSV, applies a decision to the vendor record, or
  archives on request. Use for 'fix my vendors', 'assign vendor owners', 'record the Acme Corp decision'. Write access.
compatibility: "Requires Drata MCP with tools: Drata_getCompany, Drata_listVendors, Drata_getVendor, Drata_createVendor, Drata_updateVendor, Drata_deleteVendor; Drata_listVendorSecurityReviews optional (review-history context on request). Requires create:vendor / update:vendor / delete:vendor scope + a permitting role."
metadata:
  area: Third-Party & Vendor Risk
  permission: write (create / update / delete vendor)
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

# Vendor Fix Menu

## Purpose
One flat menu of the most common vendor-record gaps, with live counts. Vendors are
**account-scoped** — never ask which workspace. Pick a gap; the skill guides the fix: one question
(answers suggested), one previewed batch, one confirmation, one receipt — then the menu again with
fresh numbers. **Follow `${CLAUDE_PLUGIN_ROOT}/shared/write-safety.md` for every write. Nothing is
written until the user confirms the exact previewed change.** If the user already named a specific
change ("archive the old Globex record"), skip the menu and run the fix directly: read → preview →
confirm → write.

## The menu
```
Vendor gaps right now (current vendors):
  1. Key info missing ............ count on select   (security owner / business unit)
  2. Security review overdue ..... [n]   ([x] more due soon)
  3. On hold ..................... [n]
  4. No renewal schedule ......... count on select
Pick one to fix.
```

## Counting — probes first, pass only on demand
- **The menu never waits on a paged crawl.** Instant probes, one parallel batch (`size=1`):
  `next_review_deadline=["OVERDUE"]` and `["DUE_SOON"]` for line 2, `status=["ON_HOLD"]` for
  line 3. **Scope every figure to current vendors via the response `breakdown.current`** —
  archived records keep lapsed review dates forever and inflate raw totals several-fold; never
  publish a raw total as the headline.
- **Lines 1 and 4 have no server-side filter and show no number** (`count on select`). The first
  pick of either runs one pass over current vendors — `Drata_listVendors(size=50,
  expand=["vendorUser"])`, 2–8 pages, **progress narrated per page** — computing both at once,
  cached for the session.
- `category` null and the string `"NONE"` both mean *no business unit* — treat them identically.
  An empty `latestSecurityReviews` means *never reviewed*, not *passed*.
- If drata-vendor-identify-gaps ran this session, reuse its rows — no probes, no pass.

## The fixes
1. **Key info missing** — "Owner or business unit first?" Owner: one name per batch (candidates:
   owners of other vendors in the same category, *Calculated*; resolved from id/email/name —
   never fabricated). Business unit: a proposed `category` per vendor from its name and
   `services_provided` (*Calculated*), approved per row or as a set.
2. **Security review overdue — the chase line.** **No MCP write can create, submit, or complete a
   security review — never claim one was scheduled or done; the review itself happens in the
   Drata UI.** What lands here: assign an owner where the row has none (an ownerless review
   cannot be chased) · fix `renewal_date` / `renewal_schedule_type` where the deadline machinery
   is wrong · draft the chase to each owner naming the vendor and deadline (delivered via the
   host's messaging tools, only on approval — say so if none exist), or roll the batch into one
   chase list.
3. **On hold** — per vendor: *what was the hold's outcome?* Release to ACTIVE · REJECTED ·
   ARCHIVED · stay ON_HOLD with a dated note in `notes`. One previewed status write each; name
   the transition in the preview.
4. **No renewal schedule** — pick a cadence (presets, or CUSTOM with an explicit date); batch
   apply.

## Flow rules
- Every fix is three beats: one question → one full-list preview (every row, before → after,
  labeled account-scoped; more than 3 rows → mini-branded artifact per rule 1, otherwise chat) →
  one explicit confirmation. Then sequential writes, one parallel read-back pass scoped to the
  touched fields, a receipt (more than 3 rows written → mini-branded artifact per rule 1,
  otherwise one line of chat), and the menu with refreshed counts.
- Batches cap at 25 rows per confirmation; a failed write stops the batch and reports.
- Suggestions are labeled *Calculated* and applied only after approval. Never invent a field
  value the user did not give; any field you send replaces the stored value.
- On request (not on the menu):
  - **Onboard a vendor** — `name` required; unmentioned fields stay unset.
  - **Import a CSV / pasted list** — propose the column mapping (unmapped columns listed, never
    silently dropped) → dupe-check every name once against `Drata_listVendors` → full preview of
    creates vs updates vs skips → one approval per ≤25 rows → sequential writes → receipt.
  - **Apply a decision to the vendor record** — status (`APPROVED` / `REJECTED` / `ON_HOLD` /
    `UNDER_REVIEW` / `FLAGGED`) + conditions appended to `notes` + owner + next review date, in
    one previewed write — **stating plainly that the security-review object itself is completed
    in the Drata UI.** Risk-register escalation stays in drata-vendor-identify-gaps.
  - **Archive / delete** — delete is two-call (`confirm=false` preview → explicit approval →
    `confirm=true` → verify absence; never batched).
- No write scope → say so and stop; offer drata-vendor-identify-gaps.

## Example invocations
- "Fix my vendors." / "What's missing on our vendor records?"
- "Assign security owners to the vendors that have none."
- "Chase the overdue vendor reviews."
- "Resolve the vendors on hold."
- "Import this vendor CSV." *(on request — no menu)*
