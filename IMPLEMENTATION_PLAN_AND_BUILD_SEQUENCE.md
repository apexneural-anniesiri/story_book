# Implementation Plan & Build Sequence

## Overview
- **Project**: ApexNeural Personalized Story Books Platform
- **MVP Target Date**: TBD
- **Approach**: Iterative, milestone-based, continuous testing

## Build Philosophy
- Documentation-first
- Test after every step
- Deploy to staging after each milestone
- Feedback loop after each milestone

## Phase 1: Project Setup & Foundation

### Step 1.1 — Monorepo/workspace initialization
- **Duration estimate**: 3 hours
- **Goal**: Create pnpm workspace with Next.js SSR frontend and NestJS REST backend.
- **Tasks (commands)**:
  1. ```bash
     mkdir -p apps/web apps/api packages/config
     ```
  2. ```bash
     pnpm init
     ```
  3. ```bash
     cat > pnpm-workspace.yaml <<'YAML'
     packages:
       - apps/*
       - packages/*
     YAML
     ```
  4. ```bash
     pnpm dlx create-next-app@14.2.5 apps/web --typescript --tailwind --app --eslint --use-pnpm
     ```
  5. ```bash
     pnpm add -g @nestjs/cli@10.4.2
     nest new apps/api --package-manager pnpm
     ```
  6. ```bash
     pnpm add -D concurrently@9.0.1 -w
     ```
  7. ```bash
     node -e "const fs=require('fs');const p='package.json';const j=JSON.parse(fs.readFileSync(p));j.packageManager='pnpm@9.0.0';j.scripts={...(j.scripts||{}),dev:'concurrently \"pnpm --filter web dev\" \"pnpm --filter api start:dev\"'};fs.writeFileSync(p,JSON.stringify(j,null,2));"
     ```
- **Success Criteria**:
  - [ ] `pnpm -w install` succeeds.
  - [ ] `pnpm dev` starts web and api processes.
  - [ ] `apps/web` runs with Next.js 14.2.5.
  - [ ] `apps/api` runs with NestJS 10.4.2.
- **References**:
  - TECH_STACK.md → Frontend Stack (Next.js/React/TypeScript/Tailwind)
  - TECH_STACK.md → Backend Stack (NestJS/Node)

### Step 1.2 — Lint/format/husky/lint-staged baseline
- **Duration estimate**: 2 hours
- **Goal**: Enforce formatting and code quality gates per toolchain.
- **Tasks (commands)**:
  1. ```bash
     pnpm add -D -w eslint@9.9.0 prettier@3.3.3 husky@9.1.4 lint-staged@15.2.8 @typescript-eslint/eslint-plugin@8.1.0 @typescript-eslint/parser@8.1.0
     ```
  2. ```bash
     pnpm dlx husky-init && pnpm install
     ```
  3. ```bash
     npx husky add .husky/pre-commit "pnpm lint-staged"
     ```
  4. ```bash
     cat > .lintstagedrc.json <<'JSON'
     {
       "*.{ts,tsx,js,jsx}": ["pnpm -r lint", "prettier --write"],
       "*.{json,md,yml,yaml}": ["prettier --write"]
     }
     JSON
     ```
- **Success Criteria**:
  - [ ] Pre-commit hook blocks bad formatting.
  - [ ] `pnpm -r lint` passes in both apps.
  - [ ] Prettier runs successfully.
- **References**:
  - TECH_STACK.md → Development Tools

