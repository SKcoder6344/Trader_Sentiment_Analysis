# Trader Performance vs Market Sentiment
### Hyperliquid × Fear/Greed Index 

---

## Overview

This project analyzes how Bitcoin market sentiment (Fear/Greed) relates to trader behavior and performance on Hyperliquid. The goal is to uncover patterns that could inform smarter, sentiment-aware trading strategies.

---

## Datasets

| Dataset | Source | Size |
|---|---|---|
| Hyperliquid Trader Data | Provided (.csv.gz) | 211,224 trades, 32 accounts |
| Bitcoin Fear/Greed Index | Alternative.me | 2,644 daily readings |

**Date range after merge:** May 2023 – May 2025 (479 days, 211,218 trades retained)

---

## Project Structure

```
├── trader_sentiment_analysis.ipynb   # Main notebook
├── fear_greed_index.csv              # Sentiment dataset
├── README.md                         # This file
├── chart1_performance_fear_vs_greed.png
├── chart2_behavior_fear_vs_greed.png
├── chart3_segments.png
├── chart4_segment_behavior.png
├── chart5_insights.png
├── chart6_clusters.png
└── chart7_model.png
```

---

## Setup & How to Run

1. Clone this repository
2. Upload `compressed_data.csv.gz` and `fear_greed_index.csv` to your Google Drive
3. Open `trader_sentiment_analysis.ipynb` in Google Colab
4. Update the file paths in Block 2 to match your Drive location:
```python
TRADER_FILE    = '/content/drive/MyDrive/compressed_data.csv.gz'
SENTIMENT_FILE = '/content/drive/MyDrive/fear_greed_index.csv'
```
5. Run all cells top to bottom (Runtime → Run All)

**Required libraries:** pandas, numpy, matplotlib, seaborn, scipy, scikit-learn — all available by default in Google Colab.

---

## Methodology

**1. Data Preparation**
- Cleaned comma-formatted numeric columns in the trader dataset
- Parsed IST timestamps into daily date keys for alignment
- Merged both datasets on date using an inner join — 6 rows lost (<0.01%)
- Simplified sentiment into Fear vs Greed by collapsing Extreme labels, retaining the full classification for reference

**2. Feature Engineering**
- Built daily per-account metrics: total PnL, win rate, trade frequency, estimated leverage, long/short ratio, consistency score (% of profitable days), and a Sharpe-like ratio (mean return / std of returns)
- Estimated leverage as Size USD / abs(Start Position), capped at 200x to remove data artifacts

**3. Analysis**
- Compared performance and behavior across Fear vs Greed days
- Used Mann-Whitney U test for statistical significance (non-parametric — PnL distributions are heavily skewed)
- Segmented traders into three schemes: Leverage, Frequency, and Winner Type

**4. Bonus**
- KMeans clustering with silhouette-score-based k selection to identify behavioral archetypes
- Logistic regression to predict next-day trader profitability using today's behavior and sentiment features

---

## Key Insights

**Insight 1 — Greed days produce consistent profits; Fear days produce outliers**

Median PnL on Greed days ($265) is more than double Fear days ($123), and losing days drop from 12% to 7.6%. However, mean PnL is actually higher on Fear days ($5,185 vs $4,144) — driven by a small number of traders making outsized gains during volatile conditions. The difference is borderline significant (Mann-Whitney p = 0.062), meaning the edge is real but not guaranteed.

**Insight 2 — Traders get more aggressive during Fear, not less**

On Fear days, traders place more trades (31 vs 28), use higher leverage (7.7x vs 6.4x), and generate higher daily volume ($83,640 vs $61,128). This is counter-intuitive — fear does not cause retreat, it triggers overactivity. Only per-trade size drops slightly ($1,854 vs $2,005), the single sign of caution.

**Insight 3 — Win rate is a misleading performance metric**

Unprofitable traders in this dataset have a 72.9% win rate but negative total PnL. Consistent Winners have an 83.4% win rate and strongly positive total PnL. The difference is risk management — unprofitable traders win small and lose large. Consistency score (% of profitable days) predicts total PnL far more reliably than win rate alone.

---

## Trader Segments

| Segment | Traders | Median PnL | Win Rate | Consistency |
|---|---|---|---|---|
| High Leverage | 16 | $120,598 | 73.3% | 58.9% |
| Low Leverage | 16 | $114,961 | 67.2% | 45.0% |
| Frequent | 16 | $129,522 | 73.3% | 67.0% |
| Infrequent | 16 | $106,037 | 69.9% | 47.6% |
| Consistent Winner | 13 | $168,658 | 83.4% | 77.1% |
| Profitable but Inconsistent | 16 | $117,655 | 58.9% | 47.6% |
| Unprofitable | 3 | -$70,436 | 72.9% | 42.4% |

---

## Behavioral Archetypes (KMeans Clustering)

Silhouette score selected k=2 as optimal.

**Archetype A — High-Output Specialists (2 traders)**
457 trades/day, $1.27M median PnL, 86% win rate, 19x leverage. Frequency and consistency far exceed the rest of the dataset. Almost certainly algorithmic or semi-automated traders.

**Archetype B — Passive Participants (30 traders)**
58 trades/day, $107k median PnL, 71.5% win rate, 31x leverage. Active human traders who compensate for lower frequency with higher leverage.

---

## Predictive Model

A logistic regression model predicts whether a trader will be profitable the next day using today's behavior and sentiment as features.

| Metric | Value |
|---|---|
| ROC-AUC | 0.6913 |
| Accuracy | 71% |
| Training samples | 1,023 |
| Test samples | 256 |

**Top predictors:**
- Fear/Greed index value (+0.49) — strongest positive predictor
- Sentiment direction / Greed label (-0.30) — negatively correlated, suggesting Greed days can precede pullbacks
- Win rate (+0.24) and trade frequency (+0.23) both improve next-day profitability odds

The raw Fear/Greed index value is the strongest predictor of next-day profitability, while the binary sentiment direction is negatively correlated — indicating that the magnitude of sentiment matters more than its label alone.

Note: The model is conservative and biased toward predicting profitable outcomes (recall of 12% for the Not Profitable class), reflecting the class imbalance in the dataset (69% profitable days).

---

## Strategy Recommendations

**Rule 1 — On Fear Days: Trade more, size down**
Frequent traders earn $330 median PnL on Fear days vs just $1 for infrequent traders. The edge on Fear days comes from activity and adaptability, not from large bets. Increase number of trades, reduce per-trade USD exposure.

**Rule 2 — On Greed Days: Scale up position size**
Low leverage traders see a $230 Greed-day lift vs only $70 for high leverage traders. Greed days are trending markets — even modest sizing captures the move effectively. Increase position size and prioritize trend-following setups.

**Rule 3 — Manage risk/reward, not win rate**
Unprofitable traders win 72.9% of trades but still lose money. Consistent Winners win 83.4% and are robustly profitable. The differentiator is not entry skill — it is cutting losses faster and letting winners run. Implement hard stop-losses and asymmetric position sizing.

---

