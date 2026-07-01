# Spec — Part 6: AI Strategy (Cadence)

**Functional Requirements**
- Late-payment prediction (GBM/small model, batched nightly) producing per-invoice risk + timing.
- Message drafting (cheap model + templates) personalized by segment/tone.
- Negotiation/edge reasoning (frontier model) only on live customer replies.
- Reply intent classification (cheap model): promise-to-pay, dispute, unsubscribe/STOP, question, other.

**Non-functional Requirements**
- AI COGS < 6% revenue; graceful degradation to templates if LLM unavailable; versioned prompts.

**User Stories**
- As an operator, I want messages personalized yet always within tone policy.
- As finance, I want at-risk invoices flagged before they go late.

**Acceptance Criteria**
- Tone-safety eval gate passes 100% before any drafting model ships.
- Intent classification routes replies correctly ≥ 90% on golden set.
- Frontier model invoked only for negotiation/edge, never bulk drafting.

**Technical Constraints**
- Difficulty-based model routing; batch scoring; cache templates/embeddings; LLM calls queued + rate-limited.

**Edge Cases**
- Ambiguous reply → safe fallback (pause + route to human); STOP/unsubscribe → immediate suppression.
- No history → cohort/segment priors for scoring.

**Success Metrics**
- Collection lift vs control > 15%; tone violations = 0; AI COGS < 6%.
