# ORCHESTRA-OS Final Research, Product, and Capability Report

**Report date:** 2 September 2026

**Repository:** ORCHESTRA-OS

**Revision used by the principal validation archive:** `94664aeb001d8b3552245aedfd78254fbb5f13b8`

**Validated host:** ThinkPad T14 Gen 1, Intel Core i5-10310U, 4 cores/8 logical CPUs, one NUMA node, Kali Linux, kernel `7.0.12+kali-amd64`

**Product class:** research-grade release candidate; not deployment-ready

**Overall evidence class:** `USERSPACE_VALIDATED`, `KERNEL_PROTOTYPED`, and bounded `EXPERIMENTALLY_VALIDATED`

## 1. Executive conclusion

ORCHESTRA-OS successfully converted important parts of the professor's
simulation research into a real, opt-in Linux `sched_ext` prototype. The work
demonstrated an end-to-end control path from structured userspace records,
through a bridge and BPF maps, into a kernel scheduler that can own selected
tasks, apply the five canonical actions, expose requested-versus-effective
telemetry, reject invalid or stale inputs, and unload cleanly.

The most important conclusion is also the most honest one:

> ORCHESTRA-OS is currently stronger as an observable and bounded scheduler-
> control research platform than as a faster replacement for Linux CFS/EEVDF.

The strongest application result is controlled foreground protection. In 20
counterbalanced ImageMagick-foreground/bzip2-background pairs, foreground
completion improved in all 20 ORCHESTRA phases, with a mean paired reduction
of 74.2405%. That benefit was achieved while mean background CPU service fell
97.6851%. This is strong evidence that the scheduler can enforce a service
policy; it is not evidence of free total-system acceleration.

The general-purpose performance result is mixed. In three-repetition fixed-
work trials, ORCHESTRA took 1.58x to 2.05x as long as CFS for pure CPU work,
depending on worker count. The small mixed CPU/I/O sample was directionally
faster, with ratios from 0.86x to 0.94x at one, two, and four workers. Those
mixed results are useful hypotheses, but the sample is too small and the
workload too synthetic for a broad product claim.

The project team's strongest demonstrated abilities are:

1. translating research concepts into a working Linux/eBPF prototype;
2. designing observable, fail-closed control paths;
3. constructing reproducible tests that separate requests from effective
   scheduler behavior;
4. diagnosing subtle systems failures across userspace, BPF, scheduler state,
   process identity, timing, and workload behavior; and
5. communicating negative as well as positive results with traceable evidence.

These capabilities fit systems research engineering, Linux/eBPF prototyping,
performance and validation engineering, observability/reliability engineering,
and research software engineering better than roles centered on general ML,
hard real-time guarantees, distributed production systems, or proven HPC
optimization.

## 2. Evidence and claim discipline

This report uses the repository's strongest available evidence and keeps four
boundaries explicit:

- The professor's paper is a pre-kernel simulation study. Its numerical results
  are `SIMULATED`, not hardware results.
- A BPF build is not an attach; an attach is not proof that a workload was
  owned by `sched_ext`; ownership is not proof that an action was effective.
- Real-machine results apply only to the recorded host, kernel, artifact,
  workload, affinity, duration, and repetition protocol.
- The inspected working tree was dirty. Results are attributed to frozen
  archives and recorded hashes, not automatically to later uncommitted changes.

The formal 1 September evidence package normalizes 47 checklist rows to 38
`PASS`, two preserved `FAIL`, and seven `BLOCKED`. The failures were retained
as harness/path-attempt evidence; they were not hidden after corrected retries.
The package records `sched_ext=disabled` at campaign end.

## 3. What the professor's research established

The 6 July 2026 paper defined a predictive, cryptographically protected,
hierarchical, signal-coordinated scheduling architecture and used seven rounds
of simulation to discover design flaws before kernel implementation.

The paper's strongest contribution was not merely its final simulated score.
It showed a scientific method: create an observable model, reproduce a failure,
locate the earliest causal mismatch, apply a theoretically justified change,
and measure again.

