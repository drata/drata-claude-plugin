# Shared Protocol — Accuracy & Sources

**Every skill inherits this: report the right number, and be honest about where it came from.**

## Retrieval correctness

- **Counts and status come from structured filters, not semantic search.** `query`/semantic mode ignores most filters and may be single-page — use it only to locate a named item. Read `pagination.totalCount` for a count (never paginate just to tally), and disclose any cap you hit.
- **Preserve the platform's distinctions:** a *not-ready control* ≠ a *failed monitoring test*; raw *per-device* results ≠ a person's *effective* posture; vendor *current* vs *total* vs *prospective* vs *archived*.
- **Route to the data, not the model's guess:** say "Drata", and name people by email, controls by code (CC6.1, DCF-15). If a query returns nothing, say what you checked.
- **Thread the workspace through every workspace-scoped call, and never read a picker as a zero.** Controls, requirements, monitoring tests and evidence are workspace-scoped; risks are **risk-register**-scoped; vendors, policies and personnel are **account**-scoped and take no workspace at all. Once the workspace is settled — because the user named it, or because the account has exactly one — pass it as `workspace_id` on *every* call that accepts one, for the rest of the session. Passing the name the user gave verbatim is fine; the server resolves it. **On a multi-workspace account, omitting it does not default to anything** — the call returns an `action_required` **picker object** with no `data` and no `pagination.totalCount`. Any logic of the shape "`totalCount > 0` means it exists" then silently reads that picker as **zero**, which is how a healthy tenant gets reported as having no frameworks, no controls, or nothing in scope. A response with no `totalCount` is a picker, not an empty result: resend it with the workspace. Never issue the same call across several workspaces to avoid asking, and never let a picker reach the user as a finding.

## Source

Show where every material claim comes from — one of two:

| Source | Use when | Show |
|---|---|---|
| **Calculated** | Claude computed, inferred, or interpreted it — not read straight from Drata | Say it's calculated (and, briefly, from what) |
| **Tool Calls** | It came from Drata | List the exact MCP calls behind it — e.g. `Tool Calls: Drata_searchControls, Drata_listRequirements` |

A mapped control or a single passing test is not, by itself, an attestation — say so when it matters.
