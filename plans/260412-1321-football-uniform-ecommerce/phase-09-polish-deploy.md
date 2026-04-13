---
phase: 9
title: "Polish, SEO, Deploy"
status: pending
priority: P2
effort: 3d
---

# Phase 9: Polish, SEO, Deploy

## Context
- Depends on all previous phases

## Overview
Production readiness: SEO metadata, OG images, performance optimization, error pages, deploy to Vercel + Railway.

## Implementation Steps

1. **SEO**
   - Dynamic metadata for product pages (title, description, OG image)
   - Structured data (JSON-LD: Product, BreadcrumbList)
   - Sitemap generation (`next-sitemap`)
   - robots.txt
   - Vietnamese language meta tags

2. **Performance**
   - Image optimization: Cloudinary loader with Next/Image
   - Lazy load product images below fold
   - Bundle analysis: ensure no unnecessary client-side JS
   - Server Components for all data-fetching pages

3. **Error Handling**
   - Custom 404 page
   - Custom 500 page
   - Error boundaries for client components
   - Toast notifications for cart/checkout errors

4. **UX Polish**
   - Loading skeletons for product grid
   - Optimistic cart updates
   - Smooth step transitions in builder
   - Mobile bottom sheet for cart preview
   - Vietnamese copy throughout

5. **Deploy**
   - Vercel project setup, connect to GitHub
   - Railway PostgreSQL instance
   - Environment variables in Vercel
   - Domain setup (DNS)
   - Cloudinary production account
- Resend domain verification
- VNPay/MoMo production credentials (Phase 2 - defer for MVP)

6. **Monitoring**
   - Vercel Analytics (free)
   - Error tracking: Sentry (free tier)
   - Uptime monitoring

## Todo
- [ ] SEO metadata for all pages
- [ ] Structured data (JSON-LD)
- [ ] Sitemap + robots.txt
- [ ] Image optimization
- [ ] Error pages (404, 500)
- [ ] Loading states + skeletons
- [ ] Vietnamese copy review
- [ ] Vercel deployment
- [ ] Railway PostgreSQL setup
- [ ] Domain + DNS
- [ ] Environment variables
- [ ] Sentry error tracking
- [ ] Final E2E test: browse → custom builder → checkout → payment → email

## Success Criteria
- Lighthouse score > 90 (Performance, SEO)
- Full flow works on production URL
- Shop receives order email within 1 min of checkout
- No console errors on any page
- Mobile responsive on iPhone SE → iPad Pro
