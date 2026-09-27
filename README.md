# refine-skills

A Claude Code plugin marketplace containing the `refine` plugin, which provides
the `refine-opus-5-5` skill.

`/refine:refine-opus-5-5` turns an informal task prompt into a visible task
contract with goal, scope, constraints, success criteria, verification and stop
conditions. It shows exactly what it added, then **waits for your approval**
before doing anything.

## Inspiration

This skill is based on
[Getting the Most Out of Opus 5.5 in Claude and Claude Code](https://claude.dev/blog/getting-the-most-out-of-opus-5-5/)
by Addy Osmani (Sep 22, 2026). The article's main prompting advice for Opus 5.5:

- **Give the whole task and name the finish line.** *"Give the whole task in one
  message. Name the finish line, like 'the tests pass' or 'every endpoint is
  migrated.'"*
- **Drop filler instructions.** *"Opus 5.5 always thinks before it replies, and
  it decides how much. You don't need to ask it to think."*
- **Be specific about constraints.** A list of concrete patterns works better
  than vague directives.
- **Define stopping points.** Decide up front when to proceed and when to ask,
  especially before anything destructive.
- **Ask for verification.** Make it clear how results should be checked and
  flagged.

Most of us don't write prompts like that on the first try. This skill applies
the advice for you: it fills in the missing success criteria, scope,
constraints, verification and stop conditions, and removes filler like
"think step by step". It shows every addition so you learn what your original
prompt was missing, and you stay in control of what actually runs.

## Install

```
/plugin marketplace add thiphu1012/refine-skills
/plugin install refine@team-skills
```

Restart Claude Code if prompted.

## Use

```
/refine:refine-opus-5-5 <your task prompt>
```

Claude replies with a prompt assessment, the proposed task contract and a
"What changed" list. Then reply with one of:

| Reply | Effect |
| --- | --- |
| `run` | Execute the proposed prompt |
| `edit: <instruction>` | Revise the proposal |
| `original` | Execute your original prompt as written |
| `cancel` | Stop |

Nothing is executed until you reply `run` or `original`.

### Example

```
/refine:refine-opus-5-5 fix pagination in the users table
```

Claude proposes a contract scoped to the pagination bug, with success criteria
(e.g. correct page boundaries, page count updates after filtering) and the
verification that applies in your repo. Then it stops and waits for you.

## Update

```
/plugin marketplace update team-skills
```
