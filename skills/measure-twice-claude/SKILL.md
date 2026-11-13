---
name: measure-twice-claude
description: "Claude Code edition of measure-twice; when measure-twice is also installed, use this one. A full cycle to design, implement, audit and close a change in any project: agents that read the code and test the app (each on the model tier that fits its job), brainstorming with the user, a written plan with success criteria and an independent critique, an audit and a closing pass based on systematic debugging, and the user's decisions at each gate. Use it when the user asks for brainstorm or brainstorming (for example, utiliza brainstorm), a plan with an audit, an audit of what was implemented, or names measure-twice-claude. In those cases it replaces measure-twice and the superpowers brainstorming and systematic-debugging skills."
---

# Measure twice — Claude

How to design, build, audit and close a change in any project.
**Agents read and test; you talk, write and implement; the user decides.**

This skill is self-contained: it does not depend on `measure-twice`, `brainstorming` or
`systematic-debugging`. If those exist and say something different, this one wins.

Two reference files hold the detail you need only at specific moments. Read each one when
you reach that moment, not before:

- `references/agent-briefs.md` — before launching any agent (steps 2, 5, 7b, 10, 15).
- `references/templates.md` — before writing the plan (step 7) or the audit (step 11).

## Before you start

1. **Check for a cycle already in progress.** Look in the project's plans folder for a
   plan on this topic whose Status is not closed. If there is one, go to "Resuming" at
   the end of this file instead of starting over.
2. Read the project's rules: `CLAUDE.md` / `AGENTS.md` and the project memory. That is
   where you find what must not break, the verification gate (build, lint, tests), where
   plans live, and which actions have external effects.
3. Find the folder where the project keeps plans and audits (files like `*-plan-*.md` or
   `*-audit*.md`) and follow its naming. If there is none, ask once where to put them and
   record the answer in the project memory.
4. Classify the request and say it out loud, so the user can correct it:
   - **Spike:** a feasibility question ("can we…?"). The output is an answer, not code
     that stays.
   - **Bounded:** a contained change to a flow that already exists in the repo and can be
     read. If the flow does not exist yet, it is not bounded.
   - **Architectural:** something new, a subsystem, or a change to how parts fit together
     or to interfaces other code depends on.

   If unsure, pick the heavier path. If complexity shows up midway, move up a path and
   say so. Never move down.
5. If the harness has a task list, put the cycle's steps on it and keep it current. The
   plan's **Status** line is still the source of truth across sessions.

## Systematic debugging, in four phases

The cycle refers to these phases by number. They are defined here once.

- **Phase 1 — Evidence.** Reproduce it, or trace it: what enters and leaves at each
  boundary between components, and where each piece of data comes from and goes.
- **Phase 2 — Compare.** Set it against what works (the precedent, the previous version,
  a neighboring flow) and list every difference, including configuration, flags and
  environment.
- **Phase 3 — Hypothesis.** One at a time, stated as "X breaks because Y", tested with the
  smallest possible check.
- **Phase 4 — Fix.** A failing test that shows the error, a single fix, and verification
  that the test passes and nothing else broke.

Agents only ever do Phases 1 to 3, and Phase 3 in writing. Phase 4 is yours, and only for
what the user approved.

## Who does what

| Role | Does | Never |
|---|---|---|
| **User** | Decides: which idea, whether the plan is approved, what to do with each finding | — |
| **You (main model)** | Talk with the user, write the agents' briefs, write the files, implement, confirm findings | Implement without approval |
| **A — reads** | Code map, precedents, impact map | Edit, or run anything that changes state |
| **B — tests** | Uses the project for real: the app, the tests, read-only queries and, when needed, temporary logs in a worktree | Edit the working code; cause external effects without permission |
| **C — audits** | Critiques the plan before it is approved; compares what was implemented against the plan and looks for errors, by reading | Edit or fix |

Agents do not see the conversation. Each brief carries everything: the request, what was
decided, the project's rules, the relevant paths and what to return. Write in the brief
what the agent must not do, and when the harness offers an agent type whose tools already
enforce it (a read-only type for A and C), prefer that type: a missing tool is a stronger
guarantee than a sentence. When two agents run in parallel, launch them in the same
message.

**If something is unclear, say so** ("I don't understand X") instead of guessing. This
applies to you and to all three agents, and it goes in every brief.

## Which model does each job

Each job goes to the cheapest tier that does it well. The audit is the exception: it gets
the strongest read available, because a missed error costs more than every agent in the
cycle put together.

| Tier | What it is good for here | Claude models |
|---|---|---|
| **Deep** | Design trade-offs, finding errors by reasoning, judging whether a fix is correct | Fable 5.2 (`fable`), Opus 5.5 (`opus`) |
| **Standard** | Reading and following flows across many files, driving the app and the browser, running and interpreting tests | Sonnet 5.5 (`sonnet`) |
| **Fast** | Locating files and usages, running a fixed list of commands and returning the output | Haiku 5.5 (`haiku`) |

