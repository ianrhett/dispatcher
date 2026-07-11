# Dispatch Protocol

The queue between the brain (an orchestrating LLM with repo write
access) and workers (coding agents, one per repo). State lives in this
directory. Git is the single source of truth. No other state store
exists.

## Roles

- Brain: writes WP files, reviews RESULT blocks, sets status to done or
  rejected, merges or closes branches.
- Worker: one agent per repo. Claims WPs, executes them, reports back.
  Never works outside the claimed WP scope.
- Operator: says "next" or "drain" to a worker. Merges approved work.

## WP file format

Each work package is one markdown file: `dispatch/WP-NNN.md`

```
---
status: ready
branch: wp-NNN-short-name
executor: any
---
# WP-NNN: Title

## Context
Why this exists, in two or three sentences.

## Task
The concrete steps. Be specific enough that scope is unambiguous.

## Acceptance
- [ ] Checkable criteria

## Out of scope
- Everything the worker must NOT touch
```

Status lifecycle: `ready -> claimed -> review -> done | rejected`

Workers may set: `claimed`, `review`.
Only the brain sets: `done`, `rejected`.

## Worker protocol

1. Find the lowest-numbered WP with `status: ready`. If none, report
   "queue empty" and stop.
2. Create the branch named in the frontmatter from the default branch.
3. On that branch, flip status to `claimed`. Commit: "claim WP-NNN".
4. Execute the Task. Stay inside Acceptance and Out of scope. Any repo
   rules (style, hard stops) apply in full.
5. Append a RESULT section to the WP file:

```
## RESULT
- Built:
- Files touched:
- Tests:
- Concerns:
- Escalations:
```

6. Flip status to `review`. Commit: "WP-NNN: <summary>". Push the branch.
7. Print the RESULT block in the session so the operator sees it.

## Rules

- One WP per branch. One worker per repo. No parallel WPs in one repo.
- A worker never edits another WP, this README, or agent rules files.
- Escalations are questions for the brain, not blockers to invent
  answers for. When in doubt, escalate and stop.
- If acceptance criteria cannot be met, still push the branch with
  RESULT explaining why, status `review`. Never abandon silently.
