# Intake

Working folder for **everything you know about a topic before there's a formal spec to put it in**. Two kinds of content land here:

1. **Interview Q&A** — when Claude grills a UX/product person about a feature (often after they paste in a Figma screenshot or sketch a flow).
2. **Information from clients and stakeholders** — facts, business rules, constraints, and decisions that arrive async from recorded calls, design reviews, or chat threads. No question attached; just "here's something we know".

The goal is to stop losing this material in Figma developer notes, Slack threads, and meeting recordings. Capture it here, then write the formal spec from it.

## One file per topic

```
intake/
├── README.md         ← you are here
├── _template.md      ← copy this when starting a new topic
└── <topic-slug>.md   ← one file per feature/topic; accumulates over time
```

A topic is usually a feature-in-the-making. The slug becomes the eventual `features/<slug>/` folder name once the spec is written, so pick it carefully — but it's fine to rename later if the framing shifts.

**There is no per-session file and no meeting metadata.** This folder is not a meeting log. Each topic file accumulates content over weeks; nothing about dates, attendees, or which meeting an item came from.

## How to use it

**Adding inbound info (no question).** Open `<topic-slug>.md`, find the *What we've been told* section, add a bullet. That's it. No format ceremony.

**Running an interview.** Use the [`spec-intake`](../.claude/skills/spec-intake/SKILL.md) skill — it wraps the [`grill-me`](../.claude/skills/grill-me/SKILL.md) interviewing style with file-writing as a side effect. It will create or open the topic file, run the interview, and append each resolved Q&A pair to the *Interview Q&A* section as the conversation proceeds, so nothing is lost mid-session. The original `grill-me` skill is still available for plan-pressure-testing where no file capture is needed.

**Writing the spec.** When you're ready, read the topic's intake file in full and use it as raw material for the formal spec. The intake stays where it is — flip its `Status:` line from `collecting` to `spec-written` and link the new spec files at the bottom. Do not delete anything; the intake is the audit trail.

## File shape

See [`_template.md`](./_template.md) for the canonical shape. It has four sections, in this order:

- **What we've been told** — standalone facts (bullets, no question).
- **Interview Q&A** — `**Q:** ... **A:** ...` pairs from grill sessions.
- **Open** — unresolved items. Annotate with a resolution date when answered; don't delete.
- **Linked specs** — populated when the formal spec lands.

## Lifecycle

- A topic file is born with `Status: collecting`.
- It accumulates content for as long as you keep finding things out about that topic.
- When the formal spec is written, flip to `Status: spec-written` and list the spec links.
- The file then goes quiet but stays in the repo as history. Don't delete.
- If the topic gets cancelled, mark `Status: abandoned` with a one-line reason. Still don't delete.

## What goes here vs. in the formal spec

| Goes in `intake/` | Goes in `features/<name>/` |
|---|---|
| Open questions before a spec exists | Open questions about the spec, in the use-case file |
| Client-provided business rules awaiting formal framing | Acceptance criteria phrased as testable conditions |
| Brain-dump of "things we know" with no structure yet | Structured use-case files following `conventions.md` |
| Q&A pairs from grill sessions | Cross-references between use-cases, data-model, api-contract |

Rule of thumb: if the same information ends up in both places, the spec is the source of truth — the intake is the history of how we got there.
