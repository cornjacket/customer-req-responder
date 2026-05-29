# Open Questions — for next session

Questions surfaced after the additions to `spec-rough-draft.md`. Answer inline, then we can fold the resolved ones back into the draft.

**Blocking** (need answers before writing code): 2, 4, 5, 7, 9, 11.

## Stage 1 (classify)

1. What does Stage 1 see as input — just the email body, or also subject / from / headers?
   > _answer:_

2. **[blocking]** Can you give an example of the expected XML output? Specifically: what goes in `request` (1-line summary vs. full quoted text), and what happens when a field can't be extracted (empty tag? omitted? forces `urgency=unknown`)?
   > _answer:_

3. Why XML over JSON for the Stage 1 response — just preference, or a specific reason (better LLM compliance, ease of parsing partial output)?
   > _answer:_

## Stage 2 (draft)

4. **[blocking]** Does Stage 2 see the original email plus the extracted map, or only the map?
   > _answer:_

5. **[blocking]** What does Stage 2 produce — reply body only, or subject + body, or a full RFC-822 message? Signature included?
   > _answer:_

6. Should the 1-shot examples be invented now, drawn from a corpus you'll provide, or stubbed for me to fill in later?
   > _answer:_

## Error / edge handling

7. **[blocking]** Malformed-XML behavior: retry once, immediately escalate as `unknown`, or hard-fail?
   > _answer:_

8. How does an `unknown` escalation actually surface in the CLI — exit code, written to a directory, logged with a flag, printed to stderr?
   > _answer:_

## LLM / Gemini

9. **[blocking]** Which Gemini model (`gemini-2.5-flash`, `gemini-2.5-pro`, etc.)? Direct Google API or via Vertex?
   > _answer:_

10. What keys does the existing `.env` define — just `GOOGLE_API_KEY`, or also model name / project ID?
    > _answer:_

## CLI shape

11. **[blocking]** Invocation: `cli < email.txt`, `cli ./email.eml`, `cli --fixture name`, or something else? One-at-a-time or batch a directory?
    > _answer:_

12. Output destination: stdout, a `drafts/` directory, both?
    > _answer:_

## Eval

13. What does the Gemini-as-judge eval actually check — that the urgency tone matches, that the request is addressed, that the reply is well-formed, all of the above? Pass/fail or scored?
    > _answer:_

## Project meta

14. Test framework preference (vitest / jest / node:test)?
    > _answer:_

15. Are these test inputs fixtures committed to the repo, or generated on the fly?
    > _answer:_

---

## Workflow-decision session (2026-05-29) — eval & TDD discussion points

Context: decided to use **Superpowers** (not GSD) for this project. Superpowers is TDD-first and
has no LLM/eval phase, so two design points surfaced that we must own deliberately. Capturing here
for the discuss/spec session — these refine open questions #2, #3, #13 and the pre-existing
"Eval pass criteria" / "Deflection target" items.

### Root framing — two kinds of code, two kinds of "correctness"
The app has (a) **deterministic plumbing** (XML parse, urgency routing, escalation, malformed
handling) where the exact output is knowable in advance, and (b) **non-deterministic generation**
(the Gemini draft reply) where the same email yields different valid prose and "good" is a
judgment call. These need different correctness tools; most confusion comes from using one tool for
both. → TDD owns (a); an eval loop owns (b).

### Discussion point 1 — we need an eval, and the framework won't remind us
- **Problem:** the *generation* half has no expected value to assert, so unit tests can't say if it's
  good. The thing that answers "is it good?" is an **eval**: a small labeled dataset (~20 emails) →
  run the pipeline → score each output → aggregate number to compare prompt changes against.
- **Scoring via Gemini-as-judge:** a second Gemini call grades the first ("here's the email + the
  drafted reply — does it address the request? is the tone right for `urgent`? pass/fail + reason").
- **Why it's not optional:** the success metric *is* deflection rate (% auto-handled without
  escalation) — can't measure success without an eval harness.
- **What GSD gave that Superpowers doesn't:** `gsd-ai-integration-phase` (AI-SPEC forces designing
  the eval — failure modes, judge rubric, reference dataset — before coding) and `gsd-eval-review`
  (after-the-fact coverage audit). Superpowers is silent on grading non-deterministic output, so
  nothing will prompt us to build the eval.
- **Mitigation / decision to make:** hand-spec the eval (small — one judge prompt + ~20 fixtures +
  a tally script), OR author our own reusable **`eval` skill** (fits the ai-builder framework goal).
  → Open: which path? what does the judge rubric check? pass threshold? where does the fixture set
  live?

### Discussion point 2 — TDD structurally can't cover generation
- **Problem:** TDD assumes you can write the exact correct output in advance. True for plumbing
  (`parse(...) → {urgency:"urgent"}`, `route({urgency:"unknown"}) → escalate()`), false for the
  draft (Gemini won't reproduce an exact string; a different good reply would fail the test).
- **The trap:** Superpowers will show all-green while the most valuable + fragile part (reply
  quality) has zero coverage — green means "plumbing works," not "product works."
- **Mitigation / decision:** split correctness by the two-kinds distinction — TDD the plumbing hard
  (let Superpowers drive), let the Gemini-as-judge eval loop own generation quality, run them as
  two separate disciplines.

> Net: discussion points 1 and 2 are the same gap from two sides — Superpowers covers the
> deterministic half and is blind to the non-deterministic half, which is where the product's value
> and the success metric live. Go in knowing it; stand up the eval loop alongside TDD.

---

## Pre-existing open questions from spec-rough-draft.md

- **Pure-LLM realism** — is unaided generation good enough, or will we need RAG / templates / account context sooner than v1+1?
- **Deflection target** — what % counts as success? How do we measure (manual review, customer reply rate, escalation rate)?
- **Eval pass criteria** — what does the Gemini-as-judge check for, and what's the pass threshold? (overlaps with #13)
- **Escalation surface** — how does the `unknown` path actually escalate in v1? (overlaps with #8)
- **Defer-to-discuss list** — what else should we explicitly leave open until discuss-phase vs. lock down now?
