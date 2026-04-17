# Report Deployment Action

A GitHub Action that reports deployment events to [Ewake](https://www.ewake.ai), so deployments can be correlated with incidents and surfaced as context during investigations.

The action is **non-blocking by design**: it always exits 0, even on network errors, timeouts, or non-2xx responses. Reporting failures will never break your deploy.

## Quick start

```yaml
- name: Report deployment to Ewake
  uses: ewake-ai/report-deployment-action@v1
  with:
    api-key: ${{ secrets.EWAKE_API_KEY }}
    artifact-name: my-service
```

That's it. The action picks up `repository`, `commitSha`, and `timestamp` from the GitHub context and sends them along with a `source: github-actions` tag.

## Inputs

| Input           | Required | Default                  | Description                                                                 |
| --------------- | -------- | ------------------------ | --------------------------------------------------------------------------- |
| `api-key`       | Yes      | —                        | Your Ewake API key. Generate one in **Settings → API Keys**.                |
| `artifact-name` | Yes      | —                        | Name of the deployed artifact (e.g. service or package name).               |
| `api-url`       | No       | `https://api.ewake.ai`   | Ewake API base URL. Override only for self-hosted or testing environments.  |
| `version`       | No       | —                        | Version identifier (e.g. semver tag, image tag, or short SHA).              |
| `message`       | No       | —                        | Free-text description of the deployment.                                    |
| `labels`        | No       | `{}`                     | JSON object of key-value metadata, e.g. `'{"env":"production"}'`.           |

> **Tip:** Set `artifact-name` to match the service name as it appears in your observability stack (logs, metrics, traces). This lets Ewake correlate deployment events with telemetry from the same service automatically.

The following fields are populated automatically from the GitHub Actions context and cannot be overridden:

- `repository` — from `github.repository`
- `commitSha` — from `github.sha`
- `timestamp` — current time at execution
- `source` — always `github-actions`
- `url` — link back to the workflow run

## Example

A more complete example with optional metadata:

```yaml
- name: Report deployment to Ewake
  uses: ewake-ai/report-deployment-action@v1
  with:
    api-key: ${{ secrets.EWAKE_API_KEY }}
    artifact-name: checkout-service
    version: ${{ github.ref_name }}
    message: "Released by ${{ github.actor }}"
    labels: '{"env":"production","region":"eu-west-3"}'
```

## Failure behavior

The action will not fail your workflow. If the API is unreachable or returns an error, it logs a `::warning::` annotation in the workflow log and exits cleanly. To check whether reporting succeeded, look for warning annotations on your job summary.

If you want belt-and-suspenders isolation, you can also add `continue-on-error: true` to the step — but it's redundant.

## Where deployments appear

Successful events appear in the Ewake dashboard under **Events → Deployments** within a few seconds. Each event includes a `traceId` (logged by the action on success) for support and debugging.

## License

[MIT](./LICENSE)
