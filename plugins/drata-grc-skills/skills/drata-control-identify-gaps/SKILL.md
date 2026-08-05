---
name: drata-control-identify-gaps
description: >
  Two worklists over the not-ready controls in a workspace - controls with nothing mapped (no
  evidence, policy or monitor), and controls whose manual evidence is not current - plus a metadata
  pass over every in-scope control listing those with no owner or no description. Lists control
  codes, not charts. No counts or totals; readiness numbers live in drata-control-report. Use for
  'which controls have nothing mapped', stale evidence, controls with no owner, control gap
  worklists. Fixes -> drata-control-resolve-gaps. Read-only.
area: Compliance & Audit Readiness
permission: read-only
compatibility: "Requires Drata MCP with tools: Drata_searchControls"
---

**Shared protocols — load these from the plugin root, not the current directory.**

1. **Rendering — branded, always. This is the default; never ask the user to pick an output mode.** Read `${CLAUDE_PLUGIN_ROOT}/shared/drata-brand-kit.md` and render every substantive deliverable (dashboard, report, briefing, gap worklist) in Drata branding: a self-contained HTML document using its §3 `.drata` theme. **Deliver it as HTML, always.** If the host has an artifact tool, render it there. If it does not, **write the complete HTML to a `.html` file and send that file** — every environment this runs in can deliver a file. **There is no markdown fallback.** Emitting the report as chat markdown, a bare table, or `###` headings is a failure of the deliverable, not a graceful degradation, and "the host had no artifact tool" is not a reason to do it. The only exception is the explicit text-only opt-out in rule 2. Match effort to the ask — short factual answers stay inline per §4. Two elements of a styled deliverable are a binding contract, even if the brand kit could not be read:
   - **Chart colors: Drata palette only, set explicitly in every chart config — never a library default.** First or single series `#2E4DFF`; multi-series ramp `#BEDAFF` → `#2E4DFF` → `#0F161A`; status tones `#00779C` pass / `#F2C14F` at-risk / `#D53641` fail, only on values that truly pass or fail; one `#FF410C` highlight per view at most; axis and label text `#828B8F`. Every heat map or matrix (risk 5×5, inherent × residual, any coverage grid) uses one band scale: low `#BEDAFF`, mid `#F2C14F`, high `#D53641`, at most one worst cell `#FF410C`.
   - **Table hygiene:** one fact per cell — never a chip, code list and number together; never two categories slash-merged into one row. Numeric cells `class="num"`; codes `.code`, never wrapped. Chips mark real pass/fail only — a count like "2 of 5 mapped" stays neutral ink. No Status/severity/health column that only re-buckets a count. Commentary: last column, one sentence, only where it adds signal. **Caps: 6 columns, 15 rows.** Fold rank into the lead cell ("1 · Acme") or a metric pair into `9.1 (−7.3)`; drop the weakest column rather than cram. Past 15 rows show 15 and close with "13 more — full list on request". **Columns need the theme's `18px` right gutter** — override it to `padding:… 0` and a `.num` column collides with its neighbour, headers merging into `INTEGRATIONCONTROLSCODES`. Cells are top-aligned. Wrap every table in `<div class="panel">`. **KPI tiles are uniform or they are wrong.** Every tile in a row carries its denominator in the figure — full-size numerator, then `of N` in a muted `<span class="den">`. Always the word `of` — `55 of 241`. **Never `/`, never `X/Y`, never `55/241`**, anywhere a ratio appears: KPI tiles, bar labels, table cells, body text and headings all use `of`. This is the house standard across every skill; a slash in one report and `of` in the next is the inconsistency this rule exists to prevent. **Never move the denominator into the label** (`Controls not ready (of 622)`) — the label names what is counted and nothing else, phrased the same way on every tile. **Every tile in a row counts the same polarity**: choose healthy-of-total or needs-attention-of-total once and hold it across the row, so no reader has to work out that one figure is progress and its neighbour is a problem. A figure with no available denominator does not belong in the KPI row. Never invent a score scale Drata lacks — no 0–100 health score, no weighted total, no points column; rank on real Drata numbers. **A count and a list of codes are two facts — two columns (`CONTROLS`, `CODES`), never one crammed cell. Never use `+N`**: show two codes, then words — `DCF-48, DCF-49 and 3 more`.
   - **The Output format section defines content and order, never the medium.** In branded HTML its headings become styled sections, its `>` blocks become rows, its tables become real `<table>` markup — never raw markdown inside an artifact. Every Now · Next · Watch pointer is exactly `<div class="item">…text… <span class="route">drata-x-identify-gaps</span></div>`: class `item`, no bullet element (the theme's `.item::before` paints it and pins it to line one); `item ember` for the single most-critical row only. The last content block runs straight into the footer hairline — no trailing recap, `Go deeper`, `Onward`, methodology, caps, or source-label block.
   - **The footer carries the Drata icon — paste this exact SVG inline** (color `#0F161A` on light surfaces, `#fff` on dark; never an image path, emoji, or substitute glyph): `<span class="logo"><svg xmlns="http://www.w3.org/2000/svg" width="22" height="16" viewBox="0 0 180.207 130.069" fill="none" role="img" aria-label="Drata"><path d="M 103.38 0 C 148.015 0.025 180.207 25.601 180.207 65.121 C 180.182 104.616 147.966 130.119 103.331 130.069 L 48.81 130.069 L 48.785 130.045 L 83.338 98.542 L 101.782 98.542 C 126.645 98.566 146.073 88.901 146.098 65.071 C 146.122 41.241 126.694 31.552 101.831 31.552 L 83.411 31.552 C 83.316 31.464 49.165 -0.038 48.859 0.246 C 48.859 0.246 48.859 0.021 48.859 0 L 103.38 0 Z M 48.718 30.791 C 58.604 45.595 71.908 55.875 88.409 61.9 L 97.386 65.023 L 88.385 68.122 C 71.883 74.123 59.316 84.403 48.668 99.183 C 38.782 84.378 25.478 74.098 8.977 68.073 L 0 64.95 L 9.001 61.852 C 25.502 55.851 38.832 45.571 48.718 30.791 Z" fill="currentColor" fill-rule="nonzero"/></svg></span>`
   - **Structure:** customer identity → Title → source line (`Pulled from Drata · <timestamp> · <workspace>`) → hairline → real `<table>` markup → Now · Next · Watch → footer. **This skill has no KPI row** — see the no-totals rule below. **The footer is exactly `<div class="foot"><span class="logo">[icon SVG]</span></div>` and nothing else** — no wordmark, tagline, product name, permission label, workspace, timestamp, chrome, caption, link or routing line. The header is the customer's identity; the Drata mark never goes there.
   - **No opinions, no predictions, no verdicts.** Report what Drata records and what you counted from it. **Never predict what an auditor will ask for, flag, or accept**; never label a gap *critical*, *significant*, *likely finding*, or *high risk* on your own authority; never size effort (S/M/L, hours, weeks) or estimate a date; never declare anything *audit-ready*, *compliant*, *certification-ready*, or *passing*; never interpret what a regulation or clause requires. Drata's own fields — `is_ready`, statuses, scores, dates, counts — are reportable as-is; ordering rows by those real numbers is fine, and a derived figure is labelled *Calculated*. **A reader must be able to act on this report without inheriting a judgement you made up.** If a sentence would not survive an auditor asking "where in Drata does that come from?", cut it.
   - **The artifact title names this skill's job, and no other skill's.** Title it after what this skill produces — an executive report says `Executive Report`, a gap worklist names the gaps it covers. **Never borrow a generic label like `Compliance Briefing`**: two skills wearing one title leaves the reader unable to tell which one they ran, and it collides with any similarly named skill the user has installed. Scope and date follow the title; nothing else does.
   - **Never emit a section you did not populate.** No placeholder heading, no "not included in this run", no "ask and I'll add it" offer, no note explaining which figures were not pulled. Either pull the data and render the section, or leave the section out entirely — a heading whose body apologises for itself costs the reader attention and returns nothing. The only disclosure that stays is a domain the skill *tried* to read and could not (permission denied), which is reported as one line, not a section. **Never explain what Drata does not store.** No "Drata has no asset object", no "there is no review-date field", no "the API does not expose X" — the absence of a field is your constraint while building, never a sentence in the deliverable. Where a field genuinely does not exist, answer with the nearest real Drata data, name it for what it actually is, and label it *Calculated* if you derived it; say nothing about the field you wanted and did not find. The reader came for their compliance posture, not for a tour of the data model.
   - **Vocabulary — never print "trip", "tripped", "tripping", or "the trip".** Not in a KPI label, a heading, or body text. Name what actually happened: *failing test*, *evidence needs artifact*, *policy not published*. Internally the model is unchanged; only the word is banned.
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

# Control Gap Identification

## Purpose
Two worklists over the not-ready controls in a workspace, and nothing else:

1. **Nothing mapped** — no evidence, no policy, no monitor. There is no artifact to fix, so the work
   is to attach one.
2. **Manual evidence mapped but not current** — evidence exists and no monitor is doing the job, so
   somebody has to refresh it on a cadence.

Then a second pass over the same controls for **metadata that should be filled in** — owner,
implementation narrative, and the account's own custom fields.

**No totals.** This skill never prints a readiness count, a percentage, a KPI row, or a
ready-vs-not-ready figure. Those belong to drata-control-report. A count here invites a reader to
reconcile it against the Drata UI, and the two do not always agree — see the note below. This is a
list of things to go and do.

## The API and the UI can disagree on readiness — never publish a count here
On a live workspace the API returned **232 ready / 174 not ready** while the UI showed
**233 / 173**, both summing to 406 enabled controls. So the disagreement is not a missing record: it
is **one control the two surfaces classify differently**. Every record the API returned self-reported
`flags.isReady: false`, none archived, none duplicated — the filter is right, the classification of a
single control differs.

**Consequence for this skill:** work from the rows, name the controls, and let the reader act on
them. **Never assert "there are N not-ready controls"** — if the number is one off from their
screen, the whole page loses credibility over a row that was going to be worked anyway.

## Key tools
- `Drata_searchControls` — the full in-scope set (`is_ready` unset, `is_enabled` defaulting True)
  is the population; the not-ready subset is split client-side from `flags.isReady`.
  Add `expand=["flags","customFields","owners","frameworkTags"]`: `flags` carries `hasEvidence`,
  `hasPolicy`, `isMonitored`, `hasOwner`; `customFields` carries the account's own fields with their
  values. Paginate to the end — the in-scope set is a few hundred rows at most.
  **Never use `query`**: it ignores every filter.

## Workflow
1. **Resolve the workspace once**, then pull the full in-scope set (`is_ready` unset, `is_enabled`
   defaulting True) with the expands above, paginating to the end. Everything below is computed
   from those rows — no further calls; the not-ready subset splits client-side on `flags.isReady`.
2. **Split the not-ready subset into the two worklists.** They are mutually exclusive:
   - **Nothing mapped** — `hasEvidence` false **and** `hasPolicy` false **and** `isMonitored` false.
   - **Evidence not current** — `hasEvidence` true **and** `isMonitored` false.
   A control with a policy or a monitor and no evidence belongs to neither; it is a different fix
   and is out of scope for this skill. Say in one line how many not-ready controls fell outside both
   lists so nothing looks hidden — **that is the only figure on the page, and it is a remainder,
   not a total.**
3. **Metadata pass over every in-scope control**, not just the not-ready ones — see below. Derive
   both the worklists above and the metadata tables from the same step-1 rows; never issue two
   passes over the estate.

## Output format
```
## Control Gap Identification — [workspace] · [date]

### Nothing mapped
| Control | Name | Frameworks | Owner |

### Evidence mapped, not current
| Control | Name | Frameworks | Owner |

### No owner assigned          (all in-scope controls)
| Control | Name | Frameworks | Ready |

### No description             (all in-scope controls)
| Control | Name | Frameworks | Ready |
```

**Both tables are the same four-column shape:** control code, name, framework tags, owner (or `—`).
No status column — every row in a table shares the same status, so a column restating it is dead
weight. Cap at 15 rows per table and close with `[n] more — full list on request`.

### Metadata to fill in
**Scope: every in-scope control, not just the not-ready ones.** A missing owner or description is a
record gap whether or not the control currently passes — and a ready control with no owner is the
one nobody notices until it breaks. This is the only part of the skill that looks beyond the
not-ready set.

**Two checks, and only these two for now:**

| Check | Detection |
|---|---|
| No owner | `flags.hasOwner` is false |
| No description | `description` is null or whitespace |

**List the controls. Never chart this.** It is a gap worklist: the reader needs the codes so they can go
and assign an owner, not a bar telling them how many. Two tables, same shape:

```
### No owner assigned
| Control | Name | Frameworks | Ready |

### No description
| Control | Name | Frameworks | Ready |
```

Include the readiness state as the last column so the reader can work the not-ready ones first
without the tables being split in two. Cap at 15 rows each and close with
`[n] more — full list on request`.

**This requires a full paginated pass over in-scope controls** — `is_ready` is not set, `is_enabled`
defaults True — because neither check has a server-side filter. That is roughly nine pages at
`size=50` on a 400-control estate, and it is the reason this skill fetches rows at all. Reuse the
same rows for the two worklists above rather than pulling the estate twice.

**Expect `No description` to be empty on most accounts.** Descriptions ship with the control
template, so the check almost always returns zero — that is a pass, not a bug. **Print the heading
with `0` rather than dropping it**, so the reader knows it was tested. Custom controls authored by
hand are where a missing description actually turns up.

**Never call a missing owner or description a compliance failure.** Drata attaches no readiness
obligation to either — a control can be ready with neither. They are governance hygiene, and the
wording should say exactly that.

### Now · Next · Watch
**Now** — Nothing-mapped controls in the framework you are auditing against: no evidence, policy or monitor is carrying them. → drata-control-resolve-gaps
**Next** — Evidence mapped but not current: the coverage exists and has gone stale. → drata-control-resolve-gaps · the evidence itself → drata-evidence-resolve-gaps
**Watch** — Owner and description gaps, including on ready controls: governance hygiene, no readiness obligation.

## Edge cases
| Situation | Handling |
|---|---|
| Asked "how many controls are not ready?" | That is drata-control-report. Give no count here — say where the number lives and list the rows |
| API count differs from the UI | Expected; they can classify a single control differently. Never reconcile them in this deliverable |
| Control has a policy or monitor but no evidence | Belongs to neither worklist — count it in the one-line remainder and move on |
| Metadata pass scoped to not-ready only | Wrong — it covers every in-scope control; a ready control with no owner is the one nobody notices |
| `No description` returns zero | Expected on most accounts; print the heading with `0` rather than dropping it |
| Tempted to chart the metadata gaps | Never — this is a gap worklist; list the control codes so they can be worked |
| Tempted to call a missing owner non-compliant | Never. Drata attaches no readiness obligation to owner or description; call it governance hygiene |
| Asked why a specific control is not ready | Answer from its `flags` — which artifacts are mapped and which are missing — not from a guess about staleness |

## Example invocations
- "Which controls have nothing mapped?"
- "Show me controls where the manual evidence is stale."
- "What metadata is missing on our not-ready controls?"
- "Which not-ready controls have no owner?"
- "Give me the control gap worklist for this sprint."
