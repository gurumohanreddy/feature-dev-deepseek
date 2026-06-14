---
description: Cost-optimized guided feature development (graphify exploration, single pragmatic architecture, single git-diff reviewer, per-phase models, terse summary)
argument-hint: Optional feature description
---

# Feature Development (Lite)

You are helping a developer implement a new feature with a **cost-optimized**
workflow. Keep the rigor of feature-dev where it matters, but minimize token
spend: fewer agents, graph-driven exploration, fewer file reads, and terse I/O.

## Core principles

- **Be frugal with tokens**: don't bulk-read files, don't restate context, don't
  spawn more agents than specified below. Prefer `file:line`, symbols, and short
  bullets over prose.
- **Ask clarifying questions** (once, consolidated) after exploration and before
  architecture. Wait for answers before designing/implementing.
- **Understand before acting**, but lean on the graphify graph rather than wide
  manual exploration.
- **Track progress** with the todo/TodoWrite tooling.

## Models per phase

| Phase | Runs on |
|-------|---------|
| 1, 3, 5 (orchestrator) | this main session — **deepseek-v4-pro** |
| 2 `code-explorer-lite` | **deepseek-v4-pro** (subagent) |
| 4 `code-architect-lite` | **deepseek-v4-pro** (subagent) |
| 6 `code-reviewer-lite` | **deepseek-v4-pro** (subagent) |
| 7 `summary-writer-lite` | **deepseek-v4-pro** (subagent) |

Run this command on **deepseek-v4-pro** (`/model deepseek-v4-pro`). The subagents
carry their own model in their frontmatter — do not override it.

> **Fast-path**: this workflow is for multi-file or ambiguous features. For a
> trivial/well-defined change, say so and just make the change directly instead
> of running all 7 phases.

---

## Phase 1: Discovery — *deepseek-v4-pro (this session)*

**Goal**: Understand what needs to be built.

Initial request: $ARGUMENTS

**Actions**:
1. Create a todo list for the phases.
2. If the feature is unclear, ask the user: what problem, desired behavior, and
   constraints. Keep it brief.
3. Summarize your understanding in 2–4 lines and confirm.

---

## Phase 2: Codebase Exploration — *single agent, graphify-driven*

**Goal**: Understand the relevant code with minimal token spend.

**Actions**:
1. **Ensure the graphify graph exists.** Check for `graphify-out/graph.json`.
   If missing, build it once from this session: run `graphify .` (graphify is
   assumed installed via `graphify install --platform copilot`). If graphify is
   **not installed**, STOP and tell the user to install it
   (`graphify install --platform copilot`) and re-run — do **not** fall back to
   a full manual exploration.
2. Launch **exactly one** `code-explorer-lite` agent. It queries the graph
   (`GRAPH_REPORT.md`, `graph.json`) and returns a concise map plus a ranked list
   of **at most ~5 essential files**.
3. Read **only** those essential files (cap ~5) — not "all files". Rely on the
   graph for the rest.
4. Present a short findings summary (bullets, `file:line`).

> Tip: after this phase, consider `/compact` to drop bulky exploration context
> before implementation.

---

## Phase 3: Clarifying Questions — *deepseek-v4-pro (this session)*

**Goal**: Resolve all blocking ambiguities before designing. **DO NOT SKIP.**

**Actions**:
1. Review findings + original request.
2. Identify only the **genuinely blocking** underspecified aspects: edge cases,
   error handling, integration points, scope boundaries, compatibility,
   performance.
3. Present **one consolidated, prioritized list** of questions (keep it tight —
   don't pad with low-value questions).
4. **Wait for answers** before Phase 4. If the user says "whatever you think is
   best", give your recommendation and get explicit confirmation.

---

## Phase 4: Architecture Design — *single agent, deepseek-v4-pro*

**Goal**: One solid, decisive approach — not a menu.

**Actions**:
1. Launch **exactly one** `code-architect-lite` agent. It designs a single
   **pragmatic-balance** approach, reusing the graphify graph and the explorer's
   findings.
2. Present the approach concisely: summary, key trade-off, components, and the
   implementation map (`file:line`).
3. **Ask the user to confirm the approach before implementing.** Do not start
   Phase 5 without explicit approval.

---

## Phase 5: Implementation — *deepseek-v4-pro (this session)*

**Goal**: Build the feature.

**DO NOT START WITHOUT USER APPROVAL.**

**Actions**:
1. Wait for explicit approval of the Phase 4 approach.
2. Read any remaining essential files you have not already read (stay frugal).
3. Implement following the chosen architecture and codebase conventions.
4. Write clean code; comment only where it adds clarity.
5. Update todos as you progress.

---

## Phase 6: Quality Review — *single agent, deepseek-v4-pro*

**Goal**: Catch high-impact issues with one reviewer by default.

**Actions**:
1. Launch **exactly one** `code-reviewer-lite` agent with the **`git diff`**
   (unstaged changes) as the default scope.
2. **Escalation (ask, don't auto-spawn):** If the change appears to touch
   **security, auth, payments, or data migration**, do **not** automatically add
   reviewers. Instead pause and ask the user:
   *"This touches <area>. Add a second, security/correctness-focused reviewer?"*
   Only launch a second `code-reviewer-lite` (focused on that concern) if the
   user agrees.
3. Consolidate findings (Critical vs Important, confidence ≥ 80 only) and present
   them concisely.
4. Ask the user what to do (fix now, fix later, proceed as-is) and act on it.

---

## Phase 7: Summary — *single agent, deepseek-v4-pro*

**Goal**: Document what was accomplished — **concisely**.

**Actions**:
1. Mark all todos complete.
2. Launch the `summary-writer-lite` agent (deepseek-v4-pro) to produce the wrap-up.
   It must **prefer file paths, symbols, and decisions over prose**: what was
   built (symbol + `file:line`), key decisions (one line each), files changed,
   and optional next steps. No process recap, no marketing language.

---
