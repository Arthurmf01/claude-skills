---
name: plan-solo
description: Plan a substantial build/create project for individual execution — software system, CLI tool, automation, research effort, infrastructure, anything that mixes automated steps (the agent does them) with manual steps (you do them) toward building something. Use on "let's plan / spec / design a project", "I want to build X", "let's plan out Y", or when a request implies multi-hour scoped work needing research, design decisions, and a step-by-step build plan. Trigger when scope warrants formal planning AND the work fits the "build/create something with mixed automated+manual steps" shape (webapp, CLI tool, data pipeline, agent, server setup). Produces self-contained docs (Specifications, Implementation Strategy, an optional Design spec for UI-bearing projects, and an Overview for multi-version plans) inside the target repo's Docs/ folder. Stops at "docs written" and hands off to the build skill.
---

# Plan Solo (`/plan-solo`)

Walks you through planning a build project for individual execution. Output is a set of self-contained markdown files that capture **what to build** (Specifications) and **how to build it** (Implementation Strategy). For projects with a visual surface, a **Design** doc also captures **how it should look and feel**. Multi-version plans also get an Overview.

Docs are born directly in the target repo, so they sit with the code they drive. The canonical Specifications and Implementation Strategy always live in the repo, the single source of truth. The build skill reads the repo copy as its acceptance-criteria contract.

## Where things live

Planning docs go in the target repo's `Docs/` folder:

- **New top-level project:** `<repo>/Docs/`. Create the repo folder and `Docs/` if they don't exist yet.
- **Sub-project inside an existing repo:** `<repo>/Docs/<Sub-project>/`.

The canonical Specifications, Implementation Strategy, and Overview live **only** in that repo `Docs/` folder. Never copy their bodies elsewhere — a duplicated spec is a second master that drifts the moment either side is edited.

Respect the repo's own conventions once inside it (language version, dependency manager, type hints, logging, test framework, file-size limits), as recorded in its own project instructions. Don't impose one stack's conventions on another.

### Optional: a pointer note in your notes system

If you keep a personal knowledge base (Obsidian, Notion, a notes repo), you can drop a **lightweight pointer note** there so the project is visible and cross-linkable alongside your other work. Keep it a router, not a copy: goal, status, your manual steps, and a link to the repo docs — never the spec bodies. The repo stays the single source of truth. This step is optional and system-specific; skip it if you don't keep a second index.

## When to trigger

Trigger when BOTH hold:

1. **Scope is multi-hour to multi-day, structured.** Not a single quick task.
2. **Implementation mixes automated steps (the agent) with manual steps (you) toward building something.** The "something" can be a webapp, CLI tool, automation, data pipeline, agent, research write-up, infra setup, etc.

**Don't trigger for:** ad-hoc questions, single-file edits, purely exploratory conversation, work that's all-manual or all-automated, or anything that fits inside one response.

If unsure, ask once: *"This feels substantial enough to spec out properly. Should we?"*

## Workflow

### 1. Get oriented

Before research, anchor the basics:

- New project/sub-project, or an extension of an existing one?
- **Which repo does it live in?** New top-level project, or a sub-project inside an existing repo? When in doubt, ask, don't assume from topic alone.
- What's the name?
- Is this a **multi-version** plan (the work evolves across iterations) or **single-version** (build once and done)?
- Vocabulary is fixed: *version* -> *phase* -> *step*. Don't substitute "stages", "milestones", or "sprints".

### 2. Research empirically

**Always empirical. Never guess. Never assume.** This is the core discipline.

If a fact matters to the design, find it:

- **Software:** read the actual source of candidate libraries/frameworks (fetch from raw GitHub), check current docs, verify provider APIs, library versions, compatibility, licence.
- **Infra / sysadmin:** verify exact install paths, current versions, platform compatibility.
- **Research write-ups:** read recent sources in the area; look up methodology comparisons.

When you don't know, say "I don't know" and either go research it or flag it as an open question. Don't fill gaps with plausible-sounding inference. State any assumption you proceed on and mark what you couldn't verify.

### 3. Run a dialogue