### Step 1.3 — Environment variables and `.env.example`
- **Duration estimate**: 2 hours
- **Goal**: Standardize environment config across web/api.
- **Tasks (commands)**:
  1. ```bash
     mkdir -p apps/api/config apps/web/config
     ```
  2. ```bash
     cat > .env.example <<'ENV'
     DATABASE_URL=
     DIRECT_DATABASE_URL=
     PGBOUNCER_DATABASE_URL=
     JWT_ACCESS_SECRET=
     JWT_REFRESH_SECRET=
     JWT_ACCESS_TTL=15m
     JWT_REFRESH_TTL=30d
     WHATSAPP_CLOUD_API_VERSION=v20.0
     WHATSAPP_PHONE_NUMBER_ID=
     WHATSAPP_BUSINESS_ACCOUNT_ID=
     WHATSAPP_ACCESS_TOKEN=
     WHATSAPP_APP_SECRET=
     WHATSAPP_WEBHOOK_VERIFY_TOKEN=
     RAZORPAY_KEY_ID=
     RAZORPAY_KEY_SECRET=
     RAZORPAY_WEBHOOK_SECRET=
     PAYPAL_CLIENT_ID=
     PAYPAL_CLIENT_SECRET=
     PAYPAL_ENVIRONMENT=live
     PAYPAL_WEBHOOK_ID=
     AWS_REGION=ap-south-1
     AWS_ACCESS_KEY_ID=
     AWS_SECRET_ACCESS_KEY=
     AWS_S3_BUCKET_ASSETS=
     AWS_S3_BUCKET_PRIVATE=
     S3_PRESIGNED_UPLOAD_TTL_SECONDS=900
     RESEND_API_KEY=
     NEXT_PUBLIC_APP_URL=
     NEXT_PUBLIC_MARKETING_URL=
     API_BASE_URL=
     FEATURE_PRINT_SHIP=true
     FEATURE_WHATSAPP_FLOW=true
     FEATURE_AI_ASSISTED_SELECTION=true
     FEATURE_PREVIEW_WATERMARK=true
     ENV
     ```
  3. ```bash
     pnpm --filter api add zod@3.23.8
     ```
  4. ```bash
     # Add zod-based env parser in apps/api/src/config/env.ts and fail fast on boot
     ```
- **Success Criteria**:
  - [ ] API fails startup when required vars missing.
  - [ ] `.env.example` covers all required groups.
- **References**:
  - TECH_STACK.md → Environment Variables

### Step 1.4 — PostgreSQL + Prisma initialization + first migration
- **Duration estimate**: 4 hours
- **Goal**: Establish schema foundation with local users mapping and single-child constraint.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add @prisma/client@5.18.0
     pnpm --filter api add -D prisma@5.18.0
     ```
  2. ```bash
     pnpm --filter api prisma init
     ```
  3. ```bash
     # Implement Prisma schema with snake_case models/mappings from BACKEND_ARCHITECTURE_AND_DB_STRUCTURE.md
     ```
  4. ```bash
     pnpm --filter api prisma migrate dev --name init_storybook_core
     ```
  5. ```bash
     pnpm --filter api prisma generate
     ```
- **Success Criteria**:
  - [ ] Migration creates all core tables.
  - [ ] `child_profiles.user_id` unique constraint exists.
  - [ ] Prisma client generation passes.
- **References**:
  - BACKEND_ARCHITECTURE_AND_DB_STRUCTURE.md → Section 2 and Section 3

### Step 1.5 — Redis + S3 dev configuration + signed URL smoke test
- **Duration estimate**: 3 hours
- **Goal**: Validate cache and private asset flow.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add redis@4.7.0 @aws-sdk/client-s3@3.637.0 @aws-sdk/s3-request-presigner@3.637.0
     ```
  2. ```bash
     # Configure Redis client and S3 service in apps/api/src/infrastructure/
     ```
  3. ```bash
     pnpm --filter api add @nestjs/config@3.2.3
     ```
  4. ```bash
     # Implement POST /api/v1/assets/upload-url minimal route for smoke test
     ```
  5. ```bash
     curl -X POST http://localhost:3001/api/v1/assets/upload-url -H "Content-Type: application/json" -d '{"fileName":"child.png","mimeType":"image/png","sizeBytes":1000,"type":"CHILD_PHOTO"}'
     ```
