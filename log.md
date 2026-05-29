# Work log

Task-granularity record of work in this repo, indexed by short commit hash. See `CLAUDE.md` → "Work log (log.md)" for the rule. Each entry is one line:

```
- **YYYY-MM-DD** — <one or two sentences of what changed and why>. Task: `<task-name>`. [Subtask: `<subtask-name>`.] Commit: `<short-hash>`.
```

Newest entries at the bottom.

- **2026-05-09** — Initial brainstorm captured as `spec-rough-draft.md` and `open-questions.md` to seed the project before formal spec work begins. Task: `initial-brainstorm`. Commit: `c974694`.
- **2026-05-29** — Evaluated candidate workflow frameworks (GSD, Superpowers, Claude Skills, chub, Everything Claude Code, paperclip, RooFlow, Open Design) via parallel research subagents; decided to switch this project from GSD to Superpowers (single framework, not stacked) for its TDD fit and fresh learning value, keeping the framework-agnostic ai-builder log/daily-plan discipline. Refreshed `daily-plan.md`, and captured the eval-gap + TDD-non-determinism discussion points in `open-questions.md` for the discuss/spec session. Task: `workflow-decision`. Commit: `d4c3406`.
- **2026-05-29** — Sketched the deferred `llm-eval` skill blueprint (directory shape, SKILL.md, judge prompt, fixture format, run-eval flow, reusable-vs-project-specific split) in `eval-skill-sketch.md`, and decided its home: a new `ai-skills` repo (reusable skills, distinct from `ai-builder`), symlinked into `~/.claude/skills/`. Task: `workflow-decision`. Subtask: `eval-skill-sketch`. Commit: `e98779e`.

