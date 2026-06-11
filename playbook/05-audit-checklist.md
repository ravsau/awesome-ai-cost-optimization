# The AI Cost Audit Checklist

Part of the [AI Cost Optimization Playbook](00-start-here.md). Previous: [Culture](04-culture.md)

---

Run this against your own setup once a quarter. Model prices, commitment options, and the cheapest way to run a workload reset every few months — an audit from last year is a historical document.

Work top to bottom. Items are ordered by how often they find money.

## Visibility (if any of these fail, stop and fix them first)

- [ ] **1. Can you say what each feature costs per use?** Not the invoice total — dollars per ticket summarized, per document processed, per task completed. If not: no gateway, no tags, no audit. Start at [Must-Dos](01-must-dos.md).
- [ ] **2. Does every API key have a hard dollar cap and a model allow-list?** Including dev keys, CI keys, and that service account from the hackathon.
- [ ] **3. Would you know within an hour if spend went 10× overnight?** An alert someone is actually on call for, plus a gate that stops the next call — not just a dashboard.

## The easy money (each of these is typically 30–70% on the affected workload)

- [ ] **4. Is prompt caching on, and is it hitting?** Check `cache_read_input_tokens` (Anthropic) or `cached_tokens` (OpenAI) in real responses. A timestamp or session ID at the top of the system prompt silently kills it.
- [ ] **5. Is every cron-driven LLM job on the batch API?** Evals, embedding backfills, nightly reports. Same tokens, 50% off the on-demand price, on all three major providers.
- [ ] **6. Which calls run on the flagship model, and why?** List every call site and its model. Classification, extraction, and routine summarization on a frontier model is the single most common finding. The cheap tiers are 25–50× cheaper.
- [ ] **7. What's in the prompt that doesn't need to be?** Unused tool definitions (billed every call), boilerplate instructions, whole documents where a section would do, uncapped `max_tokens`.
- [ ] **8. What do retries cost?** Bad JSON parses, timeout retries, agent re-plans. Structured outputs and idempotent design kill most of them.

## Architecture (worth a look once the easy money is banked)

- [ ] **9. Is anything generated in real time that could be precomputed?** Anything shown to many users, or predictable in advance, can be batch-generated offline at half price.
- [ ] **10. What does your vector layer cost per GB, all-in?** Compare against pgvector on the database you already run, or object-storage-backed stores. Storage is the most overpaid line item in RAG.
- [ ] **11. For agents: what's the cost per completed task, and what's the context size at step 20?** Agent context compounds. Context editing, summarization-based memory, and step caps are the fixes.
- [ ] **12. Are duplicate questions paying twice?** If users ask near-identical questions, semantic caching turns repeats into cache hits.

## Commitments and contracts (where the big-company money hides)

- [ ] **13. Are you on the right pricing structure for your actual volume?** Pay-as-you-go vs provisioned/committed capacity, checked against 30–60 days of real telemetry. A committed unit running below ~50% utilization is a price increase wearing a discount costume. Conversely: steady high volume on pay-as-you-go is leaving the commitment discount on the table.
- [ ] **14. What do AI seats cost vs what gets used?** Coding-tool seats at $100–200/engineer/month for people who open the tool twice a month; or the inverse, engineers rate-limited on $20 plans while their time costs 100× the upgrade.
- [ ] **15. When does each commitment renew, and who owns that date?** Use-it-or-lose-it capacity, auto-renewing reservations, and expiring credits all need a named owner and a calendar entry.

---

## Scoring it honestly

- **12–15 checked:** your setup is sound. Re-run next quarter; the pricing landscape will have moved.
- **8–11:** there's real money here. Items 4–8 alone usually pay for the time several times over.
- **Under 8:** you're not running an AI cost problem, you're running an unmeasured system. Go back to [Must-Dos](01-must-dos.md) and build the visibility layer first.

---

*This checklist is maintained by [Saurav Sharma](https://linkedin.com/in/saurav-sharma-cloud) (ex-Amazon, 12× AWS certified). If you'd rather have someone who does this every week run it against your real billing data and hand your team a ranked fix list with dollar figures — that's the [AI Cost Audit](https://cloudyeti.io). Start with a [free 30-minute call](https://cloudyeti.io/chat).*
