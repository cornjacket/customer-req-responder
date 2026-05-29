# Eval skill — sketch (`llm-eval`)

> Status: **sketch / deferred**. Build during the discuss/spec phase, when we reach the eval
> open-questions (#13 and the "eval pass criteria" / "deflection target" items in
> `open-questions.md`). Captured 2026-05-29 during the workflow-decision session — see the
> "eval & TDD discussion points" section of `open-questions.md` for the rationale (Superpowers is
> TDD-first and has no eval phase, so we own the eval loop deliberately).

## What it is, in one line
A reusable skill that stands up an **LLM-as-judge eval harness** for any LLM pipeline: run your
pipeline over a fixture set, have a judge model grade each output against a rubric, and emit a
pass-rate you can track across prompt changes.

## Directory shape

```
llm-eval/                      # ~/.claude/skills/ (reusable)  OR  .claude/skills/ (this repo)
├── SKILL.md                   # the instructions Claude loads
├── references/
│   ├── judge-rubric.md        # template: what "good" means + scoring scale
│   └── eval-design.md         # checklist: failure modes, dimensions, dataset (the AI-SPEC bit GSD gave you)
├── scripts/
│   ├── run-eval.ts            # loop fixtures → pipeline → judge → tally
│   └── judge.ts               # the Gemini-as-judge call wrapper
└── assets/
    └── fixtures.example.jsonl # one example labeled case to copy
```

## SKILL.md (frontmatter + body outline)

```markdown
---
name: llm-eval
description: Set up and run an LLM-as-judge evaluation harness for an LLM
  pipeline — fixtures, judge rubric, pass-rate tracking. Use when building or
  tuning any app whose output is non-deterministic generated text.
allowed-tools: Read, Write, Edit, Bash
---

# LLM-as-judge eval harness

## When to use
The app produces generated text (no single correct string), so unit tests
can't grade quality. Use this to measure quality as a number.

## Procedure
1. Design the eval (references/eval-design.md): list failure modes, pick 2-4
   scoring dimensions, define the pass threshold, assemble ~20 fixtures.
2. Write fixtures as JSONL (assets/fixtures.example.jsonl format).
3. Wire run-eval.ts to the target pipeline's entry point.
4. Run; review per-case verdicts + aggregate pass-rate; iterate prompts.

## Guardrails
- The judge is itself an LLM call — spot-check ~20% of its verdicts by hand
  before trusting the aggregate (judges drift / are sycophantic).
- Keep fixtures version-controlled and frozen; changing them invalidates trend.
```

## The judge prompt (the heart of it — `judge-rubric.md`)

```
You are grading a customer-support auto-reply.

ORIGINAL EMAIL:
{email}

CLASSIFIED AS: urgency={urgency}
DRAFTED REPLY:
{reply}

Score each dimension PASS/FAIL with a one-line reason:
1. Addresses the request — does the reply actually respond to what was asked?
2. Tone match — does it match the {urgency} branch (urgent = prompt, reassuring;
   non-urgent = friendly, unhurried)?
3. Well-formed — greeting, body, sign-off; no hallucinated commitments/links.

Overall: PASS only if all three pass. Output strict JSON:
{ "addresses": {...}, "tone": {...}, "wellformed": {...}, "overall": "PASS|FAIL" }
```

## Fixture format (`fixtures.jsonl`)

```jsonl
{"id":"urgent-01","email":"Our prod login is down...","expect_urgency":"urgent"}
{"id":"nonurgent-01","email":"Quick question about billing cycle...","expect_urgency":"non-urgent"}
{"id":"ambiguous-01","email":"hey","expect_urgency":"unknown"}
```

The `expect_urgency` field does double duty: a deterministic assertion for **routing** (TDD-able —
Stage 1 must classify `urgent-01` as urgent) *and* it selects which rubric branch the judge applies.
The two-disciplines split living side by side in one dataset.

## What `run-eval.ts` does (pseudo-flow)

```
load fixtures
for each fixture:
  result = pipeline(fixture.email)              # your real Stage1→Stage2
  assert result.urgency === fixture.expect_urgency   # ← deterministic check (routing)
  if result.escalated: record "escalated"; continue
  verdict = judge(fixture.email, result)        # ← non-deterministic check (quality)
  record verdict
report:
  deflection_rate = handled / total             # ← your actual success metric
  quality_pass_rate = PASS / handled
  per-dimension breakdown + list of FAILs to eyeball
```

`deflection_rate` is the payoff — literally the success metric from the spec, computed automatically.

## Why a skill (reusable core vs. project-specific config)

| Reusable across projects (the skill)        | Project-specific (config you fill in)         |
|---------------------------------------------|-----------------------------------------------|
| run-eval loop + report structure            | the rubric dimensions + pass threshold        |
| judge-call wrapper + JSON-verdict contract  | the fixture set                               |
| eval-design checklist (failure modes, etc.) | wiring to *this* pipeline's entry point       |
| guardrails (spot-check judge, freeze fixtures) | the judge model choice (Gemini here)       |

The skill carries the ~70% scaffolding + discipline; each project supplies the ~30% that's truly
its own. This is the AI-SPEC discipline GSD gave us, re-homed into our own toolkit so switching to
Superpowers doesn't lose it.

## Placement — DECIDED (2026-05-29)
- The `llm-eval` skill (and the eval harness) will live in a **new `ai-skills` repo** — a dedicated
  home for reusable Claude skills, **distinct from the `ai-builder` repos**. Source is versioned
  there, then installed/symlinked into `~/.claude/skills/` so it is user-scoped (available to every
  project), not committed into `customer-req-responder`.

## Open threads still to settle when building
- **Judge rubric** dimensions + pass threshold (ties back to open-questions #13).
- Whether `ai-skills` also gets added to `ai-project-status` tracking (log.md / daily-plan.md).
