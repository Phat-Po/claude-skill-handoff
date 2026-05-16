# Friend Circle Hackathon — Status Log

> **Sanitized example `STATUS.md`** maintained automatically by the `handoff` skill at the project root.
> Each entry is append-only. Reading just the last entry should be enough to orient a fresh agent.

---

## 2026-05-10 | Project scaffolding + Supabase auth wired

**Done this session:**
- Initialized Next.js 16 project with TypeScript + Tailwind
- Set up Supabase project, configured `@supabase/ssr` client
- Built login/signup pages, magic-link flow working end-to-end
- Deployed to Vercel, confirmed auth works in prod

**Current state:**
- Build green, all 12 tests pass
- Auth surface is locked — DO NOT modify `src/lib/auth/*` going forward without explicit approval

**Next steps:**
1. Define the `products` and `orders` schema in Supabase
2. Build the product browse page
3. Wire cart state (client-side first, persistence later)

**Decisions / notes:**
- Chose Supabase magic link over Google OAuth because operator wants email-first onboarding
- Did NOT add Stripe yet — payment is Phase 3
- Vercel preview deploys are public — be careful with content before launch

---

## 2026-05-12 | Phase 2 complete — product catalog + cart UI

**Done this session:**
- Created `products` table + 50 seed rows
- Built browse page at `/products` with filtering by category
- Cart state in localStorage (Zustand store at `src/stores/cart.ts`)
- Cart page at `/cart` shows items, totals, remove button

**Current state:**
- Build green, 47 tests pass
- Cart persists across refreshes (localStorage)
- No checkout yet — "Checkout" button currently `console.log`s

**Next steps:**
1. Phase 3A: lock the order schema (orders table + RLS)
2. Phase 3B: wire `/api/orders` POST so "Checkout" button creates real order rows
3. Phase 3C: integrate Stripe Checkout

**Decisions / notes:**
- Zustand over Redux because cart is small + zero learning curve for operator
- Skipped server-side cart persistence — re-evaluate after Stripe lands

---

## 2026-05-15 | Phase 3A complete — orders schema + RLS

**Done this session:**
- `orders` table migration applied (`supabase/migrations/0008_orders.sql`)
- RLS policy: users can only see/modify their own orders
- Helper `getCurrentUserOrders()` at `src/lib/orders/queries.ts`

**Current state:**
- Build green, 138 tests pass
- Migration applied to staging Supabase — DO NOT re-run
- Orders table is empty (no creation flow yet — Phase 3B)

**Next steps:**
1. Phase 3B: `/api/orders` POST endpoint that creates order rows from cart contents
2. Wire cart "Checkout" button to call it
3. Then Phase 3C: Stripe Checkout integration

**Decisions / notes:**
- `total_cents` stored as integer to avoid floating-point money bugs
- `status` is an enum: `'pending' | 'paid' | 'fulfilled' | 'cancelled'`
- RLS uses `auth.uid()` — service role key needed for any admin tooling

---

## 2026-05-17 | Phase 3B complete — order creation wired, handoff to 3C

**Done this session:**
- `src/lib/orders/create.ts` — pure function, 4 unit tests green
- `/api/orders` POST route handler live
- Cart "Checkout" button now creates real order rows (no payment redirect yet)
- 1 uncommitted change in `src/app/cart/page.tsx` deliberately left for Phase 3C to bundle with Stripe redirect

**Current state:**
- Build green, 142 tests pass
- Local: cart → Checkout → order row appears in Supabase ✓
- See `handoff-3B-to-3C.md` for the full handoff doc

**Next steps:**
1. Install Stripe SDK
2. Build Stripe client wrapper at `src/lib/stripe/client.ts`
3. Build `/api/checkout/session` POST route
4. Update cart page to redirect to Stripe after order creation
5. Add success/cancel pages
6. Test full flow in Stripe test mode

**Decisions / notes:**
- Did NOT clean up the cart page styling even though it's ugly (out of scope, backlog item #47)
- `docs/checkout-flow-v1.md` is now partially stale — update or delete in 3C
