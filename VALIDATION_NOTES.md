# Validation and Calibration Notes

This file explains two common questions about the project:

1. Why is the FY26 Q1 backtest 85.7% rather than higher?
2. Is the model fully fine-tuned?

## Why the Score is 85.7%

The notebook reports an FY26 Q1 backtest Cost-Weighted Accuracy (CWA) of 85.7%, compared with 87.2% for Demand Planners.

That score is shaped by a few important tradeoffs:

- The project aims for interpretability, not only raw backtest optimization.
- The demand mix includes volatile low-volume products, decline-stage products, and one NPI-ramp product, all of which are harder to model reliably.
- The pipeline includes structural improvements such as big-deal decomposition and external signal modeling from SCMS and VMS data.
- Those upstream improvements change the behavior of the baseline forecast, which means downstream correction rules can become miscalibrated unless they are re-tuned.
- The benchmark itself is strong because Cisco's internal planner forecasts already embed domain knowledge and operational context.

In other words, the 85.7% result reflects a competitive and explainable model, but not a fully score-maximized one.

## Is the Model Fine-Tuned?

The most accurate description is:

**The model is calibrated and partially tuned, but not fully fine-tuned in a strict end-to-end validation framework.**

What is tuned or calibrated:

- Baseline window selection
- Seasonal weighting
- Trend clipping
- Statistical vs expert blend ratio
- Outlier handling
- PLC-aware correction heuristics

What is not fully completed:

- End-to-end re-tuning after every structural change
- Strict rolling-origin cross-validation across all correction parameters
- Leakage-free expert weighting for every backtest scenario
- Separate specialized handling for the hardest low-volume and NPI cases

## Why This Still Matters

Even without full end-to-end fine-tuning, the project is useful as a portfolio case study because it demonstrates:

- forecasting pipeline design
- integration of multiple data sources
- explainable model structure
- business-aware calibration
- dashboard-driven communication of results

## Best Interpretation

This repository should be read as an applied analytics and forecasting project that balances modeling, interpretability, and storytelling.

It is not presented as a final production forecasting system, and that distinction is intentional.
