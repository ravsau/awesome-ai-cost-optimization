# Low-Hanging Fruit

[Playbook](00-start-here.md) · Previous: [Must-Dos](01-must-dos.md) · Next: [Big Levers](03-big-levers.md)

Ships in days, low risk. Prices verified June 2026 against official pricing pages.

## 1. Prompt caching

| Provider | How | Numbers |
|---|---|---|
| [Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) | Opt-in `cache_control` breakpoints, prefix-matched | Reads 0.1× input. **Writes cost extra:** 1.25× (5-min TTL) or 2× (1-hour TTL) |
| [OpenAI](https://developers.openai.com/api/docs/guides/prompt-caching) | Automatic, prompts ≥1,024 tokens | Cached input = 10% of input price, no write cost |
| [Gemini](https://ai.google.dev/gemini-api/docs/caching) | Implicit by default (2.5+) | Cached ~10% of input; explicit adds $1.00/M tokens/hour storage |

Traps: caching is prefix-matched — a timestamp at the top of the system prompt means zero hits. And on Anthropic, an unread cache costs more than no cache; check `cache_read_input_tokens` is non-zero.

## 2. Batch what can wait

Same tokens, 50% off the on-demand token price, all three providers. Anything on a cron qualifies: evals, embedding backfills, nightly jobs.

- [Anthropic Message Batches](https://platform.claude.com/docs/en/build-with-claude/batch-processing) — stacks with caching.
- [OpenAI Batch](https://developers.openai.com/api/docs/guides/batch) — with caching, ~75% off repeated prompts.
- [OpenAI Flex](https://developers.openai.com/api/docs/guides/flex-processing) — batch pricing on sync calls, beta.
- [Gemini batch mode](https://ai.google.dev/gemini-api/docs/pricing).
- Inverse exists: [OpenAI priority processing](https://openai.com/api-priority-processing/) is ~2–2.5× standard — don't pay it by accident.

## 3. Downshift by task

Cheap tiers, per million tokens (input/output): Gemini 2.5 Flash-Lite $0.10/$0.40 · gpt-5.4-nano $0.20/$1.25 · gpt-5.4-mini $0.75/$4.50 · Claude Haiku 4.5 $1/$5. The flagship gap is 25–50×.

Route classification/extraction/summarization to the cheap tier. Gate every downgrade with the golden set from [Must-Dos](01-must-dos.md).

## 4. Stop paying for unused tokens

- Tool definitions bill as input on **every call**, plus a 290–675 token hidden system prompt ([docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)). Delete tools the model never calls.
- Structured outputs kill retry-on-bad-parse — each retry is a full-price request ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [OpenAI](https://developers.openai.com/api/docs/guides/structured-outputs)).
- Set `max_tokens`; use stop sequences.
- [`count_tokens`](https://platform.claude.com/docs/en/build-with-claude/token-counting) is free — measure the system prompt before shipping it.

## 5. Dev/test hygiene

Surprise bills often come from dev, not prod (a retry loop in CI over a weekend).

- [vcrpy](https://github.com/kevin1024/vcrpy) — record once, replay in CI.
- [llmock](https://github.com/CopilotKit/llmock) / [mockllm](https://github.com/StacklokLabs/mockllm) — mock LLM servers.
- [LiteLLM per-key budgets](https://docs.litellm.ai/docs/proxy/users) — caps the blast radius of any runaway script.

## Checklist

- [ ] Stable content first in long prompts; cache reads confirmed non-zero
- [ ] Cron jobs on batch API
- [ ] One task downshifted, eval-gated
- [ ] Unused tools deleted; `max_tokens` set
- [ ] CI mocked; budgets on dev keys

Next: [Big Levers →](03-big-levers.md)