### 3.1 Main simulation findings

| Discovery | Evidence in the paper | Meaning for implementation |
|---|---|---|
| Temporal blind spot | The original S1/S2/S3 metric allowed synchronized mass switching to look perfect | Add S4 so temporal instability and herd behavior affect Q |
| Aggregation artifact | A raw product shrank when new healthy factors were added | Use a geometric mean for a comparable multi-factor Q |
| Wrong controller target | A PID actuator saturated while the deficient submetrics were elsewhere | Map each actuator to the submetric it can causally influence |
| Coupled-loop instability risk | Learning and control modified one another | Use explicit two-timescale separation |
| Reward/directive contradiction | Reward encouraged RUN where the directive required MIGRATE; correcting it raised S2 from 0.57 to 0.79 | Audit objective functions together, not independently |
| State-bin mismatch | One discrete state contained states with different correct actions; realignment raised S2 to 0.90 | Align representations with policy decision boundaries |
| Exploration ceiling | Persistent exploration limited achievable compliance | Treat exploration scheduling as a structural design choice |
| Predictor identifiability failure | Online adaptive Kalman MSE was about seven times worse than simple fixed alternatives | Prefer offline calibration when noise parameters are not identifiable |
| Missing causal credit | Individual agents lacked feedback for their contribution to population behavior | Blend difference rewards with a clear local objective |
| Need for simulation gate | Seven failures were visible in simulation before expensive kernel work | Keep simulation between specification and kernel implementation |

The final simulation reported Q=0.868 versus a structurally advantaged reactive
baseline at Q=0.861, and 100% rejection of injected tamper events in that
simulation. The paper itself limits these results to a synthetic trace family,
a single modeled tier, and mostly seed-42 round progression, with multi-seed
sensitivity only for the final configuration. It does not prove kernel speed,
kernel cryptographic verification, NUMA, or cluster behavior.

## 4. What was built from the research

The implementation preserved the five canonical actions exactly: `RUN`,
`YIELD`, `MIGRATE`, `THROTTLE`, and `SLEEP/DEFER`.

| Research component | Product realization | Strongest current evidence | Boundary |
|---|---|---|---|
| State and signal acquisition contract | Versioned state/signal records with schema, generation, freshness, and identity gates | Userspace contract tests and bounded kernel-local records | Broad real sensor acquisition is incomplete |
| Prediction | Confidence, generation, expiry, and observed-state fallback fields | `USERSPACE_VALIDATED` contract | Hardware prediction accuracy/convergence untested |
| Signal Bus | Userspace frames, bridge maps, two-bank/generation publication | Userspace integrity plus kernel-local transport | No HMAC verifier inside BPF; not end-to-end kernel cryptography |
| Adaptive response | Canonical decision pipeline and policy bank | `KERNEL_PROTOTYPED` | Broad learned-policy effectiveness unproven |
| Five actions | Bounded BPF backends with fallbacks | All five observed in the action matrix | Not every workload shape, topology, or fairness condition |
| Hybrid Safety Layer | Admission guard for protected scheduling classes | Bounded FIFO/RR/DEADLINE refusal and FIFO/RR coexistence | No hard-real-time latency guarantee; DEADLINE contention blocked |
| S1/S2/S3/S4/Q | Fixed-point coordination windows and geometric-mean Q | Limited owned-host telemetry | No broad population/statistical validation |
| Feedback controller | Deficit classification, bounded actuators, hysteresis, rate limits, saturation, rollback/recovery states | Userspace tests and limited live transitions | Full actuator causality and rollback effectiveness unproven |
| Hierarchy | CPU/NUMA/global ABI slots | Single-node observations only | Cross-node NUMA not tested |
| Distributed tier | No backend or remote protocol | `NOT_IMPLEMENTED` | Loopback transfer is not distributed scheduling |

This is a meaningful research prototype. It is not the complete architecture
described by all work packages, and planned work-package deliverables are not
treated as completed features.

### 4.1 Development journey and tangible output

