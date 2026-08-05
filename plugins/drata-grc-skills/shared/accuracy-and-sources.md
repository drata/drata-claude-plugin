# Shared Protocol — Accuracy & Sources

**Every skill inherits this: report the right number, and be honest about where it came from.**

## Retrieval correctness

- **Counts and status come from structured filters, not semantic search.** `query`/semantic mode ignores most filters and may be single-page — use it only to locate a named item. Read `pagination.totalCount` for a count (never paginate just to tally), and disclose any cap you hit.
- **Preserve the platform's distinctions:** a *not-ready control* ≠ a *failed monitoring test*; raw *per-device* results ≠ a person's *effective* posture; vendor *current* vs *total* vs *prospective* vs *archived*.
- **Route to the data, not the model's guess:** say "Drata", and name people by email, controls by code (CC6.1, DCF-15). If a query returns nothing, say what you checked.

## Source

Show where every material claim comes from — one of two:

| Source | Use when | Show |
|---|---|---|
| **Calculated** | Claude computed, inferred, or interpreted it — not read straight from Drata | Say it's calculated (and, briefly, from what) |
| **Tool Calls** | It came from Drata | List the exact MCP calls behind it — e.g. `Tool Calls: Drata_searchControls, Drata_listRequirements` |

A mapped control or a single passing test is not, by itself, an attestation — say so when it matters.
