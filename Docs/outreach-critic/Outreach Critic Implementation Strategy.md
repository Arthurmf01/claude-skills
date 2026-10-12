# Outreach Critic: Implementation Strategy

Written 12/10/2026. Runbook for the build. Read the Specifications first: the acceptance criteria (AC1 to AC12) referenced here are defined there.

Vocabulary: version, phase, step. This is a single version with four phases.

Legend: **[Claude]** automated, **[User]** manual.

## Ground rules for the whole build

- Source of truth is this repo. The installed copies under `~/.claude/` are copies; edit here, then install.
- The real `rules.md` is never created inside the repo. Add `outreach-critic/rules.md` to `.gitignore` before anything else, as a guard.
- Back up any existing file under `~/.claude/` before changing it, using the existing convention: `<file>.bak-YYYY-MM-DD-<tag>`. That directory has no git history.
- A background drafting agent may read its own skill file every couple of minutes. Edit that file in one write, never in a half-finished state.
- No commit and no push without the user's explicit go. This repo is public.

## Phase 1: Prove the mechanism

Purpose: retire the one real technical risk before writing anything else.

1. **[Claude]** Check current Claude Code documentation for the sub-agent definition format (file location, frontmatter fields, how tools are restricted, how a model is set). Record what was verified in `DECISIONS.md`.
2. **[Claude]** Add `outreach-critic/rules.md` to `.gitignore`.
3. **[Claude]** Write a throwaway ten-line sub-agent definition, install it to `~/.claude/agents/`, and call it from a non-interactive `claude -p` run in a scratch directory.
4. **[Claude]** Decide the path:
   - It loads: proceed with a named sub-agent (the design in the Specifications).
   - It does not load: switch to the fallback (critic prompt carried inline by the skill, spawned as a general-purpose sub-agent). Update the Specifications' component table and AC1, AC8, AC9, AC11 wording to match, then proceed.

Exit: the path is chosen and written down.

## Phase 2: Build the critic and the skill

Depends on: Phase 1.

1. **[Claude]** Write `outreach-critic/agent/outreach-critic.md`: the recipient persona instructions, the general rules, the claims-pass-through rule, the SEND-is-legitimate rule, and the fixed output block. Define the persona by what the recipient does and wants, not by a list of prohibitions.
2. **[Claude]** Write `outreach-critic/SKILL.md`: input collection, the missing-input stop, how the brief is assembled, the spawn call, how the result is returned, and the headless calling contract (what a calling agent passes in and gets back).
3. **[Claude]** Write `outreach-critic/rules.example.md` with invented content showing the shape: cold framing, warm framing, the offer, banned phrases, spelling and punctuation rules.
4. **[Claude]** Write the nine fixtures (F1 to F9) with invented people and organisations. Each fixture file carries the draft, the recipient, the relationship, the channel, any recipient notes, and the expected result.
5. **[Claude]** Draft the user's real `rules.md` directly in `~/.claude/skills/outreach-critic/` from the user's known preferences.
6. **[User]** Read the real `rules.md` and correct it. This is the only file where the user's own taste enters, so it is worth five minutes.
7. **[Claude]** Install both files to `~/.claude/`.

Exit: files exist and are installed.

## Phase 3: Verify against the fixtures

Depends on: Phase 2. The reviewer here must be a separate agent from the one that wrote the critic.

1. **[Claude]** Run all nine fixtures through the skill. Record verdict, weakest lines, and rewrite per fixture.
2. **[Claude]** A separate evaluator checks AC2 to AC8 against the recorded outputs and defaults to fail on doubt. AC5 needs five consecutive runs of F9.
3. **[Claude]** Fix and rerun until all pass. The usual failure is a critic that cannot say SEND; fix it in the agent's instructions, not by weakening the fixture.
4. **[Claude]** Run AC9 (headless load) on the finished critic.
5. **[User]** Run `/outreach-critic` on two real drafts of your own and judge the result. If the rewrites do not sound like you, the fix belongs in `rules.md`.

Exit: AC1 to AC9 pass and the user is satisfied with two real runs.

## Phase 4: Apply to the live setup

Depends on: Phase 3. This phase touches a running agent, so each step is small and reversible.

1. **[Claude]** Back up the drafting agent's skill file.
2. **[Claude]** Replace the hard-coded voice line in its outreach step with an instruction to read `rules.md` for framing. This removes the stale template so that generator and critic share one set of rules.
3. **[Claude]** Add the critic call to each step that produces a message for a person (single outreach, list sourcing, and message drafts filed as outreach): draft, critique once, file the rewrite with the verdict line beneath, fall back to the original on failure.
4. **[Claude]** Re-read the file the drafts are filed into immediately before any test write. Other processes write to it.
5. **[User]** Capture one real test outreach task from your phone, the way you normally would.
6. **[Claude]** Confirm AC10 on that task once the headless run has processed it.
7. **[Claude]** Run AC11: rename the critic file, queue a second test task, confirm the original draft is filed with "critic unavailable", restore the file.
8. **[Claude]** Remove the test entries, or leave them for the user to tick off, as the user prefers.
9. **[Claude]** Check AC12, then write the skill's section in the repo `README.md`.
10. **[User]** Give the go for commit and push to the public repo, or hold it.

Exit: AC10 to AC12 pass. The build is complete.

## What can run in parallel

- Phase 2 steps 1 to 4 are independent of each other.
- Phase 3 step 5 (the user's two real runs) can overlap with Phase 4 steps 1 to 3.

Everything else is sequential.

## Rollback

- Hand skill: delete `~/.claude/skills/outreach-critic/` and `~/.claude/agents/outreach-critic.md`.
- Headless integration: restore the drafting agent's skill file from the backup made in Phase 4 step 1. The agent picks it up on its next run.
