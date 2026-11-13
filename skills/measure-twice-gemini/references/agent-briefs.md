# Agent briefs

What each agent does, in order, at each point of the cycle. Copy the steps into the
brief. The tier for each agent is in "Which model does each job" in `SKILL.md`; the
model, the agent type and how to launch it are in `references/models.md`.

## Contents

- How to write every brief
- Step 2 — A: map
- Step 2 — B: how it behaves today
- Step 5 — A continued: impact map
- Step 7b — C: plan critique
- Step 10 — B: the plan's tests
- Step 10 — C: audit
- Step 15 — B: closing tests
- Step 15 — C: review of the fixes' diff
- B's limits, always
- Temporary logs

## How to write every brief

Write it in this order:
1. **Role and prohibitions first.** For example: "You are agent C (audit, read-only). Do
   not edit or create files in the repo, do not run the app, do not write to databases,
   do not commit." Temporary files go to the scratchpad, never the repo.
2. **Project:** path, stack and how to run what the agent needs (test command, database,
   ports). Nothing more.
3. **Context:** the request in a few lines, the success criteria, the user's decisions
   with dates, the project rules that apply (from `AGENTS.md`, `GEMINI.md` or the
   project's rules file, and the memory), and the plan sections and files to read, with
   line numbers when you have them. Agents do not see the conversation: anything left out
   does not exist for them.
4. **What is already verified,** so they do not repeat it (for example "the tests run
   against the test database; I checked"), and **what another agent covers,** so they do
   not overlap.
5. **The task as numbered steps,** taken from this file.
6. **What to return and in what shape:** tables, `file:line`, severity, pass / fail. Ask
   for the conclusion first and the evidence after, and for a final section "Not covered"
   listing what the agent did not get to or could not check. An agent that says nothing
   about its gaps is assumed to have covered everything.
7. **Always this line:** "If something is unclear, say so ('I don't understand X')
   instead of guessing."

For C, give the code and the plan, not your opinion of them. A brief that says "I think
the retry logic is fine, but check it" gets back "the retry logic is fine".

Launch agents that run in parallel in the same message, in the background.

## Step 2 — A: map

- **Gets:** the request, the paths you already know, the earlier plans and audits of the
  area. On the bounded path, also the change you expect to make.
- **Steps:**
  1. Find the entry points of every flow the request touches.
  2. Follow each flow end to end: where each piece of data goes, and what enters and
     leaves at each boundary between components (routes, services, external APIs,
     databases).
  3. Note the flags, configuration and environment variables each flow depends on.
  4. For each piece that might change, find **who else uses it**, not just the main path.
  5. Read the precedent in full, not a skim: how existing code, or the previous version,
     solves the same thing today.
  6. Read what earlier plans and audits of the area say: decisions, open items, accepted
     gaps.
  7. Read the git history of that area: the `git log` output you saved in `scratch/`,
     since a read-only agent cannot run `git`.
  8. Note which tests cover each flow today, and which parts have none.
  9. Bounded path only: for the expected change, write the "could break" list as in
     step 5 below.
- **Never:** edit, or run anything that changes state.
- **Returns:** the map with `file:line`, the differences from the precedent, the existing
  test coverage, and the project rules that apply. Excerpts, not whole files.

## Step 2 — B: how it behaves today

- **Gets:** the flow to exercise, how to start the app and run the tests, the test data
  to use, and B's limits (below).
- **Steps:**
  1. Before running anything that writes, confirm that it hits the test environment (test
     database, test number, sandbox keys) and not the user's working data. If it cannot
     confirm it, it stops and asks.
  2. Record the state of the working tree (`git status` and a hash of `git diff`).
  3. Run the project's verification gate and the existing tests for that area, and record
     the result as the **baseline**: which tests fail or are unstable before any change.
     Step 10 is compared against this.
  4. Use the real flow: in CLIs or APIs, the terminal; in web apps, the UI, through a
     separate B-browser (a `browser` agent) that gets the URL and the steps and returns
     what it saw as text. See "Agent types" in `references/models.md`.
  5. If there is a previous version of the flow, test that one too.
  6. If the flow does not exist yet, test the neighboring flows the request will touch;
     if there are none, B is skipped.
  7. Record the state of the tree again. If it changed while B was testing, say so and
     rerun on the final state.
- **Never:** edit the working code, or step outside B's limits.
- **Returns:** the baseline; what happens today, step by step, with evidence as text: the
  exact commands, what it saw, console errors, responses. Screenshots stay in its own
  context.

## Step 5 — A continued: impact map of the chosen idea

Continue the same agent A if you can, since it has already read the code. If not, launch
a new one with A's report.
- **Gets:** the chosen idea, the success criteria and the design sections validated so
  far.
- **Steps** (systematic debugging without touching code):
  1. **Phase 1:** for each boundary in the map, what enters and leaves after the change;
     where each new piece of data comes from and where it goes.
  2. **Phase 2:** compare against the precedent and list **every** difference, saying
     whether it is intended; list the dependencies on configuration, flags and
     environment.
  3. **Phase 3, in writing only:** "I think X could break because Y". Each one becomes a
     verification item in the plan.
  4. For anything that changes the shape of stored data or of an interface other code
     uses: what happens to what already exists (rows, queued jobs, callers), and whether
     the change can be undone.
- **Never:** Phase 4. No new tests, no fixes.
- **Returns:** the impact map and the list of "could break" items.

## Step 7b — C: plan critique (architectural path)

A fresh agent that has not seen the brainstorming. It reads the plan the way the audit
will later read the code, while changing it is still cheap.
- **Gets:** the plan's path, A's map and impact map, the project's rules. Not the
  conversation, and not your reasons for preferring this design.
- **Steps:**
  1. For each success criterion: which step of the plan achieves it, and which test
     proves it? List criteria with no step or no test.
  2. For each "could break" item in the impact map: does the plan handle it or verify it?
  3. For each step: could it be carried out by someone who only has the plan? List what
     is ambiguous or missing (a file, an order, a decision).
  4. Check the plan against the project's rules and against A's map: steps that
     contradict a rule, or that assume something the map does not show.
  5. Look for what the plan does not mention: consumers in the map that no step touches,
     existing data, order of deployment, how to go back.
  6. Keep only what would make the implementation fail or the result wrong. If the plan
     holds, say so.
- **Never:** edit the plan; propose a different design; comment on wording.
- **Returns,** per objection: the plan section, what is wrong, the scenario where it
  fails, and the smallest change to the plan that would resolve it.

You resolve each objection in the plan, or note next to it why it does not apply, before
showing the plan to the user. Tell the user in one or two lines what the critique changed.

## Step 10 — B: the plan's tests

- **Gets:** the plan's path (its live test list and its criteria-to-test table), the
  baseline from step 2, how to run the tests, B's limits.
- **Steps:**
  1. Confirm the test target, as in step 2.
  2. Run the build gate the project requires (lint, typecheck, build), the automated
     tests, and each test in the plan's live list, in order.
  3. Run the automated tests twice, to see whether they are stable and whether they leave
     data behind.
  4. Compare with the baseline: tests that passed before and fail now, tests that failed
     before and still fail (not caused by this change), and tests that are new.
  5. A live test that needs something outside B's limits (a real number, a paid call, a
     QR scan) is not run: it is reported as "could not run", with the reason and what
     would be needed to run it.
- **Never:** edit code or tests to make them pass.
- **Returns:** each test with pass / fail / could not run, its evidence as text, the
  comparison with the baseline, and the totals (tests, assertions).

## Step 10 — C: audit

- **Gets:** the plan's path, the diff from the starting commit recorded in the plan (plus
  uncommitted changes, minus the ones that were already there) saved by you as a file in
  `scratch/`, the project's rules and A's map.
