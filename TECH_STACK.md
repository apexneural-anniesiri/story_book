# Technology Stack Documentation

## 1. Stack Overview
- **Last Updated**: 2026-02-17
- **Version**: 1.0
- **Architecture Pattern**: Modular monolith (domain modules) with API-first backend, event-assisted background workers, and managed cloud deployment (Vercel for web app + AWS-managed services for backend data/storage + managed Redis).

This architecture is chosen to keep V1 delivery fast while preserving clean module boundaries for growth. It supports the required dual-channel flow (Website + WhatsApp), strict preview-before-payment gating, secure storage of child photos/PDF assets, and auditable user/book history without premature microservice complexity.

### Architecture Pattern
- **Type:** Modular Monolith
- **Pattern:** API-first REST + domain modules + background workers
- **Deployment:** Cloud (Vercel + AWS + Upstash)

## 2. Frontend Stack

### Core framework
- **Name + Version**: Next.js 14.2.5
- **Reason**: Supports hybrid rendering, route protection, dashboard UX, SEO-friendly marketing pages, and smooth integration with authenticated app pages.
- **Docs**: https://nextjs.org/docs
- **License**: MIT
- **Alternatives considered**:
  - Remix (rejected: smaller ecosystem fit for this team’s planned hiring profile).
  - Nuxt (rejected: non-React stack mismatch with internal frontend capability).

### UI library
- **Name + Version**: React 18.2.0
- **Reason**: Stable ecosystem for dashboard-heavy app, strong compatibility with Next.js and ecosystem libraries.
- **Docs**: https://react.dev/
- **License**: MIT
- **Alternatives considered**:
  - Preact (rejected: compatibility edge cases with enterprise packages and monitoring tooling).

### Language + strictness
- **Name + Version**: TypeScript 5.5.4
- **Reason**: Type safety across auth, checkout, and document-status state transitions.
- **Docs**: https://www.typescriptlang.org/docs/
- **License**: Apache-2.0
- **tsconfig strict**: `"strict": true` (required)
- **Alternatives considered**:
  - JavaScript only (rejected: higher defect risk in complex stateful flows).

### UI components strategy
- **Name + Version**: shadcn/ui (registry snapshot with Radix UI primitives 1.1.1)
- **Reason**: Fast, customizable, design-system-friendly components without heavy runtime overhead.
- **Docs**: https://ui.shadcn.com/docs
- **License**: MIT
- **Alternatives considered**:
  - Material UI 5.16.7 (rejected: heavier visual opinion and bundle tradeoff for this brand style).

### Styling
- **Name + Version**: Tailwind CSS 3.4.10
- **Reason**: Rapid implementation of professional responsive layouts for marketing + app surfaces.
- **Docs**: https://tailwindcss.com/docs
- **License**: MIT
- **Alternatives considered**:
  - CSS Modules only (rejected: slower consistent scaling across many screens/states).

### State management
- **Name + Version**: TanStack Query (React Query) 5.51.24
- **Reason**: Server-state heavy product (books, children, preview/payment status) benefits from caching, retries, invalidation, and background refresh.
- **Docs**: https://tanstack.com/query/latest/docs/framework/react/overview
- **License**: MIT
- **Alternatives considered**:
  - Zustand 4.5.4 (rejected as primary: better for local UI state, weaker built-in server-state lifecycle handling).

### Forms + validation
- **Name + Version**: React Hook Form 7.52.2 + Zod 3.23.8
- **Reason**: High-performance forms with explicit schema validation for auth, child data capture, and checkout.
- **Docs**:
  - https://react-hook-form.com/docs
  - https://zod.dev/
- **License**:
  - React Hook Form: MIT
  - Zod: MIT
- **Alternatives considered**:
  - Formik + Yup (rejected: heavier rerender profile and less preferred TS ergonomics).

### HTTP client
- **Name + Version**: Axios 1.7.4
- **Reason**: Predictable interceptors for auth/session handling, error normalization, and request cancellation.
- **Docs**: https://axios-http.com/docs/intro
- **License**: MIT
- **Alternatives considered**:
  - Native fetch (rejected: more boilerplate for interceptors and consistent error shaping).

