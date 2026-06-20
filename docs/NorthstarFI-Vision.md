# NorthstarFI — Vision

*Dallas AI Summer Camp · Team 5 · v1 (2026-06-20)*

## The intent

Most people chasing FIRE (Financial Independence, Retire Early) don't fail for lack of a
calculator — they fail because no one ever challenges the **assumptions** their plan quietly rests
on. They punch in "4% withdrawal, 7% returns, retire at 50," get a confident number back, and walk
away believing a forecast that was never stress-tested.

**NorthstarFI is a thinking partner that stress-tests your FIRE plan.** It surfaces the assumptions
you didn't realize were assumptions, shows you how fragile or robust your plan is when they're
perturbed, and helps you rebuild a more defensible belief — grounded in real research, every claim
cited. It is a curious, well-read friend who has done the homework, **not** a fiduciary financial
advisor.

## Objectives

1. **Reveal hidden assumptions.** Take a user's plan and identify the beliefs most likely to be
   wrong — especially the structural ones hiding inside familiar rules (e.g. the constant-spending
   assumption baked into the 4% rule).
2. **Stress-test, transparently.** Compute how the plan holds up when key assumptions are perturbed,
   using deterministic math the user can trust and trace.
3. **Reflect and correct, never prescribe.** Mirror the user's belief back, explain what the
   evidence says, and guide them toward a sounder framing — without ever telling them what to do
   with their money.
4. **Ground every claim.** Draw only from an approved, cited research corpus; show the source.
5. **Ship a working, demonstrable prototype** by the July 25 showcase.

## Approach

- **AI explains; Python calculates.** All financial math (FIRE number, safe withdrawal rate, savings
  gap, sequence-of-returns risk) runs in deterministic Python. The language model only *interprets*
  results and *converses* — it never invents numbers.
- **Bounded research, not the open web.** A curated corpus of credible sources (Trinity Study,
  Bengen, Pfau, Bogleheads, IRS limits, asset-class return tables) backs every factual statement.
  If there's no source, the app says so rather than guessing.
- **A reflect-and-correct conversation.** Each exchange follows three beats — *echo* the user's
  belief, *reveal* the hidden assumption inside it (one finding, one citation), and *return a
  question* so the user reaches the insight themselves. One challenge at a time, so it informs
  rather than overwhelms.
- **Guardrails in code, copy, and tests.** The "not advice" boundary is enforced three ways: the
  model is given no way to compute or prescribe, the output format rejects dollar figures and action
  verbs, and an adversarial test set checks that "correct" never drifts into "you should."
- **Lean, boring stack.** Python, Streamlit, SQLite, a small local vector store, and a low-cost LLM
  (e.g. GPT-4o-mini). Built test-first, deterministic parts first.

## Why this matters (justification)

- **The market is crowded with calculators and empty of partners.** cFIREsim, ProjectionLab, Boldin,
  and "ask ChatGPT" all *compute*. None of them pressure-test what the user already believes. The
  open gap is comprehension and confidence, not arithmetic.
- **Transparency builds trust where money meets anxiety.** Separating trustworthy math (Python) from
  explanation (AI), and citing every claim, makes the tool credible without pretending to be an
  advisor.
- **The constraint is the strategy.** "Not a fiduciary" keeps us out of regulatory scope *and*
  pushes us toward the more valuable role — helping users think, not deciding for them.
- **It fits the timeline and budget.** A focused thinking-partner prototype is achievable by a
  student team in six weeks; full advisory features are not — and aren't the point.

## Scope boundary (for the prototype)

**In:** a single FIRE-curious user enters a plan and gets personalized, cited, assumption-level
pushback in conversation. **Out:** bank/brokerage integration, account login, multi-user/household
planning, live web research, mobile app, and any personalized investment recommendation.

## Key assumptions

- Users have rough numbers (savings, target, age) and a few untested beliefs — but not exact,
  audited portfolio data. The experience must work with approximate, incomplete input.
- A curated corpus can answer the large majority of common FIRE questions; gaps are handled by the
  app honestly saying "I don't have a source for that."
- A low-cost LLM, constrained to explanation over a structured pipeline, is reliable enough — the
  guardrails, not the model size, ensure safety.
- Educational, clearly-disclaimed positioning is sufficient for a non-commercial summer prototype;
  no integration with real accounts means no live-money or compliance risk.
- Reference data (returns, IRS limits) is point-in-time and labeled "as of" its date; it is not
  guaranteed current.

---

*NorthstarFI shows you what you're betting on — so the decision stays yours.*
