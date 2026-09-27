# K·Glow — Luxury Korean Beauty Store

A premium, mobile-first Korean beauty e-commerce storefront with a secure admin dashboard.
Currency: **Moroccan Dirham (MAD / DH)**.

## Tech stack

- **TanStack Start v1** (React 19, file-based routing, SSR + server functions)
- **Vite 7** build tooling
- **TypeScript**
- **Tailwind CSS v4** (theme tokens in `src/styles.css`)
- **TanStack Query** for data fetching
- **Supabase** (Postgres, Auth, Storage) as the backend

## Features

- Homepage with hero, promos, featured products, brand marquee, reviews
- Shop page with category filters and sorting
- Product detail page with zoom gallery, stock status, ingredient/benefit accordions
- Admin dashboard at `/admin`: create/edit/delete products, upload images,
  manage categories, manage stock and prices (in MAD), view and update orders
- Role-based access control (`user_roles` table + `has_role()` DB function)
- SEO: per-route metadata, `robots.txt`, generated `sitemap.xml`

## Project structure

```
src/
  routes/          # file-based routes (index, shop, product.$id, admin, auth)
  components/site/ # Nav, Footer, ProductCard
  components/ui/   # shadcn-style primitives
  lib/             # products data layer, currency helpers, image resolution
  integrations/supabase/  # generated clients & types (do not edit by hand)
  styles.css       # Tailwind v4 theme tokens
supabase/          # project config
public/            # static assets
```

## Prerequisites

- Node.js 20+ and npm (or bun)
- A Supabase project (free tier is fine)

## Installation

```sh
npm install
```

## Environment variables

Create a `.env` file in the project root:

```
VITE_SUPABASE_URL="https://<project-ref>.supabase.co"
VITE_SUPABASE_PUBLISHABLE_KEY="<publishable-anon-key>"
VITE_SUPABASE_PROJECT_ID="<project-ref>"

SUPABASE_URL="https://<project-ref>.supabase.co"
SUPABASE_PUBLISHABLE_KEY="<publishable-anon-key>"
SUPABASE_PROJECT_ID="<project-ref>"
```

Server-only secrets (never expose to the browser) are set in your hosting
provider's dashboard:

```
SUPABASE_SERVICE_ROLE_KEY="<service-role-key>"
```

## Run locally

```sh
npm run dev
```

The app starts on http://localhost:8080.

## Build

```sh
npm run build      # production build
npm run start      # serve the production build
```

## Database

The backend uses these tables:

| Table        | Purpose                                             |
| ------------ | --------------------------------------------------- |
| `products`   | Catalogue: name, brand, category, price (MAD), stock, media, copy |
| `categories` | Category names and slugs                            |
| `orders`     | Customer orders with JSON line items and totals in MAD |
| `user_roles` | `admin` / `user` roles, checked by `has_role()`     |

Row Level Security is enabled on every table:

- `products` and `categories`: public read, admin-only write
- `orders`: public insert (checkout), admin read/update/delete
- `user_roles`: users can read their own roles only

Product images live in the private `product-images` storage bucket and are
served through long-lived signed URLs.

### Admin access

The first account that signs up is granted the `admin` role automatically
(`handle_new_user_role()` trigger). Sign up at `/auth`, then open `/admin`.

## Currency

All monetary values are stored as plain numbers and rendered in Moroccan
Dirham through `src/lib/currency.ts`:

```ts
import { formatMAD, formatMADShort } from "@/lib/currency";

formatMAD(249);       // "249,00 DH"
formatMADShort(249);  // "249 DH"
```

Admin price inputs (`Price (MAD)`, `Compare-at (MAD)`) are entered and stored
in dirhams. To change the currency, edit `CURRENCY_SYMBOL` / `CURRENCY_CODE`
in `src/lib/currency.ts` — no other file hardcodes a currency symbol.

## Deployment

The app targets an edge runtime (Cloudflare Workers compatible) and also runs
on any Node host.

1. Run `npm run build`.
2. Deploy the generated output with your provider (Cloudflare Workers/Pages,
   Vercel, Netlify, or a Node server via `npm run start`).
3. Set the environment variables listed above in the provider's dashboard.
4. Add your production domain to the Supabase Auth redirect URL allow-list so
   sign-in works after deployment.

Backend changes (schema, policies, storage) apply immediately; frontend changes
require a redeploy.

## License

Proprietary — all rights reserved.
