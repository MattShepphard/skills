# The files

Default names: `YYYY-MM-DD-plan-<topic>.md` and `YYYY-MM-DD-audit-<topic>.md`, in the
project's plans folder. If the project already has its own naming, follow it. If the
folder has an index (README), add one line per file with its status: 🟡 in progress or
waiting on a decision · ✅ implemented and verified (after the closing pass) · 🔴 with an
open error.

Write the files in the user's language, section titles included.

## Plan

```
# Plan — <topic>
**Status:** step N — <step name>
**Path:** bounded | architectural
**Starting point:** <branch> @ <commit hash>      (filled in at step 9)
**Background:** why this plan exists

## 0. Summary              short, in plain words
## 1. Decisions            the user's, with dates
                           Success criteria: C1…Cn, each one checkable
## 2. Evidence             from A and B, with file:line; B's baseline;
                           which model produced each report
## 3. What already exists  the precedent and each difference: intended or not
## 4. The steps            per step: what, why, how it solves it,
                           files and boundaries, risk, how it is verified,
                           and how to go back if it changes stored data
                           or an interface
## 5. Flows
## 6. Implementation phases
## 7. Verification         table: criterion → test → who runs it → result
                           "could break" items → how each is checked
                           the live test list for B
## 8. Open points
## 9. Out of scope and accepted gaps
## 10. Implementation: what changed from the plan   (filled in as it happens)
```

On the bounded path the same sections apply, each as short as it can be; a section with
nothing to say gets one line saying so, not a deletion, so that audits can refer to
sections by number.

**Self-review before showing it (step 7):**
1. Gaps: "TBD", "TODO", empty sections or vague requirements.
2. Contradictions between sections.
3. Scope: does it fit in one plan, or does it need splitting?
4. Ambiguity: can any requirement be read two ways? Pick one and write it down.
5. Coverage: does every success criterion have a step that achieves it and a test that
   proves it? Does every "could break" item from the impact map have a check?
6. Order: can each phase be implemented and pass the gate on its own, without the ones
   after it?

Fix things as you go; no need to repeat the review. On the architectural path, C's
critique (step 7b) follows the self-review; it does not replace it.

## Audit

```
# Audit Report — <topic> (`systematicDebugging`)
**Result:** two or three lines
**Audited:** <starting commit>..<audited commit>; models used for B and C

## 1. What was implemented   files; B's tests: verified live / not verified;
                             comparison with the baseline
## 2. Errors found           E1…En
## 3. Change map             table: E#, severity, file, proposed change, why,
                             risk, user's decision
## 4. Deviations from the plan
## 5. Open items and gaps    including tests that could not run
## 6. Closing                per error: fixed and its no-regression check;
                             per success criterion: test and result;
                             B's and C's results in step 15
```

Each error carries: how it was found (reading, live test, test, temporary logs), the
evidence or the reproduction, the cause, the difference from the precedent, whether the
change introduced it or it already existed, the severity, the proposed change, and
**confirmed (reproduced)**, **confirmed** or **unconfirmed**.

Before writing down one of C's findings, check it yourself by reading the cited code. If
it does not hold up, mark it "unconfirmed" or drop it and say why.

After a clean audit (no confirmed finding, every test passed), sections 2 and 3 say so in
one line and section 6 is written from step 10's results.

## How to write

- In the user's language. The short, plain version first; the detail after.
- Evidence as `file:line`. Anything not verified is marked as a guess.
- Decisions are attributed to the user, with dates.
- Update **Status** at every step, so another session knows where to pick up.
- A reader who was not in the conversation must be able to follow the file: no labels
  coined during the work without saying what they mean, no "as discussed".
