# simple-example

This scenario uses a single shared evalset baseline and compares two agentspec
variants in the same run: the codemode agent against its no-codemode twin.

## Multi-Agentspec Variants

Use these agentspec ids:

- `example-evals` — codemode enabled.
- `example-evals-nocodemode` — codemode disabled (A/B control).

## Files

- `simple-example.evalset.json`: canonical evalset spec for this scenario.

## How To Run In GitHub Actions

The repository workflow `.github/workflows/datalayer-evals.yml` runs this scenario with:

- `evalset-spec-file: simple-example/simple-example.evalset.json`
- `agentspec-ids: example-evals,example-evals-nocodemode`

It runs both lanes with a matrix:

- `sdk`
- `sdk-proxy`

## Billable Account UID

The workflow supports an optional billing context. Use either method:

- Set workflow input `billable_account_uid` when manually dispatching.
- Leave input empty and set repository secret `DATALAYER_BILLABLE_ACCOUNT_UID`.

The workflow passes billing context with this fallback:

```yaml
billable-account-uid: ${{ inputs.billable_account_uid || secrets.DATALAYER_BILLABLE_ACCOUNT_UID }}
```

This applies billing context to eval operations and optional runtime creation.

## Access Uploaded Reports

The Datalayer action uploads report artifacts in the run automatically.

From the GitHub UI:

1. Open the workflow run.
2. Open the `Artifacts` section.
3. Download lane artifacts:
	- `datalayer-evals-reports-sdk`
	- `datalayer-evals-reports-sdk-proxy`
	- `datalayer-evals-final-reports-sdk`
	- `datalayer-evals-final-reports-sdk-proxy`

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
