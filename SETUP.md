# Setup

Stand up a dispatch loop in one repo. About ten minutes for a fresh
repo; add the reconciliation step for a repo with history.

## 0. Preflight checklist (every repo, before anything runs)

- [ ] docs/PRD.md exists and is ratified (intent, goals, non-goals).
      No PRD, no lane: lane heads hard-stop without one.
- [ ] docs/ROADMAP.md exists with a "Now" section (current focus and
      next action).
- [ ] dispatch/README.md and the worker rule are in place (step 1).
- [ ] LEGACY RECONCILIATION, for repos with history: inventory any
      pre-existing authority docs (a CLAUDE.md with operating rules,
      STATE.md, an existing WP ledger or ID scheme, an alternate PRD
      file). Write a reconciliation ADR in docs/adr/ that: merges
      legacy PRD content into docs/PRD.md, folds STATE-style truth
      into the roadmap Now, maps legacy WP IDs to references, and
      rules on each legacy operating rule individually. Safety rules
      ("live systems read-only", "email is radioactive") are ALWAYS
      retained and promoted to hard rails. The conformance work is
      the lane's first WP. Skipping this step gives your lane head
      two constitutions and a stall.
- [ ] Labels available on the repo/org as used here: needs-approval
      (gated PRs), and on the tower repo: escalation, operator.
- [ ] Repo setting: Settings -> General -> Pull Requests -> check
      "Automatically delete head branches".
- [ ] Parked lanes: create dispatch/PARKED (any content) to freeze a
      lane; workers and lane heads skip it and the watchdog stays
      quiet about it. Delete the file to unpark.

## 1. Add the files

Copy into the target repo:

- `dispatch/README.md` (the protocol)
- `dispatch/WP-000-example.md` (template; already marked done)
- `.cursor/rules/next.mdc` (the worker trigger; adapt for non-Cursor
  agents using that tool's rules mechanism)

Recommended document layers alongside the queue:

- `docs/PRD.md`: requirements. What and why.
- `docs/ROADMAP.md`: ordered decomposition, with a "Now" section.
- `docs/adr/`: architecture decision records. Copy
  `templates/adr-template.md`.

Commit to the default branch. The queue is now initialized.

## 2. Connect the brain

Give your orchestrating LLM write access to the repo. See
[prompts/brain-setup.md](prompts/brain-setup.md). Verify it can read
the repo and push a branch before relying on it.

## 3. Open the repo as a worker lane

Open the repo as its own workspace window in your coding agent (in
Cursor: Open Folder, not a multi-root workspace). One repo, one
window, one agent chat.

## 4. Configure autonomy and hygiene

Enable agent auto-run, then scope it:

- Allowlist: git checkout, pull, fetch, branch, add, commit, push,
  plus sed, grep, gh, and your test runner.
- Denylist: git push --force, rm -rf, anything piping a download to a
  shell.

Note: if your agent has a separate sandbox/auto-review layer, keep it
on; a complete allowlist plus unambiguous rules (the kit's sync
carve-out exists for this) means it only pauses for genuinely novel
commands.

Lane sync is the worker's job (protocol step 0). The operator never
runs git.

## 5. First run: "next"

Write one small WP (copy WP-000's shape, set status: ready). In the
lane, say "next". Watch one full cycle before you stop watching.

## 6. Steady state: "drain" or a lane manager

Operator says "drain" and walks away, or (recommended) run a lane
manager session per prompts/lane-head.md that stocks, dispatches,
reviews, and gates without the operator.

## 7. Scale out

Repeat per repo. One worker per repo; different repos in parallel;
never parallel within a repo. Keep an operator queue and a watchdog
so the system's asks reach a phone, not a screen.