The repository history reachable from the inspected HEAD contains 85 commits
from the first userspace prototype on 5 August 2026 through the recorded runtime
hardening revision on 25 August 2026. The progression is visible in the history:

- userspace research prototype, controller state machine, policy lifecycle, and
  metric/schema evolution;
- minimal `sched_ext` scheduler and exact-kernel bring-up;
- userspace-to-BPF bridge, lifetime identity, and five-action execution;
- verifier, attach, ownership, runtime, stress, and benchmark campaigns;
- loader recovery, ABI/schema validation, security boundaries, and packaging;
- 12 architecture decision records; and
- final evidence archives containing raw measurements, machine state, hashes,
  findings, and reproducible report calculations.

The speed of this progression is notable, but commit count and speed are not
quality measures by themselves. The stronger evidence is that later campaigns
revisited early claims, added ownership/effectiveness gates, retained failures,
and narrowed conclusions when the measurements did not support them.

## 5. What real-machine testing proved

### 5.1 Kernel lifecycle and ownership

The target-matched BPF object, bridge, and loader were built for the recorded
Kali 7.0.12 kernel and BTF. The scheduler passed verifier/attach, entered the
enabled state, owned exact target TIDs under partial switching, and returned to
`sched_ext=disabled` after tested phases. Cleanup was loader-scoped and avoided
broad deletion of unrelated BPF state.

This is one of the project's strongest results because it closes a common
evidence gap: a command that says “loaded” is insufficient. The campaign also
checked target identity, ownership counters, action generation, dispatch,
running, effectiveness, fallback, and final lifecycle state.

### 5.2 Canonical actions

The 81-row runtime matrix passed all 81 assertions. Examples include:

- `RUN`: specialized dispatch delta 31, running delta 34, effective delta 34;
- `YIELD`: 45 specialized dispatches, 44 effective observations, and peer
  progress;
- valid `MIGRATE`: 28 accepted, 28 dispatched, 27 effective target-CPU runs,
  with requested, dispatched, and actual CPU all equal to CPU 1;
- invalid `MIGRATE`: 44 bad-CPU fallbacks, zero MIGRATE acceptance/dispatch,
  and effective fallback to RUN;
- `SLEEP`: one acceptance, defer, release, and subsequent forward progress;
- `THROTTLE`: repeated immediate, post-RUN, generation-rollover, republished,
  and two-CPU scenarios with observed deferrals, releases, and effective work.

These numbers prove bounded behavior under the specific action probes. They do
not prove universal fairness, all timing semantics, or production safety.

### 5.3 Signal and integrity behavior

The userspace signal-publication microbenchmark completed 12/12 runs, 12,000
successful publications, and 27,161 verified reads with zero invalid, HMAC, or
stale failures. Separate tamper cases were rejected. Kernel-local checks
rejected duplicate sequence, stale signal, and out-of-range fields.

The important limitation is architectural: the BPF-side map is a local-trust
IPC contract and does not perform HMAC verification. Therefore the result is
strong userspace integrity validation and bounded kernel freshness/validation,
not cryptographic end-to-end kernel signal authentication.

### 5.4 Stress, recovery, and real-time boundary

Owned CPU, I/O, and mixed phases passed at 600 seconds each with zero recorded
errors or warnings, followed by clean unload. The memory phase was blocked
because exact child ownership was not proven. FIFO and RR contention probes
showed the RT task outside the adaptive path while a normal ORCHESTRA task made
progress. DEADLINE contention was blocked by `EPERM`.

This supports bounded coexistence and recovery. It does not establish a hard
real-time guarantee, production soak, hotplug safety, starvation freedom, or
priority-inversion freedom.

## 6. Performance findings

### 6.1 General fixed-work comparison

Three repetitions per configuration produced the following means:

| Workload | Workers | CFS mean | ORCHESTRA mean | ORCHESTRA/CFS | Interpretation |
|---|---:|---:|---:|---:|---|
| CPU | 1 | 208.67 ms | 330.00 ms | 1.58x | Slower |
| CPU | 2 | 208.00 ms | 418.67 ms | 2.01x | Slower |
| CPU | 4 | 225.67 ms | 461.67 ms | 2.05x | Slower |
| CPU | 8 | 295.00 ms | 511.00 ms | 1.73x | Slower |
| Mixed CPU/I/O | 1 | 1632.33 ms | 1398.00 ms | 0.86x | Directionally faster |
| Mixed CPU/I/O | 2 | 1958.67 ms | 1843.00 ms | 0.94x | Directionally faster |
| Mixed CPU/I/O | 4 | 1955.00 ms | 1759.67 ms | 0.90x | Directionally faster |

The pure-CPU result is a clear weakness for the tested artifact and protocol.
The mixed result is promising but still exploratory because N=3 is small and
the mixed workload is synthetic. A target-matched `scx_simple` comparison was
blocked because the tool was unavailable.

### 6.2 Strongest application experiment: foreground protection

The held-out photo/archive protocol used ImageMagick foreground work and eight
bzip2 background workers. Across 20 counterbalanced pairs:

- all 40 phase rows exited zero and verified output;
- all 20 ORCHESTRA phases proved ownership and effective THROTTLE;
- all 40 phases ended with the scheduler disabled;
- CFS foreground mean was 7,987.8 ms;
- ORCHESTRA foreground mean was 2,036.6 ms;
- mean paired foreground reduction was 74.2405%;
- the descriptive paired 95% t interval was 72.7217% to 75.7592% reduction;
- ORCHESTRA won 20/20 pairs; and
- mean background CPU ticks fell from 4,872.85 to 112.80, a 97.6851%
  reduction.

The correct interpretation is:

> ORCHESTRA successfully enforced foreground priority by sharply reducing
> service to known, deferrable background tasks.

The experiment did not measure background completion or total makespan and did
not compare against a matched CFS cgroup/CPU-quota policy. Consequently, it
does not prove superior global efficiency. It proves explicit service
reallocation and foreground protection.

### 6.3 Log/archive experiment

A five-pair jq-foreground/gzip-background experiment showed a 60.4678% mean
paired foreground reduction and 5/5 wins, with ownership and output gates.
However, CFS phases reached 96–97°C against a reported 100°C critical point.
This result is `EXPLORATORY_THERMALLY_CONFOUNDED`, not a headline proof.

## 7. What we learned while turning the research into a product

### 7.1 Correctness needs a chain of evidence

The project learned to distinguish build, verifier acceptance, attach,
ownership, request, acceptance, dispatch, effectiveness, fallback, and unload.
This evidence chain is more valuable than a single “scheduler loaded” result
and is reusable in other eBPF and kernel projects.

### 7.2 Exact target matching is a product constraint

`sched_ext` and BPF behavior depend on the target kernel/API family, headers,
BTF, toolchain, and loader. Earlier preparation could build userspace tools but
could not produce a defensible target object when the available source tree did
not match the running kernel. The later campaign succeeded by recording an
exact target build manifest and hashes. Kernel compatibility must therefore be
managed as a release matrix, not assumed from a successful build elsewhere.

### 7.3 Observability is itself a product strength

Requested-versus-effective action telemetry exposed cases where a counter
increase did not mean the intended physical effect occurred. It also enabled
the team to diagnose stale generation, invalid migration, ownership, deferred
release, controller state, and cleanup. This is a strong transferable skill:
building systems that explain their own behavior.

### 7.4 Service policy and speed are different objectives

The foreground experiment looked like a large speedup until background CPU
service was measured. The service reduction explained the result. This is a
major scientific and engineering finding: compare total policy outcomes, not
only the favored task's elapsed time.

### 7.5 Process identity is not enough for all applications

Exact-TID admission works well for single-threaded workers, but some nominally
single-threaded tools expose multiple active TIDs. Controlling only a leader
cannot prove control over the complete application. A product aimed at general
applications needs a defensible process-group/cgroup/thread-group ownership
model and matching telemetry.

