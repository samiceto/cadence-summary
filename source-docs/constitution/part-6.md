# Constitution — Part 6: AI Strategy (Cadence)

**Where AI is used:**
1. **Late-payment prediction** (gradient-boosted / small model on payment-history features) — cheap, batched.
2. **Message drafting** (cheap model + templates, personalized by segment/tone).
3. **Negotiation & edge reasoning** (frontier model) — only when a customer replies/negotiates.
4. **Reply intent classification** (cheap model) — route replies (promise-to-pay, dispute, unsubscribe).

**Guardrails (relationship-safety):**
- Tone bounded by policy; never threatening; VIP/blocklist respected; max-contact frequency enforced.
- Human approval for sensitive actions (large discounts, legal-adjacent language, VIP outreach).
- All agent actions logged + reversible.

**Cost discipline:** route cheap tasks to cheap/open models; frontier only on live negotiation; batch predictions nightly; cache templates/embeddings. AI COGS < 6% revenue.

**Model governance:** versioned prompts; eval suite on tone-safety + intent accuracy + collection lift; no customer data trains shared models without opt-in.

**Consistency check:** supports relationship-safety (Part 1), margin (Part 3), quality (Part 10).
