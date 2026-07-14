# LEB — LLM Engineering Benchmark

> ⚠️ **Before working on this repository: `git pull`.**

🇧🇷 [Versão em português](README_br.md)

**A Software Engineering evaluation standard for LLMs.** Spec version: **1.0.0**.

LEB is not a prompt benchmark and does not measure who writes the prettiest code. It measures **who can evolve a legacy system without breaking it** — finding real flaws, fixing them, preserving compatibility, and explaining decisions like a senior engineer would.

## Why another benchmark?

Existing benchmarks measure greenfield code generation or isolated issue-solving. None of them measures the work that dominates real-world engineering: **maintaining legacy systems while consumers depend on the current behavior**. LEB scores engineering, compatibility, architecture, security, performance and technical maturity — and **deducts** points from models that rewrite everything, swap technologies without need, or break public contracts.

## How it works (60 seconds)

1. The model receives a **real legacy system** (with planted flaws) + a fixed, neutral task statement.
2. It reports, fixes and justifies — without knowing which or how many flaws exist.
3. The submission is checked against the **[Official Failure Matrix](matrix/MATRIX.md)** — a hidden answer key stating exactly what exists (and what is a decoy). This is what makes the evaluation objective, reproducible and comparable across LLMs: we verify exactly which flaws the model found, fixed, ignored — and which ones it *invented*.
4. Out comes a **[scorecard](scoring/scorecard-template.md)** from 0 to 1000.

| Category | Weight |
| --- | ---: |
| Security (SEC) | 250 |
| Architecture (ARCH) | 200 |
| Bugs (BUG) | 150 |
| Performance (PERF) | 150 |
| Clean Code (CLN) | 100 |
| Compatibility (COMP) | 100 |
| Technical Explanation (EXPL) | 50 |

Compatibility is **conduct-based**: it starts at 100 and every violation deducts (mysqli→PDO without need: **−20**; changing a public signature: **−30**). Global penalties (new bug, regression, unnecessary rewrite, false positive against decoys) deduct from the total — so a model can't "win" by throwing everything away and starting from scratch.

## Scoring model

Each planted flaw is scored on independent, cumulative criteria. Two templates:

- **Template C — Fix** (SEC, PERF, BUG): Found / Explained / Fixed / No regression / Kept compatibility (e.g. a Critical flaw: 2+2+3+2+1 = 10 pts).
- **Template R — Refactoring** (ARCH, CLN): Identified / Explained / Refactored / Compatible (e.g. Critical: 3+2+5+2 = 12 pts).

Raw category points are normalized to the official weights, so **every instance is worth exactly 1000**. Details: [scoring/SCORING.md](scoring/SCORING.md).

## Levels

| Level | Lines | Goal |
| --- | ---: | --- |
| LEB-100 | ~300 | Simple refactoring |
| LEB-200 | ~1,000 | Small legacy system |
| LEB-300 | ~3,000 | Multiple files |
| LEB-400 | ~8,000 | Enterprise project |
| LEB-500 | ~20,000 | Full corporate system |

## Documents

| Doc | Contents |
| --- | --- |
| **[SPEC.md](SPEC.md)** | Normative core (RFC-style): scoring, penalties, invariants |
| [taxonomy/](taxonomy/) | The 85 official failures: [SEC](taxonomy/SEC.md) · [ARCH](taxonomy/ARCH.md) · [PERF](taxonomy/PERF.md) · [BUG](taxonomy/BUG.md) · [CLN](taxonomy/CLN.md) · [COMP](taxonomy/COMP.md) |
| [scoring/SCORING.md](scoring/SCORING.md) | Criteria per severity, normalization, explanation rubric, grade seals |
| [matrix/MATRIX.md](matrix/MATRIX.md) | **Official Failure Matrix** — construction, decoys, hash-based concealment |
| [levels/LEVELS.md](levels/LEVELS.md) | LEB-100 (~300 lines) → LEB-500 (~20,000 lines) |
| [protocol/PROTOCOL.md](protocol/PROTOCOL.md) | Canonical statement, S/A modes, 3 runs → median, anti-gaming |
| `*/**.schema.json` | Machine-readable formats for matrix and scorecard |

Note: the specification documents are currently written in Portuguese (pt-BR); English translations are planned.

## Running it

1. Pick a current (non-retired) instance and its level.
2. Give the model `code/` + `manifest.md` + the canonical statement ([protocol/PROTOCOL.md §2](protocol/PROTOCOL.md)) — never anything from `private/`.
3. Run 3 independent executions; the official score is the median total.
4. Evaluate: public-surface diff → characterization tests → per-flaw verifies → report×matrix matching → explanation rubric → scorecard (`.md` + `.json`).

## Status

- [x] Specification 1.0.0 (this repository)
- [x] First instance: **[LEB-100-A](instances/LEB-100-A/)** — PHP legacy code, 13 planted flaws + 2 decoys, private matrix, characterization + verify probes (validated live: characterization 22/22 green on both pristine and fixed code; probes flip PLANTADA→CORRIGIDA)
- [ ] Evaluation harness (public-surface diff + verify runner + scorecard computation)
- [ ] Reference runs with current models

## License & contributing

Taxonomy IDs are immutable (SPEC §9). Proposals for new failures/levels: open an issue with the real-world case motivating it. Matrices of **active** instances never enter this public repository — only their SHA-256 hashes; matrices are revealed when an instance is retired.
