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
*all* files they surface, and writes prose-heavy summaries. `feature-dev-deepseek`
trims that while protecting output quality.

### What changed vs feature-dev

| Phase | feature-dev | feature-dev-deepseek |
|-------|-------------|------------------|
| 2 Exploration | 2–3 `code-explorer` agents, read all surfaced files | **1** `code-explorer-lite` driven by the **graphify** graph; read only ~5 essential files |
| 4 Architecture | 2–3 `code-architect` agents, multiple approaches | **1** `code-architect-lite`, **one** pragmatic-balance approach, confirm before building |
| 6 Review | 3 `code-reviewer` agents | **1** `code-reviewer-lite`, **`git diff`** default scope; ask before adding a 2nd on security/auth/payments/data-migration |
| 7 Summary | prose summary | **1** `summary-writer-lite` (deepseek-v4-pro); file paths, symbols, decisions over prose |
| Models | deepseek-v4-pro | **per-phase** model assignment (below) |

## Per-phase model assignment

| Phase | Work | Model |
|-------|------|-------|
| 1 Discovery | orchestrator | deepseek-v4-pro (main session) |
| 2 Exploration | `code-explorer-lite` | `deepseek-v4-pro` |
| 3 Clarifying Questions | orchestrator | deepseek-v4-pro (main session) |
| 4 Architecture | `code-architect-lite` | `deepseek-v4-pro` |
| 5 Implementation | orchestrator | deepseek-v4-pro (main session) |
| 6 Review | `code-reviewer-lite` | `deepseek-v4-pro` |
| 7 Summary | `summary-writer-lite` | `deepseek-v4-pro` |

Run the command on deepseek-v4-pro (`/model deepseek-v4-pro`). Subagents carry
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

## Command: `/feature-dev-deepseek`

```bash
/feature-dev-deepseek Add user authentication with OAuth
```

or just:

```bash
/feature-dev-deepseek
```

> **Fast-path**: this workflow is for multi-file or ambiguous features. For a
> trivial, well-defined change, skip it and make the edit directly.

## The 7-phase workflow

### Phase 1: Discovery — *deepseek-v4-pro*
Clarify the request (problem, behavior, constraints), then confirm a 2–4 line
understanding.

### Phase 2: Codebase Exploration — *1 agent, graphify-driven*
The orchestrator ensures the graphify graph exists (`graphify-out/graph.json`),
building it with `graphify .` if needed. A single `code-explorer-lite` agent
queries the graph (`GRAPH_REPORT.md`, `graph.json`) and returns a concise map
plus **≤ ~5 essential files**. Only those files are read. Consider `/compact`
afterward.

### Phase 3: Clarifying Questions — *deepseek-v4-pro*
One consolidated, prioritized list of **blocking** questions. Waits for answers
before designing. (Do not skip.)

### Phase 4: Architecture Design — *1 agent, deepseek-v4-pro*
A single `code-architect-lite` agent designs **one** pragmatic-balance approach
(reusing the graph + explorer findings) with a concrete implementation map.
You're asked to **confirm before implementation**.

### Phase 5: Implementation — *deepseek-v4-pro*
Starts only after approval. Implements the chosen architecture, follows codebase
conventions, tracks todos.

### Phase 6: Quality Review — *1 agent, deepseek-v4-pro*
A single `code-reviewer-lite` agent reviews the unstaged `git diff`
(confidence ≥ 80 only). If the change touches **security / auth / payments /
data migration**, the workflow **pauses and asks** whether to add a second,
focused reviewer — it never auto-spawns extra reviewers. You decide what to do
with findings (fix now / later / proceed).

### Phase 7: Summary — *1 agent, deepseek-v4-pro*
A `summary-writer-lite` agent writes a terse wrap-up: what was built
(symbol + `file:line`), key decisions, files changed, optional next steps —
**no prose recap**.

## Agents

- **`code-explorer-lite`** (deepseek-v4-pro) — graph-first exploration; fails fast if
  graphify's graph is absent; returns ≤ ~5 essential files.
- **`code-architect-lite`** (deepseek-v4-pro) — one decisive pragmatic-balance blueprint.
- **`code-reviewer-lite`** (deepseek-v4-pro) — single reviewer, `git diff` default,
  confidence-filtered.
- **`summary-writer-lite`** (deepseek-v4-pro) — terse path/symbol/decision summary.

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

1.0.1
