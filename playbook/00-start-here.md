# Start Here: How to Think About AI Cost

This is the playbook half of [awesome-ai-cost-optimization](../README.md) — the part that tells you what to do and in what order, rather than just what exists.

---

## The one idea behind all of it

AI is rented intelligence. Every token you buy has to convert into output, and that output has to convert into revenue or a clean customer outcome. The question is never "how do we spend less on AI" — it's "how much output are we getting per dollar of tokens, and where is that ratio broken?"

Cutting the bill of a feature that drives revenue is a mistake. Paying flagship prices for a task a model 25× cheaper handles fine is also a mistake. You can't tell the two apart without measurement, which is why this playbook starts there.

## The order of operations

Most teams do this backwards. They start with the exciting lever (self-hosting! fine-tuning!) and skip the boring layer that makes any of it safe. The order that works:

### [1. Must-Dos](01-must-dos.md) — *before anything else*
One gateway for receipts. Hard caps on every key. Team and feature tags on every request. A golden eval set before the first model downgrade. Cost per completed task as the metric. None of this saves money directly; all of it is what makes saving money possible.

### [2. Low-Hanging Fruit](02-low-hanging-fruit.md) — *the first week*
Prompt caching (reads at ~10% of input price on all three major providers). Batch APIs (a flat 50% off the on-demand token price for anything that can wait). Downshifting easy tasks to cheap tiers (the gap is now 25–50×). Deleting unused tool definitions. Mocking LLM calls in CI. Ships in days, cuts 30–70%, near-zero risk.

### [3. Big Levers](03-big-levers.md) — *when the easy wins are banked*
Model routing and cascades. Self-hosting (a payroll decision, not a GPU decision — the line is ~10M steady tokens/day). Distillation for high-volume narrow tasks. Context engineering for agents. The vector storage layer. Commitment pricing, bought only against real telemetry. Weeks of work, 50–90% cuts, real failure modes.

### [4. Culture](04-culture.md) — *what keeps it from growing back*
Cost on the engineering dashboard next to latency. Caps as a visibility tool, not a punishment. Postmortems for cost anomalies. Token forecasts in planning. Every technical fix decays; only habits compound.

### [5. The Audit Checklist](05-audit-checklist.md) — *run it quarterly*
The full list of questions to ask your own setup, in priority order. If you only read one page, read this one and let it route you to the rest.

## Three rules that survive every model generation

Prices and model names in this playbook will go stale. These won't:

1. **The cheapest token is the one you never send.** Caching, compression, trimming, and deduplication beat clever model selection, because they apply before any model is chosen.
2. **Every discount is a trade against flexibility.** Batch trades latency. Commitments trade forecast risk. Self-hosting trades an engineer. Distillation trades adaptability. Take the trade when you have the thing being traded away in surplus — and not before.
3. **Enforcement beats observation.** Dashboards tell you about the fire. Gates stop it. Every published runaway-bill postmortem had monitoring; none had a gate.

---

Next: [Must-Dos →](01-must-dos.md)
