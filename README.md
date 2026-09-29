# claude-skills

A small set of [Claude Code](https://claude.com/claude-code) skills for planning and building software with agents. They encode a single opinion: **an agent should not grade its own work, and taste has to be injected deliberately.** Everything here follows from that.

The three skills chain together:

```
plan-solo  ->  build  ->  (ship)
  plan          build loop with a
  the spec      SEPARATE evaluator
```

Plus `vault-audit`, a read-only drift check for a markdown knowledge base.

---

## The skills

### `plan-solo` — plan before you build
Turns a fuzzy "I want to build X" into self-contained spec docs (Specifications, Implementation Strategy, and an optional Design doc) that live in the repo next to the code. The core discipline is **empirical, never assumed**: read the actual library source, verify the API, confirm the version, before any of it enters the plan. Stops at "docs written" and hands off to `build`.

**Use it when:** the work is multi-hour and structured, and mixes steps the agent does with steps you do.

### `build` — a build loop that can't lie to itself
Executes a spec with a tight Generator -> Evaluator loop. The generator writes code; a **separate evaluator subagent** exercises the running artifact against fixed acceptance criteria and defaults to FAIL on doubt. State survives across sessions in `PROGRESS.md` / `DECISIONS.md`, and only a verification command (never the model's own say-so) can mark a feature `passing`.

For anything with a visual surface, a **second** subagent — a design-critic that sees only screenshots, never the code — scores the UI against a rubric and pushes it off the generic "AI slop" default.

**Use it when:** a spec exists and the job is "build it" or "continue building it".

### `vault-audit` — keep a knowledge base honest
Read-only audit of a markdown vault (Obsidian or similar) against its own rules file: structural drift, stale tasks/follow-ups, duplicate notes. Writes one dated report and touches nothing else.

**Use it when:** your notes system has drifted from the structure you meant it to have.

---

## Why a separate evaluator

Self-evaluation is systematically over-positive: agents confidently praise their own mediocre work. A separate judge, in a fresh context, is the single strongest lever on output quality, and it is model-independent, so it stays load-bearing as the base model improves. The design-critic is the same idea applied to aesthetics: a model reviewing its own code defaults to the safe, generic look, so the critic is shown only the pixels.

Most of the classic long-running-agent scaffolding (context-reset rituals, per-sprint decomposition, one-feature-per-session limits) is dead weight on a current frontier model. These skills strip it and keep only what stays load-bearing: separation of generator and judge, cross-session continuity, and acceptance criteria fixed before coding.

---

## Install

Claude Code reads skills from `~/.claude/skills/` (global) or `.claude/skills/` (per project). Copy the folders you want:

```bash
git clone https://github.com/Arthurmf01/claude-skills.git
cp -r claude-skills/plan-solo  ~/.claude/skills/
cp -r claude-skills/build      ~/.claude/skills/
cp -r claude-skills/vault-audit ~/.claude/skills/
```

Then invoke them by name in a session (`/plan-solo`, `/build`, `/vault-audit`) or just describe the task and let Claude pick.

---

## Notes

- These are prompt/instruction files (`SKILL.md`), not code. Read them before running: they describe git behaviour, when to spawn subagents, and what the agent will and won't do on its own.
- Tune the git and commit-approval conventions in `build` to match how you work.
- Built and used in anger, then generalised. Issues and forks welcome.

## Licence

MIT. See [LICENSE](LICENSE).
