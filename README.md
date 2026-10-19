# grammar-companion

A local-first writing companion for **French and English** that lives in the browser,
flags grammar and spelling issues as you type, and adapts to how *you* write.

Main purpose: **learn Rust by building something used every day.**
The Rust core owns the logic. The browser extension stays a thin TypeScript client.

> Status: M0 — nothing built yet. See [ROADMAP.md](ROADMAP.md).

---

## How to work in this repo

1. Open this folder with Claude and run `/wise-code:learn`.
   Learning state is already seeded in `.wise-code/`. Onboarding has two questions left.
2. Assist level is **Hint** by default: Claude gives direction, you write the code.
   Switch for one step with "pseudo for this step", or for Rust globally with `/wise-code:level`.
3. Milestone loop:
   1. Read the milestone in `ROADMAP.md`.
   2. Write the ADR for its decision **before** writing code (`docs/adr/`).
   3. Build. Say "done" to get a code review (`cargo check`, `clippy`, tests).
   4. Tick the milestone's "Done when" boxes. Add what you learned to `docs/til.md`.

## Layout

```
README.md          this file
ROADMAP.md         milestones: outcome, Rust focus, decisions, done-when, traps
docs/adr/          architecture decision records (you write them)
docs/resources.md  reading list per milestone
docs/til.md        "today I learned" log — feeds the personal wiki later
fixtures/          FR/EN test corpus (you build it in M1)
.wise-code/        learning-mode state (profile, progress, project map)
```

There is no `Cargo.toml` yet on purpose: creating the crate layout is M0 and ADR-0001.

## Principles

- **Local first.** Typed text never leaves the machine unless you explicitly opt in.
- **Rules before models.** Deterministic checker first, LLM as an add-on.
- **Measure.** Every milestone ends with a number (latency, precision, false positives).
- **Decide in writing.** No ADR, no code.
