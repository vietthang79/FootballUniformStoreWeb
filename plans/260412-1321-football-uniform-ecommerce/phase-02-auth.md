---
phase: 2
title: "Auth — Login / Register / Forgot Password"
status: pending
priority: P1
effort: 2d
---

# Phase 2: Auth — Login / Register / Forgot Password

## Context
- Depends on Phase 1 (Better Auth + Prisma + Resend configured)
- [Brainstorm Report](../reports/brainstorm-260413-1553-auth-feature.md)
- Added in Validation Session 3 — 2026-04-13

## Overview
Email + password auth via Better Auth. Mandatory login before checkout. Admin shares same user system with role-based access. No email verification on register.

## Requirements
- Register: Họ tên + SĐT (optional) + Email + Password + Confirm password
- Login: Email + Password (no phone login)
- Forgot password: email reset link via Resend, expires 1h
- No email verification — register → login immediately
- Checkout requires session → redirect `/auth/login?redirect=/checkout`
- Admin: same user table, `role = "admin"`

## Architecture

```
/auth/login            → Email + Password form
/auth/register         → Họ tên, SĐT (opt), Email, Password form
/auth/forgot-password  → Enter email → send reset link
/auth/reset-password   → Enter new password (token from URL)

src/middleware.ts:
  /checkout, /orders, /profile → require session
  /admin/**                    → require role=admin
```

## Better Auth Config

```typescript
// src/lib/auth.ts
export const auth = betterAuth({
  database: prismaAdapter(db, { provider: 'postgresql' }),
  plugins: [
    emailAndPassword({
      enabled: true,
      requireEmailVerification: false,
      sendResetPassword: async ({ user, url }) => {
        await resend.emails.send({
          from: 'no-reply@yourshop.vn',
          to: user.email,
          subject: 'Đặt lại mật khẩu',
          html: `<a href="${url}">Đặt lại mật khẩu</a>`,
        })
      },
    }),
    admin(),
    // anonymous plugin removed
  ],
  trustedOrigins: [process.env.NEXT_PUBLIC_APP_URL!],
})
```

## DB Schema Changes

```prisma
// Additional fields on Better Auth's User model
// Add to schema after running `npx @better-auth/cli generate`
model User {
  // ... Better Auth auto fields ...
  phone   String?  // Contact info — not used for login
  role    String   @default("user") // "user" | "admin"
  orders  Order[]
}

// Order model: userId NOT NULL (mandatory login)
model Order {
  userId  String   // NOT NULL — was String?
  user    User     @relation(fields: [userId], references: [id])
  // sessionId removed
}
```

## Related Code Files

### Create
- `src/app/auth/layout.tsx` — centered card layout (max-w-md)
- `src/app/auth/login/page.tsx`
- `src/app/auth/register/page.tsx`
- `src/app/auth/forgot-password/page.tsx`
- `src/app/auth/reset-password/page.tsx`
- `src/components/auth/login-form.tsx`
- `src/components/auth/register-form.tsx`
- `src/components/auth/forgot-password-form.tsx`
- `src/components/auth/reset-password-form.tsx`
- `src/middleware.ts`

### Modify
- `src/lib/auth.ts` — swap anonymous → emailAndPassword + admin plugins
- `prisma/schema.prisma` — User.phone/role fields, Order.userId NOT NULL, remove Order.sessionId
- `src/app/layout.tsx` — header: show Login/Register or user session menu

## Implementation Steps

1. Update `src/lib/auth.ts`:
   - Remove `anonymous` plugin
   - Add `emailAndPassword({ requireEmailVerification: false, sendResetPassword })` 
   - Add `admin()` plugin
2. Update `prisma/schema.prisma`:
   - Add `phone String?`, `role String @default("user")` to User model
   - Change `Order.userId` → `String` (NOT NULL), add `@relation` to User
   - Remove `Order.sessionId`
3. Run `npx prisma migrate dev --name auth-mandatory-login`
4. Create `src/app/auth/layout.tsx` — white card, centered, responsive
5. `src/components/auth/login-form.tsx`:
   - React Hook Form: email + password (show/hide toggle)
   - Calls `authClient.signIn.email({ email, password })`
   - Error: "Email hoặc mật khẩu không đúng"
   - Link: Quên mật khẩu / Chưa có tài khoản
6. `src/components/auth/register-form.tsx`:
   - Fields: Họ tên*, SĐT (optional), Email*, Password*, Confirm Password*
   - Calls `authClient.signUp.email({ name, email, password })` + save phone via user update
   - Auto-redirect to homepage after register
7. `src/components/auth/forgot-password-form.tsx`:
   - Field: Email
   - Calls `authClient.forgetPassword({ email, redirectTo: '/auth/reset-password' })`
   - Show success message after submit (don't reveal if email exists)
8. `src/components/auth/reset-password-form.tsx`:
   - Fields: New Password, Confirm Password
   - Read `token` from `useSearchParams()`
   - Calls `authClient.resetPassword({ newPassword, token })`
9. Create page files wrapping each form component
10. `src/middleware.ts`:
    ```typescript
    import { getSessionCookie } from 'better-auth/cookies'
    const PROTECTED = ['/checkout', '/orders', '/profile']
    const ADMIN = ['/admin']
    export function middleware(request: NextRequest) {
      const session = getSessionCookie(request)
      const path = request.nextUrl.pathname
      if (PROTECTED.some(p => path.startsWith(p)) && !session) {
        const url = new URL('/auth/login', request.url)
        url.searchParams.set('redirect', path)
        return NextResponse.redirect(url)
      }
    }
    ```
11. Update header: conditional render — Login/Register links if no session, User dropdown if session

## Todo
- [ ] Update `src/lib/auth.ts` (emailAndPassword + admin, remove anonymous)
- [ ] Migrate Prisma schema (User phone/role, Order.userId NOT NULL, remove sessionId)
- [ ] Auth layout (centered card)
- [ ] Login form + page
- [ ] Register form + page
- [ ] Forgot password form + page
- [ ] Reset password form + page (token from URL)
- [ ] Middleware: protect /checkout, /orders, /profile, /admin
- [ ] Update header with session-aware nav
- [ ] E2E test: register → login → forgot → reset flow

## Success Criteria
- Register → auto-login, redirect to homepage
- Login valid → redirect to intended page (from `redirect` param)
- Login invalid → error message (không tiết lộ email có tồn tại không)
- Forgot password → email nhận trong <1 min → reset link hoạt động
- Reset password → can login với password mới
- `/checkout` without session → redirect `/auth/login?redirect=/checkout`
- `/admin` without role=admin → redirect

## Risk
- Better Auth schema: run `npx @better-auth/cli generate` để sync User model, không tự viết migration
- Resend `sendResetPassword` depends on Resend configured in Phase 1

## Security
- Password hashing: Better Auth built-in (bcrypt/argon2)
- CSRF: Better Auth built-in
- Reset token: 1h expiry (Better Auth default)
- Forgot password: always return success response (prevent email enumeration)
- Rate limit auth endpoints via Next.js middleware