- **Splitting C.** Launch more than one C, in the same message, when either applies:
  - The change spans independent areas (for example engine and domain): one C per area,
    each with its own files.
  - The path is architectural: one C for **conformance** (steps 1, 2 and 4 below) and one
    for **errors** (steps 3 and 5). They are different kinds of reading, and an agent
    asked to do both tends to do the first well and the second quickly.
- **Steps:**
  1. For each step of the plan, check whether it was done as written.
  2. List what was done outside the plan, and compare it with section 10 of the plan:
     deviations that are not recorded there are findings.
  3. Follow the data flow of the changed code (Phases 1 and 2), including the consumers
     in A's map, looking for errors. Pay particular attention to the "could break" items
     in the plan.
  4. If there is an earlier audit or report of this work, check each of its claims
     against the code and list the ones that do not hold.
  5. Read the new tests: does each one prove what it claims, or would it pass with the
     code broken?
  6. Keep only what would cause real problems. If nothing serious turns up, say so.
- **Never:** edit or fix; comment on style.
- **Returns,** per finding: severity (critical / high / medium / low, as defined in
  `SKILL.md` — paste the table into the brief), evidence as `file:line` with the key
  snippet, a concrete scenario (input → wrong result), whether the change **introduced it
  or it already existed**, how sure it is and what it could not check, and the change it
  proposes, in words.

