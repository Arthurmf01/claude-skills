# Outreach Critic: Decisions

Append-only. Newest at the bottom. Dates are DD/MM/YYYY.

**12/10/2026. Continuity files live in `Docs/outreach-critic/`, not at the repo root.** This repo holds several skills; a root-level progress file for one of them would mislead a reader. Rejected: repo root (the build skill's default).

**12/10/2026. Named sub-agent, not an inline prompt.** A probe sub-agent installed to `~/.claude/agents/` loaded under `claude -p` (Claude Code 2.1.296) and reported exactly the three tools its `tools:` line allowed. Rejected: carrying the critic prompt inside the skill and spawning a general-purpose agent, which would have inherited every tool.

**12/10/2026. The critic does not fact-check the sender.** The author's call. Claims pass through to the rewrite verbatim. Rejected: a claims ledger that flags or corrects disputed statements.

**12/10/2026. Rules travel as text in the brief.** First fixture run: 12 of 13 headless runs could not read the rules file (it sat outside the working directory) and judged on general rules only, which let the house-style fixture through as SEND. Rejected: passing a path and widening permissions, which every caller would then have to get right.

**12/10/2026. Review scale: a reviewer pair per round, no fleet.** One sub-agent file, one skill file, nine fixtures: nothing here decomposes into parallel builders. Two rounds were run. Reviewer A checked recorded fixture outputs mechanically; reviewer B attacked the live artifact.

**12/10/2026. Fixes taken from review round 1.** Code fence removed from the output contract; INCOMPLETE verdict for a missing draft or recipient, enforced in the critic and not only in the skill; no-invention rule for rewrites with a flagged exception for a missing ask; RULES line; draft wrapped in tags against field spoofing; one standard for weakest lines; length alone is FIX; no self-reported counts. The example rules' transition line was shortened so that the clean fixture fully meets it.

**12/10/2026. The critic keeps Read, Grep and Glob although it no longer needs to read anything.** Omitting `tools:` would inherit every tool, and an explicit read-only list is the safe statement of intent. Known consequence: run from the repo, it could open `fixtures/expected.md`. Its instructions tell it to work from the brief alone.

**12/10/2026. List sourcing is not sent to the critic.** It files a one-line hook per person, not a message.

**12/10/2026. Fixes taken from review round 2.** The relationship line is an allowed source for the rewrite (a spec omission, not a critic fault). Asks are ranked by cost to the recipient: a one-line question, then a short call, then introductions, referrals and document reviews. A sentence with no rule-based reason to change stays word for word. The output gives no word or character figures. A third reviewer confirmed these on a fresh run.

**12/10/2026. In the author's drafting agent, the critic note sits on the entry line, not under the message.** The agent files a message as quoted lines beneath a checklist entry, and two other tools read every quoted line as message text, one of which can send it. A note filed as a quoted line could have been sent to the recipient. The calling contract now says the note must be kept apart from the message. Rejected: a second quoted line (the original design).

**12/10/2026. "Critic: not run (reason)" replaces "critic unavailable".** One label for outage, incomplete brief, unreadable result and unresolved recipient hid which had happened.

**12/10/2026. On REWORK the original draft is kept only where the format allows.** In the author's setup the original is the agent's own first draft, not the author's writing, and there is no safe place for it beside the entry.