### Auth UI pages
- **Pages covered**: Login, Signup, Forgot Password, Reset Password, Change Password
- **Frontend implementation basis**: Next.js route groups + React Hook Form + Zod validation + Axios calls to SaaS auth endpoints.
- **Reason**: Required user journeys for website and WhatsApp account mapping.

### PDF preview viewer
- **Name + Version**: react-pdf 9.1.1 (uses pdf.js 4.4.168)
- **Reason**: In-browser rendering of watermarked preview with pagination/zoom.
- **Docs**:
  - https://github.com/wojtekmaj/react-pdf
  - https://mozilla.github.io/pdf.js/
- **License**:
  - react-pdf: MIT
  - pdf.js: Apache-2.0
- **Alternatives considered**:
  - Native `<iframe>` PDF rendering (rejected: inconsistent UX/control across browsers).

### File upload client approach
- **Approach**: S3 presigned URL uploads
- **Reason**: Keeps backend stateless for large files, reduces API bandwidth pressure, and improves upload reliability for child photos and PDF assets.
- **Alternatives considered**:
  - Direct multipart upload through app API (rejected: increased backend load and latency risk).

### Frontend error monitoring
- **Name + Version**: @sentry/nextjs 8.20.0
- **Reason**: Production-grade error tracing for app pages, checkout failures, and preview rendering issues.
- **Docs**: https://docs.sentry.io/platforms/javascript/guides/nextjs/
- **License**: MIT
- **Alternatives considered**:
  - LogRocket-only approach (rejected: weaker backend correlation for full-stack incidents).

## 3. Backend Stack

### Runtime
- **Name + Version**: Node.js 20.16.0 (LTS)
- **Reason**: Stable LTS runtime with strong ecosystem support for payments, WhatsApp, and file workflows.
- **Docs**: https://nodejs.org/docs/latest-v20.x/api/
- **License**: MIT
- **Alternatives considered**:
  - Node.js 22.x (rejected for V1 baseline due to broader dependency certification lag).

### Framework
- **Name + Version**: NestJS 10.4.2
- **Reason**: Structured module architecture, validation pipeline support, background job integration, and maintainability for multi-domain flows.
- **Docs**: https://docs.nestjs.com/
- **License**: MIT
- **Alternatives considered**:
  - Express 4.19.2 (rejected: less opinionated architecture for growing team complexity).
  - Fastify 4.28.1 (rejected as primary framework choice due to lower internal familiarity vs NestJS).

### Database
- **Name + Version**: PostgreSQL 16.4
- **Reason**: Reliable relational consistency for account, child, book, payment, and audit history records.
- **Docs**: https://www.postgresql.org/docs/16/index.html
- **License**: PostgreSQL License
- **Alternatives considered**:
  - MySQL 8.4 (rejected: team preference for PostgreSQL ecosystem and JSON/query ergonomics).

### ORM
- **Name + Version**: Prisma 5.18.0
- **Reason**: Type-safe data access, migration workflows, and strong DX for modular backend.
- **Docs**: https://www.prisma.io/docs
- **License**: Apache-2.0
- **Alternatives considered**:
  - TypeORM 0.3.20 (rejected: team preference for Prisma migration and client ergonomics).

### Redis
- **Name + Version**: Redis 7.2.5
- **Reason**: Session/cache acceleration, rate-limit counters, webhook idempotency, and short-lived workflow state.
- **Docs**: https://redis.io/docs/latest/
- **License**: RSALv2/SSPLv1 (source-available distribution terms)
- **Alternatives considered**:
  - In-memory node cache (rejected: not durable/scalable for multi-instance deployment).

### Auth strategy
- **Strategy**: Email+Password with short-lived JWT access token + rotating refresh token
- **Hashing**: Argon2id via argon2 0.41.1
- **Password policy**: minimum 12 chars, complexity checks, breached-password blocklist (TBD source)
- **Reason**: Supports website auth pages and WhatsApp mapping flows while enabling auditable session control.
- **Docs**:
  - JWT: https://www.rfc-editor.org/rfc/rfc7519
  - argon2 package: https://github.com/ranisalt/node-argon2
