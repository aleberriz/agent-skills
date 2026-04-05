---
name: analytics-ml
description: "Machine learning modeling conventions for applied analytics: feature engineering, model evaluation, validation strategy, and deployment considerations. Use this skill when building predictive models, selecting features, choosing evaluation metrics, setting up train/test splits, or discussing ML methodology for business applications. Also apply when the user asks about classification, regression, churn prediction, forecasting, or any supervised/unsupervised learning task in an analytics context."
---

# ML Modeling

*This skill is planned but not yet implemented.*

Conventions for applied machine learning in analytics contexts: predicting outcomes, scoring users, forecasting metrics. This is the prescriptive/predictive domain, distinct from experimentation (which establishes causation through randomization).

When complete, this skill will cover:

- Feature engineering patterns for tabular business data
- Model evaluation: choosing the right metric for the business question (precision vs. recall, RMSE vs. MAE, AUC vs. F1)
- Validation strategy: time-based splits for temporal data, stratification, cross-validation
- Overfitting discipline: regularization, feature selection, holdout discipline
- Forecasting conventions: time series decomposition, seasonality, trend
- Deployment considerations: batch vs. real-time, monitoring, drift detection

## References

- [Anthropic Agent Skills](https://github.com/anthropics/skills): check for updated patterns and templates
- [scikit-learn](https://scikit-learn.org/): the default ML library for tabular analytics work
- [statsmodels](https://www.statsmodels.org/): statistical modeling in Python