### 7.6 Transition behavior matters

The project reproduced a RUN-to-THROTTLE accounting bottleneck in which a
generation transition could bypass state synchronization and suppress expected
deferral behavior. A controlled diagnostic isolated the conditions under which
deferrals reappeared. This demonstrates strong causal diagnosis, while also
showing that transition matrices need broader validation.

### 7.7 Negative results improved the design

The work retained predictor failure, pure-CPU slowdown, thermally confounded
results, missing memory ownership, unavailable `scx_simple`, single-node NUMA,
and absent distributed scheduling. This improves research credibility and
prevents the product story from becoming stronger than the evidence.

## 8. Strength assessment

Scores below assess demonstrated project capability, not personal intelligence
or long-term potential. “Strong” means the repository contains direct,
repeatable evidence; “developing” means important proof is missing.

| Rank | Capability | Assessment | Evidence |
|---:|---|---|---|
| 1 | Evidence-led systems validation and diagnosis | **Very strong** | Ownership gates, 81/81 action assertions, preserved failures, protocol boundaries, root-cause reports |
| 2 | Linux/eBPF/sched_ext prototyping | **Strong** | Target-matched build, verifier, attach, exact-TID ownership, five bounded action paths, clean unload |
| 3 | Research-to-code translation | **Strong** | S4, geometric-mean Q, signal generation/freshness, bounded controller and policy concepts mapped into implementation contracts |
| 4 | Observability and safety-boundary design | **Strong** | Requested-to-effective telemetry, fallbacks, identity/admission gates, loader-scoped lifecycle |
| 5 | Reproducibility and technical communication | **Strong** | ADRs, work-package mapping, manifests, raw CSV/JSON, hashes, negative-result retention, auditable calculations |
| 6 | Controlled foreground service policy | **Strong within one protocol** | 20/20 paired wins with ownership, correctness, action, and unload gates |
| 7 | Userspace integrity and schema engineering | **Strong** | 12/12 publication runs, 12,000 publications, 27,161 verified reads, tamper/stale validation |
| 8 | General scheduler performance optimization | **Weak/developing** | Pure CPU was 1.58x–2.05x slower; mixed advantage remains exploratory |
| 9 | Hardware predictor validation and adaptive ML | **Developing** | Contract exists; hardware accuracy/convergence and benefit are not established |
| 10 | Hard real-time, NUMA, distributed, production security | **Not yet demonstrated** | DEADLINE contention blocked, one NUMA node, no distributed backend, no kernel HMAC verifier |

## 9. Best-fit job families

### 9.1 Highest-confidence matches

| Job family | Fit | Why the project is relevant | Honest interview boundary |
|---|---|---|---|
| Systems Research Engineer / Research Software Engineer | **Excellent** | Converted a simulation architecture into a userspace and kernel prototype; designed experiments and retained negative results | Say “bounded prototype and validation,” not “production scheduler” |
| Linux Systems or eBPF Engineer | **Strong** | BPF CO-RE, maps, loader lifecycle, `sched_ext`, task ownership, BTF/kernel matching, verifier-aware testing | Target prototype/junior-to-mid research roles unless other production experience exists |
| Performance/Benchmarking Engineer | **Excellent** | Controlled baselines, ownership proof, repetitions, counterbalancing, context-switch and service analysis, confounder detection | Emphasize methodology; do not claim universal ORCHESTRA speedup |
| Validation / Systems Test Engineer | **Excellent** | Built functional, action, stress, fault, recovery, RT-boundary, integrity, and checklist evidence | Highlight exact evidence chains and preserved first failures |
| Observability / Reliability Engineer | **Strong** | Designed requested/accepted/dispatched/effective/fallback telemetry and clean lifecycle checks | Reliability was bounded to tested phases, not production uptime |
| Edge/Resource-Management Research Engineer | **Strong** | Opt-in service policy and mixed CPU/I/O hypothesis match constrained single-host worker pools | Field benefit needs representative workload confirmation |
| Build/Release Engineer for native systems | **Good** | Exact-kernel manifests, hashes, ABI/schema gates, packaging and reproducibility discipline | Cross-distro/architecture release matrix remains incomplete |

