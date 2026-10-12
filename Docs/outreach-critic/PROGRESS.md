# Outreach Critic: Progress

Updated 12/10/2026. Branch: `build/outreach-critic` (not merged, not pushed).

## State

Phases 1 to 3 complete. Phase 4 complete except the two steps that need the author: one end-to-end capture through the real scheduled agent, and the go for merge and push.

Smoke test (run before any further work): from `outreach-critic/`, run fixture F9 through the skill headlessly and expect `VERDICT: SEND` with the rewrite identical to the draft. Last result: green, 5 of 5, four runs in a row.

## Features

State moves to `passing` only on the listed verification, checked by a reviewer that did not write the critic.

| AC | Behaviour | Verification | State |
|---|---|---|---|
| AC1 | Skill and sub-agent are listed after install | A new session lists both | passing |
| AC2 | Hand run returns the six-part plain-text block | Fixture runs, format check by script | passing |
| AC3 | Missing recipient: asks, no critique | Live run with no recipient, and with only a greeting name | passing |
| AC4 | Each seeded flaw is caught and named | Fixtures F1 to F8 against `fixtures/expected.md` | passing |
| AC5 | Clean draft returns SEND unchanged | F9 five times, diff against the draft | passing (5 of 5) |
| AC6 | Sender's claims pass through verbatim | Claim-by-claim check on all fixtures, plus an implausible-claim probe | passing |
| AC7 | Rewrites obey the rules and the length bar | Word counts and banned-pattern search on all rewrites | passing |
| AC8 | Read-only tools, no file change on a hand run | Tool list check; repo status and file times before and after 13 runs | passing |
| AC9 | Loads headless; direct spawn works; no recipient gives INCOMPLETE | `claude -p` from an empty directory | passing |
| AC10 | Drafting agent files the rewrite with the critic note | Dry run of the edited agent step in a scratch copy: passed. Real scheduled run: not yet done | active |
| AC11 | Critic missing: original filed, run continues | Dry run with the sub-agent file renamed: "Critic: not run (unavailable)", exit 0 | passing (dry run) |
| AC12 | No personal data in the repo | Search of the repo for names, organisations, emails, links; `rules.md` ignored and absent | passing |

## Review record

- Round 1: fixture reviewer found two blocking defects (code fence, inconsistent standard); red team found two (no recipient guard in the critic, invented sender content). All fixed.
- Round 2: red team passed 11 of 11 over 38 live runs, no blocking defects. Fixture reviewer found three (a stated word figure, the heaviest ask kept, needless rewording). All fixed.
- Round 3: a fresh reviewer confirmed the three fixes and passed every line of `expected.md`.

## Known non-blocking issues

- WHAT CHANGED can under-report cuts on a heavily shortened draft (seen on F6).
- The critic sometimes adds "In return," before the offer.
- The fixtures give one sample per flawed draft; run-to-run stability of F1 to F8 is not measured.
- No fixture covers the INCOMPLETE path (it was tested live, not recorded).

## Next steps

1. Author: read and correct the private `rules.md`.
2. Author: capture one real outreach task; confirm AC10 on the scheduled agent's output.
3. Author: go or no-go on merging to `main` and pushing.
