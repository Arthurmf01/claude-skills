---
name: build
description: Execute the implementation phase of a planned project as a lightweight long-running-agent harness. Drive a Generator->Evaluator loop against a written spec until acceptance criteria pass. Use AFTER planning (see the plan-solo skill) has produced Specifications + Implementation Strategy and the work is now "build it". Trigger on "let's build / implement / ship this", "start the build", "work through the implementation strategy", or resuming a half-built project ("pick up where we left off"). Runs in the code repo. Maintains cross-session continuity (PROGRESS.md, DECISIONS.md, a verifiable feature list) and always uses at least two SEPARATE, confrontational evaluator subagents, never the generator grading its own work, scaling to a full multi-agent builder-fleet + red-team orchestration (via a workflow-orchestration tool) when genuine scope/complexity signals warrant it, never by default. For UI-bearing projects that carry a Design spec, also runs a separate design-critic loop (screenshots only, scored against a design quality bar). Do not trigger for one-off edits, debugging a single failure, or planning.
---

# Build Skill (`/build`)

Operationalises the **implementation phase**. plan-solo is the Planner; this skill is the Generator + Evaluator. It turns a written spec into a working artifact by running a tight build->verify loop with cross-session continuity, so work survives across context windows and sessions without rebuild cost.

Load-bearing pieces: an independent Evaluator (mandatory, not optional), `PROGRESS.md` / `DECISIONS.md` continuity artifacts, and the feature-list-as-primitive (`behaviour + verification command + state`, where the generator cannot self-mark `passing`).

## Harness philosophy — strip what isn't load-bearing

Every harness component encodes an assumption about what the model can't do alone. On a current frontier model most classic long-running-agent scaffolding is **dead weight**. Re-check this each time a new model lands:

- **Strip:** context resets / compaction rituals; per-sprint decomposition; the artificial "one feature per session" limit; a separate initialiser-vs-coder agent split. Modern models run continuously through these.
- **Keep (still load-bearing on any model):**
  1. **Generator-Evaluator separation.** Self-evaluation is systematically over-positive: agents confidently praise their own mediocre work. A separate judge is the single strongest lever. Model-independent; never strip this. Floor: **at least two** independent, confrontational reviewers, not one — a lone evaluator can still rubber-stamp; two reviewers hunting for bugs give a tie-break signal a single judge can't (see §4, §4a).
  2. **Cross-session continuity artifacts.** Whenever work spans more than one context window, the next session needs machine-readable state or it drifts and re-does work.
  3. **Acceptance criteria fixed before coding.** Verifiable, not vibes.
  4. **The evaluator exercises the live artifact** (not just reads code) for anything with runtime/UI behaviour.

Scale all of this to project size. A 200-line CLI needs a thin version; a multi-surface app needs the full loop.

## When to trigger

Trigger when BOTH hold:

1. A spec exists (plan-solo output in `Docs/`, or equivalent), OR a half-built repo with continuity files to resume from.
2. The intent is "build/continue building it", not plan it, not fix one isolated bug.

If there's no spec yet, redirect to **plan-solo** first. If the intent is to confirm one already-finished change works, that's a verification pass (a single behaviour, end-to-end), not this build loop.

## Prerequisites

Runs in the **code repo** for the project. Expected inputs:

- `Docs/` — the planning outputs (plan-solo writes them straight here). Specifications = what to build + acceptance criteria; Implementation Strategy = the sequence; **Design (if present)** = creative direction, design principles, and the design-critic rubric + moodboard (the visual acceptance bar). A Design file exists only for UI-bearing projects; its absence means no design loop.
- If `Docs/` is missing but the spec lives elsewhere, read it by absolute path; don't move the session root.

If the repo doesn't exist yet, create it (`git init`, scaffold per the Implementation Strategy's stack), then proceed. Honour the repo's own stack conventions (language version, dependency manager, type hints, logging, test framework, file-size limits) as recorded in its own project instructions.

## Git model — read this before the first commit

The build harness relies on git as its checkpoint/rollback primitive. Set a commit policy that fits how you work; a safe default:

