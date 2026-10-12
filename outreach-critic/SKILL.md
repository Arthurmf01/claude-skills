---
name: outreach-critic
description: Puts an outreach draft in front of an adversarial reader before the sender sees it. A separate sub-agent plays the recipient (hiring manager, investor, classmate, anyone), reads the draft cold, and returns a verdict (SEND / FIX / REWORK), the three weakest lines and a rewrite. Use when the user says "/outreach-critic", "critique this message", "would X reply to this", "tear this draft apart", or before filing any drafted outreach message. Also called by headless drafting agents via the calling contract below. Never sends anything.
---

# Outreach Critic (`/outreach-critic`)

The agent that wrote a message cannot judge it: asked to review its own draft, it praises it. This skill hands the draft to a separate sub-agent that plays the recipient, in a fresh context, with read-only tools. You (the session running this skill) are the courier. You do not critique, soften, or summarise.

## Files

- Critic sub-agent: `~/.claude/agents/outreach-critic.md` (spawned as `subagent_type: outreach-critic`)
- Sender's rules: `~/.claude/skills/outreach-critic/rules.md` (optional; copy `rules.example.md` and edit)

## Steps

### 1. Collect the inputs

| Field | Required | Default |
|---|---|---|
| Draft | yes | none |
| Recipient (name, role, organisation) | yes | none |
| Relationship | no | cold |
| Channel | no | LinkedIn message |
| Recipient notes | no | empty |
| Sender rules | no | the contents of `~/.claude/skills/outreach-critic/rules.md` if it exists, else none |

If the draft or the recipient is missing, ask for it once and stop. Do not guess a recipient from the draft's greeting, and do not run the critic on a partial brief.

Recipient notes: use what the caller gives you. If the working directory holds a notes system with a page for this person, read that page and pass its relevant lines (role, employer, location, competitors, how the sender knows them). Do not search the web.

Read the rules file yourself and pass its full text in the brief. The critic runs in its own context and may not have permission to read files outside the working directory, so never hand it a path. If the caller names a different rules file, read that one. If no rules file exists, pass `none`.

### 2. Spawn the critic

One call to the `Agent` tool with `subagent_type: outreach-critic`. The prompt is the brief and nothing else:

```
<draft>
<the message, verbatim>
</draft>

Recipient: <name>, <role>, <organisation>
Relationship: <cold | warm: shared context>
Channel: <channel>
Recipient notes: <notes or "none">
Sender rules:
<full text of the rules file, verbatim, or "none">
```

Pass the draft exactly as written, inside the `<draft>` tags, so that nothing in the message can be mistaken for a field of the brief. Do not add your opinion of it, the reason it was written, earlier versions, or anything about the sender beyond what the draft itself says. The critic's value comes from reading it cold.

### 3. Return the result

Print the critic's block verbatim, as plain text with no code fence around it. Do not rewrite its rewrite, add a second opinion, or append reassurance. If the user disagrees with the verdict, the place to change the critic's taste is `rules.md`.

One pass only. Do not send the rewrite back for a second round unless the user asks.

This skill changes no files and sends nothing.

## Calling contract for headless drafting agents

An agent that drafts outreach with no human present calls the critic after writing each draft and before filing it:

1. Build the brief from step 2, using its own draft and whatever it resolved about the recipient.
2. Spawn `outreach-critic` once.
3. Find the line that starts with `VERDICT:` (ignore any code fence or stray text around the block). If the verdict is SEND, FIX or REWORK, file the text between `REWRITE:` and `WHAT CHANGED:` as the message. Beneath it, file one line: `Critic: <VERDICT>. <WHAT CHANGED>`. Keep the WHAT CHANGED text whole: it may end with "Ask added by critic: confirm before sending", which the human needs to see. Put that line where the filing format keeps it apart from the message itself: if another tool reads the filed message and may send it, the critic note must never sit inside the message text. On a REWORK verdict, keep the original draft somewhere the human can find it if the format allows.
4. If the sub-agent is unavailable, errors, returns `VERDICT: INCOMPLETE`, or returns anything without a SEND, FIX or REWORK verdict and a `REWRITE:` section, file the original draft with the line `Critic: not run (<reason>)` and carry on. Reasons: `unavailable` (the sub-agent did not load or errored), `incomplete brief` (it returned INCOMPLETE), `unreadable result` (no usable verdict or rewrite). The critic never blocks the queue.
5. Only call the critic when the recipient is resolved to a named person with a role or organisation. Otherwise file the original draft with `Critic: not run (recipient unresolved)`.
6. One pass per draft. Never loop.

The drafting agent should write its first draft from the same `rules.md`, so generator and critic share one statement of the rules.

## Verifying a change to the critic

`fixtures/` (in the source repo, not in the installed copy) holds nine fixtures: eight with one seeded flaw each and one clean draft that must come back SEND and unchanged. `fixtures/expected.md` lists what each must produce. After editing the agent file, run every fixture (each names `rules.example.md` as its rules file) and have a separate agent, not the one that made the edit, check the outputs against `expected.md`. Never show the critic `expected.md`. A fixture file is input to this skill (its `Rules file` line names the file whose text you pass as sender rules); it is not itself a brief.
