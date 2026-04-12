---
phase: 7
title: "Admin + Order Management"
status: pending
priority: P2
effort: 3d
---

# Phase 7: Admin + Order Management Basics

## Context
- Depends on Phase 5+6 (orders exist in DB)

## Overview
Simple admin panel: view orders, update status. No product CRUD — products managed via seed script + Prisma Studio. Protected by auth.
<!-- Updated: Validation Session 1 - Admin scope reduced: order management only, no product CRUD web UI -->

## Architecture

```
/admin              → Dashboard (order count, revenue today/week/month)
/admin/orders       → Order list with filters (status, date, search by order number)
/admin/orders/[id]  → Order detail: customer info, items, player list, download Excel, logo previews, status update
```

## Related Code Files

### Create
- `src/app/admin/layout.tsx` — admin layout with sidebar nav + auth guard
- `src/app/admin/page.tsx` — dashboard
- `src/app/admin/orders/page.tsx` — order list
- `src/app/admin/orders/[id]/page.tsx` — order detail
- `src/components/admin/order-status-badge.tsx`
- `src/components/admin/order-status-updater.tsx` — dropdown to change status
- `src/app/api/admin/orders/[id]/status/route.ts` — PATCH status

### Modify
- `src/lib/auth.ts` — add admin role check

## Implementation Steps

1. Add admin role to Better Auth config (email whitelist in env var)
2. Create admin layout with auth guard middleware
3. Dashboard: aggregate queries (order count, revenue by period)
4. Order list: paginated table, filter by status/date, search by order number
5. Order detail: full info, player list table, download Excel link, logo previews, update status
6. Status update API with email notification to customer on ship/deliver

## Todo
- [ ] Admin auth guard (email whitelist)
- [ ] Dashboard with stats
- [ ] Order list with filters + search
- [ ] Order detail page (player list, logos, status)
- [ ] Order status update (with customer email notification)

## Success Criteria
- Admin can view all orders, filter by status/date, search by order number
- Admin can update order status → customer gets email
- Admin can download Excel + view logos from order detail
- Non-admin users cannot access /admin

## Risk
- Scope creep — keep admin minimal, avoid building full CMS
- Image management complexity — start with simple upload, no reordering
