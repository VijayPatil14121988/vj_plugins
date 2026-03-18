# OMOP ETL Design Patterns

## FHIR R4 to OMOP CDM Mapping

| FHIR Resource | OMOP Table(s) | Notes |
|---|---|---|
| Patient | `person`, `location`, `death` | Demographics, address, deceased info |
| Encounter | `visit_occurrence`, `visit_detail` | visit_detail for sub-encounters (ED→ICU) |
| Condition | `condition_occurrence` | Use `clinicalStatus` and `verificationStatus` to filter |
| Observation | `measurement`, `observation`, `condition_occurrence`, or `procedure_occurrence` | Route by standard concept's `domain_id` |
| MedicationRequest | `drug_exposure` | Prescriptions |
| MedicationAdministration | `drug_exposure` | Administered medications |
| MedicationStatement | `drug_exposure` | Self-reported medications |
| Immunization | `drug_exposure` | Vaccines mapped to RxNorm/CVX |
| Procedure | `procedure_occurrence` | Surgical/diagnostic procedures |
| ServiceRequest | `procedure_occurrence` | Ordered procedures/services |
| DiagnosticReport | `note`, `note_nlp` | Report text + structured NLP extractions |
| AllergyIntolerance | `observation` | Allergies without quantitative values |
| Device | `device_exposure` | Medical devices |
| Specimen | `specimen` | Biospecimen data |
| Coverage | `payer_plan_period` | Insurance enrollment |
| Claim / ExplanationOfBenefit | `cost` | Cost/charge data |

**Critical rule**: The target OMOP table is determined by the standard concept's `domain_id`, NOT the FHIR resource type. A FHIR Observation with a LOINC code mapping to a Condition-domain concept must go into `condition_occurrence`.

## Concept Mapping Strategy

1. **Extract code + system** from FHIR `coding` element (e.g., `code: "427.31"`, `system: "http://hl7.org/fhir/sid/icd-9-cm"`)
2. **Map FHIR system to OMOP vocabulary_id**:
   | FHIR System URI | OMOP vocabulary_id |
   |---|---|
   | `http://snomed.info/sct` | SNOMED |
   | `http://loinc.org` | LOINC |
   | `http://www.nlm.nih.gov/research/umls/rxnorm` | RxNorm |
   | `http://hl7.org/fhir/sid/icd-10-cm` | ICD10CM |
   | `http://hl7.org/fhir/sid/icd-9-cm` | ICD9CM |
   | `http://www.ama-assn.org/go/cpt` | CPT4 |
   | `urn:oid:2.16.840.1.113883.6.285` | HCPCS |
   | `http://hl7.org/fhir/sid/ndc` | NDC |
3. **Look up concept**: `SELECT concept_id FROM concept WHERE concept_code = :code AND vocabulary_id = :vocab_id`
4. **If non-standard** (`standard_concept IS NULL`): Map via `concept_relationship` with `relationship_id = 'Maps to'`
5. **If no mapping exists**: Set `concept_id = 0`, store source code in `_source_value`, log for unmapped data report
6. **For custom local codes**: Use `source_to_concept_map` table

### source_to_concept_map

The standard OHDSI mechanism for persisting organization-specific mappings:

| Column | Purpose |
|---|---|
| `source_code` | Local code value |
| `source_vocabulary_id` | Identifier for the local code system |
| `source_code_description` | Human-readable description |
| `target_concept_id` | Standard OMOP concept it maps to |
| `target_vocabulary_id` | Target vocabulary (SNOMED, RxNorm, etc.) |
| `valid_start_date` / `valid_end_date` | Mapping validity window |
| `invalid_reason` | NULL if valid |

## Processing Order

Process FHIR resources in dependency order:

1. **Infrastructure**: Location → Organization (`care_site`) → Practitioner (`provider`)
2. **Identity**: Patient (`person`)
3. **Encounters**: Encounter (`visit_occurrence`, `visit_detail`)
4. **Clinical data**: Condition, Observation, Procedure, Medication*, Immunization, Device, etc.
5. **Documents**: DiagnosticReport (`note`), DocumentReference
6. **Derived tables**: `observation_period`, `condition_era`, `drug_era`, `dose_era`

*For medications: if using `medicationReference` (reference to a Medication resource), process Medication resources BEFORE MedicationRequest to populate a medication code cache.

## ID Generation Strategies

| Strategy | Pros | Cons |
|---|---|---|
| **Deterministic** (hash of source ID) | Idempotent reloads, no DB lookup needed | Hash collisions (extremely rare) |
| **Auto-increment** (DB sequence) | Simple, guaranteed unique | Not idempotent, need dedup logic |
| **Source ID as person_id** (e.g., `vibrentId`) | Direct traceability | Must ensure uniqueness across sources |

## Date Handling

| FHIR Format | OMOP Mapping |
|---|---|
| `dateTime` (full) | Split into `_date` (DATE) and `_datetime` (DATETIME) |
| `date` (YYYY-MM-DD) | Use directly in `_date`, set `_datetime` to midnight |
| Partial date (YYYY or YYYY-MM) | Use `year_of_birth`/`month_of_birth` for person; for events, use first day of period |
| `Period.start` / `Period.end` | Map to `_start_date` / `_end_date` |
| Missing end date | Leave NULL or set equal to start_date (table-dependent) |

## Unmapped Code Handling

- **Never silently drop** records with unmapped codes
- Set `_concept_id = 0` (no matching concept)
- Store original code in `_source_value`
- Store source vocabulary concept_id in `_source_concept_id` (if it exists in `concept`)
- Track unmapped codes in a reporting table with counts per vocabulary

## Deduplication

- Deduplicate by source identifier (e.g., FHIR resource `id`)
- Use provenance tables to track which source records have been processed
- Design for idempotent loads: reprocessing the same source should not create duplicates

## Visit Linkage

- Resolve `visit_occurrence_id` via FHIR `encounter` reference on clinical resources
- If encounter reference is missing, attempt temporal matching (event date within visit window)
- If no visit match, set `visit_occurrence_id = NULL` (not 0, not -1)

## CDM Source Metadata

Populate `cdm_source` after ETL completes:
- `cdm_source_name`: descriptive name of the CDM instance
- `cdm_source_abbreviation`: short identifier
- `source_release_date`: date of source data extract
- `vocabulary_version`: version of loaded OMOP vocabulary
- `cdm_release_date`: date this CDM was built
