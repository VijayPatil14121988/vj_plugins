# OMOP CDM Table Guide (v5.4)

## Domain-to-Table Routing

When deciding which CDM table to store data in, route by clinical domain:

| Clinical Domain | Target Table | When to Use |
|---|---|---|
| Lab results with numeric values | `measurement` | Quantitative results: labs, vitals, anthropometrics |
| Diagnoses, problems | `condition_occurrence` | ICD/SNOMED conditions, problem list entries |
| Medications | `drug_exposure` | Prescriptions, dispensings, administrations, immunizations |
| Surgeries, diagnostic procedures | `procedure_occurrence` | CPT/SNOMED procedures, services |
| Medical devices | `device_exposure` | Implants, devices, equipment |
| Clinical notes | `note` / `note_nlp` | Free text, structured NLP extractions |
| Death | `death` | Date, cause, type of death record |
| Biospecimens | `specimen` | Blood, tissue, urine samples |
| Cost/charge data | `cost` | Charges, costs, payments (consolidated in v5.4) |
| Insurance enrollment | `payer_plan_period` | Plan enrollment start/end, payer |
| Oncology episodes | `episode` / `episode_event` | Treatment episodes (v5.4) |
| Cohort membership | `cohort` / `cohort_definition` | Research cohorts |
| Cross-event links | `fact_relationship` | Links between any two clinical events |
| **Everything else** | `observation` | Social history, family history, surveys, allergies, lifestyle factors |

**Key rule**: The target table is determined by the standard concept's `domain_id`, not the source data format. A FHIR Observation with a LOINC code that maps to a Measurement-domain concept goes into `measurement`, not `observation`.

## Infrastructure Tables

| Table | Purpose | Key Fields |
|---|---|---|
| `person` | One row per patient | `person_id`, `year_of_birth`, `gender_concept_id`, `race_concept_id`, `ethnicity_concept_id`, `location_id` |
| `observation_period` | Continuous observation windows (derived) | `person_id`, `observation_period_start_date`, `observation_period_end_date` |
| `visit_occurrence` | Full encounters/visits | `visit_occurrence_id`, `person_id`, `visit_concept_id`, `visit_start_date`, `visit_end_date` |
| `visit_detail` | Sub-encounters within a visit (e.g., ED→ICU) | `visit_detail_id`, `visit_occurrence_id`, `visit_detail_concept_id` |
| `provider` | Clinicians/practitioners | `provider_id`, `provider_name`, `specialty_concept_id`, `care_site_id` |
| `care_site` | Facilities/organizations | `care_site_id`, `care_site_name`, `place_of_service_concept_id`, `location_id` |
| `location` | Addresses/geography | `location_id`, `city`, `state`, `zip`, `county`, `country` |
| `cdm_source` | CDM instance metadata | `cdm_source_name`, `cdm_source_abbreviation`, `source_release_date`, `vocabulary_version` |
| `metadata` | Key-value metadata | `metadata_concept_id`, `name`, `value_as_string` |

## Clinical Table Field Conventions

Every clinical event table follows these patterns:

| Field Pattern | Purpose | Example |
|---|---|---|
| `{table}_id` | Primary key | `condition_occurrence_id` |
| `person_id` | Links to `person` (required) | Always NOT NULL |
| `{domain}_concept_id` | Standard concept (required) | `condition_concept_id` — must be standard (`S`) |
| `{domain}_source_value` | Original source string | `condition_source_value = 'I48.91'` |
| `{domain}_source_concept_id` | Source vocabulary concept | `condition_source_concept_id` (ICD10CM concept_id) |
| `{domain}_type_concept_id` | Data provenance type | How the record was captured |
| `{domain}_start_date` | Event start (required) | `condition_start_date` |
| `{domain}_end_date` | Event end (nullable) | `condition_end_date` |
| `visit_occurrence_id` | Links to visit (nullable) | Foreign key to `visit_occurrence` |
| `provider_id` | Links to provider (nullable) | Foreign key to `provider` |

## Type Concepts (`_type_concept_id`)

Type concepts describe **how** the data was recorded:

| concept_id | Name | Use When |
|---|---|---|
| 32817 | EHR | Record from electronic health record |
| 32810 | Claim | Record from insurance claim |
| 32882 | Survey | Patient-reported/survey data |
| 32862 | EHR dispensing record | Pharmacy dispensing in EHR |
| 32838 | EHR administration | Medication administration in EHR |
| 32879 | Registry | Data from a clinical registry |

## Fixed Concept ID Reference

### Gender
| concept_id | Meaning |
|---|---|
| 8507 | Male |
| 8532 | Female |
| 8551 | Unknown (source explicitly recorded "Unknown") |
| 0 | No matching concept (no mappable gender info) |

### Race (US OMB categories — other race vocabularies exist for international use)
| concept_id | Meaning |
|---|---|
| 8527 | White |
| 8516 | Black or African American |
| 8515 | Asian |
| 8657 | American Indian or Alaska Native |
| 8557 | Native Hawaiian or Other Pacific Islander |

### Ethnicity (US OMB categories)
| concept_id | Meaning |
|---|---|
| 38003563 | Hispanic or Latino |
| 38003564 | Not Hispanic or Latino |

### Visit Types
| concept_id | Meaning |
|---|---|
| 9201 | Inpatient Visit |
| 9202 | Outpatient Visit |
| 9203 | Emergency Room Visit |
| 42898160 | Long Term Care Visit |
| 262 | Emergency Room and Inpatient Visit |

## Key concept_class_id Values

Use these to filter concepts by granularity:

| concept_class_id | Domain | Purpose |
|---|---|---|
| `'Clinical Finding'` | Condition | Standard clinical diagnoses |
| `'Procedure'` | Procedure | Standard procedures |
| `'Lab Test'` | Measurement | Laboratory tests |
| `'Observable Entity'` | Observation | Observable clinical facts |
| `'Ingredient'` | Drug | Drug active ingredients |
| `'Clinical Drug'` | Drug | Specific drug formulations |
| `'Branded Drug'` | Drug | Brand name formulations |

## person Table Details

```sql
-- Required fields
person_id              -- NOT NULL, unique patient identifier
gender_concept_id      -- NOT NULL (8507, 8532, 8551, or 0)
year_of_birth          -- NOT NULL
race_concept_id        -- NOT NULL (use 0 if unknown)
ethnicity_concept_id   -- NOT NULL (use 0 if unknown)

-- Optional but recommended
month_of_birth, day_of_birth
birth_datetime
location_id            -- FK to location
provider_id            -- FK to provider (primary care)
care_site_id           -- FK to care_site

-- Source value fields
gender_source_value, race_source_value, ethnicity_source_value
gender_source_concept_id, race_source_concept_id, ethnicity_source_concept_id
```