- **License**:
  - argon2: MIT
- **Alternatives considered**:
  - Server sessions only (rejected: more operational overhead across channels and APIs).
  - bcrypt 5.1.1 (rejected in favor of Argon2id memory-hard properties).

### WhatsApp integration
- **Approach**: Meta WhatsApp Cloud API
- **Reason**: Direct integration path, no BSP lock-in, predictable webhook control for account mapping and flow state.
- **Docs**: https://developers.facebook.com/docs/whatsapp/cloud-api
- **License**: Platform service terms
- **Alternatives considered**:
  - Twilio WhatsApp API (rejected: additional intermediary cost/control layer).

### Payments
- **Razorpay integration**: razorpay 2.9.4
- **PayPal integration**: @paypal/checkout-server-sdk 1.0.3
- **Reason**: Required dual-gateway support for PDF-only and Print+Ship checkout.
- **Docs**:
  - https://razorpay.com/docs/
  - https://developer.paypal.com/docs/api/overview/
- **License**:
  - razorpay: MIT
  - @paypal/checkout-server-sdk: Apache-2.0
- **Alternatives considered**:
  - Raw REST only (rejected: extra signature/webhook boilerplate and lower maintainability).

### Storage
- **Primary storage**: AWS S3
- **SDK**: @aws-sdk/client-s3 3.637.0, @aws-sdk/s3-request-presigner 3.637.0
- **Optional CDN**: Amazon CloudFront
- **Reason**: Durable object storage for child photos, watermarked previews, and final PDFs with signed access.
- **Docs**:
  - https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html
  - https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/
  - https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html
- **License**: Apache-2.0 (AWS SDK for JavaScript)
- **Alternatives considered**:
  - Cloudinary-only storage (rejected: less aligned for long-term document archive/access controls).

### Email delivery
- **Provider SDK**: resend 4.0.0
- **Template engine**: @react-email/components 0.0.22
- **Reason**: Transactional email for final PDF availability and support notifications.
- **Docs**:
  - https://resend.com/docs
  - https://react.email/docs
- **License**:
  - resend: MIT
  - @react-email/components: MIT
- **Alternatives considered**:
  - SendGrid 8.x (rejected: team preference for simpler API ergonomics in V1).

### PDF generation pipeline
- **Renderer**: @react-pdf/renderer 4.0.0
- **PDF utility**: pdf-lib 1.17.1
- **Reason**: Deterministic server-side composition of preview and final PDF outputs, including watermark pipeline.
- **Docs**:
  - https://react-pdf.org/
  - https://pdf-lib.js.org/
- **License**:
  - @react-pdf/renderer: MIT
  - pdf-lib: MIT
- **Alternatives considered**:
  - Puppeteer HTML-to-PDF 22.13.0 (rejected: higher runtime overhead for this pipeline).

### Logging
- **Name + Version**: pino 9.3.2
- **Reason**: High-throughput structured logging for request traces, payment events, and audit workflows.
- **Docs**: https://getpino.io/#/
- **License**: MIT
- **Alternatives considered**:
  - winston 3.13.0 (rejected: lower throughput profile for high-volume JSON logging).

### API validation
- **Name + Version**: class-validator 0.14.1 + class-transformer 0.5.1
- **Reason**: Native fit for NestJS DTO validation pipelines.
- **Docs**:
  - https://github.com/typestack/class-validator
  - https://github.com/typestack/class-transformer
- **License**: MIT
- **Alternatives considered**:
  - Joi 17.13.3 (rejected: less idiomatic with NestJS decorators).

### Rate limiting
- **Name + Version**: @nestjs/throttler 6.2.1
- **Reason**: Route-level and channel-level abuse protection (auth endpoints, upload requests, webhook endpoints).
- **Docs**: https://docs.nestjs.com/security/rate-limiting
- **License**: MIT
- **Alternatives considered**:
  - express-rate-limit 7.4.0 (rejected: not primary fit for NestJS module architecture).

