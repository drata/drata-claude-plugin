# Shared Protocol — Write Safety

**Every skill that writes to Drata (create / update / delete of controls, evidence, risks, vendors, personnel) MUST follow this protocol. It is an invariant, not an optional step.**

## The lifecycle

```
Analyze → Recommend → Preview exact diff → Confirm → Write → Read back → Verify → Receipt
```

1. **Capability check.** Confirm the tool exists and the user's OAuth scope + Drata role permit the write. If not, degrade to read-only and say so — never promise a write you cannot perform.
2. **Scope check.** Resolve the exact workspace, risk register, or account-scoped object and the specific target ID/code. Resolve people by name/email, controls by code, groups by name — search before using an opaque ID.
3. **Current-state read.** Retrieve the complete current fields and relationships that could change.
4. **Validation.** Never guess an identity. Reject ambiguous people, controls, evidence, vendors, risks, or policies and ask one clarifying question.
5. **Diff preview.** Show exactly what will be added, changed, removed, or cleared — **including relationship members that would disappear.** (Medium per the skill's output rule: a batch preview of more than 3 rows may render as a mini-branded artifact alongside the chat prompt; the confirmation itself always happens in chat.)
6. **Consequence disclosure.** State replacement semantics, irreversible effects, and external-sync effects before asking to proceed.
7. **Explicit confirmation.** Require approval of the *exact diff*, not a broad earlier intent.
8. **Single execution.** Make the smallest scoped change.
9. **Read-back verification.** Re-fetch the canonical object and compare intended vs actual. Personnel/search indexes can lag — verify by direct ID/email lookup.
10. **Receipt.** Report object ID/code, scope, before→after, verification result, and any field the API silently dropped. (A batch write of more than 3 rows may render its receipt as a mini-branded artifact, per the skill's output rule.)

## Relationship fields REPLACE — they do not append

Control owners/policies/requirements/tests, risk owners/reviewers/categories/controls, and evidence↔control mappings are **replacement** sets. "Set the owner to Alice" or "add Alice" must never be sent as `[Alice]` if that silently removes existing owners.

```
read current set → compute the full intended set (union/difference) → preview the complete replacement → confirm → write → verify
```

If the user says "add X **without removing the current ones**," include the current members in the payload.

## Destructive operations (delete risk / evidence / vendor)

Two-call pattern, always:
1. Preview with confirmation **disabled** — show exactly what will be removed.
2. Obtain explicit approval.
3. Execute with confirmation **enabled**.
4. Verify absence and issue a deletion receipt.

## Silent-drop awareness

Some writes silently discard fields (e.g. risk treatment details/residual scores when a risk stays untreated; owners/reviewers whose roles are ineligible). Always read back and report anything the API dropped.
