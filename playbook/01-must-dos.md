# Must-Dos: Measure Before You Optimize

Part of the [AI Cost Optimization Playbook](00-start-here.md). Previous: [Start Here](00-start-here.md) · Next: [Low-Hanging Fruit](02-low-hanging-fruit.md)

---

Every failed cost-cutting effort I've seen failed the same way: the team tried to optimize a number they couldn't see. They knew the monthly invoice total and nothing else. So they guessed, switched a model, broke quality somewhere, and rolled it all back.

This page is the layer you build before touching anything else. None of it saves money directly. All of it is what makes saving money possible — and safe.

## 1. Route everything through one gateway

Not for fallbacks. Not for caching. For the receipts.

If API keys are scattered across laptops, Lambda functions, and three different services, you have no spend data to optimize. A gateway gives every request a paper trail: who called, which model, how many tokens, what it cost.

- [LiteLLM](https://github.com/BerriAI/litellm) — the default self-hosted pick. 100+ providers, per-key budgets in Postgres. Known to strain around ~2K requests/sec; fine for almost everyone.
- [Portkey Gateway](https://github.com/Portkey-AI/gateway) — went fully open source (Apache 2.0) in March 2026. Governance, cost controls, and MCP gateway included.
- [Bifrost](https://github.com/maximhq/bifrost) — Go gateway, under 100µs overhead at 5K req/sec. The pick when Python proxies become the bottleneck.
- [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) — managed, generous free tier, takes an afternoon to adopt.
- [Helicone](https://github.com/Helicone/helicone) — open source observability plus a Rust gateway, one-line integration.

An afternoon of work. Do it first.

## 2. Hard caps on every key

A budget without enforcement is a hope. Every API key gets a monthly dollar cap and a model allow-list **before it ships**. One agent stuck in a loop on a flagship model can burn a month's budget overnight. This is now table stakes in every gateway:

- [LiteLLM budgets and rate limits](https://docs.litellm.ai/docs/proxy/users) — per-key, per-user, per-team caps; hard caps return 429, soft caps fire a Slack alert.
- [Cloudflare AI Gateway spend limits](https://blog.cloudflare.com/ai-gateway-spend-limits/) — shipped June 2026: dollar budgets per model, provider, or custom attribute, with the option to downgrade to a cheaper model instead of blocking when the cap is hit.
- [LiteLLM agent iteration budgets](https://docs.litellm.ai/docs/a2a_iteration_budgets) — caps agent loop iterations; the closest thing to a native runaway-agent kill switch today.

## 3. Tag every request with team and feature

You cannot retrofit attribution. Two extra metadata fields on every request — who, and which feature — is the difference between "we spent $40K on AI last month" and "the support summarizer costs $0.11 per ticket."

The FinOps Foundation now treats AI as a formal category and their sequencing is right: **showback first, chargeback only once the numbers are trusted.**

- [FinOps for AI (FinOps Foundation)](https://www.finops.org/wg/finops-for-ai-overview/) — the official framework. Cost-per-token as a first-class KPI.
- [CloudZero: FinOps for AI](https://www.cloudzero.com/blog/finops-for-ai/) — why traditional tagging breaks for LLMs (an API call is a transaction, not a taggable asset) and how to capture allocation at the gateway layer instead.
- [Finout: FinOps for AI agents](https://www.finout.io/blog/finops-for-ai-agents-a-four-step-allocation-framework) — a four-step allocation framework for agentic workloads.

## 4. Build a golden set before your first model downgrade

You don't need an eval suite on day one. You need it before the first time you want to swap a model for a cheaper one. Without a baseline, "switch to the cheaper model" is a quality gamble, not a decision.

The workflow: pull 50–100 real examples from production traces → score your current model as the baseline → run the cheaper candidate against the same set → ship only if nothing critical regresses.

- [promptfoo](https://github.com/promptfoo/promptfoo) — declarative YAML evals, ~15 minutes to wire into GitHub Actions. Used by OpenAI and Anthropic themselves.
- [DeepEval](https://github.com/confident-ai/deepeval) — pytest-style, 50+ metrics, built for CI gating.
- [Inspect](https://inspect.aisi.org.uk/) — UK AI Security Institute's framework; 200+ pre-built evals, strong for agents.
- [Braintrust](https://www.braintrust.dev/docs/evaluate) — commercial; the "block on regression, not on absolute threshold" workflow with baseline history.

## 5. Measure cost per completed task, not per API call

Agentic systems make request counts meaningless — one task might be one call or forty. The metric that matters is dollars per successful outcome: per resolved ticket, per merged PR, per generated report.

- [ccusage](https://github.com/ryoppippi/ccusage) — CLI that reads local data for Claude Code, Codex, Gemini CLI and a dozen more. The de facto standard for individual developers.
- [Claude Code monitoring docs](https://code.claude.com/docs/en/monitoring-usage) — Claude Code exports OpenTelemetry metrics natively: tokens, cost, cache efficiency, broken down by user, model, and subagent.
- [claude-code-otel](https://github.com/ColeMurray/claude-code-otel) — ready-made OTel Collector + Prometheus + Grafana stack for team-level agent spend.
- [SigNoz Claude Code monitoring guide](https://signoz.io/docs/claude-code-monitoring/) — turnkey OTel-to-dashboard setup.

## When you're selling AI, add metering

Full metering and billing platforms are a nice-to-have for internal tools but become mandatory the moment AI cost is your cost of goods sold and you need per-customer margin:

- [OpenMeter](https://github.com/openmeterio/openmeter) — open source usage metering with first-class token metering.
- [Orb](https://www.withorb.com/) — usage-based billing engine; metrics defined in SQL.
- Signal worth knowing: Stripe acquired Metronome — the metering layer behind OpenAI and Anthropic's own billing — for roughly $1B in January 2026. Usage metering is now core payments infrastructure.

---

## The order of operations

1. Gateway (afternoon)
2. Hard caps on every key (hour)
3. Team + feature tags on every request (day)
4. Golden set from production traces (before the first downgrade)
5. Cost per completed task as the reported metric (ongoing)

Do these five things and every page that follows in this playbook becomes executable. Skip them and the rest is guesswork.

Next: [Low-Hanging Fruit →](02-low-hanging-fruit.md)