### 9.2 Possible matches after one focused project

- **Kernel scheduler engineer:** strengthen with upstream Linux scheduler code
  review, `sched_ext` API-family matrix work, perf tracing, fairness/tail-latency
  analysis, and a public patch-review history.
- **Secure systems engineer:** add threat-driven privileged testing, signed
  artifact provenance, fuzzing, dependency/SCA review, and kernel-side trust
  boundary analysis. The current userspace HMAC result alone is insufficient.
- **Applied control/ML systems engineer:** validate predictor accuracy and
  controller causality on hardware across multiple regimes and seeds.
- **SRE/platform engineer:** demonstrate deployment, rollback, monitoring,
  capacity, and incident response on a real service rather than only a
  scheduler test host.

### 9.3 Roles not currently supported by this project alone

- production hard-real-time or safety-critical scheduler engineer;
- distributed/cluster scheduler engineer based on implemented experience;
- NUMA/HPC performance specialist;
- general machine-learning engineer based on predictive accuracy;
- cryptographic implementation/security-assurance specialist; or
- senior production kernel maintainer.

These are evidence gaps, not permanent limitations. They identify what a next
portfolio project should prove.

## 10. Best kinds of work for this team

The evidence suggests the team performs best when the work has the following
shape:

1. a research paper, architecture, or difficult systems requirement must be
   converted into a bounded prototype;
2. behavior crosses userspace/kernel or multiple subsystem boundaries;
3. success requires instrumentation and causal diagnosis, not only feature
   implementation;
4. the environment is experimental and uncertainty must be documented;
5. safety, fallback, identity, lifecycle, and reproducibility matter; and
6. negative results are useful inputs to the next design decision.

Examples include eBPF policy engines, kernel observability tools, resource
governors, edge workload controllers, performance test frameworks, systems
research prototypes, fault/recovery harnesses, and reproducible infrastructure
for native software.

## 11. Weaknesses and how to close them

| Weak area | Why it matters | Best next evidence |
|---|---|---|
| Pure-CPU overhead/context switching | Prevents a general scheduler-performance claim | Use `perf`, tracepoints, BPF callback cost, migrations, cache misses, and matched `scx_simple` to localize overhead |
| No matched service-policy baseline | Foreground result may be reproducible with standard Linux controls | Compare ORCHESTRA with CFS+cgroup/CPU quota/weight under identical total-service constraints |
| Small mixed-work sample | Directional results may be noise or protocol-specific | At least 20 counterbalanced matched pairs on an idle host with confidence intervals and tail metrics |
| Exact-TID application coverage | Multi-threaded applications can escape or invalidate ownership claims | Establish and validate thread-group/cgroup admission and per-thread telemetry |
| Kernel crypto gap | Userspace HMAC does not protect the BPF-local trust boundary | Specify the kernel threat model, key lifecycle, verifier-compatible design, replay model, and approved tests |
| Predictor evidence gap | “Predictive scheduler” is stronger than what hardware results prove | Collect stable/increasing/decreasing/periodic/bursty traces; report horizon, error, confidence calibration, latency, and fallback |
| Controller causality gap | Parameter movement is not proof of successful control | Excite each submetric independently and measure actuator, response, saturation, oscillation, and recovery |
| RT/NUMA/hotplug gaps | Block deployment and architecture claims | Dedicated multi-node host, CPU online/offline matrix, bounded FIFO/RR/DEADLINE contention, starvation/priority-inversion study |
| Distributed tier absent | Paper's hierarchy is not fully realized | Build a separate, fault-aware protocol only after single-node semantics are stable |
| Production readiness | A bounded soak is not operations evidence | Multi-hour/day soak, crash recovery, upgrade/rollback, signed provenance, independent review, and supported-kernel matrix |

## 12. Recommended next experiment

The most valuable next experiment is not another broad demo. It is a controlled
service-policy comparison:

