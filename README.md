# Socialize — Venue Management & Interactive Guest Experience

**Hack Brooklyn 2026 · Next.js · TypeScript · React · Supabase · PostgreSQL · Realtime · Gemini AI**

A team-built hospitality platform that connects **venue operators** with **guests** through an editable floor plan, menu management, QR-accessible venue pages, ordering, and engagement analytics.

> **Portfolio case study:** This repository is maintained by [Akshat Agrawal](https://github.com/Akshat3004) to present the team's work. The implementation, Git history, and collaborative contributions live in the **[original Socialize team repository](https://github.com/matheusfarocha/Socialize)**. No source code has been copied or represented as solely authored by Akshat.

**[Original source code](https://github.com/matheusfarocha/Socialize)** · **[Architecture](docs/ARCHITECTURE.md)** · **[Engineering case study](docs/CASE_STUDY.md)** · **[Run guide and limitations](docs/DEMO_AND_LIMITATIONS.md)** · **[Credits](CONTRIBUTORS.md)**

## At a glance

Socialize consists of **two separate Next.js applications** sharing a Supabase-backed data model:

| Business-side experience | Customer-side experience |
| --- | --- |
| Owner authentication and venue dashboard | Venue discovery by slug / QR-linked route |
| Editable, multi-zone floor-plan canvas | Interactive venue and table layout |
| Menu category and item management | Menu browsing and cart |
| Order management queue and status changes | Dine-in or pickup order submission |
| Revenue and operational summaries | Guest presence and social interactions |
| Gemini-assisted, signal-grounded insights | Table-aware venue navigation |

The current source includes **order persistence via a Supabase database function**, with a direct table-insert fallback. Payment selection remains **mock UI**—there is no verified card-processing integration or real-money transaction flow.

## System overview

~~~mermaid
flowchart TB
  subgraph Business["Business app — Next.js"]
    BO[Owner login and dashboard]
    FP[Floor-plan editor]
    MC[Menu and orders]
    IN[Insights]
  end
  subgraph Guest["Customer app — Next.js"]
    VP[Venue page and floor plan]
    ME[Menu, cart and checkout]
    PR[Presence and social]
  end
  DB[(Supabase Postgres)]
  RT[Supabase Realtime]
  AI[Gemini + deterministic insight fallback]
  FP --> DB
  MC --> DB
  BO --> DB
  VP --> DB
  ME -->|create_order_with_items RPC| DB
  PR <--> RT
  RT <--> DB
  DB --> IN
  IN <--> AI
~~~

## Engineering highlights

### 1. Spatial venue management

Venue operators edit room geometry, add zones, place tables and structural elements, and persist changes through Supabase. Customers see a rendered version of the layout and can select an appropriate table.

[Floor plan editor](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/floor-plan/page.tsx) · [Canvas components](https://github.com/matheusfarocha/Socialize/tree/main/apps/business/components/floor-plan) · [Guest venue page](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/page.tsx)

### 2. Menu management and ordering

The business app supports categories and menu item CRUD backed by Supabase. The guest app loads active menu items, manages a local browser cart, and submits orders. The latest code uses a **Supabase RPC** to insert an order and its items, falling back to direct inserts if the RPC is unavailable. An owner-facing queue reads persisted orders and supports status progression.

[Menu management](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/menu/page.tsx) · [Guest checkout](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/menu/page.tsx) · [Order-management service](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/lib/order-management.ts) · [Order migration](https://github.com/matheusfarocha/Socialize/blob/main/supabase/migrations/20260418201600_create_orders.sql)

### 3. Operational analytics and AI-assisted insights

The operator dashboard derives revenue, order, seating and sales summaries from Supabase records. An authenticated insights endpoint computes signals from venue activity and asks Gemini for actionable cards, with a **template-based fallback** if the provider cannot return usable results. This is a hybrid AI-plus-rules feature, not a trained forecasting model.

[Dashboard](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/%28dashboard%29/page.tsx) · [Insights endpoint](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/app/api/insights/route.ts) · [Signal-engine code](https://github.com/matheusfarocha/Socialize/blob/main/apps/business/lib/insights-engine.ts)

### 4. Real-time venue presence

The guest experience includes table-aware active-user presence, Supabase Realtime channels, and experimental social interactions. These features are demo/prototype scope; we have **not** independently tested cross-device reliability or privacy/abuse protections.

[Guest presence page](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/app/v/%5Bslug%5D/people/page.tsx) · [Presence state helper](https://github.com/matheusfarocha/Socialize/blob/main/apps/customer/lib/use-venue-presence.ts)

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | **Next.js**, **React**, **TypeScript**, Tailwind CSS |
| Database and auth | **Supabase**, PostgreSQL, Supabase Auth and row-level security migrations |
| Data flows | Venue / zone / table records, menu categories and items, orders, QR scans and customer sessions |
| Realtime | Supabase Realtime, active guest presence |
| AI | Gemini-backed insight generation with deterministic fallback |
| UI and mapping | Interactive floor plan, SVG/canvas interactions, Mapbox in customer dependencies |

The two apps use **different Next.js versions**, and there is no single root workspace setup in the team repository. They should be installed and run separately.

## Running the original project

The code is in [matheusfarocha/Socialize](https://github.com/matheusfarocha/Socialize), **not this documentation repository**.

~~~bash
git clone https://github.com/matheusfarocha/Socialize.git
cd Socialize/apps/business
npm install
npm run dev
~~~

In a different terminal, from the clone root:

~~~bash
cd apps/customer
npm install
npm run dev -- --port 3001
~~~

Both apps require properly configured **Supabase URL and anon key**; server-side AI insights also require a Gemini key. Follow [the configuration and database guide](docs/DEMO_AND_LIMITATIONS.md). These commands have **not** been end-to-end verified in preparing this showcase.

## Honest limitations

- No real payment processing; card/Apple Pay selections are mock payment methods.
- Order submission has a Supabase persistence path, but that does **not** prove successful live deployment or transactional correctness under all schema versions.
- Migration files include duplicate/overlapping snapshots; database setup requires review.
- Customer presence/chat and AI insights depend on service configuration and may use fallback or demo data.
- No independently established active-user count, revenue, uptime, automated-test pass rate, or production traffic metric is claimed.

## Attribution

Socialize was built collaboratively for Hack Brooklyn. This showcase is a documentation and navigation layer; [the original repository](https://github.com/matheusfarocha/Socialize) remains the canonical source for team authorship and code history. See [CONTRIBUTORS.md](CONTRIBUTORS.md).

*Resume framing: Collaborated on Socialize, a two-app Next.js/Supabase hospitality prototype with editable venue floor plans, menu management, persistent order workflows, and AI-assisted operator insights.*