### Database Schema Strategy
- **Migrations**: Prisma Migrate, committed in source control, required for every schema change.
- **Seeding**: Prisma seed scripts for baseline app config (policy pages metadata, feature flags defaults, static templates metadata).
- **Backup policy**:
  - Automated daily full backups.
  - Point-in-time recovery enabled.
  - Backup retention: 30 days minimum.
- **Connection pooling**: Managed PgBouncer (or cloud-managed pooling equivalent) required for production.

## 4. DevOps & Infrastructure

### Source Control
- **Git + GitHub** for repository hosting, protected branches, and release traceability.

### Branch Strategy
- `main`: production branch (protected).
- `develop`: integration branch.
- `feature/*`, `bugfix/*`, `hotfix/*` branches.
- PR review required before merge.

### CI/CD (GitHub Actions)
- **On Pull Request**:
  - Install dependencies (frozen lockfile).
  - Lint + format check.
  - Type check.
  - Unit tests.
  - Integration tests (API + DB via ephemeral test DB).
  - Build frontend and backend artifacts.
- **Deploy rules**:
  - Merge to `develop` deploys to staging.
  - Merge to `main` deploys to production after required checks pass.
  - Production deployment requires successful migration dry-run and smoke checks.

### Hosting Decisions
- **Frontend hosting**: Vercel
  - **Reason**: First-class Next.js deployment, preview environments per PR, edge delivery performance.
  - **Docs**: https://vercel.com/docs
- **Backend hosting**: AWS ECS Fargate
  - **Reason**: Managed container orchestration, predictable scaling, strong VPC/security control for payment and PII workloads.
  - **Docs**: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html
- **DB hosting**: Amazon RDS for PostgreSQL 16
  - **Reason**: Managed backups, PITR, high availability options.
  - **Docs**: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html
- **Redis hosting**: Upstash Redis
  - **Reason**: Managed Redis with simple ops for rate-limits/cache and cost-efficient scale.
  - **Docs**: https://docs.upstash.com/redis

### Monitoring & Observability
- **Sentry frontend**: @sentry/nextjs 8.20.0
- **Sentry backend**: @sentry/node 8.20.0
- **Metrics/logs**: CloudWatch metrics/log groups + structured JSON logs from pino.
- **Docs**:
  - https://docs.sentry.io/platforms/javascript/guides/node/
  - https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html

### Testing Stack
- **Unit tests**: Vitest 2.0.5 (frontend), Jest 29.7.0 (backend)
- **Integration tests**: Supertest 7.0.0 + Jest 29.7.0
- **E2E tests**: Playwright 1.46.0
- **Reason**: Balanced coverage across UI flows, API behavior, and end-to-end preview/payment gating.
- **Docs**:
  - https://vitest.dev/
  - https://jestjs.io/docs/getting-started
  - https://github.com/ladjs/supertest
  - https://playwright.dev/docs/intro

### Infrastructure Security Posture
- HTTPS only (TLS 1.2+), HSTS enabled.
- Secrets managed via cloud secret manager; no plaintext secrets in repo.
- IAM least-privilege roles for S3, DB, queue, and messaging access.
- Restricted network access for DB/Redis (private networking where available).

## 5. Development Tools

- **ESLint**: 9.9.0
  - **Config**: eslint-config-next 14.2.5 + @typescript-eslint/eslint-plugin 8.1.0 + @typescript-eslint/parser 8.1.0
  - **Docs**: https://eslint.org/docs/latest/
- **Prettier**: 3.3.3
  - **Docs**: https://prettier.io/docs/en/
- **Husky**: 9.1.4
  - **Docs**: https://typicode.github.io/husky/
- **lint-staged**: 15.2.8
  - **Docs**: https://github.com/lint-staged/lint-staged
- **Recommended IDE**: Visual Studio Code 1.92.2
  - **Recommended extensions**:
    - ESLint (dbaeumer.vscode-eslint)
    - Prettier - Code formatter (esbenp.prettier-vscode)
    - Prisma (Prisma.prisma)
    - Tailwind CSS IntelliSense (bradlc.vscode-tailwindcss)
    - Playwright Test for VSCode (ms-playwright.playwright)