1. freeze one source revision and target artifact;
2. use an idle, thermally safe, authorized host;
3. compare CFS, CFS with matched cgroup/CPU service controls, `scx_simple`, and
   ORCHESTRA;
4. give every mode identical affinity, initial state, foreground and background
   work, and total service objective;
5. counterbalance order and collect at least 20 matched pairs;
6. prove exact ownership and output correctness for every accepted ORCHESTRA
   row;
7. record foreground latency, background progress/completion, total makespan,
   CPU time, context switches, migrations, utilization, temperature, and energy
   if available; and
8. retain all failed/excluded trials under a predeclared rule.

This experiment would answer the product question directly: whether ORCHESTRA
adds value beyond standard Linux resource controls, and at what total-system
cost.

## 13. Portfolio and interview positioning

### Defensible one-sentence description

> Built and validated a research-grade, opt-in Linux `sched_ext` control and
> observability prototype that maps predictive scheduling research into five
> bounded kernel actions with exact ownership, fallback, and lifecycle evidence.

### Defensible résumé bullets

- Translated a simulation-derived scheduling architecture into a Linux
  userspace/bridge/eBPF prototype with five canonical actions and explicit
  requested-to-effective telemetry.
- Built target-matched `sched_ext` validation covering verifier attach,
  exact-TID ownership, action effectiveness, invalid-action fallback, and clean
  unload; the final runtime matrix passed 81/81 bounded assertions.
- Designed an auditable evidence pipeline with versioned schemas, ADRs,
  manifests, hashes, raw CSV/JSON results, preserved failures, and explicit
  simulation/userspace/kernel claim classes.
- Demonstrated a controlled foreground-protection policy across 20/20 paired
  trials, reducing foreground completion time by 74.24% while identifying the
  associated 97.69% reduction in background CPU service.
- Diagnosed negative and confounded results, including 1.58x–2.05x pure-CPU
  slowdown, thermal interference, exact-TID coverage limits, and a
  RUN-to-THROTTLE transition/accounting bottleneck.

### Claims to avoid

- “ORCHESTRA is faster than Linux.”
- “The product is deployment-ready.”
- “The kernel signal path is cryptographically authenticated.”
- “Hard real-time safety is proven.”
- “NUMA or distributed scheduling is implemented and validated.”
- “The predictor improves real hardware scheduling.”

## 14. Final product judgment

ORCHESTRA-OS has achieved more than a paper reproduction and less than a
production scheduler.

What it has achieved is technically meaningful:

- a coherent research-to-code mapping;
- a target-specific working `sched_ext` prototype;
- exact ownership and action-effect evidence;
- unusually strong observability and claim discipline;
- bounded RT admission, freshness, integrity, stress, and recovery evidence;
- a reproducible foreground service-control result; and
- a clear catalogue of negative results and remaining work.

What it has not achieved is equally clear:

- a general performance advantage over CFS/EEVDF;
- a complete predictive hardware scheduler;
- kernel-side cryptographic verification;
- broad fairness, tail-latency, RT, hotplug, NUMA, or architecture validation;
- distributed scheduling; or
- deployment readiness.

The best current external description is:

> ORCHESTRA-OS is a research-grade, opt-in Linux scheduler control and
> observability platform. It has bounded real-machine validation of ownership,
> five scheduler actions, fallback, lifecycle, and foreground service
> protection, while general performance and production-readiness claims remain
> open.

The team's best professional story is therefore not “we built a scheduler that
beats Linux.” It is stronger and more credible:

> We took an ambitious operating-systems research architecture, found where its
> assumptions broke, translated its strongest ideas into a real Linux/eBPF
> prototype, proved what the kernel actually did, preserved the failures, and
> identified the exact experiments required for the next claim.

## 15. References and evidence map

### Primary research and specification

