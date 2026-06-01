# cicd-pipeline-templates

A library of **reusable GitHub Actions workflows** for containerised apps — build, test, scan, sign,
and deploy — plus a sample application that consumes them end to end. Drop the reusable workflows
into any repo and call them with a few lines instead of copy-pasting hundreds of YAML lines per
project.

## Why reusable workflows

Most teams copy the same pipeline into every repo, then maintain N divergent copies. These templates
live in one place and are called via `workflow_call`, so a fix or hardening change propagates to
every consumer.

## Templates

| Workflow | Purpose |
|----------|---------|
| [`reusable-build-test.yml`](.github/workflows/reusable-build-test.yml) | Lint, run tests, build a Docker image |
| [`reusable-security-scan.yml`](.github/workflows/reusable-security-scan.yml) | Trivy image scan + gitleaks secret scan |
| [`reusable-build-push-sign.yml`](.github/workflows/reusable-build-push-sign.yml) | Build, push to GHCR, and sign the image with cosign |
| [`reusable-deploy.yml`](.github/workflows/reusable-deploy.yml) | Deploy a built image (kubectl/Helm placeholder) |

## How a consumer uses them

```yaml
# .github/workflows/ci.yml in YOUR app repo
name: CI
on: [push]
jobs:
  build-test:
    uses: durrello/cicd-pipeline-templates/.github/workflows/reusable-build-test.yml@main
    with:
      language: python
      image-name: my-app

  security:
    needs: build-test
    uses: durrello/cicd-pipeline-templates/.github/workflows/reusable-security-scan.yml@main
    with:
      image-name: my-app
```

See [`examples/app/.github/workflows/ci.yml`](examples/app/.github/workflows/ci.yml) for a working
caller wired to the sample app.

## Pipeline flow

```
build-test ──► security-scan ──► build-push-sign ──► deploy
   lint            Trivy            GHCR + cosign        kubectl/Helm
   test            gitleaks         (signed image)
   docker build
```

## Sample app

[`examples/app`](examples/app) is a tiny Flask service with a Dockerfile and a test, used to prove
the templates actually run.

## Notes

- Image signing uses [cosign](https://github.com/sigstore/cosign) keyless OIDC — no long-lived keys.
- The deploy job is a safe placeholder; wire it to your cluster by adding `KUBE_CONFIG` secrets and
  uncommenting the apply step.

## License

MIT
