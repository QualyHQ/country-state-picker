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
- **PR template:** fill it: Summary under 4 lines, everything long in Details, summarise any ADR
  in the 3-line block, apply the risk label (`risk:money`, `risk:auth`, `risk:migration`).
- **After opening a PR:** watch it and fix it yourself until it merges, without being asked. In
  the Claude Code desktop app, turn on the PR's Auto-fix and CI monitor as soon as it is open;
  other agents keep checking. Merge on `ci-gate`. Fix each failure on the same branch: reproduce
  it locally on one worker, fix the cause, push. Each push starts a full run, so verify locally
  first and batch related fixes into one push. Never raise retries or timeouts, skip or delete a
  test, or bypass a gate to get green; a flaky test gets fixed, not rerun. Ask the owner after 3
  failed attempts on the same failure, or when the fix needs a product decision.
<!-- /agent-rules:core -->