Plan quality comes from back-and-forth, not one-shot writing. The dialogue can run across many turns.

**Question framework** — the *functions* are invariant; the phrasing adapts to the project:

- **Goal.** What are you trying to build? What does "done" look like?
- **Non-goal.** What this explicitly is NOT trying to be.
- **Constraints.** Immovable: deadline, budget, hardware, compatibility, ethical, regulatory.
- **Design / approach.** The chosen architecture, methodology, or strategy.
- **Components / dependencies.** What's used to build it. Flag any new external dependency explicitly.
- **Sequence.** What order the work happens in. What's parallelisable.
- **Deferred.** What's explicitly out of scope for this version, parked for a future version.
- **Uncertain.** What we don't yet know: open items to resolve during the build.

Ask in the order that fits the topic. **Do not impose a rigid section template** — doc structure emerges from the answers. A useful confidence discipline while you dialogue: put a rough confidence % on non-trivial design calls; high confidence, state your recommendation and move on; medium, recommend but flag it as vetoable; low, stop and ask. Be blunt: push back on assumptions, surface trade-offs, correct a wrong premise in the first sentence. Recommend with reasoning, don't just enumerate options.

The dialogue continues until you signal you're ready to commit to writing files. *That signal is the gate.*

### 3b. Design gate — UI-bearing projects only

Runs after the dialogue, before writing files. Two parts: automatic detection, then a one-question involvement gate. This exists because design quality comes from *human taste injected deliberately*, not from the model's defaults: an LLM is a next-token predictor, so left alone it makes the safest, most generic choice at every step and produces "AI slop". Taste is yours to give; the skill's job is to open the door, not to auto-decide aesthetics.

**Detect (automatic).** Does this project have a real visual surface a user looks at — a webapp, landing page, mobile UI, deck, or similar? If it's a CLI, agent, data pipeline, library, or pure backend, there is no design surface: skip this whole section and write no Design doc.

**Gate (one question).** If a surface exists, ask exactly one thing, with B as the default:

- **A — You drive.** You supply the creative direction and any references; the agent structures them into the Design doc.
- **B — Propose and you react (default).** The agent drafts 2-3 distinct directions (using seed strings and concrete inspirations to escape the generic default), you pick or veto; the chosen one becomes the Design doc.
- **C — Skip.** No Design doc this version (throwaway, internal-only, or you'll design later). Record the skip so it reads as a choice, not an oversight.

Don't fully automate this and don't force it on every project: one gate, sensible default, skippable.

**What the Design doc captures** (only on A or B). Same repo-doc rules as the other files. Sections emerge from the project, but cover:

1. **Creative direction / intention** — the ONE feeling the design should create, plus a *concrete* reference (a brand, era, game, object), never "clean and modern".
2. **Design principles / guardrails** — the non-negotiables baked in up front: restraint over addition, native components, no decorative gradients or glows for their own sake. (Polish-by-subtraction, decided now so it isn't retrofitted later.)
3. **Variety strategy** — whether to seed with random strings, which directions to explore before committing.
4. **The design-critic rubric + moodboard** — the visual acceptance bar the build skill verifies against: a /10 quality target, 3-5 reference screenshots to rank against, and the stopping criterion. This is to the design-critic what the Specifications' acceptance criteria are to the functional evaluator.
5. **Asset plan** — any image / video / shader generation needed, and how keys are handled (gitignored, never shipped in the product).

The Design doc is the input that makes the build skill's design-critic objective. Without it, a UI build has no visual bar and drifts back to the generic default.

### 4. Write output files in parallel at the gate

When you give the go-ahead, write all output files in a **single batch (parallel tool calls)**.

Create the repo folder and `Docs/` if they don't exist. For a genuinely new repo, `git init` it (a repo folder without git isn't protected). Don't commit — that's an explicit call.

**Naming convention:**

| Plan type | Files |
|---|---|
| Single-version | `<Name> Specifications.md`, `<Name> Implementation Strategy.md`, `<Name> Design.md`* |
| Multi-version | `<Name> v<N> Specifications.md`, `<Name> v<N> Implementation Strategy.md`, `<Name> v<N> Design.md`*, `<Name> Overview.md` |

*Design file only for UI-bearing projects, and skipped if you chose "Skip" at the design gate (§3b): omit it entirely otherwise.

**Output file rules — strict:**

- **No frontmatter.**
- **Self-contained.** No dependency on any file outside `Docs/`. The docs must make sense to a reader who only has the repo.
- **Specifications must carry testable acceptance criteria.** Every feature/capability gets explicit, verifiable pass conditions: a command to run, a check to perform, or an observable behaviour. Never a vague "works correctly". These criteria are the contract the build skill verifies against; without them the build loop has nothing concrete to test. Keep them at the user-story / behavioural level, not granular implementation detail (over-specified specs cascade errors into the build).
- **Implementation Strategy is the runbook**: the ordered phases and steps, which are automated (the agent) vs manual (you), and what each phase depends on.
- **Design (UI-bearing projects only)** carries the creative direction, design principles, variety strategy, and crucially the **design-critic rubric + moodboard** — the visual acceptance bar the build skill verifies against. Omit the file entirely for non-visual projects.

**Where decisions live:** in the *latest version's* Specifications. Single source of truth. When a new version supersedes an earlier decision, edit the earlier version's Spec to mark that decision **DEPRECATED** with a one-line pointer to the new version. No dual logs; git is the version history.

**Cross-version concerns** (multi-version only) live in the Overview:
- Lineage: what each version added.
- Cross-version goals / non-goals.
- Strategic decisions that hold across versions.
- Cross-version open questions.
- Document map (which file is authoritative for what).

### 5. Stop at "docs written" — then hand off to the build skill

The skill's job ends when the files exist on disk. Execution is handled by the **build** skill, which reads these docs (Specifications for what + acceptance criteria, Implementation Strategy for the sequence) and drives the build loop. plan-solo plans; build executes. You can also execute manually from the docs: the Implementation Strategy is the runbook either way.

**Always end by emitting ONE ready-to-paste line** as the very last thing the skill outputs, using the *actual on-disk absolute path* of the file just written. Default to the lowest unbuilt version.

Multi-version — point at the Overview:
```
go ahead and /build v1 <abs path to "<Name> Overview.md">
```

Single-version — point at the Specifications:
```
go ahead and /build <abs path to "<Name> Specifications.md">
```

## Revisions during execution

If you come back mid-execution saying the spec is wrong:

1. **Ask first.** Surface the conflict. Propose the fix.
2. **Edit the latest version's Spec in place.** Git is the version log.
3. If the change invalidates the Implementation Strategy too, re-engage *this skill* — it's a substantial replan, not a patch.

## Critical disciplines

- **Docs travel.** Everything inside a repo `Docs/` file must make sense to someone who only has the repo.
- **Repo is canonical.** The Specifications/Strategy bodies live in exactly one place: the repo. Any external index summarises and links, never copies. If they ever disagree, the repo wins.
- **Empirical over assumed.** Verify libraries, versions, APIs, prices before they enter the design. Flag what you couldn't verify.
- **Never commit without an explicit go.** Writing plan files is fine (reversible, git is the safety net); committing and pushing is a separate call.

## Example invocations

*"Let's spec this out — I want to build a self-hosted RSS reader."*
-> Software, multi-version likely. Trigger. Research candidate stacks by reading their READMEs/source. Dialogue. Write to `<repo>/Docs/`.

*"I want to plan out a new automation that syncs my notes into a database."*
-> Sub-project or new repo? Mixed automated/manual. Trigger. Get oriented (which repo, multi-version?), research the relevant APIs empirically, dialogue through goals + constraints + sequence, write at the gate.

*"What's the capital of France?"* -> Don't trigger. Answer directly.

*"Can you fix this typo on line 42?"* -> Don't trigger. Just fix.

*"What do you think about this idea I have for X?"* -> Don't trigger yet, you're exploring. Engage. *Offer* to formalise into a plan if/when it matures into "let's actually do this."
