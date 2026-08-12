# Jenkins to GitHub Actions Migration Report

## Summary

Migrated the repository Jenkins pipelines to GitHub Actions workflows and archived the original Jenkinsfiles under `.github/ci-archive/`.

## Source Pipelines

| Original file | Pipeline type | Archived file | Replacement workflow |
| --- | --- | --- | --- |
| `Jenkinsfile` | Declarative Jenkins pipeline | `.github/ci-archive/Jenkinsfile` | `.github/workflows/build-and-deploy.yml` |
| `whenconditionscomplex/Jenkinsfile` | Declarative Jenkins pipeline | `.github/ci-archive/whenconditionscomplex/Jenkinsfile` | `.github/workflows/when-conditions-complex.yml` |

No Jenkins shared library calls were present, so no shared library expansion was required.

## Workflow Mapping

### Build and Deploy

- Jenkins `agent any` maps to `runs-on: ubuntu-latest`.
- Jenkins `sh 'npm install'` maps to an npm dependency installation step that uses `npm ci` when a lockfile is present and falls back to `npm install`.
- Jenkins `sh 'npm run build'` maps to a GitHub Actions build step.
- Jenkins `NETLIFY_SITE_ID = 'classy-paletas-f45a67'` maps to workflow environment variable `NETLIFY_SITE_ID`.
- Jenkins `credentials('netlify-token')` maps to GitHub Actions secret `NETLIFY_AUTH_TOKEN`.
- Jenkins Netlify deployment maps to a deploy step that runs only for non-pull-request events on `main` or `master`.

The repository currently does not contain a Node.js application manifest or build output directory, so the migrated workflow skips those commands when the corresponding files are absent. This keeps the migrated workflow valid for the current repository while preserving the npm build and Netlify deployment behavior when the application files are present.

### When Conditions Complex

- Jenkins environment values map to workflow-level `env` values.
- Jenkins `when { expression { false } }` maps to an environment-controlled condition, `if: ${{ env.RUN_SKIPPED_STAGE == 'true' }}`, with `RUN_SKIPPED_STAGE` defaulting to `false`.
- Jenkins `branch 'master'` maps to `if: ${{ github.ref_name == 'master' }}`.
- Jenkins `expression` conditions map to equivalent GitHub Actions expressions.
- Jenkins `allOf`, `anyOf`, and `not` conditions map to equivalent boolean expressions.
- Jenkins `echo` steps map to shell `echo` commands.

## Actions and Security

The workflows use verified GitHub-owned actions pinned to immutable commit SHAs:

| Action | Version tag | Pinned SHA |
| --- | --- | --- |
| `actions/checkout` | `v4` | `11d5960a326750d5838078e36cf38b85af677262` |
| `actions/setup-node` | `v4` | `49933ea5288caeca8642d1e84afbd3f7d6820020` |

Workflow permissions are restricted to `contents: read`.

## Required Secrets and Variables

| Jenkins credential or variable | GitHub Actions mapping | Required for |
| --- | --- | --- |
| `netlify-token` | Repository or environment secret `NETLIFY_AUTH_TOKEN` | Netlify production deployment |
| `NETLIFY_SITE_ID` | Workflow environment variable `NETLIFY_SITE_ID` | Netlify site selection |

## Validation

- GitHub Actions workflow syntax is intended to be validated with `actionlint`.
- Repository build/test validation is limited because this repository does not include application source files, package manifests, or existing test tooling.

## Knowledge Base Access

The migration issue requested fetching private knowledge-base files from `jenkins-migrations/.github-private`. The repository was discoverable, but file content requests returned `404 Not Found` from the available GitHub MCP tool, so this migration was completed using the requirements included in the issue body.
