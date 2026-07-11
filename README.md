# dispatcher

A git-backed work queue for autonomous coding agents.

The brain (an orchestrating LLM with repo write access) issues tightly
scoped work packages. Workers (coding agents, one per repo) claim and
execute them in isolated branches. Git is the only state store. There is
no server, no database, no message bus, no API metering, and no
copy-paste.

## Why this exists

Coordinating AI coding agents usually means one of two bad options:

1. Copy-paste between a chat model and a coding tool. Productive, but a
   manual tax you pay on every hand-off.
2. A bespoke orchestration server that dispatches work over API calls.
   Powerful, but it meters every token and becomes a thing you maintain
   instead of the thing you wanted to build.

dispatcher is the third option. The insight: a shared queue for agents
decomposes into a message store plus something that wakes each agent.
Git is already a message store with auth, history, and diffs solved. The
wake-up is a human (or a cron) saying one word to an agent that reads
the repo it is already sitting in. Every participant runs on a
subscription it already has. Nothing touches a per-token meter.

## The mental model

- **Brain**: writes work-package (WP) files, reviews results, merges or
  rejects. Connects to git through whatever gives it repo write access
  (e.g. the GitHub MCP connector). Few, small turns: specs and reviews.
- **Worker**: one agent per repo, running locally in that repo's
  workspace. Claims WPs, executes them on a branch, reports back. Many,
  large turns: actual code churn. Cheap hands.
- **Operator**: a human. Says "next" or "drain" to a worker. Merges what
  the brain approves. That is the entire operational surface.
- **Queue**: the `dispatch/` directory in each repo. One markdown file
  per WP. Frontmatter carries status and branch. Git history is the log.

Put maximum judgement where token volume is lowest (the brain) and cheap
execution where volume is highest (the workers).

## Status lifecycle

```
ready  ->  claimed  ->  review  ->  done
                                \->  rejected
```

Workers may set `claimed` and `review`. Only the brain sets `done` or
`rejected`. State lives in the WP file's frontmatter; git is the single
source of truth.

## The loop

1. Brain writes `dispatch/WP-NNN.md` with `status: ready`.
2. Operator says "next" (one WP) or "drain" (up to N WPs) to the worker.
3. Worker claims the lowest-numbered ready WP, branches, executes inside
   scope, appends a RESULT block, flips to `review`, pushes.
4. Brain reads the branch, reviews the diff, merges (`done`) or closes
   (`rejected`) and writes a follow-up WP for any concerns.

No paste anywhere. The operator's only input is one trigger word and a
merge click, and the merge can move to the brain too.

## Parallelism

One worker per repo, one repo per workspace window. Different repos run
in parallel; work never runs in parallel *within* a repo (one WP per
branch keeps reviews tractable). The brain stocks queues across many
repos from one seat.

## Safety model

A misbehaving worker's blast radius is a branch nobody merges. Four
layers keep it in its lane:

1. **WP scope**: it can only do what the WP states.
2. **Branch isolation**: it never commits to main, never merges.
3. **Hard rails** (in the worker rule): no new dependencies, endpoints,
   schema, or guessing through ambiguity; escalate instead.
4. **Brain review before merge**: the one control that makes the rest
   safe to automate.

## Get started

See [SETUP.md](SETUP.md). Roughly: drop `dispatch/` and
`.cursor/rules/next.mdc` into a repo, bootstrap a local agent with
[prompts/worker-bootstrap.md](prompts/worker-bootstrap.md), write a WP,
say "next."

## Portability

The worker rule here is written for Cursor (`.cursor/rules/next.mdc`),
but nothing about the protocol is Cursor-specific. Any coding agent that
can read a repo, run git, and follow a markdown spec works: Claude Code,
Codex CLI, or others. Point it at `dispatch/README.md` via that agent's
own rules mechanism.

## License

MIT. See [LICENSE](LICENSE).
