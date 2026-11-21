# SDD Architecture Demo — AI Gateway (VS Code)

For architects and engineers. It's the same HR Policy Assistant as the first demo, but here the rules are checked against the **architecture model**, not just the documents. About 10 minutes.

**What it shows that demo 1 doesn't:**

- Rules are tested by **fitness functions** that run against the architecture model.
- The **diagram is generated** from the model, so it can't drift from it.
- Traceability includes **decisions (ADR)** and **controls tied to real components**.
- The check catches **architecture drift** and **rule conflicts**.

## Setup (once)

1. Install Python 3.8+ (no packages needed).
2. Open this folder in VS Code and install the recommended extension **Markdown Preview Mermaid Support** (VS Code prompts you).
3. Layout: `sdd/spec.md` and `sdd/constitution.md` on the left. On the right, open `architecture-diagram.md` and press `Ctrl+K V` for a live preview. The terminal goes at the bottom.
4. Optional: install Claude Code or GitHub Copilot for the AI step. Both read `CLAUDE.md` / `.github/copilot-instructions.md`.

**Run the check:** `Ctrl+Shift+B` (Mac: `Cmd+Shift+B`) or `python arch.py check`.
The check also regenerates the diagram, and the preview refreshes on its own.
**Before each rehearsal:** Terminal → Run Task → **SDD: Reset demo to start**

```
sdd/constitution.md    rules + the check each one runs
sdd/spec.md            requirements, architecture model (zones, components,
                       flows), decisions (ADR), controls (@component)
sdd/tasks.md           tasks per control
sdd/validation.md      validations per control
architecture-diagram.md  GENERATED from the model
arch.py                checker: traceability + conformance + diagram
demo/start, demo/solution   reset copies and fallback
docs/*.png             before/after diagrams for your slides
```

## Terminal commands

Prefer the terminal (PowerShell, or the VS Code terminal) to VS Code tasks? Run these from the repo root. Use `python3` on Mac/Linux.

| Command | Same as VS Code task | What it does |
|---|---|---|
| `python arch.py check` | SDD: Check architecture (`Ctrl+Shift+B`) | Checks traceability and conformance; regenerates the diagram |
| `python arch.py reset` | SDD: Reset demo to start | Copies `demo/start` into `sdd/` (2 rules), then runs the check |
| `python arch.py solution` | SDD: Load finished solution (fallback) | Copies `demo/solution` into `sdd/` (RULE-03 done), then runs the check |

To discard all local edits using git instead: `git checkout -- sdd architecture-diagram.md`

## Script

### 1. Baseline (2 min)
Walk the architecture model in `spec.md`: two trust zones, six components, five flows. Each flow has a data classification and an access type.
Show one full chain: **RULE-02 → REQ-002 → CTRL-002 @ hr-system → TASK-002 → VAL-002**.
Press `Ctrl+Shift+B`.

> ✅ Both rules: traced, validated, and the architecture conforms (ARCH ✅). **PASS.**

**Say:** "The ARCH column isn't someone's opinion. A fitness function tested the model."

### 2. The trigger (30 sec)
> "Security and Finance want every LLM call logged, redacted and cost-capped. We have five AI apps calling providers directly."

### 3. Add the rule (1 min)
In `sdd/constitution.md`, add under RULE-02:

```
| RULE-03 | All calls to external AI models go through the enterprise AI Gateway. | Central logging, redaction, cost limits and provider switching | Chief Architect | external-ai-via-gateway |
```

Press `Ctrl+Shift+B`.

> ❌ RULE-03: no requirement, **and** `F5 assistant → llm-provider calls an external AI model without going through the AI gateway`.

Point at the diagram: **F5 is now red.**
**Say:** "The rule found a real design flaw, not a paperwork gap."

### 4. Re-engineer and propagate (3 min). Choose one.

**A. With AI (recommended).** In Claude Code or Copilot Chat:

```
I added RULE-03 to sdd/constitution.md. Make the architecture conform and propagate
the change through spec, tasks and validation, following CLAUDE.md. Then run python arch.py check.
```

Review the diffs. **Say:** "AI proposed the design change and the paperwork. ADR-002 is *Proposed*. The Review Board decides."

**B. By hand.**

