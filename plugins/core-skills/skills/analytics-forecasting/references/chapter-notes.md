# Chapter notes

The tips from each chapter of *Forecasting: Principles and Practice* that a forecasting analysis should not skip. Read the section matching the workflow step at hand; follow the links for the full treatment.

## Sync status

| Book | URL | Last synced | What to watch for changes |
|---|---|---|---|
| fpppy (Python, primary) | <https://otexts.com/fpppy/> | 2026-10-23 (preface dated May 2026) | Chapter list below; pinned library versions in [requirements.txt](https://otexts.com/fpppy/data/requirements.txt). No public changelog; errata in [OTexts GitHub Discussions](https://github.com/orgs/OTexts/discussions). |
| fpp3 (R, secondary) | <https://otexts.com/fpp3/> | 2026-10-23 (fpp3 meta-package 1.0.3) | [Changelog](https://otexts.com/fpp3/changelog.html). |

The "last updated" date on both sites changes on every rebuild, so it does not signal content changes. Compare the chapter and section lists instead.

URL pattern: fpppy is `https://otexts.com/fpppy/NN-slug.html#anchor`; fpp3 is `https://otexts.com/fpp3/slug.html#anchor`. Section numbers 1 to 13 match between the books.

## Differences between the books

- **Only in fpppy:** Ch 14 neural networks, Ch 15 foundation models, and conformal prediction in 5.5.
- **Only in fpp3:** 12.4 neural network autoregression (NNETAR), 13.5 [prediction intervals for aggregates](https://otexts.com/fpp3/practical.html#sec-aggregates), 13.8 [forecasting on training and test sets](https://otexts.com/fpp3/practical.html#sec-training-test).
- **Distributions:** fable returns a distribution object per forecast and bias-adjusts back-transformations automatically. fpppy returns point and interval columns and bias-adjusts by hand.
- **Graphics:** fpp3 now draws with `ggtime` (seasonal and subseries plots built in). fpppy builds those plots by hand with matplotlib and seaborn.

## 1. Getting started ([fpppy](https://otexts.com/fpppy/01-intro.html), [fpp3](https://otexts.com/fpp3/intro.html))

- Five steps: define the problem, gather information, explore, choose and fit models, use and evaluate. Problem definition is often the hardest.
- Predictability depends on how well the drivers are understood, how much data exists, how similar the future is to the past, and whether the forecast affects the outcome.
- [1.7](https://otexts.com/fpppy/01-intro.html#sec-perspective): the thing being forecast is a random variable; the forecast is its distribution given what we know; the point forecast is its mean; spread grows with the horizon. This is the basis of Principle 1.

## 2. Time series graphics ([fpppy](https://otexts.com/fpppy/02-graphics.html), [fpp3](https://otexts.com/fpp3/graphics.html))

- Always start by plotting the data.
- Distinguish trend, seasonality (fixed period) and cycles (no fixed period, usually longer) ([2.3](https://otexts.com/fpppy/02-graphics.html#sec-tspatterns)).
- [Seasonal](https://otexts.com/fpppy/02-graphics.html#seasonal-plots) and [subseries](https://otexts.com/fpppy/02-graphics.html#sec-subseries) plots expose the seasonal pattern and changes in it; they are the best stakeholder views.
- In the ACF, trend shows as slowly decaying positive spikes, seasonality as peaks at multiples of the period. For [white noise](https://otexts.com/fpppy/02-graphics.html#sec-wn), 95% of spikes fall within ±1.96/√T.

## 3. Time series decomposition ([fpppy](https://otexts.com/fpppy/03-decomposition.html), [fpp3](https://otexts.com/fpp3/decomposition.html))

- [3.1](https://otexts.com/fpppy/03-decomposition.html#sec-transformations): adjust for calendar, population, inflation and composition before modelling. Choose Box-Cox lambda so seasonal variation is roughly constant; Guerrero automates it. Simple values are easier to explain.
- Prefer STL over classical decomposition: it handles any seasonal period, lets seasonality change over time, and has a robust option for outliers ([3.6](https://otexts.com/fpppy/03-decomposition.html#sec-stl)).
- Default STL windows may need tuning. If a known event leaks into the remainder, shorten the trend window.
- Seasonally adjusted series contain the remainder, so they are not smooth; do not read turning points from them.

## 4. Time series features ([fpppy](https://otexts.com/fpppy/04-features.html), [fpp3](https://otexts.com/fpp3/features.html))

- Trend strength and seasonal strength ([4.3](https://otexts.com/fpppy/04-features.html#sec-stlfeatures)) turn the decomposition into two numbers between 0 and 1 that are easy to report.
- With many series, compute features for all of them to find the unusual ones before modelling.

## 5. The forecaster's toolbox ([fpppy](https://otexts.com/fpppy/05-toolbox.html), [fpp3](https://otexts.com/fpp3/toolbox.html))

The chapter most often skipped, and the one that makes a forecast trustworthy.

- **Workflow:** tidy, visualize, specify, estimate, evaluate, forecast.
- **Benchmarks** ([5.2](https://otexts.com/fpppy/05-toolbox.html#sec-simple-methods)): mean, naive, seasonal naive, drift. A method that does not beat them is not worth considering.
- **Residuals** ([5.3](https://otexts.com/fpppy/05-toolbox.html#sec-residuals), [5.4](https://otexts.com/fpppy/05-toolbox.html#sec-diagnostics)): innovation residuals must be uncorrelated with zero mean; constant variance and normality are useful but not necessary. Ljung-Box with lag 10 (non-seasonal) or 2m (seasonal), at most T/5. Residual checks show whether a model can be improved; they do not choose between models.
- **Prediction intervals** ([5.5](https://otexts.com/fpppy/05-toolbox.html#sec-prediction-intervals)): point forecasts can be of almost no value without intervals. When residuals are not normal, use [bootstrapped](https://otexts.com/fpppy/05-toolbox.html#prediction-intervals-from-bootstrapped-residuals) or [conformal](https://otexts.com/fpppy/05-toolbox.html#conformal-prediction-for-distribution-free-prediction-intervals) intervals.
- **Back-transformation** ([5.6](https://otexts.com/fpppy/05-toolbox.html#bias-adjustments)): the back-transformed point is the median. Bias-adjust to get the mean, which is what adds up across periods or regions.
- **Forecasting with decomposition** ([5.7](https://otexts.com/fpppy/05-toolbox.html#sec-forecasting-decomposition)): forecast the seasonally adjusted series and the seasonal component separately; easy to explain.
- **Point accuracy** ([5.8](https://otexts.com/fpppy/05-toolbox.html#sec-accuracy)): hold out about 20% of the data, at least the maximum horizon. Residual accuracy understates true error. MAE and RMSE are scale-dependent; MASE and RMSSE compare across series. MAPE needs a meaningful zero and fails near zero; avoid sMAPE.
- **Distributional accuracy** ([5.9](https://otexts.com/fpppy/05-toolbox.html#sec-distaccuracy)): quantile score, Winkler score for intervals, CRPS for the whole distribution, and skill scores relative to a benchmark.
- **Time series cross-validation** ([5.10](https://otexts.com/fpppy/05-toolbox.html#sec-tscv)): rolling forecast origin; also look at accuracy by horizon.

## 6. Judgmental forecasts ([fpppy](https://otexts.com/fpppy/06-judgmental.html), [fpp3](https://otexts.com/fpp3/judgmental.html))

- Use when there is no data (new product, policy change) or to add information the data lacks.
- Set the task clearly, be systematic, document and justify, and keep forecasters separate from users.
- [Adjust](https://otexts.com/fpppy/06-judgmental.html#sec-judgmental-adjustments) statistical forecasts sparingly, only with real outside information, never to "fix" patterns the model supposedly missed. Record every adjustment.

## 7. Time series regression ([fpppy](https://otexts.com/fpppy/07-regression.html), [fpp3](https://otexts.com/fpp3/regression.html))

- [Useful predictors](https://otexts.com/fpppy/07-regression.html#sec-useful-predictors): trend, seasonal dummies, Fourier terms, interventions, trading days, distributed lags, holidays.
- Check residuals against time, fitted values and each predictor. Watch for spurious regression between trending series.
- Distinguish ex-ante forecasts (predictors themselves forecast) from ex-post; scenario forecasts ignore predictor uncertainty, so say so.
- Correlation is not causation ([7.8](https://otexts.com/fpppy/07-regression.html#sec-causality)).

## 8. Exponential smoothing ([fpppy](https://otexts.com/fpppy/08-exponential-smoothing.html), [fpp3](https://otexts.com/fpp3/exponential-smoothing.html))

- ETS(Error, Trend, Season) state-space models give the smoothing forecasts plus intervals and AICc selection ([taxonomy](https://otexts.com/fpppy/08-exponential-smoothing.html#sec-taxonomy)).
- The components (level, slope, season) are directly explainable; plot them for stakeholders.
- Damped trends are often more accurate at long horizons. Multiplicative errors require strictly positive data.

## 9. ARIMA ([fpppy](https://otexts.com/fpppy/09-arima.html), [fpp3](https://otexts.com/fpp3/arima.html))

- Procedure: plot, transform if needed, difference until stationary (KPSS), inspect ACF and PACF, choose by AICc, check residuals. Automatic selection covers only part of it.
- AICc cannot compare models with different differencing orders; use a test set.
- [ARIMA vs ETS](https://otexts.com/fpppy/09-arima.html#sec-arima-ets): ARIMA is not more general than ETS; compare them on out-of-sample accuracy only.

## 10. Dynamic regression ([fpppy](https://otexts.com/fpppy/10-dynamic-regression.html), [fpp3](https://otexts.com/fpp3/dynamic-regression.html))

- Regression with ARIMA errors keeps interpretable predictor effects while handling autocorrelation.
- [Dynamic harmonic regression](https://otexts.com/fpppy/10-dynamic-regression.html#sec-dhr) (Fourier terms) handles long seasonal periods such as weekly data with annual seasonality.
- Deterministic and stochastic trends give similar point forecasts but very different intervals.
- Future values of predictors must be known or forecast; their uncertainty is not in the intervals.

## 11. Hierarchical and grouped series ([fpppy](https://otexts.com/fpppy/11-hierarchical-forecasting.html), [fpp3](https://otexts.com/fpp3/hierarchical-forecasting.html))

- When series add up, forecasts should too. Bottom-up, top-down and middle-out each lose information; [reconciliation](https://otexts.com/fpppy/11-hierarchical-forecasting.html#sec-reconciliation) (MinT) uses all levels and usually improves accuracy.
- Reconcile probabilistic forecasts, not only points ([11.5](https://otexts.com/fpppy/11-hierarchical-forecasting.html#sec-rec-prob)).

## 12. Advanced methods ([fpppy](https://otexts.com/fpppy/12-advanced.html), [fpp3](https://otexts.com/fpp3/advanced.html))

- [Complex seasonality](https://otexts.com/fpppy/12-advanced.html#sec-complexseasonality): MSTL or Fourier terms for daily and sub-daily data with several periods.
- [Prophet](https://otexts.com/fpppy/12-advanced.html#sec-prophet) is convenient but did worse than ETS and ARIMA in the book's example; benchmark it like anything else.
- VAR for series that influence each other; bagging with block bootstrap can improve accuracy at a cost to explainability.

## 13. Practical issues ([fpppy](https://otexts.com/fpppy/13-practical.html), [fpp3](https://otexts.com/fpp3/practical.html))

- [Weekly, daily and sub-daily data](https://otexts.com/fpppy/13-practical.html#sec-weekly): non-integer and multiple seasonal periods; use Fourier terms or MSTL.
- [Counts](https://otexts.com/fpppy/13-practical.html#sec-counts): Croston's method for intermittent demand.
- [Limits](https://otexts.com/fpppy/13-practical.html#sec-limits): keep forecasts positive with a log transform, or within bounds with a scaled logit.
- [Combinations](https://otexts.com/fpppy/13-practical.html#sec-combinations): a simple average of a few models is hard to beat, and remains explainable if its members are.
- [Short and long series](https://otexts.com/fpppy/13-practical.html#sec-long-short-ts): there is no magic minimum sample size; very long series may need a shorter window.
- [Missing values and outliers](https://otexts.com/fpppy/13-practical.html#sec-missing-outliers): statsforecast needs complete series; do not replace outliers without understanding why they occurred.

## 14. Neural networks (fpppy only: [link](https://otexts.com/fpppy/14-neural-networks.html))

- Global models learn across many series; they need scaling, probabilistic losses for distributional output, and hyperparameter tuning.
- Exogenous covariates are static, historic or future; only future-known covariates can drive forecasts.
- Least explainable rung; must clear the decision rule against ETS or ARIMA.

## 15. Foundation models (fpppy only: [link](https://otexts.com/fpppy/15-foundation-models.html))

- Zero-shot forecasting with pretrained models (TimeGPT, Chronos, Moirai, TimesFM and others); the book calls them early-stage.
- Strengths: ease of use and limited data. Weaknesses: opacity, and accuracy that must still be verified by cross-validation against benchmarks.
