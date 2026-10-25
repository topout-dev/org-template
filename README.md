# Apps

Your organization's apps on [Topout](https://www.npmjs.com/package/topout): a pnpm and Turborepo
monorepo. Apps live in `apps/<name>/` (or `apps/<team>/<name>/`), shared packages in `packages/`. A push
to the default branch builds every app in this repo's GitHub Actions and deploys the changed ones.

## Connect it

If `npx topout github connect <owner>/<repo>` made this repo, it's connected: skip to the next section.

If you made it from the template on GitHub, connect it as an admin of your Topout org:

```sh
pnpm install
pnpm topout github connect <owner>/<repo>
```

It opens the Topout GitHub App's install page (install it on **this repo only**), then asks you to
confirm in the browser and in the terminal.

**Staging:** until production exists, every repo sets the repo variable `TOPOUT_API_URL` to the API it
deploys to. `connect` sets it for you with the GitHub CLI, or prints the command:

```sh
gh variable set TOPOUT_API_URL --body <api origin> -R <owner>/<repo>
```

## Add an app

```sh
pnpm topout create app <name>
git add apps/<name> pnpm-lock.yaml
git commit -m "Add <name>"
git push
```

The first push registers the app. If it declares values (variables or secrets), the run says which
`pnpm topout vars set …` or `pnpm topout secrets set …` command to run; run it, then choose "Re-run
failed jobs" on the run.

- Preview locally with `pnpm topout dev` in the app's folder.
- See what's live with `pnpm topout status --app <name>`.
- Promote staging to production with `pnpm topout promote --app <name>` (a person runs it).

## Don't edit the workflow

`.github/workflows/topout.yml` only calls Topout's reusable workflow,
`topout-dev/actions/.github/workflows/deploy.yml@v1`, which builds in a job with read-only access to
this repo and no deploy token, then deploys from a second job that holds only the OIDC permission.
Topout accepts a deploy only from this file, on the default branch, calling a released version of that
workflow, so an edit can stop your deploys. Changes reach you when Topout moves `@v1`.

When `@v1` moves to a new CLI, each app in this repo redeploys on your next push, whether or not you
changed it: the artifact changes with the CLI, so no app is "unchanged since" the last deploy. Each
gets a new deployment with the same code and configuration, in the usual order (staging first on a
staged app).

## If your org restricts GitHub Actions

If your org's Actions policy allows only selected actions and reusable workflows, allowing
`topout-dev/actions/.github/workflows/deploy.yml@*` alone isn't enough: GitHub applies your policy to the
actions inside our workflow, and refuses the whole run before any job starts (`startup_failure`). Also
tick **"Allow actions created by GitHub"**: our workflow uses only GitHub's own actions
(`actions/checkout`, `actions/setup-node`, `actions/upload-artifact` and `actions/download-artifact`),
each pinned to a full commit SHA, so **"Require actions to be pinned to a full-length commit SHA"** is
fine too.

## Cloud roles that trust this repo

The deploy job can request GitHub OIDC tokens for any audience. If an AWS, Google Cloud or Azure role
trusts tokens from this repo, pin the role's trust policy to your own workflow's `job_workflow_ref`, not
just the repository, so Topout's deploy job can't assume it.
