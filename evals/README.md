# Evaluator Regression Suite

This suite is the regression safety net for the **policy in this repository** — the
standards and the evaluator prompt. It tests one thing: does the evaluator flag what the
standards say it should, and stay silent on what it shouldn't.

It lives here, with the standards, on purpose. Each fixture is the **executable form of a
standard's "what is / isn't a violation" prose** — the standard states the intent in words,
the fixture proves it in behavior. They evolve together: a change to a standard, its
fixtures, and the baseline all belong in the same pull request, reviewed together. And
because the suite is wired into this repo's CI (`.github/workflows/evals.yml`), a standards
or prompt change that regresses the corpus **blocks its own PR** — the policy is held to a
regression test before it can be pinned and rolled out to consuming repos.

The suite depends only on this repo (the prompt, the standards, and the fixtures' embedded
synthetic diffs). It does **not** depend on any application repository.

## Layout

```
evals/
├── fixtures/        labeled cases — should-flag (recall) and should-not-flag (precision)
├── baseline.json    last-accepted per-standard recall / false-positive rates
├── run_evals.py     the harness (runner + tolerance scorer + baseline diff)
└── evaluate.py      the evaluator under test (see note below)
```

## Running it

```bash
# Stub — deterministic self-test, no API key, no cost (verifies the harness plumbing).
python3 evals/run_evals.py

# Live — the real regression test, against this repo's standards.
ANTHROPIC_API_KEY=sk-... python3 evals/run_evals.py --live --standards-dir .

# Re-baseline after an intentionally accepted behavior change.
python3 evals/run_evals.py --live --standards-dir . --update-baseline
```

Scoring is by tolerance, not equality (the model is non-deterministic): an `expect` passes
if the standard fired in ≥ `passThreshold` of N runs; an `expectNot` passes if it fired in
≤ `tolerance`. The runner prints per-standard recall / false-positive rates, a per-fixture
breakdown, and a comparison to `baseline.json`, and exits non-zero on any regression so CI
can gate on it.

## The flywheel

Every false positive or false negative seen on a real PR becomes a new fixture. Over time
the corpus becomes the institutional memory of every mistake the evaluator has made, and no
future policy change can silently reintroduce one.

## Growing coverage (multi-language)

The seed fixtures are Java because the POC's demo app was Java, but the standards are
language-agnostic. As the fleet spans languages, grow the corpus to cover them — structure
fixtures by standard and language so coverage is legible (e.g. "STD-001 proven on Java and
Python, not yet Go").

## Note: the evaluator copy (interim)

`evaluate.py` here is the evaluator the suite tests; the production pipeline in each app
repo currently carries its own copy. In the enterprise architecture these converge into a
single shared engine in the central `governance-ci` repo, consumed by both the app pipelines
and this suite — so there is exactly one evaluator, tested here and run in production. Until
then, keep the two copies in sync when `evaluate.py` changes. See `SCALING-ARCHITECTURE.md`.
