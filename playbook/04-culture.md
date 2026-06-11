# Culture

[Playbook](00-start-here.md) · Previous: [Big Levers](03-big-levers.md) · Next: [Audit Checklist](05-audit-checklist.md)

Technical fixes regress; these are the habits that hold them.

## Ownership

Cost visibility belongs with the engineers who can change it, not just finance. 78% of FinOps practices now report into the CTO/CIO org ([FinOps Foundation](https://www.finops.org/wg/finops-for-ai-overview/)).

- [Ramp: token spend up 13× since Jan 2025 across their customers](https://ramp.com/blog/trillion-dollar-ai-blindspot) · [how their engineers built attribution](https://builders.ramp.com/post/ai-token-spend-management)
- [Geoff Charles on Ramp's AI rollout](https://creatoreconomy.so/p/inside-ramp-the-32b-company-ai-agents-geoff-charles) — cost governance built into adoption, not bolted on.

## Caps and seats

- [Uber](https://www.washingtontimes.com/news/2026/jun/3/uber-capping-internal-use-ai-coding-software-blowing-budget/) — annual AI coding budget gone in 4 months; now $1,500/mo cap per tool per engineer.
- [Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/the-impact-of-ai-on-software-engineers-2026) — typical spend: $100–200/engineer/mo on max-tier coding agents; budget orgs at $20 Copilot tiers.
- [Ramp AI Index](https://thenextweb.com/news/ai-pilled-firms-7500-per-employee-spending) — top 1% of firms: $7,500/employee/mo; median: $11.

## Postmortems worth reading

Common thread: monitoring without enforcement — dashboards but no gate.

- [$47K in 11 days](https://dev.to/dingdawg/how-an-ai-agent-ran-up-a-47000-bill-in-11-days-and-how-to-stop-it-1fk) — agents in an infinite clarification loop; a payload-hash check would have caught it on iteration two.
- [$4,200 in 63 hours](https://medium.com/@sattyamjain96/the-agent-that-burned-4-200-in-63-hours-a-production-ai-postmortem-d38fd9586a85) — retry loop, ~4,800 calls/hour, weekend, nobody on call.
- [$48K in 14 hours](https://www.truefoundry.com/blog/llm-cost-attribution-agentic-cicd) — one session, no per-session budget, retries compounding context.
- [Ramp Labs: agents ignore budgets written into their prompts](https://x.com/RampLabs/status/2046642262042657176) — enforce in the gateway, not the prompt.

## Token budgeting

- [Traceloop](https://www.traceloop.com/blog/from-bills-to-budgets-how-to-track-llm-token-usage-and-cost-per-user) — budgets per user/feature/team, not per invoice.
- [TrueFoundry](https://www.truefoundry.com/blog/llm-cost-attribution-team-budgets) — chargeback down to team/agent/model.

## Case studies with numbers

| Who | What | Number | Source |
|---|---|---|---|
| Notion | turbopuffer + serverless | 10× scale at 1/10th cost | [blog](https://www.notion.com/blog/two-years-of-vector-search-at-notion) |
| ProjectDiscovery | prompt caching | 59% cut at launch, ~70% now | [blog](https://projectdiscovery.io/blog/how-we-cut-llm-cost-with-prompt-caching) |
| Thomson Reuters Labs | prompt caching | 60% cut; $0.34 → $0.14/request | [blog](https://medium.com/tr-labs-ml-engineering-blog/prompt-caching-the-secret-to-60-cost-reduction-in-llm-applications-6c792a0ac29b) |
| LMSYS/Canva | RouteLLM routing | 85% cut at 95% GPT-4 quality | [blog](https://lmsys.org/blog/2024-07-01-routellm/) |
| Klarna | AI support assistant | 2.3M chats month one; later re-added humans | [press](https://www.klarna.com/international/press/klarna-ai-assistant-handles-two-thirds-of-customer-service-chats-in-its-first-month/) |
| Personal (81K-token prompt) | prompt caching | $720 → $72/mo | [Medium](https://labeveryday.medium.com/prompt-caching-is-a-must-how-i-went-from-spending-720-to-72-monthly-on-api-costs-3086f3635d63) |

Next: [Audit Checklist →](05-audit-checklist.md)
