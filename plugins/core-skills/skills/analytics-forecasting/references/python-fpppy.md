# Python patterns (fpppy stack)

Default for forecasting work. Code follows [fpppy](https://otexts.com/fpppy/), which uses the Nixtla libraries: `statsforecast` (statistical models), `mlforecast` (machine learning), `neuralforecast` (deep learning), `hierarchicalforecast` (reconciliation), `utilsforecast` (plotting, preprocessing, evaluation) and `coreforecast` (Box-Cox). Decomposition and diagnostics come from `statsmodels`.

Snippets were run against the versions fpppy pins in its [requirements.txt](https://otexts.com/fpppy/data/requirements.txt): statsforecast 2.1.1, mlforecast 1.1.0, utilsforecast 0.2.17. If an import fails on newer versions, check the library docs before improvising. The book's datasets are in [fpppy_data.zip](https://otexts.com/fpppy/data/fpppy_data.zip).

Deliver a Jupyter notebook whose sections follow the report template in [SKILL.md](../SKILL.md#report-template). Restart and run all before handing it over.

## Data format

Every Nixtla library takes a long data frame with `unique_id`, `ds` and `y`:

```python
import os
os.environ["NIXTLA_ID_AS_COL"] = "true"   # as in every fpppy chapter: keep unique_id as a column

import pandas as pd
df = (pd.read_csv("data.csv", parse_dates=["date"])
        .rename(columns={"date": "ds", "signups": "y"})
        .assign(unique_id="signups")[["unique_id", "ds", "y"]])

from utilsforecast.preprocessing import fill_gaps
df = fill_gaps(df, freq="D")      # make missing timestamps explicit
```

`freq` takes pandas aliases: `"D"`, `"W-SUN"`, `"MS"` (month start), `"QS"`, `"h"`. Seasonal period `m`: 7 for daily data with weekly seasonality, 12 for monthly, 4 for quarterly, 52 for weekly (approximate; see fpppy 13.1).

`statsforecast` models do not accept missing values. Fill gaps deliberately (interpolation, or the seasonal naive value), and state what you did and why. Do not replace outliers without understanding their cause (fpppy 13.7).

## Look (Ch 2)

```python
from utilsforecast.plotting import plot_series
plot_series(df)                                     # quick time plot

from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
plot_acf(df["y"], lags=3 * m)
```

fpppy builds seasonal and subseries plots by hand. Build them in plotnine: a seasonal plot draws `y` against the position within the year, one line per year; a subseries plot facets by month (or weekday) with a horizontal line at each facet's mean. They are the most didactic views for non-technical readers.

## Transform (Ch 3, 5.6)

```python
import numpy as np
from coreforecast.scalers import boxcox_lambda, boxcox, inv_boxcox
lam = boxcox_lambda(train["y"].to_numpy(), method="guerrero", season_length=m)
# non-seasonal data: method="loglik"
```

Fit lambda on the training set only. Round to 0, 0.5 or 1 when close and say so. `statsforecast` models do not transform for you (AutoARIMA's `blambda` raises NotImplementedError), so follow fpppy: transform `y`, model, then back-transform the point and the interval bounds.

```python
train_bc = train.assign(y=boxcox(train["y"].to_numpy(), lam))
fc_bc = sf.forecast(df=train_bc, h=h, level=[80, 95])

# Back-transformed point = median. Bias-adjust to get the mean (fpppy eq. 5.3).
w = fc_bc["AutoETS"]
sigma_h = (fc_bc["AutoETS-hi-80"] - fc_bc["AutoETS-lo-80"]) / (2 * 1.2816)
median = inv_boxcox(w.to_numpy(), lam)
if lam == 0:
    mean = np.exp(w) * (1 + sigma_h**2 / 2)
else:
    mean = (lam * w + 1) ** (1 / lam) * (1 + sigma_h**2 * (1 - lam) / (2 * (lam * w + 1) ** 2))
# interval bounds: inv_boxcox each lo/hi column (quantiles survive monotone transforms)
```

Report the mean when forecasts will be summed (budgets, regional totals); otherwise state that the point is the median. For machine learning models, `mlforecast.target_transforms.LocalBoxCox()` handles the transform.

## Decompose (Ch 3, 4)

```python
from statsmodels.tsa.seasonal import STL
stl = STL(train.set_index("ds")["y"], period=m, robust=True).fit()   # tune seasonal=, trend= windows
stl.plot()                         # trend, seasonal, remainder: y-scales differ, say so

var = np.var
trend_strength    = max(0, 1 - var(stl.resid) / var(stl.trend + stl.resid))
seasonal_strength = max(0, 1 - var(stl.resid) / var(stl.seasonal + stl.resid))
```

Check the remainder: if a known event (a recession, an outage, a launch) shows up there rather than in the trend, the trend window is too long (fpppy 3.6 shows this with the global financial crisis). For several seasonal periods, use `statsmodels.tsa.seasonal.MSTL` or `statsforecast.models.MSTL(season_length=[7, 365])` (fpppy 12.1). With fewer than about four years of daily data, the annual component is mostly noise: decompose with the weekly period only and a longer trend window (for example 91 days), or model the annual pattern with a few Fourier terms (`utilsforecast.feature_engineering.fourier`). For many series, `tsfeatures` computes STL and ACF features in bulk (fpppy Ch 4).

## Split, benchmark and fit (Ch 5, 8, 9)

```python
h = 28
test  = df.groupby("unique_id").tail(h)
train = df.drop(test.index)
print(f"Train {train.ds.min():%Y-%m-%d} to {train.ds.max():%Y-%m-%d} (n={len(train)}); "
      f"test {test.ds.min():%Y-%m-%d} to {test.ds.max():%Y-%m-%d} (n={len(test)})")

from statsforecast import StatsForecast
from statsforecast.models import (HistoricAverage, Naive, SeasonalNaive,
                                  RandomWalkWithDrift, AutoETS, AutoARIMA)
sf = StatsForecast(
    models=[HistoricAverage(), Naive(), SeasonalNaive(season_length=m),
            RandomWalkWithDrift(), AutoETS(season_length=m), AutoARIMA(season_length=m)],
    freq="D",
)
fc = sf.forecast(df=train, h=h, level=[80, 95], fitted=True)
# columns: AutoETS, AutoETS-lo-80, AutoETS-lo-95, AutoETS-hi-80, AutoETS-hi-95, ... (RWD for drift)
fitted = sf.forecast_fitted_values()              # in-sample fitted values, for residuals
```

Write the selected specification in the report; the label carries most of the explanation:

```python
from statsforecast.arima import arima_string
sf.fit(train)
sf.fitted_[0, 4].model_["method"]          # e.g. 'ETS(M,A,M)'   (index = position in models list)
arima_string(sf.fitted_[0, 5].model_)      # e.g. 'ARIMA(1,1,1)(0,1,1)[12]'
```

Regression (Ch 7) in fpppy uses `MLForecast(models=LinearRegression())` with `utilsforecast.feature_engineering.pipeline`, `trend` and `fourier` features; dynamic regression (Ch 10) passes exogenous columns to `AutoARIMA` and future values through `X_df`.

## Diagnose (Ch 5.3, 5.4)

```python
from statsmodels.stats.diagnostic import acorr_ljungbox
resid = (fitted["y"] - fitted["AutoETS"]).dropna()
acorr_ljungbox(resid, lags=[2 * m], model_df=0)   # report lb_stat, lag, lb_pvalue; model_df = ARIMA p+q+P+Q
```

Plot residuals over time, their ACF and their histogram as one three-panel figure. For transformed models, compute residuals on the transformed scale (innovation residuals).

## Evaluate (Ch 5.8 to 5.10)

```python
from utilsforecast.evaluation import evaluate
from utilsforecast.losses import mae, rmse, mase, rmsse, scaled_crps, winkler_score, coverage
from functools import partial

cv = sf.cross_validation(df=train, h=h, n_windows=8, step_size=h, level=[80, 95])
scores = evaluate(
    cv, train_df=train, level=[80, 95],
    metrics=[mae, rmse, partial(mase, seasonality=m), partial(rmsse, seasonality=m),
             scaled_crps, winkler_score, coverage],
)
# one row per (cutoff, metric): per-window scores, ready for the explainability rule
```

For each candidate, report the mean score, the spread across windows, and the number of windows in which it beats the more explainable model. Also look at accuracy by horizon step (fpppy 5.10). fpppy defines its own Winkler and CRPS helpers; the `utilsforecast` versions above give the same quantities.

## Known issue: seasonal ETS intervals in statsforecast 2.1.1

Verified on 2026-10-23 against the analytic variance formulas and against simulated sample paths. Point forecasts are correct; the intervals are not:

- **Additive seasonal models** (A,N,A and A,A,A, including on log data): the seasonal variance term uses the total requested horizon for every step, so short-horizon intervals are too wide and the widths change with `h`.
- **Multiplicative models** (M,A,M and similar): widths stay flat within each seasonal cycle instead of scaling with the seasonal level, so intervals at seasonal peaks are too narrow (about 25 to 30% on the a10 data).
- **Non-seasonal models** (A,N,N and A,A,N) match the formulas.

Status on statsforecast main as of 2026-10-23, none of it released yet: the additive bug is fixed ([#1243](https://github.com/Nixtla/statsforecast/pull/1243)). The multiplicative bug is only partly fixed ([#1218](https://github.com/Nixtla/statsforecast/pull/1218)): `_class3models` still flattens `Mh` in row-major order where R's `c(Mh)` is column-major, which leaves intervals too wide beyond the first seasonal cycle: about 20% for M,A,M and up to 50% for M,Ad,M by the third cycle. Until a release with both fixes is available and pinned, select the specification with `AutoETS`, then take seasonal ETS intervals from statsmodels with the same specification, and say so in the notebook:

```python
from statsmodels.tsa.exponential_smoothing.ets import ETSModel
y = train.set_index("ds")["y"].asfreq("MS")
res = ETSModel(y, error="mul", trend="add", seasonal="mul", seasonal_periods=m).fit(disp=False)
pred = res.get_prediction(start=len(y), end=len(y) + h - 1)
pred.summary_frame(alpha=0.2)                       # mean, pi_lower, pi_upper (80%)
paths = res.simulate(nsimulations=h, repetitions=5000, anchor="end")   # sample paths for totals
```

Do not compare AICc between statsforecast and statsmodels; their likelihood constants differ. Delete this section once the pinned statsforecast version includes both fixes.

## Intervals beyond the normal assumption (Ch 5.5)

When residuals are non-normal or heteroscedastic, use bootstrapped or conformal intervals. fpppy 5.5 simulates bootstrapped sample paths with numpy and explains split conformal prediction. The libraries also offer conformal intervals directly:

```python
from statsforecast.utils import ConformalIntervals
AutoETS(season_length=m, prediction_intervals=ConformalIntervals(n_windows=5, h=h))
```

Conformal intervals need `n_windows * h` observations beyond what the model needs to fit. Name the method in the caption.

## Machine learning, neural and foundation models (Ch 7, 14, 15)

```python
import lightgbm as lgb
from mlforecast import MLForecast
from mlforecast.target_transforms import Differences
from mlforecast.utils import PredictionIntervals

pi = PredictionIntervals(n_windows=5, h=h)
mlf = MLForecast(models={"lgbm": lgb.LGBMRegressor(verbosity=-1)}, freq="D",
                 lags=[1, 7, 14, 28], date_features=["dayofweek", "month"],
                 target_transforms=[Differences([7])])
mlf.fit(train, prediction_intervals=pi)
fc_ml = mlf.predict(h, level=[80, 95])
cv_ml = mlf.cross_validation(df=train, h=h, n_windows=8, step_size=h,
                             prediction_intervals=pi, level=[80, 95])   # intervals need pi here too
```

Machine learning models give no distribution unless asked; always request conformal intervals so they can be scored on distributional metrics. Report feature importance as the minimum explanation.

Neural models (Ch 14, `neuralforecast`) are global models: scale inputs, use a probabilistic loss such as `DistributionLoss`, and tune hyperparameters. Foundation models (Ch 15: TimeGPT, Chronos, Moirai and others) forecast zero-shot; the book calls them early-stage. Both sit at the bottom of the explainability ladder; they must clear the decision rule against ETS or ARIMA.

## Forecast and show the distribution

```python
final = sf.forecast(df=df, h=h, level=[80, 95])     # refit on all data
```

Build the fan chart in plotnine per the chart conventions in [SKILL.md](../SKILL.md#charts): `geom_ribbon` for the 95% then the 80% band, `geom_line` for history and the mean, `geom_vline` at the forecast origin. `plot_series(df, final, level=[80, 95])` is fine for exploration.

For intervals on totals (a month's sum of daily forecasts), simulate sample paths (fpppy 5.5 bootstrapping code) and take quantiles of the simulated sums. Never add interval bounds. fpp3 13.5 covers this; fpppy has no equivalent section.

## Hierarchies (Ch 11)

`hierarchicalforecast`: build the summing matrix with `aggregate`, forecast every level with `statsforecast`, then `HierarchicalReconciliation(reconcilers=[BottomUp(), MinTrace(method="mint_shrink")])`. Reconcile the intervals as well as the points (fpppy 11.5).
