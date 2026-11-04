# country-state-picker

<!-- agent-rules:core (QualyHQ/standards templates/agent-machine-rules.md; edit there, sync with scripts/sync-agent-rules.mjs) -->
## Working on a developer machine

Applies to every coding agent (Claude Code, Codex, Cursor and others) on macOS, Windows and Linux.
Developers often run several sessions at once; keep CPU, memory and disk quiet.

- **Scratch work:** put extra clones, temporary installs, test databases and caches in the
  repo's own worktree folder (`.claude/worktrees/`) or the OS temp directory, never across the
  developer's repos folder. Delete a clone or worktree as soon as its work is pushed; keep at most
  two at a time. On macOS, prefer a temp folder whose name ends in `.noindex` (Spotlight skips it).
- **Tests:** run local tests on one worker, one suite at a time (Jest/Vitest `--maxWorkers=1`,
  Playwright `--workers=1`; never two suites in parallel) and pass `--forceExit` to Jest runs you
  script. Several sessions share the machine; parallel runs locked it. CI sizes its own workers.
- **Flaky tests:** run a new or changed test repeatedly before marking a PR ready (Playwright
  `--repeat-each=5`, Vitest `--sequence.shuffle --repeat=3`, Jest `--randomize`); it must pass every
  time. Wait for state, not time; no real clocks, randomness or network; each test sets up its own
  data. Never raise retries or timeouts to hide a flake (QualyHQ/standards STANDARDS "Tests that
  don't flake").
- **Processes:** stop every background process you started (test runners, mongod, dev servers,
  watchers) before finishing, and give long waits a time limit. Never stop processes another
  session or the developer started; report them instead.
- **Secrets:** never print a secret value: not `.env` contents, Secret Manager versions, CI
  secrets or Cloud Build trigger substitutions (they hold plaintext GitHub tokens). Print key
  names only, and never paste a value into a commit, PR, log or chat.
- **Words users see** (UI, emails, docs, templates, schemas): sector-agnostic, with no education
  terms (say contact, supplier or partner, item, tenant), and say "split", never "commission".
- **Qualy MCP:** when it is available, use it before guessing. It reaches product analytics
  (usage, funnels, session recordings), support tickets and chats, and production data through
  staff impersonation. Use it to size a feature by real usage, reproduce a reported issue, shape
  test fixtures and mocks like real accounts (data shapes and product usage across several
  tenants, business types and countries), check a script or migration dry run, and judge the
  risk of a deploy.
  - Read-only: no writes, sends, approvals or money actions through it unless the user asked for
    that specific action. If unsure whether a call reads or writes, stop and ask.
  - Least access: only the tenants and records the task needs; several when comparing data sets
    is the point. Say which tenants and why in the session, so the impersonation audit log is
    explainable.
  - Never copy real data out: no names, emails, amounts, ids or documents in fixtures, tests,
    commits, PR text, screenshots or logs. Take aggregates and shapes, and write synthetic data.
  - Never from CI.
- **Instrumentation is part of the feature.** When you build or change something people use,
  decide in the same change what we will want to know about it (is it used, by whom, where do
  people stall or fail) and add the events for it, following the service and frontend rules
  below. Name the question each new event answers in the PR's Details. Skip it, and say so, when
  nothing would be decided from the data: internal tools, automated jobs, plumbing.
- **Machine setup:** if installs or tests are unexpectedly slow, run
  `node scripts/dev-machine-check.mjs` from QualyHQ/standards and pass on what it reports
  (indexing and antivirus scanning of `node_modules` are the usual cause).

## Pull requests

- **Master:** changes land by PR only; direct pushes to `master` are refused (admins too).
- **Drafts first:** open PRs as drafts (`gh pr create --draft`) and iterate there; CI runs nothing
  on a draft, and a draft's `ci-gate` cancels itself so it can never merge on a skipped check.
- **Before `gh pr ready <n>`,** run locally, on one worker, and fix what fails: `npm run lint` and
  `npm run typecheck`; the tests related to your change (Jest
  `npx jest --maxWorkers=1 --findRelatedTests <files>`, Vitest
  `VITEST_MAX_WORKERS=1 npx vitest related --run <files>`); and the whole suite, one suite at a
  time, when you changed shared code (stores, composables, utils, config, `package.json`). CI is
  the confirmation, not the first test. One PR per real change, not one per tweak.
- **Format before you push.** A fresh clone or worktree has no git hooks until `npm ci` runs
  `prepare`, so nothing formats your commits there. Run `npm run lint` (and `format:check` where it
  exists) before every push; `npm run lint:fix` fixes most of it.
- **Advisories:** a high or critical production advisory fails CI. Fix it by upgrading; an
  exception in `package.json#qualyAdvisoryExceptions` (id, owner, reason, expiry) only when no fix
  exists or the fix is younger than the 3-day release age.
- **Review before ready.** Get an independent review of every non-trivial change before
  `gh pr ready`: Codex or Kimi through the `peer` skill when you have it, otherwise a fresh
  sub-agent given the diff and the goal, not your conclusions. UI work also gets the critic pass
  in the voice of Don Norman and Steve Krug, and a new feature, its name or its copy gets Rory
  Sutherland's read (questions in the kit's `docs/design-practices.md` "Critic pass"). Reviews
  correct the change; they do not widen it.
- **Open the PR complete, in one command:** `gh pr create --draft --body-file <file>`, plus
  `--label <risk>` only when the change needs one: `risk:money` when it touches money paths (CI
  fails without it), `risk:auth` when it changes who can do what, `risk:migration` when a script
  or migration must run. The body follows the template: `## Summary` first, 2 to 4 lines on why,
  filled; long text in `## Details`; any ADR in its 3-line block. The template and label checks
  run when the PR opens, so a body or label added afterwards leaves a red run behind.
- **After opening a PR:** watch it and fix it yourself until it merges, without being asked. In
  the Claude Code desktop app, turn on the PR's Auto-fix and CI monitor as soon as it is open;
  other agents keep checking. Merge on `ci-gate`. Fix each failure on the same branch: reproduce
  it locally on one worker, fix the cause, push. Each push starts a full run, so verify locally
  first and batch related fixes into one push. Never raise retries or timeouts, skip or delete a
  test, or bypass a gate to get green; a flaky test gets fixed, not rerun. Ask the owner after 3
  failed attempts on the same failure, or when the fix needs a product decision.
<!-- /agent-rules:core -->
