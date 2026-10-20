# Roadmap

Each milestone is a shippable increment. Every milestone lists:

- **Outcome** — what works when it is done
- **Rust focus** — what the milestone is designed to teach
- **Decide** — the open question you answer in an ADR before coding
- **Done when** — checkable exit criteria
- **Traps** — known pitfalls, named but not solved

Nothing below is a design. Components and choices stay yours until an ADR records them.

---

## M0 — Toolchain and skeleton

**Outcome:** an empty Rust project that builds, lints and tests, locally and in CI.

**Rust focus:** `rustup`, `cargo`, packages vs crates vs modules, editions, `clippy`, `rustfmt`.

**Decide → ADR-0001:** single crate, or a Cargo workspace with several crates?
What would each crate own, and why would the boundary help later milestones?

**Done when:**
- [ ] `cargo fmt --check` passes
- [ ] `cargo clippy --all-targets -- -D warnings` passes
- [ ] `cargo test` passes (even with a placeholder test)
- [ ] CI runs the three commands above on every push
- [ ] ADR-0001 written

**Traps:** designing five crates before writing one line of logic.

---

## M1 — CLI checker, in two parts

M1 is split on purpose. **M1a** builds a small checker yourself, so the core ideas are
yours. **M1b** plugs real engines in behind it for coverage, because full EN and FR
grammar coverage is years of rule writing.

Reference project: [Harper](https://github.com/automattic/harper) (Rust, Apache-2.0,
English only). Read it **after** M1a, not before: reading first means copying a design
without knowing why it is shaped that way.

### M1a — Own mini-checker (from scratch)

**Outcome:** a command-line tool that reads text from a file or stdin, takes a language,
runs **your own** rules, and prints diagnostics with exact positions and suggestions.

**Rust focus:** `&str` vs `String`, ownership across function boundaries, enums and `match`,
`Result` and `?`, custom error types, traits for swappable behaviour, iterators, unit tests.

**Build yourself:** a tokenizer, a span type, a diagnostic type, a rule abstraction,
and 3–5 rules. Candidate rules (pick, don't treat as a spec):
- French typography: no-break space before `; : ! ?`, spacing inside `« »`
- Repeated word: "the the", "le le"
- English `a` / `an`

**Decide → ADR-0007 (core types):** what fields does a diagnostic carry, in which unit is a
position measured, and what does a rule receive and return?

**Done when:**
- [ ] Tokenizer, span, diagnostic and rule types are your own code; no grammar-checking crate yet
- [ ] ≥ 3 rules, at least one French typography rule
- [ ] Fixture corpus started: ≥ 10 EN + ≥ 10 FR cases, including clean sentences (no issue expected)
- [ ] Fixtures run as `cargo test`
- [ ] Positions are correct on text containing `é è ç œ`, emoji, and French guillemets `« »`
- [ ] ADR-0007 written
- [ ] **Then, not before:** read Harper's equivalents (`Document`, `Token`/`Span`, `Linter`,
      `ExprLinter`, `FstDictionary`) and write a "mine vs theirs" entry in `docs/til.md`:
      one thing they did better, one choice of yours you would keep, and what unit their span counts in

**Traps:**
- Rust strings are UTF-8; slicing a `&str` at a non-character boundary panics.
- The browser will speak UTF-16 indices later. Byte, `char` and UTF-16 offsets all differ.
- French uses two no-break spaces (U+00A0 and the narrow U+202F); sources and tools disagree on which goes where.
- Building a generic rule engine or rule language before three concrete rules exist.

### M1b — Engine adapters

**Outcome:** the same CLI also runs real engines behind your rule abstraction, and merges
their diagnostics with your own rules' output.

**Rust focus:** depending on external crates, adapting foreign types into yours
(`From`/`Into`, newtypes), a blocking HTTP call, merging and de-duplicating results,
optional dependencies with Cargo features.

**Decide → ADR-0002 (engine strategy).** Facts to weigh, not a verdict:
- `harper-core` is a Rust crate, fast and offline, but **English only** (French not supported as of Oct 2026).
- LanguageTool covers French, runs as a local Java HTTP server, checked via `/v2/check`.
- An LLM-only checker is slow, non-deterministic and hard to test.

