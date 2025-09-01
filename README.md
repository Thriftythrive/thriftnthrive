# Thrift N Thrive — Next.js + Serverless API (Stripe + Supabase)

## Quick start
```bash
npm i
cp .env.local.example .env.local
# fill keys
npm run dev
```

## Stripe webhook (local)
```bash
stripe login
stripe listen --forward-to localhost:3000/api/webhook
```

## Deploy (Vercel)
- Add envs: STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, SUPABASE_URL, SUPABASE_SERVICE_KEY
- Deploy, then:
```bash
stripe listen --forward-to https://YOUR-VERCEL-URL/api/webhook
```

## Supabase orders table
```sql
create table public.orders (
  id text primary key,
  session_id text,
  email text,
  status text default 'Processing',
  date timestamptz,
  total numeric,
  items jsonb
);
```

## Admin inventory (serverless + Supabase)

### Products table
```sql
create table public.products (
  id uuid primary key default gen_random_uuid(),
  title text,
  brand text,
  category text,
  size text,
  condition text,
  price numeric,
  img text,
  stock int default 1,
  added_at timestamptz default now()
);
```

### API
- `GET /api/products` → public (used by the shop UI)
- `POST /api/products` → create (requires header `x-admin-token: $ADMIN_TOKEN`)
- `PUT /api/products?id=...` → update by id (requires header token)
- `DELETE /api/products?id=...` → delete by id (requires header token)

Set **ADMIN_TOKEN** in `.env.local` and enter it once in the admin page UI to unlock write actions.


## Product images (Supabase Storage)
- Create a Storage bucket named `products` and set it Public (read).
- Admin uploads via `/api/upload-url` to get a signed URL; then PUTs the file.
- The server returns a public URL; the admin UI fills the image field automatically.
- Add `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` to `.env.local`.

## Multi‑image products (gallery + cover)
Add these columns to your `products` table (or create new):
```sql
alter table public.products
  add column if not exists images jsonb default '[]'::jsonb,
  add column if not exists cover_index int default 0;

-- Optional: keep legacy single image column for compatibility
alter table public.products
  add column if not exists img text;
```
- **images**: array of image URLs (strings)
- **cover_index**: which image is the cover (0-based). The shop uses cover → falls back to `img` if empty.


## Product detail pages
- Pages Router dynamic route at `/product/[id]`.
- Server-side fetch from Supabase using service key.
- The grid links to details via the **Details** button.
