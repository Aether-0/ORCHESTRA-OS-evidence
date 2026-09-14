# Requirement-by-requirement claim audit

| Requested area | Direct evidence | Current conclusion |
|---|---|---|
| Performance | 3 matched CPU and 3 matched mixed repetitions; ownership yes | Bounded exploratory claim; measured CPU slowdown, no superiority claim |
| Stress | 5 s and 30 s CPU/I/O/mixed tiers; memory tool absent | Bounded CPU/I/O/mixed claim; memory blocked |
| RT | FIFO/RR/DEADLINE policy checks, bridge refusal, normal-task coexistence, FIFO/RR contention ownership | Bounded admission/coexistence claim; DEADLINE contention EPERM; no hard-RT guarantee |
| Cryptographic integrity | Userspace HMAC tamper counters plus 12/12 documented microbenchmark runs; kernel replay/stale checks | Userspace HMAC claim; kernel cryptographic auth not implemented |
| NUMA | One-node topology; same-node migration to CPU 1 | Single-node claim; cross-node blocked by hardware |
| Distributed | No backend/protocol in source; loopback only | Not implemented; full distributed claim cannot be made |
| Long duration | 600 s CPU + 600 s I/O + 600 s mixed, exact ownership, clean unload | Bounded 30-minute soak claim |
| CFS versus ORCHESTRA security | CFS full security wrapper pass; ORCHESTRA direct security components, attach/unload, and userspace tamper tests | ORCHESTRA adds a root-controlled signal/control surface and userspace HMAC, but kernel transport remains local-trust; it is not intrinsically safer than CFS |

The evidence is strong enough for the scoped rows and explicitly insufficient
for the unimplemented or unavailable rows. Expanding the latter claims would
require a new implementation, a multi-NUMA/multi-node test target, or both;
it cannot be achieved by relabeling this single-host campaign.

The focused security rerun also preserved one wrapper interaction: the
observer-only installer test passes with CFS but refuses to disable an active
ORCHESTRA scheduler when no trusted loader is installed. Direct policy/fuzz/
ABI/source-security components passed while attached, so this is a test-harness
boundary rather than evidence of an HMAC or scheduler-integrity failure.