- **Success Criteria**:
  - [ ] Redis connect/read/write works.
  - [ ] Presigned URL API returns URL + key + expiry.
  - [ ] No public S3 ACL usage.
- **References**:
  - TECH_STACK.md → Storage / Redis
  - BACKEND_ARCHITECTURE_AND_DB_STRUCTURE.md → Assets endpoints

## Phase 2: Backend Core (NestJS)

### Step 2.1 — Module scaffolding (modular monolith)
- **Duration estimate**: 3 hours
- **Goal**: Create module boundaries matching architecture.
- **Tasks (commands)**:
  1. ```bash
     cd apps/api
     nest g module auth-mapping
     nest g module child-profile
     nest g module series
     nest g module storyline
     nest g module books
     nest g module ai-generation
     nest g module preview-pdf
     nest g module payments
     nest g module assets
     nest g module chat
     nest g module audit
     nest g module admin-support
     ```
  2. ```bash
     mkdir -p src/common/{dto,guards,interceptors,filters,constants}
     ```
- **Files/folders created**:
  - `apps/api/src/<module>/*`
  - `apps/api/src/common/*`
- **Tests**:
  - Create smoke unit tests for each module bootstrap.
- **Success Criteria**:
  - [ ] API boots with all modules wired.
  - [ ] Module imports remain acyclic.
- **Cache invalidation keys**: N/A

