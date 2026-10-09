# do-gh-shared-actions

Shared CI/CD automation layer: the central library of reusable GitHub Actions workflows.
Every data pipeline repo calls these with `workflow_call` and pins a release tag, so a fix
here reaches all repos as soon as the tag moves.

| Workflow | What it does |
|---|---|
| `pull-request-checks.yaml` | PR title (conventional commits), pre-commit, PyTest; optional terraform fmt + tflint |
| `glue-build-publish-deploy.yaml` | Uploads the `src/glue/` entry script + zip to S3, writes `glue_job_config.yaml` into `do-aws-pipeline-infra` |
| `lambda-build-publish-deploy.yaml` | Zips `src/lambda/` to S3, writes `config.yaml` into `do-aws-pipeline-infra` |
| `unlock-terraform-state.yaml` | Emergency `terraform force-unlock` for one infra path |

## Calling

```yaml
jobs:
  checks:
    uses: KasiDandu/do-gh-shared-actions/.github/workflows/pull-request-checks.yaml@release/v1
```

Callers must grant `id-token: write` and pass `INFRA_REPO_TOKEN` (a PAT; pushes made with
`GITHUB_TOKEN` do not trigger the infra pipeline).

## Releasing

```bash
git tag -f release/v1 && git push -f origin release/v1   # move v1 to the latest fix
git tag release/v2 && git push origin release/v2          # breaking change: new major tag
```
