# Wildfire Risk: Milestone 2

This project investigates whether information available before a wildfire is discovered can help identify higher-risk locations. The current milestone builds a reproducible fire-record pipeline and evaluates two simple probability baselines. Weather and satellite features have not yet been integrated.

## Current prediction task

Each observation is a 5 km grid cell on a calendar date. The label is 1 if at least one fire is recorded as discovered in that cell on any of the next three calendar days, and 0 otherwise.

**Discovery time is not ignition time.** This label does not establish a 24–72-hour pre-ignition warning. A zero label means no recorded discovery in the window; it does not prove that no fire occurred.

- Study period: January 1, 2011–December 31, 2020.
- Study area: latitude `[39, 40)` and longitude `[-122, -121)` in California.
- Spatial projection: WGS 84 / UTM zone 10N (`EPSG:32610`).
- Grid: 5,000 m cells anchored to the UTM coordinate origin, including cells intersecting the study boundary. Only events inside the geographic study boundary are counted, so edge cells represent smaller observed areas.

## Repository files

Keep these files together in the repository root:

| File | Purpose |
|---|---|
| `milestone2_analysis.ipynb` | Data checks, grid construction, labels, plots, and baseline evaluation |
| `Fires_202610082057.csv` | California fire-record extract used by the notebook |
| `requirements.txt` | Python dependencies |
| `README.md` | Data provenance, instructions, results, and limitations |

The original SQLite database is not included. The notebook constructs the cell-day panel from the CSV, so the generated panel does not need to be uploaded separately.

## Data source and extraction

Source: USDA Forest Service, *Spatial wildfire occurrence data for the United States, 1992–2024*, seventh edition (`FPA_FOD_20260615`).

- Version-specific archive: https://doi.org/10.2737/RDS-2013-0009.7
- Official dataset page: https://research.fs.usda.gov/firelab/products/dataandtools/us-wildfires

## Baseline results

Both baselines are estimated using training labels only:

1. **Constant probability:** predict the training positive rate for every test row.
2. **Monthly probability:** predict the training positive rate for the corresponding calendar month.

| Baseline | Average precision (higher is better) | Brier score (lower is better) |
|---|---:|---:|
| Constant probability | 0.007017 | 0.006968 |
| Monthly probability | 0.011565 | 0.006954 |

The monthly baseline provides some ranking improvement, while Brier scores differ very little. Training positive rates are highest in June and July. These are preliminary seasonal results, not evidence of a reliable pre-ignition signal. Metrics use scikit-learn's `average_precision_score` and `brier_score_loss`. Accuracy is not emphasized because positive labels are rare.

## Limitations and next steps

- Discovery-hour information is missing for 3.01% of the statewide extract and 20.72% of its 2020 records. Date-only labels retain these records, but hour-level lead times are unsupported.
- Adjacent cells and overlapping three-day windows create dependent observations. Row counts are not counts of independent events, and uncertainty intervals have not yet been estimated.
- Spatial holdout validation remains to be implemented. Future model development should use training-period validation and preserve an untouched final test set.
- Weather and satellite features require explicit observation-window and publication-availability rules. Historical revised weather values may differ from what was available at the prediction time.
- Active-fire detections and contemporaneous fire counts must not be used as pre-fire predictors. Vegetation composites must be complete and available before the prediction cutoff.

Next work: acquire weather features, align them to the grid and decision cutoff, audit missingness and leakage, then compare a weather model against the seasonal baseline using temporal and spatial validation.