- **At clock-in, work on a dedicated build branch** (`build/<feature>` off `main`), never directly on `main`.
- **Checkpoint commits on that build branch** at each meaningful chunk are the harness's rollback primitive. If your policy is "no autonomous commits", checkpoint via `PROGRESS.md` + staged (not committed) changes instead.
- **Never push, and never merge to `main`, without an explicit go.** Pushing is outward-facing; merging to `main` is a real decision.
- Commit messages: short, imperative ("add rectangle fill tool", not "added"/"adding").

## Continuity artifacts

Two files at repo root, plus one feature list. These are the handoff between sessions.

- **`PROGRESS.md`** — current commit hash; smoke-test status (green/red); the feature list with per-feature state; what's in progress; blockers; next steps.
- **`DECISIONS.md`** — append-only: each decision, why, and the alternatives rejected. Prevents re-litigating settled choices on resume.
- **Feature list** (inside `PROGRESS.md`, or `features.json` if a scheduler will consume it). Each item:
  - `behaviour` — one user-facing capability, drawn from the spec's acceptance criteria.
  - `verification` — the exact command or browser flow that proves it (`pytest -k "rectangle_fill"`, a dev-server route check, or a browser-automation click-path).
  - `state` — `not_started` | `active` | `blocked` | `passing`.
  - **Hard rule:** only the verification command may move an item to `passing`. The generator never hand-edits state to `passing`. JSON/structured form is preferred precisely because the model is reluctant to overwrite it.

## Workflow

### 1. Clock-in

- Confirm working directory (`pwd`); read `Docs/`, `PROGRESS.md`, `DECISIONS.md`.
- Confirm/create the build branch (see Git model above).
- **Run the smoke test before any new work** — start the app / run the build+test and confirm it's green. This catches breakage *inherited* from the last session before you build on top of it. Cheapest high-value step in the harness. If red, fixing inherited breakage is the first task.
- Pick the highest-priority `not_started` / `active` feature (order from the Implementation Strategy).

### 2. Derive the feature list (first session only)

If no feature list exists, build it from the spec's acceptance criteria — one item per testable behaviour, each with its verification command. If the spec lacks acceptance criteria, that's a planning gap: write them now and note it, but don't invent scope the spec doesn't imply.

### 3. Generate

- Implement the selected feature(s). Run continuously; no artificial per-session limit on a capable model.
- Checkpoint with **git** (on the build branch) + a `PROGRESS.md` update at each meaningful chunk (descriptive messages; git is the rollback path to a known-good state).
- Run each feature's verification command. Only a passing command sets `passing`.

### 4. Evaluate — a reviewer pair, never self-review

When a chunk of features reports `passing`, spawn **at least two distinct evaluator** subagents, each in its own context. The main session is the generator; the subagents are the judges. **This floor of two is not optional and does not scale down** — even a one-file fix gets a reviewer pair before it's called done; only the orchestration *above* that floor (§4a) scales with project size. Each evaluator:

- Receives the relevant acceptance criteria and is told to be **adversarial and nitpicky**: default to FAIL on doubt, probe edge cases, don't praise, actively try to break the work rather than confirm it.
- **Exercises the running artifact**, not just the source:
  - For a **web UI**, drive the live app through browser-automation tooling. Create a new tab, don't reuse the user's.
  - For a **CLI/API**, run real invocations and assert on output/exit codes.
- Returns structured findings: per criterion -> `pass`/`fail` + evidence + `file:line` for each defect. Granularity wanted: *"Rectangle fill tool — FAIL — only places tiles at drag start/end, doesn't fill the region (src/tools/fill.py:42)."*
- Known blind spots to state, not hide: vision misses some layout bugs; browser automation can't drive native OS modals; deeply nested features slip through. Flag coverage gaps rather than implying full coverage.

**A feature only reaches `passing` once neither reviewer has a live blocking finding.** A single reviewer's approval is never sufficient — that's the whole point of the floor. Feed any failures back to the generator as the next work items. **Calibration:** if a reviewer runs too lenient or too strict over a couple of rounds, tighten its prompt — emphasise the dimensions the model is weak on by default, not the ones it already nails.

### 4a. Scaling the review — from a reviewer pair to full orchestration

The two-reviewer floor above is the minimum for every build, regardless of size. Whether to go further — a full builder fleet plus a dedicated red team via a workflow-orchestration tool — is a judgement call made fresh each time, not a default. Spinning up a large multi-agent orchestration costs real tokens; most builds (a single feature, a small app, a handful of pages) stay inside the lightweight loop in §4.

