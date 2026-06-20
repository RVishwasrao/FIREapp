# NorthstarFI — Design Session Decisions

> Output of a BMAD party-mode roundtable (Dallas AI Summer Camp, Team 5).
> Source ideas: `ref_docs/Proposal Creation Draft.docx`, `ref_docs/Team 5 - Ideation DALLAS AI 2026..docx`.
> Date: 2026-06-20.

## Product in one line

NorthstarFI is **not another FIRE calculator**. It is a **thinking partner that stress-tests a
user's FIRE plan** — surfacing the assumptions they didn't realize were assumptions, grounded in a
bounded set of approved, cited sources. It is explicitly **not a fiduciary financial advisor**.

## Locked decisions

1. **Positioning pivot.** Stop competing as a calculator. Compete on *comprehension and
   commitment* — help the user see and pressure-test what they already believe. (The market is
   crowded with calculators: cFIREsim, ProjectionLab, Boldin, "ask ChatGPT".)

2. **AI = explainer, Python = calculator.** All deterministic FIRE math (FIRE number, safe
   withdrawal rate, savings gap, sequence-of-returns / fragility perturbations) is computed in
   Python. The LLM never computes numbers — it only synthesizes and explains.

3. **Bounded, approved research sources** (Winston's guardrail, endorsed). The research layer draws
   only from a curated corpus, never the open web.

4. **On finding a broken assumption: REFLECT and CORRECT.** Mirror the user's belief back, then
   guide them toward a more defensible one. Never **prescribe** a financial action — that is the
   not-a-fiduciary line.

## Core mechanism (Dr. Quinn)

The root problem: **users don't know their inputs are assumptions.** A FIRE plan is a product of
compounding assumptions (savings rate, real return, withdrawal rate, horizon, sequence-of-returns
risk). Calculators treat these as ground truth.

The loop:
1. **Elicit** the assumption explicitly ("You entered 4% withdrawal — what's that based on?").
2. **Surface its epistemic status** from a cited source (Trinity Study modeled 30-year windows; your
   plan runs 47).
3. **Perturb the *hidden* assumption, not just the slider** — e.g. the constant-real-spending
   assumption baked inside the 4% rule, since real spending is rarely flat.

Weakest assumption most FIRE hopefuls get wrong: **the withdrawal rate** (the 4% rule, over-trusted
and misapplied to long horizons).

## The reflect-and-correct interaction (Sally)

Three beats per turn:
- **Echo** — play the assumption back in plain language (prove you understood).
- **Reveal the seam** — surface the hidden assumption. One finding, one citation, short.
- **Return the wheel** — end with a question, so the user discovers the flaw themselves.

Rules:
- **One assumption challenged per turn** (two only if tightly related) — more is demoralizing.
- The UI **never shows a "corrected plan"**, only *scenarios* ("here's what 35 years looks like").
- Never "you should." The not-an-advisor frame lives in the **grammar**, not a footer disclaimer.
- Every factual claim is attributed to a source inline.

## MVP scope (John) — for the July 25 showcase

**One user:** a 30-something professional who already has a FIRE number in their head and has never
had it seriously challenged.
**One job:** *surface the assumptions in my plan most likely to be wrong.*

**Thinnest slice:** plan in (free text / fields) → personalized provocation out → conversational
follow-up. Text in, provocation out.

**Out of scope:** bank/brokerage integration, multi-user/household, live web research, account
auth/login/saved sessions, mobile UI, full education module, PDF reports.

**Acceptance criteria (live-checkable on demo day):**
1. User enters a FIRE plan in under 2 minutes, no tutorial.
2. The app surfaces ≥2 non-obvious assumption challenges specific to *that user's* inputs.
3. Every claim traces to an approved source visible on screen.
4. The user can respond and the app sharpens its pushback (a conversation, not one-shot).
5. Start → first insight in under 90 seconds.

**Riskiest product assumption:** the app produces *personalized* challenges, not generic platitudes.
De-risk this week with a "does this feel like it's talking to ME or everyone?" rater test.

## Architecture (Winston + Amelia)

**Stack:** Python, pandas/numpy, ChromaDB (or FAISS, local), SQLite, Streamlit, an LLM
(GPT-4o-mini default; Gemini Flash fallback if Google credits exist). Keep it boring — one process.

**Corpus, three buckets:**
- *Static research* (embed once): Bengen 1994, Trinity Study, Pfau withdrawal research, Bogleheads
  wiki sections, ERN SWR series, IRS Pub 590-A/B excerpts, Vanguard/Fidelity return tables.
- *Structured reference tables* (query, don't embed): SWR outcomes by horizon/allocation, asset-class
  return distributions, IRS limits by year, tax brackets, FIRE milestone formulas. Each row carries a
  `data_as_of` timestamp.
- *Live numeric* (narrow): yfinance prices, current TIPS yield, I-bond rates. Nothing else.

**4-step pipeline** (LLM only touches step 4):
```
user_input
  → [1] assumption_extractor.py   → structured assumptions
  → [2] fire_engine.py            → baseline + perturbed scenarios   (PURE Python, no LLM)
  → [3] retriever.py              → cited chunks (ChromaDB + SQLite)
  → [4] prompts/reflect_correct.py→ LLM → ReflectResponse (Pydantic)
```

**Keeping the LLM off the math & off prescription (guardrail at 3 layers):**
- *Input*: numbers handed to the LLM as pre-computed strings; it has nothing to compute.
- *Schema*: `ReflectResponse` has **no numeric fields** — invented numbers have nowhere to land.
- *Validator*: post-LLM regex sweep rejects `$`-figures and action verbs (invest/move/buy/sell/
  allocate/"you should"); retry once, else fall back to a canned template.

This guardrail is the *same rule expressed three ways*: Sally's "never say you should" (copy),
Amelia's `validator.py` reject (code), Murat's prescription classifier (test).

## Quality & risk plan (Murat)

Risks ranked by impact × likelihood:
1. **"Correct" drifting into "prescribe"** (showcase-ender in front of finance-savvy judges). Test:
   regex prescription classifier (0% on test set) + a **12-input adversarial red-team**, target
   **12/12 pass**, <10/12 is a blocker.
2. **Hallucinated/uncited claims.** Test: citation-audit harness — every cited source ID must exist
   in ChromaDB and be semantically relevant (cosine ≥ 0.75) to the claim.
3. **Python math correctness.** Easiest win — exhaustive `pytest` on `fire_engine.py`, week 1.
4. **Demo fragility** (live LLM latency/timeout on stage). Mitigation: a **demo-mode cache** of
   pre-generated responses for the golden-set profiles.

**"Personalized, not generic" as a measurable bar:** golden set of **8 narrative user profiles**,
each with 2–3 expert-written expected challenges. Specificity score 0–3, redundancy 0–1 (inverted).
Pass = mean specificity ≥ 2.0 and redundancy hits < 2/8. Blind rater pair, Cohen's κ ≥ 0.6. Run
weekly; watch for regression.

**Test-first build order:** Wk1–2 Python math + prescribe-drift red-team; Wk3–4 citation audit +
first golden-set eval; Wk5 full regression + small load test; Wk6 freeze, cache, rehearse.

## Open threads (not yet decided)

- **Session/memory model:** does the companion remember the user between visits? Victor flagged this
  as make-or-break for the "companion" promise. Affects the SQLite schema.
- **Off-corpus boundary:** is "should I pay off my mortgage before retiring?" in scope? Needs source
  material if yes.
- **Corpus ownership:** one named owner vs. rotating. Corpus drift is the most likely showcase
  failure; ~15–20 hrs of curation, assigned explicitly.
