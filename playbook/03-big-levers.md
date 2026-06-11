# Big Levers: Structural Changes Worth Real Engineering

Part of the [AI Cost Optimization Playbook](00-start-here.md). Previous: [Low-Hanging Fruit](02-low-hanging-fruit.md) · Next: [Culture](04-culture.md)

---

These are the moves that cut a bill 50–90% but take weeks, not days. Each one has a clear "when it's worth it" line. Pull the lever only when you're past the line — every one of these has a failure mode where it costs more than it saves.

## 1. Model routing and cascades

**Worth it when:** your traffic is a mix of easy and hard queries on one API. **Skip when:** you call one model for one narrow task — there's nothing to route.

A trained router sends the easy 80% of queries to a cheap model and reserves the frontier model for the rest. RouteLLM's published result: 85% cost reduction on MT-Bench while keeping 95% of GPT-4-level quality.

- [RouteLLM](https://github.com/lm-sys/RouteLLM) — the canonical open source baseline from LMSYS.
- [LLMRouter](https://github.com/ulab-uiuc/LLMRouter) — UIUC's routing library (Dec 2025): 16+ routing methods across single-round, multi-round, agentic, and personalized categories.
- [RouterArena](https://arxiv.org/abs/2510.00202) — live leaderboard comparing academic and commercial routers head-to-head. Check here before picking one.
- [OpenRouter Auto Router](https://openrouter.ai/docs/guides/routing/routers/auto-router) — `openrouter/auto` routes across 19 models; the routing itself is free, you pay the selected model's normal rate.
- [FrugalGPT](https://github.com/stanford-futuredata/FrugalGPT) — the foundational cascade paper and code. Required reading before building any router.

## 2. Self-hosting

**Worth it when:** sustained 10M+ tokens/day, steady traffic, and you have (or are hiring) the engineer to own it. **Skip when:** traffic is spiky or under ~5M tokens/day — idle GPUs bill the same as busy ones.

The break-even math, June 2026:

- An H100 runs ~$2.00–2.80/hr on-demand ([RunPod](https://www.runpod.io/pricing): H100 PCIe $1.99/hr; [Vast.ai](https://vast.ai/pricing) marketplace from $1.49/hr) — roughly $1,500–2,000/month running 24/7.
- A quantized 70B-class model on vLLM serves enough throughput that against Sonnet-class API pricing ($3 in / $15 out per million), break-even lands around 2–5M tokens/day; self-hosting wins clearly by 10M/day.
- Against Haiku-class pricing ($1 in / $5 out), break-even pushes to ~15–25M tokens/day. Cheap APIs are very hard to beat.
- The hidden line item that flips the math: setup plus 10–20% of an engineer permanently, call it $4–6K/month. **Self-hosting is a payroll decision, not a GPU decision.**

The serving stack:

- [vLLM](https://github.com/vllm-project/vllm) — the default; V1 engine, deepest ecosystem and Kubernetes maturity.
- [SGLang](https://github.com/sgl-project/sglang) — wins on prefix-heavy and structured-output workloads (~29% higher throughput on H100, ~3× faster constrained JSON decoding).
- [NVIDIA Dynamo](https://github.com/ai-dynamo/dynamo) — datacenter-scale disaggregated serving (separate prefill/decode pools); GA since March 2026.
- [LMCache](https://github.com/LMCache/LMCache) — persists and shares KV caches across requests on top of vLLM/Dynamo.
- [SkyPilot](https://github.com/skypilot-org/skypilot) — auto-routes workloads to the cheapest available cloud/region/spot.
- [GetDeploying GPU price tracker](https://getdeploying.com/gpus/nvidia-h200) — live H100/H200 prices across 32+ providers.

## 3. Distillation: train a small model on the big model's answers

**Worth it when:** one narrow task runs millions of times a day. **Skip when:** the task keeps changing — a distilled model is frozen the day you train it.

If a single prompt pattern dominates your bill, generate training data with the frontier model once, fine-tune a small open model on it, and serve it for cents.

One thing changed in 2026: OpenAI is winding down self-serve fine-tuning (new orgs blocked since May 2026, existing access ends January 2027 — [deprecations page](https://developers.openai.com/api/docs/deprecations)). Open weights are now the durable path — it's the one that can't be taken away.

- [Unsloth](https://github.com/unslothai/unsloth) — fastest single-GPU fine-tuning: 2–5× faster, up to 80% less VRAM.
- [torchtune](https://github.com/pytorch/torchtune) — PyTorch-native recipes including knowledge distillation and quantization-aware training.
- [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) — config-driven multi-GPU training for teams past the single-GPU stage.
- [Is Fine-Tuning Still Valuable? (Hamel Husain)](https://hamel.dev/blog/posts/fine_tuning_valuable.html) — fine-tune only after evals and prompting hit a wall.

## 4. Context engineering for agents

**Worth it when:** you run multi-turn agents whose context grows every step. Agent loops compound: a 26–54% context cut saves more than any single-call trick.

- [Anthropic context editing + memory tool](https://claude.com/blog/context-management) — platform-native: context editing cut token use 84% in a 100-turn agent test.
- [LLMLingua / LLMLingua-2](https://github.com/microsoft/LLMLingua) — the reference prompt compressor, up to 20× compression. Stable but quiet since 2024.
- [ACON](https://arxiv.org/abs/2510.00615) — context compression built for long-horizon agents; 26–54% peak-token reduction with task performance held.
- [GPTCache](https://github.com/zilliztech/GPTCache) — semantic caching: near-duplicate questions stop costing money. Cache lookup in milliseconds vs a full LLM call.

## 5. The storage layer: vectors are the most overpaid line item

Most teams don't need a hosted vector database. pgvector on the PostgreSQL you already run handles millions of vectors; object-storage-backed stores handle billions at commodity prices.

- [Amazon S3 Vectors](https://aws.amazon.com/s3/features/vectors/) — $0.06/GB/month storage vs Pinecone's $0.33/GB (plus Pinecone Standard's $50/month minimum, which is where the dramatic small-workload savings come from). Caveat: S3 Vectors query cost scales with index size scanned, so at very high query volume on large indexes the gap narrows.
- [pgvector](https://github.com/pgvector/pgvector) — use the database you already have.
- [turbopuffer](https://turbopuffer.com/) — object-storage-backed; powers Notion's vector search at 10× scale for 1/10th the cost.
- [Matryoshka embeddings](https://huggingface.co/blog/matryoshka) — truncate embeddings to 256 dims for ~6× storage savings at marginal recall loss.
- [Model2Vec](https://github.com/MinishLab/model2vec) — static distilled embeddings, ~500× faster on CPU; embedding generation cost goes to roughly zero.

## 6. Commitment pricing: a bet on your own forecast

**Worth it when:** 30–60 days of real pay-as-you-go telemetry shows high, steady volume. **Never** buy committed capacity for traffic you hope to have.

Be precise about what's being discounted — these are capacity prices, not token prices:

- [Azure provisioned throughput (PTU)](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/provisioned-throughput) — three purchase modes: hourly (no commitment), monthly reservation, yearly reservation. A monthly reservation discounts the hourly PTU rate; yearly saves roughly another ~35% vs monthly. Total savings vs pay-as-you-go token pricing can reach ~70% — but only at high sustained utilization. Entry point is ~$2,400+/month.
- [AWS Bedrock provisioned throughput](https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html) — billed hourly per model unit; no-commitment, 1-month, and 6-month tiers. The 6-month commit typically runs 20–40% below the 1-month hourly rate. The discount is off the hourly model-unit rate, not off token prices.
- [nOps Bedrock pricing guide](https://www.nops.io/blog/amazon-bedrock-pricing/) — practitioner walkthrough of on-demand vs batch vs provisioned with model-unit examples.

The rule: a PTU you run at 40% utilization is a price increase wearing a discount costume.

---

## Which lever first?

1. **Routing** — fastest payback, a day or two of work, no infrastructure.
2. **Context engineering** — if agents dominate your bill.
3. **Storage layer** — if RAG dominates your bill.
4. **Distillation** — if one high-volume task dominates your bill.
5. **Commitments** — only after 30–60 days of telemetry.
6. **Self-hosting** — last, and only past ~10M steady tokens/day with an owner on payroll.

Next: [Culture →](04-culture.md)
