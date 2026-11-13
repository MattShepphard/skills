# Models in Antigravity

Which Gemini model belongs to each tier in `SKILL.md`, and how to pick it. This is the
only file that names models: when the lineup changes, update this file and the date
below, not `SKILL.md`.

**Last checked:** 2026-11-13, against the `swarm` tool's `models` list.

## Contents

- Tiers and models
- Choosing the model when you launch an agent
- Agent types
- Continuing, running in parallel and isolating agents
- When quota runs out
- If the session is not on a Gemini model

## Tiers and models

| Tier | Gemini model | `invoke_subagent` alias | Exact IDs (`swarm` only) |
|---|---|---|---|
| **Deep** | Gemini 4 Argon | `pro` | `gemini-4-argon-high` / `medium` / `low` |
| **Standard** | Gemini 4 Hydrogen | `flash` | `gemini-4-hydrogen-high` / `medium` / `low` |
| **Standard, top effort** | Gemini 4 Hydrogen XHigh | — | `gemini-4-hydrogen-xhigh` |
| **Fast** | Gemini 3.8 Flash | `flash_lite` | `gemini-3.8-flash-high` / `medium` / `low` |

- **Hydrogen XHigh is Standard, not Deep.** It is the same model at its highest reasoning
  effort. It is the first fallback for C when Deep is not available (see "When quota runs
  out"), and it is never what you pick first for a job the table in `SKILL.md` marks Deep.
- **Gemini 4 Carbon** may be offered as the session's model in the model selector, but it
  was not in the list of models that subagents can run on when this file was last
  checked. Do not plan a C on Carbon; if Carbon shows up in that list, it counts as Deep
  and is the preferred C when you run on Argon.
- The aliases point to tiers. Which exact model sits behind each one is decided by the
  harness, so the table above describes the mapping found on the date at the top. If
  an agent's report shows a different model, record the one it actually ran on.

## Choosing the model when you launch an agent

- **Use `invoke_subagent` with its `Model` parameter:** `pro` for Deep, `flash` for
  Standard and `flash_lite` for Fast. `inherit` gives the agent your own model; use it
  only when the job really needs your tier.
- **Exact IDs are only for `swarm`.** A swarm is heavy on quota and in preview. Do not
  start one just to choose a model or an effort level. Use it only when the user asks for
  a swarm.
- **A different model for C.** With `invoke_subagent` you choose a tier, not a model.
  - If you run on a model outside Argon (Carbon, or a model from another provider),
    `pro` already gives C a different model.
  - If you run on Argon, `pro` gives C the same model. That is acceptable: C is a fresh
    agent that has not seen your reasoning, and the tier matters more than the variety
    (see "Rules for choosing" in `SKILL.md`). Say in the audit that C ran on the same
    model as you.

## Agent types

| Agent | Type | Why |
|---|---|---|
| **A** | `research` | Read-only, with no network. The tools enforce A's "never edit". |
| **C** | `research` | The same, for "never edit or fix". |
| **B** (terminal: gate, tests, CLI, API) | `self` | Needs a shell. The brief carries B's limits, because the type does not enforce them. |
| **B** (web UI) | `browser` | The only agent that drives the browser. It has no shell. |
| **B** (temporary logs) | `self` with `Workspace: "branch"` | Edits only in its own clone. |

What this changes in the briefs:

- **A and C cannot run `git`.** Before launching them, run the commands yourself and
  save the output in the conversation's `scratch/` folder: `git log` of the area for A's
  step 2, and for C the diff from the starting commit plus the uncommitted changes. Pass
  the paths in the brief.
- **B is split in two when there is a UI.** B-shell starts the app (as a long-running
  process) and runs the gate and the tests. B-browser gets the URL and the live UI steps
  and returns what it saw as text. Launch them in the same message when they do not
  depend on each other. If B-browser needs the app that B-shell starts, start the app
  yourself first, then launch both. Each one gets B's limits in its brief.
- `preview_start` from other harnesses does not exist here; the `browser` agent replaces
  it.

## Continuing, running in parallel and isolating agents

- **Continuing an agent** (A at step 5, B reproducing a finding at step 11, an agent with
  a gap in its report): use `send_message` to its conversation ID. Do not launch a new
  one unless the old one is gone.
- **In parallel:** launch both agents in the same response. Antigravity notifies you
  when each one finishes. Do not poll and do not set timers to check on them.
- **Temporary logs:** launch that B with `Workspace: "branch"`. It gets an isolated clone,
  never your working folder. When it is done, ending it with `manage_subagents` (`kill`)
  deletes its workspace. Check that you are killing the logging agent, not another one.
  Its changes are never merged into your working copy.

## When quota runs out

The `swarm` tool's `models` action lists each model with its remaining quota and when it
resets. Checking it does not start a swarm. Look at it before step 7b or 10 if earlier
steps used a lot of Deep.

Fallback order for C (steps 7b, 10 and 15):

1. Deep: `pro`.
2. If Deep is out of quota or failing: Hydrogen XHigh (Standard, top effort), only through
   a swarm the user agrees to, or `flash` if the user prefers not to start one. Tell the
   user before launching, and write in the audit: "C ran below Deep (model, reason)".
3. If the user does not accept that: wait for the quota to reset, with the plan's Status
   saying "waiting on Deep quota for C".

Never Fast for C. Never move C down a tier without telling the user.

For the other agents, step down one tier and note it. A Fast agent's report on something
that needs judging is a lead you confirm yourself.

## If the session is not on a Gemini model

The cycle does not change. Your tier is your model's tier. If it is a Deep-tier model,
`pro` already gives C a different model. If it is not Deep, say so at step 1, as
`SKILL.md` says.
