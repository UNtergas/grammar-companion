# Architecture Decision Records

One file per decision: `NNNN-short-title.md`, copied from `0000-template.md`.
Write the ADR **before** the code it governs. Supersede, don't edit history.

## Open decisions

| ADR  | Question | Milestone | Status |
|------|----------|-----------|--------|
| 0001 | Single crate or workspace? What does each crate own? | M0 | open |
| 0002 | Which checking engine(s) for EN and FR? Merge rule for overlaps? Behaviour when an engine is down? | M1b | open |
| 0003 | Browser ↔ core transport, and how random websites are kept out | M2 | open |
| 0004 | Model runtime (HTTP server vs in-process) and which model | M4 | open |
| 0005 | What history is stored, where, how long, how it changes behaviour, what counts as the "same" mistake | M5 | open |
| 0006 | What counts as "unclear", how ambiguity is handled, how meaning preservation is checked | M4 | open |
| 0007 | Core types: what a diagnostic carries, position unit, what a rule receives and returns | M1a | open |

Numbers are assigned in the order questions were raised, not milestone order.
