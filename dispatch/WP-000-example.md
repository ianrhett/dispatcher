---
status: done
branch: wp-000-example
executor: any
---
# WP-000: Example work package

This file is a template and a worked example. Copy its shape when
writing real WPs. It is marked `done` so a worker skips it.

## Context

A good WP is small enough that the worker cannot go off the rails and
specific enough that "done" is unambiguous. Context tells the worker why
the task exists so it makes sane micro-decisions inside scope.

## Task

1. State concrete, ordered steps.
2. Name exact files or directories where possible.
3. Prefer several tight WPs over one broad one.

## Acceptance

- [ ] Each criterion is checkable by looking at the diff
- [ ] Criteria cover the whole task, nothing implicit
- [ ] The worker records test results in RESULT

## Out of scope

- Anything not named in Task
- Refactors, cleanups, or "while I'm here" changes
- Editing other WPs, the dispatch README, or rules files

## RESULT

- Built: (worker fills this in: what was produced)
- Files touched: (exact paths)
- Tests: (command run and pass/fail counts)
- Concerns: (anything the brain should know; not blockers)
- Escalations: (open questions; the worker stops rather than guessing)
