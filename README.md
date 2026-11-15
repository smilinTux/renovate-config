# renovate-config

The shared Renovate policy for every smilinTux repository. A repo onboards with
one file, `renovate.json`:

```json
{ "extends": ["github>smilinTux/renovate-config"] }
```

and skcapstone, which carries the fleet lock, also extends the lock preset:

```json
{ "extends": ["github>smilinTux/renovate-config", "github>smilinTux/renovate-config:python-lock"] }
```

Change the policy here, by PR; every repo picks it up on Renovate's next run.

## What the policy does (`default.json`)

| Area | Setting |
| --- | --- |
| Coverage | Renovate's recommended managers: Python (pyproject, requirements, pip-compile/uv locks), npm (package.json and lockfiles), GitHub Actions, Dockerfiles and compose files |
| GitHub Actions | pinned to a commit SHA (`helpers:pinGitHubActionDigests`) |
| Docker | images pinned to a digest (`docker:pinDigests`) |
| Security | GitHub and OSV vulnerability alerts (`osvVulnerabilityAlerts`) open a PR at once, outside the schedule and the rate limits, labelled `security`, advisory link in the body; merged by Renovate when every check is green |
| Schedule | ordinary updates before 6am Monday (America/Chicago); releases must be 3 days old (not for security fixes) |
| Grouping | one weekly `minor updates` PR and one weekly `patch updates` PR per repo; majors ungrouped, one open at a time, release notes in the PR |
| Dashboard | a Dependency Dashboard issue per repo |
| Lock files | weekly lock-file maintenance PR |
| Merging | only patch updates and security fixes, only when ALL checks are green: Renovate merges itself (`platformAutomerge: false`), because GitHub auto-merge waits only for required checks. Minors and majors need a human or orchestrator review |
| Limits | 4 PRs an hour, 10 open (security PRs are exempt) |

## Caps (`caps.json`, included by default)

An upper bound on a third-party library stays only with a written reason and a
test that proves it (tracked in skcapstone `deploy/dependency-caps.md`). Renovate
never moves a capped requirement past its cap in an ordinary update, and once a
month opens a `cap-lift` PR for each cap with a release above it. That PR's CI
is the test.

## The fleet lock (`python-lock.json`)

skcapstone's `deploy/fleet-constraints.txt` is a uv-generated lock of the whole
fleet runtime. Renovate's `pip-compile` manager reruns the uv command recorded in
the lock's header, so a version bump arrives as a regenerated lock. That PR faces
skcapstone's fleet-lock gate (resolve on 3.12 and 3.14, `pip check`, smoke,
caps, security downgrades, vulnerability audit) and, once merged, reaches hosts
only through the canary auto-rollout.