### Package Manager
- **pnpm:** 9.x (exact version in root `package.json` `packageManager` field)
- **Enforced via:**
  - `"packageManager": "pnpm@X.X.X"` in root `package.json`
  - CI uses `--frozen-lockfile`

## 5b. Setup & Installation (Local Development)

### Prerequisites
- **Node.js:** 20.16.0 (LTS) — exact version required
- **pnpm:** Exact version as in root `package.json` `packageManager` field (9.x)
- **Docker:** Optional (for local PostgreSQL/Redis if not using cloud instances)

### Install
```bash
pnpm install --frozen-lockfile
```

### Dev run
```bash
pnpm dev
```
(Runs `concurrently "pnpm --filter web dev" "pnpm --filter api start:dev"`)

### Database migrate + seed
```bash
pnpm db:migrate
pnpm db:seed
```
(Or: `pnpm db:generate` before first run; `pnpm db:studio` for Prisma Studio.)

### Test commands
```bash
pnpm test
pnpm test:e2e
```

## 6. Environment Variables

```bash
# Database
DATABASE_URL=postgresql://<user>:<password>@<host>:5432/<db>?schema=public
DIRECT_DATABASE_URL=postgresql://<user>:<password>@<host>:5432/<db>
PGBOUNCER_DATABASE_URL=postgresql://<user>:<password>@<pool-host>:6432/<db>

# Auth
JWT_ACCESS_SECRET=<strong-random-secret>
JWT_REFRESH_SECRET=<strong-random-secret>
JWT_ACCESS_TTL=15m
JWT_REFRESH_TTL=30d
ARGON2_TIME_COST=3
ARGON2_MEMORY_COST=65536
ARGON2_PARALLELISM=1
SESSION_COOKIE_NAME=apex_session
SESSION_COOKIE_DOMAIN=.apexneural.com
SESSION_COOKIE_SECURE=true
SESSION_COOKIE_SAMESITE=lax

# WhatsApp
WHATSAPP_CLOUD_API_VERSION=v20.0
WHATSAPP_PHONE_NUMBER_ID=<meta-phone-number-id>
WHATSAPP_BUSINESS_ACCOUNT_ID=<meta-business-account-id>
WHATSAPP_ACCESS_TOKEN=<meta-system-user-token>
WHATSAPP_APP_SECRET=<meta-app-secret>
WHATSAPP_WEBHOOK_VERIFY_TOKEN=<verify-token>

# Payments
RAZORPAY_KEY_ID=<razorpay-key-id>
RAZORPAY_KEY_SECRET=<razorpay-key-secret>
RAZORPAY_WEBHOOK_SECRET=<razorpay-webhook-secret>
PAYPAL_CLIENT_ID=<paypal-client-id>
PAYPAL_CLIENT_SECRET=<paypal-client-secret>
PAYPAL_ENVIRONMENT=live
PAYPAL_WEBHOOK_ID=<paypal-webhook-id>

# Storage
AWS_REGION=ap-south-1
AWS_ACCESS_KEY_ID=<aws-access-key>
AWS_SECRET_ACCESS_KEY=<aws-secret-key>
AWS_S3_BUCKET_ASSETS=<bucket-name>
AWS_S3_BUCKET_PRIVATE=<bucket-name>
S3_PRESIGNED_UPLOAD_TTL_SECONDS=900
CLOUDFRONT_DISTRIBUTION_DOMAIN=<distribution-domain>

# Email
RESEND_API_KEY=<resend-api-key>
EMAIL_FROM=no-reply@apexneural.com
EMAIL_REPLY_TO=support@apexneural.com

# App URLs
NEXT_PUBLIC_APP_URL=https://app.apexneural.com
NEXT_PUBLIC_MARKETING_URL=https://www.apexneural.com
API_BASE_URL=https://api.apexneural.com
WHATSAPP_DEEP_LINK=https://wa.me/<number>
WEBHOOK_BASE_URL=https://api.apexneural.com/webhooks

# Feature flags
FEATURE_PRINT_SHIP=true
FEATURE_WHATSAPP_FLOW=true
FEATURE_AI_ASSISTED_SELECTION=true
FEATURE_PREVIEW_WATERMARK=true
FEATURE_EMAIL_RESEND=true
```

