---
name: system-study
description: Analyze any system (repo, framework, running app, protocol, doc URL) through 8 fixed first-principles steps, pausing after each for the user's reaction, then write a Markdown report plus an archify HTML version, then interview the user from the finished report to check they understood. Use when the user says "study X", "understand how X works", "learn X to implement it myself", "analyze this system", "first principles", "nghiên cứu hệ thống", or asks how to approach a system they want to rebuild or adopt.
---

# system-study

Goal: the user ends able to explain the system and rebuild a small version of it, not recall a feature list.
Input: any system the user names (repo path, running instance, framework name, doc URL). Inspect it with real
tools: read the code, run it, trace a request. Do not analyze from memory when you can check.

## Rules

- No upfront quiz. Ask at most one short question first, only if the system or the user's goal is unclear (use AskUserQuestion).
- Run the 8 steps in order, one at a time. After each step, show its result (short) and stop for the user's reaction. Fold their corrections into the report. If they say "continue" or "run all", stop pausing.
- Every claim cites evidence: `file:line`, a command you ran with its result, or a doc URL. If unchecked, mark it `unverified`.
- Be concise per step: a table, a diagram or a few bullets, not an essay. Match the user's language. Use they/them for people.

## The 8 steps

1. **Find the invariant.** The one idea that generates the rest (e.g. "schema generates DB, API and UI"). One sentence, then 3-5 features it explains.
2. **Follow the data.** Table: each piece of state | where born | who may change it | where stored | who reads it. ASCII flow diagram. Flag state with two writers.
3. **Find the seams.** Where things change at different rates or for different reasons (schema vs content, config vs code, auth vs logic). For each seam: what is on each side and why the boundary is there.
4. **Trace one real request.** Pick a concrete operation (propose one, let the user change it). Run it if possible. Follow it layer by layer with evidence. Mark which layers are generic framework and which are specific to this case.
5. **Cost and benefit of each design choice.** The 3-5 most important choices: what it bought, what it cost, the alternative. If you can't name the cost, investigate before writing.
6. **Failure modes.** How it breaks on wrong input, a down dependency, or a hostile caller. Test at least one on the running system when safe.
7. **Smallest version.** A 100-500 line miniature that keeps the invariant: pieces in build order, which will be hard, what it omits. Do not scaffold it unless asked.
8. **Build or adopt on the hard 20%.** Name the parts that look easy but cost the most (relations, i18n, migrations, permissions, scale). Recommend, with evidence and cost, for the user's goal.

## Output

After step 8:

1. Write `system-study-<system>-<yyyymmdd>.md` in `plans/reports/` of the current repo if that directory exists, else the working directory. One section per step, in order, plus a final "Open questions and next steps" section with a concrete first action. Under ~250 lines, evidence inline.
2. Invoke the `archify` skill on that report to produce the explorable HTML version (diagrams for the data flow, seams and traced request; the invariant and trade-offs as panels). Save it next to the Markdown file with the same name and `.html`.
3. Give the user both paths.

## Interview (after the output)

Generate 4-6 questions from the finished report that test understanding, not recall: predict behavior in a changed scenario, explain why a design choice exists, spot the failure in a short snippet. Ask them with AskUserQuestion or in chat, one round at a time. Grade each answer honestly (solid / partial / missing); for partial or missing, explain once and re-ask a variant. Append the questions, answers and grades to the Markdown report under "Teach-back results" and list the 2-3 gaps to study next.
