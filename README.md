# skills

Agent skills for Claude Code.

## measure-twice

*Measure twice, cut once.* A full cycle to design, build, audit and close a change in any
project. Agents do the reading and testing; you make the decisions.

- **Agents read and test.** A maps the code and its precedents, B runs the app and the
  tests, and C audits the result against the plan. None of them edits your working code.
- **The model talks, writes and implements.** It brainstorms with you, writes the plan and
  the audit, and implements only what you approved.
- **You decide at three gates:** which approach, whether the plan is approved, and what to
  do with each finding.

### Why

Each step targets a kind of miss that shows up in real projects: a flow nobody mapped, a
change that touched code it should not have, an error that only appeared when someone
used the app.

### The cycle

| # | Who | What |
|---|---|---|
| 1 | Model and you | Framing: what it is and why |
| 2 | A and B | Code map and current behavior, in parallel |
| 3 | Model and you | Brainstorming: size, questions, 2-3 approaches |
| 4 | **You** | **Pick an approach** |
| 5 | A | Impact map of the chosen approach |
| 6 | Model and you | Design, section by section |
| 7 | Model | Written plan, self-reviewed |
| 8 | **You** | **Approve the plan** |
| 9 | Model | Implementation |
| 10 | B and C | Live tests and audit, in parallel |
| 11 | Model | Written audit, every finding confirmed |
| 12 | Model and you | Brainstorming on the findings |
| 13 | **You** | **Decide each change** |
| 14 | Model | Fixes, one at a time |
| 15 | B, C and model | Closing pass with full systematic debugging |

Smaller requests take a lighter path: a *spike* answers a feasibility question in the chat,
and a *bounded* change gets a short plan.

### What you get

Two documents per change, in the project's plans folder and in your language:

- `YYYY-MM-DD-plan-<topic>.md`: the steps, why each one is needed, how it solves the
  problem, its risks and how it is verified.
- `YYYY-MM-DD-audit-<topic>-systematicDebugging.md`: the errors found, the change map,
  your decisions and the result of the closing pass.

If the project already has its own naming, the skill follows it.

### Install

```bash
npx skills add MattShepphard/skills --skill measure-twice
```

Or copy `skills/measure-twice/` into `~/.claude/skills/` for all your projects, or into a
project's `.claude/skills/` for just that one.

### Use

Ask for it in your own words: "brainstorm this feature", "plan and audit this change",
"audit what was implemented". Requests in Spanish such as "utiliza brainstorm" work too.

### Requirements

- Claude Code with subagents.
- Optional: the built-in browser, for live tests of web apps, and git worktrees, for
  temporary diagnostic logs.

### Project rules it reads

Put these in the project's `CLAUDE.md` or memory and the skill will use them:

- where plans and audits live, and how they are named;
- what must not break;
- actions the test agent must never take without asking, such as sending real emails,
  writing to a CRM, making payments or touching production data.

### Credits

The method adapts ideas from the `brainstorming` and `systematic-debugging` skills in
[obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent (MIT License).
This skill is a rewrite in its own words and does not depend on those skills.

### License

[MIT](LICENSE)
