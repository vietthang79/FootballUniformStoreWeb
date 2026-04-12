# Next.js 15 + Supabase E-Commerce Architecture Research

## 1. Prisma ORM vs Supabase JS Client

**Recommendation: Use Prisma for CRUD in Server Actions/Route Handlers**

- **Prisma advantages**: Full type safety with IDE autocomplete, nested relations, predictable queries, explicit control
- **Supabase client**: Better if you rely heavily on auto-generated APIs or need RLS enforcement at query-level
- **Production pattern**: Prisma + PostgreSQL beats Supabase client SDK alone for SaaS e-commerce where type safety & control are critical
- **Gotcha**: Prisma with service role bypasses RLS policies; use anon role key in client for RLS, service role only in server
- **Next.js 15 fit**: Server Actions work natively with Prisma; no custom serialization pain points

## 2. Supabase Storage for File Uploads

**Recommendation: Signed Upload URLs + Private Bucket for Logo Images**

- **Pattern**: Generate signed upload URL server-side in Route Handler, send to client, client uploads directly to Storage (bypasses 1MB Server Action body limit)
- **Why not public buckets**: Security risk—anonymous users can enumerate all files. Public CDN caching is overrated vs. signed URL caching
- **Logo serving**: Create signed GET URLs server-side when rendering product pages (30-day expiry typical)
- **Fallback**: If logos are non-sensitive branding, use public bucket with CDN for simple case
- **Session token tip**: Extract JWT once, cache in memory to avoid repeated `supabase.auth.getSession()` calls

## 3. Cart State Without User Accounts

**Recommendation: Hybrid Approach**

- **Storage**: localStorage for cart items (5-10MB capacity, XSS-vulnerable but acceptable for non-sensitive cart data)
- **Server validation**: Use Supabase Anonymous Sign-In to get anon JWT; store cart in DB (cart_items table, anon session ID as FK)
- **Why hybrid**: localStorage survives page reload; DB survives tab close + 24hrs expiry
- **Complex JSONB**: Store `custom_config` (player roster) as JSONB in cart_items → stringified JSON in localStorage
- **Serialization**: Server Actions auto-serialize JSON; no custom codec needed if cart config stays under 500KB
- **Gotcha**: localStorage accessible to XSS; use Content Security Policy headers + store sensitive tokens in HttpOnly cookies only

## 4. NextAuth.js (Auth.js v5) Admin Route Protection

**Recommendation: Auth.js v5 with Middleware + Role Checking in Server Components**

- **Setup**: Configure Auth.js v5 with Supabase provider (OAuth), define admin role in JWT custom claims
- **Middleware**: Use `proxy.ts` (Next.js 16) or `middleware.ts` (Next.js 15) with matcher to protect `/admin/*` routes
- **Server-side validation**: In Server Component, await `auth()`, check `session.user.role === 'admin'`, redirect if not
- **RLS enforcement**: Create Postgres role `admin` with elevated permissions; use `auth.jwt()` in RLS policies
- **Gotcha**: Middleware alone isn't enough—always revalidate session in server component/route handler
- **Provider**: Use Supabase as provider; Supabase handles user table, Auth.js manages sessions

## 5. PostgreSQL JSONB Schema for E-Commerce

**Recommendation: Hybrid Relational + JSONB Pattern**

```sql
-- Products table
CREATE TABLE products (
  id UUID PRIMARY KEY,
  name VARCHAR NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  inventory INT NOT NULL,
  -- Fixed columns for frequent queries
  category_id UUID REFERENCES categories(id),
  -- JSONB for flexible product data
  variants JSONB, -- [{ id, color, size, stock }]
  specs JSONB -- { material, weight, dimensions }
);

-- Orders table
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  total DECIMAL(10,2) NOT NULL,
  -- JSONB for custom order configs
  custom_config JSONB, -- { roster: [...], jersey_numbers: [...] }
  status VARCHAR NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Cart items (for anon sessions)
CREATE TABLE cart_items (
  id UUID PRIMARY KEY,
  session_id UUID NOT NULL, -- anon auth session
  product_id UUID REFERENCES products(id),
  quantity INT NOT NULL,
  custom_config JSONB, -- { player_name, number }
  created_at TIMESTAMP DEFAULT NOW()
);
```

- **Indexing**: Only index JSONB keys you query; GIN index on `variants` if filtering by color frequently
- **Performance**: JSONB > 2KB gets TOAST-compressed off-page; optimize for <500KB payloads
- **Querying**: Use `variants @> '{"color":"blue"}'` syntax; Prisma can query JSONB but may need raw SQL for complex filters
- **Custom config example**: Store `{ roster: [{ name, number, position }], colors: [primary, secondary] }` in single JSONB column per order

## Key Gotchas

1. **Prisma + RLS**: Bypass happens with service role; enforce auth in Prisma schema or use anon client for public queries
2. **Server Actions body limit**: 1MB default; use signed URLs for file uploads over 100KB
3. **JSONB serialization**: No issues in Server Actions if config stays <1MB; but localStorage has 5-10MB practical limit
4. **Anon sessions**: Supabase expires after 1 hour by default; refresh tokens in middleware
5. **Storage RLS**: Private buckets require authenticated user or signed URL; can't use anon role without explicit RLS policy

## Recommended Tech Stack

- **ORM**: Prisma with Supabase PostgreSQL
- **Storage**: Supabase Storage (signed URLs for uploads, private bucket)
- **Auth**: Auth.js v5 + Supabase provider for admin; Supabase anon for cart sessions
- **State**: localStorage (client) + Supabase DB (anon cart sessions)
- **Database**: PostgreSQL with JSONB for variants/custom_config, GIN indexes on hot keys
- **Schema validation**: Zod for form inputs, Prisma types auto-generated for type safety

---

**Sources:**
- [Prisma with Next.js and Supabase](https://dev.to/mridudixit15/using-prisma-with-nextjs-and-supabase-what-actually-works-and-what-doesnt-4e76)
- [Signed URL file uploads with NextJS and Supabase](https://medium.com/@olliedoesdev/signed-url-file-uploads-with-nextjs-and-supabase-74ba91b65fe0)
- [Supabase Storage Fundamentals](https://supabase.com/docs/guides/storage/buckets/fundamentals)
- [Session Management in Next.js](https://medium.com/@mulugeta.adamu97/the-ultimate-guide-to-storing-sessions-in-react-and-nextjs-6a2188e7a292)
- [Auth.js Protecting Routes](https://authjs.dev/getting-started/session-management/protecting)
- [Next.js Server Actions Guide](https://nextjs.org/docs/app/guides/data-fetching/server-actions)
- [JSONB PostgreSQL Best Practices](https://aws.amazon.com/blogs/database/postgresql-as-a-json-database-advanced-patterns-and-best-practices/)
- [Supabase RLS Policies](https://supabase.com/docs/guides/database/postgres/row-level-security)
