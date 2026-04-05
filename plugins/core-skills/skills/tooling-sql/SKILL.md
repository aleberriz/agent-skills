---
name: tooling-sql
description: "SQL formatting and style conventions for analytical work, covering dialect-aware patterns for Redshift, DuckDB, and BigQuery. Use this skill when writing SQL queries, reviewing SQL style, setting up SQL linting rules, or when the user asks about SQL formatting, CTE structure, naming conventions, or performance-aware query patterns. Also apply when writing SQL for dbt models, semantic layer definitions, or ad-hoc analysis."
---

# SQL Style

*This skill is planned but not yet implemented.*

Universal SQL conventions for analytics work, independent of domain. Domain-specific SQL patterns (e.g., MetricFlow metric definitions, Omni view YAML) belong in the corresponding domain skills.

When complete, this skill will cover:

- Formatting conventions: uppercase keywords, lowercase identifiers, trailing commas, CTE structure
- Naming conventions: snake_case for columns, plural for tables, clear prefixes for staging/intermediate/mart layers
- Dialect awareness: Redshift, DuckDB, BigQuery (flag syntax that is not portable)
- Performance-aware patterns: predicate pushdown, sort key alignment, distribution key considerations (Redshift-specific)
- Comment conventions: explain the *why* in SQL comments, not the *what*

## References

- [Anthropic Agent Skills](https://github.com/anthropics/skills): check for updated patterns and templates
- [dbt SQL style guide](https://docs.getdbt.com/best-practices/how-we-style/2-how-we-style-our-sql): dbt Labs' recommended SQL formatting
