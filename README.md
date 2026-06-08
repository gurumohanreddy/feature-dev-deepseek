# Feature Development Lite Plugin

A **cost-optimized** fork of the feature-dev plugin. It keeps the structured,
high-quality 7-phase feature workflow but cuts token spend: graph-driven
exploration, a single pragmatic architecture, a single git-diff reviewer,
per-phase model assignment, and terse summaries.

Primarily intended for **GitHub Copilot CLI** (loadable via `/plugin`).

> This is a streamlined fork of **feature-dev** by Sid Bidasaria. The 7-phase
> structure and philosophy are preserved; the changes below are purely about
> managing cost.

## Why "lite"?

Under the new Copilot pricing, the original workflow burns a lot of tokens: it
spawns 2–3 explorer agents, 2–3 architect agents, and 3 reviewer agents, reads
*all* files they surface, and writes prose-heavy summaries. `feature-dev-lite`
trims that while protecting output quality.

### What changed vs feature-dev

| Phase | feature-dev | feature-dev-lite |
|-------|-------------|------------------|
| 2 Exploration | 2–3 `code-explorer` agents, read all surfaced files | **1** `code-explorer-lite` driven by the **graphify** graph; read only ~5 essential files |
| 4 Architecture | 2–3 `code-architect` agents, multiple approaches | **1** `code-architect-lite`, **one** pragmatic-balance approach, confirm before building |
| 6 Review | 3 `code-reviewer` agents | **1** `code-reviewer-lite`, **`git diff`** default scope; ask before adding a 2nd on security/auth/payments/data-migration |
| 7 Summary | prose summary | **1** `summary-writer-lite` (Haiku); file paths, symbols, decisions over prose |
| Models | sonnet everywhere | **per-phase** model assignment (below) |

## Per-phase model assignment

| Phase | Work | Model |
|-------|------|-------|
| 1 Discovery | orchestrator | Sonnet 4.6 (main session) |
| 2 Exploration | `code-explorer-lite` | `claude-sonnet-4.6` |
| 3 Clarifying Questions | orchestrator | Sonnet 4.6 (main session) |
| 4 Architecture | `code-architect-lite` | `claude-opus-4.8` |
| 5 Implementation | orchestrator | Sonnet 4.6 (main session) |
| 6 Review | `code-reviewer-lite` | `claude-opus-4.8` |
| 7 Summary | `summary-writer-lite` | `claude-haiku-4.5` |

Run the command on Sonnet 4.6 (`/model claude-sonnet-4.6`). Subagents carry
their own model in frontmatter — don't override them.

## Requirements

- **GitHub Copilot CLI** (or another agent host that loads this plugin format).
- **graphify** installed and on PATH — exploration depends on it:
  ```bash
  uv tool install graphifyy        # or: pipx install graphifyy
  graphify install --platform copilot
  ```
  See https://github.com/safishamsi/graphify. The workflow **assumes graphify is
  installed and fails fast** if the graph is missing (it does not fall back to a
  full manual exploration).
- Git repository (Phase 6 reviews `git diff`).
- An existing codebase to learn from.

## Command: `/feature-dev-lite`

```bash
/feature-dev-lite Add user authentication with OAuth
```

or just:

```bash
/feature-dev-lite
```

> **Fast-path**: this workflow is for multi-file or ambiguous features. For a
> trivial, well-defined change, skip it and make the edit directly.

## The 7-phase workflow

### Phase 1: Discovery — *Sonnet 4.6*
Clarify the request (problem, behavior, constraints), then confirm a 2–4 line
understanding.

### Phase 2: Codebase Exploration — *1 agent, graphify-driven*
The orchestrator ensures the graphify graph exists (`graphify-out/graph.json`),
building it with `graphify .` if needed. A single `code-explorer-lite` agent
queries the graph (`GRAPH_REPORT.md`, `graph.json`) and returns a concise map
plus **≤ ~5 essential files**. Only those files are read. Consider `/compact`
afterward.

### Phase 3: Clarifying Questions — *Sonnet 4.6*
One consolidated, prioritized list of **blocking** questions. Waits for answers
before designing. (Do not skip.)

### Phase 4: Architecture Design — *1 agent, Opus 4.8*
A single `code-architect-lite` agent designs **one** pragmatic-balance approach
(reusing the graph + explorer findings) with a concrete implementation map.
You're asked to **confirm before implementation**.

### Phase 5: Implementation — *Sonnet 4.6*
Starts only after approval. Implements the chosen architecture, follows codebase
conventions, tracks todos.

### Phase 6: Quality Review — *1 agent, Opus 4.8*
A single `code-reviewer-lite` agent reviews the unstaged `git diff`
(confidence ≥ 80 only). If the change touches **security / auth / payments /
data migration**, the workflow **pauses and asks** whether to add a second,
focused reviewer — it never auto-spawns extra reviewers. You decide what to do
with findings (fix now / later / proceed).

### Phase 7: Summary — *1 agent, Haiku 4.5*
A `summary-writer-lite` agent writes a terse wrap-up: what was built
(symbol + `file:line`), key decisions, files changed, optional next steps —
**no prose recap**.

## Agents

- **`code-explorer-lite`** (Sonnet 4.6) — graph-first exploration; fails fast if
  graphify's graph is absent; returns ≤ ~5 essential files.
- **`code-architect-lite`** (Opus 4.8) — one decisive pragmatic-balance blueprint.
- **`code-reviewer-lite`** (Opus 4.8) — single reviewer, `git diff` default,
  confidence-filtered.
- **`summary-writer-lite`** (Haiku 4.5) — terse path/symbol/decision summary.

## Additional cost-saving behaviors

- Reuse the **one** graphify graph across phases 2/4/6 instead of re-exploring.
- Read only the top ~5 essential files, not everything surfaced.
- Cap clarifying questions to genuinely blocking ones.
- Trimmed agent tool lists to keep context small.
- Terse agent prompts and outputs (`file:line`, symbols, bullets).
- `/compact` suggested after exploration; fast-path for trivial changes.

## Attribution

Based on the **feature-dev** plugin by Sid Bidasaria (sbidasaria@anthropic.com).
The "lite" adjustments optimize for cost on GitHub Copilot CLI.

## Version

1.0.0
