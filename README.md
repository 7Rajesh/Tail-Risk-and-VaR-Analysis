# NIFTY 50 Tail Risk and VaR Analysis

Value at Risk (VaR) estimation and non-normality tests on NIFTY 50 daily returns, with event studies on the 2008 financial crisis and the 2020 COVID-19 crash.

## Overview

Standard risk models assume returns follow a bell curve. Equity returns have fat tails, so those models understate how often large losses occur. This project measures that gap on the NIFTY 50 index.

I estimate daily VaR with four methods, count how often actual losses exceeded each estimate, and test the return series for fat tails, skew, autocorrelation, and volatility clustering.

The project measures tail risk. It does not build or test a hedging strategy.

## Key Results

Data: 4,072 daily returns on `^NSEI` from Yahoo Finance, ending April 2024. VaR figures are daily losses in percent. A breach is a day when the actual loss exceeded the VaR estimate.

| Method | 95% VaR | 99% VaR | Breaches at 95% | Breaches at 99% |
|---|---|---|---|---|
| Expected count | n/a | n/a | 203.6 | 40.7 |
| Historical | 1.92 | 3.75 | 203.6* | 40.7* |
| Normal | 2.16 | 3.08 | 160 | 71 |
| GARCH(1,1) | 1.85 | 2.65 | 227 | 76 |
| ARIMA | 2.16 | 3.07 | 156 | 70 |

\*The historical method fits the same sample it is tested on, so it matches the expected count by construction.

**Findings**

- The normal model overstates risk at 95% (2.16% against 1.92% historical) and understates it at 99% (3.08% against 3.75%). It logged 71 breaches at 99% confidence where 41 were expected.
- Q-Q plots show a Student's t distribution fits the returns better than a Gaussian.
- GARCH(1,1) parameters are significant (alpha t-statistic 3.39, beta t-statistic 9.77, p < 0.001), which confirms volatility clustering. Rolling 30-day volatility peaks in 2008 and 2020.
- GARCH gives the lowest mean VaR, but it also logs more breaches than expected at both levels (227 and 76). A lower VaR here means a less conservative one.
- ARIMA adds nothing over the normal model. Lag 1 autocorrelation of daily returns is about 0.05, so there is little to forecast, and its VaR matches the normal estimate.
- Skewness is positive in the 2008 to 2009 window (0.46) and negative in 2020 (-1.42). Kurtosis exceeds the normal value of 3 in the full sample and in both windows.
- In March 2020, trading volume spiked while cumulative abnormal returns (CAR) fell.

![VaR comparison](figures/var_comparison.png)

![Rolling volatility](figures/rolling_volatility.png)

## Repository Structure

```
.
├── 01_var_estimation.ipynb            # Historical, normal, GARCH VaR and breach counts
├── 02_non_normality_analysis.ipynb    # Normality tests, ACF, ARIMA, GARCH, event studies
├── figures/                           # Plots used in this README
├── requirements.txt
└── README.md
```

## Methods

**VaR estimation** (`01_var_estimation.ipynb`)
- Historical simulation: the 5th and 1st percentile of observed daily returns.
- Normal: `VaR = mu - z * sigma` from the sample mean and standard deviation.
- GARCH(1,1): fitted with the `arch` library. Daily conditional volatility feeds a normal quantile.
- ARIMA: models the expected daily return, then applies a normal quantile to the residuals.

**Non-normality analysis** (`02_non_normality_analysis.ipynb`)
- Skewness and kurtosis for the full sample, 2008 to 2009, and 2020.
- Shapiro-Wilk, Kolmogorov-Smirnov, and Anderson-Darling normality tests.
- Gaussian and Student's t fits with probability plots.
- ACF for serial correlation and a 30-day rolling standard deviation for volatility clustering.
- Event studies with abnormal returns, CAR, and trading volume for 2008 and 2020.

## Getting Started

```bash
git clone https://github.com/<your-username>/NIFTY50-Tail-Risk-and-VaR-Analysis.git
cd NIFTY50-Tail-Risk-and-VaR-Analysis
pip install -r requirements.txt
jupyter notebook
```

Run `01_var_estimation.ipynb` first, then `02_non_normality_analysis.ipynb`. Both notebooks download data from Yahoo Finance, so they need an internet connection.

**Requirements:** Python 3.9+, pandas, NumPy, SciPy, statsmodels, arch, Matplotlib, yfinance.

**Data notes**
- The notebooks read the `Adj Close` column. Recent yfinance versions set `auto_adjust=True` by default and drop that column. Pass `auto_adjust=False` to `yf.download` or pin an older version.
- Yahoo Finance history for `^NSEI` starts in late 2007, so the sample opens during the 2008 crisis window.
- Yahoo reports zero volume on some index days. Those days appear as dips in the volume plot.
- Numbers shift slightly if you change the end date or if Yahoo revises its data.

## Limitations

- VaR is estimated and tested on the same full-sample history. A rolling out-of-sample backtest would give a fairer comparison.
- Breach counts are compared with expected counts by eye. No formal test (Kupiec or Christoffersen) is run.
- GARCH VaR uses normal innovations, which undercuts the fat-tail finding. A Student's t innovation would fit better.
- The link between CAR and trading volume is visual. No correlation test is run.

## Next Steps

- Add Kupiec and Christoffersen backtests.
- Fit GARCH with Student's t innovations and compare breach counts.
- Add Expected Shortfall (CVaR) next to VaR.
- Repeat the analysis on the S&P 500 to compare an emerging and a developed market.
- Test a simple hedge, such as protective puts or a volatility-triggered equity cut, against the 2008 and 2020 drawdowns.

## References

- Bollerslev, T., Chou, R. Y., & Kroner, K. F. (1992). ARCH modeling in finance: A review of the theory and empirical evidence. *Journal of Econometrics*.
- Lo, A. W., & MacKinlay, A. C. (1988). Stock market prices do not follow random walks: Evidence from a simple specification test. *The Review of Financial Studies, 1*(1).
- Lo, A., & Wang, J. (2001). Stock market trading volume.
- Ju, X., & Pearson, N. D. (1998). Using Value-at-Risk to control risk taking: How wrong can you be? University Library of Munich.
