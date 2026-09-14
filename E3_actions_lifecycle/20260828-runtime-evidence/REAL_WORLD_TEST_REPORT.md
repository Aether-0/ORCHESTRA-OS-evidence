# Real-World Validation and Fix Report

Campaign: `20260828-132302-redshadow-fix-implementation`  
Host: `redshadow`  
Kernel: `7.0.12+kali-amd64`  
Hardware: Intel i5-10310U, 4 cores/8 logical CPUs, one NUMA node, ~31 GiB RAM
Source commit at audit: `94664aeb001d8b3552245aedfd78254fbb5f13b8`; the
working tree was dirty, so the uncommitted source/documentation changes are
identified by the hashes in `audit/final_worktree_and_hashes_post_docs.txt`.

## Result at a glance

The two confirmed defects are fixed. The target-matched BPF/bridge/loader
build, userspace gates, verifier attach, ownership gate, action matrix, RT
admission boundary, integrity probes, same-node NUMA migration, stress tiers,
and 30-minute soak all completed within their declared scopes.

The evidence supports bounded `EXPERIMENTALLY_VALIDATED` claims for the tested
prototype paths. It does not support a general performance win, hard-real-time
guarantee, kernel cryptographic authentication, multi-node NUMA behavior, or
distributed scheduling.

## Performance

The repository `benchmark_suite.sh` was run with the same fixed-work helper,
worker counts, CPU set, and collection protocol for three repetitions in each
of CFS and ORCHESTRA. The `scx_simple` comparison was blocked because it is not
installed. ORCHESTRA exact-TID ownership was `yes` for every row.

CPU fixed-work mean completion times (milliseconds; n=3) were:

| Workers | CFS mean ± SD | ORCHESTRA mean ± SD | ORCHESTRA/CFS |
|---:|---:|---:|---:|
| 1 | 208.67 ± 1.53 | 330.00 ± 32.05 | 1.58x |
| 2 | 208.00 ± 1.73 | 418.67 ± 57.05 | 2.01x |
| 4 | 225.67 ± 30.60 | 461.67 ± 16.17 | 2.05x |
| 8 | 295.00 ± 75.03 | 511.00 ± 31.95 | 1.73x |

Mixed CPU/I/O completion means (n=3) were 1,632.33/1,398.00 ms (CFS/
ORCHESTRA) at one worker, 1,958.67/1,843.00 ms at two, and 1,955.00/
1,759.67 ms at four. These are exploratory completion-time observations, not
causal scheduler superiority: the fixed-work runner recorded positive
accepted/dispatched/running ownership but workers could exit before the
effective counter snapshot.

Raw runs are under `performance/benchmark-r{2,3,4}` and
`performance/benchmark-m{1,2,3}`. The earlier timed-spin harness run is kept
as `performance/benchmark-r1` and is excluded from the fixed-work aggregate.

## Stress and long duration

- 5-second full-CPU CFS and ORCHESTRA smoke tiers passed CPU, I/O, mixed, and
  health phases. The memory row was blocked by the missing `stress` command.
- 30-second CFS and ORCHESTRA tiers on CPUs 0–1 passed CPU, I/O, mixed, exact
  ownership (ORCHESTRA), and health phases. Memory remained blocked for the
  same reason.
- The ORCHESTRA long-run tier completed three 600-second phases (CPU, I/O,
  mixed) on CPU 0: 1,800 seconds of scheduler-enabled workload time over a
  30-minute wall-clock campaign. All three phases had zero errors, exact
  ownership, and no new panic/BUG/stall/hung-task/RCU lines. Maximum sampled
  package temperature was 87 °C; no thermal trip event was reported by the
  suite's watchdog. Unload returned sched_ext to `disabled`.

Evidence: `stress/cfs-5`, `stress/orchestra-5`, `stress/cfs-30`,
`stress/orchestra-30`, and `longrun/orchestra-600`.

## RT boundary

With ORCHESTRA attached, `chrt` confirmed `SCHED_FIFO`, `SCHED_RR`, and
`SCHED_DEADLINE` policies. The bridge rejected adaptive opt-in and publication
for each RT task (exit code 6), while a normal task was admitted, owned,
completed, and unloaded cleanly. A separate short contention probe kept FIFO
and RR tasks on CPU 0 while an ORCHESTRA-owned normal task ran on CPU 1; both
normal tasks reached accepted/dispatched/running/effective counts. The
DEADLINE contention launch returned `EPERM` in this environment, which is
preserved separately; the policy-admission result still passed. This validates
the implemented admission guard and bounded coexistence observation. It is not
a hard-real-time, starvation, priority-inversion, or deadline-latency guarantee.

Evidence: `rt/policy_matrix.txt`, `rt/normal_status_last.txt`,
`rt/lifecycle.txt`, `rt/contention-v2/contention_matrix.txt`,
`rt/contention-v2/deadline.rt.stderr`, and the stopped-process retries under
`rt/contention-v3` and `rt/contention-v4`.

## Integrity and security

The maintained userspace integration suite exercised valid publication,
generation-stamped concurrent reads, and explicit tamper injection. The
tamper run recorded `hmac=176` rejected frames; controller-tamper recorded
`hmac=35`; valid readers completed without HMAC failures. The suite returned
`rc=0` and preserved all CSV/log artifacts under `integrity/userspace-r1`.

The documented signal-publication microbenchmark added 12/12 validated runs
(two repetitions each for legacy and generation-stamped transports at 1, 2,
and 4 readers). Across those runs, 12,000 publications and 27,161 verified
reads completed with zero HMAC failures, invalid reads, stale frames, or
excluded runs. Its protocol is explicitly a local userspace measurement, not
kernel authentication evidence.

