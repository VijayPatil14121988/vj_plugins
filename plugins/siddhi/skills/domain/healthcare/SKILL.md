---
name: healthcare
description: Use when working with OMOP CDM, clinical data, PHI, HIPAA compliance, or healthcare data pipelines - provides domain-specific conventions, checks, and patterns for clinical data engineering
---

# Healthcare Domain — Clinical Data Engineering

Domain-specific guidance for healthcare data engineering. Auto-activated when OMOP/clinical code is detected in the project.

## When This Activates

Triggered when the project contains:
- OMOP CDM table references (person, condition_occurrence, drug_exposure, etc.)
- Clinical vocabulary references (SNOMED, ICD10, RxNorm, LOINC, etc.)
- PHI-related code patterns
- Healthcare-specific dependencies

## OMOP CDM Conventions

### Table Naming
- Use standard OMOP CDM table names: `person`, `visit_occurrence`, `condition_occurrence`, `drug_exposure`, `measurement`, `observation`, `procedure_occurrence`
- Domain tables link to `concept` table via `*_concept_id` foreign keys
- Always use `concept_id` (integer), never raw codes, for standardized data

### Concept Mapping
- Source values → `source_concept_id` (the original code)
- Standardized values → `*_concept_id` (the OMOP standard concept)
- Always map through `concept_relationship` table for standard mappings
- Vocabulary hierarchy: Standard concepts → Classification concepts → Non-standard concepts

### ETL Patterns
- Source-to-OMOP transformations should be idempotent
- Track ETL provenance: which source record produced which OMOP record
- Use `source_value` fields to preserve original data alongside mapped concepts
- Validate concept_ids exist in the concept table before inserting

## PHI / HIPAA Guidelines

### What Is PHI
Protected Health Information includes: names, dates (except year), phone numbers, email addresses, SSN, MRN, device identifiers, URLs, IPs, biometric identifiers, photos, account numbers, certificate/license numbers, any unique identifying number.

### Hard Rules
- **Never log PHI** — not in application logs, not in error messages, not in debug output
- **Never include PHI in URLs** — no patient IDs in query parameters or path segments
- **Never include PHI in error responses** — stack traces must not contain patient data
- **Never store PHI in plain text** — encrypt at rest, encrypt in transit
- **Audit trail** — all PHI access must be logged (who accessed what, when)

### Testing with PHI
- **Never use real patient data in tests** — always synthetic/anonymized
- **Test fixtures** should use obviously fake data (e.g., "Test Patient", DOB: 1900-01-01)
- **Testcontainers databases** should be loaded with synthetic OMOP data

## Data Quality Checks

When working with clinical data, verify:
- **Completeness**: Required fields populated (person_id, concept_id, dates)
- **Validity**: concept_ids exist in concept table, dates are reasonable
- **Consistency**: visit dates encompass condition/drug dates, person birth_date before all events
- **Uniqueness**: No duplicate records for the same clinical event

## Code Review Additions (Tier 3)

When healthcare domain is active, code review adds:
- [ ] PHI never logged or in error messages
- [ ] OMOP concept_ids validated against concept table
- [ ] Clinical data quality assertions in tests
- [ ] HIPAA-safe error handling
- [ ] Synthetic test data, never real patient data
- [ ] ETL transformations are idempotent
- [ ] Audit trail for data access/modification
