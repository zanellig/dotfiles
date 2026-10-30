# Model and subagent routing

## Choose a configuration

Route by model, resolved version, reasoning effort, and agent harness together.
Effort labels are model-specific controls. Equal labels do not imply equal
capability, token use, cost, or runtime across models.

1. Identify the task, failure cost, verification strength, and latency budget.
2. Apply availability, explicit user requests, and the provider routes below.
3. Compare eligible model-and-effort configurations. Prefer the least costly
   configuration that meets the task's reliability needs. Compare raising effort
   with changing models; neither is an automatic next step.
4. Escalate when expected reductions in failure or rework justify the added cost.
   Ambiguity, weak verification, cross-domain changes, and costly rollback raise
   the required reliability. Extra reasoning does not replace verification.

Consult [model-benchmarks.md](model-benchmarks.md) when comparing effort levels,
cost, or competing configurations. Match the benchmark component to the work:
implementation, terminal use, repository Q&A, or general reasoning. A composite
index score is not a task success probability or a measure of taste.

## Cost and local preferences

Keep these measures separate:

- **Subscription quota use:** local observations for the account and resolved
  configuration. API dollars and token counts do not establish quota charges.
- **API cost per task:** benchmark workload expenditure, including input, cache,
  reasoning, and answer tokens. Per-token prices alone miss differences in usage.
- **Runtime:** measured task duration. Output speed alone misses the amount of
  output and reasoning. A low-dollar worker can still take a long time.
- **Rework:** expected retries, supervision, and failure consequences. Benchmark
  expenditure does not include these local costs.

The following are inherited, subjective local preferences, not measured
configuration costs or Artificial Analysis scores. Higher is better. The legacy budget
preference retains the old mixed cost rating, which combined quota and failure-risk
impressions, but has no recorded version, effort, or measurement method; use it only as a provisional tie-breaker within acceptable
configurations. Taste covers UI/UX, code quality, API design, and copy.

| Model | Legacy budget preference | Local taste preference |
| --- | --- | --- |
| Sol | 8 | 7 |
| Terra | Uncalibrated | Uncalibrated |
| Astra | 1 | 7 |
| Luna | 7 | 4 |
| Fable, unavailable in our Claude plan | 3 | 10 |
| Opus | 8 | 9 |
| Sonnet | 9 | 8 |

## Model-specific effort defaults

These are starting policies, subject to task evidence and the cost comparison
above. Verify that the resolved runtime supports the requested effort.

| Model | Starting effort and escalation |
| --- | --- |
| Sol | `medium` for bounded execution; `high` for harder or partly ambiguous work; `xhigh` for consequential review and ambiguous or cross-module tasks. Consider `max` for difficult work when the measured gain justifies its cost, including before switching to a more expensive model. Use `ultra` only with workload-specific evidence. |
| Terra | `medium` for defined screen tasks and source-grounded drafting; `high` for bounded repository tracing. Trial `xhigh` for harder tracing or scoped implementation with strong verification. Compare it with Sol at lower effort. Use `max` after a workload-specific comparison. |
| Luna | `high` for mechanical work. Consider `xhigh` or `max` with strong verification and an acceptable runtime budget. Low API cost does not imply low latency or sufficient capability. Use `low` or `medium` only for deterministic lookup or tiny, immediately checked work. |
| Opus | `medium` for bounded implementation with clear verification; `high` for interactive work, planning, specs, and ambiguous execution; `xhigh` for contract decisions, ADRs, and the hardest long-horizon or taste-critical work. Reserve `max` for a demonstrated task benefit; the current aggregate gain over `xhigh` is small relative to its cost. |
| Sonnet | `high` for scoped execution; `xhigh` for harder work where its substantial capability gain justifies the added cost. Compare with Opus and eligible OpenAI configurations before assuming Sonnet is the inexpensive option. |
| Astra | `high`, only when explicitly requested. Compare further escalation using task-specific evidence. |
| Fable | Unavailable in our current plan. Preserve local taste preferences for future evaluation; revalidate version, effort, and reasoning capability before enabling a route. |

## Use case notes