### Step 2.2 — SaaS auth mapping + local session endpoints
- **Duration estimate**: 4 hours
- **Goal**: Implement `/auth/sync`, `/auth/session`, `/auth/logout`, `/auth/refresh` without building auth UI.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add jsonwebtoken@9.0.2 argon2@0.41.1
     pnpm --filter api add @nestjs/jwt@10.2.0
     ```
  2. Implement:
     - `src/auth-mapping/auth-mapping.controller.ts`
     - `src/auth-mapping/auth-mapping.service.ts`
     - `src/auth-mapping/dto/*.ts`
  3. Add `users` Prisma repository functions (upsert by `saas_user_id`).
  4. Add HttpOnly refresh cookie handling middleware/interceptor.
- **Tests**:
  - Unit: token issuance, refresh rotation, logout revocation.
  - Integration: `POST /api/v1/auth/sync` + session lifecycle.
- **Success Criteria**:
  - [ ] Local `users` record created/updated from SaaS identity.
  - [ ] Access + refresh behavior matches policy.
- **Cache invalidation keys**:
  - `session:{userId}:*` on logout/refresh revoke

### Step 2.3 — Child profile API with single-child enforcement
- **Duration estimate**: 3 hours
- **Goal**: Implement `GET /api/v1/child`, `POST /api/v1/child` with one-child-per-user rule.
- **Tasks (commands)**:
  1. Implement controller/service/repository in `src/child-profile/`.
  2. Enforce DB unique constraint (`child_profiles.user_id`) and service-level guard.
  3. Add validation DTOs (name/age/interests/photoAssetId).
- **Tests**:
  - Unit: update/upsert logic.
  - Integration: duplicate profile creation attempt returns 409.
- **Success Criteria**:
  - [ ] Exactly one child profile per user enforced.
  - [ ] API returns normalized child profile.
- **Cache invalidation keys**:
  - Invalidate `child:{userId}` and dependent `series:list:{childId}`.

### Step 2.4 — Series generation (exactly 5)
- **Duration estimate**: 4 hours
- **Goal**: Implement `POST /api/v1/series/generate` and `GET /api/v1/series`.
- **Tasks (commands)**:
  1. Implement service in `src/series/series.service.ts` to create exactly 5 records (`position` 1..5).
  2. Add generation policy checks and selection status defaults.
  3. Add AI log write for generation call (`ai_generation_logs`).
- **Tests**:
  - Unit: generator returns exactly 5 records.
  - Integration: `POST /series/generate` persists 5 rows with valid positions.
- **Success Criteria**:
  - [ ] Exactly 5 series enforced.
- **Cache invalidation keys**:
  - Invalidate `series:list:{childProfileId}`.

### Step 2.5 — Storyline generation (exactly 5 per series)
- **Duration estimate**: 4 hours
- **Goal**: Implement `POST /api/v1/series/:id/storylines/generate` and `GET /api/v1/series/:id/storylines`.
- **Tasks (commands)**:
  1. Build `src/storyline/storyline.service.ts` for exact-5 generation.
  2. Validate parent series ownership and status.
  3. Persist AI generation logs.
- **Tests**:
  - Unit: series->storylines count and position checks.
  - Integration: unauthorized series access blocked.
- **Success Criteria**:
  - [ ] Exactly 5 storylines per generation call.
- **Cache invalidation keys**:
  - Invalidate `storyline:list:{seriesId}`.

### Step 2.6 — Book lifecycle and status machine
- **Duration estimate**: 4 hours
- **Goal**: Implement `POST /books`, `GET /books`, `GET /books/:id` with transitions.
- **Tasks (commands)**:
  1. Implement book aggregate logic in `src/books/books.service.ts`.
  2. Add transition guard utility (`src/books/book-status-machine.ts`).
  3. Store `regeneration_count`, preview/final references.
- **Tests**:
  - Unit: allowed/disallowed transitions.
  - Integration: books list returns past books and statuses.
- **Success Criteria**:
  - [ ] State transitions follow DRAFT→...→DELIVERED sequence.
- **Cache invalidation keys**:
  - Invalidate `book:detail:{bookId}`, `books:list:{userId}:*`.

### Step 2.7 — AI generation module + audit fields
- **Duration estimate**: 3 hours
- **Goal**: Persist prompt versions/model params/input-output metadata for every generation.
- **Tasks (commands)**:
  1. Implement `src/ai-generation/ai-generation.service.ts`.
  2. Ensure image model validation: only DALL·E accepted.
  3. Write logs to `ai_generation_logs` for series/storylines/books/images.
- **Tests**:
  - Unit: reject non-DALL·E image model.
  - Integration: log row exists per generation operation.
- **Success Criteria**:
  - [ ] Prompt version + model params + duration persisted.
- **Cache invalidation keys**: N/A

### Step 2.8 — Assets module (S3 signed upload/download)
- **Duration estimate**: 3 hours
- **Goal**: Implement `/assets/upload-url` and `/assets/:id/download-url`.
- **Tasks (commands)**:
  1. Implement controller/service under `src/assets/`.
  2. Persist asset metadata in `image_assets` / `pdf_assets`.
  3. Enforce MIME/size validation.
- **Tests**:
  - Unit: MIME allowlist, size limit.
  - Integration: signed URL issuance and ownership check.
- **Success Criteria**:
  - [ ] Private bucket signed URLs only.
- **Cache invalidation keys**: N/A

### Step 2.9 — PDF preview/final generation pipeline
- **Duration estimate**: 4 hours
- **Goal**: Implement `POST /books/:id/generate`, `POST /books/:id/preview` and final PDF worker.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add @react-pdf/renderer@4.0.0 pdf-lib@1.17.1
     ```
  2. Build worker/service:
     - `src/preview-pdf/preview-pdf.service.ts`
     - `src/preview-pdf/final-pdf.service.ts`
  3. Add watermark step for preview assets.
  4. Save outputs in `pdf_assets` and update `books` references.
- **Tests**:
  - Unit: watermark application path.
  - Integration: preview blocked until generation complete; final blocked until paid.
- **Success Criteria**:
  - [ ] Preview is watermarked.
  - [ ] Final PDF not accessible pre-payment.
- **Cache invalidation keys**:
  - Invalidate `book:detail:{bookId}` after preview/final generation.

### Step 2.10 — Payments (Razorpay + PayPal) + webhooks + idempotency
- **Duration estimate**: 4 hours
- **Goal**: Implement create-order, verify, webhooks, and unlock flow.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add razorpay@2.9.4 @paypal/checkout-server-sdk@1.0.3
     ```
  2. Implement endpoints:
     - `/payments/razorpay/create-order`
     - `/payments/paypal/create-order`
     - `/payments/verify`
     - `/webhooks/razorpay`
     - `/webhooks/paypal`
     - `/books/:id/unlock`
  3. Add webhook signature verification and `webhook_events` persistence.
  4. Add idempotency key handling in Redis + DB unique event checks.
- **Tests**:
  - Unit: webhook signature verification, idempotency dedupe.
  - Integration: payment captured updates book to FINAL_READY/DELIVERED path.
- **Success Criteria**:
  - [ ] Duplicate webhook does not duplicate state transitions.
  - [ ] Payment capture unlocks final PDF only once.
- **Cache invalidation keys**:
  - Invalidate `book:detail:{bookId}`, `books:list:{userId}:*`, `payments:{bookId}`.

### Step 2.11 — Chat + audit logs + webhook events + standard errors/rate limits
- **Duration estimate**: 4 hours
- **Goal**: Complete resilience and traceability layer.
- **Tasks (commands)**:
  1. Implement chat endpoints in `src/chat/`.
  2. Implement audit append-only writes in `src/audit/`.
  3. Implement global exception filter for standard error envelope.
  4. ```bash
     pnpm --filter api add @nestjs/throttler@6.2.1 class-validator@0.14.1 class-transformer@0.5.1
     ```
  5. Configure throttler groups per endpoint category.
- **Tests**:
  - Unit: audit logging service, error envelope formatter.
  - Integration: rate limit returns 429 with expected error schema.
- **Success Criteria**:
  - [ ] Users can resume chat sessions.
  - [ ] Audit logs present for critical actions.
  - [ ] Global error envelope consistent.
- **Cache invalidation keys**:
  - `chat:session:{id}`, `books:list:{userId}:*` when chat triggers generation actions.

## Phase 3: Frontend (Next.js SSR)

### Step 3.1 — App router and protected route shell
- **Duration estimate**: 3 hours
- **Goal**: Build routing skeleton around SaaS-authenticated session handshake.
- **Tasks (commands)**:
  1. Create routes in `apps/web/app/`:
     - `/dashboard`
     - `/child`
     - `/series`
     - `/series/[id]/storylines`
     - `/books`
     - `/books/[id]`
     - `/books/[id]/preview`
     - `/books/[id]/checkout`
     - `/chat`
  2. Add server-side protected layout using SaaS session check + `/api/v1/auth/sync` handshake.
- **Success Criteria**:
  - [ ] Unauthenticated protected routes redirect to SaaS auth entry.
  - [ ] Authenticated users map to local user on first load.
- **References**:
  - APP_FLOW.md → Navigation Map / Screen Inventory

### Step 3.2 — Client data layer (TanStack Query + Axios)
- **Duration estimate**: 2 hours
- **Goal**: Standardize data fetching, mutation, and cache keys.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter web add @tanstack/react-query@5.51.24 axios@1.7.4
     ```
  2. Create query client provider and API client interceptors.
  3. Define query keys:
     - `['child']`
     - `['series', childProfileId]`
     - `['storylines', seriesId]`
     - `['books']`
     - `['book', bookId]`
- **Success Criteria**:
  - [ ] Mutations invalidate matching keys.
  - [ ] Request failures map to standard error UI.

### Step 3.3 — Child profile screen (single child)
- **Duration estimate**: 3 hours
- **Goal**: Implement child create/update flow only (no multi-child UI).
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter web add react-hook-form@7.52.2 zod@3.23.8
     ```
  2. Build child form with Zod schema and upload URL integration.
  3. Save through `POST /api/v1/child`.
- **Success Criteria**:
  - [ ] User can create/update exactly one child profile.
  - [ ] Validation and error states displayed.

### Step 3.4 — Series selection UI
- **Duration estimate**: 3 hours
- **Goal**: Display and generate exactly 5 series with manual/auto-select actions.
- **Tasks (commands)**:
  1. Implement generate button calling `/series/generate`.
  2. Render 5-card list with loading/empty/error states.
  3. Add select CTA leading to storyline screen.
- **Success Criteria**:
  - [ ] Exactly 5 items shown after generation.
  - [ ] Selected series persisted.

### Step 3.5 — Storyline selection UI
- **Duration estimate**: 3 hours
- **Goal**: Generate/select 5 storylines per series and start book draft.
- **Tasks (commands)**:
  1. Generate via `/series/:id/storylines/generate`.
  2. Select one storyline and create book via `POST /books`.
  3. Show regenerate CTA and round counter from book state.
- **Success Criteria**:
  - [ ] Exactly 5 storylines listed.
  - [ ] Book draft created from selected storyline.

### Step 3.6 — Book generation progress + preview viewer
- **Duration estimate**: 4 hours
- **Goal**: Show generation state and watermarked preview.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter web add react-pdf@9.1.1
     ```
  2. Build `/books/[id]` progress panel based on book status.
  3. Build `/books/[id]/preview` with watermark badge and gated actions.
- **Success Criteria**:
  - [ ] Preview view loads only when `PREVIEW_READY`.
  - [ ] Final download action hidden/disabled until paid.

### Step 3.7 — Checkout pages (Razorpay + PayPal)
- **Duration estimate**: 4 hours
- **Goal**: Implement payment initiation and status handling.
- **Tasks (commands)**:
  1. Build checkout form with fulfillment type (`PDF_ONLY` / `PRINT_SHIP`).
  2. For `PRINT_SHIP`, require shipping address.
  3. Integrate create-order endpoints and redirect/confirmation handling.
- **Success Criteria**:
  - [ ] Payment pending/success/failure states rendered.
  - [ ] Book unlock triggers after verified payment.

### Step 3.8 — Library + purchases + regeneration history + chat
- **Duration estimate**: 4 hours
- **Goal**: Provide complete user history visibility and repeat flow controls.
- **Tasks (commands)**:
  1. Build `/books` list with purchase status and counts.
  2. Build `/books/[id]` sections for regeneration history and payments.
  3. Build `/chat` start/resume session and message feed with “Generate another book” CTA.
- **Success Criteria**:
  - [ ] Past books visible.
  - [ ] Past purchases visible.
  - [ ] Regeneration history visible.

## Phase 4 (Optional): WhatsApp Channel

### Step 4.1 — WhatsApp Cloud webhook integration
- **Duration estimate**: 3 hours
- **Goal**: Receive WhatsApp events and map to chat sessions.
- **Tasks (commands)**:
  1. Implement webhook endpoint in NestJS (`/webhooks/whatsapp` optional internal route).
  2. Validate Meta signature and verify token.
  3. Store inbound events in `webhook_events` and `chat_messages`.
- **Success Criteria**:
  - [ ] Incoming WhatsApp message creates/updates chat session.

### Step 4.2 — WhatsApp user mapping and shared flow reuse
- **Duration estimate**: 3 hours
- **Goal**: Map WhatsApp identity to existing SaaS/local user and reuse same series/storyline/book APIs.
- **Tasks (commands)**:
  1. Add mapping resolver service in `chat` or `auth-mapping` module.
  2. Serialize chat workflow into `chat_sessions.state_json`.
  3. Trigger existing generation/payment APIs from chat commands.
- **Success Criteria**:
  - [ ] No parallel “special WhatsApp” business logic divergence.
  - [ ] Same status machine works for web + WhatsApp.

### Step 4.3 — Idempotency and rate limiting hardening for WhatsApp webhooks
- **Duration estimate**: 2 hours
- **Goal**: Prevent duplicate processing and abuse.
- **Tasks (commands)**:
  1. Add Redis idempotency key: `idempotency:webhook:whatsapp:{eventId}`.
  2. Add throttler policy for webhook endpoint.
  3. Add audit events for every processed WhatsApp action.
- **Success Criteria**:
  - [ ] Duplicate message delivery does not duplicate operations.

## Phase 5: Testing & Quality Gates

### Step 5.1 — Backend unit tests (Jest)
- **Duration estimate**: 4 hours
- **Goal**: Cover validators, state transitions, webhook verification, idempotency.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api test
     ```
  2. Add tests for:
     - Child single-profile rule
     - Series/storyline exact count generation
     - Book/payment status transitions
     - Webhook signature verification
     - DALL·E-only image model guard
- **Success Criteria**:
  - [ ] Critical domain logic has passing unit tests.

### Step 5.2 — Integration tests (Supertest + ephemeral DB/Redis)
- **Duration estimate**: 4 hours
- **Goal**: Verify end-to-end API behavior and side effects.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api add -D supertest@7.0.0
     ```
  2. Run integration suite with isolated test DB and Redis namespace.
  3. Validate cache invalidation and idempotency behavior.
- **Success Criteria**:
  - [ ] Auth sync → child → series → storyline → book → preview flow passes.
  - [ ] Duplicate webhook ignored safely.

### Step 5.3 — E2E tests (Playwright)
- **Duration estimate**: 4 hours
- **Goal**: Validate critical user journey (with payment mocked).
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter web add -D @playwright/test@1.46.0
     ```
  2. Implement E2E scenario:
     - create child
     - generate series
     - choose storyline
     - generate preview
     - mock payment success
     - unlock final
     - view library
     - regenerate
  3. ```bash
     pnpm test:e2e
     ```
- **Success Criteria**:
  - [ ] End-to-end happy path and regeneration path pass.

### Step 5.4 — CI gates (GitHub Actions)
- **Duration estimate**: 3 hours
- **Goal**: Enforce release-quality checks on PR.
- **Tasks (commands)**:
  1. Add `.github/workflows/ci.yml` with jobs:
     - install (`pnpm install --frozen-lockfile`)
     - lint
     - type-check
     - test
     - build
  2. Configure required checks in branch protection.
- **Success Criteria**:
  - [ ] PR cannot merge with failing checks.

## Phase 6: Deployment

### Step 6.1 — Staging infrastructure and deploy pipelines
- **Duration estimate**: 4 hours
- **Goal**: Deploy staging across fixed stack.
- **Tasks (commands)**:
  1. Provision:
     - Vercel (frontend)
     - AWS ECS Fargate (api)
     - RDS PostgreSQL 16
     - Upstash Redis
     - S3 private buckets
  2. Configure env vars and secret stores.
  3. Deploy from `develop` branch.
- **Success Criteria**:
  - [ ] Staging URLs healthy for web and API.
  - [ ] S3 signed URL and DB connectivity verified.

### Step 6.2 — Production readiness checklist + release
- **Duration estimate**: 4 hours
- **Goal**: Safe production launch with rollback capability.
- **Tasks (commands)**:
  1. ```bash
     pnpm --filter api prisma migrate deploy
     ```
  2. Deploy web to Vercel production and API to ECS Fargate.
  3. Enable Sentry + CloudWatch + pino log shipping.
  4. Run smoke tests on production.
- **Success Criteria**:
  - [ ] Preview gating and payment unlock verified.
  - [ ] Audit and webhook events visible in logs.

### Step 6.3 — Rollback plan drill
- **Duration estimate**: 2 hours
- **Goal**: Ensure recoverability for failed release.
- **Tasks (commands)**:
  1. Test rollback to previous ECS task definition.
  2. Validate RDS PITR restore runbook in staging.
  3. Re-run post-rollback smoke tests.
- **Success Criteria**:
  - [ ] Rollback procedure documented and validated.

## Milestones & Timeline

### Milestone 1: Foundation complete
- **Deliverables checklist**:
  - [ ] Monorepo and tooling initialized.
  - [ ] Env config and Prisma baseline complete.
  - [ ] Redis/S3 connectivity validated.
- **Smoke tests to run**:
  - `pnpm dev`
  - Prisma migration apply
  - Signed URL create request returns valid payload

### Milestone 2: Backend core complete
- **Deliverables checklist**:
  - [ ] Auth mapping, child, series, storyline, books, payments, assets, chat endpoints.
  - [ ] Status machines + idempotency + audit logs implemented.
  - [ ] Preview/final PDF pipeline available.
- **Smoke tests to run**:
  - API flow: auth sync → child → series/storyline generate → book → preview
  - Payment webhook replay does not duplicate unlock

### Milestone 3: Frontend core flows complete
- **Deliverables checklist**:
  - [ ] Protected SSR pages and local API handshake.
  - [ ] Child, series, storyline, preview, checkout, library, chat UI.
  - [ ] Loading/error/empty states done.
- **Smoke tests to run**:
  - Browser flow from child creation to preview and mock payment unlock
  - Library shows books/purchases/regeneration history

### Milestone 4: MVP launch
- **Deliverables checklist**:
  - [ ] Staging and production deployed.
  - [ ] CI gates active.
  - [ ] Observability and rollback plan validated.
- **Smoke tests to run**:
  - Production auth sync handshake
  - Payment success updates book to FINAL_READY/DELIVERED
  - Signed final PDF download URL accessible post-payment only

## Risk Mitigation

### Technical Risks
| Risk | Impact | Mitigation | Owner |
|---|---|---|---|
| Late schema changes break migrations | High | Freeze schema after Milestone 2; additive migrations only | Backend Lead |
| Webhook duplicates/out-of-order events | High | `webhook_events` unique keys + Redis idempotency + replay-safe handlers | Payments Engineer |
| Preview watermark bypass | High | Enforce S3 private storage + signed URLs + gated endpoint checks | Backend Lead |
| Payment idempotency gaps | High | Require `Idempotency-Key`, provider IDs unique constraints, retry-safe services | Payments Engineer |
| DALL·E model drift | Medium | Validate image model constant and reject non-DALL·E requests | AI Engineer |

### Timeline Risks
| Risk | Impact | Mitigation | Owner |
|---|---|---|---|
| Scope creep during frontend polish | Medium | Lock MVP scope to PRD P0 features | Product + Engineering |
| External dependency delays (payments/WhatsApp approvals) | High | Parallel mock integrations; keep Phase 4 optional | Tech Lead |
| Environment setup delays | Medium | Use scripted infra templates and checklist by Phase 1 | DevOps |
| Rework due to unclear acceptance tests | Medium | Define smoke tests per milestone and CI gate early | QA Lead |

## Success Criteria
- [ ] Local `users` mapping from SaaS auth is operational.
- [ ] Exactly one child profile per user enforced at DB and API layers.
- [ ] Series generation returns exactly 5 series.
- [ ] Storyline generation returns exactly 5 storylines per series.
- [ ] Book follows required lifecycle states and transitions.
- [ ] Preview PDF is watermarked and available before payment.
- [ ] Final PDF unlock happens only after verified payment capture.
- [ ] Past books, purchases, and regeneration history visible in library.
- [ ] Prompt versions and model parameters logged for every generation.
- [ ] DALL·E-only policy enforced for image generation.
- [ ] Webhook handlers are idempotent and auditable.
- [ ] CI gates pass (lint, type-check, tests, build) before deployment.

## Post-MVP Roadmap
- P1: Gentle reminders for pending series/storyline selection (no auto-selection).
- P1: Enhanced progress visibility across generation/payment/delivery stages.
- P1: Faster repeat purchase flow optimization from existing child profile.
- P2: Optional tone presets during plot review.
- P2: Occasion tagging for repeat books in library.
