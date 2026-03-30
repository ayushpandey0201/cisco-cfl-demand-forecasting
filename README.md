# Layered Signal Decomposition for Expert–Statistical Demand Forecast Integration

> Evaluated on Cisco Champions Forecasting League (CFL) dataset (30 SKUs, multi-quarter backtesting)
> 
> A structured experimental study on decomposing enterprise demand into interpretable signal layers and quantifying the marginal contribution of expert knowledge, external market signals, and lifecycle-aware corrections to forecast accuracy.

## Abstract

Enterprise hardware demand is heavy-tailed, lifecycle-heterogeneous, and partially observable through multiple expert channels of varying reliability. We design a six-layer forecasting pipeline that decomposes demand into baseline, seasonal, trend, expert-blend, external-signal, and lifecycle-correction components, enabling per-layer ablation and inspection. On a 30-SKU Cisco product portfolio, the pipeline achieves **85.7% CWA** on a held-out quarter, outperforming Data Science (+4.3 pp) and Marketing (+7.1 pp) expert baselines while closing to within 1.5 pp of Demand Planners who access private deal-pipeline information. Ablation isolates expert blending as the dominant contributor (−3.3 pp when removed), with big-deal decomposition and lifecycle corrections each contributing 1.6–1.8 pp on complementary SKU segments.

## Pipeline Overview

A six-layer pipeline decomposes demand into interpretable components, enabling targeted calibration and ablation at each stage.

![Architecture](CFL_Dashboard.png)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA INGESTION                                  │
│  Actuals · Big-Deal Log · SCMS Channel · VMS Vertical · Expert Fcsts    │
└──────────────────────────────┬──────────────────────────────────────────┘
                               ▼
              ┌────────────────────────────────┐
              │  L1  Baseline Estimation       │  Windowed mean on stable
              │      (big-deal separated)      │  (non-lumpy) demand
              ├────────────────────────────────┤
              │  L2  Seasonality Adjustment    │  Multiplicative Q-over-Q
              │                                │  indices, decay-weighted
              ├────────────────────────────────┤
              │  L3  Trend Estimation          │  Linear trend on
              │                                │  deseasonalized residuals
              ├────────────────────────────────┤
              │  L4  Expert Blend              │  Accuracy-weighted fusion
              │                                │  of L1–L3 + 3 expert fcsts
              ├────────────────────────────────┤
              │  L5  External Signal Adj.      │  Bounded multiplicative
              │                                │  SCMS + VMS adjustments
              ├────────────────────────────────┤
              │  L6  PLC Correction            │  Lifecycle-stage floors,
              │                                │  NPI ramps, variance clips
              └────────────────┬───────────────┘
                               ▼
              ┌────────────────────────────────┐
              │  Backtest · Forecast · Dashboard│
              └────────────────────────────────┘
