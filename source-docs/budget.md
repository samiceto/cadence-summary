# Cadence — Production Cost Analysis

A breakdown of what it costs to **run Cadence in production for a client**, across every paid service in the stack. Figures are monthly USD estimates at typical 2026 list prices — **verify current pricing with each provider**, as it changes. Stripe transaction fees are pass-through (see §5) and excluded from the run-cost totals.

---

## 1. The stack — what costs money

| Layer | Service | Pricing driver |
|-------|---------|----------------|
| Hosting — API | Render web service | Instance plan (always-on) |
| Hosting — Worker | Render background worker | Instance plan |
| Hosting — Web | Render web service (or Vercel) | Instance plan |
| Cache/Queue | Render Key Value (Redis) | Memory size |
| Database | Neon Postgres | Compute + storage |
| Auth | Clerk | Monthly active users (dashboard logins) |
| Email | Resend | Emails / month |
| SMS | Twilio | Segments + number + A2P registration |
| AI drafting | OpenAI | Tokens (very low) |
| Monitoring | Sentry + PostHog | Errors / events |
| Object storage | Cloudflare R2 | GB stored (minimal) |
| Domain | Registrar | Annual, amortized |

---

## 2. Assumptions per scenario

| | **Small** | **Medium** | **Large** |
|---|---|---|---|
| Active overdue invoices chased / mo | ~300 | ~1,500 | ~6,000 |
| Email reminders / mo | ~1,000 | ~5,000 | ~25,000 |
| SMS reminders / mo | ~200 | ~1,500 | ~8,000 |
| AI drafts / mo | ~1,000 | ~5,000 | ~25,000 |
| Dashboard users (Clerk MAU) | 1–3 | 3–10 | 10–25 |

> Note: the people **receiving** reminders are *not* Clerk users — only the client's finance team logging into the dashboard counts toward Clerk MAU. That keeps auth costs near zero even at scale.

---

## 3. Per-service cost

### Hosting (Render) — must be **always-on** in production
Free-tier instances sleep on idle and **miss Stripe webhooks during cold starts**, so production needs paid plans.

| Service | Small | Medium | Large |
|---------|-------|--------|-------|
| API (web) | Starter $7 | Standard $25 | Standard ×2 ≈ $50 |
| Worker | Starter $7 | Standard $25 | Standard ×2 ≈ $50 |
| Web | Starter $7 *(or Vercel free)* | Starter $7 | Standard $25 |
| Redis (Key Value) | ~$10 | ~$10 | ~$30 |
| **Render subtotal** | **~$31** | **~$67** | **~$155** |

*The production Blueprint autoscales the API/worker; under sustained bursts the large tier can exceed this — autoscaling is a ceiling, not a fixed cost.*

### Database (Neon)
| | Small | Medium | Large |
|--|-------|--------|-------|
| Plan | Free–Launch | Launch ~$19 | Scale ~$69 |

### Email (Resend)
- Free up to ~3,000/mo; Pro ~$20/mo includes 50,000.
- Small: **$0** · Medium: **$20** · Large: **$20**

### SMS (Twilio) — US, ~$0.0079/segment + number ~$1.15 + A2P 10DLC campaign ~$4/mo
- Small (~200): **~$7** · Medium (~1,500): **~$17** · Large (~8,000): **~$70**
- *Multi-segment or international messages cost more; budget ~$0.008–0.016 per effective message.*

### AI (OpenAI) — `gpt-4o-mini` drafting ≈ $0.0002 per message
- Small: **~$1** · Medium: **~$2** · Large: **~$5–15** (incl. occasional `gpt-4o` synthesis)
- **The cheapest line item by far** — AI is not a meaningful cost driver here.

### Auth (Clerk)
- Free up to 10,000 MAU. All scenarios fit the free tier (few internal users): **$0**
  *(Add ~$25/mo only if you want Pro features like enhanced session controls.)*

### Monitoring (Sentry + PostHog) — optional
- Both have free tiers. Paid: Sentry Team ~$26, PostHog usage-based.
- Small: **$0** · Medium: **$0–26** · Large: **~$26–52**

### Storage (Cloudflare R2) + Domain
- R2: a few GB, free egress → **~$0–1**. Domain: **~$2/mo** amortized.

---

## 4. Total monthly run-cost

| Service | Small | Medium | Large |
|---------|------:|-------:|------:|
| Render (hosting + Redis) | $31 | $67 | $155 |
| Neon (database) | $0–19 | $19 | $69 |
| Resend (email) | $0 | $20 | $20 |
| Twilio (SMS) | $7 | $17 | $70 |
| OpenAI (AI) | $1 | $2 | $10 |
| Clerk (auth) | $0 | $0 | $0 |
| Sentry + PostHog | $0 | $0–26 | $26–52 |
| R2 + domain | $2 | $2 | $3 |
| **Estimated total / mo** | **≈ $41–60** | **≈ $127–155** | **≈ $350–390** |

> **Rule of thumb:** a small client runs around **$50/mo**, a mid-size around **$150/mo**, a large one around **$400/mo**. Hosting + database dominate at small scale; **SMS volume** becomes the main variable cost at large scale.

---

## 5. Costs that are NOT in the totals above

- **Stripe processing fees** — ~2.9% + $0.30 per payment. This is **pass-through** on money the client *collects*, not a platform operating cost. (If you bill the client *through* Stripe too, that subscription incurs the same fee.)
- **One-time setup:** Twilio A2P 10DLC brand registration (~$4–44 + vetting), domain purchase, your implementation/onboarding time.
- **Accounting marketplace:** Intuit/Xero developer accounts are free; public listing requires app-review (time, not money).
- **Your margin / labor:** implementation, support, and maintenance — price this into the client contract separately.

---

## 6. Cost-control levers

1. **Right-size hosting** — start on Starter plans; move to Standard + autoscaling only when load demands it. This is the biggest lever.
2. **Email over SMS** — email is effectively free up to 50k/mo; SMS is the priciest channel. Use SMS only for high-risk/aged invoices (the risk model already gates channel by risk).
3. **Database** — Neon's autoscaling-to-zero (Launch plan) keeps idle cost low for smaller clients.
4. **Web on Vercel free/hobby** — shave the web service cost if you prefer.
5. **Monitoring free tiers** — sufficient until real volume justifies paid.

---

## 7. Margin example (illustrative)

If the client is on the **Growth tier ($349/mo)** and their run-cost is **~$150/mo**, gross infrastructure margin is **~$199/mo (~57%)** before your labor — and that margin *improves* with scale because hosting/DB don't grow linearly with message volume.

> All figures are planning estimates. Confirm against live pricing from Render, Neon, Clerk, Resend, Twilio, OpenAI, Sentry, and PostHog before quoting a client.
