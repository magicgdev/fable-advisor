# Fable Advisor

**The smartest model runs the show. Cheaper models do the typing.**

Claude Code lets every subagent run on a different model — and lets the session itself run on a different model than its subagents. This plugin exploits that with the **architect pattern**: your session runs on **Fable 5**, Anthropic's most capable model, acting as a full-time architect. It owns requirements, decomposition, specs, and verification — and routes every implementation task to the cheapest adequate lane:

| Lane | Producer | Invocation | Route here when |
|---|---|---|---|
| Routine | **Sonnet** | `implementer` agent (default) | The spec fully determines the outcome — boilerplate, wiring, mechanical edits, test bodies |
| Hard | **Opus** | `implementer` agent with `model: "opus"` | Subtle or high-stakes: concurrency, security-sensitive paths, hard debugging, wide refactors |
| Integration | Sonnet | `integrator` agent | Merging parallel worktree lanes back into the current branch, locally |
| Judgment | Fable 5 | `fable-advisor` agent | Commitment boundaries — see below |

Tokens route by volume: the expensive model emits the fewest tokens (judgment and specs), cheap lanes emit the most (code). Implementation mechanics are ~90% of a session's tokens and Sonnet handles them at near-parity when the spec is complete — so this runs far cheaper than Fable-for-everything, and every diff still gets reviewed by a stronger model than the one that wrote it.

**Parallel fan-out.** Independent, substantial tasks launch as parallel `implementer` lanes in isolated git worktrees — but only after a mandatory commit gate: the architect shows you what's uncommitted and the planned lanes, asks you to commit to HEAD (worktrees branch from commits, not the dirty tree), and starts nothing until you confirm. Each lane commits on its own branch, and the `integrator` agent merges the branches back into your current branch locally — resolving only mechanical conflicts, re-running verification on the combined result, and never pushing. Anything with shared files, ordering dependencies, or machine-written formats (Godot `.tscn`/`.tres`) stays serial.

The plugin ships the **orchestration skill** — the routing doctrine that teaches the session when to use each lane, the cost discipline that keeps the expensive model's own token volume minimal (emit judgment not volume, keep context lean, reason once then hand off), the five-part spec contract that makes context-free delegation safe, the parallel fan-out/merge flow, a game-dev profile for Godot projects, and the verification rules that keep cheap lanes honest.

## Install

```
claude plugin marketplace add DannyMac180/fable-advisor
claude plugin install fable-advisor@fable-advisor
```

Updating an existing installation to the latest release:

```
claude plugin marketplace update fable-advisor
claude plugin update fable-advisor@fable-advisor
```

Then start your session as the architect:

```
/model fable
```

**Lite mode — one file, 30 seconds.** Don't want the full pattern? Copy [`agents/fable-advisor.md`](agents/fable-advisor.md) into `~/.claude/agents/` and keep your session on Sonnet. You get advisor consults at commitment boundaries without the orchestration layer (see "Advisor-only mode" below).

## Requirements

- **Claude Code ≥ 2.1.170** with a subscription that includes Fable 5 (Pro, Max, Team, or Enterprise — all current consumer plans qualify).
- **No Fable access** (e.g. API-key billing)? Use `/model opus` for the session and change `model: fable` → `model: opus` in the advisor file. Same pattern, model tiers shift down one.
- No external CLIs — all lanes are plain Claude Code subagents.
- Heads-up: if a pinned Claude model isn't available on your account, Claude Code silently falls back to your session model — the pattern degrades quietly rather than erroring. If results feel unremarkable, check your plan.

Model resolution order in Claude Code: `CLAUDE_CODE_SUBAGENT_MODEL` env var → per-invocation `model` parameter → agent frontmatter → session model.

## Use it

With the session on Fable, just ask for work — the orchestration skill routes it:

```
Add rate limiting to our public API. Design it, delegate the
implementation, and verify the evidence before you call it done.
```

The architect writes the spec, picks the lane (rate limiting touches concurrency — a good case for the Opus tier, or for racing Sonnet and Opus on the same spec and picking the stronger diff), reads the diff and verification evidence when the report comes back, and only then reports done.

For a multi-task build, independent specs fan out in parallel:

```
Rework the fire tower's DOT stacking, add the new wind-tower tornado
pull, and fix the save-slot naming bug. These don't touch each other —
run them in parallel and merge when they're all green.
```

Three worktree lanes run concurrently; the integrator lands all three on your branch and re-runs the scoped tests on the combined result.

To make the doctrine always-on, add one line to your project's `CLAUDE.md`:

```
You are the architect running the most expensive model — minimize your
own token volume. Delegate all implementation through the orchestration
skill's routing table (never type code yourself), delegate broad codebase
exploration to cheap read-only agents, and verify evidence before
accepting any lane's report.
```

## Commitment boundaries

Even the architect gets a second opinion. The `fable-advisor` agent is a read-only skeptic — consulted before architecture decisions, migrations, API designs, and whenever a problem has resisted two attempts. It reads your actual code and returns a verdict in under 300 words. It never implements. Running it from a Fable session still pays: it sees the code fresh, without your conversation's accumulated assumptions.

## Advisor-only mode (the original pattern)

The inverse arrangement, for when you'd rather keep the session cheap: run the session on Sonnet and consult `fable-advisor` only at commitment boundaries.

```
Migrate our checkout sessions from Postgres to Redis — plan it,
consult your advisor before committing, then implement.
```

A typical consult costs cents. To make it automatic, add to your project's `CLAUDE.md`:

```
Before committing to any architecture decision, migration, or refactor
touching 3+ files, consult the fable-advisor agent and act on its verdict.
```

## FAQ

**Is this Anthropic's "advisor tool"?** No — that's a server-side API feature. These are plain Claude Code subagents plus a skill: readable, editable, no beta flags.

**Does this work on claude.ai?** No — subagent model routing is Claude Code only (CLI, desktop, VS Code, web).

**Why not just run everything on Fable?** You can. It's excellent. It's also the most expensive lane per token, and most of a session's tokens are implementation mechanics that the cheap lanes handle at near-parity. Spend the premium where judgment lives.

**Upgrading from v3?** v4 removes the `grok-implementer` and `codex-implementer` CLI lanes and returns to the all-Claude `implementer` agent (Sonnet default, `model: "opus"` escalation) — no external CLIs to install or authenticate. It adds the `integrator` agent and the parallel worktree fan-out/merge flow. The `fable-advisor` agent and advisor-only mode work exactly as before.

**What happened to cross-vendor review?** v3's Grok/Codex lanes traded two CLI dependencies and per-request API billing for vendor diversity. v4 trades it back: within one subscription, Sonnet lanes are effectively covered by your plan, verification is enforced by the doctrine (evidence, not reports), and a Sonnet-vs-Opus race on the same spec still buys an independent second implementation when stakes are high.

## Go deeper

I write [**Attention Heads**](https://attentionheads.substack.com/?utm_source=github&utm_medium=readme&utm_campaign=fable-advisor) — deep, evidence-backed writing on AI, cognition, and agentic engineering. The **Agentic Engineering Field Notes** series is where I publish practical advice on the craft of using AI. [Subscribe](https://attentionheads.substack.com/subscribe?utm_source=github&utm_medium=readme&utm_campaign=fable-advisor) to get new posts to your inbox.

## License

MIT
