---
name: spec-intake
description: Interview a UX or product person about a feature they're scoping, and capture everything to the project's intake folder so nothing gets lost. Use whenever the user wants to be grilled about a feature, shares a Figma screenshot and asks for clarifying questions, dumps client-provided business rules and wants them recorded, or says "spec intake on <topic>". Wraps the grill-me interviewing style with a file-writing contract: every resolved Q&A is appended to `intake/<topic-slug>.md` as the conversation proceeds, and standalone facts (no question) go into the same file's "What we've been told" section. Distinct from `grill-me`, which is interview-only with no file capture.
---

# Spec intake

Interview a UX or product person about a feature they're scoping, and capture the conversation to disk so it survives the session. The intake file becomes the raw material for the formal spec later.

## When to use this skill

- The user pastes a Figma screenshot or sketch and asks for clarifying questions.
- The user says "grill me on <feature>" or "spec intake on <feature>".
- The user wants to dump client-provided information and have it recorded somewhere central (away from Figma notes / Slack threads).
- Any time you would have used `grill-me` for a feature scoping session and want the result captured.

If the user just wants to pressure-test a plan with no need to keep the transcript, use plain `grill-me` instead.

## Phase 1: orient yourself in the project

1. **Read `intake/README.md`** to confirm the folder structure and conventions for this project. If it doesn't exist, the project hasn't adopted the intake pattern — stop and tell the user.
2. **Read `intake/_template.md`** so you know the section shape you'll be writing into.
3. **List existing topic files** (`intake/*.md` excluding README and template). If the user's topic matches one, you'll append to it rather than create a new file.

## Phase 2: name the topic

Ask the user one short question:

> What topic should I file this under? (suggest a slug, e.g. `achievements`, `physio-dashboard`, `onboarding`)

If an existing intake file matches, confirm:

> I found `intake/<slug>.md` already. Append to it, or start a new one?

If creating a new file, copy `intake/_template.md` to `intake/<slug>.md` and replace the `# <Topic>` header with the real topic name. Leave `Status: collecting` as-is.

## Phase 3: handle the inbound case

If the user starts with **standalone information** rather than a question for them ("the client just told us X and Y, save it"), don't run an interview. Open the topic file, append each fact as a bullet under **What we've been told**, and confirm with the user when done. Then offer: "Want me to grill you on anything related, or are we done for now?"

If the user wants to **be interviewed**, continue to phase 4.

## Phase 4: interview, writing as you go

Use the `grill-me` interviewing style:

- One question at a time.
- Walk each branch of the decision tree, resolving dependencies between decisions one-by-one.
- For each question, provide your recommended answer.
- If a question can be answered by reading existing project files (`project-overview.md`, `conventions.md`, other intake files, existing specs), read those instead of asking.

After **each Q&A is resolved** (user gives an answer you're confident enough about to write down), immediately append it to the file's **Interview Q&A** section in this exact format:

```
- **Q:** <the question you asked, verbatim or near-verbatim>
  **A:** <the user's answer, paraphrased or quoted; preserve their meaning>
```

Do not batch writes to the end of the session. Append after every resolved pair. If the session ends abruptly (user steps away, runs out of time), the file should still contain everything that was actually answered.

If the user **provides a fact without you asking** mid-interview ("oh, also, the client said X"), pause the interview, append the fact to **What we've been told**, then return to the next question.

If a question is **raised but unresolved** (user says "I need to check with the client" or "we haven't decided"), append the question to the **Open** section, then move on. Do not pretend it was answered.

## Phase 5: close the session

When the user signals they're done (or you've exhausted the decision tree):

1. Read back a one-paragraph summary of what's now captured in the file.
2. Point out anything that's still in **Open** that they should chase down before spec writing starts.
3. **Do not flip the `Status:` field.** That's a human decision, made when the formal spec is being written. Leave it on `collecting`.

## Writing rules

- **Append, never edit prior entries.** History matters; if a previous answer was wrong, add a new entry that supersedes it rather than overwriting.
- **Q&A entries stay short.** If an answer is long (multi-paragraph), capture the gist as the answer and link to a separate document if the user has one. Long prose belongs in the spec, not in intake.
- **No invented detail.** If you weren't told something, don't write it. If a recommendation you proposed got accepted, that's fine to record — but mark it: `**A:** Yes, your recommendation: <restate>`. Don't claim the user said something they didn't.
- **Preserve code/identifier names exactly.** If the user names a field, table, column, endpoint, stored procedure, or evaluator, write it the way they said it. These names propagate into specs.

## What this skill does not do

- It does not write or modify formal specs in `features/<name>/`. That's a deliberate next step the user takes after the intake is rich enough.
- It does not flip `Status: spec-written` or create links from intake to specs. The human owns that transition.
- It does not delete anything from the intake file, ever.
