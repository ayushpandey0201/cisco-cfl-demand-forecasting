# Architecture Flow

This project is organized as a layered forecasting workflow that starts with competition data ingestion, builds product-level demand forecasts through six model stages, and ends with backtest, dashboard, and submission exports.

```mermaid
flowchart TD
    A["Competition Data Pack (.xlsx)"] --> B["Step 1-5: Load and Parse Data"]

    B --> B1["Actual Bookings\n30 products x 12 quarters"]
    B --> B2["Big Deal Sheet\nAvg-deal vs big-deal split"]
    B --> B3["SCMS Sheet\nChannel mix trends"]
    B --> B4["VMS Sheet\nVertical market trends"]
    B --> B5["Accuracy / Bias Tables\nDP, Marketing, Data Science"]

    B2 --> C["Demand Decomposition"]
    B3 --> D["External Signal Engine"]
    B4 --> D
    B5 --> H["Expert Weighting + Outlier Handling"]
    B1 --> E["6-Layer Forecasting Pipeline"]

    D --> D1["Channel momentum + vertical momentum\ncombined into per-product multipliers"]
    D1 --> E
    C --> E
    H --> E

    subgraph PIPE["Core Modeling Pipeline"]
        E1["L1 Baseline\nRolling mean on avg-deal demand"]
        E2["L2 Seasonality\nRecency-weighted same-quarter index"]
        E3["L3 Trend\nYoY growth ratio with clipping"]
        E4["L4 Expert Blend\n0.4 statistical + 0.6 expert consensus"]
        E5["L5 External Signals\nSCMS + VMS multiplier"]
        E6["L6 Correction Layer\nPLC-aware business adjustments"]
    end

    E --> E1 --> E2 --> E3 --> E4 --> E5 --> E6

    E6 --> F["FY26 Q1 Backtest"]
    E6 --> G["FY26 Q2 Forward Forecast"]

    F --> F1["Cost-Weighted Accuracy (CWA)\nBias and per-product accuracy"]
    F --> F2["Multi-quarter comparison\nvs Demand Planners / DS / Marketing"]
    F --> F3["Stress test\n10 noisy reruns"]

    G --> G1["Submission CSV"]
    G --> G2["Forecast decomposition table"]
    G --> G3["Lifecycle mix analysis"]

    F1 --> J["Dashboard + Project Story"]
    F2 --> J
    F3 --> J
    G1 --> J
    G2 --> J
    G3 --> J

    J --> K["Final Repo Assets\nNotebook + Dashboard + CSVs + README"]
```

## What the Diagram Shows

- The notebook is not just a time-series forecast. It combines historical demand, business decomposition, expert input, and external market signals.
- The model is intentionally layered so each stage can be inspected and explained.
- The project output is broader than a single metric: it includes validation, scenario robustness, visualization, and exported deliverables.
