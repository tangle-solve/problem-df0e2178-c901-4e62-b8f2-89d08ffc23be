<!-- solve-problem:df0e2178-c901-4e62-b8f2-89d08ffc23be -->
# Hardware-portable parallel gas accounting

A [Tangle Solve problem](https://solve.tangle.tools/problems/df0e2178-c901-4e62-b8f2-89d08ffc23be).

## What is the problem?

In blockchain systems with parallel transaction execution (e.g., blockchains using optimistic/multiversion concurrency control, like Fuel, Solana, Aptos-style executors), each transaction is charged a **gas fee** that is supposed to reflect the resources it consumes. Classical gas models are sequential: one unit of gas ≈ one unit of CPU work on a reference machine. With parallel execution, actual cost is governed by **shared hardware contention**: cache misses, memory bandwidth, I/O, branch mispredicts, thread scheduling, and interconnect effects. The same transaction charged N gas may cost wildly different wall-clock validation work depending on (i) which hardware the validator runs, and (ii) which other transactions run concurrently.

The problem: **construct a gas/cost model and fee schedule such that the total actual validation work (makespan, or weighted resource-area) on *any* supported hardware configuration and under *any* admissible scheduler is bounded by a function of the total gas charged**, with a provable (or empirically validated) approximation factor. Formally: define a set of hardware profiles H, a workload W of transactions with per-resource demand vectors, a scheduler class S, and a charging function g(·); establish a bound of the form

  Work(hw, sched, W) ≤ α · g(W) + β for all hw ∈ H, sched ∈ S,

and design g so the bound is tight up to constants while remaining computable without executing the transaction.

Current 2025-era models (Acilan et al. 2025) abstract cost via **weighted area / makespan** under simplified resource assumptions, ignoring hardware portability. The census flags the gap: no known model bounds *actual* validation work per charged gas across heterogeneous hardware, caches, storage, and scheduling policies.

## Why does it matter?

Without such a bound, a fee schedule calibrated on one machine underprices transactions that are pathologically expensive on cheaper or differently-architected validators — making heterogeneous (non-datacenter) validation economically unviable and exposing the network to denial-of-service via "cheap-gas, expensive-hardware" transactions. A solution would let home/staked validators on consumer hardware stay competitive, ground multiresource fee markets (EIP-4844-style multidimensional pricing) in real cost, and connect WCET/architecture-independent analysis theory to deployed fee mechanisms.

## What's already known?

- **Acilan et al. (2025)** design transaction fee markets for parallel-execution blockchains, using abstract cost models based on makespan/weighted-area under parallel execution with contention (read/write set conflicts). They assume a roughly uniform machine model; hardware portability is out of scope. This is the baseline abstract model the census says must be surpassed.
- **Multidimensional fee markets** (e.g., EIP-1559 extensions, multidimensional EIP-4844-style pricing) separate gas dimensions (compute, storage, bandwidth) but dimension demands are hand-tuned, not derived from hardware bounds.
- **Architecture-independent WCET** (AIWCET): the real-time-systems literature provides upper bounds on execution time using metrics like memory-bound iterations (cache-miss counts that are hardware-independent, per Berg's "observations on AIWCET" line of work). This gives tools for hardware-portable per-transaction demand estimation, but nothing composes it with *concurrent* scheduling and *fee markets*.
- Known shortfall: makespan bounds for parallel machines (Graham-style list scheduling, related-machine scheduling with speed augmentation) exist per-resource, but no one has combined them into a gas-accounting bound that holds across a hardware *distribution* under adversarial co-scheduling.

## What would count as solving it?

Any of the following, in increasing strength:
1. **Empirical:** a measurement framework plus a regression/audit showing a charged-gas-to-actual-work ratio bounded within factor α across a declared hardware set (e.g., 3+ distinct CPU/cache/storage configs) under realistic schedulers, with worst-case transaction classes identified.
2. **Constructive:** a charging function g over measurable, hardware-independent transaction features (working-set size, I/O ops, instruction mix, conflict footprint) such that the bound above holds for all hw in a modeled hardware class H and all online schedulers in S, with explicit α, β.
3. **Negative/lower bound:** a proof that no single-scalar gas schedule can achieve bounded ratio across a natural hardware class (forcing multidimensional pricing), which would itself resolve the design question.
4. **Matching:** upper and lower bounds within constant factors for a concrete hardware model (e.g., cache-hierarchy contention only).

## How to start

1. Read Acilan et al. (2025), especially their cost model section — identify precisely which assumptions (uniform machine, atomic contention model) fail under hardware heterogeneity. That's your delta.
2. Read the architecture-independent WCET literature (Berg's AIWCET work using instruction cache miss bounds) to get hardware-portable per-transaction demand metrics; these become your gas dimensions.
3. Model the problem as scheduling with related machines plus cache-contention interference: try extending Graham-style list-scheduling makespan bounds (speed augmentation) to a weighted-area objective where each transaction's "speed" depends on co-runners' cache footprints.
4. Main obstacle: cache contention is *non-monotone* in co-runner set (helpers/thrashers), breaking standard scheduling analysis; expect to either restrict the scheduler class or overapproximate interference pessimistically.

## Key references

- Acilan, C., et al. (2025). *Transaction Fee Market Design for Parallel Execution*. arXiv:2502.11964. Designs fee markets for blockchains with parallel transaction execution using makespan/contention-aware abstract cost models; the direct baseline whose simplifications this problem targets. https://arxiv.org/abs/2502.11964
- Berg, C. (2006). *Observations on the Integration of Instruction Cache Analysis into WCET Tools* (and follow-up AIWCET work). Develops architecture-independent WCET upper bounds via hardware-independent metrics (e.g., loop-wise memory-hierarchy considerations) — the closest toolbox for portable per-transaction cost. (verify exact paper; the AIWCET literature is the relevant body)
- Related background on multidimensional fee pricing: EIP-4844 (blob gas) and Vitalik Buterin's multidimensional EIP-1559 proposal posts — establish the deployed fee-market context separating resource dimensions. (verify)

## Warnings

- The census record is sparse and machine-generated ("fleet" source); treat the problem framing as the organizer's paraphrase, not a canonical published open problem — there is no referee-held exact statement.
- Don't conflate throughput of the whole block with per-transaction validation cost; the bound must be attributable to charged gas, so accounting/aggregation rules matter as much as the scheduling bound.
- Pure benchmarking is a trap: reviewers will demand a hardware *model* for which the bound provably holds, or at least a stated, falsifiable hardware class — unbounded "works on my machines" studies won't close this.
- Non-monotone cache interference means naive "sum of individual costs" charging provably fails; look for counterexamples early.

---
*`BCGT-000293` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:
1. **Empirical:** a measurement framework plus a regression/audit showing a charged-gas-to-actual-work ratio bounded within factor α across a declared hardware set (e.g., 3+ distinct CPU/cache/storage configs) under realistic schedulers, with worst-case transaction classes identified.
2. **Constructive:** a charging function g over measurable, hardware-independent transaction features (working-set size, I/O ops, instruction mix, conflict footprint) such that the bound above holds for all hw in a modeled hardware class H and all online schedulers in S, with explicit α, β.
3. **Negative/lower bound:** a proof that no single-scalar gas schedule can achieve bounded ratio across a natural hardware class (forcing multidimensional pricing), which would itself resolve the design question.
4. **Matching:** upper and lower bounds within constant factors for a concrete hardware model (e.g., cache-hierarchy contention only).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/df0e2178-c901-4e62-b8f2-89d08ffc23be?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000293.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000293)
