# Multipath Reasoning: method overview

The core ideas behind the multipath-reasoning skill: why counting is not
verification, the false-attractor problem, the vocabulary and the default
five-path loop. Moved from the root README (2026-10). For how the loop is
wired, see [technical-description.md](technical-description.md); for the full
practitioner method, see the [developer guide](../skill/developer-guide.md).

## Why majority vote is not enough

Five paths might say: database race, database race, retry ordering, database race, stale cache.

A vote would crown “database race.” All three agreeing paths may share one bad assumption.

Multipath distinguishes **agreement** (population stability) from **support** (source, tests, constraints, independent reconstruction). Consensus is not truth.

The operational form of that is: **verify, don't count.** Reduce the population to its
*distinct* claims, discard the frequencies, and check each against a rule derived from the
evidence. Counting weights by frequency, and when a model is biased the correct answer is
by definition the infrequent one. Verification ignores frequency, so a 1-in-15 answer is
worth exactly as much as a 9-in-15 one — which is how a recorded run recovered a minority
answer that every counting rule buried.

## The false-attractor problem

A naïve recursive loop:

```
G0 (5 independent paths)
    → one synthesis C0
    → every later path inherits C0
    → stronger agreement
    → still stronger agreement
```

Later paths are no longer independent. If C0 is wrong, recursion can raise apparent confidence while amplifying the error.

**Coherence is not fidelity.** A population can agree strongly and still be wrong.

Multipath therefore does **not** feed a single prose answer to every later path. It also does not feed the full parent `state.json` to source-heavy paths. Those paths get a **constraint view**: SOURCE + constraints + open questions, without conserved findings, scores, or stability status. That view can still name prior *hypotheses* (disagreements, minority findings). It is a reconstructability *probe*, not a blind SOURCE-only test. See [docs/technical-description.md](technical-description.md).

## Core ideas (short)

| Idea | Meaning |
|------|---------|
| **SOURCE** | The original request, constraints, and evidence. Never replaced by later summaries. |
| **Admissibility state** | Compact `state.json`: what is known, uncertain, and still allowed. Not “the answer.” |
| **Retention ↔ Fresh Actualisation** | Experimental pair: keep what earned preservation; reconstruct the rest from SOURCE. Not a 50/50 law. |
| **Reconstructability** | Can the claim be recovered again without being *told* the previous answer? Stronger than inheritance. |
| **Provenance** | Where a claim came from, who reconstructed it, who challenged it, what evidence supports it. |
| **Cross-order consistency** | Locally correct can still break a module, a document, or a theory. |
| **STABLE_WITH_UNCERTAINTY** | A legitimate outcome. Do not invent a unique winner. |

Claim classes: `VERIFIED_STABLE` > `RECONSTRUCTED_STABLE` > `MIXED_STABLE` > `INHERITED_STABLE` > `UNSTABLE`.

Inherited stability is weaker than reconstructed stability.

## Default five-path loop

**Generation 0:** five independent paths, identical SOURCE, no sibling answers.

Then evaluate into `state.json`. A schema check (`STRUCTURAL_OK`) only means the JSON has the required keys. It is **not** proof that the method worked.

**Later generations (default mix, `ROLE_SEQUENCE`, not a law):**

| Count | Role | What they see |
|------:|------|----------------|
| 1 | Blind | SOURCE + hard/soft constraints only (the reconstructability probe) |
| 1 | Dissent / minority | Disagreements and minority findings as tests |
| 1 | Source-heavy | Constraint view: constraints + open questions; still names prior hypotheses |
| 1 | Retained-structure | Conserved findings + provenance, not scores |
| 1 | Full-state | Complete state; still not a verdict |

Repeat only while another generation would add information. A default run that can satisfy the two-transition stop test is **15 path invocations** (N=5 × 3 generations), up to **25** at the generation cap. Recorded experiments are in [`experiments/`](../experiments/README.md).
