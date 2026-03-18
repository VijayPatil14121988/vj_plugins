---
name: omop-cdm
description: Use when working with OMOP CDM tables, vocabularies, concept mapping, OHDSI conventions, ETL transformations, or clinical data quality — any task involving observational health data in the OMOP Common Data Model
---

# OMOP CDM v5.4 Knowledge

Provides OMOP Common Data Model conventions, vocabulary navigation, SQL patterns, ETL design guidance, and data quality rules.

## Routing

Determine which domain the user's task falls into and read the relevant reference file(s) from this directory:

| Task Domain | Reference File |
|---|---|
| Concept lookup, code mapping, vocabulary hierarchy | Read `vocabulary-navigation.md` |
| Which CDM table, field conventions, type concepts | Read `cdm-table-guide.md` |
| SQL queries, cohorts, eras, joins | Read `sql-patterns.md` |
| ETL mapping, FHIR-to-OMOP transformation | Read `etl-design-patterns.md` |
| Data quality, DQD checks, THEMIS rules | Read `data-quality.md` |
| Composite task spanning multiple domains | Read the relevant files for each domain |

## Universal Principles

- Always use standard concepts (`standard_concept = 'S'`) in `_concept_id` fields; source concepts go in `_source_concept_id` fields
- `concept_id = 0` means unmapped — never silently drop, always track
- All dates use `DATE` type, datetimes use `DATETIME`
- Person-centric model: every clinical event links to `person_id`
- Visit-centric linkage: clinical events should link to `visit_occurrence_id` when available
- Vocabulary is the backbone — always map through `concept_relationship` with `relationship_id = 'Maps to'`
- Eras are derived (observation_period, condition_era, drug_era, dose_era), not stored directly from source data
- THEMIS conventions are authoritative for ambiguous CDM questions
- Store original source strings in `_source_value` fields
- The CDM version is v5.4
- Vocabulary versions update quarterly — respect `valid_start_date`/`valid_end_date` on the `concept` table

## Common Mistakes

- Using `concept_code` instead of `concept_id` in clinical tables
- Querying source concepts when you need standard concepts (or vice versa)
- Forgetting `concept_relationship` has directionality (`concept_id_1` → `concept_id_2`)
- Confusing `observation` (catch-all domain) with `measurement` (quantitative results with values)
- Building eras from raw data instead of using the standard persistence window algorithms
- Using `concept_id = 0` and `concept_id = 8551` (Unknown) interchangeably — `8551` means source explicitly recorded "Unknown"; `0` means no mappable information was available
