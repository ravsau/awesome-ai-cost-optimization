# Awesome AI Cost Optimization

A curated list of tools, techniques, benchmarks, and resources for reducing AI infrastructure costs without reducing AI output.

Most teams overspend on AI inference, underuse open source models, and run agentic workflows at a fraction of their potential. This repo is for engineers and leaders who want to fix that.

---

## Contents

- [Model Selection and Routing](#model-selection-and-routing)
- [Evals and Model Comparison](#evals-and-model-comparison)
- [Open Source Models](#open-source-models)
- [Inference Optimization](#inference-optimization)
- [Prompt Compression](#prompt-compression)
- [Self-Hosting and Serving Infrastructure](#self-hosting-and-serving-infrastructure)
- [Embeddings and Vector Storage](#embeddings-and-vector-storage)
- [Agentic Workflow Efficiency](#agentic-workflow-efficiency)
- [Monitoring, Observability, and Gateways](#monitoring-observability-and-gateways)
- [Pricing Comparisons](#pricing-comparisons)
- [Patterns and Snippets](#patterns-and-snippets)
- [Case Studies](#case-studies)
- [Articles, Papers, and Talks](#articles-papers-and-talks)
- [Related Lists](#related-lists)
- [Tools](#tools)

---

## Model Selection and Routing

Picking the right model for the right task is the single biggest cost lever. Stop using Opus for tasks Haiku can handle.

- [Anthropic Model Comparison](https://docs.anthropic.com/en/docs/about-claude/models) - Claude model tiers, pricing, and capability tradeoffs
- [OpenAI Pricing](https://openai.com/api/pricing/) - GPT model pricing per token
- [Google AI Pricing](https://ai.google.dev/pricing) - Gemini model pricing
- [Artificial Analysis](https://artificialanalysis.ai/) - Independent LLM benchmarks with price/performance rankings
- [OpenRouter](https://openrouter.ai/) - Unified API with price comparison across 100+ models
- [Martian Model Router](https://github.com/withmartian/routerbench) - Benchmark for LLM routing systems
- [RouteLLM](https://github.com/lm-sys/RouteLLM) - Framework for serving and evaluating LLM routers (from LMSYS). Cuts GPT-4 calls to 14% of traffic while keeping 95% of quality.
- [FrugalGPT (reference impl)](https://github.com/stanford-futuredata/FrugalGPT) - The canonical LLM cascade paper + code. Matches GPT-4 at up to 98% lower cost.
- [OptiLLM](https://github.com/codelion/optillm) - OpenAI-compatible proxy with 20+ inference-time techniques (MoA, CoT decoding, MCTS) that let you downshift to a cheaper model.
- [Cascade Routing (ETH)](https://github.com/eth-sri/cascade-routing) - Unified routing + cascading; beats either alone on the cost/quality frontier.
- [Not-Diamond awesome-routing](https://github.com/Not-Diamond/awesome-ai-model-routing) - Curated index of routing research.

**Key insight:** Route 80% of requests to small/cheap models. Reserve frontier models for tasks that actually need them. Most classification, extraction, and summarization tasks don't need GPT-4 or Opus.

## Evals and Model Comparison

Before you can route to a cheaper model, you need to prove it's good enough on your actual tasks. Eval tools let you A/B prompts and models against a fixed dataset and compare cost, latency, and quality side-by-side.

- [promptfoo](https://github.com/promptfoo/promptfoo) - Open source CLI to test prompts, models, and RAG against your own dataset. Side-by-side cost + quality matrix.
- [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) - UK AISI's eval framework. Solid for agentic and tool-use evals.
- [OpenAI Evals](https://github.com/openai/evals) - Open framework for evaluating LLMs and LLM systems
- [DeepEval](https://github.com/confident-ai/deepeval) - Pytest-style LLM eval framework with cost/latency tracking
- [Braintrust](https://www.braintrust.dev/) - Hosted eval platform with per-run cost tracking and model comparison
- [LangFuse Evals](https://langfuse.com/docs/scores/overview) - Open source eval scoring tied to traces and cost
- [Helicone Experiments](https://www.helicone.ai/) - Run prompt experiments against production traffic with cost deltas
- [Ragas](https://github.com/explodinggradients/ragas) - Eval framework specifically for RAG pipelines
- [ai-evaluation](https://github.com/future-agi/ai-evaluation) - Open-source LLM evaluation framework with 50+ metrics, LLM-as-Judge, and guardrail scanners (jailbreak, PII, injection). Use to qualify cheaper models against your workload.

**Key insight:** "Cheaper model" is a guess until you eval it. Build a 50-100 example dataset of your real workload, run promptfoo against 4-5 candidate models, and pick the cheapest one that clears your quality bar. Most teams skip this step and overpay forever.

## Open Source Models

Open source can handle 80% of what proprietary APIs do, at a fraction of the cost.

- [Ollama](https://ollama.com/) - Run open source models locally with one command
- [Qwen 3.5](https://huggingface.co/Qwen) - "Towards Native Multimodal Agents." Outperforms GPT-5 mini on function calling by 30%. Small versions (0.8B-9B) run on laptops. The before-and-after moment for local AI agents.
- [Llama 4](https://llama.meta.com/) - Meta's open source model family
- [Mistral](https://mistral.ai/) - European open source models, strong multilingual
- [DeepSeek](https://github.com/deepseek-ai) - Cost-efficient open source models from China. DeepSeek-V3 was trained for ~$5.6M total — the upper bound on what "frontier" actually has to cost.
- [LMSYS Chatbot Arena](https://chat.lmsys.org/) - Crowdsourced LLM rankings (find where open source matches proprietary)

**Key insight:** Run Ollama locally for development, testing, and low-stakes tasks. Use APIs only for production workloads that need frontier capability. Hybrid architecture = massive cost reduction.

## Inference Optimization

Reduce cost per token without changing models.

- [Prompt Caching (Anthropic)](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) - Cache system prompts to reduce repeat token costs by 90%
- [Batch API (Anthropic)](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing) - 50% discount for async batch processing
- [Batch API (OpenAI)](https://platform.openai.com/docs/guides/batch) - 50% discount for batch requests
- [Distillation](https://platform.openai.com/docs/guides/distillation) - Train smaller models on larger model outputs
- [Semantic Caching](https://github.com/gptcache/gptcache) - Cache similar queries to avoid redundant API calls
- [LiteLLM](https://github.com/BerriAI/litellm) - Unified proxy for 100+ LLMs with spend tracking and rate limiting
- [Medusa](https://github.com/FasterDecoding/Medusa) - Speculative decoding with multiple heads; ~2x throughput without a separate draft model.
- [EAGLE](https://github.com/SafeAILab/EAGLE) - Feature-aware speculative sampling; 3x faster than vanilla, 1.6x faster than Medusa.
- [LookaheadDecoding](https://github.com/hao-ai-lab/LookaheadDecoding) - Jacobi-style parallel decoding; 1.5–2.3x speedup, no draft model required.

**Key insight:** Prompt caching alone can cut costs 50-90% for applications with repeated system prompts. If you're not using it, you're leaving money on the table.

## Prompt Compression

The cheapest token is the one you never send. These tools shrink prompts before they hit the API, with measurable quality preservation.

- [LLMLingua](https://github.com/microsoft/LLMLingua) - Microsoft Research. Compresses prompts up to 20x with minimal accuracy loss.
- [LLMLingua-2](https://github.com/microsoft/LLMLingua) - Task-agnostic BERT-class compressor; 3–6x faster compression than LLMLingua-1.
- [Selective-Context](https://github.com/liyucheng09/Selective_Context) - Drops low self-information tokens to cut input length 50%+ on RAG contexts.
- [500xCompressor](https://github.com/ZongqianLi/500xCompressor) - Compresses up to 500 natural-language tokens into 1 soft token; retains 62–73% of capability. ACL 2025.
- [PCToolkit](https://github.com/3DAgentWorld/Toolkit-for-Prompt-Compression) - Unified benchmarking toolkit comparing LLMLingua, SCRL, KiS, Selective Context, RECOMP side-by-side.

**Key insight:** Long contexts in agent loops compound. A 20x compression on a 32k-token context saves more per call than any caching trick. Pair compression with caching for the largest wins.

## Self-Hosting and Serving Infrastructure

When API costs cross ~$5K/month for steady traffic, self-hosting often beats them. These tools change the economics.

**Inference engines:**

- [vLLM](https://github.com/vllm-project/vllm) - High-throughput LLM serving with PagedAttention. Industry standard for OSS serving.
- [SGLang](https://github.com/sgl-project/sglang) - RadixAttention prefix cache + FP4/FP8/INT4 quantization; powers ~400k GPUs in production.
- [TGI (Text Generation Inference)](https://github.com/huggingface/text-generation-inference) - HF's production server with continuous batching + AWQ/GPTQ/EETQ/fp8.
- [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) - NVIDIA's optimized kernels; 2–4x throughput vs vanilla on H100/H200 = direct $/token reduction.
- [llama.cpp](https://github.com/ggerganov/llama.cpp) - Pure C/C++ inference; the reference CPU/Metal/edge runtime.
- [llamafile](https://github.com/Mozilla-Ocho/llamafile) - Single-file LLM executable; deployment cost-of-ops near zero.
- [LocalAI](https://github.com/mudler/LocalAI) - Drop-in OpenAI-compatible API for local inference.

**Quantization:**

- [AutoAWQ](https://github.com/casper-hansen/AutoAWQ) - 4-bit weight quantization; cuts VRAM ~70%, runs 70B on a single A100.
- [ExLlamaV2](https://github.com/turboderp-org/exllamav2) - Fastest 4-bit consumer-GPU inference; 70B on 2x 3090s.
- [Unsloth](https://github.com/unslothai/unsloth) - 2x faster fine-tuning at 50% less VRAM; Dynamic v2.0 GGUFs preserve accuracy at 4-bit.

**Edge / browser:**

- [MLC LLM](https://github.com/mlc-ai/mlc-llm) - Compiles models to phones, browsers, edge. Eliminates server $ entirely for many use cases.
- [WebLLM](https://github.com/mlc-ai/web-llm) - In-browser inference via WebGPU; ~85% native Metal performance, $0 inference cost per user.

**Orchestration:**

- [SkyPilot](https://github.com/skypilot-org/skypilot) - Auto-routes workloads to the cheapest available cloud/region/spot across 20+ providers.
- [dstack](https://github.com/dstackai/dstack) - Open GPU orchestrator across clouds + on-prem; spot/preemptible scheduling.
- [Truss (BaseTen)](https://github.com/basetenlabs/truss) - Containerization standard for model serving; sub-second cold starts via cached layers.
- [BentoML / OpenLLM](https://github.com/bentoml/OpenLLM) - One-command OpenAI-compatible serving of any open model on any cloud.

**Key insight:** A 70B model that costs ~$15 per million output tokens via API runs at ~$0.50 per million on an idle H100 at spot price. The break-even is steady utilization. SkyPilot + spot instances + AWQ quantization shifts the curve hard.

## Embeddings and Vector Storage

Embedding and vector infra are some of the most overpaid line items in AI infrastructure.

**Vector storage:**

- [Amazon S3 Vectors](https://aws.amazon.com/s3/features/vectors/) - $0.06/GB vs Pinecone $0.33/GB vs Weaviate ~$25/GB. Up to 400x cheaper.
- [pgvector](https://github.com/pgvector/pgvector) - Open source vector extension for PostgreSQL (use your existing DB).
- [Chroma](https://github.com/chroma-core/chroma) - Open source embedding database.
- [SQLite-VSS](https://github.com/asg017/sqlite-vss) - SQLite extension for vector search (zero-cost for small datasets).
- [Qdrant](https://github.com/qdrant/qdrant) - Open source vector DB with on-premise option.
- [turbopuffer](https://turbopuffer.com/) - Object-storage-backed vector DB; powers Notion's 10x scale at 1/10th cost.

**Embedding cost reduction:**

- [Model2Vec](https://github.com/MinishLab/model2vec) - Static distilled embeddings ~50x smaller (~8–30MB) and ~500x faster than sentence-transformers on CPU.
- [FastEmbed](https://github.com/qdrant/fastembed) - Quantized ONNX embedding models; no PyTorch dependency, runs on CPU.
- [fastembed-rs](https://github.com/Anush008/fastembed-rs) - Rust port; even lower memory/cost for edge serving.
- [sentence-transformers quantization](https://github.com/UKPLab/sentence-transformers) - Built-in INT8/binary quantization; ~32x less memory at ~96% recall, 3x speedup.
- [Matryoshka embeddings (truncation)](https://huggingface.co/blog/matryoshka) - Truncate `text-embedding-3-*` to 256 dims; ~6x storage savings at marginal recall loss.

**Key insight:** Most teams don't need a hosted vector database. pgvector on your existing PostgreSQL handles millions of vectors. S3 Vectors handles billions at commodity storage prices. Stop paying Pinecone $70/mo for 10K vectors.

## Agentic Workflow Efficiency

Agents multiply costs because they loop. Efficient agent design is cost design.

- [agenttrace](https://github.com/luoyuctl/agenttrace) - Tracks AI coding-agent token, cost, latency, and failure regressions from local trace logs
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) - Agentic coding with cost tracking built in
- [LangSmith](https://smith.langchain.com/) - Trace and monitor agent costs per run
- [Braintrust](https://www.braintrust.dev/) - Eval and monitoring for LLM apps with cost tracking
- [AgentOps](https://github.com/AgentOps-AI/agentops) - Observability for AI agents (tracks cost per agent run)

**Key insight:** An unoptimized agent can burn $5-50 per run. Set token budgets, use smaller models for tool selection, and cache intermediate results. Measure cost per successful task completion, not cost per API call.

## Monitoring, Observability, and Gateways

You can't optimize what you don't measure. Gateways add a control plane in front of every model call.

**Observability:**

- [LiteLLM](https://github.com/BerriAI/litellm) - Proxy with per-key spend tracking, budgets, and rate limits
- [Helicone](https://helicone.ai/) - LLM observability with cost dashboards
- [Portkey](https://portkey.ai/) - AI gateway with cost tracking, caching, fallbacks
- [LangFuse](https://github.com/langfuse/langfuse) - Open source LLM observability and cost analytics
- [OpenMeter](https://openmeter.io/) - Usage metering for AI (track cost per customer/feature)
- [tokencost](https://github.com/AgentOps-AI/tokencost) - $ estimates for 400+ LLMs *before* you call. AgentOps.
- [traceAI](https://github.com/future-agi/traceAI) - Open-source OpenTelemetry-native tracing for LLM and agent apps with 50+ framework integrations; captures token and cost per span for downstream attribution.

**Gateways:**

- [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) - 350+ models, edge cache cuts repeat-query cost toward $0. Generous free tier.
- [Kong AI Gateway](https://github.com/Kong/kong) - AI plugins on Kong's OSS gateway; rate-limit, cost-budget, semantic cache.
- [Vercel AI Gateway](https://vercel.com/ai-gateway) - Sub-20ms routing across 100+ models.
- [Bifrost](https://github.com/maximhq/bifrost) - Newer OSS gateway by Maxim AI; 12+ providers, streaming-aware semantic cache.

## Pricing Comparisons

Side-by-side cost comparisons and calculators.

- [LLM Price Check](https://llmpricecheck.com/) - Compare LLM API prices across providers
- [Artificial Analysis](https://artificialanalysis.ai/) - Price/performance/speed benchmarks
- [Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/) - AWS Bedrock model pricing
- [Together AI Pricing](https://www.together.ai/pricing) - Open source model hosting pricing
- [Groq Pricing](https://groq.com/pricing/) - Fast inference pricing
- [TinyTools AI Cost Calculator](https://tinytools-smoky.vercel.app/ai-cost-calculator/) - Free browser-based calculator for estimating LLM API costs across providers, no signup, runs client-side
- [LiteLLM model_prices JSON](https://github.com/BerriAI/litellm/blob/main/model_prices_and_context_window.json) - Authoritative price table used by most cost tools.

## Patterns and Snippets

Concrete, copy-pasteable techniques. Each snippet is the smallest thing that moves cost meaningfully.

### Anthropic prompt caching

Reads are billed at 10% of input price. ~90% savings on repeated long system prompts.

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

## Case Studies

Real-world examples of AI cost optimization, with concrete numbers.

- [Two Years of Vector Search at Notion](https://www.notion.com/blog/two-years-of-vector-search-at-notion) - Notion Engineering. Migrated embeddings to Ray + turbopuffer: **90% reduction** in embedding infra costs at 10x scale.
- [How ProjectDiscovery Cut LLM Costs by 59% With Prompt Caching](https://projectdiscovery.io/blog/how-we-cut-llm-cost-with-prompt-caching) - Cache hit rate climbed from 7% → 84% on Opus 4.5 agents consuming ~60M tokens per task.
- [From $720 to $72/month on API costs](https://labeveryday.medium.com/prompt-caching-is-a-must-how-i-went-from-spending-720-to-72-monthly-on-api-costs-3086f3635d63) - Concrete per-request math on caching: $0.24 → $0.024 per call (90% reduction) with an 81k-token system prompt.
- [Anthropic Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) - Contextualizing chunks across a 500-page PDF = $6.30 total with prompt caching. 49–67% drop in retrieval failures.
- [RouteLLM benchmark results](https://lmsys.org/blog/2024-07-01-routellm/) - 85% cost reduction on MT-Bench, 45% on MMLU, 35% on GSM8K, holding 95% of GPT-4 quality.
- LG CNS: 50% faster security testing, ~30% lower costs with AWS Security Agent.
- WGU: Incident resolution dropped from hours to minutes with AWS DevOps Agent.
- The $20-is-the-new-t2.micro analysis: AI subscription tiers mirror cloud compute pricing (burstable vs committed).

*Have a case study? [Submit a PR.](#contributing)*

## Articles, Papers, and Talks

- [Everyone Needs an OpenClaw Strategy](https://www.youtube.com/@CloudYeti) - Jensen Huang said it. Here's what he actually meant: 24/7 agents need a cost strategy, not just a framework. (CloudYeti)
- [The token-to-revenue ratio](https://cloudyeti.io/blog) - Measuring AI spend against business output (CloudYeti)
- [S3 Vectors: 400x cheaper vector search](https://www.youtube.com/watch?v=BWURf-oVFfg) - RAG with S3 Vectors + Bedrock (CloudYeti)
- [Patterns for Building LLM-based Systems & Products](https://eugeneyan.com/writing/llm-patterns/) - Eugene Yan. Reframes caching as "shifting from LLM generation (dollars) to cache storage (cents)." Best single overview of the seven cost+quality patterns.
- [FrugalGPT (paper)](https://arxiv.org/abs/2305.05176) - Chen, Zaharia, Zou (Stanford). The foundational LLM-cascade paper. Required reading before building any router.
- [LLMLingua: Innovating LLM efficiency with prompt compression](https://www.microsoft.com/en-us/research/blog/llmlingua-innovating-llm-efficiency-with-prompt-compression/) - Microsoft Research. 20x compression with minimal accuracy loss.
- [Is Fine-Tuning Still Valuable?](https://hamel.dev/blog/posts/fine_tuning_valuable.html) - Hamel Husain. Fine-tune only after evals + prompting hit a wall; otherwise you're spending training $ to solve a routing problem.
- [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437) - 671B-param frontier model trained for $5.576M / 2.788M H800 GPU-hours. The upper bound on what "frontier" actually has to cost.
- [The Economics of $20K/month AI Agents](https://medium.com/@mcunningham1440/the-economics-of-openais-20000-month-ai-agents-26b329f301c4) - Daily rate analysis of always-on agents.
- [The True Cost of AI Agents: Hourly Pricing](https://retool.com/blog/cost-of-ai-agents-hourly-pricing-model) - Retool's agent cost framework.

*More articles coming. Follow [@CloudYeti](https://www.youtube.com/@CloudYeti) for updates.*

## Related Lists

Other curated lists adjacent to this one. Pull from them for deeper coverage of specific niches.

- [Awesome-LLMOps](https://github.com/tensorchord/Awesome-LLMOps) - Broadest LLMOps tooling list.
- [Awesome-LLM-Inference](https://github.com/xlite-dev/Awesome-LLM-Inference) - Inference papers + code (FlashAttention, PagedAttention, INT4/8).
- [Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM) - Efficient-LLM research index; well-maintained.
- [Awesome-LLM-Compression](https://github.com/HuangOwen/Awesome-LLM-Compression) - Quantization, pruning, distillation, prompt compression.
- [Awesome-AI-Model-Routing](https://github.com/Not-Diamond/awesome-ai-model-routing) - The canonical routing list.

## Tools

Open source tools specifically built for AI cost management.

| Tool | What It Does | License |
|------|-------------|---------|
| [LiteLLM](https://github.com/BerriAI/litellm) | Unified LLM proxy with spend tracking | MIT |
| [OpenRouter](https://openrouter.ai/) | Multi-model API with price routing | Commercial |
| [GPTCache](https://github.com/gptcache/gptcache) | Semantic cache for LLM responses | MIT |
| [Ollama](https://ollama.com/) | Local model inference | MIT |
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput serving | Apache 2.0 |
| [SGLang](https://github.com/sgl-project/sglang) | RadixAttention serving + FP4/FP8/INT4 quant | Apache 2.0 |
| [LangFuse](https://github.com/langfuse/langfuse) | Open source LLM cost analytics | MIT |
| [QuotaFlow](https://quotaflow.ai/) | Governed AI quota pooling to reduce wasted subscribed capacity | Commercial |
| [promptfoo](https://github.com/promptfoo/promptfoo) | Eval prompts/models on your data with cost matrix | MIT |
| [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) | Eval framework for LLMs and agents | MIT |
| [LLMLingua](https://github.com/microsoft/LLMLingua) | Prompt compression up to 20x | MIT |
| [tokencost](https://github.com/AgentOps-AI/tokencost) | Pre-call $ estimation for 400+ LLMs | MIT |
| [SkyPilot](https://github.com/skypilot-org/skypilot) | Cheapest-cloud GPU orchestrator | Apache 2.0 |
| [FastEmbed](https://github.com/qdrant/fastembed) | CPU-friendly quantized embeddings | Apache 2.0 |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

The bar: every link must help someone reduce their AI costs or measure their AI spend more effectively. No vendor marketing pages. No "10 tips" listicles. Tools, benchmarks, techniques, and real case studies only.

---

## About

Maintained by [Saurav Sharma](https://linkedin.com/in/saurav-sharma-cloud) — ex-Amazon SDE, 13x AWS certified. I help teams use AI without wasting money.

- Workshops and bootcamps: [cloudyeti.io/catalog](https://cloudyeti.io/catalog)
- Book a discovery call: [cloudyeti.io/chat](https://cloudyeti.io/chat)
- YouTube: [@CloudYeti](https://www.youtube.com/@CloudYeti)

---

## License

[CC0 1.0 Universal](LICENSE) — Public domain. Use however you want.
