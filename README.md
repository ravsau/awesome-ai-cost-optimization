# Awesome AI Cost Optimization

A curated list of tools, techniques, benchmarks, and resources for reducing AI infrastructure costs without reducing AI output.

Most teams overspend on AI inference, underuse open source models, and run agentic workflows at a fraction of their potential. This repo is for engineers and leaders who want to fix that.

---

## Contents

- [Model Selection and Routing](#model-selection-and-routing)
- [Evals and Model Comparison](#evals-and-model-comparison)
- [Open Source Models](#open-source-models)
- [Inference Optimization](#inference-optimization)
- [Vector Storage and RAG Cost](#vector-storage-and-rag-cost)
- [Agentic Workflow Efficiency](#agentic-workflow-efficiency)
- [Monitoring and Observability](#monitoring-and-observability)
- [Pricing Comparisons](#pricing-comparisons)
- [Case Studies](#case-studies)
- [Articles and Blog Posts](#articles-and-blog-posts)
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
- [RouteLLM](https://github.com/lm-sys/RouteLLM) - Framework for serving and evaluating LLM routers (from LMSYS)

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

**Key insight:** "Cheaper model" is a guess until you eval it. Build a 50-100 example dataset of your real workload, run promptfoo against 4-5 candidate models, and pick the cheapest one that clears your quality bar. Most teams skip this step and overpay forever.

## Open Source Models

Open source can handle 80% of what proprietary APIs do, at a fraction of the cost.

- [Ollama](https://ollama.com/) - Run open source models locally with one command
- [vLLM](https://github.com/vllm-project/vllm) - High-throughput LLM serving engine
- [llama.cpp](https://github.com/ggerganov/llama.cpp) - Inference of Meta's LLaMA model in pure C/C++
- [LocalAI](https://github.com/mudler/LocalAI) - Drop-in OpenAI-compatible API for local inference
- [Qwen 3.5](https://huggingface.co/Qwen) - "Towards Native Multimodal Agents." Outperforms GPT-5 mini on function calling by 30%. Small versions (0.8B-9B) run on laptops. The before-and-after moment for local AI agents.
- [Llama 4](https://llama.meta.com/) - Meta's open source model family
- [Mistral](https://mistral.ai/) - European open source models, strong multilingual
- [DeepSeek](https://github.com/deepseek-ai) - Cost-efficient open source models from China
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

**Key insight:** Prompt caching alone can cut costs 50-90% for applications with repeated system prompts. If you're not using it, you're leaving money on the table.

## Vector Storage and RAG Cost

Vector databases are one of the most overpaid line items in AI infrastructure.

- [Amazon S3 Vectors](https://aws.amazon.com/s3/features/vectors/) - $0.06/GB vs Pinecone $0.33/GB vs Weaviate ~$25/GB. Up to 400x cheaper.
- [pgvector](https://github.com/pgvector/pgvector) - Open source vector extension for PostgreSQL (use your existing DB)
- [Chroma](https://github.com/chroma-core/chroma) - Open source embedding database
- [SQLite-VSS](https://github.com/asg017/sqlite-vss) - SQLite extension for vector search (zero-cost for small datasets)
- [Qdrant](https://github.com/qdrant/qdrant) - Open source vector DB with on-premise option

**Key insight:** Most teams don't need a hosted vector database. pgvector on your existing PostgreSQL handles millions of vectors. S3 Vectors handles billions at commodity storage prices. Stop paying Pinecone $70/mo for 10K vectors.

## Agentic Workflow Efficiency

Agents multiply costs because they loop. Efficient agent design is cost design.

- [agenttrace](https://github.com/luoyuctl/agenttrace) - Tracks AI coding-agent token, cost, latency, and failure regressions from local trace logs
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) - Agentic coding with cost tracking built in
- [LangSmith](https://smith.langchain.com/) - Trace and monitor agent costs per run
- [Braintrust](https://www.braintrust.dev/) - Eval and monitoring for LLM apps with cost tracking
- [AgentOps](https://github.com/AgentOps-AI/agentops) - Observability for AI agents (tracks cost per agent run)

**Key insight:** An unoptimized agent can burn $5-50 per run. Set token budgets, use smaller models for tool selection, and cache intermediate results. Measure cost per successful task completion, not cost per API call.

## Monitoring and Observability

You can't optimize what you don't measure.

- [LiteLLM](https://github.com/BerriAI/litellm) - Proxy with per-key spend tracking, budgets, and rate limits
- [Helicone](https://helicone.ai/) - LLM observability with cost dashboards
- [Portkey](https://portkey.ai/) - AI gateway with cost tracking, caching, fallbacks
- [LangFuse](https://github.com/langfuse/langfuse) - Open source LLM observability and cost analytics
- [OpenMeter](https://openmeter.io/) - Usage metering for AI (track cost per customer/feature)

## Pricing Comparisons

Side-by-side cost comparisons and calculators.

- [LLM Price Check](https://llmpricecheck.com/) - Compare LLM API prices across providers
- [Artificial Analysis](https://artificialanalysis.ai/) - Price/performance/speed benchmarks
- [Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/) - AWS Bedrock model pricing
- [Together AI Pricing](https://www.together.ai/pricing) - Open source model hosting pricing
- [Groq Pricing](https://groq.com/pricing/) - Fast inference pricing

## Case Studies

Real-world examples of AI cost optimization.

- LG CNS: 50% faster security testing, ~30% lower costs with AWS Security Agent
- WGU: Incident resolution dropped from hours to minutes with AWS DevOps Agent
- The $20-is-the-new-t2.micro analysis: AI subscription tiers mirror cloud compute pricing (burstable vs committed)

*Have a case study? [Submit a PR.](#contributing)*

## Articles and Blog Posts

- [Everyone Needs an OpenClaw Strategy](https://www.youtube.com/@CloudYeti) - Jensen Huang said it. Here's what he actually meant: 24/7 agents need a cost strategy, not just a framework. (CloudYeti)
- [The token-to-revenue ratio](https://cloudyeti.io/blog) - Measuring AI spend against business output (CloudYeti)
- [S3 Vectors: 400x cheaper vector search](https://www.youtube.com/watch?v=BWURf-oVFfg) - RAG with S3 Vectors + Bedrock (CloudYeti)
- [The Economics of $20K/month AI Agents](https://medium.com/@mcunningham1440/the-economics-of-openais-20000-month-ai-agents-26b329f301c4) - Daily rate analysis of always-on agents
- [The True Cost of AI Agents: Hourly Pricing](https://retool.com/blog/cost-of-ai-agents-hourly-pricing-model) - Retool's agent cost framework

*More articles coming. Follow [@CloudYeti](https://www.youtube.com/@CloudYeti) for updates.*

## Tools

Open source tools specifically built for AI cost management.

| Tool | What It Does | License |
|------|-------------|---------|
| [LiteLLM](https://github.com/BerriAI/litellm) | Unified LLM proxy with spend tracking | MIT |
| [OpenRouter](https://openrouter.ai/) | Multi-model API with price routing | Commercial |
| [GPTCache](https://github.com/gptcache/gptcache) | Semantic cache for LLM responses | MIT |
| [Ollama](https://ollama.com/) | Local model inference | MIT |
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput serving | Apache 2.0 |
| [LangFuse](https://github.com/langfuse/langfuse) | Open source LLM cost analytics | MIT |
| [promptfoo](https://github.com/promptfoo/promptfoo) | Eval prompts/models on your data with cost matrix | MIT |
| [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) | Eval framework for LLMs and agents | MIT |

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
