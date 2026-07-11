# Setup

Stand up a dispatch loop in one repo. About ten minutes.

## 1. Add the files

Copy into the target repo:

- `dispatch/README.md` (the protocol)
- `dispatch/WP-000-example.md` (template; already marked done)
- `.cursor/rules/next.mdc` (the worker trigger; adapt for non-Cursor
  agents using that tool's rules mechanism)

Recommended document layers alongside the queue:

- `docs/PRD.md`: requirements. What and why.
- `docs/ROADMAP.md`: ordered decomposition, with a "Now" section for
  current truth and next action.
- `docs/adr/`: architecture decision records. Copy
  `templates/adr-template.md`. The brain writes one whenever it makes
  a decision with consequences.

Commit to the default branch. The queue is now initialized.

## 2. Connect the brain

Give your orchestrating LLM write access to the repo. See
[prompts/brain-setup.md](prompts/brain-setup.md). Verify it can read the
repo and push a branch before relying on it.

## 3. Open the repo as a worker lane

Open the repo as its own workspace window in your coding agent (in
Cursor: Open Folder, not a multi-root workspace). One repo, one window,
one agent chat. That window is this repo's lane.

## 4. Configure autonomy and hygiene

In the agent's settings, enable auto-run so it does not ask per command,
then scope what it may run:

- Allowlist: git add, commit, push, checkout, branch, fetch, pull, and
  your test runner.
- Denylist: git push --force, rm -rf, and anything piping a download to
  a shell.

In the repo's GitHub settings, enable "Automatically delete head
branches" so merged WP branches clean themselves up.

Lane sync is the worker's job, not the operator's: protocol step 0 has
the worker fetch, check out the default branch, and pull --rebase
before selecting work and between WPs. The operator never runs git.

## 5. First run: "next"

Write one small WP (copy WP-000's shape, set `status: ready`). In the
lane, say "next". Watch it sync, claim, branch, execute, append RESULT,
flip to review, and push, exactly once. Confirm the mechanics before
you stop watching.

## 6. Steady state: "drain"

After the first run proves out, the operator says "drain" and walks
away. The worker processes up to three ready WPs and reports. The brain
reviews from its side, merges the good, and restocks the queue.

## 7. Scale out

Repeat steps 1 and 3 to 6 in each repo. One worker per repo, the brain
stocking queues across all of them from one seat. Different repos run in
parallel; work never runs in parallel within a repo.
