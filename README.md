[![Datalayer](https://assets.datalayer.tech/datalayer-25.svg)](https://datalayer.ai)

[![Become a Sponsor](https://img.shields.io/static/v1?label=Become%20a%20Sponsor&message=%E2%9D%A4&logo=GitHub&style=flat&color=1ABC9C)](https://github.com/sponsors/datalayer)

# ☰ 🧪 Context Engineering for Agentic Data Analysis

See the [Research section on Datalayer AI](https://datalayer.ai/research/context-optimized-agents).

This repository studies how **context engineering** — and in particular the
**Codemode** execution strategy — changes the accuracy, latency, and cost of
agentic data-analysis workflows. It pairs a written research framing with a
runnable, evaluation-driven setup that compares an agent **with codemode** to
the same agent **without codemode** on a shared evalset.

## TLDR

1. Create GitHub secrets and get values from Datalayer SaaS.
  - Open your repository secrets page: [Settings > Secrets and variables > Actions](https://github.com/datalayer-research/context-engineering-agentic-data-analysis/settings/secrets/actions).
  - Use **Repository secrets** for this workflow.
  - **Organization secrets** also work if they are shared with this repository.
  - **Environment secrets** are not used by this workflow as currently written (no job `environment` is configured).
  - Create required secret `DATALAYER_API_KEY`.
  - Optionally create `DATALAYER_BILLABLE_ACCOUNT_UID`.
  - Get both values from [Datalayer SaaS](https://datalayer.ai): sign in, go to your profile and create a API key, copy your API key, and (if needed) copy the billable account UID from your account/organization billing context.

2. Run the GitHub Action.
  - Open [Actions](https://github.com/datalayer-research/context-engineering-agentic-data-analysis/actions) in your repository.
  - Select workflow `.github/workflows/datalayer-evals.yml` at the top of the left sidebar.
  - Click **Run workflow** and keep defaults, or override:
    - `evalset_spec_file` (default: `simple-example/simple-example.evalset.json`)
    - `agentspec_ids`
    - `run_environments` (default: `sdk`; set `sdk,sdk-proxy` to run both lanes)
    - `run_limit` (default: `3`)

3. Get the report and view the evalset in Datalayer SaaS.
  - In the workflow run, download artifacts:
    - `datalayer-evals-reports-sdk` / `datalayer-evals-reports-sdk-proxy`
    - `datalayer-evals-final-reports-sdk` / `datalayer-evals-final-reports-sdk-proxy`
  - Open markdown/CSV report files from `artifacts/`.
  - In Datalayer SaaS, open [Evals](https://datalayer.ai/evals) and use the evalset ID shown in the workflow summary/report to inspect experiments and runs.



---

## Research Framing

### The Accuracy–Latency–Cost Trade-off

Three opposing forces define the optimization space:

- **Accuracy** tends to degrade as the context given to the agent shrinks.
- **Latency** improves as the prompt gets smaller.
- **Per-task cost** drops because pricing is token-based.

The Pareto-optimal region — where the marginal accuracy loss is small relative
to the efficiency gains — is the deployment sweet spot.

**Accuracy curve (descending).** Task accuracy holds on a plateau at low
context-reduction levels, then degrades through a transition zone before
reaching a floor. The plateau width varies by task complexity: simple lookups
tolerate aggressive reduction, while multi-step reasoning needs richer context.
Finding each task type's inflection point is the core optimization target.

**Latency and cost (improving).** Latency drops almost linearly with context
reduction, because fewer prompt tokens lower both time-to-first-token and
generation time. API cost follows the same curve. The most dramatic gains occur
in the Codemode tier, where a single code cell can replace dozens of
natural-language reasoning turns.

### Three Mechanistic Tiers

Reduction techniques are grouped by mechanism. Each tier maps to a clear
trade-off control.

1. **Prompt-engineering reduction**
   - System-prompt compression and deduplication
   - Few-shot example pruning by task similarity
   - Schema normalization (strip verbose descriptions)
   - Conversation-history windowing (last N turns)
   - _Expected impact: ~15–25% fewer tokens, usually < 2% accuracy impact._

2. **Retrieval and summarization reduction**
   - Dynamic RAG with semantic relevance scoring
   - Tool-output summarization (compress prior results)
   - MCP resource pruning (load schemas on demand)
   - Episodic memory with decay (forget stale context)
   - _Expected impact: ~40–60% fewer tokens, ~3–7% accuracy impact._

3. **Codemode execution reduction**
   - Code generation replaces natural-language reasoning chains
   - Kernel state as implicit context (variables, DataFrames)
   - Single-shot tool orchestration (batch MCP calls)
   - Intermediate-result caching in the execution environment
   - _Expected impact: ~70–92% fewer tokens, ~5–15% accuracy impact depending on task class._

### Evaluation-Driven Development (EDD)

Non-deterministic AI agents demand continuous, multi-dimensional evaluation —
not just pass/fail tests. EDD treats evaluation as a first-class development
practice, with metrics spanning accuracy, efficiency, reliability, and cost:

- **Task completion rate** — binary and partial-credit scoring of whether the
  agent reaches its analytical objective.
- **Answer fidelity** — semantic similarity against ground truth, typically via
  a calibrated LLM-as-judge plus a human-agreement baseline.
- **Token efficiency** — quality-adjusted output per token spent; the primary
  metric for context optimization.
- **Latency profile** — p50, p95, and p99 response times across reduction tiers.
- **Hallucination rate** — frequency of unsupported claims, measured against
  source grounding.
- **Cost per task** — total API cost per successful task, including retries and
  tool-invocation overhead.

### Evaluation Methods

- **Calibrated LLM-as-judge** — model-based scoring against explicit rubrics,
  calibrated to human agreement.
- **Execution tracing and observability** — end-to-end tracing of decisions,
  tool calls, context assembly, and MCP access.
- **Synthetic benchmark generation** — parameterized task generation for broad,
  reproducible coverage.
- **Regression testing for agents** — automatic detection of quality regressions
  across code, prompt, and model changes.
- **Paired A/B comparison** — identical tasks across adjacent reduction tiers to
  quantify the marginal quality impact. _This is exactly what the first
  experiment below does for the codemode-vs-no-codemode pair._

### Context Engineering as a Discipline

Context engineering is the discipline of curating exactly the right information
for an agent's limited window. It combines prompt design, memory management,
retrieval strategy, and tool-schema optimization into one coherent system
design practice.

- **MCP-aware context assembly** — use the Model Context Protocol to load tool
  schemas, resource descriptions, and capability manifests on demand, so the
  baseline context footprint stays small.
- **Memory architecture** — design working memory (current task state), episodic
  memory (prior interactions with decay), and semantic memory (domain knowledge
  via retrieval) with distinct retention and relevance policies.
- **Codemode as a context strategy** — Codemode externalizes state to a Jupyter
  kernel. Variables and DataFrames hold intermediate state outside the prompt,
  effectively extending usable context through execution rather than token
  accumulation.

> The full written paper draft lives in [`paper/PAPER.md`](paper/PAPER.md).

---

## First Experiment: Codemode vs No-Codemode

The first runnable case is a paired A/B comparison that runs **one shared
evalset** against **two agentspec variants** in the same run:

| Agentspec | Codemode | Role |
| :-- | :-- | :-- |
| `example-evals` | enabled | Codemode agent (state externalized to a kernel) |
| `example-evals-nocodemode` | disabled | Same agent, natural-language reasoning only |

Because both variants run against the identical evalset, the resulting report
isolates the effect of codemode on the pass rate. The scenario configs live in
[`simple-example/`](simple-example/):

- `simple-example.evalset.json` — canonical baseline evalset for the scenario.

Each spec declares the comparison metadata:

```json
"metadata": {
  "agentspec_ids": ["example-evals", "example-evals-nocodemode"],
  "comparison_mode": "same-evalset-multiple-agentspecs"
}
```

---

## Automated Evals in GitHub Actions

The workflow at [`.github/workflows/datalayer-evals.yml`](.github/workflows/datalayer-evals.yml)
runs the shared evalset baseline across both agentspec variants, with a
configurable run-environment matrix (default lane: `sdk`; optional additional
lane: `sdk-proxy`), and uploads markdown + CSV reports as artifacts.

### Required and optional secrets

Configure these in **Settings → Secrets and variables → Actions**:

- Primary scope: **Repository secrets**.
- Also supported: **Organization secrets** (if configured to be available to this repository).
- Not used in the current workflow: **Environment secrets** (unless you update the job to use an `environment`).

| Secret | Required | Purpose |
| :-- | :-- | :-- |
| `DATALAYER_API_KEY` | ✅ Required | Authenticates the Datalayer action. Passed as `api-key`. |
| `DATALAYER_BILLABLE_ACCOUNT_UID` | Optional | Billable account context for cloud runtime creation and eval operations. Passed as `billable-account-uid`. |

> Never hard-code the API key or billable account UID in the workflow file or in
> spec files — always reference them through `secrets.*`.

### Setup

1. Add the repository secret `DATALAYER_API_KEY` (and optionally
   `DATALAYER_BILLABLE_ACCOUNT_UID`).
2. Review or customize the evalset spec under `simple-example/`.
3. Optionally override `agentspec_ids` when dispatching the workflow.

### Run

Trigger the **datalayer-evals** workflow from the Actions tab
(`workflow_dispatch`). All inputs have sensible defaults:

| Input | Default | Notes |
| :-- | :-- | :-- |
| `evalset_spec_file` | `simple-example/simple-example.evalset.json` | Shared baseline evalset. |
| `agentspec_ids` | `example-evals,example-evals-nocodemode` | Comma-separated variants to compare. |
| `run_environments` | `sdk` | Lanes to execute (matrix). Use `sdk,sdk-proxy` to run both lanes. |
| `run_limit` | `3` | Runs fetched per experiment. |
| `ai_agents_url` | _(empty)_ | Optional API URL override. |

### Outputs

The workflow publishes per-lane artifacts:

- `datalayer-evals-reports-sdk` / `datalayer-evals-reports-sdk-proxy`
  (uploaded by the Datalayer action)
- `datalayer-evals-final-reports-sdk` / `datalayer-evals-final-reports-sdk-proxy`
  (final workflow upload step)

Typical files inside those artifacts:

- `artifacts/simple-example-sdk-report.md`
- `artifacts/simple-example-sdk-report.csv`
- `artifacts/simple-example-sdk-report.md.log` (full structured report JSON)
- `artifacts/simple-example-sdk-proxy-report.md`
- `artifacts/simple-example-sdk-proxy-report.csv`
- `artifacts/simple-example-sdk-proxy-report.md.log`
- timestamped markdown/csv outputs

Download from the GitHub UI (workflow run → **Artifacts**) or via the CLI:

```bash
gh run download <run-id> --name datalayer-evals-final-reports-sdk --dir ./artifacts
gh run download <run-id> --name datalayer-evals-final-reports-sdk-proxy --dir ./artifacts
```

The workflow summary links the generated markdown/csv paths and the agentspec
ids used for the run.

### Billable account UID usage

Billing context is sourced from the repository/organization secret
`DATALAYER_BILLABLE_ACCOUNT_UID`.

The workflow wiring is:

```yaml
billable-account-uid: ${{ secrets.DATALAYER_BILLABLE_ACCOUNT_UID }}
```

This value is forwarded to the Datalayer action and applied to eval operations
and optional cloud runtime creation.

---

## How to Interpret the Report

Each report compares the two agentspecs on the same cases. Read it in this order:

1. **Header** — generated timestamp, number of experiments/agentspecs/cases, and
   a deep link to open the evalset in Datalayer.
2. **Agentspec Coverage** — confirms both `example-evals` and
   `example-evals-nocodemode` were exercised, with run counts.
3. **Experiment Overview** — per-variant **Latest** pass rate, **Baseline**, and
   **Drift** (latest − baseline). This is the headline accuracy comparison.
4. **Comparison Combinations** — rankings by latest pass rate, by drift, and by
   stability, plus **pairwise deltas**. The codemode-vs-no-codemode delta is the
   key research number: a positive delta means codemode improved the pass rate.
5. **Per-Experiment Details** — run timelines, sparklines, and any failure causes.
6. **Appendix: Run Details** — every fetched run with its prompt, agent output,
   summary, and report (the same content the in-app run-details dialog shows).

### Reading the deltas

Deltas are rendered as percentage-point changes with an emoji indicator:

- 🟢 positive (e.g. `🟢 +8.0 pts`) — the left/latest variant did better.
- 🔴 negative (e.g. `🔴 -5.0 pts`) — the left/latest variant did worse.
- ⚪ flat (`⚪ +0.0 pts`) — no measurable change.

For the codemode comparison, look at the **pairwise latest-pass delta** between
`example-evals` and `example-evals-nocodemode`: it quantifies the accuracy cost
or benefit of codemode for this evalset, which you then weigh against the
latency and token-cost gains described in the research framing above.

The CSV export carries the same data in tabular form for downstream analysis
(spreadsheets, notebooks, or further EDD regression tracking).
