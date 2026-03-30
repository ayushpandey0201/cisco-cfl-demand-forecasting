# Hierarchical Demand Forecasting with Layered Signal Decomposition: A Study on Expert–Statistical Forecast Integration

## Abstract

We present a six-layer hierarchical forecasting pipeline for predicting quarterly unit demand across 30 Cisco networking products spanning heterogeneous product lifecycle stages. The pipeline decomposes demand into stable base demand and lumpy big-deal contributions, blends statistical baselines with accuracy-weighted expert forecasts, and applies external market signals (channel sell-through and vertical momentum) as multiplicative adjustments. On an FY26 Q1 holdout, the pipeline achieves 85.7% Component Weighted Accuracy (CWA), narrowing the gap to Cisco's internal Demand Planners benchmark (87.2%) while substantially outperforming both Data Science (81.4%) and Marketing (78.6%) expert baselines. Ablation experiments reveal that big-deal decomposition and the external signal layer each contribute approximately 1–2 percentage points of CWA, while the PLC-aware correction layer is critical for tail-SKU stability. Stress testing across perturbed demand scenarios yields a mean CWA of 85.53% (σ = 0.59%), indicating forecast robustness under distributional shift.

## 1. Problem Statement

Enterprise demand forecasting for networking hardware exhibits several structural challenges that limit the effectiveness of standard time-series methods: (i) demand distributions are heavy-tailed due to irregular large-deal orders, (ii) product portfolios span multiple lifecycle stages (NPI ramp, growth, maturity, decline), each with distinct signal characteristics, and (iii) multiple expert forecasts (Demand Planners, Marketing, Data Science) are available but vary in accuracy across product segments. The question we investigate is whether a structured decomposition approach—separating stable demand from lumpy demand, weighting experts by historical accuracy, and incorporating external market signals—can produce forecasts that are both competitive with domain-expert benchmarks and interpretable at each layer.

## 2. Hypothesis

A layered pipeline that explicitly separates demand components (baseline, seasonality, trend, expert opinion, external signals, lifecycle corrections) will outperform individual expert forecasts and single-model statistical approaches by enabling targeted calibration at each layer. Specifically, we hypothesize that:

1. Big-deal decomposition will reduce forecast variance on SKUs with irregular large orders.
2. Accuracy-weighted expert blending will dominate any single expert source.
3. External market signals (SCMS channel trends, VMS vertical momentum) will capture demand shifts not visible in historical shipment data alone.
4. Product-lifecycle-aware corrections will improve accuracy on tail SKUs (NPI, decline-stage) that are poorly served by aggregate statistical models.

## 3. Methodology

### 3.1 Dataset

Cisco Champions Forecasting League (CFL) Phase 1 data pack containing:

| Data Source | Description | Granularity |
|---|---|---|
| Actuals | Historical quarterly shipment units | SKU × Quarter |
| Big-Deal Log | Large-order decomposition flags | SKU × Quarter |
| SCMS Channel Data | Sell-through channel trends | Product family × Quarter |
| VMS Vertical Data | Vertical market momentum indices | Vertical × Quarter |
| Expert Forecasts | Demand Planner, Marketing, Data Science forecasts | SKU × Quarter |
| Accuracy Tables | Historical expert accuracy by SKU | SKU × Expert × Quarter |

Target: FY26 Q2 unit demand for 30 SKUs. Holdout: FY26 Q1 (one-quarter-ahead backtest).

### 3.2 Pipeline Architecture

The pipeline processes demand through six sequential layers:

| Layer | Operation | Key Parameters |
|---|---|---|
| L1: Baseline | Windowed mean over stable (non-big-deal) demand | Window = 4Q |
| L2: Seasonality | Multiplicative seasonal indices from historical Q-over-Q ratios | Weight decay on older quarters |
| L3: Trend | Linear trend component estimated on deseasonalized residuals | — |
| L4: Expert Blend | Accuracy-weighted combination of L1–L3 output with expert forecasts | Weights from accuracy tables |
| L5: External Signals | Multiplicative adjustment from SCMS channel trends and VMS vertical momentum | Signal-specific scaling bounds |
| L6: PLC Correction | Product-lifecycle-aware clipping, floor/ceiling rules, and NPI ramp adjustments | PLC stage classification |

### 3.3 Key Techniques

**Big-deal decomposition.** Orders flagged as big-deals are separated before baseline estimation. L1 computes the baseline on the residual (stable) demand series, and big-deal contributions are added back as a separate stochastic component with empirical frequency weighting. This prevents single large orders from distorting the moving-average baseline.

**Accuracy-weighted expert blending.** Expert weights in L4 are derived from historical per-SKU accuracy tables rather than uniform averaging. Experts with consistently higher accuracy on a given SKU receive proportionally higher weight. The blend ratio between the statistical forecast (L1–L3) and the weighted expert consensus is a tunable parameter.

**External signal integration.** L5 applies multiplicative adjustments derived from two external sources: (i) SCMS channel sell-through trends, which capture downstream demand shifts before they appear in shipment data, and (ii) VMS vertical momentum indices, which reflect sector-level demand acceleration or deceleration. Signal multipliers are bounded to prevent overcorrection.

**PLC-aware corrections.** L6 applies lifecycle-stage-specific rules: floor constraints for decline-stage SKUs (preventing negative or implausible forecasts), ramp curves for NPI SKUs, and variance clipping for mature SKUs with stable demand histories.

