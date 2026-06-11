# image-build-external-controller

This repo builds hardened go binaries from [kubernetes-ci/external-controller](https://github.com/kubernetes-csi/external-snapshotter): csi-snapshotter and snapshot-controller. Binaries are then published in scratch based docker images.

## CVE go.mod overrides

This repo rebuilds upstream at a pinned tag; it does not patch source. When a CVE is found in
a Go dependency that upstream has **not** yet fixed in a release, we pin the dependency to a
patched version with `go mod edit -replace` in the `Dockerfile`. These pins live in the
managed block in the `Dockerfile` between the
`# === BEGIN CVE go.mod overrides ... ===` and `# === END CVE go.mod overrides ===` markers.

Maintaining that block is automated:

- **Agent definition** — [`.github/agents/cve-gomod-override.md`](.github/agents/cve-gomod-override.md)
  is a [GitHub Copilot CLI custom agent](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/invoke-custom-agents)
  / playbook. It adds or raises overrides from Trivy findings and removes overrides once
  upstream (at the pinned `TAG`) already requires an equal-or-newer version. You can run it
  locally with `copilot --agent cve-gomod-override -p "..."`.
- **Scheduled workflow** — [`.github/workflows/cve-gomod-override.yml`](.github/workflows/cve-gomod-override.yml)
  runs weekly (and on demand). It builds both images, scans them with Trivy, invokes the agent
  to update the managed block, and opens/updates a PR on the `cve-gomod-override` branch.

The workflow needs a `COPILOT_CLI_TOKEN` repository secret: a fine-grained personal access
token with the **Copilot Requests** permission (the default `GITHUB_TOKEN` cannot authenticate
Copilot requests).