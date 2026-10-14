---
name: measure-twice
description: "A full cycle to design, implement, audit and close a change in any project: agents that read the code and test the app, brainstorming with the user, a written plan, an audit and a closing pass based on systematic debugging, and the user's decisions at each gate. Use it when the user asks for brainstorm or brainstorming (for example, utiliza brainstorm), a plan with an audit, or an audit of what was implemented. In those cases it replaces the superpowers brainstorming and systematic-debugging skills."
---

# Measure twice

How to design, build, audit and close a change in any project.
**Agents read and test; you talk, write and implement; the user decides.**

This file is self-contained: it holds everything this cycle takes from the superpowers
`brainstorming` and `systematic-debugging` skills, rewritten in our own terms. If those
skills exist in the project and say something different, this file wins.

## Before you start

1. Read the project's rules: `CLAUDE.md` and the project memory. That is usually where
   you find what must not break, where plans live, and which actions have external
   effects.
2. Find the folder where the project keeps plans and audits (files like `*-plan-*.md` or
   `*-audit*.md`) and follow its naming. If there is none, ask once where to put them and
   record the answer in the project memory.
3. Classify the request and say it out loud, so the user can correct it:
   - **Spike:** a feasibility question ("can we…?"). The output is an answer, not code
     that stays.
   - **Bounded:** a contained change to a flow that already exists in the repo and can be
     read. If the flow does not exist yet, it is not bounded.
   - **Architectural:** something new, a subsystem, or a change to how parts fit together
     or to interfaces other code depends on.

   If unsure, pick the heavier path. If complexity shows up midway, move up a path and
   say so. Never move down.

## Who does what

| Role | Does | Never |
|---|---|---|
| **User** | Decides: which idea, whether the plan is approved, what to do with each finding | — |
| **You (main model)** | Talk with the user, write the agents' briefs, write the files, implement, confirm findings | Implement without approval |
| **A — reads** | Code map and precedents | Edit, or run anything that changes state |
| **B — tests** | Uses the project for real: the app, the tests, read-only queries and, when needed, temporary logs in a worktree | Edit the working code; cause external effects without permission |
| **C — audits** | Compares what was implemented against the plan and looks for errors, by reading | Edit or fix |

Agents do not see the conversation. Each brief carries everything: the request, what was
decided, the project's rules, the relevant paths and what to return. Use `general-purpose`
agents and write in the brief what they must not do. When two agents run in parallel,
launch them in the same message.

**If something is unclear, say so** ("I don't understand X") instead of guessing. This
applies to you and to all three agents, and it goes in every brief.

## The cycle

| # | Who | What | Output |
|---|---|---|---|
| 1 | You and the user | Framing: 1-2 questions about what it is and why | — |
| 2 | A and B, in parallel | A: map. B: how it behaves today | Two reports |
| 3 | You and the user | Brainstorming: size, questions, 2-3 ideas | Ideas in the chat |
| 4 | **User** | **Picks an idea** | Decision 1 |
| 5 | A, continued | Impact map of the chosen idea | Impact map |
| 6 | You and the user | Design by sections, asking "does this look right?" after each | Design validated part by part |
| 7 | You | Write the plan and self-review it | Plan file |
| 8 | **User** | **Approves or corrects the plan** | Decision 2 |
| 9 | You | Implement in phases | Code and tests |
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
- **Bounded:** A covers only the flow that changes and whatever uses it; B, only that
  flow's behavior. The design fits in one or two sections and the plan is short. Steps
  10 to 15 in full.
- **Architectural:** the whole cycle.

## Brainstorming, step by step

**Step 3 — before the ideas**
1. **Size it.** If the request bundles several independent systems, say so right away
   and propose splitting it: which pieces, how they relate, and in what order. Each piece
   gets its own cycle; start with the first.
2. **Ask one question at a time.** Multiple choice when possible; open questions are
   fine too. Cover purpose, constraints, what success looks like and what is out of
   scope. Concrete beats abstract ("what would make you say this went wrong?"). A guess
   the user can correct is often worth more than a question.
3. **Propose 2-3 approaches** with pros and cons. Lead with the one you recommend and say
   why. Build on what A and B brought back: each approach says which parts of the map it
   touches. Cut anything that is not needed.

