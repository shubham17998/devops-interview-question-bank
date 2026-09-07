# GitHub Actions Interview Questions

## Core Concepts
Q: What is a Workflow in GitHub Actions?\
Ans: A YAML file (under `.github/workflows/`) that defines an automated process made up of one or more jobs, triggered by events like pushes, pull requests, schedules, or manual dispatch.

Q: What is a Job in GitHub Actions?\
Ans: A set of steps that execute on the same runner. Jobs run in parallel by default, unless you declare dependencies between them with `needs`.

Q: What is an Action in GitHub Actions?\
Ans: A reusable unit of code (JavaScript, Docker container, or composite) that performs a specific task — e.g., `actions/checkout`, `actions/setup-node`. Actions are the building blocks you call inside a job's steps.

Q: What is a Runner in GitHub Actions?\
Ans: The machine (VM or container) that executes a job. Can be a GitHub-hosted runner (Ubuntu, Windows, macOS images maintained by GitHub) or a self-hosted runner you register and manage yourself.

Q: In a GitHub Actions workflow, what is the `on` attribute used for?\
Ans: It defines the event(s) that trigger the workflow — e.g., `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual), `release`, or `repository_dispatch` (external trigger via API).

Q: Are jobs in a GitHub Actions workflow executed in parallel by default?\
Ans: Yes. Unless a job specifies `needs: [other_job]`, all jobs in a workflow start running concurrently on separate runners.

Q: How do you create a dependency between jobs so one runs after another?\
Ans: Use the `needs` keyword: `needs: build` makes a job wait until `build` completes successfully before starting. You can also reference outputs from the job it depends on via `needs.build.outputs.<name>`.

## Workflow Types
Q: What are the different types of GitHub Actions workflows?\
Ans: Push-triggered, pull-request-triggered, scheduled (cron), manually-triggered (`workflow_dispatch`), reusable workflows (`workflow_call`), and event-triggered by other repos/services (`repository_dispatch`).

Q: What is a Push workflow?\
Ans: A workflow triggered by `on: push`, running whenever commits are pushed to matching branches/tags — typically used for build/test/deploy on merge.

Q: What is a Pull Request workflow?\
Ans: A workflow triggered by `on: pull_request` (or `pull_request_target`), commonly used to run tests, linting, and status checks before a PR can be merged.

Q: What is a Scheduled workflow?\
Ans: A workflow triggered by `on: schedule` using cron syntax, used for periodic tasks like nightly builds, dependency scans, or cleanup jobs.

Q: What is a Manual workflow?\
Ans: A workflow triggered by `on: workflow_dispatch`, letting a user manually trigger it from the Actions UI or API, optionally passing input parameters.

Q: What is a Reusable workflow?\
Ans: A workflow defined with `on: workflow_call`, which other workflows can invoke via `uses: org/repo/.github/workflows/file.yml@ref`, passing inputs/secrets — lets you centralize and share entire pipelines across repos.

Q: What is a Composite Action?\
Ans: An action (defined with `runs: using: composite`) that bundles multiple steps into a single reusable action, invoked with a single `uses:` line in a job's steps — good for sharing a handful of steps rather than a whole workflow.

Q: Difference between Reusable Workflow and Composite Action?\
Ans:
| | Reusable Workflow | Composite Action |
|---|---|---|
| Scope | Whole workflow (jobs, runs-on, strategy) | A set of steps within one job |
| Invocation | `jobs.<id>.uses:` | `steps[].uses:` |
| Can define its own jobs/runner | Yes | No — runs on the caller's runner |
| Use case | Share entire CI/CD pipelines | Share a common sequence of steps |

Q: What is workflow_dispatch?\
Ans: An event trigger that adds a manual "Run workflow" button in the GitHub UI (and an API endpoint), letting you trigger a workflow on demand and optionally supply typed input parameters.

## Security
Q: How do you secure GitHub Actions secrets?\
Ans: Store them in GitHub's encrypted **Secrets** (repo, environment, or org-level) rather than in code. Use **Environments** with required reviewers/protection rules to gate access to production secrets. Scope secrets to the minimum set of workflows/environments that need them, avoid printing secrets in logs (GitHub auto-masks known secret values), and prefer OIDC federation to cloud providers (e.g., AWS `aws-actions/configure-aws-credentials` with an IAM role) over long-lived cloud access keys.

Q: How do you handle secrets in CI/CD?\
Ans:
- Store secrets in a dedicated secrets manager (GitHub Secrets, AWS Secrets Manager, HashiCorp Vault) — never in the repo or plaintext config.
- Inject them as environment variables/masked build variables at runtime, not baked into images.
- Use short-lived, scoped credentials (OIDC-based federation) instead of long-lived static keys wherever possible.
- Restrict which branches/environments can access which secrets.
- Rotate regularly and audit access.
- Scan repos/commits for accidentally committed secrets (GitGuardian, gitleaks, trufflehog) and revoke immediately if one leaks.

## Optimization
Q: How do you optimize GitHub Actions execution time?\
Ans:
- Cache dependencies (`actions/cache`) for package managers (npm, pip, Maven, etc.) and Docker layers.
- Run independent jobs in parallel instead of chaining them unnecessarily with `needs`.
- Use a build matrix only where truly needed, and fail fast (`strategy.fail-fast`) to cut wasted runs.
- Use `paths`/`paths-ignore` and `branches` filters so workflows only run when relevant files change.
- Prefer smaller/leaner runner images or self-hosted runners with pre-warmed toolchains for heavy pipelines.
- Split large monolithic jobs into smaller concurrent jobs, and use `concurrency` to cancel superseded runs on the same branch/PR.
