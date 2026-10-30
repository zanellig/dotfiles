# Model and subagent routing

#### Picking the model for workflows and subagents

Cost reflects OpenAI's Plus and Anthropic's Pro plan usage quotas and the risk of failures or token blow-up on complex tasks. Intelligence is unsupervised problem difficulty. Taste covers UI/UX, code quality, API design, and copy.

| model                                                | cost | intelligence | taste |
| ---------------------------------------------------- | ---- | ------------ | ----- |
| sol                                                  | 8    | 8.5          | 7     |
| terra _(provisional)_                                 | TBD  | TBD          | TBD   |
| astra                                                | 1    | 9.5          | 7     |
| luna                                                 | 7    | 5            | 4     |
| fable _(currently not available in our Claude plan)_ | 3    | 10           | 10    |
| opus                                                 | 8    | 9.5          | 9     |
| sonnet                                               | 9    | 6.5          | 8     |

<sub>Higher is better.</sub>

Terra has provisional workflow routes below. Its numeric ratings await local
comparisons; API prices alone do not measure subscription cost or taste.

##### Model-specific effort defaults

| Model    | Recommended effort                                                                                                                                                                                                     |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sol**  | Use `medium` for routine and bounded non-trivial execution, `high` when requirements are partly ambiguous, and `xhigh` for ambiguous or cross-module tasks and code review. Reserve `max` for high-blast-radius decisions; its marginal gain rarely repays the latency. Do not use `ultra` by default. |
| **Terra** | Use `medium` for defined screen tasks and drafting from supplied material; `high` for bounded repository tracing; trial `xhigh` for harder tracing or scoped implementation with strong verification. Use `max` only after a workload-specific comparison shows a benefit. |
| **Luna** | Use `high` by default. Escalate to `xhigh` or `max` for inexpensive, long-running workers with strong automated verification. Avoid `low` or `medium`, moreso when failure is costly.                                  |
| **Opus** | Use `medium` for bounded implementation with clear verification, `high` for interactive sessions, planning, specs, and ambiguous or cross-module work, and `xhigh` for contract decisions, ADRs, and the hardest long-horizon, orchestration, or taste-critical work. Avoid `max` by default: observed gains over `xhigh` are often negligible despite substantially greater compute. |
| **Sonnet** | Use `high` or above: `high` by default, `xhigh` when quality matters. |
| **Astra** | Use `high`. |

##### Use case notes

| Model  | Default use                                                                                                                          | Avoid or escalate when                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Luna   | Cheap, fast scouts; repository mapping; log/test analysis; summaries; mechanical edits with strong verification                      | The task is ambiguous, cross-module, weakly tested, or expensive to redo                 |
| Terra  | Provisional route for bounded repository tracing, screen inspection and computer use with a defined result, source-grounded drafting, and scoped fixes with strong verification | Escalate to Sol for architectural decisions, ambiguous diagnosis, consequential review, orchestration, or weak verification |
| Sol    | Primary execution, architecture, difficult diagnosis, synthesis, orchestration, high-risk review, and underspecified multi-file work | Use Luna for mechanical scouts, or trial Terra for its bounded routes, when Sol's added reliability does not repay its cost |
| Fable  | Major UI overhauls, end-to-end product builds, taste-critical direction, hardest Claude orchestration                                | Bounded work; prefer Opus or Sonnet                                                      |
| Opus   | Default executor; small-to-medium UI and taste/code/API/copy review                                                                  | Escalate major direction or long-horizon work to Fable                                   |
| Sonnet | Scoped execution, UI variants, debugging/tests, and cheap parallel workers                                                           | Ambiguous, taste-critical, or long-horizon work                                          |
| Astra  | Hardest unsupervised reasoning when the user asks for it                                                                             | Use Sol for default work                                                                 |

Computer use: trial Terra `medium` for defined workflows with a checkable saved
result. Use Sol `medium+` when the workflow requires ambiguous planning or
diagnosis. Use Claude when taste dominates GUI execution.

Repository exploration: keep Luna for mechanical lookup and inventories. Trial
Terra `high` for a bounded question requiring code tracing. Require source
locations, evidence for each conclusion, and explicit search limits for absence
claims. Sol owns architectural synthesis and design decisions.

Security work: Sol `xhigh` for review and audits; Opus `xhigh` for security decisions and adversarial work with PoCs. Re-route when a task turns into security work mid-session.

#### Choose reasoning effort

> Reasoning effort roughly maps to "thoroughness", or how much does the model is willing to work before returning an answer or giving up.

- `low`: deterministic lookup or tiny mechanical work with immediate checks.
- `medium`: bounded exploration or a clear, bounded change.
- `high`: default for non-trivial implementation, diagnosis, and review.
- `xhigh`: ambiguous or cross-module work, subtle edge cases, or weak tests.
- `max`: hardest long-horizon or high-blast-radius work where a failed attempt or
  bad decision costs more than the extra run. Do not use it with larger models by default.

Model and effort are separate levers. Prefer the cheapest configuration that
clears the task's risk threshold. Escalate family or effort when requirements
are ambiguous, verification is weak (including a skipped review loop or an
unreviewed deploy), the change crosses authoritative domain boundaries, or
rollback would be costly.

Run at most one Opus `xhigh` or Astra session at a time. Never launch the same prompt in parallel.

#### Cross-provider collaboration

These rules take precedence when delegating across providers:

- OpenAI to Anthropic: use Sonnet by default. Use Opus when ambiguity,
  architectural tradeoffs, or costly errors require deeper judgment.
- Anthropic to OpenAI: use Sol by default. Trial Terra for its bounded routes
  above; keep Sol for architectural synthesis, consequential review, and
  orchestration. Adjust reasoning effort to the task.

Use the latest stable version available to the runtime and account, including
when the CLI examples below name an older version. Resolve at delegation time
and keep that version for the session. Explicit user model requests take precedence.
Use Claude's `sonnet` and `opus` aliases; resolve Sol with
`python3 ~/.agents/subagents/codex-sol.py resolve` and use its model and binary.

#### CLI

Run from the target repository. Use one-shot mode for delegated work.

Example usage:

|        | Interactive                                                                 | One-shot                                                                       | Resume                                                                                   |
| ------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| Claude | `claude --dangerously-skip-permissions --model opus --effort high "{task}"` | `claude --dangerously-skip-permissions -p --model opus --effort high "{task}"` | `claude --dangerously-skip-permissions -c` or `claude --dangerously-skip-permissions -r` |
| Codex  | `codex --yolo -m gpt-6.1-sol -c 'model_reasoning_effort="high"' "{task}"`   | `codex --yolo exec -m gpt-6.1-sol -c 'model_reasoning_effort="high"' "{task}"` | `codex --yolo resume --last`                                                             |

> For `codex` subagents you should set the `CODEX_HOME` environment variable to `~/.codex_p` if not already set by the environment, as that is the directory that holds our active paid subscription.

Replace model and effort from the routing table. Prompts should concisely state scope, file ownership, verification, and expected output.
