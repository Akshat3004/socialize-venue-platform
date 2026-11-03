# Socialize system architecture

**Canonical source:** [matheusfarocha/Socialize](https://github.com/matheusfarocha/Socialize)

## Two-app structure

~~~mermaid
flowchart TB
    subgraph Operator["Business Next.js app"]
      AUTH[Supabase Auth]
      FLOOR[Editable zones, tables and floor plan]
      MENU[Menu/category CRUD]
      QUEUE[Orders queue + metrics]
      INSIGHTS[AI-assisted insights]
    end
    subgraph Guest["Customer Next.js app"]
      VENUE[Venue by slug / floor map]
      CART[Menu + local cart]
      CHECK[Checkout]
      PEOPLE[Presence / social prototype]
    end
    DB[(Supabase PostgreSQL)]
    RT[Supabase Realtime]
    AI[Gemini]
    AUTH --> DB
    FLOOR --> DB
    MENU --> DB
    QUEUE --> DB
    VENUE --> DB
    CHECK -->|create_order_with_items RPC| DB
    PEOPLE <--> RT
    RT <--> DB
    DB --> INSIGHTS
    INSIGHTS --> AI
    CART --> CHECK
~~~

The source is organized into **two independent Next.js applications** under \`apps/business\` and \`apps/customer\`. Each has its own package manifest and lockfile; there is not a verified root workspace orchestration layer.

## Business features

- Supabase authenticated operator access and venue lookup
- Floor geometry, zone, table, and structural-element editor, with database save
- Menu category and item management
- Order queue, status transitions, recent sales summaries and revenue-trend calculations
- An authenticated, signal-based insight generator using Gemini and template fallback

Relevant files: [floor-plan editor](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/floor-plan/page.tsx), [menu editor](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/menu/page.tsx), [order service](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/lib/order-management.ts), [insights endpoint](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/api/insights/route.ts).

## Guest features

- Venue lookup by URL slug, and database-backed zones/tables
- Interactive floor-plan rendering and a table-aware menu view
- Local browser cart; persistent **orders** through Supabase RPC (with direct insert fallback)
- Experimental presence features using Supabase Realtime
- Demo social activity and QR scan/session tracking

Relevant files: [venue page](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/page.tsx), [menu checkout](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/menu/page.tsx), [cart cache](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/lib/order-storage.ts), [presence](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/lib/use-venue-presence.ts).

## Data layers

The Supabase schema uses venues, zones, tables, structural elements, menu categories/items, orders and order items, along with tracking/presence records. The original source includes SQL migrations, an order RPC, and RLS policies. Their presence does **not** prove all migrations were applied successfully to a deployed database or fully audited for security.

[View Supabase migrations](https://github.com/matheusfarocha/Socialize/tree/main/supabase/migrations)

## Boundaries

- Card checkout is a **mock method selection**, not a real payment integration.
- AI insights summarize observed signals and may fall back to templates; they are not revenue forecasts with measured model accuracy.
- Source contains demo content and partial/fallback experiences; production adoption and reliability remain unverified.
