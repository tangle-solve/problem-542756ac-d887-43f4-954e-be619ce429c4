<!-- solve-problem:542756ac-d887-43f4-954e-be619ce429c4 -->
# Cross-platform energy-to-verified-solution benchmark

A [Tangle Solve problem](https://solve.tangle.tools/problems/542756ac-d887-43f4-954e-be619ce429c4).

## What is the problem?

There is no reproducible, standardized benchmark that measures the **total energy cost, in joules, of producing one independently verified solution** to a computational problem, across platforms and algorithms. Formally: for an algorithm *A* run on hardware *H* against problem instance distribution *D*, define

> E(A,H,D) = total energy (J) consumed by all phases ÷ number of certificates accepted by an independent checker,

where "all phases" includes compilation/preprocessing, failed or timed-out runs, data movement (I/O, memory, network), and the energy of the certificate-checking procedure itself. The problem is to (a) define this metric precisely enough to be reproducible (accounting rules for what counts, measurement hardware/APIs, e.g. RAPL, IPMI, nvidia-smi), (b) build a reference benchmark suite with published, versioned data, and (c) validate that cross-platform comparisons under the metric are meaningful (low run-to-run variance, hardware-independent conclusions where claimed). The target metric is a *rate* (J per verified solution), not a total or a carbon estimate. The census leaves open whether "solution" means any feasible solution or optimal solution; the most reasonable interpretation is any output passing an independent verifier, per instance drawn from D, with solvers allowed multiple attempts.

## Why does it matters?

Such a benchmark would settle whether algorithmic improvements (better asymptotics, better constants) or hardware improvements (accelerators, DVFS) actually reduce end-to-end energy, or merely shift it into I/O and failed runs. Practitioners in HPC, ML, and combinatorial solving (SAT competitions already run at massive scale) currently rely on wall-clock rankings and coarse carbon calculators with no measurement of retry/verification overhead. Without it, "green computing" claims are unverifiable and hardware procurement and algorithm selection decisions are made blind to energy.

## What's already known?

- **Green Algorithms (Lannelongue, Grealey, Inouye 2021)** provides a carbon-estimation calculator from CPU/GPU hours, core counts, and region carbon intensity — a *model*, not a measurement, with no notion of verified output, retries, or data movement. It cannot distinguish algorithms; it scales a runtime number.
- **SAT Competition methodology** produces solver rankings by wall-clock and solved-instance counts on fixed hardware with per-run time limits; energy is not measured, and failed runs are counted only as failures, not as energy expenditure.
- Hardware-level energy measurement is mature: Intel RAPL, AMD APML, NVIDIA power counters, and tools like Scaphandre/CodeCarbon wrap these. Prior academic work measures energy of specific kernels (sparse linear algebra, sorting) but not a cross-algorithm, verification-inclusive, joules-per-solution metric.
- What's missing: no agreed accounting standard, no public dataset comparing ≥2 algorithms × ≥2 platforms under the full metric, no variance analysis showing cross-platform conclusions are stable.

## What would count as solving it?

1. **A specification**: a written accounting standard defining the metric (which phases counted, how checker energy is attributed, how failures amortize, measurement API per platform, uncertainty reporting).
2. **A reference implementation**: open-source harness that measures RAPL/GPU counters around every phase and runs an independent verifier.
3. **A published dataset**: at least two algorithm families (e.g., CDCL vs. local-search SAT solvers, or exact vs. heuristic TSP) on at least two hardware platforms, with per-instance joules-per-verified-solution, failed runs included, plus run-to-run variance.
4. **A validation finding**: evidence on whether the metric changes algorithm rankings relative to wall-clock, with confidence intervals. A referee accepts: reproducible (third party re-runs the harness and matches within stated tolerance) and complete (no unmeasured phases).

## How to start

1. Read the Green Algorithms paper to fix vocabulary for carbon conversion; note explicitly where its model is too coarse (no retries, no I/O, no verification) — this defines your delta.
2. Read the SAT Competition 2025 solver/benchmark description (verify availability; fall back to SAT Competition 2024) to inherit a standard instance set and certificate-checking practice (DRAT proofs for CDCL, locally verified assignments for local search).
3. Build the harness on one machine using RAPL (Linux perf or powercap sysfs) plus `nvidia-smi` polling if GPUs are involved; measure compilation, a fixed warm-up, each run (success or timeout), proof checking, and file I/O.
4. Main obstacle: attribution and variance — RAPL accuracy across sockets, DRAT checking dominating solver energy, and deciding whether checker energy counts in the numerator for failed attempts. Decide and document; the accounting rules *are* the contribution.

## Key references

- Lannelongue, L., Grealey, J., Inouye, M. (2021), "Green Algorithms: Quantifying the Carbon Footprint of Computation," *Advanced Science* 8(12), 2100707. DOI: 10.1002/advs.202100707 — a calculator estimating carbon from runtime/hardware/region data; the baseline model this benchmark must go beyond.
- SAT Competition 2025 methodology (proceedings/repository; verify exact citation) — defines standardized instance suites, run protocols, and (for CDCL) DRAT proof checking, which supplies the "verified solution" machinery to instrument.

## Warnings

- RAPL measures package energy domains, not wall/cooling; cross-platform joules compare compute energy only — state this limitation.
- Certificate checking can cost more energy than solving (DRAT checking is often slower than solving); the metric may inadvertently benchmark the verifier. Consider reporting solve-phase and check-phase energy separately.
- Beware double-counting: if a solver runs k attempts and one succeeds, energy per verified solution includes the k−1 failures — this is the intended amortization but must be explicit, or rankings are meaningless.
- The census is a "proposed" target from a speculative source; there is no existing canonical formulation to match, so precision of your standard is the deliverable, not agreement with prior definitions.

---
*`ALGO-000247` · flagged for review*

## Acceptance guidance

1. **A specification**: a written accounting standard defining the metric (which phases counted, how checker energy is attributed, how failures amortize, measurement API per platform, uncertainty reporting).
2. **A reference implementation**: open-source harness that measures RAPL/GPU counters around every phase and runs an independent verifier.
3. **A published dataset**: at least two algorithm families (e.g., CDCL vs. local-search SAT solvers, or exact vs. heuristic TSP) on at least two hardware platforms, with per-instance joules-per-verified-solution, failed runs included, plus run-to-run variance.
4. **A validation finding**: evidence on whether the metric changes algorithm rankings relative to wall-clock, with confidence intervals. A referee accepts: reproducible (third party re-runs the harness and matches within stated tolerance) and complete (no unmeasured phases).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/542756ac-d887-43f4-954e-be619ce429c4?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: ALGO-000247.

- [Public source catalog](https://drewstone.github.io/problems/#/p/ALGO-000247)
