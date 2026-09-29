# helm-charts

Automated Helm-chart mirror: a GitHub Actions pipeline that syncs
upstream Helm charts (currently [code-server](https://github.com/coder/code-server))
from their source repos into a private OCI registry (GHCR).

The repo contains no chart sources. Charts are pulled from upstream,
linted, packaged, and pushed to `oci://ghcr.io/lukasgierth/code-server`
only when the chart files changed upstream.

## How it works

The workflow [`.github/workflows/code-server.yaml`](.github/workflows/code-server.yaml)
runs daily at 00:00 UTC and on manual `workflow_dispatch`:

1. **Probe** – GitHub API: latest commit in `coder/code-server`
   that touched `ci/helm-chart`.
2. **Decide** – compare with `state/last-chart-sha`; skip if unchanged
   (manual dispatch always builds).
3. **Build** – check out upstream `main` (depth 1), `helm lint` + `helm package`.
4. **Push** – `helm push` to `oci://ghcr.io/lukasgierth/code-server`:
   - ghcr.io: `GITHUB_TOKEN` (username = repo owner)
   - other registries: `REGISTRY_USER` / `REGISTRY_PASSWORD` secrets
5. **Record** – commit the new SHA to `state/last-chart-sha`
   as `github-actions[bot]`.

## Repo layout

```
.github/workflows/code-server.yaml   # publish pipeline
state/last-chart-sha                 # last published upstream SHA (managed by the bot)
LICENSE                              # GPL-3.0
```

## Usage

```sh
helm pull oci://ghcr.io/lukasgierth/code-server
helm install code-server code-server-<version>.tgz   # adjust values as needed
```

## Maintaining

- **Force a re-publish** – Actions tab → `publish-code-server-chart`
  → *Run workflow*.
- **Change the schedule** – edit the `cron` entry in the workflow.
- **Add another chart** – copy the workflow; adjust `UPSTREAM_REPO`,
  `CHART_PATH`, `OCI_REPOSITORY`, and use a separate `state/` file per chart.
- **Upstream moved the chart** – the probe fails with
  "No commits for path …"; update `CHART_PATH` accordingly.

## License

[GPL-3.0](LICENSE)