**Step 6 — the design, by sections**
- Present the chosen idea's design in parts: architecture, components, data flow, error
  handling and testing. Size each part to what it needs: a few sentences if simple, up
  to 200-300 words if it has nuance.
- After each part, ask whether it looks right. If something does not fit, go back.

**Step 12 — on the audit**
- For each finding: fix now, accept as a gap, or leave for later. If a fix can be done
  more than one way, propose 2-3 options with your recommendation.

**Always:** mark your own guesses as guesses. Only what the user said goes in as theirs.

## The briefs

### A — map (step 2)
- Walk end to end through the flows the request touches: entry points, where each piece
  of data goes, what enters and leaves at each boundary between components (routes,
  services, external APIs, databases), flags and configuration.
- Find **who else uses** each piece that might change, not just the main path.
- Precedents: how existing code or the previous version solves the same thing today
  (read it in full, not a skim); what earlier plans and audits of that area say
  (decisions, open items, accepted gaps); the git history of that area.
- Returns a map with `file:line`, the differences from the precedent, and the project
  rules that apply. Not whole files.

### B — how it behaves today (step 2)
- Uses the real flow. In web apps, the built-in browser (`preview_start` with the
  project's configuration, if the environment has it); in CLIs or APIs, the terminal. If
  there is a previous version of the flow, it tests that one too.
- Runs the existing tests for that area.
- If the flow does not exist yet, it tests the neighboring flows the request will touch,
  or is skipped.
- Returns what happens today, step by step, with evidence as text: what it saw, console
  errors, responses. Screenshots stay in its own context.

### A continued — impact map of the chosen idea (step 5)
Continue the same agent A if you can, since it has already read the code. If not, launch
a new one with A's report. It applies systematic debugging without touching code:
- **Phase 1:** what enters and leaves at each boundary after the change; where each new
  piece of data comes from and where it goes.
- **Phase 2:** compare against the precedent and list **every** difference, saying
  whether it is intended; dependencies on configuration, flags and environment.
- **Phase 3, in writing only:** "I think X could break because Y". Each one becomes a
  verification item in the plan.
- **Never Phase 4:** no new tests, no fixes.

### B — the plan's tests (step 10)
- Goes through the plan's live test list and runs the tests.
- Returns each test with its result (pass / fail / could not run) and evidence as text.

### C — audit (step 10)
Receives the plan's path, the diff (`git diff` against the base branch, plus uncommitted
changes), the project's rules and A's map. Read-only.
- Was each step of the plan done as written?
- What was done outside the plan?
- Errors, following the data flow of the changed code (Phases 1 and 2), including the
  consumers in A's map.
- Only what would cause real problems; no style comments. If nothing serious turns up, it
  says so.
- For each finding: severity (critical / high / medium / low), evidence as `file:line`,
  whether the change **introduced it or it already existed**, and the change it proposes.

### B's limits, always
- Local only, with the project's test data (seeds, fixtures, example configuration).
  Never real credentials.
- Nothing with effects outside the machine without the user's permission: real emails or
  messages, writes to third-party systems (CRM, payments…), production data, paid calls
  that are not on the plan's list. The project's rules say which actions those are
  there.
- Does not write to databases on its own. Whatever the app saves while being used is part
  of the test.
- If a test needs any of the above, it stops and reports it; you ask the user.

### Temporary logs (when needed)
Only when reading the code and testing the app do not explain what is happening. If an
agent runs into that, it says so; you launch a separate B to instrument. Tell the user
in one line what will be instrumented and why.
- **Where:** in its own worktree, never in the user's working folder or branch. In
  Claude Code, launch the agent with `isolation: "worktree"`.
- **Setup:** git does not copy what it ignores. Copy what the app needs into the worktree
  (such as `.env`) without showing its contents, and install or link the dependencies. If
  what is being diagnosed is not committed, apply the diff there. If the user's app is
  already running, use another port.
- **Logs only observe:** they do not change logic, do not print keys or credentials, and
  the report does not copy personal data.
- **Output:** a report for whoever needed it. What is useful goes into the evidence of
  the plan or the audit, noted as "obtained with temporary logs".
