# ORCHESTRA-OS Evidence Repository

Evidence-only companion for **ORCHESTRA-OS: Implementation and Experimental Validation**.

This repository intentionally excludes implementation source code, kernel/BPF source, bridge source, and test programs. It contains the records needed to inspect the principal claims E1--E6 in the paper.

## Evidence map

| ID | Contents |
|---|---|
| E1 | 20 paired foreground trials, raw pair rows and statistics |
| E2 | Fixed-work raw runs and summary CSV |
| E3 | Action/lifecycle reports and archived runtime logs; compiled binaries are excluded |
| E4 | 14 September userspace build, unit, integration and security evidence |
| E5 | 14 September five-repetition CFS baseline; no fresh ORCHESTRA timing rows were produced |
| E6 | Exact copy of the supplied 6 July architecture/simulation paper |

The paper and verification record are at the repository root. SHA-256 values in `CHECKSUMS.sha256` verify file identity; they do not independently prove measurement validity. The September measurements were collected on a host with an active graphical desktop and background services. The professor's manuscript is architectural and simulation context, not a hardware result.

## Publication policy

This artifact may be published without releasing the implementation source. Keep the repository private or use a restricted/embargoed archival record when reviewer-only access is required.
