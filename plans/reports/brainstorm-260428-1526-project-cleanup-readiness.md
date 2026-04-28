# Brainstorm Report — Project Cleanup + Readiness Validation

**Date:** 2026-04-28
**Project:** FootballUniformStoreWeb
**Type:** Pre-implementation gap + cleanliness audit
**Session:** 11 (post-10-brainstorm freeze gate)

---

## Mục tiêu session

Sau 10 brainstorm session liên tiếp (26-04-12 → 26-04-17) → plan có dấu hiệu churn. Session này check:
1. Project có thực sự sạch sàng implement chưa?
2. Còn gap/mâu thuẫn nào trong plan không?
3. Có over-engineered không?
4. Đóng băng plan để implement.

---

## Vấn đề phát hiện

### A. Project chưa sạch (boilerplate residue)
- `docs/project-overview-pdr.md`, `docs/codebase-summary.md` còn nội dung **ClaudeKit Engineer** (boilerplate template), không phải football store
- `AGENTS.md` cũng boilerplate (OpenCode-specific, không liên quan dự án)
- `README.md` **không tồn tại** (CLAUDE.md yêu cầu đọc README.md trước khi implement)
- `prisma/` directory tạo sẵn nhưng rỗng → conflict với `npx create-next-app` + `npx prisma init` ở Phase 1
- `index.html` (1690 LOC) + `index-vi.html` (2158 LOC) ở root — Phase 1 step 0 chỉ nói xóa `index.html`

### B. Plan có mâu thuẫn nội tại
- **Phase 4 ↔ Phase 10 vòng tròn**: Phase 4 nói depends Phase 3 (product detail shell), nhưng Phase 3 đã trim chỉ còn homepage; product detail thuộc Phase 10
- **Cart sync mâu thuẫn**: Session 3 quyết mandatory login để checkout → Session 10 vẫn thêm `CartItem` table + localStorage + merge on login. Logic guest cart chỉ phục vụ "browse" → server cart không cần thiết
- **Phase 3 (1.5d) vs Phase 10 (6d)**: tách 2 nhưng đụng chung layout/header/cart icon

### C. Decision flip-flop history
- Builder lib: CSS overlay (S1) → Fabric.js (loại sớm) → react-draggable (S9) → react-moveable (S10) — chưa có POC
- Image angles: 4 góc cứng (S4) → free-form sortOrder (S8)
- ProductVariant FK: productId (S6) → colorSetId (S8)
- Phase 3 scope: full storefront → trim → trim → chỉ homepage

### D. Risk areas
- **Phase 4 builder (5d)**: react-moveable + multi-logo + % coords + dynamic tabs + ColorSet swap giữ overlay + Excel paste → rủi ro lớn nhất
- **Phase 8 CSV wizard (5d)**: nhiều UI cho MVP shop nhỏ
- **Phase 11 Scraper P1**: dev tool one-shot, không user-facing

---

## Quyết định confirm (10/10 questions)

### Round 1 — Cleanup
1. **[Cleanup]** Xóa sạch boilerplate docs + viết lại từ đầu — `docs/project-overview-pdr.md`, `docs/codebase-summary.md`, `AGENTS.md` đều bị xóa. README.md sẽ tạo mới từ context plan.md.
2. **[Cleanup]** `index.html` + `index-vi.html` → move sang `docs/references/` (giữ làm design reference, không xóa). Override Phase 1 step 0 "rm index.html".
3. **[Phase Structure]** Gộp Phase 3 + Phase 4 + Phase 10 → **Phase 3 "Storefront"** duy nhất. Lý do: cùng đụng layout/header/Zustand store/auth state, merge tránh handoff overhead + vòng tròn dependency.
4. **[Cart]** Bỏ `CartItem` table + `/api/cart/merge` endpoint. Guest dùng localStorage cho browse; checkout bắt login → submit cart từ localStorage thẳng vào `POST /api/orders`. Override Session 10 cart sync.

### Round 2 — Scope risk
5. **[Builder]** Tin react-moveable, không làm POC. Proceed Phase 4 (cũ) / Phase 3 (new merged) như planned.
6. **[Admin CSV]** Giữ wizard full 5d như plan (column mapping + manual upload images + manual stock per ColorSet × size).
7. **[Scraper]** Giữ P1 priority, chạy song song Phase 1-2 như plan.
8. **[Cloudinary]** Chấp nhận orphan logos cho MVP, monitor; cleanup cron nếu cần sau launch.