```

## 1. Problem Statement

Standard time-series methods underperform on enterprise networking hardware demand due to three structural properties: (i) heavy-tailed distributions from irregular large-deal orders, (ii) lifecycle heterogeneity across NPI-ramp, growth, maturity, and decline SKUs, and (iii) multiple expert forecasts that vary in reliability across product segments. We investigate whether explicit signal decomposition—isolating stable demand from lumpy demand, weighting experts by per-SKU historical accuracy, and integrating external channel/vertical signals—yields forecasts that are competitive with domain-expert benchmarks while remaining interpretable at each layer.

## 2. Hypothesis

A layered pipeline with per-component calibration will outperform both individual expert forecasts and monolithic statistical approaches. Four specific sub-hypotheses:

1. **Big-deal decomposition** reduces baseline variance on SKUs with irregular large orders.
2. **Accuracy-weighted expert blending** dominates any single expert source and uniform averaging.
3. **External market signals** (SCMS channel, VMS vertical) capture demand shifts invisible in shipment history alone.
4. **PLC-aware corrections** recover accuracy on tail SKUs (NPI, decline) that aggregate models systematically misforecast.

## 3. Methodology

### 3.1 Dataset

Cisco Champions Forecasting League (CFL) Phase 1 data pack:

| Source | Content | Granularity |
|---|---|---|
| Actuals | Historical quarterly shipments | SKU × Quarter |
| Big-Deal Log | Large-order flags | SKU × Quarter |
| SCMS Channel | Sell-through trends | Product family × Quarter |
| VMS Vertical | Sector momentum indices | Vertical × Quarter |
| Expert Forecasts | Demand Planner / Marketing / Data Science | SKU × Quarter |
| Accuracy Tables | Per-SKU expert historical accuracy | SKU × Expert × Quarter |

**Target:** FY26 Q2 unit demand, 30 SKUs. **Holdout:** FY26 Q1 (one-quarter-ahead backtest).

### 3.2 Layer Design

- **L1 – Baseline:** 4-quarter windowed mean computed on stable demand after big-deal separation. Large-order contributions re-injected as a stochastic component weighted by empirical frequency.
- **L2 – Seasonality:** Multiplicative seasonal indices from historical Q-over-Q ratios with exponential decay on older quarters.
- **L3 – Trend:** Linear trend estimated on deseasonalized residuals from L2.
- **L4 – Expert Blend:** Per-SKU accuracy-weighted combination of L1–L3 statistical output with three expert forecasts. Blend ratio (statistical vs. expert consensus) is a tunable parameter.
- **L5 – External Signals:** Bounded multiplicative adjustments from SCMS channel sell-through (leading indicator of downstream demand) and VMS vertical momentum (sector-level acceleration). Scaling bounds prevent overcorrection.
- **L6 – PLC Correction:** Lifecycle-stage-specific rules — floor constraints for decline SKUs, ramp curves for NPI SKUs, variance clipping for mature SKUs.

## 4. Experiments

### 4.1 Configuration

- **Metric:** Component Weighted Accuracy (CWA), Cisco's standard forecast accuracy measure
- **Holdout:** FY26 Q1 (most recent complete quarter)
- **Baselines:** Three expert forecasts evaluated independently on the same holdout

### 4.2 Main Results

| Method | FY26 Q1 CWA (%) | Δ vs Ours |
|---|---|---|
| Marketing Forecast | 78.6 | −7.1 |
| Data Science Forecast | 81.4 | −4.3 |
| **Pipeline (Ours)** | **85.7** | **—** |
| Demand Planners Forecast | 87.2 | +1.5 |

The 1.5 pp gap to Demand Planners is attributable to private information asymmetry: planners access customer conversations and deal-pipeline data unavailable to any data-driven model.

### 4.3 Robustness

| Metric | Value |
|---|---|
| Multi-quarter rolling CWA | 84.1% |
| Stress test mean CWA | 85.53% |
| Stress test σ | **0.59%** |

Sub-percent stress-test variance confirms that holdout performance is not an artifact of favorable quarter selection.

## 5. Ablation Study

Each component removed independently; re-evaluated on the FY26 Q1 holdout.

| Configuration | CWA (%) | Δ |
|---|---|---|
| Full pipeline (L1–L6) | **85.7** | — |
| Remove big-deal decomposition | 83.9 | −1.8 |
| Remove expert blend (statistical only) | 82.4 | −3.3 |
| Remove external signals (skip L5) | 84.5 | −1.2 |
| Remove PLC corrections (skip L6) | 84.1 | −1.6 |
| Uniform expert weights | 84.8 | −0.9 |

**Component ranking by marginal CWA contribution:**
Expert blend (3.3) > Big-deal decomposition (1.8) > PLC corrections (1.6) > External signals (1.2) > Accuracy-weighted vs. uniform blending (0.9).

## 6. Key Findings

- **Statistical and expert signals are complementary, not substitutable.** Removing the expert blend degrades CWA by 3.3 pp — the largest single ablation drop — because expert forecasts encode deal-pipeline and market context absent from shipment history. Conversely, the statistical backbone captures systematic trend and seasonality that experts estimate less reliably.
- **Big-deal separation operates on distributional shape, not level.** The 1.8 pp gain comes from variance reduction on heavy-tailed SKUs: separating lumpy demand prevents single large orders from biasing the moving-average baseline, without suppressing genuine demand-level shifts.
- **Lifecycle corrections have outsized per-SKU impact.** PLC corrections affect a small subset of products (NPI-ramp and decline-stage) but contribute 1.6 pp to aggregate CWA. This confirms that portfolio-level heterogeneity requires explicit handling — aggregate calibration systematically misforecasts tail SKUs.
- **External signals act as leading indicators with diminishing marginal returns.** Channel and vertical signals contribute 1.2 pp, consistent with their role as noisy but directionally informative signals. Bounded scaling is essential: unbounded signal integration degraded CWA by 0.4 pp in preliminary experiments.
- **Per-SKU expert weighting outperforms uniform blending by 0.9 pp.** Accuracy-weighted fusion exploits the observation that expert reliability varies substantially across SKUs — some products are systematically better forecast by planners, others by marketing.

## 7. Limitations

- **Private information gap.** The pipeline lacks access to deal-pipeline data, customer conversations, and sales-team intelligence that Demand Planners use. This likely accounts for most of the 1.5 pp gap and represents a structural ceiling for any model without CRM integration.
- **Pipeline coupling.** Layers are sequential and coupled: modifying upstream parameters (e.g., big-deal separation threshold, baseline window) shifts input distributions to downstream layers, requiring recalibration. No end-to-end joint optimization was performed.
- **External signal dependency.** SCMS and VMS signals are Cisco-internal data streams. The pipeline's transferability to settings without comparable leading indicators is unvalidated.
- **Limited holdout depth.** The primary backtest uses a single quarter (FY26 Q1). While multi-quarter rolling evaluation and stress testing partially mitigate this, the 30-SKU portfolio is small enough that individual SKU errors can disproportionately influence aggregate CWA.
- **Calibration status.** Parameters were calibrated via informed heuristics and partial grid search, not strict rolling cross-validation. Some backtest choices (e.g., expert weighting from accuracy tables) may exhibit mild lookahead bias.

## 8. Conclusion

This work shows that structured signal decomposition with per-layer ablation can approach domain-expert accuracy in enterprise demand forecasting while maintaining full interpretability. The key empirical result is that expert knowledge and statistical signals are complementary: their fusion consistently outperforms either source alone, with each component’s contribution measurable via ablation.

The remaining 1.5 pp gap to Demand Planners is best explained by information asymmetry—specifically, access to deal-pipeline and customer-level signals—rather than methodological limitations. This identifies CRM-integrated features as the highest-leverage direction for future improvement.

More broadly, the layered decomposition paradigm—where each component is independently interpretable and tunable—offers a practical alternative to end-to-end models in operational forecasting settings requiring transparency and auditability. The approach generalizes to domains with heterogeneous lifecycles, multiple expert channels, and available leading indicators, demonstrating that interpretable, modular systems can achieve near expert-level performance without proprietary signals.

## 9. Reproducibility

```bash
# Environment: Google Colab (Python 3.10+)

# 1. Upload CFL_Forecasting_Model.ipynb to Colab
# 2. Upload competition Excel data pack when prompted
# 3. Run all cells to reproduce:
#    → CFL_v2_Backtest_FY26Q1.csv   (backtest)
#    → CFL_v2_Submission_FY26Q2.csv (forecast)
#    → CFL_Dashboard.png            (dashboard)
```

```
├── CFL_Forecasting_Model.ipynb   # Pipeline implementation
├── CFL_Dashboard.png             # Results dashboard
├── CFL_v2_Backtest_FY26Q1.csv    # Backtest outputs
├── CFL_v2_Submission_FY26Q2.csv  # FY26 Q2 submission
├── ARCHITECTURE.md               # Pipeline flow documentation
└── VALIDATION_NOTES.md           # Validation methodology notes
```

---

*Author: Ayush Pandey*
