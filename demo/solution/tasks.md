# tasks.md — HR Policy Assistant

**Implements:** spec.md v1.1

> A task is Done only when the validations for its control pass.

| ID | Task | Control | Owner | Depends on | Status |
|---|---|---|---|---|---|
| TASK-001 | Add a redaction step to the assistant's prompt builder | CTRL-001 | Platform team | — | Done |
| TASK-002 | Grant the assistant's service identity a read-only HR role | CTRL-002 | IAM team | — | Done |
| TASK-003 | 🆕 Point the assistant at the AI Gateway; add the provider to the gateway egress allow-list; block direct egress from the app subnet; set the cost limit | CTRL-003 | Platform team | ADR-002 approved | Not started |
