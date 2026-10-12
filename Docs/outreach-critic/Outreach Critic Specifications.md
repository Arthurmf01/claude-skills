# Outreach Critic: Specifications

Written 12/10/2026. Single version. Revised the same day during the build: see "Changes made during the build" at the end.

## Goal

A Claude Code skill that puts an outreach draft in front of a hostile reader before the sender sees it. A separate sub-agent plays the recipient of the message (for example a hiring manager at an AI lab, a busy partner at a fund, a classmate), reads the draft cold, and returns a verdict, the three weakest lines, and a rewrite.

It exists because the agent that wrote a draft cannot judge it. Cooperation produces sycophancy: a generator asked to review its own message praises it. The critic runs in a fresh context, sees only the draft and a short brief about the recipient, and is told to react as that person would.

"Done" means: the skill can be run by hand on any draft, a headless drafting agent calls it automatically so that drafts arrive already critiqued, and the fixture suite below passes.

## Non-goals

- It does not send anything. It has no tool that could.
- It does not fact-check the sender. The sender's claims about themselves (titles, numbers, results, dates) are taken as true and carried into the rewrite unchanged. This is a deliberate decision by the author, not an omission.
- It does not research the recipient on the web. It uses what the caller passes in and what the caller's own notes already hold.
- It is not a general writing assistant. It judges one thing: how this message lands with this person.

## Components

| Component | Purpose | Location in this repo | Installed to |
|---|---|---|---|
| Critic sub-agent | The adversarial recipient. Fresh context, read-only tools. | `outreach-critic/agent/outreach-critic.md` | `~/.claude/agents/outreach-critic.md` |
| Skill | Orchestrates: collects the draft and recipient, builds the brief, spawns the critic, returns the result | `outreach-critic/SKILL.md` | `~/.claude/skills/outreach-critic/SKILL.md` |
| Rules file (example) | A template of sender-specific rules: framing for cold and warm contacts, banned phrasing, house style | `outreach-critic/rules.example.md` | Copied by the user to `rules.md` and edited |
| Rules file (real) | The user's own rules | Never in this repo (gitignored) | `~/.claude/skills/outreach-critic/rules.md` |
| Fixtures | Drafts with one seeded flaw each, plus one clean draft | `outreach-critic/fixtures/` | Not installed; used for verification |

No new external dependency. Everything is markdown read by Claude Code.

## Behaviour

### Inputs

- **Draft** (required): the message text.
- **Recipient** (required): name, role, organisation.
- **Relationship** (optional, default cold): cold, or warm with one line of shared context (where they met, a prior thread).
- **Channel** (optional, default LinkedIn message): LinkedIn message, LinkedIn connection note, email, WhatsApp. The channel sets the length bar.
- **Recipient notes** (optional): any profile text the caller already holds about the person.
- **Sender rules** (optional): the text of the sender's rules file. The skill reads the file and passes its text; the critic is never handed a path.

If the draft or the recipient is missing, the skill asks once and stops. It never invents a recipient. The critic carries the same guard for callers that bypass the skill: given no draft, or no recipient named with a role or organisation, it returns the single line `VERDICT: INCOMPLETE` with what is missing, and no rewrite.

### What the critic is told

- Who it is: the recipient, described by role, organisation, and what a person in that seat receives and ignores every week.
- What it has: the draft (inside `<draft>` tags, so that nothing in the message can pass for a field or an instruction), the relationship line, the channel, and the text of the sender's rules.
- How to judge: react as the recipient first (would I reply, what would I think of the sender, where did I stop reading), then check the sender rules.
- What it must not do: alter, soften, or question any factual claim the sender makes about themselves. Numbers, titles, employers, and dates pass through to the rewrite exactly as written.
- That SEND is a legitimate verdict. A draft that would get a reply is returned unchanged.
- What the rewrite may contain: only the draft, the relationship line, the recipient notes, and wording the sender rules supply. It adds no experience, opinion or intention on the sender's behalf. When the draft has no ask, the critic writes the smallest natural one and flags it: "Ask added by critic: confirm before sending."

The critic never sees the generator's reasoning, earlier drafts, or the conversation that produced the draft.

### Output

A fixed plain-text block with no code fence, whose first characters are `VERDICT:`, in this order:

1. **Verdict**: one of SEND (send as is), FIX (sound, with specific things to repair, including length alone), REWORK (the approach is wrong for this recipient).
2. **Rules**: "sender rules applied" or "general rules only".
3. **Reply likelihood**: low, medium, or high, with one sentence in the recipient's voice.
4. **Weakest lines**: up to three, each quoted verbatim from the draft with the recipient's reaction and the reason. Fewer than three is allowed. None on a SEND verdict.
5. **Rewrite**: the full revised message. On SEND, the original draft repeated unchanged.
6. **What changed**: one line. On SEND: "Nothing".

### Rules the critic enforces

General rules, built into the agent:

- One specific, answerable ask. A message with no ask, or with several, fails.
- A hook that could only have been written to this person. Generic flattery fails.
- Length fits the channel: a LinkedIn connection note within 300 characters, a LinkedIn message or WhatsApp within about 90 words, an email within about 150 words.
- Warm contacts: lead with the shared context. A credentials opener to someone who already knows the sender reads as posturing.
- Never name a competitor's product or model when writing to someone at a rival organisation.
- Any place, trip, or meeting framing must be consistent with the recipient notes. If the notes do not support it, flag it.

