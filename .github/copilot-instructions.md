# SDD rules for AI assistants in this repo

This repo is a Specification-Driven Development (SDD) architecture demo. The four files in `sdd/` are the source of truth:

- `sdd/constitution.md` — rules (RULE-##), each with an architecture check. Humans approve these. Never add or change a rule unless asked.
- `sdd/spec.md` — requirements (REQ-###), the architecture model (zones, components, flows), decisions (ADR-###) and controls (CTRL-###). Each control is enforced at a component in the model.
- `sdd/tasks.md` — tasks (TASK-###), each linked to a control.
- `sdd/validation.md` — validations (VAL-###), each linked to a control.

`architecture-diagram.md` is generated. Never edit it.

When a rule is added or changed:

1. Run `python arch.py check` to see which flows violate it.
2. Change the architecture model in `spec.md` so it conforms (add or change components and flows). Keep existing IDs; add new flow IDs at the end.
3. In `spec.md`, add a requirement with measurable acceptance criteria, an ADR for the design change (status `Proposed`, with the rejected alternative), and a control enforced at a real component.
4. Add a task in `tasks.md` (status `Not started`) and a validation in `validation.md` (result `Not run`).
5. Bump version numbers and add an entry to the amendment log in `constitution.md`.
6. Run `python arch.py check` again and fix any remaining gaps.

Never:
- Add files to `sdd/`. It holds exactly four files.
- Change existing IDs.
- Mark an ADR `Accepted`, a task `Done` or a validation `Pass`. Only accountable humans do that, with evidence.
- Change a rule to make the architecture pass. If a design need conflicts with a rule, stop and report the conflict.
- Edit anything in `demo/`.

Summarize what you changed and why, then stop for human review.
