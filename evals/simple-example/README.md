# simple-example

This scenario uses a single shared evalset baseline and compares two agentspec
variants in the same run: the codemode agent against its no-codemode twin.

Canonical Evals docs: [https://datalayer.ai/docs/evals](https://datalayer.ai/docs/evals)

## Multi-Agentspec Variants

Use these agentspec ids:

- `example-evals` — codemode enabled.
- `example-evals-nocodemode` — codemode disabled (A/B control).

## Files

- `simple-example.evalset.json`: canonical evalset spec for this scenario.

## How To Run In GitHub Actions

The repository workflow `.github/workflows/simple-example.yaml` runs this scenario with:

- `evalset-spec-file: evals/simple-example/simple-example.evalset.json`
- `agentspec-ids: example-evals,example-evals-nocodemode`

It runs a run-environment matrix from the `run_environments` workflow input.
Default is:

- `sdk`

Optional additional lane:

- `sdk-proxy`

## Billable Account UID

The workflow supports an optional billing context via repository secret:

- `DATALAYER_BILLABLE_ACCOUNT_UID`

In `.github/workflows/simple-example.yaml`, billing context is passed directly from the secret:

```yaml
billable-account-uid: ${{ secrets.DATALAYER_BILLABLE_ACCOUNT_UID }}
```

This applies billing context to eval operations and optional runtime creation.

## Access Uploaded Reports

The Datalayer action uploads report artifacts in the run automatically.

From the GitHub UI:

1. Open the workflow run.
2. Open the `Artifacts` section.
3. Download lane artifacts:
	- `datalayer-evals-reports-sdk`
	- `datalayer-evals-final-reports-sdk`
	- `datalayer-evals-reports-sdk-proxy` (when `run_environments` includes `sdk-proxy`)
	- `datalayer-evals-final-reports-sdk-proxy` (when `run_environments` includes `sdk-proxy`)

Included report files:

- markdown report (`*.md`)
- csv report (`*.csv`)
- report log (`*.log`)
- optional timestamped markdown/csv outputs

From CLI (optional):

```bash
gh run download <run-id> --name datalayer-evals-final-reports-sdk --dir ./artifacts
gh run download <run-id> --name datalayer-evals-final-reports-sdk-proxy --dir ./artifacts
```
