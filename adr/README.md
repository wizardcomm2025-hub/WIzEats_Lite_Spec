# Architecture Decision Records

Numbered, immutable records of significant architectural decisions for WhizEats POS Lite.

**Status: nothing here yet.** No ADRs have been written. The settled platform choices for this project are recorded in [`../CLAUDE.md`](../CLAUDE.md) under "Platform decisions". When one of those decisions needs to be revisited, or a new architectural question arises that crosses platforms, capture it as an ADR here.

## Naming

`NNNN-short-slug.md` — e.g. `0001-use-<database>-for-primary-storage.md`. Numbers are sequential and never reused.

## Format

```
# NNNN. Title

Status: Proposed | Accepted | Superseded by NNNN

## Context
What forces are at play? What constraints, requirements, or prior decisions led to needing this one?

## Decision
What did we decide? State it plainly.

## Consequences
What follows from this decision — both the good and the bad? What does it lock us out of?
```

## Rules

- Once an ADR is **Accepted**, do not edit it. If the decision changes, write a new ADR that supersedes it and update the old one's status to `Superseded by NNNN`.
- ADRs document **decisions and trade-offs**, not how-to guides. If you find yourself writing implementation steps, you are writing the wrong document.
- Keep them short. One page is usually enough. If an ADR is growing past two pages, the decision is probably not crisp yet.

See the matching section in [`../conventions.md`](../conventions.md) for the canonical rules.