### Round 3 — Readiness
9. **[Plan freeze]** Đóng băng plan sau session này. Bắt đầu Phase 1.
10. **[Output]** Viết brainstorm report + update plan.md (Session 11 log + phase table + Key Architecture Decisions).

---

## Phase Impact

### plan.md
- Session 11 log appended
- Phase table 12 → 10 phases (gộp 3+4+10 → mới Phase 3, renumber 5-12 → 4-10)
- Key Architecture Decision #1 (cart sync) rewrite: bỏ DB sync
- Key Architecture Decision #10 (admin seed) giữ nguyên
- Total effort: 7w → ~6.8w (save 0.5d cart sync work)

### phase-01-project-setup.md
- Cleanup steps update:
  - Xóa `docs/project-overview-pdr.md`, `docs/codebase-summary.md`, `AGENTS.md`
  - `mv index.html index-vi.html` → `docs/references/`
  - Xóa `prisma/` rỗng (sẽ tạo lại bởi Next.js + Prisma init trong `apps/web/prisma/`)
  - Tạo README.md project-specific
- Schema: **Bỏ `CartItem` model** (cart sync đã loại)
- Bỏ step 14 phần "ADMIN_EMAIL/PASSWORD seed admin user" — KHÔNG, giữ vì Session 10 vẫn confirm

### phase-03 (cũ Homepage), phase-04 (cũ Builder), phase-10 (cũ Catalog)
- Đánh dấu **MERGED → phase-03-storefront.md**
- Implementation lead khi bắt đầu Phase 3 sẽ consolidate 3 file thành 1 file mới và xóa 3 file cũ

### phase-05 → phase-09 renumber
- phase-05-mockup-preview → phase-04
- phase-06-order-processing → phase-05
- phase-07-payment → phase-06
- phase-08-admin → phase-07
- phase-09-polish-deploy → phase-08
- phase-11-data-scraper → phase-09
- phase-12-payment-gateways → phase-10

> **Note:** File renumbering sẽ thực hiện cùng lúc khi consolidate phase 3+4+10 (start Phase 3 work). Hiện plan.md chỉ update reference link để không break trong lúc transition.

---

## Risk after Session 11

- Phase 3 (new merged) effort ~11d (3+1.5+6=10.5, làm tròn 11) — phase nặng nhất, cần stage tốt
- Builder không POC — accept risk, có fallback option (drag-only) nếu react-moveable lỗi edge case
- Admin CSV wizard 5d vẫn nặng — chấp nhận vì sau launch shop tự dùng

## Success Metrics

- Project root sạch sau cleanup: chỉ còn `apps/`, `packages/`, `docs/`, `plans/`, `.claude/`, config files (pnpm, .env.example, README.md, CLAUDE.md)
- `docs/` chỉ chứa file project-specific
- Plan freeze — không thêm Session 12+ trừ khi gặp blocker thực sự khi implement

## Next Steps

1. Update plan.md (Session 11 log + phase table + Key Architecture Decisions) — đang làm
2. Bắt đầu Phase 1: cleanup (xóa boilerplate docs + AGENTS.md, move HTML, xóa prisma/ rỗng, tạo README.md) → setup pnpm workspace → Next.js init → Prisma schema
3. Phase 11 Scraper start parallel with Phase 1 nếu có dev capacity

---

## Unresolved Questions

1. Admin user creation: Session 10 quyết "seed script đọc ENV" nhưng Session 7 cũ nói "register thường + update role manual". Nên xác nhận lần cuối trước Phase 1 (default: theo Session 10 vì mới nhất).
2. Phase 5 (Admin Preview, 1d): Session 5 nói "MockupCanvas đã build trong Phase 4, Phase 5 chỉ integrate read-only". Effort 1d có lẽ vẫn over — cân nhắc gộp vào Phase 7 (Admin) khi implement.
3. Brand colors trong index-vi.html có khớp 100% với plan (#FDD017, #E31E26, #00AEEF, #A7A9AC) không? Sẽ verify khi setup theme.
