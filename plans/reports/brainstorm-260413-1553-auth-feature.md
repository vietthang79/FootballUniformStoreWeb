---
name: "Auth Feature Brainstorm"
description: "Brainstorm report cho login/register/forgot password feature"
type: brainstorm
date: 2026-04-13
---

# Brainstorm Report — Auth Feature

## Problem Statement
Project hiện tại chỉ có Better Auth anonymous plugin (guest checkout). Cần bổ sung full auth flow: login, register, forgot password với email+password. Đây là thay đổi scope đáng kể vì chuyển từ "guest checkout" sang "bắt buộc login".

## Requirements

| Requirement | Decision |
|---|---|
| Login method | Email + password only |
| Register fields | Họ tên + SĐT (optional) + Email + Password + Confirm password |
| Forgot password | Email reset link qua Resend (đã có trong stack) |
| Email verification | Không — register xong login ngay |
| Guest checkout | Bỏ — bắt buộc login trước checkout |
| Admin auth | Chung hệ thống, phân quyền `role = "admin"` |
| Phone login | Không cần — email only |

## Evaluated Approaches

### Option A: Email-only Auth via Better Auth `emailAndPassword` plugin (Recommended)
**Pros:**
- Better Auth đã có trong tech stack — zero overhead adding this plugin
- emailAndPassword plugin hỗ trợ forgot/reset password built-in
- `admin` plugin quản lý role — DRY, không cần custom middleware phức tạp
- Remove anonymous plugin — simplifies auth config

**Cons:**
- Thay đổi Order schema (`userId` NOT NULL) — migration cần thiết

### Option B: NextAuth (Auth.js)
**Pros:** Popular, nhiều providers
**Cons:** Đã commit Better Auth, thêm dependency mới là YAGNI violation

### Option C: Custom JWT auth
**Cons:** Overkill, phải tự implement security. KISS violation.

**Verdict:** Option A — Better Auth `emailAndPassword` + `admin` plugin.

## Final Recommended Solution

### Better Auth Config
```typescript
betterAuth({
  plugins: [
    emailAndPassword({ requireEmailVerification: false, sendResetPassword: ... }),
    admin(),
    // anonymous removed
  ]
})
```

### Auth Pages
- `/auth/login` — Email + Password
- `/auth/register` — Họ tên + SĐT (optional) + Email + Password
- `/auth/forgot-password` — Enter email → Resend gửi link
- `/auth/reset-password` — Nhập mật khẩu mới (từ link email)

### DB Schema Changes
- User: thêm `phone String?`, `role String @default("user")`
- Order: `userId String` NOT NULL (bỏ nullable + bỏ `sessionId`)

### Route Protection (middleware.ts)
- Protected: `/checkout`, `/orders`, `/profile`
- Admin-only: `/admin/**`
- Public: tất cả còn lại

## Implementation Considerations

**Phase impact:**
- Phase 1: Update auth.ts, remove anonymous plugin, update Order schema
- NEW Phase 2 (Auth): 2d effort — toàn bộ auth pages + middleware
- Renumber phases 2→9 thành 3→9 (total: 9 phases, +2d effort)

**Checkout flow change:**
- Cũ: Guest checkout → anonymous session
- Mới: Require session → redirect `/auth/login?redirect=/checkout` nếu chưa login

## Success Metrics
- Register → login ngay (no verify step)
- Forgot password email nhận trong <1 min
- `/checkout` redirect đúng nếu chưa login
- Admin routes blocked cho non-admin

## Risks
- Better Auth schema auto-generation — test migration sớm
- `sendResetPassword` phụ thuộc Resend config từ Phase 1

## Next Steps
1. Update Phase 1 plan — remove anonymous plugin mention
2. Create Phase 2 plan file (Auth)
3. Renumber Phase 2-8 → 3-9
4. Update plan.md

---
*Session 3 — 2026-04-13 | Questions asked: 6*
