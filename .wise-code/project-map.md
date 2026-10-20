# Project Map

## Purpose
A browser companion that checks grammar and spelling in French and English as the learner types,
and adapts to the learner's writing style. Built primarily to learn Rust.

## Requirements
- Works in the browser on text the learner types (stated)
- French and English (stated)
- Lightweight model, adapted to the learner's writing style (stated)
- Rust is the learning focus (stated)
- Rephrase suggestion when a phrase's meaning is unclear, keeping the original meaning (stated)
- Not idea or content generation; that goes to Claude directly (stated)
- Report of the learner's own top recurring mistakes, to learn from them (proposed by AI, accepted by learner)
- Local-first; typed text stays on the machine by default (proposed by AI, not yet confirmed)

## Components
Nothing chosen or implemented yet.

Proposed (AI, earlier discussion — not decisions):
- Rust core: checking logic, diagnostics
- Thin TypeScript browser extension
- Local process bridging browser and core
- Optional small local model for rewrites and explanations
- Local store of accept/ignore history for adaptation

## Main Flow
Proposed only; every arrow is unresolved.

```
text field --(?)--> extension --(transport ?)--> Rust core --(engine ?)--> diagnostics
                                                      |
                                                      +--(?)--> local model (optional)
                                                      +--(?)--> history store (?)
```

## Data and Trust Boundaries
- Input text is private (emails, work messages) and comes from arbitrary web pages: unverified handling.
- Sensitive fields (password, payment) must never be read: requirement proposed, mechanism unknown.
- A localhost service is reachable by any visited web page: mitigation unknown.
- Any cloud call would move private text off-machine: out of scope until explicitly decided.

## Build and Deployment
Nothing verified yet. No Cargo project exists.

## Known Constraints (verified Oct 2026)
- harper-core (Rust grammar checker) supports English only.
- LanguageTool supports French; runs as a local Java HTTP server, `/v2/check` endpoint.
- Chrome native messaging: stdio, 32-bit native-endian length prefix + UTF-8 JSON, host-to-browser max 1 MB, host manifest with `allowed_origins`.

## Unknowns
- Crate layout (ADR-0001)
- Diagnostic shape; position unit; rule abstraction (ADR-0007, M1a)
- Engine strategy for EN and FR; overlap merging; engine-down behaviour (ADR-0002, M1b)
- Browser ↔ core transport and its security model (ADR-0003)
- Model runtime and model choice (ADR-0004)
- Clarity detection, ambiguity handling, meaning-preservation check (ADR-0006)
- Storage contents, location, retention; adaptation method (ADR-0005)
