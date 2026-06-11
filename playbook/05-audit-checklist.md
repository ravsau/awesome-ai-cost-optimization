# The AI Cost Audit Checklist

[Playbook](00-start-here.md) · Previous: [Culture](04-culture.md)

Run quarterly — model prices and commitment options reset every few months. Ordered by how often each item finds money.

## Visibility (fix these first if they fail)

- [ ] **1.** Can you say what each feature costs per use — not the invoice total?
- [ ] **2.** Does every key (including dev/CI) have a hard dollar cap and model allow-list?
- [ ] **3.** Would you know within an hour if spend went 10× overnight — with a gate, not just an alert?

## Easy money

- [ ] **4.** Is caching on and actually hitting? Check `cache_read_input_tokens` / `cached_tokens` in real responses.
- [ ] **5.** Is every cron-driven job on the batch API (50% off)?
- [ ] **6.** Which calls run on the flagship model, and why? List every call site and its model.
- [ ] **7.** What's in the prompt that doesn't need to be — unused tools, boilerplate, uncapped `max_tokens`?
- [ ] **8.** What do retries cost (bad parses, timeouts, agent re-plans)?

## Architecture

- [ ] **9.** Anything generated in real time that could be precomputed offline at batch rates?
- [ ] **10.** Vector layer cost per GB vs pgvector / object-storage alternatives?
- [ ] **11.** For agents: cost per completed task, and context size at step 20?
- [ ] **12.** Are near-duplicate questions paying twice (no semantic cache)?

## Commitments and contracts

- [ ] **13.** Pricing structure checked against 30–60 days of real telemetry? Committed capacity below ~50% utilization is losing money; steady high volume on pay-as-you-go is too.
- [ ] **14.** AI seat spend vs actual usage — both directions (unused max-tier seats, and rate-limited engineers on $20 plans)?
- [ ] **15.** Every commitment renewal has a named owner and a calendar date?

---

*Maintained by [Saurav Sharma](https://linkedin.com/in/saurav-sharma-cloud). Want this run against your real billing data with dollar figures on each finding? [AI Cost Audit](https://cloudyeti.io) — start with a [free 30-min call](https://cloudyeti.io/chat).*
