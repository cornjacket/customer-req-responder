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

## Pre-existing open questions from spec-rough-draft.md

- **Pure-LLM realism** — is unaided generation good enough, or will we need RAG / templates / account context sooner than v1+1?
- **Deflection target** — what % counts as success? How do we measure (manual review, customer reply rate, escalation rate)?
- **Eval pass criteria** — what does the Gemini-as-judge check for, and what's the pass threshold? (overlaps with #13)
- **Escalation surface** — how does the `unknown` path actually escalate in v1? (overlaps with #8)
- **Defer-to-discuss list** — what else should we explicitly leave open until discuss-phase vs. lock down now?