`sdd/spec.md`, **Architecture model**: in **Components**, add:
```
| ai-gateway | AI Gateway | gateway | internal | Platform team |
```
In **Flows**, replace the F5 row with:
```
| F5 | assistant | ai-gateway | internal | invoke | HTTPS + workload identity |
| F6 | ai-gateway | llm-provider | internal | invoke | HTTPS, egress allow-list |
```
Press `Ctrl+Shift+B`. ARCH turns ✅ and the diagram shows the gateway, but the rule is still ❌ (no requirement).
**Say:** "Fixing the design isn't enough. The intent must be specified and verifiable."

`sdd/spec.md`: add one row each to **Requirements**, **Architecture decisions** and **Controls**:
```
| REQ-003 | The assistant calls AI models only through the AI Gateway, which logs every call and enforces a monthly cost limit. | RULE-03 | 100% of model calls in gateway logs; direct calls to the provider are blocked; calls over the limit are rejected |
```
```
| ADR-002 | Route model calls through the enterprise AI Gateway | One place for logging, redaction, cost limits and switching providers; adds one network hop | Direct provider SDK calls from each app (every app re-implements controls) | Proposed |
```
```
| CTRL-003 | Only the AI Gateway may reach AI provider endpoints (egress allow-list); the gateway logs each call and enforces the cost limit | REQ-003 | ai-gateway | ADR-002 |
```

`sdd/tasks.md`:
```
| TASK-003 | Point the assistant at the AI Gateway; add the provider to the gateway egress allow-list; block direct egress from the app subnet; set the cost limit | CTRL-003 | Platform team | ADR-002 approved | Not started |
```

`sdd/validation.md`:
```
| VAL-003 | CTRL-003 | Call the provider directly from the app subnet; send 50 test questions and reconcile with gateway logs | Direct call blocked; 50/50 calls logged | Not run | — | Security Architect | — |
```

**C. Emergency.** Run Task → **SDD: Load finished solution (fallback)**.

### 5. Check again (1 min)
Press `Ctrl+Shift+B`.

> ⏳ RULE-03 → REQ-003 → CTRL-003 → TASK-003 → VAL-003, ARCH ✅. **Proposed: ADR-002 is Proposed; TASK-003 not started; validation not run.**

**Say:** "Conformant by design. Not yet approved, built or proven. This is a design check, not a build. TASK-003 is the work order, CTRL-003 says what must be true, and VAL-003 says how we'll prove it."

### 6. Optional: the path to green (2 min)
Simulate what happens after the demo, in this order, pressing `Ctrl+Shift+B` after each step:

1. **Review Board approves.** In `spec.md`, set ADR-002 status to `Accepted`. → ⏳ the ADR drops off the list.
   Valid ADR statuses: `Proposed`, `Accepted`, `Superseded`, `Rejected`. Anything else (for example `Approved`) → ❌.
2. **Engineers build it.** In `tasks.md`, set TASK-003 to `In progress`. → still ⏳.
3. **Try to cheat.** Set TASK-003 to `Done`. → ❌ *TASK-003 is marked Done, but its validation has not passed.*
   **Say:** "You can't declare victory without evidence."
4. **The test runs.** In `validation.md`, set VAL-003 Result to `Pass`, with a date, reviewer and evidence path. → ✅ **PASS.**

Reset afterwards if you want to rerun the optional beats from the ⏳ state: Run Task → **SDD: Load finished solution (fallback)**.

### 7. Optional: architecture drift (1 min)
A team adds a "quick" direct call. In `spec.md` **Flows**, add after F6:
```
| F7 | assistant | llm-provider | internal | invoke | HTTPS |
```
`Ctrl+Shift+B` → ❌ F7 bypasses the gateway, and it shows red in the diagram. Delete the row afterwards.
**Say:** "In CI, this blocks the pull request before it ships."

### 8. Optional: a rule conflict (1 min)
Product asks for the assistant to submit leave requests. In **Flows**, add after F6:
```
| F7 | assistant | hr-system | personal | write | HTTPS |
```
`Ctrl+Shift+B` → ❌ RULE-02: AI component writes to a system of record. Delete the row afterwards.
**Say:** "This isn't a coding decision. It needs a constitution amendment or an approved exception, with an owner, compensating controls and an expiry."

### 9. Close (30 sec)
- Rules are executable: each one runs a fitness function against the model.
- The diagram is generated from the model, so it can't drift from it.
- Every design change is traced: decision, control at a real component, task and evidence.
- AI accelerates the engineering; humans approve ADRs and accept evidence.

## Notes

- Names, dates and results are illustrative demo data.
- Fitness functions live in `make_checks()` in `arch.py`. Add a check there before a rule can reference it. In practice, a platform team owns this library.
- The checker reads tables by column header. Keep the headers if you adapt the files.