| Job | Step | Tier | Model to try first | When to change it |
|---|---|---|---|---|
| **You:** conversation, design, plan, implementation, confirming findings | all | Deep | The session's model (Fable 5.2 or Opus 5.5) | If the session runs on a lower tier, say so at step 1: the design and the confirmation of findings are where it shows |
| **A:** map | 2 | Standard | Sonnet 5.5 | Fast (Haiku 5.5) on a bounded path where the job is only "find the flow and who uses it" |
| **A:** impact map | 5 | Standard, continued | The same agent A, continued | A fresh Deep agent (Opus 5.5) with A's report when the map shows concurrency, transactions, migrations or more than one subsystem |
| **B:** how it behaves today; the plan's tests; closing tests | 2, 10, 15 | Standard | Sonnet 5.5 | Fast (Haiku 5.5) when B's whole job is a fixed list of commands with no live tests and no browser |
| **B:** temporary logs | on demand | Standard | Sonnet 5.5 | Deep if a first instrumented run did not explain the behavior |
| **C:** plan critique | 7b | Deep | Opus 5.5 | Fable 5.2 if you are running on Opus |
| **C:** audit | 10 | Deep | Opus 5.5 | Fable 5.2 if you are running on Opus |
| **C:** review of the fixes' diff | 15 | Deep | The same model as step 10 | Standard (Sonnet 5.5) only when every fix is low or medium severity and touches one function |

Rules for choosing:

- **C reads with a different model than the one that wrote the code, when there is one of
  the same tier.** If you implemented on Fable 5.2, C runs on Opus 5.5, and the other way
  round. A second model does not share the first one's blind spots; the same model tends
  to agree with itself.
