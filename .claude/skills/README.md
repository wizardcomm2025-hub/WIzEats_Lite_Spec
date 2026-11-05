# Spec-authoring skills

Skills here help with **writing and maintaining specifications** in this repo. They are not code skills. Each skill is a folder containing a `SKILL.md` file with frontmatter (`name`, `description`) and a procedure. They follow the standard Claude Code skill format.

When Claude Code runs in the spec repo, it auto-discovers these skills from `.claude/skills/`. You can also invoke a skill explicitly by asking for it by name.

## Installed skills

| Skill | When to use |
|---|---|
| [`spec-intake`](./spec-intake/SKILL.md) | Interview a UX or product person about a feature they're scoping, and capture everything to `intake/<topic>.md` so nothing is lost. Use when the user shares a Figma screenshot and wants to be grilled, or wants to dump client-provided business rules into a central place. Wraps `grill-me` with file-writing as a side effect. |
| [`feature-spec-from-figma`](./feature-spec-from-figma/SKILL.md) | The user has Figma screenshots (or any visual mockup) and wants a feature spec written. Enforces the clarify-before-draft workflow so silent assumptions don't end up in the spec. |
| [`grill-me`](./grill-me/SKILL.md) | The user wants to be relentlessly interviewed about a plan or design before any drafting starts — useful for pressure-testing an ADR, or whenever the user says "grill me" but doesn't need the transcript captured. For feature scoping where the Q&A should be saved, use `spec-intake` instead. Vendored verbatim from Matt Pocock's [mattpocock/skills](https://github.com/mattpocock/skills) (`skills/productivity/grill-me/SKILL.md`); to update, pull from upstream. |

## How the skills relate to each other

The typical lifecycle of a feature is: **`spec-intake` → `feature-spec-from-figma` → done**.

1. `spec-intake` runs the interview and captures Q&A and standalone facts into `intake/<topic>.md`. Use it at the discovery stage, often before there's anything visual.
2. `feature-spec-from-figma` takes Figma screenshots plus the intake file as input and produces the formal spec under `features/<name>/`. The intake's Q&A becomes the source of "we already decided that", which keeps the new spec's Open questions section honest.
3. `grill-me` is a general-purpose interviewing skill kept around for plan pressure-testing where no capture is needed. Don't reach for it during feature scoping — use `spec-intake` so you don't lose the Q&A.

## Adding a new skill

1. Create `<skill-name>/SKILL.md` in this folder.
2. Start the file with frontmatter:
   ```
   ---
   name: <skill-name>
   description: <one paragraph that says when to trigger and what the skill does — Claude reads this to decide whether to invoke>
   ---
   ```
3. Write the procedure below the frontmatter. Be specific: what to do first, what to ask, what to produce, and any anti-patterns to avoid.
4. Add a row to the table above with a one-line "when to use" description.
5. Describe the change the same way as any spec change (see `../../README.md`).

A few skills you may want next: an ADR-drafting skill, a feature-readme generator, and a `consistency-check` skill that flags drift between a use-case spec and the data-model / api-contract files in the same feature. Add them as the need arises.

## Using these skills from an implementation repo

This folder sits next to `laravel-pos-api-dev_guid` and `POS_APP-master`. Claude Code running in one of those repos only auto-discovers skills from that repo's own `.claude/` folder — not from `App-Specs/.claude/`. Two options:

1. **Run spec work here (recommended).** Open the Claude Code session in `C:\whizeats-lite\App-Specs`; the skills auto-discover.
2. **Manual invocation.** From an implementation repo, tell Claude to use the skill at `../App-Specs/.claude/skills/<name>/SKILL.md`.
