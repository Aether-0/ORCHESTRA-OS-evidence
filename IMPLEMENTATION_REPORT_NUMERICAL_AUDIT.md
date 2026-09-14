# Numerical and attribution audit

Existing records were reviewed for the final implementation report on 14 September 2026. No new scheduler or workload tests were performed.

## Recalculated from raw CSVs

The 40 rows in `final_reports/20260901-orchestra-cfs-validation-report/data/photo_archival_pairs.csv` form 20 pairs. Direct CSV calculations reproduce:

- Foreground means: CFS 7987.8 ms; ORCHESTRA 2036.6 ms.
- Sample SDs: 1202.5486 ms and 297.0383 ms.
- Medians: 8054.5 ms and 2032.0 ms.
- Mean paired reduction: 74.2404788%; sample SD 3.2451278 percentage points.
- 20/20 pairs improve; reduction range 68.7665198–79.7913950%.
- The descriptive t interval uses the archived 20-pair summary; it is not a generalization across machines or workload families.

The five raw repetition CSVs under `evidence/20260914-paper-proof/baseline-2s/` reproduce:

| Workers | All elapsed times (ms) | Mean | Sample SD |
| ---: | --- | ---: | ---: |
| 1 | 2083, 2022, 2081, 1979, 2030 | 2039.0 | 43.7893 |
| 2 | 2022, 1917, 1810, 1816, 1862 | 1885.4 | 87.6173 |
| 4 | 1926, 1872, 1888, 1871, 1935 | 1898.4 | 30.2374 |

These 15 rows are CFS measurements. No fresh ORCHESTRA timing row is inferred. The older fixed-work table matches the seven rows of `data/fixed_work_summary.csv`; its raw repetitions are retained alongside it.

## Validation chronology and trust boundaries

The 3 September integration assertion failure remains in its archive and is explicitly dated in the report. The 14 September unit, integration and security logs independently end with return code 0. Integration includes 34 CSV validator tests. One recorded publication stress pass contains 2000 successful generations and 7997 verified reads; this is not the sum across compiler passes. The aggregate documentation-path scan failure remains disclosed.

The workflow follows the documented target-matched compact enqueue executor. Storage of policy banks, prediction records or coordination fields is not treated as proof of online learning or population control. Userspace authentication and kernel-local freshness checks retain separate interpretations.

## Research attribution

The source architecture is credited to Geetha Ganesan and Panchabi Vaithiyanathan, authors of the supplied 6 July 2026 submission draft. Its corrected metric is cited as architectural context, not as a new hardware measurement. The report credits S.W. ZAW for the implementation report and retains the original manuscript at `reports/PROFESSOR_ARCHITECTURE_PAPER_20260706.pdf`.
