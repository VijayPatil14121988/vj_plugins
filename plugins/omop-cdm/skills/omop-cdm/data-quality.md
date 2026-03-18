# OMOP Data Quality & THEMIS Conventions

## DQD Check Categories

The OHDSI Data Quality Dashboard (DQD) organizes checks into three categories:

| Category | What It Checks | Examples |
|---|---|---|
| **Completeness** | Required fields are populated | `person.year_of_birth` not NULL, `_concept_id` != 0 rates |
| **Conformance** | Values match expected format/domain | `standard_concept = 'S'` in `_concept_id` fields, valid `domain_id` |
| **Plausibility** | Values are clinically reasonable | Dates after birth, measurement values in range, temporal ordering |

## THEMIS Conventions

THEMIS provides authoritative guidance for ambiguous CDM questions:

### Observation Period
- **Derived**, not loaded from source — computed from clinical event dates
- Gap threshold is **configurable and data-source-dependent**:
  - EHR data: commonly 548 days (18 months) between events
  - Claims data: typically based on enrollment gaps (32 days is common)
- Events outside `observation_period` are still valid but may indicate data quality issues
- End date: may extend beyond last event (e.g., +60 days for death)

### Era Derivation
- **Condition era**: 30-day persistence window (default, configurable). Merges condition records of the same `condition_concept_id` when the gap between the *end* of one record and the *start* of the next is <= 30 days.
- **Drug era**: 30-day persistence window at ingredient level. Drug exposures are rolled up to ingredients via `concept_ancestor`, then merged when end-to-start gaps are <= 30 days.
- **Dose era**: Groups consecutive drug exposures with the same ingredient AND same daily dose into continuous periods.

### Temporal Rules
- All clinical event dates must be **after** `person.birth_datetime`
- All clinical event dates must be **before** `death.death_date + 60 days`
- `start_date <= end_date` for all events
- Events should fall within an `observation_period` window

## Plausibility Rules

| Rule | Threshold | Action |
|---|---|---|
| `days_supply` | <= 365 | Cap at 365 if exceeded |
| `start_date` vs `end_date` | start <= end | Fix end_date to match start_date |
| `year_of_birth` | 1900 – current year | Flag records outside range |
| `visit_end_date` vs `visit_start_date` | end >= start | Flag or fix |
| Measurement `value_as_number` | Domain-specific ranges | Flag implausible values (e.g., negative lab results) |
| Pre-birth events | Event date >= birth date | Log but typically don't alter |

## Concept Validity Checks

| Check | Rule | Fix |
|---|---|---|
| Standard concept in `_concept_id` | `standard_concept = 'S'` | Remap via `concept_relationship` with `'Maps to'` |
| Domain matches table | `concept.domain_id` matches target table domain | Move record to correct table |
| Concept is valid | `invalid_reason IS NULL` | Map to replacement concept or use 0 |
| Source concept in `_source_concept_id` | Can be any `standard_concept` value | No fix needed — source concepts are informational |

## Mapping Rate Targets

These are **recommended thresholds**, not hard OHDSI standards:

| Domain | Target Mapping Rate | Meaning |
|---|---|---|
| Conditions | >= 70% | At least 70% of condition records have `condition_concept_id != 0` |
| Measurements | >= 80% | At least 80% of measurement records have `measurement_concept_id != 0` |
| Drugs | >= 70% | At least 70% of drug records have `drug_concept_id != 0` |
| Procedures | >= 70% | At least 70% of procedure records have `procedure_concept_id != 0` |

Low mapping rates indicate vocabulary gaps or ETL issues. Track unmapped codes by vocabulary for prioritization.

## Common DQD Fixes

| Issue | Fix |
|---|---|
| Event outside visit date range | Set `visit_occurrence_id = NULL` |
| Event before birth | Log for review, do not alter |
| Non-standard concept in `_concept_id` | Remap: lookup `concept_relationship` where `relationship_id = 'Maps to'` |
| `days_supply > 365` | Cap at 365 |
| `end_date < start_date` | Set `end_date = start_date` |
| Unit mismatches (CBC %, HbA1c) | Remap unit_concept_id and adjust values if needed |
| `visit_occurrence_id = -1` or sentinel values | Replace with NULL |

## Vocabulary Version Awareness

- OMOP vocabularies are updated **quarterly**
- Concept mappings can change between releases — a code mapped to concept X today may map to concept Y next quarter
- Always check `valid_end_date` and `invalid_reason` on concepts
- When updating vocabularies, re-evaluate unmapped codes — new mappings may become available
- Document the vocabulary version used in `cdm_source.vocabulary_version`
