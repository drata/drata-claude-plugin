---
description: Create, update, or retire vendor records (write).
---
Load `${CLAUDE_PLUGIN_ROOT}/skills/drata-vendor-resolve-gaps/SKILL.md` and follow it in full — do not summarize or shortcut it. Dialogue and small writes stay plain chat (menus, questions, confirmations, and previews/receipts of 3 rows or fewer); a batch preview or write receipt of more than 3 rows renders mini-branded HTML per the skill's rule 1, carrying the same `Drata_getCompany` header (verified inlined logo only where the host can truly fetch it, else company name as text) — the confirmation itself always stays in chat. Apply the accuracy-and-sources protocol the skill references. If the user supplied arguments, treat them as scope/context: $ARGUMENTS
