---
name: analytics-semantic-layer
description: "Conventions for designing and authoring semantic layer definitions: AI context fields, topic scoping, metric naming, and governance patterns aligned with the Open Semantic Interchange (OSI) standard. Use this skill when writing or reviewing semantic model definitions, authoring ai_context or description fields, designing topics or views in Omni or dbt/MetricFlow, or discussing semantic layer architecture and governance. Also use when working with OSI-aligned models or preparing a semantic layer for AI agent consumption."
---

# Semantic Layer

*This skill is planned but not yet implemented.*

This skill encodes judgment about semantic layer design: what makes a good metric definition, how to scope topics for AI consumption, and how to govern the layer so it serves both human analysts and AI agents reliably.

It sits on top of tool-specific skills like [omni-claude-skills](https://github.com/exploreomni/omni-claude-skills). Those cover the API mechanics; this skill covers when and why to make specific modeling decisions.

When complete, this skill will cover:

- AI context field authoring patterns (`ai_context`, `description`, `synonyms`): what to include, what to omit, how to test
- Topic and view scoping heuristics: one topic per business question domain
- Metric naming conventions aligned with OSI
- The ownership model: analyst-defined meaning, engineer-constrained correctness
- Reliability tier framework (GOLD / SILVER / BRONZE / DEPRECATED)
- Common failure modes in AI-facing semantic layers

## References

- [Anthropic Agent Skills](https://github.com/anthropics/skills): check for updated patterns and templates
- [Open Semantic Interchange (OSI)](https://github.com/open-semantic-interchange/OSI): vendor-agnostic semantic model standard
- [omni-claude-skills](https://github.com/exploreomni/omni-claude-skills): Omni-specific API skills (complementary, not competing)
- [dbt Semantic Layer](https://docs.getdbt.com/docs/build/about-metricflow): MetricFlow documentation
