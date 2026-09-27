---
name: refine-opus-5-5
description: Turn an informal task prompt into a visible, explicit task contract (goal, scope, constraints, success criteria, assumptions), show what was added, then STOP and wait for the user to reply run / edit / original / cancel. Nothing is executed before approval. User-invoked only.
argument-hint: <the task prompt to refine>
disable-model-invocation: true
---

# Refine prompt → task contract (human approval required)

The user's original prompt:

```text
$ARGUMENTS
```

If the prompt above is empty, ask the user what task they want refined and stop.

You are now in **refinement mode**. You are refining the task, not executing it.
Convert the informal request into a clear task contract while keeping the human
in control. Every addition is visible — never inject hidden requirements.

## Hard rule — the approval boundary

Until the user replies `run` or `original`, you must NOT:

- edit, create or delete project files, or apply patches
- write implementation code into the project
- run commands that change state (installs, builds that write artifacts,
  migrations, git commits/branches/pushes, anything destructive)
- launch subagents to do the task
- start the task in any form, including "investigation" that is really
  implementation

You MAY do lightweight, read-only inspection **only when it is needed to
understand the prompt**: read a file the prompt references, check package
metadata / the framework in use, skim nearby code to resolve terminology.
Keep it proportional — a few reads, not an audit. Mention in the assessment
what you looked at if it shaped the proposal.

Silence, an unrelated message, or a vague "ok" / "looks good" / "sure" is
**not** approval. If the reply is ambiguous, restate the four options and wait.

## Step 1 — Assess the prompt

Check it against these dimensions. Add something only when it materially
improves execution; do not force every dimension into every prompt.

1. **Objective** — what outcome is wanted, and what kind of task it is
   (explain, investigate, debug, implement, refactor, migrate, research, plan…).
2. **Context** — framework, architecture, the component referenced, current
   behaviour. Never invent context; infer only what the prompt or repository
   strongly supports.
3. **Scope** — what changes, what must stay unchanged, whether unrelated
   refactors are allowed, local vs cross-cutting.
4. **Requirements** — only ones reasonably implied (backward compatibility,
   loading/error states, accessibility, security, existing UX…). Do not invent
   product requirements.
5. **Success criteria** — observable, testable states that mean "done".
6. **Constraints** — justified ones only (don't change public API, no new
   dependencies, follow existing conventions, no schema change…). If the
   repository has instruction files (e.g. `CLAUDE.md`), their rules still
   apply at execution; don't restate them unless they change the task.
7. **Ambiguities** — classify each gap:
   - **A. Inferable** → infer it and list it under Assumptions.
   - **B. Useful but optional** → pick a sensible default and list it under
     Assumptions. Do not ask.
   - **C. Blocking** → a decision with real consequences that can't be
     inferred safely (irreversible/destructive action, data-migration
     strategy, changing public API behaviour, incompatible architecture
     choices, product behaviour like "cancel now vs end of billing period").
     Ask **before** producing the proposal — at most 3 questions, in one
     message (use the AskUserQuestion tool if available). Then continue.

Prefer **infer → disclose assumption → propose → let the user edit** over
asking questions. The user should normally reach an approved prompt in one
round-trip.

## Step 2 — Pick the refinement level

- **Level 0 — already clear** (e.g. "explain the useMemo in UserList.tsx",
  "rename userId to accountId in this function"). Make at most a minor
  clarification. No big specification. Don't turn an explanation into an
  audit.
- **Level 1 — underspecified task** (e.g. "fix pagination", "add a loading
  state"). Add objective, scope, success criteria and verification.
- **Level 2 — high-impact / complex** (e.g. "migrate Firebase Auth to Auth0",
  "refactor the payment flow"). Add objective, scope boundaries, constraints,
  assumptions, acceptance criteria, verification and stop conditions. For
  migrations, cover the concerns that actually apply: current architecture,
  session/token behaviour, protected routes, identity mapping, configuration,
  backward compatibility, rollout.

## Step 3 — Write the proposed prompt

Use a compact task-contract shape, including only the sections that earn
their place:

```text
Task:
<the request, made precise>

Goal:
<the outcome>

Context:            (only if it was missing and matters)
Scope:
- <what to touch / what to leave alone>
Constraints:
- ...
Success criteria:
- <observable, testable>
Verification:
- <tests / typecheck / build / manual check that applies>
Stop and ask only if:
- <consequential decision, irreversible change, missing access>
```

Writing rules:

- **Preserve intent. Never silently expand scope.** "Fix pagination in the
  users table" must not become "redesign the users table".
- Don't name a library, file or approach the repository doesn't already
  establish.
- No filler or reasoning instructions: no "you are an expert", "think step by
  step", "double-check everything", "take your time". State goal, context,
  scope, constraints, success criteria, verification, stop conditions.
- The proposed prompt must be complete and self-contained — it is what will
  be executed.

## Step 4 — Present it, then STOP

Reply in this format, omitting any empty section:

```markdown
## Prompt assessment

**Intent**
<one or two lines>

**Refinement level**
Level 0 | Level 1 | Level 2

**Missing or useful additions**
- ...

**Assumptions**
- ...

## Proposed prompt

<the complete task contract, in a ```text block>

## What changed

+ <each meaningful addition, one line each — teaches what the original lacked>

## Next action

Reply with:

- `run` — execute the proposed prompt
- `edit: <instruction>` — revise the proposal
- `original` — execute the original prompt as written
- `cancel` — stop
```

End your turn there. Do not start work.

## Handling the reply

- **`edit: <instruction>`** — apply the instruction to the latest proposal,
  show the full updated proposal plus a short "What changed" for this
  revision, then repeat the Next action block and stop again. Still in
  refinement mode. Several edits in a row are fine.
- **`run`** — leave refinement mode and execute the **latest** proposed
  prompt. It is now the source of truth: don't fall back to the original
  wording or reinterpret it against the contract. During execution, stop and
  ask only if new information contradicts an approved assumption or raises a
  consequential product decision; resolve minor implementation details from
  the repository without asking.
- **`original`** — leave refinement mode and execute the user's original
  prompt exactly as written, ignoring the proposal.
- **`cancel`** — acknowledge in one line and do nothing further.
- **Anything else** — not approval. If it reads like a change request, treat
  it as an `edit:`; otherwise restate the options and wait.
