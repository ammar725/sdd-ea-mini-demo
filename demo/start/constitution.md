# constitution.md — Enterprise AI rules (excerpt)

**Version:** 1.0 | **Status:** Approved
**Accountable owner:** Chief Architect | **Maintainer:** Enterprise Architecture

> Demo data. Names, dates and results are illustrative.

## Mandatory rules

Each rule names an architecture **check** (a fitness function in `arch.py`) that tests the architecture model in `spec.md`.

| ID | Rule | Why | Approver | Check |
|---|---|---|---|---|
| RULE-01 | Personal data never flows to an untrusted (external) zone. | Privacy obligations and provider contract terms | Privacy Officer | no-personal-data-external |
| RULE-02 | AI components may read systems of record, but never write to them. | Decisions that change enterprise records stay with people | Chief Architect | ai-read-only |

## How rules change

- Rules change only through an amendment with rationale, impact and approval.
- Every rule must trace to a requirement, control, task and validation, and the architecture must pass its check.

## Amendment log

| ID | Change | Approved by | Date |
|---|---|---|---|
| — | Initial version | Chief Architect | 2026-10-01 |
