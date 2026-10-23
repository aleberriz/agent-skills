---
name: analytics-forecasting
description: "Time series analysis and forecasting conventions grounded in Hyndman and Athanasopoulos's Forecasting: Principles and Practice (Python fpppy edition first, R fpp3 second). Treats every forecast as a probability distribution, not a single number, and pairs it with the analysis that explains the series: time plots, transformations, decomposition, benchmarks, explicit train/test splits, residual diagnostics, and point plus distributional accuracy. Prefers explainable models unless a complex one is clearly better. Use whenever the user asks to forecast, predict, project or extrapolate a quantity over time (revenue, signups, subscribers, demand, traffic, churn), asks what a metric will be next week, month, quarter or year, wants trend, seasonality or decomposition analysis, compares ETS, ARIMA, Prophet, regression or machine learning forecasters, evaluates forecast accuracy or prediction intervals, or uses statsforecast, mlforecast, fable or feasts, even when they only want a single number."
license: Apache-2.0
---

# Time Series Analysis and Forecasting

Conventions for forecasting work that is honest about uncertainty and explains the series it forecasts. The method follows Hyndman, Athanasopoulos and co-authors, *Forecasting: Principles and Practice*:

- **Primary:** [the Pythonic way (fpppy)](https://otexts.com/fpppy/), the default stack for code.
- **Secondary:** [the R edition, 3rd ed. (fpp3)](https://otexts.com/fpp3/), for R/Quarto work and for its didactic graphics.

The books are the source of truth. This skill encodes the workflow and judgment, and links to chapters rather than restating them. If the live book contradicts something here, follow the book and tell the user the skill needs a sync (see [Keeping in sync](#keeping-in-sync-with-the-book)).

## Principle 1: a forecast is a distribution

The future value is a random variable ([fpppy 1.7](https://otexts.com/fpppy/01-intro.html#sec-perspective)). A forecast is the probability distribution of that variable, given what we know. The point forecast is one summary of it, usually the mean. Reporting only the mean throws away what most decisions need: how wrong it could plausibly be, and in which direction.

- **Always deliver the distribution.** Every forecast table carries the point forecast plus prediction intervals (default 80% and 95%, the book's convention). Every forecast chart shows intervals as shaded bands, never a lone line.
- **Name the summary** ([5.6](https://otexts.com/fpppy/05-toolbox.html#bias-adjustments)). State whether the point is the mean or the median. After a log or Box-Cox transform, the plain back-transform gives the median; apply bias adjustment when the mean is needed (budgets, totals) and say which one you report.
- **Name the interval method and its assumptions** ([5.5](https://otexts.com/fpppy/05-toolbox.html#sec-prediction-intervals)). Model-based intervals assume uncorrelated, constant-variance, normal residuals. Bootstrapped intervals drop normality. Conformal intervals drop most distributional assumptions but need enough history to calibrate. Pick the method the residual diagnostics support, and say why.
- **Sanity-check the intervals.** They should widen with the horizon unless the smoothing parameters are near zero, and for multiplicative models they should scale with the seasonal level. If they don't, check the library before trusting them (see the known issue in [references/python-fpppy.md](references/python-fpppy.md)).
- **Say what the intervals leave out.** Model-based intervals ignore parameter and model-selection uncertainty, so they are usually too narrow, more so at long horizons. Put this in the caveats.
- **Score the distribution, not only the point.** Report a distributional score (quantile score, Winkler score or CRPS) next to point scores, and check empirical interval coverage on the test set.
- **Match the summary to the decision.** When costs of over- and under-forecasting differ (inventory, staffing, capacity, revenue commitments), the right number is a quantile, not the mean. Say so when it applies.
- **Write it as a range.** "We expect about 1,200 signups in March, with an 80% chance of landing between 1,050 and 1,370." Not "the forecast is 1,200."

## Principle 2: explain the series, then forecast it

A forecast that nobody understands is hard to trust and hard to act on. Every chapter of the book exists because skipping it produces worse or less trustworthy forecasts. Even when the user asks only for a number, deliver the analysis that explains the series, scaled to the request (see [Depth](#depth-scale-the-write-up-not-the-rigor)).

**Prefer the explainable model.** Fit candidates from the top of this ladder down, and stop climbing when the next rung stops paying for itself:

1. Benchmarks: mean, naive, seasonal naive, drift.
2. Decomposition-based and exponential smoothing (ETS): level, trend and season a stakeholder can see.
3. Regression with interpretable predictors (trend, seasonal dummies or Fourier terms, holidays, prices, marketing spend), including dynamic regression with ARIMA errors.
4. ARIMA, especially high-order or seasonal.
5. Machine learning (gradient boosting via `mlforecast`), neural networks (Ch 14) and foundation models (Ch 15).

A simple average of a few explainable models is often hard to beat ([13.4](https://otexts.com/fpppy/13-practical.html#sec-combinations)) and stays explainable.

**The decision rule.** Keep the more explainable model unless the more complex one beats it in most cross-validation windows by more than the window-to-window spread of the error. Report both models' scores either way, and state in one sentence what was gained or given up. A single test split is too noisy to justify losing explainability.

**Make every choice explicit.** The reader should be able to reproduce the analysis from the write-up alone: transformation used or not and why, decomposition method and settings, exact train and test periods, cross-validation setup, candidates compared, the winner and why, and what the residuals say.

## Workflow

Follow the book's order. Each step names what to produce and what to state. The per-chapter tips live in [references/chapter-notes.md](references/chapter-notes.md); read the matching section when a step is not routine.

1. **Frame the problem** ([Ch 1](https://otexts.com/fpppy/01-intro.html)). State what is forecast, at what frequency, over what horizon, and for which decision. Judge predictability: how well the drivers are understood, how much history exists, whether the future resembles the past, and whether the forecast itself changes the outcome.
2. **Look at the data** ([Ch 2](https://otexts.com/fpppy/02-graphics.html)). Time plot first. Then seasonal and subseries plots, lag plots and the autocorrelation function (ACF). Describe trend, seasonal period, cycles, level shifts, outliers and gaps in words. Check whether the series is distinguishable from white noise.
3. **Adjust and transform** ([Ch 3.1](https://otexts.com/fpppy/03-decomposition.html#sec-transformations)). Consider calendar adjustments (per trading day), population (per capita), inflation (real terms) and composition. Use a Box-Cox or log transform only when variance grows with the level, or a log to keep forecasts positive. Choose lambda with the Guerrero method on the training data only, round it to an interpretable value (0 for log, 0.5 for square root, 1 for none) when close, and state the choice: "Box-Cox lambda = 0.1 (Guerrero), because seasonal swings grow with the level" or "No transformation, because variance is stable across the range."
4. **Decompose** ([Ch 3](https://otexts.com/fpppy/03-decomposition.html#sec-stl), [Ch 4](https://otexts.com/fpppy/04-features.html)). Use STL by default, robust when outliers exist, and state the trend and seasonal windows. Plot the components and describe each. Quantify trend and seasonal strength. Inspect the remainder for leftover seasonality, outliers and structural breaks, and say what you found. For multiplicative patterns, log first, then decompose additively.
5. **Split explicitly** ([5.8](https://otexts.com/fpppy/05-toolbox.html#sec-accuracy), [5.10](https://otexts.com/fpppy/05-toolbox.html#sec-tscv)). Hold out the most recent data as the test set, at least as long as the forecast horizon. State the exact first and last dates and the number of observations in each set. Prefer time series cross-validation for model selection and state its initial window, step, number of windows and horizon. Never shuffle. Never let the test set inform any choice, including lambda, outlier fixes and model orders.
6. **Benchmark first** ([5.2](https://otexts.com/fpppy/05-toolbox.html#sec-simple-methods)). Fit mean, naive, seasonal naive and drift. A model that does not beat the best benchmark is not worth explaining.
7. **Fit candidates** (Ch [7](https://otexts.com/fpppy/07-regression.html), [8](https://otexts.com/fpppy/08-exponential-smoothing.html), [9](https://otexts.com/fpppy/09-arima.html), [10](https://otexts.com/fpppy/10-dynamic-regression.html), [12](https://otexts.com/fpppy/12-advanced.html), [14](https://otexts.com/fpppy/14-neural-networks.html), [15](https://otexts.com/fpppy/15-foundation-models.html)). Compare models within a family by AICc. Compare across families, and across ARIMA models with different differencing, only by out-of-sample accuracy, because their likelihoods are not comparable.
8. **Diagnose residuals** ([5.3](https://otexts.com/fpppy/05-toolbox.html#sec-residuals), [5.4](https://otexts.com/fpppy/05-toolbox.html#sec-diagnostics)). Plot innovation residuals over time, their ACF and their histogram. They must be uncorrelated with zero mean; constant variance and normality matter for the intervals. Run a Ljung-Box test (lag 10 for non-seasonal, 2m for seasonal period m, capped at T/5) and report the statistic, degrees of freedom and p-value. Correlated or biased residuals mean the model missed something: improve it. Non-normal or changing-variance residuals mean model-based intervals are unreliable: switch to bootstrap or conformal intervals. Residual checks show whether a model can be improved; they do not choose between models.
9. **Evaluate accuracy** ([5.8](https://otexts.com/fpppy/05-toolbox.html#sec-accuracy), [5.9](https://otexts.com/fpppy/05-toolbox.html#sec-distaccuracy)). Use a scaled point measure (MASE or RMSSE) to compare across series and models, MAE or RMSE in the original units to interpret, and a distributional score plus interval coverage. Look at accuracy by horizon, not only on average. Avoid MAPE when values are near zero or the scale has no natural zero.
10. **Choose by the decision rule** above, and state it.
11. **Forecast** with the chosen model refit on all data. Produce the full distribution, the fan chart and the forecast table.

Watch for structures that change the method:

- **Many related series** that add up (products to total, regions to country): forecast all levels and reconcile ([Ch 11](https://otexts.com/fpppy/11-hierarchical-forecasting.html)). Never sum independent forecasts' intervals.
- **Little or no data, or known upcoming events**: use structured judgmental methods, adjust sparingly and record every adjustment ([Ch 6](https://otexts.com/fpppy/06-judgmental.html)).
- **Holidays, launches, campaigns and other known events**: model them as regressors where the model allows (dummies in regression or dynamic regression, Ch 7 and 10; exogenous features in `mlforecast`). ETS takes no regressors, so either pick a model that does, or adjust the affected days before fitting and add the effect back afterwards. Disclose the choice either way, because a recurring holiday dip left in the data drags the estimated level.
- **Annual seasonality in daily or weekly data with only a few years of history**: an annual component estimated from fewer than about four cycles is mostly noise. Use low-order Fourier terms, or let the trend absorb it, and say why.
- **Weekly, daily or sub-daily data, counts, intermittent demand, bounded values, missing values, outliers, very short or very long series, combinations**: see the practical issues chapter ([Ch 13](https://otexts.com/fpppy/13-practical.html)) before fitting.

## Depth: scale the write-up, not the rigor

The amount of prose scales with the request; the analysis behind it does not shrink.

- **Quick ask** ("what will signups be next month?", "just the numbers"): lead with the forecast table or range, one fan chart with history, then one short paragraph (aim for under about 250 words outside the table) on what drives the series (trend and seasonality from the decomposition), the transformation decision, the train and test periods, the chosen model against the best benchmark, and the residual verdict. Keep the full analysis in the notebook.
- **Analysis or report**: use the full template below.

At every depth, never skip: prediction intervals, the benchmark comparison, explicit train and test periods, the transformation statement, and the residual check.

## Report template

Adapt headings to the medium, but keep every element. Write the prose per [process-business-writing](../process-business-writing/SKILL.md) and build the charts per [analytics-data-storytelling](../analytics-data-storytelling/SKILL.md).

```
# [Takeaway title: what will happen, as a range]

## TL;DR
Range and point for the horizon that matters, the main driver, and the
biggest caveat. One to three sentences.

## The series
What it measures, source, frequency, span. Time plot with annotations for
breaks, outliers and events.

## What drives it
Decomposition: trend, seasonality (period, strength), remainder. Adjustments
and transformation, with the reason. Anything unusual and how it was handled.

## How we tested
Train: [first date] to [last date] (n = ...). Test: [first] to [last] (n = ...).
Cross-validation: initial window, step, windows, horizon.
Models compared, benchmarks included, with point and distributional scores.
Which model won, and the explainability trade-off in one sentence.

## Diagnostics
Residual plots, Ljung-Box result, interval method and whether its assumptions hold.

## Forecast
Fan chart with history. Table: period, point (mean or median), 80% and 95% bounds.

## Caveats
What the intervals ignore, what would change the forecast, when to refit.
```

## Charts

Apply [analytics-data-storytelling](../analytics-data-storytelling/SKILL.md) to every chart and its prose. Forecast-specific conventions:

- **Fan chart.** History as a solid line; forecast mean as a line in the same hue; 80% and 95% intervals as nested bands, darker for narrower. Label the bands directly.
- **Mark the cut.** Draw a labeled vertical rule at the train/test boundary and at the forecast origin. When showing test performance, overlay actual test values on the forecast bands.
- **Show enough history.** At least three seasonal cycles where available, so the reader can judge whether the forecast continues the pattern.
- **Decomposition panels.** Share the x-axis. Components have different y-scales; say so in the subtitle or use scale bars like fpp3, so a small seasonal swing is not read as large.
- **Takeaway titles.** "Signups should keep growing about 4% a year, with a December dip", not "STL decomposition of signups".
- **Caption.** Name the model, the interval levels and method, and the training window: "Forecast from ETS(M,A,M) trained on Jan 2019 to Dec 2025; shaded bands are 80% and 95% prediction intervals." For non-technical readers, describe the model in words ("a model of trend and seasonality fitted to 2019 to 2025") and keep the specification for the appendix.
- **Diagnostics stay technical.** Residual panels belong in the appendix or notebook, not the stakeholder summary, unless they change the story.

For non-technical audiences, lean on the most didactic views: seasonal and subseries plots, decomposition components, and ETS components. These explain the "why" without equations.

## Tooling

- **Python (default):** a Jupyter notebook using the fpppy stack. Code patterns in [references/python-fpppy.md](references/python-fpppy.md).
- **R (when asked, or when the project is already in R):** a Quarto `.qmd` using the fpp3 stack (tsibble, feasts, fable). Code patterns in [references/r-fpp3.md](references/r-fpp3.md).

Use the libraries the books use, so the code maps one-to-one to the chapters the user knows. For publication charts, follow the data storytelling skill's tool escalation (plotnine or ggplot2 first); the libraries' built-in plots are fine for exploration.

## Anti-patterns

Reject these on sight:

- **A point forecast alone**, or a chart with a single forecast line and no bands.
- **Calling the back-transformed median the mean**, or summing medians across periods or series.
- **Shuffled or random train/test splits**, or tuning anything on the test set.
- **In-sample fit reported as accuracy.**
- **No benchmarks**, or jumping straight to Prophet, gradient boosting or neural nets.
- **A silent transformation**, or a silent decision not to transform.
- **AIC compared across model families** or across ARIMA models with different differencing.
- **MAPE on data near zero.**
- **Summing intervals** across periods or series to get an aggregate interval. Simulate sample paths or reconcile instead.
- **Ignoring residual autocorrelation** because the accuracy number looks good.
- **Unadjusted calendar effects**, such as comparing monthly totals across months of different length.

## Quality checklist

Before delivering a forecast, verify:

- [ ] The forecast is reported as a distribution: point (named as mean or median) plus 80% and 95% intervals, with the interval method stated.
- [ ] The time plot and the seasonal views were inspected and described in words.
- [ ] The transformation decision is stated with its reason, including "none".
- [ ] The decomposition is shown, and its components and remainder are interpreted.
- [ ] Train and test periods are stated with exact dates and sizes; cross-validation settings are stated.
- [ ] Benchmarks were fit, and the chosen model beats the best one.
- [ ] Residual diagnostics were run, with the Ljung-Box result reported and interpreted.
- [ ] Point and distributional accuracy are both reported.
- [ ] The model choice follows the explainability rule, and the trade-off is stated.
- [ ] Charts and prose follow the data storytelling and business writing skills.

## Keeping in sync with the book

Both books are living documents. [references/chapter-notes.md](references/chapter-notes.md) records the date it was last synced against each book, its chapter list with links, and the tips drawn from each chapter. To update:

1. Compare the chapter list in the notes with the live table of contents of fpppy (and fpp3).
2. Re-read changed or new chapters and update the notes and the Python and R reference files.
3. Update this file only if a principle, workflow step or anti-pattern changed.
4. Bump the sync date.

## Interaction with other skills

- **[analytics-data-storytelling](../analytics-data-storytelling/SKILL.md)** governs chart design, narrative and uncertainty communication. This skill adds forecast-specific conventions on top.
- **[process-business-writing](../process-business-writing/SKILL.md)** governs the prose of the write-up.
- **`tooling-python`** (planned) will carry environment and notebook mechanics. This skill carries the forecasting libraries.
- **`analytics-ml`** (planned) covers general supervised learning. Forecasting with machine learning models lives here, because time series need temporal splits and distributional output.

## References

- Hyndman, R.J., Athanasopoulos, G., Garza, A., Challu, C., Mergenthaler, M., and Olivares, K.G. *Forecasting: Principles and Practice, the Pythonic Way*. OTexts. <https://otexts.com/fpppy/>
- Hyndman, R.J., and Athanasopoulos, G. *Forecasting: Principles and Practice*, 3rd ed. OTexts, 2021. <https://otexts.com/fpp3/>
- [Anthropic Agent Skills](https://github.com/anthropics/skills): the skill format this document follows.
