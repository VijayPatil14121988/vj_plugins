# OMOP CDM Plugin for Claude Code

A Claude Code plugin that provides OMOP Common Data Model v5.4 knowledge for:

- **Vocabulary navigation** — concept lookup, source-to-standard mapping, hierarchy traversal
- **CDM table conventions** — domain routing, field requirements, type concepts
- **SQL query patterns** — cohort building, era derivation, concept-aware joins
- **ETL design** — FHIR R4 to OMOP mapping, concept mapping strategy, processing order
- **Data quality** — DQD checks, THEMIS conventions, plausibility rules

## Installation

```bash
# From local path
claude plugins add /path/to/omop-cdm-plugin

# From git
claude plugins add github:vijaypatil/omop-cdm-plugin
```

## Usage

The skill auto-triggers when Claude detects OMOP-related context (table names, vocabulary terms, OHDSI conventions). You can also invoke it explicitly with `/omop`.

## What It Does

This is a **knowledge-only** plugin. It does not connect to any database. It teaches Claude the correct OMOP CDM conventions so it generates accurate SQL, mappings, and guidance.

## CDM Version

OMOP CDM v5.4 (OHDSI)
