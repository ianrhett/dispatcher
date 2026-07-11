# Brain setup

The brain is the orchestrating LLM that writes work packages and reviews
results. It needs write access to the repos it manages. This file
describes the connection, independent of which model you use.

## Requirements

- An LLM that can call tools and write to git (create branches, push
  files, read diffs, merge or close pull requests).
- Repo write access. The cleanest path is a hosted GitHub MCP connector
  authenticated by OAuth, so no personal access token sits on disk. Any
  mechanism that lets the model create branches, push files, and read
  diffs works.

## What the brain does each cycle

1. Writes `dispatch/WP-NNN.md` files with `status: ready`, one tightly
   scoped task each. Prefer several small WPs over one broad one.
2. After a worker pushes a review branch, reads the diff and the RESULT
   block directly from the repo.
3. Merges approved work (sets `done`) or closes it (sets `rejected`),
   then writes follow-up WPs for any concerns the worker raised.

## Keeping the queue healthy

- Stock 2 to 3 ready WPs per repo, not 20. The cap on "drain" exists so
  the brain reviews before flawed work compounds across many WPs.
- Squash-merge each WP so the history reads as one commit per package: a
  clean dispatch log.
- Treat worker escalations as the highest-signal output. A worker that
  stops and asks is the safety model working.

## Model division of labor

Brain turns are few and small (specs, reviews, decisions). Worker turns
are large (code churn). Put your most capable model on the brain and
cheap or included models on the workers. This keeps the expensive
judgement where token volume is lowest.
