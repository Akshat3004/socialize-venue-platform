# Engineering case study — Socialize

## Product concept

Socialize targets two overlapping hospitality needs: **venue operators** need an easier way to manage layout and ordering, while **guests** need a visual way to find a table, browse a venue's menu and engage with people at the location.

The team's implementation separates these responsibilities into two Next.js apps backed by shared Supabase data.

## 1. Interactive floor plans as shared data

The business-side editor manages polygonal room outlines, zones, tables, and non-seating elements. Persistence writes editable layouts to Supabase. The guest-side venue view reloads and renders the spatial representation and supports table selection.

**Engineering discussion:** how to represent a visual layout as structured rows, maintain editor/consumer consistency, and handle save failures. The current editor deletes and reinserts tables/structures during a save; the lack of an independently verified transaction boundary deserves review.

[Editor implementation](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/floor-plan/page.tsx) · [Guest rendering](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/page.tsx)

## 2. An order flow from guest to operator

The customer app loads active menu items from Supabase, keeps an interactive cart locally, validates checkout fields, and attempts to store the order using a \`create_order_with_items\` RPC. When the RPC is missing, the code tries direct inserts into \`orders\` and \`order_items\`. The business app reads persisted orders and allows status changes through the operator queue.

**Implementation vs. verification:** these paths exist in the repository, but were **not run end-to-end** in preparing this showcase. The direct insert fallback is not necessarily atomic if inserting order items fails after the order row succeeds.

[Customer checkout](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/menu/page.tsx) · [Order management](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/lib/order-management.ts) · [Order migration](https://github.com/matheusfarocha/Socialize/blob/main/supabase/migrations/20260418201600_create_orders.sql)

## 3. AI insights with deterministic fallback

The authenticated business insights endpoint:
1. Resolves the signed-in owner's venue through Supabase.
2. Loads operational signals from orders, QR scans, seating and customer activity.
3. Uses Gemini to draft structured, actionable insight cards from those signals.
4. Falls back to template-generated cards if AI generation fails.
5. Caches the payload in process memory for an hour.

**Engineering discussion:** grounding an LLM in precomputed facts, avoiding invented numerical claims, separating core business logic from generative presentation and handling provider failures. This does not imply an independently validated forecasting model.

[Insights endpoint](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/api/insights/route.ts) · [Signal extraction](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/lib/insights-engine.ts)

## 4. Realtime people and venue experience

The guest-side code defines an active-user layer, table association, Supabase Realtime subscriptions and prototype social interaction flows. Some views intentionally display synthetic users when backend data is absent. These are promising UI/interaction features but should be carefully labeled in demos as prototype functionality.

[Guest people page](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/people/page.tsx)

## Practical next steps

- Validate fresh Supabase migrations on a disposable database and audit RLS policies.
- Test order submission and management cross-device, including failed/fallback writes.
- Add automated integration tests for editor persistence and menu/order contracts.
- Standardize framework versions and a root workspace runner.
- Document actual data privacy and social safety behavior before live usage.
- Implement real payments only through a PCI-compliant external payment provider; do not accept card credentials directly.

No measured throughput, paid transactions, adoption figures or other production outcomes are claimed.
