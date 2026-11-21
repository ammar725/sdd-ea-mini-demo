# spec.md — HR Policy Assistant

**Version:** 1.1 | **Governed by:** constitution.md v1.1

## Objective

**OBJ-001:** Reduce routine HR policy questions handled by HR staff by 30% within 6 months.

## Requirements

| ID | Requirement | Rule | Acceptance criteria |
|---|---|---|---|
| REQ-001 | Prompts sent to the AI model contain no employee identifiers. The assistant may read the employee's location from the HR system to pick the right policy, but must not forward it. | RULE-01 | 0 employee identifiers in sampled prompts |
| REQ-002 | The assistant's identity has read-only access to the HR system. | RULE-02 | Any write attempt is denied and logged |
| REQ-003 | 🆕 The assistant calls AI models only through the AI Gateway, which logs every call and enforces a monthly cost limit. | RULE-03 | 100% of model calls in gateway logs; direct calls to the provider are blocked; calls over the limit are rejected |

## Architecture model

> `arch.py` checks this model against the constitution and generates `architecture-diagram.md` from it. Edit the tables, not the diagram.

### Zones

| ID | Name | Trust |
|---|---|---|
| internal | Corporate network | trusted |
| external | Third-party SaaS | untrusted |

### Components

| ID | Name | Type | Zone | Owner |
|---|---|---|---|---|
| employee | Employee | person | internal | — |
| chat-ui | Chat UI | app | internal | Digital Workplace |
| assistant | Assistant service | ai-app | internal | Platform team |
| ai-gateway | 🆕 AI Gateway | gateway | internal | Platform team |
| policy-library | HR policy library | data-store | internal | HR Policy Lead |
| hr-system | HR system of record | system-of-record | internal | HR IT |
| llm-provider | External LLM provider | ai-model | external | Procurement (vendor) |

### Flows

Data classification: `public`, `internal` or `personal`. Access: `read`, `write` or `invoke`.

| ID | From | To | Data | Access | Protocol |
|---|---|---|---|---|---|
| F1 | employee | chat-ui | personal | invoke | HTTPS |
| F2 | chat-ui | assistant | personal | invoke | HTTPS + OIDC token |
| F3 | assistant | policy-library | internal | read | HTTPS |
| F4 | assistant | hr-system | personal | read | HTTPS + OAuth2 client credentials |
| F5 | assistant | ai-gateway | internal | invoke | 🆕 HTTPS + workload identity |
| F6 | ai-gateway | llm-provider | internal | invoke | 🆕 HTTPS, egress allow-list |

## Architecture decisions

| ID | Decision | Rationale | Rejected alternative | Status |
|---|---|---|---|---|
| ADR-001 | Answer from the approved policy library using retrieval, not a fine-tuned model | Policies change often; retrieval keeps answers current | Fine-tuning (stale after every policy change) | Accepted |
| ADR-002 | 🆕 Route model calls through the enterprise AI Gateway | One place for logging, redaction, cost limits and switching providers; adds one network hop | Direct provider SDK calls from each app (every app re-implements controls) | Proposed |

## Controls

| ID | Control | Requirement | Enforced at | ADR |
|---|---|---|---|---|
| CTRL-001 | Redact employee identifiers from prompts before they leave the assistant | REQ-001 | assistant | ADR-001 |
| CTRL-002 | Assistant service identity holds a read-only role on the HR system | REQ-002 | hr-system | — |
| CTRL-003 | 🆕 Only the AI Gateway may reach AI provider endpoints (egress allow-list); the gateway logs each call and enforces the cost limit | REQ-003 | ai-gateway | ADR-002 |

## Open decisions

- 🆕 ADR-002 awaits Architecture Review Board approval. Owner: Chief Architect.
- 🆕 Monthly cost limit for the assistant. Owner: HR Director.
