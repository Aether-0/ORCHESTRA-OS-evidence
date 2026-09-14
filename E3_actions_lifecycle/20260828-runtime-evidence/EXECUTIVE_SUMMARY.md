# ORCHESTRA-OS Validation Claims

Campaign: `20260828-132302-redshadow-fix-implementation`  
Host/kernel: `redshadow` / `7.0.12+kali-amd64`

The two confirmed defects remain fixed. Additional evidence now supports:

- exploratory matched CPU/mixed performance measurements (three repetitions);
- ownership-gated CPU/I/O/mixed stress through 30-second tiers;
- a 30-minute, three-phase ORCHESTRA soak (600 seconds per phase);
- bounded FIFO/RR/DEADLINE admission and normal-task coexistence, with FIFO/RR
  contention ownership;
- userspace HMAC/tamper integrity and kernel-local freshness/replay rejection;
- a documented userspace signal-publication microbenchmark with 12/12 valid
  runs and zero HMAC/invalid/stale failures;
- single-node NUMA/core migration behavior.
- focused security comparison: CFS full-wrapper pass, ORCHESTRA direct
  security-component pass, and clean loader detach; the one wrapper
  interaction is a preserved observer-only installer harness boundary.

Measured CPU completion time was slower than CFS for ORCHESTRA in this
prototype; no performance superiority claim is made. The memory stress row is
blocked because `stress` is not installed. Multi-node NUMA, distributed
scheduling, and kernel cryptographic authentication remain unavailable or
unimplemented and are not claimed.

See [REAL_WORLD_TEST_REPORT.md](REAL_WORLD_TEST_REPORT.md) for the scoped claim
ledger and raw evidence paths.
