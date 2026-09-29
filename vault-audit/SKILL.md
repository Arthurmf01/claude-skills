---
name: vault-audit
description: Audits a markdown knowledge base (Obsidian or any notes vault) for drift against its own rules file — stale task/follow-up items, folder-structure drift, and unconsolidated duplicate notes. Use on "check the vault", "audit my notes", "vault snapshot", or when you want a status read on your task/tracking files against what's actually true. Read-only against the vault; writes only one dated audit file.
---

# Vault Audit

Scans a markdown knowledge base against its own governance rules and writes one dated status file. Read-only against the rest of the vault — the only thing it writes is the audit file itself.

This skill assumes a vault that keeps (a) a governance/rules file at its root that describes the intended structure, and (b) one or more running lists of open items (tasks, follow-ups). Adapt the specific filenames below to your own setup; the method is what generalises.

## Instructions

1. **Locate the vault.** If the current working directory contains a governance/rules file that describes the vault's structure, use it as the vault root. Otherwise ask for the vault path before doing anything else — don't guess.
2. **Read the governance files**: the rules file plus any open-item lists (e.g. a tasks file and a follow-ups file) at vault root.
3. **Check structural drift**: list the actual top-level folders and compare against the intended set in the rules file. Flag anything new/unlisted. Flag any note sitting directly at vault root that the rules say shouldn't be there.
4. **Check the task list for staleness**: for each open task, skim its linked note/area folder for evidence it's actually done, superseded, or abandoned (e.g. dated context that's clearly long past with no update since). Don't guess — only flag when the linked note gives real evidence either way. If it's genuinely unclear, say so rather than picking a side.
5. **Check the follow-up list for staleness** the same way — flag anything that looks answered/resolved in its linked note but is still listed as open.
6. **Spot-check one or two folders** for notes that duplicate content already covered elsewhere, or raw material sitting around that should have been consolidated by now, if the rules call for one synthesised note per topic.
7. **Write the output** as `Vault Audit — YYYY-MM-DD.md` inside a dedicated audits folder (create it on first run if it doesn't exist) — not at vault root, so dated files don't pile up there. Headings, in this order:
   - **Structure** — folder-list drift, stray root files
   - **Tasks check** — tasks that look done/stale, with the evidence and a pointer to where you found it
   - **Follow-ups check** — same, for follow-ups
   - **Hygiene notes** — duplication/consolidation issues spotted
   - Any section with nothing to report should say so plainly (e.g. "no drift found") — don't pad it with filler.
8. **Rules**: only read, never invent. If a file referenced in the rules doesn't exist, or content is ambiguous, say "not there" / "unclear — worth checking" rather than guessing. Don't edit the task or follow-up lists yourself — the audit reports, it doesn't act; checking boxes or archiving items is a separate, explicit follow-up step.
9. After writing the file, report in one line where it landed and give a one-line summary of what it found (or that nothing needed flagging).

## Running it again

Invoke any time — each run creates a new dated file; old runs stay in place as a history of vault health over time. Don't overwrite or delete them.
