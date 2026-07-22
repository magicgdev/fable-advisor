---
description: Run the architect orchestration doctrine on a task — decompose, delegate to implementer lanes (parallel worktrees when independent), integrate, verify. Explicit invocation, works on any session model.
argument-hint: <task, or list of independent tasks>
---

Invoke the `orchestration` skill and act as the architect for the following work. This explicit invocation overrides the skill's Fable-only gate — apply the full doctrine regardless of which model this session runs on (note in your first reply if the session model is not Fable, so the user knows the architect and the implementer lanes may be the same tier).

Apply the doctrine end to end:

1. Decompose the work; keep judgment (architecture, interfaces, specs, routing, verdicts) with you and delegate ALL implementation typing to `implementer` lanes (Sonnet default, `model: "opus"` for the subtle/high-stakes pieces).
2. Independent, substantial tasks fan out as parallel worktree lanes — but ONLY after the mandatory commit gate: show git status and the planned lanes, ask the user to commit to HEAD (or confirm it is current), and launch nothing until they confirm. Dependent chains, single-file surgery, and Godot `.tscn`/`.tres` work stay serial in-place.
3. After parallel lanes report, hand the `integrator` agent the merge back into the current branch, locally.
4. Judge evidence, not reports: read diffs, re-run or spot-check verification, and only then call it done. Consult `fable-advisor` at commitment boundaries.

In the tdgame project, additionally apply the skill's game-dev profile (scoped `run_tests.ps1` verification, save safety, headless-only, worktree-lag preflight).

Task:

$ARGUMENTS