Sender-specific rules come from `rules.md` (preferred hooks, the offer the sender wants to make, phrases to avoid, spelling convention, punctuation bans). When `rules.md` is absent the critic runs on the general rules only and says so on the RULES line.

### Headless integration

A headless drafting agent (one that turns a captured task into an outreach draft with no human present) calls the critic after it writes each draft and before it files it.

- The filed message is the critic's rewrite. On SEND it is the original.
- One line is filed beneath the message: the verdict and the "what changed" line. The note is filed apart from the message text, so that a tool which later sends the message cannot send the note with it. On a REWORK verdict the original draft is kept as well where the filing format allows.
- One critic pass per draft. No second round, so a run cannot loop.
- If the critic fails to load, returns INCOMPLETE, or returns a malformed block, the original draft is filed with the note "Critic: not run" and the reason (unavailable, incomplete brief, unreadable result, recipient unresolved), and the run continues. The critic can never block the queue.
- The drafting agent's own generation prompt reads the same `rules.md`, so the generator and the critic work from one statement of the rules.

## Acceptance criteria

Each is verified by running the thing, not by reading the files.

**AC1. Install.** After copying the two installed files into place, a new Claude Code session lists `outreach-critic` among its skills and among its available sub-agents.

**AC2. Hand run.** `/outreach-critic` with a draft and a recipient returns the six-part block, in order, as plain text starting with `VERDICT:`, with a verdict that is exactly one of SEND, FIX, REWORK.

**AC3. Missing input.** Run with a draft and no recipient: the skill asks for the recipient and produces no critique.

**AC4. Seeded flaws are caught.** For each flawed fixture, the verdict is FIX or REWORK and the seeded flaw is named in "Weakest lines" or "What changed":

| Fixture | Seeded flaw |
|---|---|
| F1 | Names a competitor's model in a message to someone at a rival lab |
| F2 | Opens with a credentials list to a warm contact |
| F3 | No ask |
| F4 | Three asks in one message |
| F5 | Generic flattery hook that would fit any recipient |
| F6 | About 220 words for a LinkedIn message |
| F7 | Violates two house-style rules from the rules file (a banned punctuation mark and a banned phrase) |
| F8 | Refers to meeting "while you are in London" when the recipient notes place the person elsewhere |

**AC5. The clean draft passes.** Fixture F9 (a short, specific, single-ask message that meets every rule) returns SEND, with the rewrite identical to the input, in at least 4 of 5 consecutive runs.

**AC6. Claims are preserved.** For every fixture, each number, job title, employer name, and date in the draft appears unchanged in the rewrite, unless the sentence carrying it was cut for length. No output questions a claim's accuracy.

**AC7. Rewrites obey the rules.** No rewrite, across all fixtures, contains a banned punctuation mark or phrase from the rules file, and every rewrite is within the channel's length bar.

**AC8. Isolation.** The critic sub-agent's tool list contains read-only tools only (Read, Grep, Glob). A hand run produces no file change anywhere (`git status` in the working directory is unchanged).

**AC9. Headless load.** A non-interactive run (`claude -p`) from an empty directory that spawns the critic directly with a brief naming a rival's model returns a block with a FIX or REWORK verdict. This proves the custom sub-agent loads without a terminal session. The same direct spawn with no recipient returns `VERDICT: INCOMPLETE`.

**AC10. Headless integration.** Given one test outreach task in the drafting agent's queue, the filed entry contains the rewritten message and, beneath it, a line with the verdict and what changed. The source task is closed as before.

**AC11. Graceful failure.** With the critic sub-agent file temporarily renamed, the same headless run still files the original draft, marked "Critic: not run (unavailable)", and exits normally.

**AC12. No personal data in the repo.** `git status` and a search of the repo show no real `rules.md`, no real names, and no real message text. Fixtures use invented people and organisations.

## Deferred

- A second critic round, or a debate between generator and critic.
- Critiquing replies inside an existing thread (the critic would need the thread, which changes the input contract).
- Wiring into other draft sources (a reply-queue tool, third-party drafting plugins).
- Using the critic as the grader in an evaluation harness for the drafting agent.
- A library of saved recipient personas.

## Changes made during the build (12/10/2026)

- **Sub-agent loading: resolved.** A named sub-agent in `~/.claude/agents/` loads under `claude -p` on Claude Code 2.1.296 and its `tools:` restriction is honoured. Verified by a live probe. The fallback design was not needed.
- **Rules are passed as text, not as a path.** In a headless run the critic was denied permission to read a rules file outside the working directory and silently judged on general rules only. The skill now reads the file and passes its text in the brief.
- **INCOMPLETE verdict added.** Review found that a direct spawn with no recipient guessed one from the greeting and returned SEND. The guard now lives in the critic as well as the skill.
- **No-invention rule added.** Review found rewrites that put new statements in the sender's mouth. The rewrite is now limited to the brief (draft, relationship line, recipient notes) and wording from the sender rules, with a flagged exception for a missing ask.
- **Plain-text output and a RULES line.** Review found the block wrapped in a code fence and the "general rules only" notice placed inconsistently. The output now starts with `VERDICT:` and carries the notice on its own line.
- **List sourcing is not critiqued.** A step that files a one-line hook per person, not a message, gives the critic nothing to judge.

## Still open

- Which model the critic should run on. It inherits the caller's. The clean-draft criterion (AC5) held 5 of 5 in three consecutive runs, so there is no reason to change it yet.
