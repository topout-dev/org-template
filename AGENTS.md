# Agent guide

This repo holds apps that deploy to Topout. Each app lives in `apps/<name>/` (or `apps/<team>/<name>/`)
with a `topout.config.ts`; shared code lives in `packages/`.

- **Deploy by merging to the default branch.** A push there builds every app in GitHub Actions and
  deploys the changed ones to their first environment. Nothing else deploys from this repo.
- **Never edit `.github/workflows/topout.yml`.** It's a frozen shim: the deploy logic lives in
  `topout-dev/actions`, and Topout accepts deploys only from this file calling its released workflow.
- **Preview locally with `pnpm topout dev`** in the app's folder.
- **Production changes only by a person:** `pnpm topout promote` (and `pnpm topout rollback`), run by a
  person who reads its pre-flight. Never run them yourself, and never work around them.
- **Read results with `pnpm topout status --json`** (add `--app <name>`), not by scraping the Actions log.
- Add an app with `pnpm topout create app <name>`, then commit `apps/<name>/` and `pnpm-lock.yaml`.
