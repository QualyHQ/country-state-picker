# country-state-picker

<!-- agent-rules:core (QualyHQ/standards templates/agent-machine-rules.md; edit there, sync with scripts/sync-agent-rules.mjs) -->
## Working on a developer machine

For every coding agent (Claude Code, Codex, Cursor and others). Several sessions share the
machine; keep CPU, memory and disk quiet. Topic docs named "standards `docs/agents/<x>.md`" are
read on demand: `gh api repos/QualyHQ/standards/contents/docs/agents/<x>.md -H 'Accept: application/vnd.github.raw'`.

- **Scratch work** goes in `.claude/worktrees/` or a temp folder (on macOS one ending in
  `.noindex`), never across the repos folder. Delete a clone or worktree once its work is pushed;
  keep at most two.
- **Tests:** on a shared developer machine, one worker, one suite at a time (Jest/Vitest
  `--maxWorkers=1`, Playwright `--workers=1`). `GITHUB_ACTIONS=true` (CI) or `CLAUDE_CODE_REMOTE=true`
  (cloud session) means no shared machine: use what the runner has, unless a suite needs fewer for shared
  state or memory. Neither set: assume shared. `--forceExit` on Jest runs you script. A new or
  changed test passes repeated runs (Playwright `--repeat-each=5`, Vitest
  `--sequence.shuffle --repeat=3`, Jest `--randomize`): wait for state, not time; no real clocks,
  randomness or network. Never raise retries or timeouts.
- **Processes:** stop every background process you started and time-limit long waits. Never stop
  one you did not start; report it.
- **Secrets:** never print a secret value (`.env`, Secret Manager, CI secrets, Cloud Build trigger
  substitutions, which hold plaintext GitHub tokens). Key names only. A workflow that reads a
  secret at run time gets its own narrow one (standards `docs/agents/runtime-secrets.md`).
- **Words users see:** sector-agnostic, no education terms (contact, supplier or partner, item,
  tenant); say "split", never "commission".
- **No regressions:** we move money, so tests lock in current behaviour and a behaviour change
  must turn something red. Cover every path and money edge case, not the happy path; a bug fix
  ships with its test (standards `docs/agents/testing.md`).
- **ADRs:** a big feature, a money or auth change, or anything hard to reverse gets an ADR,
  `Proposed` before you build and accepted by the owner (standards `docs/agents/adr.md`).
- **Lint and CI rules:** check what lint, the hooks and CI already enforce before writing code, and
  never silence them (`eslint-disable`, `@ts-ignore`, skipped tests) to get green.
- **Qualy MCP** (product analytics, tickets, production data by impersonation): use it before
  guessing, read-only, and never copy real data out. Read standards `docs/agents/qualy-mcp.md` first.
- **Instrumentation is part of the feature:** decide what we will want to know about a new feature
  and add its events in the same change (standards `docs/agents/analytics.md`).
- **MCP tools are part of the feature:** every new feature or endpoint ships the MCP tools an LLM
  needs to use it, as a first-class interface like the UI and API, or the PR's Details say why none
  (standards `docs/agents/mcp.md`).
- **Slow machine?** Run `node scripts/dev-machine-check.mjs` from QualyHQ/standards.

## Pull requests

- **Master** takes changes by PR only. Open PRs as drafts; CI runs when `gh pr ready <n>`.
- **Open the PR complete, in one command:** body from the template (`## Summary` first, filled) and
  a risk label only when one applies. The checks run on open, so a later fix leaves a red run.
- **Before `gh pr ready`,** at the worker limit under Tests: lint, typecheck, the related tests, the whole suite when
  you changed shared code; and an independent review (Codex or Kimi via `peer`, else a fresh
  sub-agent). Detail: standards `docs/agents/pull-requests.md` and `docs/agents/reviews.md`.
- **After opening a PR:** watch it and fix it yourself until it merges, without being asked. In the
  Claude Code desktop app, turn on Auto-fix and CI monitor as soon as it opens. Merge on `ci-gate`.
  Reproduce each failure locally, fix the cause, batch related fixes into one push. Never skip a
  test or bypass a gate; ask the owner after 3 failed attempts or when it needs a product decision.
- **One push per round on a ready PR:** every push reruns its whole CI tier. Iterate in the draft;
  collect all fixes (review, CI, follow-ups) into one push; never push to see what CI says. More
  coming? `gh pr ready --undo` until it's in.
- **Merged isn't done:** watch `master`'s run for your merge commit, promote, and report it live or
  say what stopped it. An "Advisory blocks the deploy" warning means it won't deploy yet.
- **Tickets and ideas** (support-core ADR-062): a PR that fixes a support ticket carries
  `Fixes: TKT-…` in its body (`Ticket: TKT-…` for related work); one that builds an idea carries
  `Implements: IDEA-…` (`Idea: IDEA-…` for related). Then confirm the link with the Qualy MCP
  `ticket_link_add`. The ticket follows the PR on its own; never close a ticket or email a customer
  because a PR merged.
<!-- /agent-rules:core -->