1. Geetha Ganesan and Panchabi Vaithiyanathan, *ORCHESTRA-OS: Simulation-Driven
   Architectural Discovery and Design Principles for Predictive, Hierarchical,
   Signal-Coordinated Process Scheduling*, 6 July 2026, repository file
   `orchestra os tdps final 6 july.pdf`, SHA-256
   `9af8f45412e2f7d8be89f108965372caa2c420f917c7b612c3b4a86ab626561d`.
2. [`WORK_PACKAGE_DESCRIPTION.md`](../../../WORK_PACKAGE_DESCRIPTION.md) —
   planned implementation, instrumentation, validation, scale, security, and
   deployment work.
3. [`docs/research/RESEARCH_TO_CODE.md`](../../../docs/research/RESEARCH_TO_CODE.md)
   — concept-to-implementation mapping and evidence boundaries.
4. [`FINAL_PRODUCT_STATUS.md`](../../../FINAL_PRODUCT_STATUS.md) — current
   product identity and release-candidate limitations.

### Principal validation package

5. [`20260901 validation README`](../20260901-orchestra-cfs-validation-report/README.md)
   — central conclusion and package contents.
6. [`PROVENANCE.md`](../20260901-orchestra-cfs-validation-report/PROVENANCE.md)
   — host, revision, primary hashes, evidence IDs, and negative-evidence policy.
7. [`FINAL_AUDIT.md`](../20260901-orchestra-cfs-validation-report/FINAL_AUDIT.md)
   — render, numerical, provenance, and checksum audit.
8. [`EVIDENCE_CALCULATIONS.md`](../20260901-orchestra-cfs-validation-report/EVIDENCE_CALCULATIONS.md)
   — formulas and recomputed headline values.
9. [`data/implementation_matrix.csv`](../20260901-orchestra-cfs-validation-report/data/implementation_matrix.csv)
   — component maturity and claim boundaries.
10. [`data/checklist_status.csv`](../20260901-orchestra-cfs-validation-report/data/checklist_status.csv)
    — 47-row PASS/FAIL/BLOCKED record.
11. [`data/runtime_action_matrix.csv`](../20260901-orchestra-cfs-validation-report/data/runtime_action_matrix.csv)
    — 81 bounded runtime assertions.
12. [`data/fixed_work_summary.csv`](../20260901-orchestra-cfs-validation-report/data/fixed_work_summary.csv)
    — repeated CFS/ORCHESTRA CPU and mixed fixed-work means.
13. [`data/photo_archival_statistics.txt`](../20260901-orchestra-cfs-validation-report/data/photo_archival_statistics.txt)
    — P20 foreground and background-service statistics.
14. [`data/longrun_results.csv`](../20260901-orchestra-cfs-validation-report/data/longrun_results.csv)
    — 600-second CPU/I/O/mixed phases and memory ownership block.

### Supporting synthesis and diagnosis

15. [`REAL_WORLD_FIT_REPORT.md`](../20260827-012330-redshadow-field-fit/REAL_WORLD_FIT_REPORT.md)
    — field-fit analysis, iteration sensitivity, and ownership boundaries.
16. [`STRONGEST_PART_AND_COMPARISON_GUIDE.md`](../20260827-101440-redshadow-strength-map/STRONGEST_PART_AND_COMPARISON_GUIDE.md)
    — strength ranking and comparison methodology.
17. [`FINDINGS.md`](../20260827-202638-redshadow-strength-confirmation/FINDINGS.md)
    — foreground-isolation result, service cost, thermal stop, exact-TID limit,
    and transition diagnosis.
18. [`docs/validation/FINAL_VALIDATION_REPORT.md`](../../../docs/validation/FINAL_VALIDATION_REPORT.md)
    — acceptance gates and bounded real-world addenda.

## 16. Reproducibility note

No new scheduler run was performed to create this synthesis. The numerical
claims were taken from the frozen evidence package whose final audit reports
successful checksum verification and recomputation. The current repository
working tree contains staged and untracked changes, so anyone extending this
report should first freeze a new revision, capture `git status`, rebuild the
target artifact, and repeat the relevant gates rather than assuming the 1
September measurements cover the current uncommitted state.