| Model | Default use | Avoid or escalate when |
| --- | --- | --- |
| Luna | Mechanical lookup, inventories, log/test analysis, summaries, and mechanical edits with strong verification | Ambiguity, cross-module reasoning, weak tests, costly rework, or a tight latency budget |
| Terra | Provisional bounded repository tracing, defined computer workflows, source-grounded drafting, and scoped verified fixes | Architectural decisions, ambiguous diagnosis, consequential review, orchestration, or weak verification |
| Sol | Primary OpenAI execution, architecture, difficult diagnosis, synthesis, orchestration, and consequential review | A less costly eligible configuration meets the reliability requirement |
| Fable | Unavailable; retain as a candidate for major UI direction and taste evaluation | General reasoning or orchestration claims need current evidence |
| Opus | Default Claude executor; UI work and taste/code/API/copy review | Compare higher effort with Sonnet and other eligible configurations before escalation |
| Sonnet | Scoped execution, UI variants, debugging, and tests | Ambiguity, taste-critical work, or long-horizon work requires a stronger configuration |
| Astra | Hardest unsupervised reasoning when explicitly requested | Use Sol for default work |

Computer use: trial Terra `medium` for defined workflows with a checkable saved
result. Use Sol `medium` or higher when planning or diagnosis is ambiguous. Use
Claude when taste dominates GUI execution.

Repository exploration: use Luna for mechanical lookup and inventories. Trial
Terra `high` for a bounded tracing question, or `xhigh` for harder verified
tracing. Require source locations, evidence for conclusions, and explicit search
limits for absence claims. Sol owns architectural synthesis and design decisions.

Security work: Sol `xhigh` for review and audits; Opus `xhigh` for security
decisions and adversarial work with PoCs. Re-route when work becomes security work.

Run at most one Opus `xhigh` or `max`, or Astra session, at a time. Never launch
the same prompt in parallel. Apply configuration costs when sizing parallel work.

## Cross-provider collaboration

These rules constrain eligible models before comparing configurations:

- OpenAI to Anthropic: use Sonnet by default. Use Opus when ambiguity,
  architectural tradeoffs, or costly errors require deeper judgment. Choose
  effort using the model-specific guidance and compare actual configuration cost.
- Anthropic to OpenAI: use Sol by default. Trial Terra for its bounded routes;
  retain Sol for architectural synthesis, consequential review, and orchestration.

Use the latest stable version available to the runtime and account, including
when CLI examples name an older version. Resolve at delegation time and keep that
version for the session. Explicit user model requests take precedence. Use
Claude's `sonnet` and `opus` aliases; resolve Sol with
`python3 ~/.agents/subagents/codex-sol.py resolve` and use its model and binary.
Revalidate benchmark evidence when the resolved version changes; a previous
version's cost curve does not establish the new version's defaults.

## CLI

Run from the target repository. Use one-shot mode for delegated work.

Example usage:

|        | Interactive                                                                 | One-shot                                                                       | Resume                                                                                   |
| ------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Claude | `claude --dangerously-skip-permissions --model opus --effort high "{task}"` | `claude --dangerously-skip-permissions -p --model opus --effort high "{task}"` | `claude --dangerously-skip-permissions -c` or `claude --dangerously-skip-permissions -r` |
| Codex  | `codex --yolo -m gpt-6.1-sol -c 'model_reasoning_effort="high"' "{task}"`   | `codex --yolo exec -m gpt-6.1-sol -c 'model_reasoning_effort="high"' "{task}"` | `codex --yolo resume --last`                                                             |

> For `codex` subagents you should set the `CODEX_HOME` environment variable to `~/.codex_p` if not already set by the environment, as that is the directory that holds our active paid subscription.

Choose model and effort together using this guide. Prompts should concisely state scope, file ownership, verification, and expected output.

## Version control

The two Markdown guides are tracked in `zanellig/dotfiles` under
`.agents/subagents/`. The live files in `~/.agents/subagents/` are symlinks to a
persistent checkout. Edit through either path, then commit in that checkout.
Use `readlink -f ~/.agents/subagents/README.md` to locate it. Keep the checkout
while these links are active; repoint both links before removing or relocating it.

Git history preserves the original guides in a baseline commit. Policy belongs
in this file; dated measurements, sources, limitations, and historical snapshots
belong in `model-benchmarks.md`. Record model version, effort, harness, benchmark
version, and retrieval date with each new measurement. Keep local quota and taste
observations distinct from benchmark results.
