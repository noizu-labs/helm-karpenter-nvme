# CI/CD

## Workflows

### Lint and Test (`.github/workflows/lint-test.yaml`)

Triggered on PRs that modify `*.yaml`, `*.tpl`, or `*.json` files.

Steps:
1. `helm lint . -f tests/test-values.yaml` — validates chart structure and values.
2. `helm template test-release . -f tests/test-values.yaml` — renders templates and catches Go template errors.

No cluster-level tests (ct install) since the chart produces Karpenter CRD instances that require a running Karpenter controller.

### Release (`.github/workflows/release.yaml`)

Triggered on tag push matching `v*`.

Steps:
1. Authenticates to GHCR via `GITHUB_TOKEN`.
2. `helm package .` — builds the `.tgz` archive.
3. `helm push` — publishes to `oci://ghcr.io/noizu-labs`.

Consumers install via:
```bash
helm upgrade --install karpenter-nvme oci://ghcr.io/noizu-labs/karpenter-nvme \
  --version <tag> ...
```

## Test Values

`tests/test-values.yaml` provides the three required values (`nodeRole`, `clusterName`, `securityGroupTag`) plus overrides for CI validation. `examples/gnp-values.yaml` documents a production-like configuration.
