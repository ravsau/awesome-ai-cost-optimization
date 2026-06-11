# Must-Dos: Measure First

[Playbook](00-start-here.md) · Next: [Low-Hanging Fruit](02-low-hanging-fruit.md)

None of this saves money directly; it's what makes optimization possible and safe to do.

## 1. One gateway for all LLM traffic

Gives every request a record: who, which model, tokens, cost. An afternoon of work.

- [LiteLLM](https://github.com/BerriAI/litellm) — default self-hosted pick; per-key budgets in Postgres.
- [Portkey Gateway](https://github.com/Portkey-AI/gateway) — fully Apache 2.0 since March 2026.
- [Bifrost](https://github.com/maximhq/bifrost) — Go, <100µs overhead; for when Python proxies bottleneck.
- [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/) — managed, free tier.
- [Helicone](https://github.com/Helicone/helicone) — observability + Rust gateway, one-line integration.

## 2. Hard caps on every key

Monthly dollar cap + model allow-list before a key ships. One looping agent on a flagship model can burn a month's budget overnight.

- [LiteLLM budgets](https://docs.litellm.ai/docs/proxy/users) — hard caps 429, soft caps alert.
- [Cloudflare spend limits](https://blog.cloudflare.com/ai-gateway-spend-limits/) — June 2026: dollar budgets per model/provider/attribute; can downgrade to a cheaper model instead of blocking.
- [LiteLLM iteration budgets](https://docs.litellm.ai/docs/a2a_iteration_budgets) — step caps for agent loops.

## 3. Tag requests with team + feature

Two metadata fields turn "we spent $40K on AI" into "the support summarizer costs $0.11/ticket." Attribution can't be retrofitted. Showback before chargeback ([FinOps Foundation](https://www.finops.org/wg/finops-for-ai-overview/), [CloudZero](https://www.cloudzero.com/blog/finops-for-ai/)).

## 4. Golden eval set before the first model downgrade

50–100 real examples from production traces → baseline the current model → run the cheaper one → ship only if nothing critical regresses.

- [promptfoo](https://github.com/promptfoo/promptfoo) — ~15 min into CI.
- [DeepEval](https://github.com/confident-ai/deepeval) — pytest-style.
- [Braintrust](https://www.braintrust.dev/docs/evaluate) — block-on-regression workflow.

## 5. Track cost per completed task, not per call

Agent loops make request counts meaningless.

- [ccusage](https://github.com/ryoppippi/ccusage) — local spend reports for Claude Code, Codex, Gemini CLI.
- [Claude Code OTel export](https://code.claude.com/docs/en/monitoring-usage) — tokens/cost by user, model, subagent.
- [claude-code-otel](https://github.com/ColeMurray/claude-code-otel) — ready-made Grafana stack.

When AI cost is COGS and you need per-customer margin: [OpenMeter](https://github.com/openmeterio/openmeter), [Orb](https://www.withorb.com/).

Next: [Low-Hanging Fruit →](02-low-hanging-fruit.md)
