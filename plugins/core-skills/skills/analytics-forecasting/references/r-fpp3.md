# R patterns (fpp3 stack)

Use when the user asks for R, or the project is already in R. The deliverable is a Quarto `.qmd` that renders to HTML. Code follows [fpp3](https://otexts.com/fpp3/): `tsibble` for data, `ggtime` for graphics (it replaced `feasts` graphics after the 2021 print edition), `feasts` for features, `fable` for models, `distributional` for forecast distributions. `library(fpp3)` loads them all plus the book's datasets. See the [fpp3 changelog](https://otexts.com/fpp3/changelog.html) for API changes.

## Quarto skeleton

````markdown
---
title: "[Takeaway title as a range]"
format:
  html:
    code-fold: true
    toc: true
execute:
  warning: false
---

```{r}
#| label: setup
library(fpp3)
```
````

Then one section per report-template heading in [SKILL.md](../SKILL.md#report-template). Fold code by default so stakeholders read prose and charts; analysts can expand it.

## Data

```r
y <- readr::read_csv("data.csv") |>
  mutate(Month = yearmonth(date)) |>        # yearquarter(), yearweek(), as_date()
  as_tsibble(index = Month, key = NULL)     # key = series id columns for many series
has_gaps(y); y <- fill_gaps(y)              # make implicit gaps explicit first
```

## Look (Ch 2)

```r
autoplot(y, value)
gg_season(y, value)          # seasonal plot: one line per year
gg_subseries(y, value)       # subseries plot: one panel per season, with means
gg_lag(y, value, geom = "point")
ACF(y, value) |> autoplot()
```

`gg_season` and `gg_subseries` are the most didactic views for non-technical readers.

## Transform and decompose (Ch 3, 4)

```r
lambda <- train |> features(value, features = guerrero) |> pull(lambda_guerrero)
dcmp <- train |> model(STL(box_cox(value, lambda) ~ trend(window = 21) + season(window = "periodic"),
                           robust = TRUE))
components(dcmp) |> autoplot()                # scale bars show relative component size
train |> features(value, feat_stl)            # trend_strength, seasonal_strength_year, ...
```

## Split, benchmark, fit (Ch 5, 7 to 10)

```r
train <- y |> filter(Month <= yearmonth("2023 Dec"))
test  <- y |> filter(Month >  yearmonth("2023 Dec"))

fit <- train |> model(
  mean   = MEAN(value),
  naive  = NAIVE(value),
  snaive = SNAIVE(value),
  drift  = RW(value ~ drift()),
  ets    = ETS(box_cox(value, lambda)),
  arima  = ARIMA(box_cox(value, lambda))
)
report(fit |> select(ets)); tidy(fit |> select(ets))
components(fit |> select(ets)) |> autoplot()   # level, slope, season: good for stakeholders
```

Transformations inside the model formula are back-transformed automatically, with bias adjustment so the point forecast is the mean.

## Diagnose (Ch 5)

```r
fit |> select(ets) |> gg_tsresiduals()          # time plot, ACF, histogram of innovation residuals
augment(fit) |> features(.innov, ljung_box, lag = 24)  # lag = 2m seasonal, 10 non-seasonal
# for ARIMA, pass dof = p + q + P + Q, as in fpp3 Ch 9
```

## Evaluate (Ch 5)

```r
fc <- fit |> forecast(h = nrow(test))
accuracy(fc, y, measures = list(point_accuracy_measures,
                                interval_accuracy_measures,
                                distribution_accuracy_measures))
# MASE, RMSSE, winkler, CRPS, ... ; skill_score(CRPS) compares against a benchmark

# Time series cross-validation
cv <- train |> stretch_tsibble(.init = 36, .step = 3) |>
  model(ets = ETS(value), snaive = SNAIVE(value)) |>
  forecast(h = 12)
cv |> accuracy(y, by = c(".model", ".id"))    # per-window scores, to judge the spread
```

## Forecast and show the distribution

```r
final <- y |> model(ETS(box_cox(value, lambda)))
fc <- final |> forecast(h = "2 years")          # .mean plus a distribution column
fc |> hilo(level = c(80, 95)) |> unpack_hilo(c(`80%`, `95%`))   # table with bounds
fc |> autoplot(y |> filter_index("2019 Jan" ~ .), level = c(80, 95))
final |> generate(h = 24, times = 1000, bootstrap = TRUE)        # sample paths for aggregates
```

Use `generate()` sample paths to get intervals for totals (a year, a quarter) rather than summing interval bounds.

## Beyond a single series

- **Hierarchies (Ch 11):** `aggregate_key()` then `reconcile(min_trace(...))`.
- **Dynamic regression (Ch 10):** `ARIMA(value ~ x1 + fourier(K = 3))`, with future `x` supplied through `new_data`.
- **Complex seasonality (Ch 12):** `STL()` with several `season()` terms, `fourier()` terms in regression, or `fable.prophet::prophet()`.