## 7. Package.json Scripts

```json
{
  "scripts": {
    "dev": "concurrently \"pnpm --filter web dev\" \"pnpm --filter api start:dev\"",
    "build": "pnpm -r build",
    "start": "pnpm --filter api start:prod",
    "lint": "pnpm -r lint",
    "format": "prettier --check .",
    "type-check": "pnpm -r type-check",
    "test": "pnpm -r test",
    "test:e2e": "pnpm --filter e2e test",
    "db:migrate": "pnpm --filter api prisma migrate deploy",
    "db:generate": "pnpm --filter api prisma generate",
    "db:studio": "pnpm --filter api prisma studio",
    "db:seed": "pnpm --filter api prisma db seed"
  }
}
```

## 8. Dependencies Lock

### Root / workspace scripts dependencies (exact versions)
```json
{
  "concurrently": "9.0.1"
}
```

### Frontend dependencies (exact versions)
```json
{
  "next": "14.2.5",
  "react": "18.2.0",
  "react-dom": "18.2.0",
  "typescript": "5.5.4",
  "tailwindcss": "3.4.10",
  "@tanstack/react-query": "5.51.24",
  "react-hook-form": "7.52.2",
  "zod": "3.23.8",
  "axios": "1.7.4",
  "react-pdf": "9.1.1",
  "@sentry/nextjs": "8.20.0",
  "@radix-ui/react-dialog": "1.1.1",
  "@radix-ui/react-dropdown-menu": "2.1.1"
}
```

### Backend dependencies (exact versions)
```json
{
  "@nestjs/common": "10.4.2",
  "@nestjs/core": "10.4.2",
  "@nestjs/platform-express": "10.4.2",
  "@nestjs/throttler": "6.2.1",
  "@prisma/client": "5.18.0",
  "prisma": "5.18.0",
  "pg": "8.12.0",
  "redis": "4.7.0",
  "argon2": "0.41.1",
  "jsonwebtoken": "9.0.2",
  "razorpay": "2.9.4",
  "@paypal/checkout-server-sdk": "1.0.3",
  "@aws-sdk/client-s3": "3.637.0",
  "@aws-sdk/s3-request-presigner": "3.637.0",
  "resend": "4.0.0",
  "@react-email/components": "0.0.22",
  "@react-pdf/renderer": "4.0.0",
  "pdf-lib": "1.17.1",
  "pino": "9.3.2",
  "class-validator": "0.14.1",
  "class-transformer": "0.5.1",
  "@sentry/node": "8.20.0"
}
```

## 9. Security Considerations

- **Password hashing policy**:
  - Argon2id (`argon2` 0.41.1)
  - Parameters: timeCost=3, memoryCost=65536, parallelism=1 (baseline; reviewed quarterly)
- **Token/session expiry policy**:
  - Access token: 15 minutes
  - Refresh token: 30 days, rotating, revocable
  - Forced logout on password reset/change and suspicious activity
- **HTTP-only cookies + CSRF stance**:
  - Refresh token stored in HttpOnly, Secure, SameSite=Lax cookie
  - CSRF protection enforced on state-changing routes using anti-CSRF token pattern
- **CORS rules**:
  - Allowlist-only origins (`app.apexneural.com`, `www.apexneural.com`)
  - Credentials enabled only for trusted origins
- **Rate limiting thresholds**:
  - Auth endpoints: 10 requests / minute / IP
  - WhatsApp webhook endpoints: 120 requests / minute / source
  - Upload init endpoints: 20 requests / minute / user
  - Checkout initiation: 10 requests / 10 minutes / user
