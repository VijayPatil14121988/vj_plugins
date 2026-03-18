# OMOP Vocabulary Navigation

## Concept Table Structure

The `concept` table is the backbone of the OMOP vocabulary. Every coded value in the CDM references this table.

| Column | Purpose |
|---|---|
| `concept_id` | Unique integer identifier — use this in all CDM `_concept_id` fields |
| `concept_name` | Human-readable English name |
| `domain_id` | Clinical domain: Condition, Drug, Measurement, Procedure, Observation, Device, Specimen, etc. |
| `vocabulary_id` | Source vocabulary: SNOMED, LOINC, RxNorm, ICD10CM, CPT4, HCPCS, NDC, etc. |
| `concept_class_id` | Granularity within vocabulary: "Clinical Finding", "Ingredient", "Lab Test", "Procedure", etc. |
| `standard_concept` | `'S'` = Standard, `'C'` = Classification, `NULL` = Source/non-standard |
| `concept_code` | Code in the source vocabulary (e.g., '49436004' for SNOMED atrial fibrillation) |
| `valid_start_date` / `valid_end_date` | Validity period — check these, as concepts can be deprecated |
| `invalid_reason` | `NULL` = valid, `'D'` = deleted, `'U'` = updated/replaced |

## Standard vs Source Concepts

- **Standard (`S`)**: Use in `_concept_id` fields (e.g., `condition_concept_id`). These are the canonical OMOP representations.
- **Classification (`C`)**: Used for grouping/hierarchy navigation (e.g., MedDRA high-level terms). Not for clinical table `_concept_id` fields.
- **Source (`NULL`)**: Use in `_source_concept_id` fields. These represent the original coding system (ICD10CM, NDC, etc.).

## Key Vocabularies by Domain

| Domain | Standard Vocabulary | Common Source Vocabularies |
|---|---|---|
| Conditions | SNOMED | ICD10CM, ICD9CM, Read, MedDRA |
| Drugs | RxNorm | NDC, GPI, ATC |
| Measurements | LOINC | Local lab codes, CPT4 (for lab procedures) |
| Procedures | SNOMED | CPT4, HCPCS, ICD10PCS, OPCS4 |
| Devices | SNOMED | HCPCS, UDI |
| Observations | SNOMED | Various (catch-all domain) |

## 3-Step Source-to-Standard Mapping

This is the core workflow for mapping any clinical code to an OMOP standard concept:

**Step 1 — Find the source concept:**
```sql
SELECT concept_id, concept_name, vocabulary_id, standard_concept
FROM concept
WHERE concept_code = '427.31'
  AND vocabulary_id = 'ICD9CM';
-- Returns: concept_id=44821957, standard_concept=NULL (source)
```

**Step 2 — Map to standard via concept_relationship:**
```sql
SELECT c2.concept_id, c2.concept_name, c2.vocabulary_id, c2.standard_concept
FROM concept_relationship cr
JOIN concept c2 ON cr.concept_id_2 = c2.concept_id
WHERE cr.concept_id_1 = 44821957
  AND cr.relationship_id = 'Maps to'
  AND cr.invalid_reason IS NULL;
-- Returns: concept_id=313217, concept_name='Atrial fibrillation', vocabulary_id='SNOMED', standard_concept='S'
```

**Step 3 — Use in CDM:**
- `condition_concept_id = 313217` (standard SNOMED concept)
- `condition_source_concept_id = 44821957` (original ICD9CM concept)
- `condition_source_value = '427.31'` (original code string)

## Hierarchy Navigation with concept_ancestor

The `concept_ancestor` table stores pre-computed hierarchical relationships:

```sql
-- Find all descendant conditions of "Diabetes mellitus" (SNOMED 201820)
SELECT c.concept_id, c.concept_name, ca.min_levels_of_separation
FROM concept_ancestor ca
JOIN concept c ON ca.descendant_concept_id = c.concept_id
WHERE ca.ancestor_concept_id = 201820
  AND c.standard_concept = 'S';
```

```sql
-- Find ancestors of a specific concept (walk UP the hierarchy)
SELECT c.concept_id, c.concept_name, ca.min_levels_of_separation
FROM concept_ancestor ca
JOIN concept c ON ca.ancestor_concept_id = c.concept_id
WHERE ca.descendant_concept_id = 313217;
```

```sql
-- Find siblings (concepts sharing the same parent)
SELECT DISTINCT c.concept_id, c.concept_name
FROM concept_ancestor ca1
JOIN concept_ancestor ca2 ON ca1.ancestor_concept_id = ca2.ancestor_concept_id
JOIN concept c ON ca2.descendant_concept_id = c.concept_id
WHERE ca1.descendant_concept_id = 313217  -- target concept
  AND ca1.min_levels_of_separation = 1
  AND ca2.min_levels_of_separation = 1
  AND c.standard_concept = 'S';
```

## Drug Hierarchy

RxNorm drug concepts follow this hierarchy (use `concept_class_id` to filter):

| Level | concept_class_id | Example |
|---|---|---|
| Ingredient | `'Ingredient'` | Metformin |
| Clinical Drug Form | `'Clinical Drug Form'` | Metformin Oral Tablet |
| Clinical Drug Comp | `'Clinical Drug Comp'` | Metformin 500 MG |
| Clinical Drug | `'Clinical Drug'` | Metformin 500 MG Oral Tablet |
| Branded Drug | `'Branded Drug'` | Glucophage 500 MG Oral Tablet |

**drug_strength table** — links drug concepts to ingredient amounts:

| Column | Purpose |
|---|---|
| `drug_concept_id` | The drug concept |
| `ingredient_concept_id` | The ingredient |
| `amount_value` / `amount_unit_concept_id` | Solid dose (e.g., 500 MG) |
| `numerator_value` / `denominator_value` | Liquid concentration (e.g., 100 MG per 5 ML) |

## Concept Search Recipes

```sql
-- Search by name (partial match)
SELECT concept_id, concept_name, vocabulary_id, standard_concept
FROM concept
WHERE concept_name ILIKE '%atrial fibrillation%'
  AND standard_concept = 'S'
  AND invalid_reason IS NULL;
```

```sql
-- Check synonyms
SELECT cs.concept_synonym_name, c.concept_id, c.concept_name
FROM concept_synonym cs
JOIN concept c ON cs.concept_id = c.concept_id
WHERE cs.concept_synonym_name ILIKE '%afib%';
```

## source_to_concept_map

For custom/local codes not in the standard vocabulary, organizations use `source_to_concept_map`:

| Column | Purpose |
|---|---|
| `source_code` | Local code value |
| `source_vocabulary_id` | Identifier for the local vocabulary |
| `target_concept_id` | Standard concept it maps to |
| `target_vocabulary_id` | Vocabulary of the target (usually SNOMED, RxNorm, etc.) |
| `valid_start_date` / `valid_end_date` | Mapping validity period |

```sql
-- Look up a custom mapping
SELECT target_concept_id, c.concept_name
FROM source_to_concept_map stcm
JOIN concept c ON stcm.target_concept_id = c.concept_id
WHERE stcm.source_code = 'LOCAL_LAB_001'
  AND stcm.source_vocabulary_id = 'MyHospital';
```
