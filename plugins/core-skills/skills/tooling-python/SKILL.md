---
name: tooling-python
description: "Python tooling conventions for analytics and data science work: Poetry for dependency management, Jupyter for notebooks, plotnine and Plotly for visualization, pandas for data manipulation. Use this skill when setting up a Python analytics environment, creating notebooks, managing dependencies with Poetry, choosing visualization libraries, or working with data in Python. Also apply when the user asks about Python project structure for data work, virtual environments, or notebook best practices."
---

# Python Analytics

*This skill is planned but not yet implemented.*

Universal Python conventions for analytics work, independent of domain. Domain-specific Python patterns (e.g., statsmodels for experimentation, scikit-learn for ML) belong in the corresponding domain skills.

When complete, this skill will cover:

- Poetry setup and `pyproject.toml` conventions for analytics projects
- Jupyter notebook discipline (clear outputs before commit, synthetic/public data only, restart-and-run-all before merge)
- Visualization library selection: Plotly for interactive/exploratory, plotnine (ggplot2) for static/publication
- pandas and DuckDB patterns for local data analysis
- Environment isolation and reproducibility

## References

- [Anthropic Agent Skills](https://github.com/anthropics/skills): check for updated patterns and templates
- [Poetry](https://python-poetry.org/): dependency management
- [plotnine](https://plotnine.readthedocs.io/): grammar of graphics for Python
- [Plotly](https://plotly.com/python/): interactive visualization
