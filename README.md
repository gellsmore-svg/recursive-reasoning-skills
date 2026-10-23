# Recursive Reasoning Skills

**Status:** experimental (v0.1.1) · **License:** Apache-2.0

**Skills that make an LLM harness reason more reliably on hard problems, by
verifying candidate answers instead of counting them.** A single fluent
answer can be wrong, and several agreeing answers can share one mistake.
These skills treat that as an engineering problem, not a voting problem.

> Accumulate learning without allowing accumulated learning to become
> unquestionable ancestry.

| Skill | Folder | Does |
|-------|--------|------|
| `multipath-reasoning` | [`skill/`](skill/SKILL.md) | Runs independent paths across comparable models, derives an audit rule from the evidence, verifies the *distinct* candidates against it, and keeps a structured admissibility state rather than a winner paragraph |
| `recursive-confidence-loop` | [`recursive-confidence-loop/`](recursive-confidence-loop/SKILL.md) | Passes one request through an LLM repeatedly until a fixed, LLM-selected diagnostic score vector stabilizes. Not multipath, and it does not claim that score stability is verification |

## Is it for you?

**Good fit:** difficult or consequential work, such as ambiguous evidence,
competing hypotheses, hard debugging, architecture with several viable
options, substantial review, or evidence synthesis, where a confident wrong
answer is expensive.

**Not a fit:** lookups, formatting, mechanical edits, obvious one-line fixes,
or any claim cheaper to **test** than to debate. Five agents will not make a
missing semicolon more true; run the test, read the file, check the docs.
Multipath is also expensive: a default run is **15 path invocations**, up to
25.

## Try it

The skills are plain `SKILL.md` folders. Copy them unmodified into your
host's skill directory. For Claude Code:

```bash
cp -a skill ~/.claude/skills/multipath-reasoning
cp -a recursive-confidence-loop ~/.claude/skills/recursive-confidence-loop
```

Then, in a new session: `/multipath-reasoning <task>`.

[docs/installing-skills.md](docs/installing-skills.md) covers Grok Build,
Claude Code, Codex and Amazon Kiro, including each host's child-session
primitive and the traps specific to it.

To check a run's state file:

```bash
python3 skill/scripts/validate_state.py path/to/state.json
# STRUCTURAL_OK means the schema is right. It is not proof the method worked.
```

## How Multipath works

1. Draw candidate answers from **several model families of comparable
   capability**, so the correct answer has a chance of being produced at
   all.
2. Keep the paths separate long enough to preserve real variation.
3. Derive an **audit rule** from the evidence: what invariant must hold, and
   how would a violation be checked mechanically?
4. **Verify** the distinct candidates against that rule with the vote
   counts stripped, using more than one verifier.
5. Build a **structured admissibility state** (what must be respected, what
   is still open) rather than picking a winner.
6. Stop when the candidate set stops growing and the result stabilizes.
   *Stable with uncertainty* is a legitimate outcome.

Later generations do not all inherit one synthesis, because a loop that
feeds every path the same prior answer turns agreement into a false
attractor. Each generation mixes roles that see different amounts of the
prior state: Blind (source and constraints only), Dissent, Source-heavy,
Retained-structure and Full-state. The
reasoning, vocabulary and default five-path loop are in
[docs/method-overview.md](docs/method-overview.md).

## What the evidence says

### What does not work

Measured rather than assumed — but on a narrow task family, and **22 of 25 records restage a
single bug**. Two external audits (`experiments/audits/`) judged the stronger forms of these
overstated. Read them as observations, not laws:

- **Resampling one model and taking the mode.** One model returned an identical wrong answer
  21/21 times on one bug — but majority vote was correct on two other populations. Where bias
  exists it does not average out; that is narrower than "more samples never help".
- **Reviewing a single answer.** Three reviewers on one task: one kept everything, one
  changed everything, one reached +0.13 on half its responses. None was usable. On this
  evidence, splitting a stage into produce-then-review is not worth building — but it is one
  task, and the claim is "not demonstrated to help", not "proven useless".
- **Convergence by frequency**, even dressed as evaluation — a same-model parent reproduces
  the population's mode and certifies it as high confidence.

### Where the value actually is

Recorded runs (`experiments/RESULTS-2026-08-19.md`) put this narrowly:

- **One model, sampled N times: little value.** Temperature perturbs the draw, not the
  prior. Errors stay correlated and biased, so more samples make a systematic mistake more
  confident. One model returned the same wrong answer on **21 of 21** samples.
- **One model with genuinely varied output: some value.** If its candidate set really does
  vary across samples, the right answer may sit somewhere in that spread, and verification
  can pick it out. But you cannot know in advance whether it does, and a degenerate
  population looks exactly like a confident correct one.
- **Several comparable models from different families: this is where it earns its cost.**
  Different training corpora fail differently, which is the decorrelation resampling cannot
  produce. In a recorded run the correct answer entered the pool only via a second family,
  was a 3-of-16 minority, and majority vote returned the wrong answer while verification
  returned the right one.

**Comparable capability is a requirement, not a preference.** Adding members that cannot do
the task dilutes coverage — measured, 83% → 59% — and adds confident noise at the
verification step.

## Where it came from

The method was *inspired by* exploratory
[Relational-Substrate](https://github.com/gellsmore-svg/Relational-Substrate)
research, but does **not** depend on that research being correct
([origins](docs/origins.md)). It needs none of the author's other systems.
How it relates to Deborah and Hoglah, as inspected rather than assumed, is
in [system-landscape.md](docs/system-landscape.md).

## Documentation

- [Technical description](docs/technical-description.md): how the loop is wired, specified vs implemented
- [Developer guide](skill/developer-guide.md): the full method for practitioners
- [Specification](docs/specification.md): the platform-independent process
- [Experiments](docs/experiments.md): how to test the method without claiming significance; records in [experiments/](experiments/README.md)
- [CHANGELOG](CHANGELOG.md) · [CONTRIBUTING](CONTRIBUTING.md) · Apache 2.0, see [LICENSE](LICENSE)
