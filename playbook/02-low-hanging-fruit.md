# Low-Hanging Fruit: The First Week

Part of the [AI Cost Optimization Playbook](00-start-here.md). Previous: [Must-Dos](01-must-dos.md) · Next: [Big Levers](03-big-levers.md)

---

Everything on this page ships in under a week, carries near-zero risk, and routinely cuts a bill 30–70%. If you do nothing else from this repo, do this page.

All prices verified June 2026 against the official pricing pages. Prices move; the patterns don't.

## 1. Prompt caching

The same long system prompt sent a million times should not bill at full price a million times.

| Provider | How it works | The numbers |
|---|---|---|
| Anthropic | Opt-in: you set `cache_control` breakpoints. Prefix-matched. | Cache reads bill at 0.1× the input price. Cache **writes cost extra**: 1.25× input for the 5-minute TTL, 2× for the 1-hour TTL. |
| OpenAI | Automatic on any prompt ≥1,024 tokens. No code change. | Cached input tokens bill at 10% of the input price. No write surcharge. Entries live ~5–10 minutes. |
| Google Gemini | Implicit caching on by default (2.5+ models); explicit caching adds a guarantee. | Cached tokens ~10% of input price; explicit caching adds $1.00 per million tokens per hour of storage. |

- [Anthropic prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [OpenAI prompt caching docs](https://developers.openai.com/api/docs/guides/prompt-caching)
- [Gemini context caching docs](https://ai.google.dev/gemini-api/docs/caching)

Two traps worth knowing:

1. **One timestamp kills the cache.** Caching is prefix-matched. A `Today is {date}` line at the top of your system prompt means zero cache hits forever. Stable content first, variable content last.
2. **On Anthropic, an unread cache costs more than no cache.** You pay 1.25–2× to write. Check `cache_read_input_tokens` in your responses — if it's zero, you're paying a surcharge for nothing.

## 2. Batch everything that can wait

Every major provider sells the same tokens at **50% off the on-demand token price** if you can wait hours instead of seconds. Evals, embedding backfills, nightly summaries, report generation — if it runs on a cron, it has no business on the synchronous API.

- [Anthropic Message Batches](https://platform.claude.com/docs/en/build-with-claude/batch-processing) — 50% off all token usage, up to 100K requests per batch, most finish under an hour. Stacks with prompt caching.
- [OpenAI Batch API](https://developers.openai.com/api/docs/guides/batch) — flat 50% off every model, 24-hour window. Stacked with caching, repeated prompts come out ~75% off.
- [OpenAI Flex processing](https://developers.openai.com/api/docs/guides/flex-processing) — batch-level pricing on *synchronous* requests; slower and may return "resource unavailable," but no batch-file plumbing. Beta.
- [Gemini batch mode](https://ai.google.dev/gemini-api/docs/pricing) — 50% off all paid models.
- Know the inverse exists so you don't pay it by accident: [OpenAI priority processing](https://openai.com/api-priority-processing/) charges roughly 2–2.5× the standard rate for lower latency.

## 3. Downshift the model, task by task

Nobody needs the flagship model to classify a support ticket. The price gap between tiers is now 25–50×, so even rough routing pays for itself. Current cheap tiers (per million tokens, input / output):

| Model | Input | Output |
|---|---|---|
| Gemini 2.5 Flash-Lite | $0.10 | $0.40 |
| gpt-5.4-nano | $0.20 | $1.25 |
| gpt-5.4-mini | $0.75 | $4.50 |
| Claude Haiku 4.5 | $1.00 | $5.00 |

Sources: [OpenAI pricing](https://developers.openai.com/api/docs/pricing), [Claude models overview](https://platform.claude.com/docs/en/about-claude/models/overview), [Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing).

The discipline: downshift **by task, not by app**. Route classification, extraction, and summarization to the cheap tier; keep the frontier model for the 20% of calls that genuinely need it. Prove the downgrade with the golden set from [Must-Dos](01-must-dos.md) before shipping it.

## 4. Stop paying for tokens you don't use

- **Your tool list is a subscription you pay per request.** Tool definitions bill as input tokens on every single call, used or not, plus a hidden tool-use system prompt of 290–675 tokens. Twenty tools at 200 tokens each is 4,000 tokens of overhead on every request. Delete the tools the model never calls — your logs will tell you which. ([Anthropic tool-use docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview))
- **Structured outputs kill the retry loop.** Every retry on a bad JSON parse is a full-price request. Schema-enforced output makes the first answer parse. ([Anthropic structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [OpenAI structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs))
- **Set `max_tokens` deliberately and use stop sequences.** An uncapped generation on an output-priced model is an open tab.
- **Measure before you ship.** Anthropic's [`count_tokens` endpoint](https://platform.claude.com/docs/en/build-with-claude/token-counting) is free. Know what your system prompt costs before sending it a million times.

## 5. Dev and test hygiene

The biggest surprise bills come from development, not production. A retry loop in CI hitting a flagship model all weekend beats any production workload.

- [vcrpy](https://github.com/kevin1024/vcrpy) — record real API interactions once, replay them in every CI run. Test cost drops to roughly zero.
- [llmock](https://github.com/CopilotKit/llmock) — deterministic mock LLM server with streaming and record/replay.
- [mockllm](https://github.com/StacklokLabs/mockllm) — YAML-configured mock server mimicking OpenAI/Anthropic wire formats.
- [Ollama](https://github.com/ollama/ollama) — run open models locally for dev loops and smoke tests. $0 per million tokens.
- [LiteLLM per-key budgets](https://docs.litellm.ai/docs/proxy/users) — a `max_budget` on every dev key caps the blast radius of any runaway script, forever, for an afternoon of setup.

---

## The first-week checklist

- [ ] Stable content moved to the front of every long prompt; caching enabled; `cache_read_input_tokens` confirmed non-zero
- [ ] Every cron-driven LLM job moved to the batch API
- [ ] One task downshifted to a cheap-tier model, gated by a golden-set eval
- [ ] Unused tool definitions deleted; `max_tokens` set everywhere
- [ ] CI mocked or recorded; per-key budgets on every dev key

Next: [Big Levers →](03-big-levers.md)