- **Never go below Standard for anything that judges** (C in any step, A's impact map).
  Fast is for finding and running, not for deciding whether something is right.
- **Escalate on evidence, not in advance.** If an agent returns a thin or contradictory
  report, or says "I don't understand" about something that is in its brief, first fix
  the brief; if the brief was complete, relaunch one tier up and note it.
- **A continued agent keeps its model.** Continuing A at step 5 is worth more than a
  higher tier, because it has already read the code; switch only in the cases in the
  table.
- **In Claude Code,** pass the alias in the Agent tool's `model` parameter. If the harness
  cannot choose models, agents inherit yours and this section reduces to "launch fewer,
  better-briefed agents".
- **With another provider,** map by tier, not by name: that provider's strongest
  reasoning model is Deep, its general-purpose model is Standard, its small fast model is
  Fast. The rule about C using a different model still applies.
- Record in the plan's Evidence and in the audit which model produced each report. A
  finding from a Fast agent that nobody confirmed is a lead, not a finding.

## The cycle

| # | Who | What | Output |
|---|---|---|---|
| 1 | You and the user | Framing: 1-2 questions about what it is and why | — |
| 2 | A and B, in parallel | A: map. B: how it behaves today | Two reports |
| 3 | You and the user | Brainstorming: size, questions, success criteria, 2-3 ideas | Ideas in the chat |
| 4 | **User** | **Picks an idea** | Decision 1 |
| 5 | A, continued | Impact map of the chosen idea | Impact map |
| 6 | You and the user | Design by sections, asking "does this look right?" after each | Design validated part by part |
| 7 | You | Write the plan and self-review it | Plan file |
| 7b | C | Critique of the plan against the map (architectural only) | Objections, resolved in the plan |
| 8 | **User** | **Approves or corrects the plan** | Decision 2 |
| 9 | You | Implement in phases, running the gate after each | Code and tests |
| 10 | B and C, in parallel | B: the plan's tests. C: audit | Results and findings |
| 11 | You | Confirm the findings and write the audit | Audit file |
| 12 | You and the user | Brainstorming on the change map | — |
| 13 | **User** | **Decides each change** | Decision 3 |
| 14 | You | Implement what was approved, one fix at a time | Fixes |
| 15 | B, C and you | Closing pass with full systematic debugging | Closing in the audit; index ✅ |

Step 15 closes the cycle. Another audit round only if the user asks for it.

**By path:**
- **Spike:** say in 2-3 sentences what you will investigate and how, and wait for a yes.
  Then B (and A if needed) investigate, and you answer in the chat with a
  recommendation. No files. Anything built is throwaway; keeping it is a new request.
- **Bounded:** A covers only the flow that changes and whatever uses it, and returns the
  "could break" list in the same report, so step 5 is not a separate run. B covers only
  that flow's behavior. Steps 3 to 8 may be a single exchange: the approach you recommend,
  one alternative, and the short plan file, with one approval that counts as Decisions 1
  and 2. Say that you are doing it this way; if the user wants to discuss the idea first,
  go back to the full sequence. Steps 9 to 15 as written.
- **Architectural:** the whole cycle, including 7b.

**Clean audit.** If step 11 leaves no confirmed finding and every test of B passed, there
is nothing to decide or fix: tell the user, skip steps 12 to 14, and write the Closing
from step 10's results without rerunning B and C. A test B could not run is not a pass:
it goes to the user as an open item (see "Closing pass").

## Brainstorming, step by step

Where the harness has a tool for asking the user multiple-choice questions, use it at the
decision gates (steps 4, 8, 13) and for the questions in step 3.

**Step 3 — before the ideas**
1. **Size it.** If the request bundles several independent systems, say so right away
   and propose splitting it: which pieces, how they relate, and in what order. Each piece
   gets its own cycle; start with the first.
2. **Ask one question at a time.** Multiple choice when possible; open questions are
   fine too. Cover purpose, constraints, and what is out of scope. Concrete beats abstract
   ("what would make you say this went wrong?"). A guess the user can correct is often
   worth more than a question.
3. **Fix the success criteria.** Before proposing anything, write down 2 to 6 statements
   that can be checked ("a second message from the same number does not create a second
   application") and have the user confirm them. They go into the plan, each one tied to
   a test, and the closing pass answers them one by one.
4. **Propose 2-3 approaches** with pros and cons. Lead with the one you recommend and say
   why. Build on what A and B brought back: each approach says which parts of the map it
   touches and which success criteria it covers. Cut anything that is not needed.

**Step 6 — the design, by sections**
- Present the chosen idea's design in parts: architecture, components, data flow, error
  handling, testing and, when data or interfaces change shape, migration and how to go
  back. Size each part to what it needs: a few sentences if simple, up to 200-300 words
  if it has nuance.
- After each part, ask whether it looks right. If something does not fit, go back.

**Step 12 — on the audit**
- For each finding: fix now, accept as a gap, or leave for later. If a fix can be done
  more than one way, propose 2-3 options with your recommendation.
- Group the low-severity findings so the user can decide them together; present critical
  and high ones one by one.

**Always:** mark your own guesses as guesses. Only what the user said goes in as theirs.

## Implementation (step 9)

1. **Record the starting point** in the plan before the first edit: the branch and the
   commit hash. C's diff in step 10 is taken from that hash, not from a base branch that
   may include earlier work. Note any uncommitted changes that were already there, so
   they are not audited as part of this change.
2. Follow the project's rules about branches and commits. Do not commit or push unless
   the user or the project's rules say so.
3. **One phase at a time,** in the plan's order. After each phase run the project's
   verification gate (build, lint, typecheck, tests) and do not start the next phase on a
   red gate.
4. **Deviations are written when they happen,** in section 10 of the plan: what changed
   from the plan and why. Small ones (a different file name, an extra helper) you note
   and continue.
5. **If the plan turns out to be wrong** (a step cannot be done as written, or doing it
   would break a success criterion or a project rule), stop and take it to the user with
   the options. The approval covered the plan as written, not its replacement.
6. Update the plan's Status at the end of each phase.

## The agents

Each agent is a fresh subagent that knows only its brief. `references/agent-briefs.md`
has the brief structure, the exact steps for each agent at each point of the cycle, B's
limits and the procedure for temporary logs. Read it before launching anything. The table
below is only the schedule.

| Cycle step | Agents | How | Edits anything? |
|---|---|---|---|
| 2 | **A** (map) + **B** (how it behaves today) | In parallel, same message | No. A only reads; B runs things but edits no code |
| 5 | **A continued** (impact map) | Alone | No |
| 7b | **C** (plan critique) | Alone | No |
| 10 | **B** (the plan's tests) + **C** (audit) | In parallel, same message | No. C only reads |
| 15 | **B** (closing tests) + **C** (review of the fixes' diff) | In parallel, same message | No |

A never takes part in the audit or the closing; C never takes part in steps 2 or 5. You
write every file and every line of code.

While agents work, do something useful that does not overlap with them, such as reading
the code you will need to confirm their findings. Never predict or report a result before
it arrives.

**When an agent fails.** A report that is empty, cut off, off the brief or contradicts
itself is not a result. Continue the same agent with the specific gap ("you did not cover
the consumers of X"), or relaunch it with a narrower brief. Do not fill the gap with what
you assume it would have found, and do not present a partial report as complete: say what
was not covered.

## Findings (step 11)

**Severity,** the same scale for C, for you and for the audit file:

| Severity | Means |
|---|---|
| **Critical** | Data loss or corruption, a security or privacy leak, an external effect that should not happen (a message to the wrong person, a double charge), or the main flow does not work |
| **High** | A success criterion is not met, or a common case gives a wrong result |
| **Medium** | An uncommon case gives a wrong result, or an error is handled in a way that hides it |
| **Low** | Correct today but fragile: it breaks under a change that is likely to come |

Style, naming and preferences are not findings.

**Confirming:**
1. For each finding of C, and each failure of B, read the cited code yourself.
2. For critical and high findings that describe behavior and can be reproduced within B's
   limits, have B reproduce them (continue the B from step 10). A finding that was
   reproduced is marked **confirmed (reproduced)**.
3. Mark the rest **confirmed** if the code shows it, **unconfirmed** if you could not
   check it, or drop it and say why if it does not hold.
4. Write the audit file and update the plan's Status.

## Fixes and closing

### Fixes (step 14)
Phase 4, only for what the user approved:
- One fix at a time. When possible, first a failing test that shows the error; then the
  fix; then check that the test passes and nothing else broke.
- No "while I'm at it": anything not approved is noted and taken to the user.
- If a fix fails three times (here or in the closing pass), stop: the problem is probably
  in the design. Go back to brainstorming with the user on that point.

### Closing pass — full systematic debugging (step 15)
Uses the whole cycle: the plan, the audit with the user's decisions, and the diff of the
fixes.
- **B:** reruns the plan's live test list, the tests, and one test per fixed error
  (no regressions).
- **C:** reviews only the diff of the fixes: does each one do what was approved? Does it
  break anything around it?
- **You:** go through the success criteria one by one and write, for each, the test that
  proves it and its result.
- **If something approved fails,** you apply Phases 1 to 4 in full, because B and C do
  not edit. New issues are not fixed: they are noted and the user decides.
- **Tests that could not run.** Each one goes to the user: run it together now, accept it
  as a gap, or leave the cycle open. You do not decide this.
- **Result:** the "Closing" section of the audit, section 10 of the plan, and the status.
  The index turns ✅ only if the closing pass succeeds: the gate is green, every success
  criterion has a passing test or a gap the user accepted, and no approved fix is
  pending.

If B's whole job is commands you can run yourself (build and tests, with no live tests),
you may run them instead and say so in the closing. C still runs as an agent.

## The files

Templates, the plan's self-review and the writing rules are in
`references/templates.md`. In short: one plan file and one audit file per cycle, named
`YYYY-MM-DD-plan-<topic>.md` and `YYYY-MM-DD-audit-<topic>.md` unless the project has its
own naming; written in the user's language; evidence as `file:line`; the plan's
**Status** updated at every step.

## Resuming

When a plan for this topic is already in progress, or the conversation was summarized
midway:
1. Read the plan's Status, its section 10 and, if it exists, the audit. They say which
   step the cycle is on and what was decided.
2. Compare with the repo: `git status`, the current commit against the starting point
   recorded in the plan, and whether the gate is green.
3. Tell the user in two or three lines where the cycle stands and what the next step is.
   If the files and the repo disagree (code the plan does not mention, a Status that is
   behind), say so and ask before continuing.
4. Decisions already recorded are not asked again. Agents from an earlier session are
   gone: relaunch them with their earlier report in the brief.

## Signs you are skipping the process

| Thought | Reality |
|---|---|
| "It's simple, it doesn't need a plan" | Simple means a short plan, not no plan. |
| "I'll start while the user reads the plan" | Approval is the gate. Wait for the yes. |
| "They said 'sounds good' to the idea" | Approval covers only what the user has seen. The plan is approved separately, except on the bounded path when you showed both together. |
| "I don't quite get this, but I'll keep going" | Say it: "I don't understand X". Guessing costs more. |
| "The plan was wrong here, I'll do it the right way" | Stop and take it to the user. The approval covered the plan as written. |
| "I'll drop a few quick logs in the working branch" | Only in a temporary worktree, deleted afterwards. |
| "C could fix this quickly" | C only proposes. Fixes go through step 12. |
| "The audit came back clean, so it works" | Without B's tests it is not verified. |
| "The agent's report is thin, but I can guess the rest" | Continue it or relaunch it. A guess is not a report. |
| "A cheaper model is fine for the audit, the diff is small" | Small diffs hide errors too. C stays on the Deep tier, except as the table allows in step 15. |
| "I'll audit it myself, I know this code" | You wrote it. C is a different agent and, when possible, a different model. |
| "While I'm here, I'll fix this other thing" | It goes into the plan or the audit as a separate item. |
| "That fix didn't work, I'll try another" (third time) | Stop and go back to brainstorming: it's a design problem. |
| "The closing pass found something new; I'll fix it on the way" | New issues are noted and the user decides. |
| "That live test couldn't run, but the rest passed" | It goes to the user. Only they can accept it as a gap. |
| "One more audit round" | Only if the user asks for it. |
| "B needs to send a real email to test this" | B stops and you ask the user. |
