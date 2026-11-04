# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements


| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->547.026<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->8.188<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->472782.0<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->43044.9<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4536<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1: 2778.402, 2: 2318.278, 4: 2492.769, 8: 3146.47. The fastest thread count was 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive implementation spends its time executing explicit, nested Python loops that trigger continuous interpreter overhead and scalar instruction dispatches on the CPU. The vectorized version delegates the entire workload to optimized low-level C libraries (NumPy/BLAS), executing dense matrix multiplications in parallel across hardware SIMD registers and minimizing memory roundtrips.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
Parallelism stopped helping at 2 threads (2318.278 ms), after which performance degraded at 4 threads (2492.769 ms) and 8 threads (3146.47 ms). Colab's standard runtime provides limited physical cores. Beyond 2 threads, thread contention, context-switching overhead, and synchronization delays across shared caches exceed the marginal gains of dividing the remaining parallel workload.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Transformer training relies heavily on massive, parallel matrix multiplications (GEMM) across layers, which map perfectly to the thousands of arithmetic logic units (ALUs) inside a GPU. Conversely, sequential workloads with heavy branching logic or strong data dependencies—such as parsing text line-by-line or evaluating recursive algorithms—would see no speed-up and can even run slower on GPUs.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism is bounded by distinct bottlenecks across hardware scales. In intra-operation vectorization, work division hits limits on memory bandwidth, though it achieves a massive speedup (lowering naive time from 547.026 ms to 8.188 ms). In multi-threaded CPU execution, coordination overhead and hardware resource limits dominate: performance peaks at 2 threads (2318.278 ms) and degrades at 8 threads (3146.47 ms) due to thread contention. Finally, training scale is bounded by hardware architecture; massive parallel SIMD execution on the GPU drops the training step duration from 472782.0 ms on CPU to 13044.9 ms. Each layer shifts the limit from algorithmic inefficiency to thread synchronization costs, and ultimately to hardware compute density.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would measure training throughput scaling across varying batch sizes on the GPU. A plateau in GPU scaling under larger batch dimensions would prove that memory bandwidth limitations eventually supersede raw compute capacity, changing my view on where hardware bottlenecks truly originate.
<!--/answer-->

----










