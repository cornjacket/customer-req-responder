# Customer Request Responder — Rough Spec Draft

Working notes. Not a formal SPEC yet. Extend freely; we'll formalize into `SPEC.md` later.

## Problem
Auto-draft replies to inbound customer emails / tickets.

## Decisions (locked from prior session)

| Area | Decision |
|---|---|
| Channel | Email (IMAP / Gmail / etc.) |
| Interface (v1) | CLI that injects email test cases into the system |
| Send policy | Auto-send by default; escalate to human when unable to classify or handle |
| v1 pipeline | 2 stages: (1) classify, (2) generate draft reply matched to urgency branch |
| Tech stack | TypeScript / Node (likely a web service later) |
| LLM | Google Gemini; credentials in `.env` in cwd |
| Urgency taxonomy | Ternary: `urgent` \| `non-urgent` \| `unknown` (no other values) |
| Reply grounding | Pure LLM generation — no RAG / external knowledge in v1 |
| Success metric | % of emails auto-handled without escalation (deflection rate) |

## Pipeline

**Stage 1 — classify**
- Input: raw email
- Output: XML with fields `company`, `contact`, `request`, `urgency`
- Internally parsed into a map
- Routes on `urgency`:
  - `urgent` → Stage 2 urgent-prompt generation
  - `non-urgent` → Stage 2 non-urgent-prompt generation
  - `unknown` → escalate to human (skip Stage 2)

**Stage 2 — draft generation**
- Two prompt variants: urgent and non-urgent
- Each prompt includes a 1-shot example showing the corresponding urgency

## Testing

- **Routing tests:** mock LLM calls with known responses to exercise each route (`urgent`, `non-urgent`, `unknown`)
- **Robustness:** include a test input that produces a malformed XML response from Stage 1
- **Response quality eval:** use Gemini-as-judge to confirm the generated draft is acceptable

## Open Questions

1. **Pure-LLM realism** — is unaided generation good enough for the target domain, or will we need RAG / templates / account context sooner than v1+1?
2. **Deflection target** — what % deflection counts as success? How do we measure it (manual review, customer reply rate, escalation rate)?
3. **Eval pass criteria** — what does the Gemini-as-judge check for, and what's the pass threshold?
4. **Escalation surface** — how does the `unknown` path actually escalate (CLI exit code? log file? written to a queue dir?) in v1?
5. **Defer-to-discuss list** — what else should we explicitly leave open until discuss-phase vs. lock down now?

## Additions
<!-- Add new context, constraints, examples, or decisions below as you think of them. -->