## 4. Experiments

### 4.1 Backtest Configuration

- **Holdout:** FY26 Q1 (most recent complete quarter)
- **Metric:** Component Weighted Accuracy (CWA), Cisco's standard forecast accuracy metric
- **Baselines:** Three expert forecasts (Demand Planners, Marketing, Data Science), each evaluated independently on the same holdout

### 4.2 Main Results

| Method | FY26 Q1 CWA (%) |
|---|---|
| Marketing Forecast | 78.6 |
| Data Science Forecast | 81.4 |
| **Pipeline (Ours)** | **85.7** |
| Demand Planners Forecast | 87.2 |

The pipeline outperforms both Data Science (+4.3 pp) and Marketing (+7.1 pp) expert baselines. It trails Demand Planners by 1.5 pp, which is expected given that Demand Planners incorporate private contextual information (customer conversations, deal pipeline visibility) unavailable to any statistical model.

### 4.3 Multi-Quarter Backtest

To assess temporal stability, we evaluate the pipeline across multiple historical quarters using a rolling one-quarter-ahead protocol.

| Metric | Value |
|---|---|
| Weighted multi-quarter CWA | 84.1% |
| Stress test mean CWA | 85.53% |
| Stress test σ | 0.59% |

The low stress-test variance (σ = 0.59%) indicates that performance is not an artifact of favorable holdout selection.

## 5. Ablation Study

We ablate each major pipeline component by removing it and re-evaluating on the FY26 Q1 holdout.

| Configuration | FY26 Q1 CWA (%) | Δ vs Full Pipeline |
|---|---|---|
| Full pipeline (L1–L6) | 85.7 | — |
| No big-deal decomposition (raw actuals in L1) | 83.9 | −1.8 |
| No expert blend (L1–L3 statistical only) | 82.4 | −3.3 |
| No external signals (skip L5) | 84.5 | −1.2 |
| No PLC corrections (skip L6) | 84.1 | −1.6 |
| Uniform expert weights (equal blend) | 84.8 | −0.9 |

**Observations:**

- Expert blending (L4) is the single most impactful component (−3.3 pp without it), confirming that domain knowledge complements statistical baselines.
- Big-deal decomposition contributes 1.8 pp, primarily on SKUs with heavy-tailed order distributions.
- PLC corrections (−1.6 pp) are disproportionately important for tail SKUs despite affecting few products, consistent with the hypothesis that lifecycle-stage heterogeneity requires explicit handling.
- The gap between accuracy-weighted and uniform expert blending (0.9 pp) validates the per-SKU weighting strategy over naive averaging.

## 6. Key Findings

- **Expert knowledge is not redundant with statistical signal.** The optimal configuration blends both; neither alone achieves the combined result. Statistical methods capture trend and seasonality systematically, while expert forecasts encode deal-pipeline and market context that is absent from historical data.
- **Big-deal decomposition reduces variance without sacrificing level accuracy.** Separating lumpy demand prevents large orders from corrupting the baseline, yielding more stable forecasts on the majority of SKUs while preserving sensitivity to genuine demand shifts.
- **External signals provide marginal but consistent gains.** Channel and vertical momentum signals contribute +1.2 pp CWA. The bounded multiplicative design prevents overcorrection while still capturing demand acceleration visible in downstream channels before shipment data reflects it.
- **Lifecycle heterogeneity is a first-order concern.** A single model calibrated to the aggregate portfolio systematically underperforms on NPI and decline-stage SKUs. PLC-aware corrections address this at low complexity cost.
- **Pipeline coupling creates calibration fragility.** Structural changes to upstream layers (e.g., modifying the big-deal separation threshold) shift the distribution of inputs to downstream layers, requiring recalibration. This coupling is a practical limitation of layered architectures and motivates end-to-end tuning in future work.

## 7. Conclusion

This work demonstrates that a structured, interpretable forecasting pipeline can approach domain-expert accuracy on enterprise hardware demand prediction. The layered decomposition design enables per-component inspection and targeted improvement, trading marginal backtest optimality for transparency and maintainability. The 1.5 pp gap to Demand Planners likely reflects private information asymmetry rather than methodological limitation. Future work should explore end-to-end parameter optimization via rolling cross-validation and integration of deal-pipeline features to close this gap.

## 8. Reproducibility

```bash
# Environment: Google Colab (Python 3.10+)
# 1. Open the notebook
#    Upload CFL_Forecasting_Model.ipynb to Colab

# 2. Upload data
#    Upload the competition Excel data pack when prompted

# 3. Execute
#    Run all cells sequentially to reproduce:
#    - FY26 Q1 backtest (CFL_v2_Backtest_FY26Q1.csv)
#    - FY26 Q2 forecast (CFL_v2_Submission_FY26Q2.csv)
#    - Dashboard visualization (CFL_Dashboard.png)
```

### Repository Structure

```
├── CFL_Forecasting_Model.ipynb   # Full pipeline implementation
├── CFL_Dashboard.png             # Results dashboard
├── CFL_v2_Backtest_FY26Q1.csv    # Backtest outputs
├── CFL_v2_Submission_FY26Q2.csv  # FY26 Q2 forecast submission
├── ARCHITECTURE.md               # Pipeline flow documentation
└── VALIDATION_NOTES.md           # Validation methodology discussion
```

---

*Author: Ayush Pandey*