The target-kernel bridge additionally accepted a valid signal, rejected a
duplicate sequence (`rc=10`), rejected an out-of-range fixed-point field, and
produced kernel stale-signal telemetry (`signal_stale=694`) for an expired
required-signal directive. This validates local schema, generation, and
freshness fail-closed behavior.

The kernel signal map is explicitly local-trust IPC and has no HMAC verifier;
therefore the cryptographic claim is userspace-only. No kernel cryptographic
authenticity or production key-management claim is made.

Evidence: `integrity/integration_r1.stdout`, `integrity/userspace-r1/tamper.log`,
`integrity/userspace-r1/controller-tamper.log`,
`integrity/microbenchmark-v1/benchmark_result.json`, and
`integrity/kernel_matrix.txt`.

## Focused security rerun and CFS comparison

The existing `tests/security/run.sh` completed with `sched_ext=disabled`
(CFS baseline), including policy-loader mutation, ABI/state sanitizer, install
path, and source-security checks. With the target ORCHESTRA object attached,
the loader entered `enabled` and detached cleanly back to `disabled`. The
direct policy-loader, mutation, ABI-sanitizer, and source-security components
all passed while attached. The unchanged full wrapper is recorded as a
preserved harness failure only at the observer-only install/uninstall case:
its uninstall path correctly tries to disable the active scheduler, but that
temporary package has no trusted kernel loader and refuses with `untrusted
loader`. This does not indicate a cryptographic failure.

The rerun's integration baseline recorded `hmac=0`; valid ORCHESTRA recorded
`hmac=0`; explicit tamper and controller-tamper cases were rejected with
`hmac=201` and `hmac=42`, respectively. The documented userspace publication
microbenchmark completed 12/12 validated runs with zero HMAC, invalid-frame,
or stale-frame failures. A detailed posture comparison, including why CFS has
no corresponding custom signal surface and why ORCHESTRA is not intrinsically
safer, is preserved in
`security/CFS_ORCHESTRA_SECURITY_COMPARISON.md`.

## NUMA and network/distributed scope

The host reports exactly one NUMA node (`node0`, CPUs 0–7); `numactl` is not
installed. A same-node migration probe from an affinity set containing CPUs
0–1 to CPU 1 completed with `mig_acc=171`, `mig_disp=171`, and `actual_cpu=1`.
This is bounded single-node NUMA/core evidence only. Cross-node placement,
remote-memory cost, and multi-node scaling are not testable on this host.

A standard loopback netcat transfer moved 32 MiB under CFS and 64 MiB with the
ORCHESTRA loader lifecycle. The first readiness-probe harness failure is
preserved; the corrected transfer passed byte-for-byte and unload checks. The
fast network tasks did not retain task-state entries long enough for a direct
ORCHESTRA ownership snapshot, so the network timing is not attributed to
ORCHESTRA scheduling. The repository contains no distributed/cluster backend,
remote ordering, node identity, or partition protocol; distributed scheduling
remains `NOT_IMPLEMENTED`.

Evidence: `numa/topology.txt`, `numa/migration_status.txt`,
`numa/migration_matrix.txt`, `distributed/loopback_retry_matrix.txt`,
`distributed/loopback_retry_orchestra_matrix.txt`, and
`distributed/loopback_blocked_matrix.txt`.

## Claim ledger

| Area | Claim class | Scope |
|---|---|---|
| Performance | `EXPERIMENTALLY_VALIDATED` exploratory | matched CPU/mixed completion measurements; no superiority claim |
| Stress | `EXPERIMENTALLY_VALIDATED` bounded | CPU/I/O/mixed, ownership-gated ORCHESTRA, 5–600 s phases |
| RT | `EXPERIMENTALLY_VALIDATED` bounded | FIFO/RR contention plus FIFO/RR/DEADLINE admission guard; no hard-RT claim |
| Cryptographic integrity | `EXPERIMENTALLY_VALIDATED` userspace | HMAC/tamper precursor; kernel map is not HMAC-authenticated |
| NUMA | `EXPERIMENTALLY_VALIDATED` single-node | same-node domain/migration only |
| Distributed | `NOT_IMPLEMENTED` | no backend or protocol; loopback is not distributed evidence |
| Long duration | `EXPERIMENTALLY_VALIDATED` bounded | one CPU, 30-minute three-phase soak |

Missing tools and topology limits are recorded rather than silently replaced:
`stress`, `scx_simple`, `numactl`, `fio`, `iperf3`, `perf`, and hardware with
multiple NUMA nodes or multiple schedulers were unavailable.

## Evidence-package integrity

The final audit parsed `RESULTS.json`, checked every campaign CSV for a
consistent schema, passed `git diff --check`, and confirmed sched_ext was
`disabled` with no ORCHESTRA bpffs pins. The original runtime harness failure
and RT path-safety failure, together with all retry outputs, remain preserved;
no failed result was overwritten. The
campaign is indexed by `SHA256SUMS` and `inventory.json`; the manifest records
the exact file and byte totals. A current checksum recheck passed after the
archive was finalized. Final `bpftool` program/map/link inventories show no
ORCHESTRA objects or pins; unrelated pre-existing BPF state remains untouched.
The final inventory evidence is under `audit/final_sched_ext_state_current.txt`,
`audit/final_bpffs_current.txt`, and `audit/final_bpftool_*_current.txt`.
