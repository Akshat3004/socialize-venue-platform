# Local demo and reproducibility

**The code is in the [original team repository](https://github.com/matheusfarocha/Socialize).** This repository contains only portfolio documentation.

## Prerequisites

- Node.js and npm compatible with the **individual apps' package manifests**
- A Supabase project and the schema/migrations appropriate to the current code
- \`NEXT_PUBLIC_SUPABASE_URL\` and \`NEXT_PUBLIC_SUPABASE_ANON_KEY\` in each app's local environment
- \`GEMINI_API_KEY\` for the optional business-side AI insights endpoint

Both apps have \`.env.example\` files in the team repository. Review and configure these locally, without committing secrets.

## Run the apps

~~~bash
git clone https://github.com/matheusfarocha/Socialize.git
cd Socialize
cd apps/business
npm ci
npm run dev
~~~

Open a **second terminal**, from the clone root:

~~~bash
cd Socialize/apps/customer
npm ci
npm run dev -- --port 3001
~~~

The examples assume the second terminal starts in the parent directory containing the cloned \`Socialize\` folder. Avoid port conflicts by giving the customer app a different port.

The operator's local app typically uses \`http://localhost:3000\` and the customer app \`http://localhost:3001\` when started as above.

## Database setup caveats

The team repository contains both app-local and root Supabase migration directories; the root also includes duplicated and remotely captured schema migrations. **Do not blindly apply all SQL files sequentially**. Inspect and reconcile schema history first in a disposable development environment.

- [Main Supabase migrations](https://github.com/matheusfarocha/Socialize/tree/main/supabase/migrations)
- [Business tracking migration](https://github.com/matheusfarocha/Socialize/tree/main/apps/business/supabase/migrations)
- [Order RPC and RLS example](https://github.com/matheusfarocha/Socialize/blob/main/supabase/migrations/20260418201600_create_orders.sql)

## What a technical demo can show

1. Operator signs into a properly configured development venue.
2. Operator creates or edits a room, zones and tables, then checks persistence.
3. Operator creates menu items and verifies they appear in the guest venue route.
4. Guest browses the floor plan, selects a table and adds items to the cart.
5. Guest submits a **test order** and operator confirms it appears in the queue.
6. Operator changes its status; inspect insights with explicit indication of fallback vs Gemini-generated cards.
7. Explore the optional active-user/presence view with synthetic data clearly labeled.

These are **demonstration goals, not a claim they were all successfully executed** for this portfolio.

## Known limitations

- No verified real payment gateway; payment types are checkout UI placeholders.
- Cardholder name and last four digits may appear as a \`payment_label\` in stored orders; **never enter real payment details in a prototype**, and audit stored payment metadata before production use.
- The checkout fallback does separate table inserts, which can leave a partial order if item insertion fails.
- Frontend apps use different Next.js versions, and no root workspace runner was found.
- Unresolved migration overlap and database RLS correctness require testing.
- Some guest social/presence experiences use mock users or demo fallbacks.
- AI-generated insights are not independently verified forecasts; external APIs and service keys are required.
- No current production deployment, metrics, uptime, or successful full integration test has been established here.
