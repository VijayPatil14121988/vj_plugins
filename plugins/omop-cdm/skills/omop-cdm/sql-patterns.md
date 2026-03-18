# OMOP CDM SQL Patterns

All queries assume tables are in the same schema. Prefix with schema name (e.g., `omop_cdm.person`) if needed.

## Concept-Aware Joins

Always join to `concept` to get readable names:

```sql
SELECT co.condition_occurrence_id, co.person_id,
       c.concept_name AS condition_name,
       co.condition_start_date
FROM condition_occurrence co
JOIN concept c ON co.condition_concept_id = c.concept_id
WHERE co.person_id = 12345;
```

## Cohort Building with Hierarchy

Use `concept_ancestor` to capture all descendants of a clinical concept:

```sql
-- All patients with any type of diabetes (SNOMED 201820 = Diabetes mellitus)
SELECT DISTINCT co.person_id
FROM condition_occurrence co
JOIN concept_ancestor ca ON co.condition_concept_id = ca.descendant_concept_id
WHERE ca.ancestor_concept_id = 201820;
```

**With time window and exclusion:**

```sql
-- Patients with T2DM diagnosed 2020+, excluding those with T1DM
SELECT DISTINCT co.person_id
FROM condition_occurrence co
JOIN concept_ancestor ca ON co.condition_concept_id = ca.descendant_concept_id
WHERE ca.ancestor_concept_id = 201826  -- Type 2 diabetes
  AND co.condition_start_date >= '2020-01-01'
  AND co.person_id NOT IN (
    SELECT co2.person_id
    FROM condition_occurrence co2
    JOIN concept_ancestor ca2 ON co2.condition_concept_id = ca2.descendant_concept_id
    WHERE ca2.ancestor_concept_id = 201254  -- Type 1 diabetes
  );
```

## Person Demographics

```sql
-- Patients born between 1990-2000 with gender
SELECT p.person_id, p.year_of_birth, p.month_of_birth,
       g.concept_name AS gender,
       r.concept_name AS race,
       e.concept_name AS ethnicity
FROM person p
JOIN concept g ON p.gender_concept_id = g.concept_id
LEFT JOIN concept r ON p.race_concept_id = r.concept_id
LEFT JOIN concept e ON p.ethnicity_concept_id = e.concept_id
WHERE p.year_of_birth BETWEEN 1990 AND 2000;
```

## Measurement Queries

```sql
-- HbA1c results > 9% (LOINC concept for HbA1c)
SELECT m.person_id, m.measurement_date, m.value_as_number,
       u.concept_name AS unit
FROM measurement m
JOIN concept_ancestor ca ON m.measurement_concept_id = ca.descendant_concept_id
LEFT JOIN concept u ON m.unit_concept_id = u.concept_id
WHERE ca.ancestor_concept_id = 3004410  -- Hemoglobin A1c
  AND m.value_as_number > 9.0;
```

```sql
-- Latest measurement per patient (window function pattern)
SELECT person_id, measurement_date, value_as_number, concept_name
FROM (
  SELECT m.person_id, m.measurement_date, m.value_as_number,
         c.concept_name,
         ROW_NUMBER() OVER (PARTITION BY m.person_id ORDER BY m.measurement_date DESC) AS rn
  FROM measurement m
  JOIN concept c ON m.measurement_concept_id = c.concept_id
  WHERE m.measurement_concept_id = 3004410
) sub
WHERE rn = 1;
```

## Drug Exposure Queries

```sql
-- Top 10 most prescribed drug ingredients
SELECT c.concept_name AS ingredient, COUNT(*) AS prescription_count
FROM drug_exposure de
JOIN concept_ancestor ca ON de.drug_concept_id = ca.descendant_concept_id
JOIN concept c ON ca.ancestor_concept_id = c.concept_id
WHERE c.concept_class_id = 'Ingredient'
  AND c.standard_concept = 'S'
GROUP BY c.concept_name
ORDER BY prescription_count DESC
LIMIT 10;
```

```sql
-- Patients on metformin with days_supply
SELECT de.person_id, de.drug_exposure_start_date, de.days_supply,
       de.quantity, c.concept_name
FROM drug_exposure de
JOIN concept_ancestor ca ON de.drug_concept_id = ca.descendant_concept_id
JOIN concept c ON de.drug_concept_id = c.concept_id
WHERE ca.ancestor_concept_id = 1503297  -- Metformin (ingredient)
ORDER BY de.drug_exposure_start_date;
```

## Visit Analysis

```sql
-- Average length of stay for inpatient visits
SELECT AVG(visit_end_date - visit_start_date) AS avg_los_days,
       COUNT(*) AS visit_count
FROM visit_occurrence
WHERE visit_concept_id = 9201;  -- Inpatient
```

