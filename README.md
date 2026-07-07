# FRTB Expected Shortfall and PLA Validation Framework

A Python implementation of an FRTB Internal Models Approach (IMA) risk engine, built to calculate regulatory Expected Shortfall for a multi-asset equity portfolio and validate desk eligibility through P&L Attribution testing.
 
## Overview
 
This project builds a full IMA-style risk pipeline for a 50-asset equity portfolio spanning 7 sectors (Technology, Financials, Consumer, Healthcare, Energy, Industrials, Communication). It covers the three pillars that a bank's market risk desk needs under FRTB: computing capital-grade Expected Shortfall, proving the risk model tracks actual P&L well enough to stay IMA-eligible, and backtesting the model against realized outcomes.
 
Five years of daily price data was pulled via `yfinance`, with the 10-year Treasury yield from FRED used as a supporting macro factor. Each asset was mapped to a regulatory liquidity horizon (10, 20, 40, 60, or 120 days) based on its risk factor category, and to a sector ETF proxy used to estimate its systematic beta.
 
## What the project covers
 
**1. Liquidity Horizon Mapping**
Each of the 50 assets is assigned an FRTB liquidity horizon bucket. Most fall into the 10-day bucket, with fewer assets requiring longer horizons, reflecting FRTB's treatment of different risk factor types.
 
**2. Expected Shortfall Calculation**
Daily 97.5% ES is calculated separately for each liquidity horizon bucket, scaled to a 10-day base horizon using the square-root-of-time rule, and then aggregated using the FRTB regulatory formula that combines the base bucket with incremental variance contributions from longer buckets.
 
**3. P&L Attribution (PLA) Test**
Two P&L series are constructed: Hypothetical P&L (HPL), from full revaluation of actual returns, and Risk-Theoretical P&L (RTPL), built from sector-factor sensitivities (the SA-style approximation). Spearman correlation and the Kolmogorov-Smirnov statistic are computed between the two series over a rolling 250-day window and mapped to the Basel traffic-light zones to determine desk eligibility.
 
**4. Daily Backtesting**
A rolling 97.5% ES is calculated over a 250-day window and compared against realized portfolio P&L each day, flagging and counting exceptions where losses breach the ES threshold.
 
**5. SA vs IMA Capital Comparison**
A Standardized Approach capital charge is computed as an undiversified sum of sensitivity-based risk charges, and compared against the IMA capital charge (ES-based, scaled by the regulatory backtesting multiplier) to quantify the capital benefit of using an approved internal model.
 
## Key Results
 
| Metric | Result |
|---|---|
| Aggregated 97.5% Expected Shortfall (10-day) | 0.1117 |
| PLA Spearman correlation | 0.900 (Green zone) |
| PLA Kolmogorov-Smirnov statistic | 0.060 (Green zone) |
| Desk eligibility outcome | Green |
| Backtest exception rate | 1.10% (11 out of 1,004 days) |
| SA capital charge (undiversified) | 1.0279 |
| IMA capital charge | 0.1675 |
| Capital reduction vs SA baseline | 83.7% |
 
The PLA test lands comfortably in the Green zone on both metrics, meaning the desk would retain IMA eligibility under Basel rules. The backtest exception rate of 1.10% sits well under the 2.5% expected under a correctly calibrated 97.5% ES model, and the low exception count keeps the regulatory multiplier at its floor of 1.5. The resulting IMA capital charge is 83.7% lower than the undiversified SA charge, showing the capital efficiency an approved internal model can unlock relative to the standardized approach.
 
## Visuals
 
<!-- Insert: ES contribution by liquidity horizon bucket (bar chart) -->
<img width="700" height="451" alt="image" src="https://github.com/user-attachments/assets/6816e3cb-ba94-442d-8d9c-bc211cf652c1" />

 
<!-- Insert: PLA scatter plot, HPL vs RTPL -->
<img width="556" height="547" alt="image" src="https://github.com/user-attachments/assets/a25ffc67-6b72-41f9-85b6-8f104a9ca0f3" />

 
<!-- Insert: Daily backtesting chart, realized P&L vs rolling ES threshold with exceptions marked -->
<img width="866" height="470" alt="image" src="https://github.com/user-attachments/assets/58566290-ccc4-47ab-b63d-545e4de6cce4" />

 
## Tech Stack
 
- Python (NumPy, pandas, SciPy, Matplotlib)
- `yfinance` for equity and sector ETF price data
- `pandas_datareader` / FRED API for macro rate data
- Statistical tests: Spearman rank correlation, Kolmogorov-Smirnov

## Regulatory Context
 
This project follows the Basel Committee's FRTB framework (BCBS 457/491) for market risk capital, specifically the Internal Models Approach requirements around Expected Shortfall calculation, liquidity horizon scaling, P&L Attribution testing for desk eligibility, and daily backtesting.