Also decide: when two sources flag the same span, which wins? What happens when an engine is unavailable?

**Done when:**
- [ ] Fixture corpus grown to ≥ 20 EN + ≥ 20 FR cases
- [ ] Engine positions converted into your unit; the `é è ç œ` / emoji / `« »` fixtures still pass
- [ ] False-positive count on clean sentences measured and written down, per source (yours and each engine)
- [ ] With an engine stopped, the CLI still runs your own rules and reports that engine as unavailable, without panicking
- [ ] ADR-0002 written

**Traps:**
- Each engine reports positions in its own unit. Check each one; don't assume it matches yours.
- An external engine call is I/O. Think about where it sits before M2 makes everything async.
- `harper-core` releases often. Pin the version and upgrade on purpose.
- LanguageTool is a separate Java process: startup time and memory are part of the cost.

---

## S1 — Side quest: one upstream Harper contribution (after M1b)

**Outcome:** one merged pull request in [automattic/harper](https://github.com/automattic/harper).

**Why:** review from maintainers on real Rust, in a mature codebase, is the strongest
feedback available. A merged PR is also public evidence of Rust work.

**Path:**
1. Read their `CONTRIBUTING.md` and contributor docs, including the policy on AI-assisted contributions.
2. Find a false positive or false negative from your M1b fixtures. That's your issue: open or claim it, and ask whether a fix is wanted.
3. Start small: a phrase correction or a Weir rule (their rule language, no Rust), then a Rust `ExprLinter`.
   Guide: [Author a rule](https://writewithharper.com/docs/contributors/author-a-rule).
4. Open a draft PR early, as their docs recommend.

**Done when:**
- [ ] Issue opened or claimed, with maintainer interest
- [ ] PR opened, one rule only
- [ ] Review feedback addressed
- [ ] `docs/til.md` entry: what the review taught you

**Traps:** a rule nobody asked for; a PR that grows past one rule; Claude writing the PR code (you write it).

---

## M2 — Local service

**Outcome:** a long-running local process that exposes the M1 core to the browser.

**Rust focus:** `async`/`.await`, the `tokio` runtime, `serde` serialization, shared state
(`Arc`), `Send`/`Sync`, graceful shutdown, structured logging (`tracing`).

**Decide → ADR-0003 (browser ↔ core transport):**
- *Native messaging:* the browser launches your binary and talks over stdin/stdout. Each message
  is UTF-8 JSON preceded by a 32-bit length in native byte order. Host → browser max 1 MB.
  The host is registered with a manifest listing `allowed_origins`.
- *Localhost HTTP / WebSocket:* a server on a local port.

**Security:** a port on localhost is reachable by any web page you visit. Whichever transport
you pick, the ADR must say how a random site is prevented from calling the service.

**Done when:**
- [ ] A test client completes a round trip
- [ ] Malformed, empty and oversized input return errors, never panics
- [ ] p50/p95 latency measured on your fixtures and recorded
- [ ] ADR-0003 written, including the security answer

**Traps:** holding a lock across `.await`; blocking calls inside async tasks; `unwrap()` on input.

---

## M3 — Browser extension MVP (TypeScript, thin)

**Outcome:** issues underlined in `textarea` / `contenteditable` fields on 3 sites you use daily;
accept or ignore a suggestion.

**Rust focus:** protocol versioning, input limits, offset conversion on the Rust side.

**Decide:** which fields are watched, and how sensitive fields are guaranteed to be skipped;
when a check fires while you type.

**Done when:**
- [ ] Works on your 3 daily sites (Chrome first)
- [ ] Password and payment fields are never read — covered by a test
- [ ] Typing is not noticeably slower with the extension on
- [ ] Accept/ignore events reach the core (needed for M5)

**Traps:** Manifest V3 service workers stop when idle; Firefox native messaging setup differs from Chrome's.

**Study after yours works:** Harper ships its own Chrome/Firefox extension. Compare how it finds
editable fields and maps positions back to the page.

---

## M4 — Clarity rephrase (local LLM)

**Outcome:** when a sentence's meaning is unclear, propose a clearer rephrase that **keeps the
original meaning**, plus "explain why", from a small local model. The rule engine stays primary.

**Non-goal:** generating ideas or new content. For that, use Claude directly.

Example:
- Input: "something I will like to choose, to apply to get the most to improve efficient"
- Meaning-preserving: "I'd like to choose the option that improves efficiency the most."
- Not acceptable: "I would like to apply something to improve efficiency" — reads better, but drops *choose* and *the most*.

**Rust focus:** streaming responses, timeouts, cancellation of stale requests while you keep typing,
bounded channels and backpressure.

**Decide → ADR-0004 (model runtime and model):**
- Talk to a local model server over HTTP, or embed inference in-process?
- Which small instruct model? Benchmark 2–3 on your fixtures, French included. Don't pick by hype.

**Decide → ADR-0006 (clarity):**
- What makes a sentence "unclear", and who decides: rules, heuristics, the model, or you on demand?
- When a sentence has several possible meanings, what does the tool do?
- How do you check that a rephrase kept the meaning?

**Done when:**
- [ ] ≥ 20 unclear sentences from your own writing (EN + FR) added to fixtures, each with the meaning you intended
- [ ] Meaning-preservation check runs on them, with a method you defined, and its score is recorded
- [ ] Latency on your laptop measured and recorded
- [ ] Typing again cancels the in-flight request
- [ ] LLM output is validated: suggested span exists in the input, no edits outside the sentence
- [ ] ADR-0004 and ADR-0006 written

**Traps:**
- A rephrase can read better and say less. Dropping a clause is a meaning change.
- An unclear sentence can have several meanings. Picking one silently is a guess.
- Non-determinism breaks naive exact-match tests.

---

## M5 — Personalization ("adapts to my style")

**Outcome:** the tool learns from your accept/ignore history without training a model,
and shows *you* your recurring mistakes so you learn from them too.

**Rust focus:** persistence (SQLite), migrations, data modeling, aggregation (grouping, counting,
ranking with iterators or SQL), privacy by design.

**Decide → ADR-0005:** what is stored (as little as possible), where, and for how long.
Then how history changes behaviour. Candidates to evaluate: rule suppression,
personal dictionary, retrieving similar past edits as examples for the LLM.

Also decide: what counts as the *same* mistake? Same rule ID, same word pair
(`will like` → `would like`), same grammatical pattern (adjective where a noun is needed)?
Per language or combined?

**Done when:**
- [ ] A rule you ignore repeatedly stops firing
- [ ] Your own vocabulary (names, jargon, franglais) is no longer flagged
- [ ] A metric you defined improves vs. the M4 baseline on fixtures
- [ ] A `report` command lists your top recurring mistakes per language, with a count and one example each
- [ ] The report runs from stored data only, no model call
- [ ] Stored data can be exported and deleted with one command
- [ ] ADR-0005 written

**Traps:**
- Storing full sentences when a hash or rule ID would do.
- The report wants an example per mistake; minimal storage wants none. ADR-0005 has to resolve that tension.
- Grouping too finely gives a long list of one-off counts; too coarsely gives "grammar: 312".

---

## M6 — Fine-tuning (optional, gated)

**Gate:** start only if M5 metrics plateau **and** you have hundreds of clean accepted edits.

**Reality check:**
- Training tooling is mostly Python. Rust's role here is serving the result, so this milestone
  teaches ML more than Rust.
- Training on your own writing can teach the model your mistakes. A grammar model needs
  clean *(wrong → right)* pairs; a style model needs examples of your voice. Those are two different goals.

---

## Parking lot (not scheduled)

- **Cloud fallback (Claude API)** for hard paragraphs. Sends private text off-machine, so opt-in per site.
  This is where an information-flow policy engine (e.g. OpenAPPA) becomes relevant: label text by
  source (work mail = private) and block the external sink unless sanitized or approved.
- **Agno** is a Python framework — outside the Rust core. Only relevant if you build a separate agent.
- Firefox support, Windows support, installer / packaging.