- **When done:** delete the worktree and its branch, only the ones created for this
  (check the name first). They are never pushed or merged.

## Fixes and closing

### Fixes (step 14)
Phase 4 of systematic debugging, only for what the user approved:
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
- **If something approved fails,** you apply full systematic debugging, because B and C
  do not edit. Full means: Phase 1, reproduce it and gather evidence at each boundary;
  Phase 2, compare with what works; Phase 3, one hypothesis at a time, tested with the
  smallest possible change; Phase 4, a failing test, a single fix, and verification. New
  issues are not fixed: they are noted and the user decides.
- **Result:** the "Closing" section of the audit, section 10 of the plan, and the status.
  The index turns ✅ only if the closing pass succeeds.

## The files

Default names: `YYYY-MM-DD-plan-<topic>.md` and
`YYYY-MM-DD-audit-<topic>.md`, in the project's plans folder. If the
project already has its own naming, follow it. If the folder has an index (README), add
one line per file with its status: 🟡 in progress or waiting on a decision · ✅
implemented and verified (after the closing pass) · 🔴 with an open error.

Write the files in the user's language, section titles included.

### Plan

```
# Plan — <topic>
**Status:** step N — <step name>
**Background:** why this plan exists

## 0. Summary              short, in plain words
## 1. Decisions            the user's, with dates
## 2. Evidence             from A and B, with file:line
## 3. What already exists  the precedent and each difference: intended or not
## 4. The steps            per step: what, why, how it solves it,
                           files and boundaries, risk, how it is verified
## 5. Flows
## 6. Implementation phases
## 7. Verification         tests and the live test list for B
## 8. Open points
## 9. Out of scope and accepted gaps
## 10. Implementation: what changed from the plan   (filled in at the end)
```

**Self-review before showing it (step 7):**
1. Gaps: "TBD", "TODO", empty sections or vague requirements.
2. Contradictions between sections.
3. Scope: does it fit in one plan, or does it need splitting?
4. Ambiguity: can any requirement be read two ways? Pick one and write it down.

Fix things as you go; no need to repeat the review.

### Audit

```
# Audit Report — <topic> (`systematicDebugging`)
**Result:** two or three lines

## 1. What was implemented   files; B's tests: verified live / not verified
## 2. Errors found           E1…En
## 3. Change map             table: E#, file, proposed change, why, risk,
                             user's decision
## 4. Deviations from the plan
## 5. Open items and gaps
## 6. Closing                per error: fixed and its no-regression check;
                             B's and C's results in step 15
```

Each error carries: how it was found (reading, live test, test, temporary logs), the
evidence or the reproduction, the cause, the difference from the precedent, whether the
change introduced it or it already existed, the severity, the proposed change, and
**confirmed** or **unconfirmed**.

Before writing down one of C's findings, check it yourself by reading the cited code. If
it does not hold up, mark it "unconfirmed" or drop it and say why.

### How to write
- In the user's language. The short, plain version first; the detail after.
- Evidence as `file:line`. Anything not verified is marked as a guess.
- Decisions are attributed to the user, with dates.
- Update **Status** at every step, so another session knows where to pick up.

## Signs you are skipping the process

| Thought | Reality |
|---|---|
| "It's simple, it doesn't need a plan" | Simple means a short plan, not no plan. |
| "I'll start while the user reads the plan" | Approval is the gate. Wait for the yes. |
| "They said 'sounds good' to the idea" | Approval covers only what the user has seen. The plan is approved separately. |
| "I don't quite get this, but I'll keep going" | Say it: "I don't understand X". Guessing costs more. |
| "I'll drop a few quick logs in the working branch" | Only in a temporary worktree, deleted afterwards. |
| "C could fix this quickly" | C only proposes. Fixes go through step 12. |
| "The audit came back clean, so it works" | Without B's tests it is not verified. |
| "While I'm here, I'll fix this other thing" | It goes into the plan or the audit as a separate item. |
| "That fix didn't work, I'll try another" (third time) | Stop and go back to brainstorming: it's a design problem. |
| "The closing pass found something new; I'll fix it on the way" | New issues are noted and the user decides. |
| "One more audit round" | Only if the user asks for it. |
| "B needs to send a real email to test this" | B stops and you ask the user. |
