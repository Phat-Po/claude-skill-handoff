# Handoff: Phase 3B Complete → Phase 3C (Payment Integration)

> **Sanitized example handoff doc** produced by `/handoff` after Phase 3B was completed.
> Real handoff docs are project-specific — the only fixed thing is the investigation process. Structure, length, and sections are adapted to what actually happened.

---

## Core task (1 sentence)

Wire Stripe Checkout into the order flow so that completing `/checkout` redirects to a Stripe-hosted page and persists the payment intent ID back to the order row.

## Critical constraints (≤3)

- **Do NOT touch** `src/lib/auth/*` — Phase 3A locked the auth surface
- Stripe keys live in `.env.local` only (`STRIPE_SECRET_KEY` / `STRIPE_PUBLISHABLE_KEY`)
- All Stripe calls must go through `src/lib/stripe/client.ts` — never instantiate the SDK elsewhere

---

## Where things stand

### Done this session (Phase 3B)

- Created `orders` table migration (`supabase/migrations/0008_orders.sql`) with columns: `id`, `user_id`, `total_cents`, `currency`, `status`, `stripe_payment_intent_id`, `created_at`, `updated_at`
- Added RLS policy: users can only `SELECT/INSERT/UPDATE` their own rows
- Built `src/lib/orders/create.ts` — pure function, takes cart items + user_id, returns `{ orderId, totalCents }`
- Added `/api/orders` POST route handler that calls `create.ts` and returns the new order
- Wired `src/app/cart/page.tsx` to call `/api/orders` on "Checkout" button click — currently just logs the response, no redirect yet
- Unit tests for `create.ts` at `src/lib/orders/__tests__/create.test.ts` (4 tests, all green)

### Current state

- `npm run build` ✅ passes
- `npm run typecheck` ✅ passes
- `npm test` ✅ 142 passed / 0 failed
- Local dev: cart → "Checkout" → order row appears in Supabase but no payment yet (expected — Phase 3C work)
- 1 uncommitted change: `src/app/cart/page.tsx` (the click handler addition) — **deliberately left uncommitted** so Phase 3C can bundle the Stripe redirect with it in one commit

### What was deliberately NOT done

- No Stripe SDK install yet — Phase 3C's first task
- No success/cancel pages — also Phase 3C
- No webhook handler — Phase 3D
- Did NOT change the cart component's styling even though it's ugly (out of scope, see backlog item #47)

---

## Next steps (in order)

1. **Install Stripe SDK**: `npm i stripe @stripe/stripe-js`
2. **Add Stripe client wrapper** at `src/lib/stripe/client.ts` — single source of SDK instantiation, reads `STRIPE_SECRET_KEY` from env
3. **Build `/api/checkout/session` POST route** — takes `orderId`, fetches order, creates Stripe Checkout Session with `payment_intent_data.metadata.order_id`, returns `{ url }`
4. **Update `src/app/cart/page.tsx`** — after the existing `/api/orders` call, immediately call `/api/checkout/session` and `window.location = response.url`
5. **Add success/cancel routes** at `src/app/checkout/success/page.tsx` and `src/app/checkout/cancel/page.tsx` — minimal pages, real reconciliation happens in Phase 3D webhook
6. **Manually test full flow** in Stripe test mode (card `4242 4242 4242 4242`)
7. **Run** `npm run build && npm test` — must stay green
8. **Snapshot commit** + handoff to Phase 3D (webhook + payment reconciliation)

---

## Minimum reading list

The next agent should read **only these files** before starting:

1. `src/lib/orders/create.ts` — understand the order shape that needs payment
2. `src/app/api/orders/route.ts` — current order creation endpoint
3. `src/app/cart/page.tsx` — the integration point for redirect
4. `supabase/migrations/0008_orders.sql` — column `stripe_payment_intent_id` is the target field
5. `.env.local` (do not commit) — Stripe keys live here
6. `CLAUDE.md` — workspace rules, especially the `git push` + `.env*` guards
7. **Optional**: `docs/phase-3-roadmap.md` for high-level context (skip if context is tight)

---

## Ripple effects from this session

- The `orders` table now exists — any future task that touches orders should `SELECT FROM orders` rather than infer from cart state
- RLS is enabled — service-role key needed for any admin tooling (`SUPABASE_SERVICE_ROLE_KEY` in env)
- The cart page now has a real click handler with `loading` state — be careful not to remove it when adding the redirect

---

## Stale docs flagged

- `docs/checkout-flow-v1.md` is now partially outdated — it assumed orders would be created at payment time, but we now create them at "Checkout" click. Update or delete in Phase 3C.

---

## Verification commands before starting

```bash
cd /path/to/project
git status                              # should show 1 modified file: src/app/cart/page.tsx
npm run build                           # must pass
npm test                                # must pass (142 tests)
psql $DATABASE_URL -c "\d orders"       # confirms migration applied
```

If any of the above fails, **stop and ask the operator before touching code.**

---

## Prompt for the next session

Copy this into the new chat:

```
Read /path/to/project/handoff-3B-to-3C.md first. Then begin Phase 3C: wire Stripe Checkout into the order flow. Critical: do NOT touch src/lib/auth/*. Stripe keys live in .env.local only. All Stripe SDK calls go through src/lib/stripe/client.ts.
```
