---
name: healthcare
description: Use when working with OMOP CDM, clinical data, PHI, HIPAA compliance, or healthcare data pipelines - provides domain-specific questions for requirements and review checklists
---

# Healthcare Domain — Pipeline Guidance

Lightweight domain guidance for pipeline stages. Auto-activated when OMOP/clinical code is detected. Deep implementation knowledge lives in the `healthcare-engineer` agent.

## When This Activates

Triggered when the project contains:
- OMOP CDM table references (person, condition_occurrence, drug_exposure, etc.)
- Clinical vocabulary references (SNOMED, ICD10, RxNorm, LOINC, etc.)
- PHI-related code patterns
- Healthcare-specific dependencies

## Product Owner Questions

When gathering requirements for healthcare-related work:
- Does this feature handle PHI (Protected Health Information)?
- Is OMOP CDM concept mapping required? Which vocabularies?
- Are there compliance implications (HIPAA, IRB)?
- What clinical data quality checks are needed?
- Will this involve source-to-OMOP ETL transformations?
- Is audit trail required for data access or modifications?

## Code Review Checklist

When healthcare domain is active, code review adds:
- [ ] PHI never logged, never in error messages, never in URLs
- [ ] OMOP concept_ids validated against concept table
- [ ] Clinical data quality assertions present in tests
- [ ] HIPAA-safe error handling (no patient data in stack traces)
- [ ] Synthetic test data only — never real patient data
- [ ] ETL transformations are idempotent
- [ ] Audit trail for data access/modification (if applicable)
- [ ] Source values preserved alongside mapped concept IDs
