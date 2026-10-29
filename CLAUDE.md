# country-state-picker

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
- **Browser tests before merge:** browser flows (Playwright) run only after the merge, on the
  `master` and `staging` pushes, and a failure there blocks the staging deploy for everyone. So
  a UI change never reaches `master` (or `v2/master`) without them passing: before merging, run
  `npm run test:e2e` locally on every device project (desktop, tablet and mobile) with at most
  two workers (`--workers=2`), or add the `full-ci` label so the PR runs the full tier, browser
  flows included. When a change removes or renames a test ID, a page or a route, search
  `test/e2e` for it and update those tests in the same PR.
- **Processes:** stop every background process you started (test runners, mongod, dev servers,
  watchers) before finishing, and give long waits a time limit. Never stop processes another
  session or the developer started; report them instead.
- **Machine setup:** if installs or tests are unexpectedly slow, run
  `node scripts/dev-machine-check.mjs` from QualyHQ/standards and pass on what it reports
  (indexing and antivirus scanning of `node_modules` are the usual cause).
- **PR template:** fill it: Summary under 4 lines, everything long in Details, summarise any ADR
  in the 3-line block, apply the risk label (`risk:money`, `risk:auth`, `risk:migration`).
- **After opening a PR:** stay with it until it merges. Once it is ready, watch its CI (in the
  Claude Code desktop app, turn on the PR's Auto-fix and CI monitor; other agents keep checking).
  Fix each failure on the same branch: reproduce it locally on one worker, fix the cause, push.
  Each push starts a full run, so verify locally first and batch related fixes into one push.
  Never raise retries or timeouts, skip or delete a test, or bypass a gate to get green; a flaky
  test gets fixed, not rerun. Ask the owner after 3 failed attempts on the same failure, or when
  the fix needs a product decision.
- **Test against production reality (Qualy MCP):** when the Qualy MCP is available, use it (staff
  impersonation) to check a change against real data: reproduce a reported issue, validate a
  script or migration dry run, confirm a query or report matches what production holds.
  - Read-only: no writes, sends, approvals or money actions through it unless the user asked for
    that specific action. If unsure whether a call reads or writes, stop and ask.
  - Least access: one tenant and the fewest records the task needs. Say which tenant and why in
    the session, so the impersonation audit log is explainable.
  - Never copy real data out: no names, emails, amounts, ids or documents in fixtures, tests,
    commits, PR text, screenshots or logs. Turn what you learned into synthetic fixtures of the
    same shape.
  - Never from CI.
- **Master:** changes land by PR only; direct pushes to `master` are refused (admins too).
- **`[skip ci]`:** never on a commit to `master` (or its PR merge) in a repo that deploys. GitHub
  runs no workflow for a push whose head commit carries it, so promoting that commit tests and
  deploys nothing. Docs-only changes are already cheap (the `none` tier). Only the packages'
  release bot uses it.
- **Deploys:** never commit, cherry-pick or merge into `staging` or `production` (or `deploy/*`).
  Land changes on the working branch (`master`) by PR, then promote by fast-forward
  (`git push origin origin/master:staging`, then `origin/staging:production`); see
  QualyHQ/standards `docs/runbooks/promote.md`. A commit that reaches an env branch any other way
  breaks every later promotion and skips the tests that gate deploys.
