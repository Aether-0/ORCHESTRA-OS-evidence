# Campaign status

Status: COMPLETE for the supported/scoped validation envelope  
Started: 2026-08-28 16:35 IST (fix runtime retry)  
Extended validation: 2026-08-28 21:49–22:34 IST  
Host: `redshadow`, kernel `7.0.12+kali-amd64`

## Closed scoped claims

- F-001 and F-005 fixes: verified.
- CPU and mixed performance: three matched repetitions with ownership gates.
- CPU/I/O/mixed stress: 5-second and 30-second tiers; 600-second phases.
- Long duration: 30-minute, three-phase ORCHESTRA soak, clean unload.
- RT admission: FIFO/RR/DEADLINE refusal and normal-task coexistence.
- RT contention: FIFO/RR bounded contention with normal-task ownership;
  DEADLINE contention launch was environment-blocked by EPERM.
- Userspace HMAC/tamper and kernel-local freshness/replay behavior.
- Documented userspace signal-publication microbenchmark: 12/12 valid runs,
  zero HMAC/invalid/stale failures.
- Single-node NUMA/core migration behavior.
- Focused security comparison: the complete existing wrapper passed with CFS;
  direct policy/fuzz/ABI/source-security components passed while ORCHESTRA was
  attached; the observer-only installer interaction is preserved as a harness
  limitation, not a cryptographic failure.

## Explicitly not closed

- Memory pressure attribution (`stress` missing).
- `scx_simple` comparison (`/usr/bin/scx_simple` missing).
- Kernel HMAC authentication (not implemented; map is local-trust IPC).
- Cross-node NUMA (host has one node).
- Distributed scheduling (no backend/protocol exists).
- Hard-real-time guarantees, predictor convergence, signed provenance, or
  deployment readiness.
- Full security-wrapper parity while an externally attached scheduler is
  active (observer-only uninstall has no trusted kernel loader); see
  `security/CFS_ORCHESTRA_SECURITY_COMPARISON.md`.

See `REAL_WORLD_TEST_REPORT.md` for the evidence-backed claim ledger.
