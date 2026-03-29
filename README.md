# Cisco CFL Demand Forecasting

Personal forecasting project developed for the Cisco Champions Forecasting League (CFL) Phase 1 challenge.

This notebook builds a 6-layer demand forecasting pipeline to predict FY26 Q2 unit demand for 30 Cisco networking products. The approach combines statistical forecasting, expert forecast blending, external market signals, and business-rule corrections, with an emphasis on interpretability and presentation.

## Project Highlights

- 6-layer hierarchical forecasting pipeline
- Big-deal decomposition for separating stable demand from lumpy demand
- Accuracy-weighted expert blending across Demand Planners, Marketing, and Data Science
- External signal layer built from SCMS channel trends and VMS vertical momentum
- Dashboard-based summary of backtest performance, forecast decomposition, and robustness checks

## Main Results

- FY26 Q1 backtest CWA: 85.7%
- Demand Planners benchmark: 87.2%
- Data Science benchmark: 81.4%
- Marketing benchmark: 78.6%
- Weighted multi-quarter backtest CWA: 84.1%
- Stress test mean CWA: 85.53%
- Stress test standard deviation: 0.59%

## Why the Backtest is 85.7% and Not Higher

The FY26 Q1 backtest score is competitive, but not higher than Demand Planners, for a few practical reasons:

- The project prioritizes an interpretable layered pipeline over aggressive quarter-specific tuning.
- The long-tail products are harder to forecast, especially low-volume, decline-stage, and NPI-ramp SKUs.
- Structural improvements such as big-deal decomposition and data-driven external signals change the upstream forecast behavior, which means downstream correction rules need recalibration.
- Cisco's internal planner benchmark is already strong at 87.2%, so the comparison bar is high.

In short, the model is designed to be understandable and data-driven, but it is not fully optimized for maximum backtest score at any cost.

## Calibration and Fine-Tuning Status

This model is best described as **calibrated and partially tuned**, not fully fine-tuned in the strict production ML sense.

- Several parameters were chosen intentionally, including the baseline window, seasonal weights, blend ratio, clipping thresholds, and correction rules.
- The pipeline includes business-aware calibration, especially in the correction layer.
- However, it was not fully re-tuned end to end with strict rolling cross-validation after every structural change.
- Some backtest choices, such as expert weighting based on historical accuracy tables, can be made more rigorous in a future version.

For a fuller discussion of validation tradeoffs, see [VALIDATION_NOTES.md](VALIDATION_NOTES.md).

## Files

- `CFL_Forecasting_Model.ipynb` - main notebook
- `CFL_Dashboard.png` - final project dashboard
- `CFL_v2_Backtest_FY26Q1.csv` - backtest output
- `CFL_v2_Submission_FY26Q2.csv` - FY26 Q2 forecast output

## Method Overview

The forecasting pipeline is organized into six layers:

1. Baseline demand estimation
2. Seasonality adjustment
3. Trend estimation
4. Expert blend
5. External signal adjustment
6. PLC-aware correction layer

This structure was designed to keep the model interpretable, so each stage of the forecast can be inspected and explained.

## Architecture Flow

For a full project-level flow diagram, see [ARCHITECTURE.md](ARCHITECTURE.md).

```mermaid
flowchart LR
    A["Data Pack"] --> B["Parse Actuals + Big Deal + SCMS + VMS + Accuracy Tables"]
    B --> C["L1-L3 Statistical Forecast"]
    B --> D["Expert Weighting"]
    B --> E["External Signal Multipliers"]
    C --> F["L4 Expert Blend"]
    D --> F
    F --> G["L5 External Adjustment"]
    E --> G
    G --> H["L6 PLC-aware Correction"]
    H --> I["Backtest + FY26 Q2 Forecast"]
    I --> J["Dashboard + CSV Outputs"]
```

## Notes

This notebook is presented as an analytical competition project. Some backtest choices, such as expert-weighting based on historical accuracy tables, can be refined further for stricter validation.

The public version of this repository does not include the original competition Excel data pack.

## Running the Notebook

This notebook was designed for Google Colab.

To run it:
1. Open `CFL_Forecasting_Model.ipynb` in Colab.
2. Upload the competition Excel data pack when prompted.
3. Run the notebook end to end to reproduce the analysis and outputs.

## Key Learnings

- Better model structure does not always improve headline backtest metrics immediately.
- Forecasting quality depends on both statistical signal and domain-informed adjustments.
- Pipeline coupling matters: changing upstream assumptions can break downstream calibration.
- Clear storytelling and visualization are as important as model logic in real-world analytics work.

## Dashboard Preview

![CFL Dashboard](CFL_Dashboard.png)

## Author

Ayush Pandey