## Step 15 — B: closing tests

- **Gets:** the plan, the audit with the user's decisions, and the list of fixes with
  the test written for each one in step 14.
- **Steps:**
  1. Confirm the test target, as in step 2.
  2. Rerun the build gate, the automated tests (twice) and the plan's live test list.
  3. Run the test of each fixed error.
  4. Note any test whose result changed compared with step 10.
- **Returns:** the same as in step 10, plus one line per fixed error with its test and
  the result.

## Step 15 — C: review of the fixes' diff

- **Gets:** only the fixes: what was approved for each one, the files and functions it
  touched, its new tests, and the project's rules. Not the whole diff of the cycle.
- **Steps:**
  1. For each fix: does it do what was approved, no more and no less?
  2. Does it break anything around it: consumers, concurrency, transactions, types?
  3. Does it make reachable a problem that was hidden before (code that never ran and
     now does)?
  4. Do the new tests prove what they claim, or would they pass with the code broken?
- **Never:** edit, or audit code the fixes did not touch.
- **Returns:** a verdict per fix (correct / partial / wrong / introduces a regression)
  with `file:line`, the new problems with severity and scenario, and the tests that do
  not prove what they claim.

## B's limits, always

- Local only, with the project's test data (seeds, fixtures, example configuration).
  Never real credentials.
- Nothing with effects outside the machine without the user's permission: real emails or
  messages, writes to third-party systems (CRM, payments…), production data, paid calls
  that are not on the plan's list. The project's rules say which actions those are
  there.
- Does not write to databases on its own. Whatever the app saves while being used is part
  of the test.
- If a test needs any of the above, it stops and reports it; you ask the user.

## Temporary logs (when needed)

Only when reading the code and testing the app do not explain what is happening. The
agent that runs into that says so; you launch a separate B to instrument, and tell the
user in one line what will be instrumented and why. That B:
1. Works in its own isolated workspace, never in the user's working folder or branch:
   launch it with `Workspace: "branch"`.
2. Copies into the workspace what the app needs and git ignores (such as `.env`), without
   showing its contents, and installs or links the dependencies.
3. If what is being diagnosed is not committed, applies the diff there. If the user's app
   is already running, uses another port.
4. Adds logs that only observe: they do not change logic and do not print keys,
   credentials or personal data.
5. Reproduces the case and collects the output.
6. Reports to whoever needed it, without copying personal data. What is useful goes into
   the evidence of the plan or the audit, noted as "obtained with temporary logs".
7. When it is done, you end that agent with `manage_subagents` (`kill`), which deletes its
   workspace. Check first that it is the logging agent. Its changes are never pushed or
   merged.
