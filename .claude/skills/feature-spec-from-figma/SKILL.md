---
name: feature-spec-from-figma
description: Turn Figma screenshots and a brief use-case description into a written feature specification ready for the development team and Claude Code. Use this skill whenever the user shares screenshots from Figma, design mockups, or prototype frames and wants them turned into a spec, requirements doc, use case writeup, or implementation-ready document. Also trigger when the user wants to capture a UI flow for engineering handoff, document a feature for development, or write requirements based on visual designs — even if they don't explicitly say "spec." The skill enforces a clarify-before-draft workflow with structured question categories (visual states, data, behavior, alternative flows, platform divergence) so silent assumptions don't get baked into the spec.
---

# Feature spec from Figma

Turn a set of Figma screenshots and a short framing message into a written feature specification that the development team and Claude Code can act on. The spec is the contract — it has to capture not just what's drawn, but state behavior, data sources, edge cases, and platform-specific differences that aren't visible in any single frame.

## The workflow

Three phases: **input → clarify → draft**. The clarify phase is the one that earns its keep. Skipping it produces specs full of silent assumptions, and the "Open questions" section becomes a dumping ground instead of a focused list of real product decisions.

Do not jump straight to drafting. Even if the user seems impatient, run the clarify phase — at minimum, identify the gaps and present them. The user can then say "skip those, just assume X" if they want to move fast, but that's a deliberate choice rather than an oversight.

## Phase 1: Read the input

Before asking anything, do this in order:

1. **Look for project conventions.** Check the workspace for a `conventions.md` (typically in a `specs/`, `docs/`, or similarly named folder) that defines the project's spec template and writing rules. If one exists, follow it exactly — the template, section order, naming rules, all of it. Only use the default template at the bottom of this file if no conventions exist.

2. **Look at every screenshot.** Note which states are present (happy path, empty, loading, error, disabled, success, skeleton) and which are missing. Missing states are findings, not blockers — they go into the question list.

3. **Read the framing message.** Identify: which use case this is, where it sits in the project structure, who the actor is, and any context that's not visible in the design (data sources, business rules, prior decisions, dependencies on other features).

4. **Map what you have against the template.** Section by section, decide: can I fill this confidently from what I have, or do I need to ask?

## Phase 2: Ask clarifying questions

Group questions into a small number of categories so the user can answer in batches rather than wading through a flat list of 30. Skip categories that aren't relevant to this particular use case.

**Visual and state gaps.** For each interactive element on screen: when is it disabled, and what does that look like? What's the loading state? Empty state? Error state? Success feedback? Tap/press feedback? If Figma is silent on a state, call it out as a finding — don't silently invent one.

**Data gaps.** For everything on screen that isn't literal text: where does it come from (which endpoint, cached, computed client-side)? What's the format (units, rounding, date format, localization)? What shows when the value is null, zero, very long, very stale, or boundary cases? What are the validation rules? How fresh does the data need to be?

**Behavior gaps.** What happens on tap, long-press, swipe, back? What animates and what doesn't? Where does each link go (especially navigation that isn't obvious from the static frames)? What state survives navigation and what gets reset?

**Alternative flows.** What if the user has no permission for this, no data yet, no network, an outdated app version? What if the action fails halfway through? What if they navigate away mid-flow? What if they trigger this twice in quick succession?

**Cross-platform divergence.** Should the surfaces in scope behave identically here? Are there platform conventions (haptics, sheet vs. dialog, navigation patterns, system back behavior) that should legitimately differ even though the Figma is platform-agnostic?

## Sort questions into three buckets

This is the most important discipline of the skill. Not every gap needs an answer before drafting:

- **Blocking** — must be resolved or the spec will be wrong. Ask the user explicitly. Keep this list short; flag clearly that these block the draft.
- **Assumption you'll make** — you can pick a sensible default. State the assumption in the spec ("Assumption: list is paginated at 20 items per page") so the reviewer can correct it in one round of review rather than three.
- **Open question for the spec** — a genuine product or design decision that hasn't been made yet. Capture it in the "Open questions" section of the spec. Don't pretend to have an answer, and don't block the draft on it.

When presenting questions to the user, sort them this way and tell them which bucket each falls into. Communicate clearly: a useful spec with four flagged open questions is better than no spec while everything is resolved. The user shouldn't feel they need to answer 30 things before any draft gets written.

## Phase 3: Draft

Once the blocking questions are answered, write the spec. Follow the project's `conventions.md` if it exists; otherwise use the default template below.

A few drafting rules:

- **Keep platform notes light on the first draft.** They tend to grow in review when surface-owning engineers see them. Don't try to nail every detail before letting the team in.
- **State your assumptions explicitly** in the relevant section. Don't bury them. The reviewer should be able to scan for them.
- **Don't reproduce the Figma in prose.** The Figma is the visual reference. The spec captures *behavior, data, edge cases, and platform differences* — things the Figma can't show.
- **The spec defines the rules of the system, not the current values.** "Users can have at most N items in their list" belongs in config or seed data, not in the spec. The spec says "the set of items is defined in the `<ConfigTable>` table; the apps render whatever is returned."
- **Don't invent implementation details.** Naming a specific UI framework class or library is out of place in a spec. The surface is named; the implementation choice lives in code.

After writing, expect one round of follow-ups. The act of writing the spec out usually surfaces a couple of things neither you nor the user spotted in the screenshots.

## What to request upfront if the user hasn't provided it

If the user just says "make a spec from this" without enough context, ask for:

- **All screenshots for the use case in one message** — every state Figma has, not just the happy path. Multi-screen flows: every screen.
- **A short framing message** — use case name, where it sits in the project, who triggers it, what's happening from the user's perspective, anything that's not in Figma but matters.
- **The Figma prototype URL** if there is one. You can't open it, but having it lets you ask about specific navigation links explicitly ("the prototype shows tapping the card goes where?") rather than guessing.
- **For the first use case on a new project**, suggest picking one with at least one alternative flow and one async/data interaction. A trivial read-only screen won't pressure-test the workflow, and both the user and the team will get false confidence that everything is fine.

## Default spec template (fallback only)

**Use this only if the project has no `conventions.md`.** If one exists, follow that exactly instead.

```markdown
# [Use case name]

## Summary
One paragraph: what the user is trying to do and why it matters.

## Actors and preconditions
Who triggers this and what state the system must be in for the flow to start.

## Main flow
Numbered steps describing system behavior (not UI clicks). Each step says what happens, not how it looks. Aim for 5–10 steps.

## Alternative flows
- What if [edge case]?
- What if [error]?
- What if [permission/state precondition fails]?

## Acceptance criteria
Pass/fail conditions a tester can verify. Gherkin-compatible (Given/When/Then) is fine but not required. Aim for 5–15 criteria.

## Platform notes

### Backend
Endpoints, evaluation logic, auth, response shapes.

### Data
Tables, columns, keys and constraints, seed data, indexing.

### <Client surface 1>
Platform-specific behavior, navigation patterns, local caching, system integration.

### <Client surface 2>
Platform-specific behavior, navigation patterns, local caching, system integration.

## Configuration
What's data-driven vs. hardcoded. Where the configuration lives.

## Out of scope
Explicit list of what this spec does NOT cover. Prevents Claude Code and reviewers from over-reaching.

## Open questions
Numbered (Q1, Q2…) so they can be referenced in PRs and follow-up discussions.
```

## A note on rerunning the skill

The user may run this workflow many times, once per use case. Each run produces one spec. If the project has multiple related use cases under one feature, the user is responsible for the feature-level README and cross-cutting concerns; this skill focuses on individual use case specs.