- **Upload limits + scanning**:
  - Max photo upload size: 10 MB
  - Accepted types: JPG/JPEG/PNG
  - Virus/malware scanning required before marking asset active (scanner engine TBD)
- **PII handling (child photos)**:
  - Encryption at rest (S3 SSE-KMS)
  - Signed URL access only; private bucket for originals and final PDFs
  - Role-based access controls for admin/support
  - Data access logs retained for audit
- **Audit logging for orders/books history**:
  - Immutable audit events for: selection method, regeneration rounds, preview generated, payment attempts/results, final unlock, email send/retry
  - Correlation IDs across web + WhatsApp + payment webhook events

## 10. Version Upgrade Policy

- **Major upgrade schedule**:
  - Review core framework/runtime major versions quarterly.
  - Adopt within 1 quarter after ecosystem stability validation.
- **Patch/security cadence**:
  - Security patches: apply within 7 days.
  - Non-security patch updates: biweekly maintenance window.
- **Dependabot policy**:
  - Enabled for npm/github-actions.
  - Auto-PR for patch/minor updates; major updates require manual approval.
- **Staging validation + rollback**:
  - All upgrades first deployed to staging.
  - Required checks: lint, type-check, tests, smoke test for auth/preview/payment/delivery.
  - Production rollback via previous immutable deployment artifact and migration rollback plan.
- **Changelog requirement**:
  - Every release must include user-impact summary, dependency changes, migration notes, and security notes.

## 11. Compatibility Requirements

- **Node.js:** 20.16.0 (LTS) required; other versions not supported for build/run.
- **PostgreSQL:** 16.4 required for schema and features in use.
- **Redis:** 7.2.5 required (or compatible 7.x); used for session, cache, rate-limit, idempotency.
- **Browser support:** Modern evergreen browsers (Chrome, Firefox, Safari, Edge — last 2 major versions); no IE.
- **Minimum TLS:** TLS 1.2+ required for all external connections (HTTPS only; HSTS enabled).

## 12. AI Generation Pipeline Governance

- All prompts are stored in a version-controlled directory (e.g. `/apps/api/src/ai/prompts/`).
- Each generation stores:
  - model name
  - prompt version
  - temperature
  - regeneration round
  - timestamp
- Series generation prompt is versioned separately from plot generation prompt.
- Image generation prompt is versioned separately.
- Story script generation prompt is versioned separately.
- Prompt changes require a version bump.
- Prompt versions are stored in the DB with each book record.

### Image Generation Model Lock

- **Provider:** OpenAI
- **Model:** DALL·E (exact model version configured via environment variable)
- **Use Case:** Story illustration generation (cover + internal pages)
- Image generation is **NOT** allowed to switch to any other provider or open-source model without an explicit, versioned architectural change.

#### Prompt storage path
- `/apps/api/src/ai/prompts/image/`

#### Versioning rules
- Every image prompt must have a version number.
- Prompt version must be stored in DB with:
  - model name
  - prompt version
  - regeneration round
  - timestamp

#### Image size standards
- **Cover:** 1748x2480 (A5 print format)
- **Internal illustrations:** 1024x1024 or 1536x1536
- **DPI target for print:** 300 DPI

#### Regeneration policy
- Maximum regeneration attempts: 3
- Each regeneration must increment `regeneration_round`
- All generations must be audit logged

#### Negative prompt enforcement
Image prompts must explicitly prevent:
- distorted hands
- extra fingers
- text artifacts
- watermark overlays
- deformed facial proportions

#### Determinism policy
- Temperature must be fixed and defined in backend config.
- Model name must be stored in environment variable (e.g. `OPENAI_IMAGE_MODEL`).
- Any model change requires `TECH_STACK.md` version bump.

## Appendix: Assumptions & TBD

- `shadcn/ui` is treated as a component strategy with a fixed internal registry snapshot; exact generated component manifest may vary by project initialization.
- Breached-password blocklist provider/source is TBD.
- Duplicate payment callback idempotency storage pattern is TBD.
- Exact age-range constraints for child profile validation are TBD.
- Malware scanning engine choice (e.g., ClamAV service) is TBD.
