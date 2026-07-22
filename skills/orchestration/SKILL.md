---
name: orchestration
description: Routing doctrine for the architect-as-orchestrator pattern — how a session running the smartest model delegates implementation to cheaper Claude lanes to minimize cost. USE WHEN delegating implementation work, choosing implementer model tier (sonnet vs opus), writing a spec for a subagent, fanning independent tasks out to parallel worktree lanes and merging them back, deciding whether to consult fable-advisor, managing session cost or token spend, or running any multi-task build where the session is the architect.
---

# Orchestration — the architect's routing doctrine

The session is the architect: it owns requirements, architecture, decomposition, specs, routing, and verification. It should almost never type implementation code. Every implementation task gets routed to the cheapest lane that is adequate for it — escalation is deliberate, per task, never a fixed binding.

## Cost discipline — the prime directive

The session model is the most expensive lane in the system, on both input and output tokens. The whole economic case for this pattern is keeping its token volume low: spend Fable on judgment, spend Sonnet on volume. Three rules follow.

**Emit judgment, not volume.** The architect's output is decomposition, specs, routing decisions, verdicts on diffs, and short reports. It does not type implementation code, test bodies, boilerplate, or config files. A code block longer than an interface signature or a few illustrative lines is a spec that hasn't been delegated yet — stop and delegate it. Fixing a lane's bug by hand is the same failure in disguise: send a corrected spec back to the cheap lane instead.

**Keep the context lean.** Everything in the architect's context is re-read at architect prices on every turn. Delegate broad exploration, codebase searches, and log-grepping to a cheap read-only agent and keep only the conclusions; read files yourself only when the decision genuinely depends on the exact code. Don't paste long files, full diffs, or verbose command output into the conversation when a path reference or an excerpt will do.

**Reason once, then hand off.** Do the hard thinking — the architecture, the interface design, the debugging hypothesis — in one pass, capture it in the spec, and let the cheap lane carry it from there. Re-deriving decisions across turns burns the premium twice.

What stays with the architect regardless of cost: decomposition, interface design, hypothesis selection when debugging, spec writing, lane routing, and judging verification evidence. Those tokens are what the premium is for — everything else is a candidate for delegation.

## The lanes

| Lane | Producer | Invoke | Route here when |
|---|---|---|---|
| Routine | Sonnet | `implementer` agent | The spec fully determines the outcome: boilerplate, wiring, mechanical edits, straightforward features, test bodies. **Default lane.** |
| Hard | Opus | `implementer` agent with `model: "opus"` | Subtle or high-stakes: concurrency, security-sensitive paths, hard debugging, wide refactors, gnarly state machines. Escalate per task, never by default. |
| Integration | Sonnet | `integrator` agent | After a parallel worktree fan-out: commits each lane, merges branches back into the current branch locally, re-verifies the combination. |
| Judgment | Fable 5 | `fable-advisor` agent | Not an implementation lane. See "Commitment boundaries" below. |

Deciding rule: how much does the outcome depend on judgment the spec can't capture? Little → the default Sonnet lane; you will verify anyway. A lot, and mistakes are costly → the Opus lane, or keep that piece with the architect. For the highest-stakes specs, run Sonnet and Opus on the same spec in separate worktrees and pick the stronger diff — one extra lane's cost buys a genuinely independent second implementation.

## The spec contract

Implementers share none of your conversation context. Every delegation prompt carries all five parts:

1. **Objective** — what to build or change, one paragraph
2. **Files** — exact paths to create or modify
3. **Interfaces** — signatures, types, or API shapes the code must match
4. **Constraints** — project conventions, things not to touch — including the load-bearing lessons from project memory for the task's domain (the implementer cannot read your memory; paste the rule, not a pointer)
5. **Verification** — the command(s) that prove it works, scoped to the change (plus any bootstrap a fresh worktree needs first)

A spec you can't finish writing is a signal the decision isn't made yet — that's architect work, not a reason to hand the ambiguity to a cheaper model.

## Parallel fan-out and merge

Serial in-place delegation is the default — one `implementer`, working tree, no ceremony. Fan out only when ALL of these hold:

- **2–4 tasks, genuinely independent**: no shared files, no ordering dependency, and (for game work) disjoint test-scope domains.
- **Each task is substantial** — several minutes of implementation. Fanning out three one-line edits costs more coordination than it saves.
- **The current branch HEAD actually contains the state the lanes must build on.** Worktrees branch from commits, not from your dirty working tree — stale HEAD means every lane builds against code that no longer exists.

