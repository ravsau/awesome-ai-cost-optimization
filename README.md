# Awesome AI Cost Optimization

A curated list of tools, techniques, benchmarks, and resources for reducing AI infrastructure costs without reducing AI output.

Most teams overspend on AI inference, underuse open source models, and run agentic workflows at a fraction of their potential. This repo is for engineers and leaders who want to fix that.

Organized by leverage: measure first, take the easy wins, then the structural changes.

---

## The Playbook

The short version — what to do, in what order:

1. [Start Here — the order of operations](playbook/00-start-here.md)
2. [Must-Dos — measure first](playbook/01-must-dos.md)
3. [Low-Hanging Fruit — days of work](playbook/02-low-hanging-fruit.md)
4. [Big Levers — weeks of work, with break-even lines](playbook/03-big-levers.md)
5. [Culture — what keeps fixes from regressing](playbook/04-culture.md)
6. [Audit Checklist — 15 questions, quarterly](playbook/05-audit-checklist.md)

---

## Contents

**Tier 0 — Must-Dos (measure first)**
- [Monitoring, Observability, and Gateways](#monitoring-observability-and-gateways)
- [Budgets and Guardrails](#budgets-and-guardrails)
- [Evals and Model Comparison](#evals-and-model-comparison)
- [FinOps and Unit Economics](#finops-and-unit-economics)

**Tier 1 — Low-Hanging Fruit (the first week)**
- [Inference Optimization](#inference-optimization)
- [Dev and Test Hygiene](#dev-and-test-hygiene)

**Tier 2 — Big Levers (weeks of work, 50–90% cuts)**
- [Model Selection and Routing](#model-selection-and-routing)
- [Open Source Models](#open-source-models)
- [Self-Hosting and Serving Infrastructure](#self-hosting-and-serving-infrastructure)
- [Fine-Tuning and Distillation](#fine-tuning-and-distillation)
- [Prompt and Context Compression](#prompt-and-context-compression)
- [Embeddings and Vector Storage](#embeddings-and-vector-storage)
- [Agentic Workflow Efficiency](#agentic-workflow-efficiency)
- [Commitment Pricing](#commitment-pricing)

**Tier 3 — Culture (what makes it stick)**
- [Cost-Aware Engineering Culture](#cost-aware-engineering-culture)
- [Case Studies](#case-studies)
- [Anti-Patterns and Postmortems](#anti-patterns-and-postmortems)

**Reference**
- [Pricing Comparisons](#pricing-comparisons)
- [Patterns and Snippets](#patterns-and-snippets)
- [Suggested Reading](#suggested-reading)
- [Articles, Papers, and Talks](#articles-papers-and-talks)
- [Related Lists](#related-lists)
- [Tools](#tools)
- [Community Tools](#community-tools)

---

# Tier 0 — Must-Dos

Visibility and guardrails before optimization. None of this saves money directly; it's what makes the rest possible.

## Monitoring, Observability, and Gateways

A gateway gives every request a record: who called, which model, how many tokens, what it cost.

**Gateways:**

- [LiteLLM](https://github.com/BerriAI/litellm) - The default self-hosted pick. 100+ providers, per-key/team budgets in Postgres. Strains around ~2K req/s; fine for almost everyone.
- [Portkey Gateway](https://github.com/Portkey-AI/gateway) - Fully open source (Apache 2.0) since March 2026 — governance, cost controls, and MCP gateway included.
- [Bifrost](https://github.com/maximhq/bifrost) - Go gateway by Maxim AI; under 100µs overhead at 5K req/s. The pick when Python proxies become the bottleneck.
- [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) - 350+ models, edge cache, generous free tier. Managed only.
- [Kong AI Gateway](https://github.com/Kong/kong) - AI plugins on Kong's OSS gateway; rate-limit, cost-budget, semantic cache.
- [Vercel AI Gateway](https://vercel.com/ai-gateway) - Sub-20ms routing across 100+ models.

**Observability:**

- [Langfuse](https://github.com/langfuse/langfuse) - Most-adopted open source LLM observability; automatic cost calculation per generation. Acquired by ClickHouse, Jan 2026.
- [Helicone](https://github.com/Helicone/helicone) - Open source observability plus a Rust gateway; one-line integration.
- [tokencost](https://github.com/AgentOps-AI/tokencost) - $ estimates for 400+ LLMs *before* you call.
- [ccusage](https://github.com/ryoppippi/ccusage) - CLI for local coding-agent spend (Claude Code, Codex, Gemini CLI, +more). The de facto individual-dev standard.
- [Claude Code monitoring](https://code.claude.com/docs/en/monitoring-usage) - Native OpenTelemetry export: tokens, cost, cache efficiency by user/model/subagent.
- [claude-code-otel](https://github.com/ColeMurray/claude-code-otel) - Ready-made OTel + Prometheus + Grafana stack for team-level agent spend.

## Budgets and Guardrails

Every key gets a dollar cap and a model allow-list before it ships — including dev and CI keys.

- [LiteLLM budgets and rate limits](https://docs.litellm.ai/docs/proxy/users) - Per-key/user/team caps with daily/weekly/monthly resets; hard caps return 429, soft caps alert.
- [Cloudflare AI Gateway spend limits](https://blog.cloudflare.com/ai-gateway-spend-limits/) - Shipped June 2026: dollar budgets per model/provider/custom attribute, with optional downgrade-to-cheaper-model instead of blocking.
- [LiteLLM agent iteration budgets](https://docs.litellm.ai/docs/a2a_iteration_budgets) - Caps agent loop iterations; the closest thing to a native runaway-agent kill switch.
- [Traceloop: from bills to budgets](https://www.traceloop.com/blog/from-bills-to-budgets-how-to-track-llm-token-usage-and-cost-per-user) - Tag every call with user/feature/team so budgets are per-dimension, not per-invoice.

Note: budgets written into an agent's prompt don't work — [Ramp's agents ignored them](https://x.com/RampLabs/status/2046642262042657176). Enforce in the gateway.

## Evals and Model Comparison

Before routing to a cheaper model, prove it's good enough on your actual tasks.

- [promptfoo](https://github.com/promptfoo/promptfoo) - Open source CLI to test prompts, models, and RAG against your own dataset. ~15 minutes to wire into CI. Used by OpenAI and Anthropic.
- [Inspect](https://github.com/UKGovernmentBEIS/inspect_ai) - UK AI Security Institute's framework; 200+ pre-built evals, strong for agentic and tool-use evals.
- [DeepEval](https://github.com/confident-ai/deepeval) - Pytest-style eval framework, 50+ metrics, built for CI gating.
- [Braintrust](https://www.braintrust.dev/) - Hosted eval platform; the "block on regression, not absolute threshold" workflow with baseline history.
- [Langfuse Evals](https://langfuse.com/docs/scores/overview) - Open source eval scoring tied to traces and cost.
- [Ragas](https://github.com/explodinggradients/ragas) - Eval framework specifically for RAG pipelines.
- [OpenAI Evals](https://github.com/openai/evals) - Open framework for evaluating LLMs and LLM systems.

The downshift workflow: 50–100 real examples from production traces → baseline the current model → run the cheaper candidate → ship only if nothing critical regresses.

## FinOps and Unit Economics

Measure cost per completed task — per resolved ticket, per merged PR — not per API call; agent loops make request counts meaningless.

- [FinOps for AI (FinOps Foundation)](https://www.finops.org/wg/finops-for-ai-overview/) - The official framework; AI is now a formal FinOps technology category. Cost-per-token as a first-class KPI; showback before chargeback.
- [CloudZero: FinOps for AI](https://www.cloudzero.com/blog/finops-for-ai/) - Why traditional tagging breaks for LLMs and how to capture allocation at the gateway layer instead.
- [Finout: FinOps for AI agents](https://www.finout.io/blog/finops-for-ai-agents-a-four-step-allocation-framework) - Four-step allocation framework for agentic workloads.
- [OpenMeter](https://github.com/openmeterio/openmeter) - Open source usage metering with first-class token metering; the per-customer margin layer once AI cost is your COGS.
- [Orb](https://www.withorb.com/) - Usage-based billing engine; metrics defined in SQL.
- [TrueFoundry: team budgets and chargeback](https://www.truefoundry.com/blog/llm-cost-attribution-team-budgets) - Chargeback reporting down to team / agent / model.

Two metadata fields on every request — team and feature — turn "we spent $40K on AI" into "the support summarizer costs $0.11 per ticket." Attribution can't be retrofitted.

---

# Tier 1 — Low-Hanging Fruit

Ships in days, low risk.

## Inference Optimization

Reduce cost per token without changing models.

**Caching:**

- [Prompt Caching (Anthropic)](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) - Cache reads bill at 0.1× the input price. Mind the write surcharge: 1.25× input for the 5-minute TTL, 2× for the 1-hour TTL — an unread cache costs more than no cache.
- [Prompt Caching (OpenAI)](https://developers.openai.com/api/docs/guides/prompt-caching) - Automatic on prompts ≥1,024 tokens; cached input bills at 10% of the input price, no write surcharge.
- [Context Caching (Gemini)](https://ai.google.dev/gemini-api/docs/caching) - Implicit caching on by default for 2.5+ models; cached tokens ~10% of input; explicit caching adds $1.00/M tokens/hour storage.
- [Semantic Caching (GPTCache)](https://github.com/zilliztech/GPTCache) - Cache similar queries to avoid redundant API calls entirely.

**Batch and processing tiers:**

- [Message Batches (Anthropic)](https://platform.claude.com/docs/en/build-with-claude/batch-processing) - 50% off all token usage; most batches finish under an hour. Stacks with prompt caching.
- [Batch API (OpenAI)](https://developers.openai.com/api/docs/guides/batch) - Flat 50% off every model, 24-hour window. Stacked with caching: ~75% off repeated prompts.
- [Flex Processing (OpenAI)](https://developers.openai.com/api/docs/guides/flex-processing) - Batch-level pricing on synchronous calls; slower, may return "resource unavailable." Beta.
- [Gemini batch mode](https://ai.google.dev/gemini-api/docs/pricing) - 50% off all paid models.
- [Priority Processing (OpenAI)](https://openai.com/api-priority-processing/) - The inverse: ~2–2.5× the standard rate for lower latency. Know it exists so you don't pay it by accident.

**Trimming waste:**

- [Tool use overhead (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) - Tool definitions bill as input tokens on every request, plus a 290–675 token hidden system prompt. Unused tools are a recurring tax — delete them.
- [Structured Outputs (Anthropic)](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) / [(OpenAI)](https://developers.openai.com/api/docs/guides/structured-outputs) - Schema-enforced output kills the retry-on-bad-parse loop; every retry is a full-price request.
- [Token counting (Anthropic)](https://platform.claude.com/docs/en/build-with-claude/token-counting) - Free endpoint. Know what your system prompt costs before sending it a million times.

Caching is prefix-matched — a timestamp at the top of the system prompt means zero cache hits. Stable content first, then check `cache_read_input_tokens` is non-zero.

## Dev and Test Hygiene

Surprise bills often come from dev, not prod — e.g. a retry loop in CI running all weekend.

- [vcrpy](https://github.com/kevin1024/vcrpy) - Record real API interactions once, replay in every CI run; test cost drops to ~$0.
- [llmock](https://github.com/CopilotKit/llmock) - Deterministic mock LLM server with streaming and record/replay.
- [mockllm](https://github.com/StacklokLabs/mockllm) - YAML-configured mock server mimicking OpenAI/Anthropic wire formats.
- [Ollama](https://ollama.com/) - Run open models locally for dev loops and CI smoke tests. $0/M tokens.
- [LiteLLM per-key budgets](https://docs.litellm.ai/docs/proxy/users) - A `max_budget` on every dev key caps the blast radius of any runaway script.

---

# Tier 2 — Big Levers

Weeks of work, bigger cuts, real failure modes. Break-even lines in the [playbook page](playbook/03-big-levers.md).

## Model Selection and Routing

The flagship-to-cheap-tier price gap is 25–50×. Stop using Opus for tasks Haiku can handle.

- [RouteLLM](https://github.com/lm-sys/RouteLLM) - The canonical OSS router (LMSYS). Published result: 85% cost reduction at 95% of GPT-4 quality on MT-Bench.
- [LLMRouter](https://github.com/ulab-uiuc/LLMRouter) - UIUC's routing library (Dec 2025): 16+ routing methods — single-round, multi-round, agentic, personalized.
- [RouterArena](https://arxiv.org/abs/2510.00202) - Live leaderboard comparing academic and commercial routers head-to-head. Check before picking one.
- [OpenRouter Auto Router](https://openrouter.ai/docs/guides/routing/routers/auto-router) - `openrouter/auto` routes across 19 models; routing is free, you pay the selected model's normal rate.
- [FrugalGPT (reference impl)](https://github.com/stanford-futuredata/FrugalGPT) - The foundational LLM-cascade paper + code. Required reading before building any router.
- [OptiLLM](https://github.com/codelion/optillm) - OpenAI-compatible proxy with 20+ inference-time techniques that let you downshift to a cheaper model.
- [Cascade Routing (ETH)](https://github.com/eth-sri/cascade-routing) - Unified routing + cascading; beats either alone on the cost/quality frontier.
- [Artificial Analysis](https://artificialanalysis.ai/) - Independent benchmarks with price/performance rankings.

Downshift by task, not by app, and gate every downgrade with an eval.

## Open Source Models

Open-weight options for tasks that don't need a frontier API, and for self-hosting/distillation.

- [Ollama](https://ollama.com/) - Run open source models locally with one command.
- [Qwen](https://huggingface.co/Qwen) - Strong open-weight family; small versions run on laptops. The default local-agent pick.
- [Llama](https://llama.meta.com/) - Meta's open source model family.
- [Mistral](https://mistral.ai/) - European open source models, strong multilingual.
- [DeepSeek](https://github.com/deepseek-ai) - Cost-efficient open models; DeepSeek-V3 was trained for ~$5.6M total — the upper bound on what "frontier" actually has to cost.
- [LMSYS Chatbot Arena](https://chat.lmsys.org/) - Crowdsourced rankings; find where open source matches proprietary.

## Self-Hosting and Serving Infrastructure

Only pays off at sustained volume (~10M+ tokens/day) — and it permanently costs part of an engineer (~$4–6K/month). Idle GPUs bill the same as busy ones.

**Inference engines:**

- [vLLM](https://github.com/vllm-project/vllm) - The default: V1 engine, deepest ecosystem and Kubernetes maturity.
- [SGLang](https://github.com/sgl-project/sglang) - Wins prefix-heavy and structured-output workloads (~29% higher throughput on H100, ~3× faster constrained JSON decoding).
- [NVIDIA Dynamo](https://github.com/ai-dynamo/dynamo) - Datacenter-scale disaggregated serving (separate prefill/decode GPU pools). GA March 2026.
- [LMCache](https://github.com/LMCache/LMCache) - Persists and shares KV caches across requests on top of vLLM/Dynamo.
- [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) - NVIDIA's optimized kernels; 2–4× throughput vs vanilla on H100/H200.
- [TGI (Text Generation Inference)](https://github.com/huggingface/text-generation-inference) - HF's production server with continuous batching.
- [llama.cpp](https://github.com/ggerganov/llama.cpp) - The reference CPU/Metal/edge runtime.
- [llamafile](https://github.com/Mozilla-Ocho/llamafile) - Single-file LLM executable; cost-of-ops near zero.
- [LocalAI](https://github.com/mudler/LocalAI) - Drop-in OpenAI-compatible API for local inference.

**Quantization:**

- [AutoAWQ](https://github.com/casper-hansen/AutoAWQ) - 4-bit weight quantization; cuts VRAM ~70%, runs 70B on a single A100.
- [ExLlamaV2](https://github.com/turboderp-org/exllamav2) - Fastest 4-bit consumer-GPU inference; 70B on 2× 3090s.

**Edge / browser:**

- [MLC LLM](https://github.com/mlc-ai/mlc-llm) - Compiles models to phones, browsers, edge.
- [WebLLM](https://github.com/mlc-ai/web-llm) - In-browser inference via WebGPU; $0 inference cost per user.

**Orchestration and pricing:**

- [SkyPilot](https://github.com/skypilot-org/skypilot) - Auto-routes workloads to the cheapest available cloud/region/spot across 20+ providers.
- [dstack](https://github.com/dstackai/dstack) - Open GPU orchestrator across clouds + on-prem; spot/preemptible scheduling.
- [GetDeploying GPU price tracker](https://getdeploying.com/gpus/nvidia-h200) - Live H100/H200 prices across 32+ providers. (June 2026: H200 on-demand median ~$4.11/hr, spot floor ~$1/hr.)
- [RunPod](https://www.runpod.io/pricing) / [Vast.ai](https://vast.ai/pricing) - H100 from ~$1.49–2.69/hr depending on tier and reliability tolerance.
- [BentoML / OpenLLM](https://github.com/bentoml/OpenLLM) - One-command OpenAI-compatible serving of any open model on any cloud.

Rough break-even at June 2026 prices: vs Sonnet-class API pricing, self-hosting a quantized 70B wins by ~10M tokens/day; vs Haiku-class, ~15–25M/day. Below that, privacy and latency are the only arguments.

## Fine-Tuning and Distillation

When one narrow task runs millions of times a day, train a small open model on the frontier model's outputs and serve it for cents. Skip it when the task keeps changing — a distilled model is frozen the day you train it.

- [Unsloth](https://github.com/unslothai/unsloth) - Fastest single-GPU fine-tuning: 2–5× faster, up to 80% less VRAM.
- [torchtune](https://github.com/pytorch/torchtune) - PyTorch-native recipes including knowledge distillation and quantization-aware training.
- [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) - Config-driven multi-GPU training for teams past the single-GPU stage.
- [Is Fine-Tuning Still Valuable? (Hamel Husain)](https://hamel.dev/blog/posts/fine_tuning_valuable.html) - Fine-tune only after evals + prompting hit a wall; otherwise you're spending training $ on a routing problem.
- [OpenAI deprecations](https://developers.openai.com/api/docs/deprecations) - OpenAI is winding down self-serve fine-tuning (new orgs blocked May 2026, all access ends Jan 2027). Open weights are the durable distillation path.

## Prompt and Context Compression

The cheapest token is the one you never send. Long contexts in agent loops compound — a context cut saves more per call than any caching trick, and they stack.

- [LLMLingua / LLMLingua-2](https://github.com/microsoft/LLMLingua) - Microsoft Research. Up to 20× compression with minimal accuracy loss; v2 is 3–6× faster. Stable but quiet since 2024.
- [Anthropic context editing + memory](https://claude.com/blog/context-management) - Platform-native context management; context editing cut token use 84% in a 100-turn agent test.
- [ACON](https://arxiv.org/abs/2510.00615) - Context compression built for long-horizon agents; 26–54% peak-token reduction with task performance held.
- [Selective-Context](https://github.com/liyucheng09/Selective_Context) - Drops low self-information tokens; 50%+ cuts on RAG contexts.
- [500xCompressor](https://github.com/ZongqianLi/500xCompressor) - Compresses up to 500 NL tokens into 1 soft token; retains 62–73% of capability. ACL 2025.
- [PCToolkit](https://github.com/3DAgentWorld/Toolkit-for-Prompt-Compression) - Benchmarking toolkit comparing the major compressors side-by-side.

## Embeddings and Vector Storage

The most overpaid line item in RAG. Most teams don't need a hosted vector database.

**Vector storage:**

- [Amazon S3 Vectors](https://aws.amazon.com/s3/features/vectors/) - $0.06/GB/month storage vs Pinecone $0.33/GB (plus Pinecone Standard's $50/month minimum). Caveat: query cost scales with index size scanned, so at very high query volume on large indexes the gap narrows.
- [pgvector](https://github.com/pgvector/pgvector) - Open source vector extension for PostgreSQL — use the database you already run. Handles millions of vectors.
- [turbopuffer](https://turbopuffer.com/) - Object-storage-backed vector DB; powers Notion's 10× scale at 1/10th cost.
- [Qdrant](https://github.com/qdrant/qdrant) - Open source vector DB with on-premise option.
- [Chroma](https://github.com/chroma-core/chroma) - Open source embedding database.
- [SQLite-VSS](https://github.com/asg017/sqlite-vss) - SQLite vector search; zero-cost for small datasets.

**Embedding cost reduction:**

- [Model2Vec](https://github.com/MinishLab/model2vec) - Static distilled embeddings ~50× smaller and ~500× faster than sentence-transformers on CPU.
- [FastEmbed](https://github.com/qdrant/fastembed) - Quantized ONNX embedding models; no PyTorch dependency, runs on CPU.
- [sentence-transformers quantization](https://github.com/UKPLab/sentence-transformers) - INT8/binary quantization; ~32× less memory at ~96% recall.
- [Matryoshka embeddings](https://huggingface.co/blog/matryoshka) - Truncate to 256 dims; ~6× storage savings at marginal recall loss.

## Agentic Workflow Efficiency

Agents multiply costs because they loop. Efficient agent design is cost design. Measure cost per successful task completion, not cost per API call.

- [agenttrace](https://github.com/luoyuctl/agenttrace) - Tracks coding-agent token, cost, latency, and failure regressions from local trace logs.
- [ccusage](https://github.com/ryoppippi/ccusage) - Local spend reports for Claude Code, Codex, Gemini CLI, and more.
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) - Agentic coding with cost tracking and native OpenTelemetry export.
- [LangSmith](https://smith.langchain.com/) - Trace and monitor agent costs per run.
- [AgentOps](https://github.com/AgentOps-AI/agentops) - Observability for AI agents; cost per agent run.
- [LiteLLM agent iteration budgets](https://docs.litellm.ai/docs/a2a_iteration_budgets) - Step caps for agent loops.

**Key insight:** An unoptimized agent can burn $5–50 per run. Set token budgets at the platform layer, use smaller models for tool selection, cap iterations, and add a loop detector — a hash of the last tool call catches an infinite loop on iteration two.

## Commitment Pricing

Every major provider sells committed capacity or committed spend at a discount. Whether it pays off depends entirely on your traffic shape — explore it against 30–60 days of your own pay-as-you-go telemetry, not the sales deck. Be precise about what's discounted: usually a capacity rate, not token prices.

**Per provider:**

- [Azure OpenAI provisioned throughput (PTU)](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/provisioned-throughput) - Three purchase modes: hourly (no commitment), monthly reservation, yearly reservation. Monthly discounts the hourly PTU rate; yearly saves roughly another ~35% vs monthly; up to ~70% total vs pay-as-you-go token pricing at high sustained utilization. Entry ~$2,400+/month.
- [AWS Bedrock provisioned throughput](https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html) - Billed hourly per model unit; no-commitment, 1-month, and 6-month tiers. The 6-month commit typically runs 20–40% below the 1-month hourly rate. ([nOps walkthrough](https://www.nops.io/blog/amazon-bedrock-pricing/))
- [OpenAI Scale Tier](https://openai.com/api-scale-tier/) - Buy "token units" (fixed input+output TPM for one model snapshot), 30-day minimum per unit, billed monthly. Usage averaged in 15-minute windows; overflow spills to pay-as-you-go. Flat-rate capacity, so it only beats per-token billing at high sustained utilization.
- [OpenAI Guaranteed Capacity](https://openai.com/reserved-capacity/) - Committed-spend program (May 2026): 1-, 2-, or 3-year terms, discounts increase with term length, drawdown across OpenAI's portfolio. Terms reported via [CNBC](https://www.cnbc.com/2026/05/19/openai-announces-new-guaranteed-capacity-offering-for-customers-to-secure-compute.html); confirm specifics with sales.
- [Anthropic Priority Tier](https://platform.claude.com/docs/en/api/service-tiers) - Commit to input TPM + output TPM for a specific model version, terms of 1/3/6/12 months, longer terms discounted (percentages via sales). Overflow falls back to Standard tier automatically with `service_tier: "auto"`.
- [Vertex AI Provisioned Throughput](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/provisioned-throughput) - Buy GSUs (model-specific tokens/sec). Per the [pricing table](https://cloud.google.com/vertex-ai/generative-ai/pricing): 1-week $1,200/GSU, 1-month $2,700, 3-month $2,400/mo (~11% off the 1-month rate), 1-year $2,000/mo (~26% off). Non-cancelable, charged regardless of usage — but GSUs can be re-pointed to newer models mid-term, which the other providers don't allow.

**Cloud-contract angle** (where AI spend meets your existing enterprise commit):

- Azure OpenAI consumption counts toward [MACC](https://learn.microsoft.com/en-us/marketplace/azure-consumption-commitment-benefit) drawdown.
- Bedrock can count toward an AWS EDP/PPA, but AI-service inclusion is negotiated, not automatic.
- GCP CUDs do not cover Gemini token inference — Provisioned Throughput is the only commit vehicle there.
- Buying model providers through cloud marketplaces (Claude via AWS Marketplace, etc.) can burn down existing commits; drawdown caps are contract-specific.
- [FinOps Foundation: Navigating GenAI Capacity Options](https://www.finops.org/wg/genai-capacity-options/) - Cross-provider framework for exactly this decision.

**Decision rules:**

1. Break-even is the discount ratio: a 26% discount needs roughly ≥74% sustained utilization to beat the shorter term.
2. Commit to your baseload (P50 traffic), let overflow spill to pay-as-you-go — all three capacity programs overflow gracefully.
3. Use-it-or-lose-it is per minute or per 15-minute window, not per month — spiky traffic wastes committed capacity even when monthly totals look fine.
4. Check model lock before signing: OpenAI Scale Tier and Anthropic Priority Tier bind to a model snapshot; a mid-term model release can strand you. Vertex GSUs can move.

---

# Tier 3 — Culture

Technical fixes regress; these are the habits that hold them.

## Cost-Aware Engineering Culture

Cost visibility belongs with the engineers who can change it, not just finance.

- [FinOps for AI (FinOps Foundation)](https://www.finops.org/wg/finops-for-ai-overview/) - 78% of FinOps practices now report into the CTO/CIO org, up 18 points since 2023. The bill is moving from finance to engineering.
- [Ramp: The Trillion-Dollar AI Blindspot](https://ramp.com/blog/trillion-dollar-ai-blindspot) - Customer data: average monthly AI token spend up 13× since January 2025.
- [Ramp Builders: a unified pipeline for AI token spend](https://builders.ramp.com/post/ai-token-spend-management) - How Ramp's engineers built spend attribution.
- [Inside Ramp: the AI adoption playbook (Geoff Charles)](https://creatoreconomy.so/p/inside-ramp-the-32b-company-ai-agents-geoff-charles) - Define levels, measure publicly, remove constraints — with cost governance built into the rollout.
- [The Pragmatic Engineer: AI's impact on engineers in 2026](https://newsletter.pragmaticengineer.com/p/the-impact-of-ai-on-software-engineers-2026) - What companies actually pay: max-tier coding-agent plans at $100–200/engineer/month; budget orgs on $20 Copilot tiers.
- [Uber caps coding-agent spend](https://www.washingtontimes.com/news/2026/jun/3/uber-capping-internal-use-ai-coding-software-blowing-budget/) - Annual AI coding budget gone in 4 months → $1,500/month cap per tool per engineer.

Put cost per feature on the same dashboard as latency and error rate.

## Case Studies

Real-world examples with concrete numbers.

- [Two Years of Vector Search at Notion](https://www.notion.com/blog/two-years-of-vector-search-at-notion) - Migrated to turbopuffer + serverless + data-volume cuts: 10× scale at 1/10th the cost.
- [How ProjectDiscovery Cut LLM Costs by 59% With Prompt Caching](https://projectdiscovery.io/blog/how-we-cut-llm-cost-with-prompt-caching) - Cache hit rate 7% → 84% on agents consuming ~60M tokens per task; ~70% savings today.
- [Thomson Reuters Labs: prompt caching in production](https://medium.com/tr-labs-ml-engineering-blog/prompt-caching-the-secret-to-60-cost-reduction-in-llm-applications-6c792a0ac29b) - 60% cost cut; example request $0.34 → $0.14, plus 20% faster responses.
- [From $720 to $72/month on API costs](https://labeveryday.medium.com/prompt-caching-is-a-must-how-i-went-from-spending-720-to-72-monthly-on-api-costs-3086f3635d63) - Per-request caching math with an 81k-token system prompt: 90% reduction.
- [Klarna's AI assistant](https://www.klarna.com/international/press/klarna-ai-assistant-handles-two-thirds-of-customer-service-chats-in-its-first-month/) - 2.3M chats in month one, the work of ~700 agents, est. $40M profit improvement. (Later re-added humans for complex cases — automation has a quality floor.)
- [RouteLLM benchmark results](https://lmsys.org/blog/2024-07-01-routellm/) - 85% cost reduction on MT-Bench holding 95% of GPT-4 quality.
- [Anthropic Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) - Contextualizing chunks across a 500-page PDF: $6.30 total with prompt caching; 49–67% fewer retrieval failures.

*Have a case study with real numbers? [Submit a PR.](#contributing)*

## Anti-Patterns and Postmortems

Common thread: monitoring without enforcement — dashboards, but no gate that refuses the next call.

- [How an AI agent ran up a $47,000 bill in 11 days](https://dev.to/dingdawg/how-an-ai-agent-ran-up-a-47000-bill-in-11-days-and-how-to-stop-it-1fk) - Four agents in an infinite clarification loop; weekly bills $127 → $891 → $6,240 → $18,400. A payload-hash loop detector would have caught it on iteration two.
- [The agent that burned $4,200 in 63 hours](https://medium.com/@sattyamjain96/the-agent-that-burned-4-200-in-63-hours-a-production-ai-postmortem-d38fd9586a85) - A plan→call→429→replan retry loop firing ~4,800 times/hour over a weekend. Nobody on call for cost alerts.
- [TrueFoundry: agentic token explosion in CI/CD](https://www.truefoundry.com/blog/llm-cost-attribution-agentic-cicd) - $48K of spend in 14 hours from one misbehaving session; retries compounding context quadratically.
- [Ramp Labs: agents ignore their own budgets](https://x.com/RampLabs/status/2046642262042657176) - Budgets in the prompt don't work; enforcement must live in the platform layer.

---

# Reference

## Pricing Comparisons

Side-by-side cost comparisons and calculators.

- [LLM Price Check](https://llmpricecheck.com/) - Compare LLM API prices across providers.
- [Artificial Analysis](https://artificialanalysis.ai/) - Price/performance/speed benchmarks.
- [LiteLLM model_prices JSON](https://github.com/BerriAI/litellm/blob/main/model_prices_and_context_window.json) - The authoritative price table used by most cost tools.
- [OpenAI pricing](https://developers.openai.com/api/docs/pricing) / [Anthropic models](https://platform.claude.com/docs/en/about-claude/models/overview) / [Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing) - The primary sources; check them, not blog posts.
- [Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/) - AWS Bedrock model pricing.
- [Together AI Pricing](https://www.together.ai/pricing) - Open source model hosting pricing.
- [Groq Pricing](https://groq.com/pricing/) - Fast inference pricing.
- [TinyTools AI Cost Calculator](https://tinytools-smoky.vercel.app/ai-cost-calculator/) - Free browser-based calculator, no signup, runs client-side.

## Patterns and Snippets

Concrete, copy-pasteable techniques. Each snippet is the smallest thing that moves cost meaningfully.

### Anthropic prompt caching

Reads are billed at 10% of input price. ~90% savings on repeated long system prompts. (Writes cost 1.25–2× input — confirm reads are landing.)

```python
import anthropic
client = anthropic.Anthropic()

LARGE_SYSTEM_PROMPT = open("docs/handbook.md").read()  # e.g. 50k tokens

resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=512,
    system=[
        {
            "type": "text",
            "text": LARGE_SYSTEM_PROMPT,
            "cache_control": {"type": "ephemeral"},  # the cache breakpoint
        }
    ],
    messages=[{"role": "user", "content": "Summarize section 3."}],
)
print(resp.usage)  # cache_read_input_tokens > 0 on the second call
```

### OpenAI Batch API submission

50% discount on async (up to 24h) batch jobs.

```python
import json
from openai import OpenAI
client = OpenAI()

with open("batch.jsonl", "w") as f:
    for i, prompt in enumerate(prompts):
        f.write(json.dumps({
            "custom_id": f"req-{i}",
            "method": "POST",
            "url": "/v1/chat/completions",
            "body": {"model": "gpt-4o-mini",
                     "messages": [{"role": "user", "content": prompt}]}
        }) + "\n")

upload = client.files.create(file=open("batch.jsonl", "rb"), purpose="batch")
batch = client.batches.create(
    input_file_id=upload.id,
    endpoint="/v1/chat/completions",
    completion_window="24h",
)
print(batch.id, batch.status)
```

### LiteLLM fallback ladder

Cheap model first, escalate on failure. Typical mix: 80% on Haiku, ~15% on Sonnet, ~5% on Opus.

```python
from litellm import Router

router = Router(
    model_list=[
        {"model_name": "tier-cheap",
         "litellm_params": {"model": "anthropic/claude-haiku-4-5"}},
        {"model_name": "tier-mid",
         "litellm_params": {"model": "anthropic/claude-sonnet-4-6"}},
        {"model_name": "tier-strong",
         "litellm_params": {"model": "anthropic/claude-opus-4-7"}},
    ],
    fallbacks=[{"tier-cheap": ["tier-mid", "tier-strong"]}],
    num_retries=1,
)

resp = router.completion(
    model="tier-cheap",
    messages=[{"role": "user", "content": "Classify: ..."}],
)
```

### Redis-backed semantic cache

Near-duplicate questions stop costing money. Effective when input-embedding cost is small vs the completion you'd otherwise pay for.

```python
import redis, numpy as np
from openai import OpenAI
oai, r = OpenAI(), redis.Redis(decode_responses=False)

def embed(text):
    return np.array(oai.embeddings.create(
        model="text-embedding-3-small", input=text, dimensions=256
    ).data[0].embedding, dtype=np.float32)

def cached_completion(prompt, threshold=0.93):
    q = embed(prompt)
    for key in r.scan_iter(b"cache:*"):
        v = np.frombuffer(r.hget(key, b"vec"), dtype=np.float32)
        sim = float(q @ v) / (np.linalg.norm(q) * np.linalg.norm(v))
        if sim > threshold:
            return r.hget(key, b"answer").decode()
    ans = oai.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
    ).choices[0].message.content
    r.hset(f"cache:{hash(prompt)}",
           mapping={b"vec": q.tobytes(), b"answer": ans.encode()})
    return ans
```

### LLMLingua compression

Up to 20x fewer input tokens before calling the expensive model.

```python
from llmlingua import PromptCompressor
from openai import OpenAI

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-xlm-roberta-large-meetingbank",
    use_llmlingua2=True,
)
long_context = open("call_transcript.txt").read()
out = compressor.compress_prompt(
    long_context, rate=0.33, force_tokens=["\n", "?", "."]
)
print("saved tokens:", out["origin_tokens"] - out["compressed_tokens"])

resp = OpenAI().chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user",
               "content": f"{out['compressed_prompt']}\n\nSummarize."}],
)
```

### Cost-guard wrapper

Estimate $ before the call; reject if it'd blow the budget. Stops runaway agent loops.

```python
from tokencost import calculate_prompt_cost, calculate_completion_cost
from openai import OpenAI

MAX_USD_PER_CALL = 0.05
client = OpenAI()

def safe_chat(messages, model="gpt-4o", max_tokens=1024):
    in_cost = calculate_prompt_cost(messages, model)
    est_out_cost = calculate_completion_cost("x " * max_tokens, model)
    total = float(in_cost + est_out_cost)
    if total > MAX_USD_PER_CALL:
        raise RuntimeError(f"Budget exceeded: ${total:.4f} > ${MAX_USD_PER_CALL}")
    return client.chat.completions.create(
        model=model, messages=messages, max_tokens=max_tokens,
    )
```

### Loop detector

Catches an infinite agent loop on iteration two instead of day eleven.

```python
import hashlib

seen = set()

def guard_step(tool_name, payload, max_steps=25):
    h = hashlib.sha256(f"{tool_name}:{payload}".encode()).hexdigest()[:16]
    if h in seen:
        raise RuntimeError(f"Loop detected: repeated call {tool_name}")
    seen.add(h)
    if len(seen) > max_steps:
        raise RuntimeError(f"Step budget exceeded: {max_steps}")
```

## Suggested Reading

The publications worth following to stay current on AI cost and LLM economics. All verified live as of June 2026, ordered roughly by cost-signal density.

**Economics and benchmarks (the core follow list):**

- [SemiAnalysis](https://semianalysis.com) (Dylan Patel) - The reference source for GPU economics, datacenter TCO, and what a token actually costs to serve. ~Weekly; free tier + paid research.
- [Artificial Analysis](https://artificialanalysis.ai) - Independent leaderboards of price per million tokens, speed, and intelligence-per-dollar across 22+ providers. The de facto pricing table for routing decisions. Continuously updated; free.
- [FinOps Foundation](https://www.finops.org) - The standards body formalizing FinOps for AI: token economics framework, working groups, annual State of FinOps report. Community membership free.
- [Tomasz Tunguz](https://www.tomtunguz.com) - Near-daily, chart-heavy posts on AI economics: intelligence-per-dollar, GPU price moves, open-source substitution thresholds. Free.
- [Epoch AI — Gradient Updates](https://epoch.ai) - Rigorous public datasets on inference cost trends, training compute, and frontier datacenters. The primary source for any cost trendline you cite. Weekly; free.

**Practitioner blogs (cost shows up in the engineering):**

- [Simon Willison's Weblog](https://simonwillison.net) - Annotates every model release with token pricing and runs personal spend experiments. The best running ledger of what LLMs cost in practice. Free.
- [The Pragmatic Engineer](https://newsletter.pragmaticengineer.com) (Gergely Orosz) - How engineering orgs actually govern AI tooling spend. Weekly; free tier, deep dives paid.
- [Eugene Yan](https://eugeneyan.com) - Applied LLM system design and evals — the discipline that justifies routing to cheaper models without quality loss. Free.
- [Hamel Husain](https://hamel.dev) - Evals, measurement, and fine-tuning — the "prove the cheap path works" toolkit. Free.
- [Interconnects](https://www.interconnects.ai) (Nathan Lambert) - Frontier-lab analysis with a recurring open-vs-closed economics thread: when open weights change your buy-vs-host math. Free tier + paid.
- [Latent Space](https://www.latent.space) - AI engineering newsletter/podcast; recurring coverage of inference pricing wars and build-vs-buy economics. Free tier + paid.

**Cloud-bill and vendor blogs (filter the pitch, keep the data):**

- [Last Week in AWS](https://www.lastweekinaws.com) (Corey Quinn / Duckbill) - Cloud-bill analysis from people who negotiate enterprise contracts for a living; increasingly covers Bedrock and AI line items. Weekly; free.
- [Vantage Blog](https://www.vantage.sh/blog) - The best vendor blog on AI-as-cloud-cost right now: token budgeting, GPU instance pricing teardowns. Free.
- [CloudZero Blog](https://www.cloudzero.com/blog/) - Steady stream of AI ROI and per-provider pricing explainers. Free.
- [Anthropic Engineering](https://www.anthropic.com/engineering) / [Cloudflare Blog](https://blog.cloudflare.com) - Provider blogs with real cost-lever content: context engineering and agent efficiency (Anthropic); gateway spend limits and caching (Cloudflare). Free.

**If you only follow five:** SemiAnalysis, Artificial Analysis, FinOps Foundation, Simon Willison, Tomasz Tunguz — together they cover supply-side cost (chips and serving), demand-side price (per-token benchmarks), governance (FinOps practice), ground-truth practitioner spend, and the macro money flows.

## Articles, Papers, and Talks

- [Everyone Needs an OpenClaw Strategy](https://www.youtube.com/@CloudYeti) - Jensen Huang said it. Here's what he actually meant: 24/7 agents need a cost strategy, not just a framework. (CloudYeti)
- [The token-to-revenue ratio](https://cloudyeti.io/blog) - Measuring AI spend against business output (CloudYeti)
- [S3 Vectors: 400x cheaper vector search](https://www.youtube.com/watch?v=BWURf-oVFfg) - RAG with S3 Vectors + Bedrock (CloudYeti)
- [Patterns for Building LLM-based Systems & Products](https://eugeneyan.com/writing/llm-patterns/) - Eugene Yan. Reframes caching as "shifting from LLM generation (dollars) to cache storage (cents)." Best single overview of the seven cost+quality patterns.
- [FrugalGPT (paper)](https://arxiv.org/abs/2305.05176) - Chen, Zaharia, Zou (Stanford). The foundational LLM-cascade paper.
- [LLMLingua: Innovating LLM efficiency with prompt compression](https://www.microsoft.com/en-us/research/blog/llmlingua-innovating-llm-efficiency-with-prompt-compression/) - Microsoft Research. 20x compression with minimal accuracy loss.
- [Is Fine-Tuning Still Valuable?](https://hamel.dev/blog/posts/fine_tuning_valuable.html) - Hamel Husain. Fine-tune only after evals + prompting hit a wall.
- [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437) - 671B-param frontier model trained for $5.576M. The upper bound on what "frontier" has to cost.
- [RAG vs long context: a 2026 decision framework](https://open-techstack.com/blog/rag-vs-long-context-2026/) - RAG is up to ~267× cheaper per query at scale, but its fixed costs exceed long-context token costs for small internal tools.
- [The Economics of $20K/month AI Agents](https://medium.com/@mcunningham1440/the-economics-of-openais-20000-month-ai-agents-26b329f301c4) - Daily rate analysis of always-on agents.
- [The True Cost of AI Agents: Hourly Pricing](https://retool.com/blog/cost-of-ai-agents-hourly-pricing-model) - Retool's agent cost framework.

## Related Lists

Other curated lists adjacent to this one. Pull from them for deeper coverage of specific niches.

- [Awesome-LLMOps](https://github.com/tensorchord/Awesome-LLMOps) - Broadest LLMOps tooling list.
- [Awesome-LLM-Inference](https://github.com/xlite-dev/Awesome-LLM-Inference) - Inference papers + code.
- [Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM) - Efficient-LLM research index.
- [Awesome-LLM-Compression](https://github.com/HuangOwen/Awesome-LLM-Compression) - Quantization, pruning, distillation, prompt compression.
- [Awesome-AI-Model-Routing](https://github.com/Not-Diamond/awesome-ai-model-routing) - The canonical routing list.

## Tools

Open source tools specifically built for AI cost management.

| Tool | What It Does | License |
|------|-------------|---------|
| [LiteLLM](https://github.com/BerriAI/litellm) | Unified LLM proxy with spend tracking and budgets | MIT |
| [Portkey Gateway](https://github.com/Portkey-AI/gateway) | AI gateway: governance, cost controls, MCP | Apache 2.0 |
| [Bifrost](https://github.com/maximhq/bifrost) | High-throughput Go gateway with hierarchical budgets | Apache 2.0 |
| [Langfuse](https://github.com/langfuse/langfuse) | Open source LLM cost analytics | MIT |
| [Helicone](https://github.com/Helicone/helicone) | LLM observability + Rust gateway | Apache 2.0 |
| [OpenMeter](https://github.com/openmeterio/openmeter) | Usage/token metering for AI billing | Apache 2.0 |
| [ccusage](https://github.com/ryoppippi/ccusage) | Local coding-agent spend reports | MIT |
| [GPTCache](https://github.com/zilliztech/GPTCache) | Semantic cache for LLM responses | MIT |
| [promptfoo](https://github.com/promptfoo/promptfoo) | Eval prompts/models on your data with cost matrix | MIT |
| [Inspect](https://github.com/UKGovernmentBEIS/inspect_ai) | Eval framework for LLMs and agents | MIT |
| [LLMLingua](https://github.com/microsoft/LLMLingua) | Prompt compression up to 20x | MIT |
| [tokencost](https://github.com/AgentOps-AI/tokencost) | Pre-call $ estimation for 400+ LLMs | MIT |
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput serving | Apache 2.0 |
| [SGLang](https://github.com/sgl-project/sglang) | RadixAttention serving + quantization | Apache 2.0 |
| [SkyPilot](https://github.com/skypilot-org/skypilot) | Cheapest-cloud GPU orchestrator | Apache 2.0 |
| [Unsloth](https://github.com/unslothai/unsloth) | Fast, low-VRAM fine-tuning/distillation | Apache 2.0 |
| [FastEmbed](https://github.com/qdrant/fastembed) | CPU-friendly quantized embeddings | Apache 2.0 |
| [Ollama](https://ollama.com/) | Local model inference | MIT |

## Community Tools

Tools submitted by the community via PR land here first. An entry gets promoted into the curated sections above once it proves out — real adoption, active maintenance, numbers people can verify. Same bar as everything else: it must help someone cut or measure AI spend.

- [traceAI](https://github.com/future-agi/traceAI) - OpenTelemetry-native tracing for LLM and agent apps with 50+ framework integrations; captures token and cost per span.
- [ai-evaluation](https://github.com/future-agi/ai-evaluation) - Eval framework with 50+ metrics, LLM-as-Judge, and guardrail scanners; use to qualify cheaper models against your workload.
- [FerryAPI](https://www.ferryapi.io/) - OpenAI-compatible gateway with prepaid billing, customer API keys, and usage records.
- [LegacyDoc LLM Cost Regression Checker](https://www.romanticode.com/tools/llm-cost-regression-checker/) - Browser-based checker for spotting LLM cost regressions before coding-agent workflows scale.
- [QuotaFlow](https://quotaflow.ai/) - Governed AI quota pooling to reduce wasted subscribed capacity.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

The bar: every link must help someone reduce their AI costs or measure their AI spend more effectively. No vendor marketing pages. No "10 tips" listicles. Tools, benchmarks, techniques, and real case studies only.

---

## About

Maintained by [Saurav Sharma](https://linkedin.com/in/saurav-sharma-cloud) — ex-Amazon SDE, 12x AWS certified. I help teams use AI without wasting money.

- Want this run against your real billing data, with a ranked fix list and dollar figures? That's the [AI Cost Audit](https://cloudyeti.io). Start with a [free 30-minute call](https://cloudyeti.io/chat).
- YouTube: [@CloudYeti](https://www.youtube.com/@CloudYeti)
- Want your tool in front of this audience? See [SPONSORS.md](SPONSORS.md).

---

## License

[CC0 1.0 Universal](LICENSE) — Public domain. Use however you want.
