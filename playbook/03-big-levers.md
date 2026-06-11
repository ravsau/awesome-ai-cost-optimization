# Big Levers

[Playbook](00-start-here.md) · Previous: [Low-Hanging Fruit](02-low-hanging-fruit.md) · Next: [Culture](04-culture.md)

Weeks of work, 50–90% cuts, real failure modes. Each has a break-even line — don't pull early.

## 1. Routing and cascades

Worth it: mixed easy/hard traffic. Skip: one model, one narrow task.

RouteLLM's published result: 85% cost cut at 95% of GPT-4 quality (MT-Bench).

- [RouteLLM](https://github.com/lm-sys/RouteLLM) · [LLMRouter](https://github.com/ulab-uiuc/LLMRouter) (16+ methods) · [RouterArena](https://arxiv.org/abs/2510.00202) (live leaderboard) · [OpenRouter Auto](https://openrouter.ai/docs/guides/routing/routers/auto-router) (routing free, pay the selected model) · [FrugalGPT](https://github.com/stanford-futuredata/FrugalGPT)

## 2. Self-hosting

Worth it: ~10M+ steady tokens/day with an engineer who owns it. Skip: spiky or low traffic — idle GPUs bill the same as busy ones.

Break-even at June 2026 prices: H100 ~$1.50–2.80/hr ([RunPod](https://www.runpod.io/pricing), [Vast.ai](https://vast.ai/pricing), [tracker](https://getdeploying.com/gpus/nvidia-h200)) ≈ $1,500–2,000/month per GPU 24/7. Vs Sonnet-class API pricing, a quantized 70B wins by ~10M tokens/day; vs Haiku-class, ~15–25M/day. Add the operating cost: setup plus 10–20% of an engineer, ~$4–6K/month.

- Serving: [vLLM](https://github.com/vllm-project/vllm) (default) · [SGLang](https://github.com/sgl-project/sglang) (prefix-heavy + structured output) · [NVIDIA Dynamo](https://github.com/ai-dynamo/dynamo) · [LMCache](https://github.com/LMCache/LMCache) · [SkyPilot](https://github.com/skypilot-org/skypilot) (cheapest cloud/spot)

## 3. Distillation

Worth it: one narrow task at huge volume — generate training data with the frontier model, fine-tune a small open model, serve for cents. Skip: tasks that keep changing; the distilled model is frozen at training time.

Note: OpenAI is ending self-serve fine-tuning (new orgs blocked May 2026, all access ends Jan 2027 — [deprecations](https://developers.openai.com/api/docs/deprecations)). Open weights are the durable path.

- [Unsloth](https://github.com/unslothai/unsloth) (single-GPU) · [torchtune](https://github.com/pytorch/torchtune) (distillation recipes) · [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) (multi-GPU) · [Hamel Husain on when to fine-tune](https://hamel.dev/blog/posts/fine_tuning_valuable.html)

## 4. Context engineering for agents

Worth it: multi-turn agents whose context grows every step — savings compound across the loop.

- [Anthropic context editing + memory](https://claude.com/blog/context-management) — 84% token cut in a 100-turn agent test.
- [LLMLingua-2](https://github.com/microsoft/LLMLingua) — up to 20× prompt compression; stable, quiet since 2024.
- [ACON](https://arxiv.org/abs/2510.00615) — 26–54% peak-token cut for long-horizon agents.
- [GPTCache](https://github.com/zilliztech/GPTCache) — semantic cache for near-duplicate queries.

## 5. Vector storage

- [S3 Vectors](https://aws.amazon.com/s3/features/vectors/) — $0.06/GB/mo vs Pinecone $0.33/GB + $50/mo Standard minimum. Caveat: query cost scales with index scanned.
- [pgvector](https://github.com/pgvector/pgvector) — the Postgres you already run; handles millions of vectors.
- [turbopuffer](https://turbopuffer.com/) — Notion's pick: 10× scale at 1/10th cost.
- [Matryoshka truncation](https://huggingface.co/blog/matryoshka) (~6× storage) · [Model2Vec](https://github.com/MinishLab/model2vec) (~500× faster CPU embeddings)

## 6. Commitment pricing

Worth it: only after 30–60 days of telemetry shows steady volume above break-even. These discount **capacity rates, not token prices**.

- [Azure PTU](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/provisioned-throughput) — hourly / monthly / yearly modes; yearly ~35% below monthly; up to ~70% vs pay-as-you-go tokens at high sustained utilization. Entry ~$2,400+/mo.
- [Bedrock provisioned throughput](https://docs.aws.amazon.com/bedrock/latest/userguide/prov-throughput.html) — per model unit; 6-month commit ~20–40% below 1-month hourly rate.
- [nOps Bedrock guide](https://www.nops.io/blog/amazon-bedrock-pricing/)

Next: [Culture →](04-culture.md)
