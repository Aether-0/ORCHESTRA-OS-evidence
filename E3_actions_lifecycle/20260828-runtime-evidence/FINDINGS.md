# Findings

## F-001 — RESOLVED (HIGH)

RUN→THROTTLE generation mismatch prevented `.running` from correlating with
task telemetry. `sync_task_state_for_action()` now resets period/runtime only
on generation or action change and preserves runtime for same-generation
requeues. Immediate, after-RUN, rollover, republished-generation, and two-TID
runtime cases passed.

## F-005 — RESOLVED (MEDIUM)

The live compact `.enqueue` path bypasses `orchestra_execute_action()` and its
action-specific counters. `record_dispatched_legacy()` now maps compact-path
RUN/YIELD/MIGRATE dispatches correctly. YIELD and MIGRATE acceptance/dispatch
telemetry passed, including invalid-target fallback.

## Validation claims now supported

- Performance: three matched CPU and three matched mixed benchmark repetitions
  with positive ORCHESTRA ownership gates. Results are exploratory and show
  ORCHESTRA slower than CFS for fixed CPU work on this host.
- Stress: 5-second and 30-second CPU/I/O/mixed tiers passed; memory is blocked
  by the missing `stress` binary.
- Long duration: one-CPU ORCHESTRA CPU/I/O/mixed phases each ran 600 seconds,
  with exact ownership, zero errors, no health delta, and 87 °C maximum sample.
- RT: FIFO, RR, and DEADLINE tasks were refused by the adaptive bridge while a
  normal task coexisted and completed. This is an admission/coexistence claim,
  not a hard-real-time guarantee.
- A short simultaneous contention probe passed for FIFO and RR with an
  ORCHESTRA-owned normal task. The DEADLINE contention launch returned EPERM;
  that environment failure is preserved, while the separate DEADLINE
  admission row remains valid.
- Running and stopped-process DEADLINE retries (`contention-v3`/`v4`) also
  returned EPERM; no DEADLINE contention claim is made.
- Cryptographic integrity: maintained userspace HMAC/tamper tests passed
  (`hmac=176` and `hmac=35` rejected frames); kernel map checks cover local
  schema/freshness/replay only because no kernel HMAC verifier exists.
- The documented signal-publication microbenchmark passed 12/12 bounded
  legacy/generation-stamped runs across 1/2/4 readers, with 12,000 successful
  publications, 27,161 verified reads, and zero HMAC/invalid/stale failures.
- NUMA: same-node migration and topology detection passed on the one-node host.

## F-006 — OPEN (SECURITY HARNESS INTERACTION)

The unchanged full `tests/security/run.sh` wrapper passes with CFS
(`sched_ext=disabled`) but exits at the observer-only installer/uninstaller
case when ORCHESTRA is attached. The installed observer-only package has no
trusted kernel loader, while `uninstall.sh` correctly tries to disable the
active scheduler and refuses the untrusted/missing loader. The direct
policy-loader, mutation, ABI-sanitizer, and source-security components all
passed while ORCHESTRA was attached, and loader detach returned to
`disabled`. This is a preserved test-harness/environment interaction, not a
cryptographic rejection failure; the comparison and raw traces are in
`security/CFS_ORCHESTRA_SECURITY_COMPARISON.md`.

## Remaining hard boundaries

The host has one NUMA node and no `numactl`; cross-node behavior is not
testable. The repository has no distributed backend or protocol, so loopback
network transfer is not distributed scheduling evidence. `scx_simple`,
`stress`, `fio`, `iperf3`, and `perf` are unavailable. These are preserved as
scope limitations rather than converted into passes.

## Preserved harness result

The first loopback network attempt failed because its readiness probe consumed
the one-shot netcat listener; it is preserved. The corrected loopback transfer
passed 32/64 MiB byte counts and clean unload, but its short-lived tasks did not
retain task-state entries for direct ownership attribution.

The first RT contention invocation used the campaign-owned BPF path and was
correctly rejected by the loader's root-safe artifact-path gate (`EPERM`). The
identical probe was rerun with the validated root-owned build path; both the
original rejection and corrected result are retained.

## Evidence-package issue resolved

Two checklist evidence paths used comma-containing brace notation without CSV
quoting. They were quoted and the complete campaign CSV set was reparsed with
consistent column counts. This was a report-format defect only; test outcomes
and raw evidence were unchanged.
