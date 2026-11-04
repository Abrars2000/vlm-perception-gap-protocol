# Kill conditions — Option B v3

Pre-commitment rule: every numeric threshold below is a design rule, fixed
before any predictor result is computed. Thresholds are not taken from the
reviewed PDFs. They must not change after viewing P1/P2 values, ΔR², or gap
results. Any change requires a new protocol version (e.g. v4) committed before
the confirmatory run.

Computation: ΔR² for a predictor = R²(full law) − R²(law without that
predictor), using identical folds and pooled out-of-fold predictions.
For the primary law (P1 + P2):
- ΔR²(P1) = R²(P1+P2) − R²(P2)
- ΔR²(P2) = R²(P1+P2) − R²(P1)
Report LOAO and LOMO separately. 95% CI is a crossed cluster-bootstrap CI
over attribute and model clusters.

## Primary hypotheses

| ID | Predictor | Metric | Support | Fail | Inconclusive |
|---|---|---|---|---|---|
| H1 | P1 — language prior | ΔR²(P1) on LOAO and LOMO; 95% CI | ΔR² ≥ 0.02 on both axes, and both CI lower bounds > 0 (design rule, fixed before any predictor result is computed) | ΔR² < 0.00 on either axis, and that axis's CI upper bound < 0.00 (design rule, fixed before any predictor result is computed) | Any other result. Reported as "not supported", not "refuted" |
| H2 | P2 — feature separability | ΔR²(P2) on LOAO and LOMO; 95% CI | ΔR² ≥ 0.02 on both axes, and both CI lower bounds > 0 (design rule, fixed before any predictor result is computed) | ΔR² < 0.00 on either axis, and that axis's CI upper bound < 0.00 (design rule, fixed before any predictor result is computed) | Any other result. Reported as "not supported", not "refuted" |
| H4 | Overall law P1 + P2 | Pooled OOF R² on LOAO and LOMO; 95% CI | LOAO R² ≥ 0.10 and LOMO R² ≥ 0.10, both CI lower bounds > 0 (design rule, fixed before any predictor result is computed) | LOAO R² < 0.00 or LOMO R² < 0.00, with that axis's CI upper bound < 0.00 (design rule, fixed before any predictor result is computed) | Any other result, including 0.00 ≤ R² < 0.10 or a CI spanning zero. Reported as "generalisation not supported", not "refuted" |

Consistency: the H4 support threshold (0.10 on each axis) is the same
generalisation criterion stated in claim_v3.txt.

## Inconclusive outcomes (all hypotheses)

Any result that meets neither Support nor Fail is Inconclusive and is reported
as "not supported", never as "refuted". For H1 and H2 this includes
0.00 ≤ ΔR² < 0.02 or a CI spanning zero. For H4 this includes
0.00 ≤ R² < 0.10 on either axis or a CI spanning zero.
(design rule, fixed before any predictor result is computed)

Mixed outcomes (Support on one axis, Inconclusive on the other) are reported as
"not supported" for that hypothesis. They are not reported as partial support.

## Sensitivity (not confirmatory)

H3 — P3 projector bandwidth. Sensitivity analysis only. Not a confirmatory
hypothesis and not used to support any primary claim.
- Quantity: ΔR²(P3) = R²(with-P3) − R²(without-P3), on LOAO and LOMO.
- Control: false-positive rate of ΔR²(P3) ≥ 0.02 under β_P3 = 0 with ρ = 0.
  Report the rate with a binomial CI. Tolerance and reporting rule are fixed
  before the run (design rule).
- Result is reported descriptively. No support/fail verdict is issued for H3.

## Why each threshold is used (provisional)

- ΔR² ≥ 0.02: a predeclared practical-effect threshold (about two
  percentage points of held-out variance). Not literature-derived.
- CI lower bound > 0: the interval must exclude zero on the positive side.
- R² ≥ 0.10: a pragmatic minimum for each generalisation axis. Not
  literature-derived.
- Thresholds are provisional. They will be justified by simulation before
  the confirmatory run, and any change requires a new protocol version.

## Interpretation safeguard

With 4 attributes and 3 models there are 12 cells and only 4 attribute and 3
model clusters. LOAO/LOMO estimates and crossed cluster-bootstrap CIs will be
unstable. Report this as a small-scope exploratory study. Image-level
replication does not increase the number of independent attribute or model
clusters.