```sql
-- 30-day readmission
SELECT v1.person_id, v1.visit_start_date AS index_visit,
       v2.visit_start_date AS readmission_date,
       v2.visit_start_date - v1.visit_end_date AS days_to_readmission
FROM visit_occurrence v1
JOIN visit_occurrence v2
  ON v1.person_id = v2.person_id
  AND v2.visit_start_date > v1.visit_end_date
  AND v2.visit_start_date <= v1.visit_end_date + 30
WHERE v1.visit_concept_id = 9201
  AND v2.visit_concept_id = 9201;
```

## Era Derivation Patterns

**Observation Period** — gap threshold is data-source-dependent (548 days common for EHR, shorter for claims):

```sql
-- Derive observation periods from clinical events
WITH all_events AS (
  SELECT person_id, condition_start_date AS event_date FROM condition_occurrence
  UNION ALL
  SELECT person_id, drug_exposure_start_date FROM drug_exposure
  UNION ALL
  SELECT person_id, measurement_date FROM measurement
  UNION ALL
  SELECT person_id, observation_date FROM observation
  UNION ALL
  SELECT person_id, procedure_date FROM procedure_occurrence
),
ordered AS (
  SELECT person_id, event_date,
         CASE WHEN event_date - LAG(event_date) OVER (PARTITION BY person_id ORDER BY event_date) > 548
              THEN 1 ELSE 0 END AS new_period
  FROM all_events
),
groups AS (
  SELECT person_id, event_date,
         SUM(new_period) OVER (PARTITION BY person_id ORDER BY event_date) AS period_group
  FROM ordered
)
SELECT person_id,
       MIN(event_date) AS observation_period_start_date,
       MAX(event_date) AS observation_period_end_date
FROM groups
GROUP BY person_id, period_group;
```

**Condition Era** — 30-day persistence window (gap between *end* of one record and *start* of next):

The standard algorithm merges condition records of the same `condition_concept_id` when the gap between end and start is <= 30 days.

**Drug Era** — 30-day persistence window + ingredient rollup:

Drug eras roll up drug exposures to the ingredient level using `concept_ancestor`, then merge exposures where the gap between end of one and start of the next is <= 30 days.

## Measurement: Categorical Results

Some measurements have categorical results (Positive/Negative, Detected/Not Detected) stored in `value_as_concept_id` instead of `value_as_number`:

```sql
-- Categorical lab results (e.g., COVID test Positive/Negative)
SELECT m.person_id, m.measurement_date,
       c_test.concept_name AS test_name,
       c_val.concept_name AS result
FROM measurement m
JOIN concept c_test ON m.measurement_concept_id = c_test.concept_id
JOIN concept c_val ON m.value_as_concept_id = c_val.concept_id
WHERE m.value_as_concept_id != 0;
```

## Prevalence Pattern

```sql
-- Period prevalence of a condition in a given year
SELECT COUNT(DISTINCT co.person_id) * 1.0 / COUNT(DISTINCT op.person_id) AS prevalence
FROM observation_period op
LEFT JOIN condition_occurrence co
  ON op.person_id = co.person_id
  AND co.condition_start_date BETWEEN '2023-01-01' AND '2023-12-31'
  AND co.condition_concept_id IN (
    SELECT descendant_concept_id FROM concept_ancestor WHERE ancestor_concept_id = 201826
  )
WHERE op.observation_period_start_date <= '2023-12-31'
  AND op.observation_period_end_date >= '2023-01-01';
```

## Cost Analysis

```sql
-- Total cost per visit
SELECT vo.visit_occurrence_id, vo.visit_start_date,
       SUM(c.total_charge) AS total_charges,
       SUM(c.total_paid) AS total_paid
FROM visit_occurrence vo
JOIN cost c ON vo.visit_occurrence_id = c.cost_event_id
GROUP BY vo.visit_occurrence_id, vo.visit_start_date;
```

## Multi-Table Patient Timeline

```sql
-- Full clinical timeline for a patient
SELECT 'Condition' AS domain, co.condition_start_date AS event_date,
       c.concept_name AS event_name
FROM condition_occurrence co
JOIN concept c ON co.condition_concept_id = c.concept_id
WHERE co.person_id = 12345
UNION ALL
SELECT 'Drug', de.drug_exposure_start_date, c.concept_name
FROM drug_exposure de
JOIN concept c ON de.drug_concept_id = c.concept_id
WHERE de.person_id = 12345
UNION ALL
SELECT 'Measurement', m.measurement_date, c.concept_name
FROM measurement m
JOIN concept c ON m.measurement_concept_id = c.concept_id
WHERE m.person_id = 12345
ORDER BY event_date;
```
