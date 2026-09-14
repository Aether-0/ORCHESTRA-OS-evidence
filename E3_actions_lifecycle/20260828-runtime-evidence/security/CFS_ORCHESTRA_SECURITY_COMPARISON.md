# Focused CFS versus ORCHESTRA security comparison

Campaign: `20260828-132302-redshadow-fix-implementation`  
Host: `redshadow`  
Kernel: `7.0.12+kali-amd64`

## Executed controls

| Control | Scheduler state | Result | Evidence |
| --- | --- | --- | --- |
| Existing full security wrapper with CFS | `sched_ext=disabled` | `PASS` | `cfs_security_suite.stdout`, `cfs_security_suite.rc` |
| `scripts/security-scan.sh` | static | `PASS` (`SECURITY_SCAN_PASS`) | `focused_r1_scan.stdout` |
| Existing full security wrapper with ORCHESTRA attached | `sched_ext=enabled` during test; clean unload afterward | `FAIL_PRESERVED` at the observer-only installer/uninstaller case | `orchestra_security_suite.stdout`, `orchestra_security_suite.stderr`, `orchestra_security_suite.rc` |
| Direct policy, mutation, ABI-sanitizer, and source-security components with ORCHESTRA attached | `sched_ext=enabled` during test | `PASS` (all component return codes zero) | `orchestra_components_r1/` |
| ORCHESTRA detach after focused run | `sched_ext=disabled` | `PASS` | `orchestra_security_unload.rc`, `orchestra_state_after.txt` |

The preserved ORCHESTRA wrapper failure is not a cryptographic failure. The
existing `test_install_paths.sh` first installs an observer-only package with
no kernel loader, then uninstalls it. When an external ORCHESTRA scheduler is
active, `uninstall.sh` correctly attempts to disable it; the observer-only
package has no trusted loader artifact, so the safety check refuses with
`untrusted loader`. The same unchanged test passes with CFS detached. No test
script was modified.

## Integrity measurements

The existing integration run passed in both its baseline and ORCHESTRA modes.
The baseline log recorded `hmac=0`; the valid ORCHESTRA log recorded `hmac=0`.
The explicit tamper cases were rejected by the userspace verifier with
`hmac=201` (tamper) and `hmac=42` (controller tamper), and the run returned
zero. The raw run is under `integration_r2/`.

The documented userspace publication microbenchmark also completed 12/12
validated legacy/generation-stamped runs across 1/2/4 readers, with zero HMAC,
invalid-frame, or stale-frame failures. Its output is under
`microbenchmark_r2/`.

## What cryptography is actually present

ORCHESTRA has a canonical 128-byte userspace signal payload with a separate
full 32-byte HMAC-SHA256 tag. Userspace tests cover canonical serialization,
known vectors, tamper rejection, generation-stamped publication, and
freshness/replay behavior. This is a bounded userspace integrity claim.

The kernel bridge does **not** verify that HMAC. Its map transport is
local-trust and instead checks schema, identity, generation, freshness, range,
and fail-closed fallback. Therefore the result is not kernel cryptographic
authentication and not a production key-management claim.

## Security posture relative to CFS

| Property | Linux CFS baseline | ORCHESTRA prototype |
| --- | --- | --- |
| Custom scheduler control plane | None from this project | Root-controlled loader, bridge, pinned BPF maps, and policy inputs |
| Signal cryptography | No ORCHESTRA signal path | Userspace HMAC-SHA256; kernel map remains local-trust |
| Replay/freshness checks | Not applicable to ORCHESTRA signals | Userspace HMAC/generation/freshness plus kernel generation/freshness checks |
| Malformed scheduler directives | No project directive interface | Bounded parser, identity/RT admission, and RUN/fail-closed fallback |
| Privilege boundary | In-kernel scheduler and normal Linux administration | Same root-trusted boundary, plus a larger root-controlled BPF/bridge surface |
| Remote/distributed exposure | Not introduced by this project | No distributed backend or remote protocol implemented |
| Relative risk conclusion | Smaller project-specific attack surface | More controls for the added signal/control path, but also more attack surface and no kernel HMAC; not intrinsically safer than CFS |

The evidence supports `EXPERIMENTALLY_VALIDATED` userspace integrity and
`KERNEL_PROTOTYPED` local validation only. It does not support a claim that
ORCHESTRA is cryptographically stronger than CFS, because CFS has no matching
custom signal path and the ORCHESTRA kernel path does not authenticate the
userspace HMAC.