**Escalate only when a genuine signal is present:**
- The work decomposes into multiple genuinely independent, parallel-buildable units (separate pages, components, or modules that don't share in-flight state) — enough that a real fleet, not a padded one, would result.
- The project is large or unfamiliar enough that a single generator pass is likely to miss cross-cutting issues a second builder perspective would catch.
- The user has explicitly asked for orchestration on this specific build. An explicit ask satisfies a workflow-orchestration tool's own opt-in gate on its own — don't re-litigate whether to use it, only how to size it.

**When escalating, compose the fleet deliberately:**
- **Builders** — one agent per independent unit of work, weighted toward the strongest available model, with a minority on a lighter/cheaper model where that unit doesn't need the extra strength. Size to the actual decomposition; don't pad the fleet past what the task supports.
- **Red team** — always at least two agents (the §4 floor, scaled up), briefed to confrontationally hunt for bugs/regressions/edge cases rather than confirm correctness. Keep the red team's model assignment distinct from the builders' so it isn't reviewing in the same "voice" it built in. A feature still only reaches `passing` once the red team can't break it.
- Keep the design-critic (§4b) as a separate lane — it judges visual quality, the red team judges correctness. Don't merge them.

**Log the call.** Whichever size is chosen — reviewer pair only, or a full fleet — state the reasoning in `DECISIONS.md`: what signal justified (or didn't justify) the scale, and the fleet composition if escalated. So the next session doesn't have to re-derive it or wonder why a one-page fix got five agents.

This stays inside `/build`, not a separate skill: it's the same Generator-Evaluator principle from the harness philosophy, just with a higher floor and an explicit, judgement-gated escalation path. A standalone skill would duplicate `/build`'s triggering logic for no benefit.

### 4b. Design-critic — separate subagent, screenshots only

Runs only when a `Design.md` exists and the feature has a visual surface. This is a *second* separate judge, distinct from both the generator and the functional evaluator, and it is the single biggest lever on design quality: an agent cannot judge its own aesthetics because it reviews its own code, past decisions, and rationale, and it defaults to the safe, generic look ("AI slop") unless pushed.

**Design Scout — run once, before the first design loop.** If `Design.md` names reference products/sites or a vibe ("like Linear", "editorial", "warm minimal") and no `BRAND-REFERENCE.md` exists yet, spawn a scout subagent (browser automation) to *actually visit* those sites and extract **concrete tokens** — hex palette, font stacks, spacing scale, button/card/hover patterns, mood, anti-patterns — into `BRAND-REFERENCE.md` at repo root. A training-data impression of a site is not a substitute for live observation: the live site has moved on, and "inspiration" without observation produces generic design. This doc becomes the critic's north star for every round below. Skip only if `Design.md` already carries concrete tokens, or no references are named.

- Spawn a **design-critic** subagent. Prefer a strong model — taste scales with capability; the cheap generator does the implementing, the strong critic provides the bar.
- Feed it **only screenshots** of the running UI plus the Design doc's rubric + moodboard (and `BRAND-REFERENCE.md` if scouted) — never the code, implementation notes, or earlier critiques. Fresh context each round.
- **Score four criteria 1-10, weighted:** Design quality 0.35 (hierarchy, spacing, palette, type that communicates), Originality 0.30 (distinguishable from a template, made *for this product*), Coherence 0.20 (one product, consistent language across screens), Craft 0.15 (alignment, states, transitions — models nail this by default, hence the low weight). Composite = the weighted sum; that is the number the stop criterion reads.
- **Paste these calibration anchors verbatim into the critic prompt** — the single biggest lever on honest scoring. *"Most AI-generated UIs are a 4-5; that is the baseline, not 7. 3 = default framework styling with a colour swap. 5 = clean but conventional, you've seen it on 50 SaaS pages. 7 = intentional choices, palette and type that mean something. 9 = exhibition-grade, every pixel has a reason. If it looks like something an AI could generate in one shot, originality cannot exceed 6. Functional correctness does not raise design scores. Trendy flourishes (glassmorphism, gradients, dark mode) are not originality."*
- The critic also watches for overdone / obviously-AI patterns (decorative gradients, glows, generic left-text/right-graphic layouts, over-explaining) and penalises them, and returns tight, specific, *actionable* feedback ("the card grid has equal visual weight everywhere; make the primary card 2x and recolour it to create an entry point"), not vague prose. Hold the stopping criterion *outside* the critic prompt (loop until it independently scores >= the Design doc's target, typically 9/10 composite). Run one or two rounds first, confirm it's converging before adding more.
- **Decide refine vs pivot after each round.** *Refine* (polish the current direction) when the composite rose 0.5+ and the weaknesses are hierarchy/clarity/polish. ***Pivot*** (change the concept meaningfully — layout model, visual metaphor, density, tonal direction; not just swapping hex values) when originality is stuck below 6, or the composite gained <0.3 across two consecutive rounds. Don't polish mediocrity: a flat score means the direction is exhausted, not under-tweaked.
- Feed its gaps back to the generator as design work items, same loop as the functional evaluator.

Track a design-critic pass as its own feature-list verification for visual features (e.g. `design-critic >= 9/10 on <screen>`), separate from the functional pass. As with `passing`, the generator never self-scores it.

### 5. Loop, then clock-out

- Loop Generate->Evaluate until all in-scope criteria pass (or remaining failures are explicitly deferred in `PROGRESS.md`).
- **Clock-out:** update `PROGRESS.md` (commit, green smoke test, feature states, next steps) and `DECISIONS.md`; finish on a clean commit on the build branch. Never leave the tree dirty or the progress file stale: that's the unclean-handoff failure mode.
- If the build is done and it should go on `main` or be pushed, that's a separate explicit decision (see Git model).

## Scaling guidance

- **Small (single file, trivial CLI):** thin loop — generate, run tests, the two-reviewer floor (§4) at the end, checkpoint. Skip `DECISIONS.md` if there were no non-obvious choices. No orchestration tooling needed; two evaluator subagents suffice.
- **Medium (typical app):** full loop, the two-reviewer floor at the end of each feature batch. Still no full orchestration unless the project genuinely decomposes into parallel-buildable units (§4a).
- **Large / many independently-buildable surfaces / unfamiliar territory:** escalate per §4a — a weighted builder fleet plus a dedicated red team (≥2), sized to the actual decomposition. Add complexity reactively, never upfront, and log the reasoning in `DECISIONS.md`.

## Anti-patterns

- **Generator grading itself.** The whole point. Always a separate context for evaluation.
- **Marking complete on a single reviewer's approval.** The floor is two confrontational reviewers, every build, no exception — one judge can still rubber-stamp.
- **Escalating to full multi-agent orchestration by default.** Spinning up a builder fleet + red team when the work doesn't genuinely decompose into parallel units, or scope doesn't warrant it, burns tokens for no signal gain. Default to the §4 reviewer pair; escalate only on a real signal (§4a).
- **Design-critic seeing the code.** For UI projects, the critic judges screenshots against the design bar, nothing else; showing it the implementation collapses it back into self-review.
- **Imagining reference sites instead of scouting them.** If `Design.md` names inspiration sites, the scout must visit them live and extract real tokens; "I know what Linear looks like" produces generic design against a north star that doesn't exist.
- **Over-specifying in the spec** so errors cascade into the build. Keep specs at user-story/architectural level; that's plan-solo's job.
- **Marking `passing` by hand.** Only the verification command may.
- **Building on red.** Run the smoke test at clock-in first.
- **Leaving an unclean handoff.** Stale `PROGRESS.md` or a dirty tree forces the next session to debug before it can build.
- **Pushing or merging to `main` without an explicit go.** In-loop checkpoints on the build branch are fine; anything outward-facing is not.
- **Keeping dead scaffolding** because an article said so. Strip components that aren't load-bearing on the current model.

## Example invocations

*"The spec's done — let's build the website blocker."*
-> Trigger. Clock-in (build branch, smoke-test the scaffold), derive feature list from the Specifications' acceptance criteria, generate, evaluate via a subagent driving the running app, loop to green.

*"Pick up the build where we left off."*
-> Trigger. Read `PROGRESS.md`/`DECISIONS.md`, run smoke test to verify inherited state, continue from the next `not_started` feature.

*"Does my fix to the auth flow actually work?"*
-> Don't trigger. That's a single-change verification, not a build loop.

*"Let's plan out a new RSS reader."*
-> Don't trigger. That's plan-solo. This skill starts once the spec exists.