**Commit gate — mandatory, no exceptions.** Before launching ANY worktree fan-out, stop and prompt the user: show `git status` (what's uncommitted) and the planned lane list, and ask them to commit the current state to HEAD — or confirm HEAD is already current. Launch the lanes only after the user explicitly confirms. Never commit on their behalf to unblock a fan-out, and never skip the gate because the tree "looks clean" — a clean tree can still be a stale HEAD (see the tdgame lag check below).

The flow:

1. Launch each spec as an `implementer` agent with `isolation: "worktree"`, all in a single message so they run concurrently. Route each to sonnet or opus per the lane table. Tell each lane to commit its work on its branch with a message you supply.
2. When all lanes report, read each report's evidence (not just the status line). A lane that failed gets a corrected spec re-run — don't merge known-bad work.
3. Hand the `integrator` agent the main checkout path, target branch, the lane list (worktree path, branch, summary, commit message), the merge order, and the post-merge verification command. It merges locally into the current branch — never a push, never a PR.
4. Judge the integrator's report: post-merge verification output is the only evidence that the *combination* works. Substantive conflicts come back to you; re-spec the colliding piece serially rather than arbitrating the textual conflict yourself.

What must stay serial regardless: chains with ordering dependencies, single-file surgery, and — in Godot projects — any two tasks touching the same `.tscn`/`.tres`. Scene and resource files are machine-written; parallel edits to one are a guaranteed substantive conflict.

## Game-dev profile (Godot / tdgame)

When orchestrating in the tdgame project (`tdgame/test-mcp`), the project's own knowledge base outranks generic doctrine — consult it while writing specs, and bake the relevant rules INTO the spec's constraints:

- **Route context by domain first.** The memory index (`MEMORY.md` in the project's memory dir) has a routing table: task domain → memory group to read + test scope tag + design doc. One lookup per task; read only the matching row's files. The design docs are git-untracked and exist only at the main checkout's `docs/design/` — worktree lanes cannot see them, so excerpt what the spec needs.
- **Verification is scoped, never full.** Spec verification = `./run_tests.ps1 -Changed` or `-Scope "<tags>"` from `test-mcp/` (seconds, parallel). The FULL suite is for /sync and releases only. Display-only edits scope down by hand. A fresh worktree needs `godot --headless --path . --import` once before any scene-based run, and a new `class_name` or test scene needs `--import` to register.
- **Headless only.** Never launch windowed Godot from any lane — user opens the editor themselves. Ad-hoc headless runs always get an explicit timeout (no watchdog otherwise), and every background process gets tracked to completion or killed — no spawn-and-forget.
- **Save safety is non-negotiable.** No lane may write the real `user://save_data.json` — tests go through the runner / `TestUtils` patterns (see the tdgame-testing skill's save-safe pattern); put that constraint in every spec that touches meta-progression, saves, or persistence.
- **Worktree lag check feeds the commit gate.** The live checkout runs ahead of committed history and auto-commits to `feature1`; a worktree branched from a stale commit builds against code that no longer exists. Before the mandatory commit-gate prompt, compare the current branch HEAD against the live state (check `GAME_VERSION`) and include the finding in the prompt — current, or lagging and by how much. If the user can't or won't bring HEAD current, work serially at the main checkout instead of fanning out.
- **Serial-only surfaces:** anything touching the same `.tscn`/`.tres`, `project.godot`, autoload registrations, design docs, or the player-facing changelog (that's /changelog's job). Balance passes are serial too — EV/gold discipline (bands, `gold = K·EV^P` bake) is a single coherent judgment, not parallelizable typing.
- **After the build, the project workflow still applies:** review the integrated diff per the review-discipline memory, and leave knowledge capture to /sync — lanes and integrator never write memory, design docs, or skills.

## Commitment boundaries

Consult `fable-advisor` (read-only, verdict in under 300 words) at the moments that decide whether the next hour is wasted:

- Before committing to an architecture, data migration, API shape, or refactor strategy
- Whenever the same problem has resisted two distinct attempts
- Once before declaring a multi-step deliverable done

Pass it the decision, the constraints, and the options considered. Act on the verdict or surface the disagreement — never silently ignore it. (If the session itself already runs on Fable, the advisor still earns its keep as a context-clean skeptic reading the actual code.)

## Verification

Reports are claims, not evidence. Before accepting any lane's work: read the diff, and re-run the verification command (or spot-check its quoted output against the working tree). "Should work", "tests should pass", or a report with no command output means the task is not done. After a parallel merge, only the integrator's post-merge run counts — per-lane green proves nothing about the combination. A lane that reports a spec gap gets a corrected spec, not a "use your judgment".
