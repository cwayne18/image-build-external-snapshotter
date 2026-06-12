# image-build-external-controller

This repo builds hardened go binaries from [kubernetes-ci/external-controller](https://github.com/kubernetes-csi/external-snapshotter): csi-snapshotter and snapshot-controller. Binaries are then published in scratch based docker images.

## CVE go.mod overrides

This repo rebuilds upstream at a pinned tag; it does not patch source. When a CVE is found in
a Go dependency that upstream has **not** yet fixed in a release, we pin the dependency to a
patched version with `go mod edit -replace` in the `Dockerfile`. These pins live in the
managed block in the `Dockerfile` between the
`# === BEGIN CVE go.mod overrides ... ===` and `# === END CVE go.mod overrides ===` markers.

Maintaining that block is automated with a [GitHub Agentic Workflow](https://github.com/github/gh-aw)
(`gh-aw`):

- **Workflow source** — [`.github/workflows/cve-gomod-override.md`](.github/workflows/cve-gomod-override.md)
  is the human-edited agentic workflow. It runs weekly (and on demand), builds both images and
  scans them with Trivy in deterministic pre-agent steps, then has the Copilot agent add/raise
  overrides from the Trivy findings and remove overrides once upstream (at the pinned `TAG`)
  already requires an equal-or-newer version. The change is opened as a PR via the gh-aw
  `create-pull-request` safe output, which is restricted to editing the `Dockerfile`.
- **Compiled workflow** — [`.github/workflows/cve-gomod-override.lock.yml`](.github/workflows/cve-gomod-override.lock.yml)
  is generated from the `.md` source and is what GitHub Actions actually runs. **Do not edit it
  by hand.** After changing the frontmatter of the `.md` file, recompile with:

  ```sh
  gh extension install github/gh-aw   # once
  gh aw compile cve-gomod-override
  ```

  (Editing only the markdown body below the frontmatter does not require recompiling.)

The workflow needs a `COPILOT_GITHUB_TOKEN` repository secret to authenticate the Copilot
engine (the default `GITHUB_TOKEN` cannot make Copilot requests). All GitHub writes go through
the gh-aw safe-outputs system, so the agent job itself runs read-only.