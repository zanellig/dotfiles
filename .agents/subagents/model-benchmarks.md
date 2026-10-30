# Coding-agent benchmark interpretation

Research snapshot: 2026-07-12. These benchmarks are useful routing evidence, not a universal ranking. Compare results within the same benchmark version, harness, and reasoning setting.

Historical reference only. Model versions, prices, and routing suggestions below
describe that snapshot. Use [README.md](README.md) for the current routing policy
and resolve current versions through the runtime before delegating.

## DeepSWE

**Why it matters.** DeepSWE targets the work most likely to consume an expensive subagent: original, underspecified, long-horizon repository changes. Its 113 tasks span 91 active repositories in TypeScript, Go, Python, JavaScript, and Rust; reference solutions average 668 added lines across seven files. Tasks are written from scratch to reduce training contamination, and behavioral verifiers accept different correct implementations rather than requiring a gold patch's internal structure. This makes it unusually relevant to autonomous implementation across software repositories. [Overview and task distribution](https://deepswe.datacurve.ai/blog/deepswe#overview)

**What it measures.** The score is pass@1 on executable, end-to-end behavioral verification plus regression tests. Every task has a short behavior-focused prompt, verifier, and reference solution; tasks and verifiers receive LLM-assisted and independent human review. Models get three rollouts and are run through the same model-agnostic `mini-swe-agent` bash harness and shared prompt. The leaderboard also reports average API cost, output tokens, and agent steps. DeepSWE reports a verifier audit over sampled rollouts, but that audit itself uses an LLM analyzer. [Methodology](https://deepswe.datacurve.ai/blog/deepswe#methodology) [Current leaderboard](https://deepswe.datacurve.ai/)

**Current 5.6 signal.** At each model's best listed effort, Sol max scores 73% (about $8.39/task), Terra max 70% ($4.95), and Luna max 67% ($3.03). The overlapping confidence intervals mean these quality differences are suggestive, while the cost differences are substantial. [Current leaderboard](https://deepswe.datacurve.ai/)

**Limits.** The common bash-only harness improves model-to-model control but does not reproduce Codex's native tools or prompt. The corpus excludes proprietary and long-tail repositories, covers only five languages, under-represents localization and refactoring, and concentrates most tasks in TypeScript, Go, and Python. It measures successful patches, not product taste, UX judgment, communication quality, or orchestration. [Limitations](https://deepswe.datacurve.ai/blog/deepswe#limitations-and-future-work)

**Use for routing.** Weight DeepSWE most heavily for large backend/domain changes, cross-file implementation, difficult integrations, and work where the agent must discover the solution shape. Prefer Sol when the failure cost dominates token cost; Terra or Luna are credible execution defaults when the task is bounded and verification is strong. Do not use DeepSWE alone to choose a planner, reviewer, frontend designer, or UX agent.

## CursorBench

**Why it matters.** CursorBench is built from real internal Cursor sessions rather than public issue/patch pairs. Its deliberately short, ambiguous prompts and multi-file tasks test how agents behave when developers specify outcomes rather than implementation details. Cursor states that its offline rankings track controlled online product metrics more closely than public coding benchmarks, making this the best of the three sources for day-to-day agent usability evidence. [Benchmark rationale](https://cursor.com/blog/cursorbench)

**What it measures.** CursorBench 3.2 evaluates edit, refactor, bug-fix, codebase-understanding, bug-finding, planning, code-review, instruction-following, and advanced-tool-use problems. Cursor's broader internal suite covers correctness, code quality, efficiency, and interaction behavior, although the public leaderboard is principally a score over the published task mix. It reports score plus average API cost, tokens, and steps; cost applies published input, cache-read, cache-write, and output prices with the benchmark's task weights. [CursorBench 3.2](https://cursor.com/cursorbench) [Evaluation design](https://cursor.com/blog/cursorbench)

**Current 5.6 signal.** Sol improves consistently with effort and leads the family (low 52.6%, medium 60.0%, high 63.5%, xhigh 64.5%, max 67.2%). Terra spans 46.9%–64.9% and Luna 37.6%–61.1%; both are markedly cheaper, and Luna high (56.8%, $0.82/task) is a particularly strong efficiency point. Small adjacent differences should not be over-read: Cursor explicitly warns that results vary and may not be statistically meaningful. [CursorBench 3.2 leaderboard and cost method](https://cursor.com/cursorbench)

**Limits.** The tasks and agentic graders are private, so independent reproduction and inspection are limited. Tasks come from Cursor's own engineering usage, run inside Cursor's product context, and still finish within one session; they do not yet represent multi-session autonomous projects or external-service-heavy work. Benchmark versions change their task distributions, so scores from different versions are not comparable. The benchmark also identified one model's training contamination, showing that private datasets can still leak. It provides no direct measure of visual taste. [Evaluation design and limitations](https://cursor.com/blog/cursorbench) [Version and contamination notes](https://cursor.com/cursorbench)

**Use for routing.** Weight CursorBench for ordinary repository work: ambiguous requests, bug finding, refactors, reviews, planning, and tool use. Medium/high are sensible defaults; use xhigh/max when ambiguity, blast radius, or weak verification makes a miss costly. Use low only for narrow, easily checked chores. Treat score-per-dollar as a throughput guide, not proof that the cheapest configuration is safest for a particular change.

## Artificial Analysis

**Why it matters.** Artificial Analysis separates model capability from coding-agent configuration and reports outcome, token usage, pay-per-token cost, and active runtime together. Its Coding Agent Index deliberately combines three distinct workflows, making it the broadest routing signal here and useful for selecting both a model and an effort level under budget or latency constraints. [Coding Agent Index](https://artificialanalysis.ai/agents/coding-agents) [Methodology](https://artificialanalysis.ai/methodology/coding-agents-benchmarking)

**What it measures.** Index v1.1 is the simple average of three component scores, each run three times: DeepSWE (113 long-horizon patch tasks, binary verifier), Terminal-Bench v2 (84 terminal tasks after five compatibility exclusions, binary tests), and SWE-Atlas-QnA (124 repository Q&A tasks, rubric partial credit). The 321-task suite reports average pass@1 plus pooled per-attempt token, API-cost, and wall-time metrics. Rows are agent variants—not model-only measurements—so harness, settings, and model jointly determine the result. [Index construction and scoring](https://artificialanalysis.ai/methodology/coding-agents-benchmarking)

The separate Intelligence Index is broader still: v4.1 weights agents 34%, coding 24%, scientific reasoning 24%, and general capabilities 18% over nine evaluations. It is useful evidence for planning and general reasoning but is less directly predictive of patch execution than the Coding Agent Index. [Intelligence Index methodology](https://artificialanalysis.ai/methodology/intelligence-benchmarking)

**Current 5.6 signal.** Artificial Analysis reports Sol max in Codex leading its Coding Agent Index at 80; Terra max and Luna max score 77 and 75 with roughly 60% and 80% lower per-task cost than Sol. In the Intelligence Index, Sol max is near the top at about one-third of Fable max's cost; across effort levels Sol and Luna occupy the reported intelligence/cost Pareto frontier, while every Terra point is dominated by some Sol or Luna setting. This is a strong default-routing signal, not a ban on Terra: coding-harness results and local workloads can differ from the general Intelligence Index. [GPT-5.6 evaluation](https://artificialanalysis.ai/articles/gpt-5-6-has-landed)

**Limits.** A simple average hides different strengths: repository Q&A, terminal work, and patch implementation are not interchangeable, and binary and partial-credit scores are combined. Efficiency averages exclude missing telemetry rather than imputing zero. Costs reflect API pricing and observed cache behavior, not subscriptions, supervision, infrastructure, or retry costs. Agent variants may use different harnesses and defaults, so the index is not a controlled model-only comparison. Version 1.1 also changed a component from v1.0. Neither the Coding nor Intelligence Index measures frontend taste directly. [Coding Agent Index interpretation](https://artificialanalysis.ai/agents/coding-agents) [Methodology and version history](https://artificialanalysis.ai/methodology/coding-agents-benchmarking)

**Use for routing.** Use the component breakdown before the headline index: DeepSWE for implementation, Terminal-Bench for shell/tool-heavy investigations, and SWE-Atlas-QnA for repository exploration and architectural explanation. Use cost and active runtime to choose the lowest configuration that remains comfortably above the task's risk threshold. For the 5.6 family, start with Luna for cheap, well-scoped parallel research or mechanical work; Sol for primary execution, difficult diagnosis, synthesis, and orchestration; and select Terra only when its measured coding profile or a local trial beats the neighboring Luna/Sol option.

## Combined decision rule

1. Match the benchmark component to the work; never route from a headline score alone.
2. Choose the model family member based on failure cost, then choose effort based on ambiguity and verification strength.
3. Escalate from Luna to Sol, or from medium/high to xhigh/max, when the task crosses modules, has weak tests, changes architecture, or requires coordinating other agents.
4. Prefer two independent agents over one maximal agent for review only when their roles are genuinely independent; benchmark pass@1 does not measure correlated review blind spots.
5. Evaluate frontend taste separately with visual artifacts and human judgment. These sources measure successful engineering behavior, not whether an interface feels coherent, restrained, or on-brand.
