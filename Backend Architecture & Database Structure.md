# Backend Architecture & Database Structure

## 1. Architecture Overview

### 1.1 System Architecture
- **Pattern**: RESTful API + SSR frontend (Next.js)
- **Auth choice**: **SaaS Auth + local user mapping + JWT access/refresh tokens with HttpOnly cookies**
  - **Why this approach**:
    - Auth UX already owned by external SaaS.
    - Local API still needs user identity for domain data ownership and audit trails.
    - JWT access token supports stateless API auth; refresh token in HttpOnly cookie supports secure session continuity.
    - Works consistently for Web and WhatsApp-linked session resumption.
- **Data flow**: Client (Web/WhatsApp) → API → Domain services → DB/Redis/S3/Payments → Webhooks → Status updates

```text
[Next.js SSR Web] ----\
                    ---> [NestJS REST API] ---> [PostgreSQL 16 + Prisma]
[WhatsApp Cloud API] --/           |         \-> [Redis 7]
                                   |         \-> [AWS S3 private buckets]
                                   |         \-> [PDF generation + watermark worker]
                                   |         \-> [Razorpay / PayPal]
                                   \--------> [Webhook handlers + idempotency]
```

### 1.2 Core Domain Modules (Modular Monolith)
1. **Auth Mapping Module (SaaS)**
   - Syncs SaaS user to local `users` record.
   - Issues/refreshes local JWT session tokens.
2. **Child Profile Module (single child per user)**
   - Enforces one child profile per user.
   - Stores child metadata and child photo asset reference.
3. **Series Module (5 concepts)**
   - Generates/stores exactly 5 series per generation cycle.
   - Tracks selection status and ordering.
4. **Storyline Module (5 per series)**
   - Generates/stores exactly 5 storylines per selected series.
   - Tracks selection status and ordering.
5. **Book Module (book creation, status machine)**
   - Creates books from selected storyline.
   - Owns lifecycle transitions and history visibility.
6. **AI Generation Module (prompts/models/logging)**
   - Stores prompt versions/model params.
   - Logs generation I/O, timing, costs, and model metadata.
7. **Preview/PDF Module (watermark, final PDF)**
   - Builds watermarked preview PDF.
   - Produces final non-watermarked PDF post-payment.
8. **Payments Module (Razorpay/PayPal checkout + webhooks)**
   - Creates provider orders.
   - Verifies callbacks and updates payment/book/order states.
9. **Assets Module (image assets, pdf assets, s3 signed URLs)**
   - Manages upload/download signed URLs.
   - Persists `image_assets` and `pdf_assets` metadata.
10. **Chat Module (chat session + messages)**
    - Persists conversation context and state for repeat generation.
11. **Audit Module (immutable events)**
    - Writes append-only system/user actions for compliance.
12. **Admin/Support Module**
    - Read tools for investigation, payment reconciliation, and user support workflows.

### 1.3 Key Status Machines

- **Series status**
  - `DRAFT` → `GENERATED` → `SELECTED` | `REJECTED` | `ARCHIVED`
- **Storyline status**
  - `DRAFT` → `GENERATED` → `SELECTED` | `REJECTED` | `ARCHIVED`
- **Book status**
  - `DRAFT` → `AI_GENERATING` → `PREVIEW_READY` → `PAYMENT_PENDING` → `PAID` → `FINAL_READY` → `DELIVERED` → `ARCHIVED`
  - Error branch: any active state → `FAILED`
- **Payment status**
  - `CREATED` → `AUTHORIZED` → `CAPTURED` → (`REFUNDED` optional)
  - Failure branch: any non-final state → `FAILED`
- **Regeneration rounds/limits**
  - `books.regeneration_count` stores current count.
  - Every regeneration appends a `regeneration_history` row.
  - Max rounds enforced by config (default 3).

### 1.4 Caching & Idempotency
- **Redis cached items**
  - Sessions and refresh-token references.
  - Rate-limit counters.
  - User-scoped books list summary.
  - User-scoped series/storyline list summary.
  - Temporary generation workflow state.
  - Webhook idempotency keys.
- **TTL values**
  - Session cache: 30 days (aligned to refresh token max).
  - Rate-limit counters: 60 seconds windows.
  - Books list cache: 120 seconds.
  - Series/storyline cache: 300 seconds.
  - Temporary generation state: 30 minutes.
  - Webhook idempotency keys: 7 days.
- **Idempotency approach**
  - Payment create-order endpoints accept `Idempotency-Key` header.
  - Webhook dedupe uses `webhook_events.event_id` unique + Redis key fallback.
  - Preview/final generation operations use deterministic operation key: `book:{id}:preview` / `book:{id}:final`.

### 1.5 Security & Compliance
- **PII rules for child photos**
  - Child photos treated as sensitive PII.
  - Access only via signed URLs.
  - No public ACL objects.
- **S3 private buckets + signed URLs**
  - Private bucket(s) for child photos and PDFs.
  - Signed URLs required for upload/download.
- **Encryption at rest**
  - S3 objects use SSE-KMS.
  - Database encryption enabled at storage layer.
- **Audit log policy**
  - `audit_logs` is insert-only.
  - Store actor, action, entity, payload, correlation id, timestamp.
- **Rate limits by category**
  - Auth/session endpoints: strict.
  - Generation endpoints: moderate.
  - Payments/webhooks: strict + idempotent.
  - Asset signed-url endpoints: moderate.

---

## 2. Database Schema
- **DB**: PostgreSQL 16.x
- **ORM**: Prisma 5.x
- **Naming**: snake_case tables/columns
- **Common columns**: every table includes `created_at`, `updated_at` (except immutable tables where `updated_at` is still present but unchanged after insert)
- **Primary keys**: UUID preferred for all domain tables
- **ERD**: user-centric with one child profile, many series/storylines/books, and full auditability

### 2.1 Entity Relationship Diagram (Text)
- `users` **1:1** `child_profiles` (unique `child_profiles.user_id`).
- `child_profiles` **1:N** `series`.
- `series` **1:N** `storylines`.
- `users` **1:N** `books`.
- `child_profiles` **1:N** `books`.
- `series` **1:N** `books`.
- `storylines` **1:N** `books`.
- `books` **1:N** `book_pages`.
- `books` **1:N** `regeneration_history`.
- `books` **1:N** `payments`.
- `books` **1:N** `orders`.
- `books` references `pdf_assets` for preview/final assets.
- `book_pages` references `image_assets`.
- `child_profiles` references `image_assets` for child photo.
- `users` **1:N** `chat_sessions`.
- `chat_sessions` **1:N** `chat_messages`.
- `audit_logs` links loosely to entities via (`entity_type`, `entity_id`) and optional `user_id`.
- `webhook_events` stores provider callbacks and dedupe metadata.

---

## 3. Tables & Relationships (FULL SPEC)

### 3.1 `users`
**Purpose**: Local user record mapped from external SaaS auth identity.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Local user id |
| saas_user_id | varchar(128) | NOT NULL, UNIQUE | External SaaS user identifier |
| email | varchar(320) | NOT NULL, UNIQUE | User email |
| name | varchar(120) | NULL | Display name |
| role | varchar(32) | NOT NULL, default 'user' | user/admin/support |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_users_saas_user_id`
- `uq_users_email`
- `idx_users_role`

**Relationships**
- 1:1 with `child_profiles` via `child_profiles.user_id` (ON DELETE CASCADE).
- 1:N with `books`, `payments`, `orders`, `chat_sessions`, `audit_logs`.

### 3.2 `child_profiles`
**Purpose**: Single child profile per user.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Child profile id |
| user_id | uuid | NOT NULL, UNIQUE, FK users(id) | Enforces one child per user |
| child_name | varchar(100) | NOT NULL | Child name |
| age | smallint | NOT NULL, CHECK (age BETWEEN 1 AND 12) | Child age |
| interests_json | jsonb | NOT NULL | Interests/favorites/hobbies payload |
| photo_asset_id | uuid | NULL, FK image_assets(id) | Child photo reference |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_child_profiles_user_id`
- `idx_child_profiles_photo_asset_id`

**Relationships**
- FK `user_id` → `users.id` (1:1, ON DELETE CASCADE).
- FK `photo_asset_id` → `image_assets.id` (ON DELETE SET NULL).
- 1:N with `series`, `books`, `chat_sessions`.

### 3.3 `series`
**Purpose**: Stores generated series concepts (5 per cycle for child profile).

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Series id |
| child_profile_id | uuid | NOT NULL, FK child_profiles(id) | Owner child profile |
| title | varchar(180) | NOT NULL | Series title |
| concept_overview | text | NOT NULL | Series description |
| position | smallint | NOT NULL, CHECK (position BETWEEN 1 AND 5) | Position within generated set |
| status | varchar(32) | NOT NULL, default 'GENERATED' | DRAFT/GENERATED/SELECTED/REJECTED/ARCHIVED |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_series_child_profile_id`
- `idx_series_status`
- `idx_series_child_profile_position`

**Relationships**
- FK `child_profile_id` → `child_profiles.id` (1:N, ON DELETE CASCADE).
- 1:N with `storylines` and `books`.

### 3.4 `storylines`
**Purpose**: Stores generated storylines (5 per series).

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Storyline id |
| series_id | uuid | NOT NULL, FK series(id) | Parent series |
| title | varchar(180) | NOT NULL | Storyline title |
| plot_summary | text | NOT NULL | Plot summary |
| position | smallint | NOT NULL, CHECK (position BETWEEN 1 AND 5) | Position within generated set |
| status | varchar(32) | NOT NULL, default 'GENERATED' | DRAFT/GENERATED/SELECTED/REJECTED/ARCHIVED |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_storylines_series_id`
- `idx_storylines_status`
- `idx_storylines_series_position`

**Relationships**
- FK `series_id` → `series.id` (1:N, ON DELETE CASCADE).
- 1:N with `books`.

### 3.5 `books`
**Purpose**: Book aggregate root and lifecycle state owner.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Book id |
| user_id | uuid | NOT NULL, FK users(id) | Book owner |
| child_profile_id | uuid | NOT NULL, FK child_profiles(id) | Child reference |
| series_id | uuid | NOT NULL, FK series(id) | Selected series |
| storyline_id | uuid | NOT NULL, FK storylines(id) | Selected storyline |
| title | varchar(220) | NOT NULL | Book title |
| subtitle | varchar(220) | NULL | Book subtitle |
| moral | text | NULL | Ending moral |
| status | varchar(32) | NOT NULL, default 'DRAFT' | Book status machine |
| preview_pdf_asset_id | uuid | NULL, FK pdf_assets(id) | Watermarked preview asset |
| final_pdf_asset_id | uuid | NULL, FK pdf_assets(id) | Final PDF asset |
| total_pages | smallint | NOT NULL, default 16 | Story pages count |
| regeneration_count | smallint | NOT NULL, default 0 | Regeneration rounds used |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_books_user_id`
- `idx_books_child_profile_id`
- `idx_books_series_id`
- `idx_books_storyline_id`
- `idx_books_status`
- `idx_books_created_at`

**Relationships**
- FK `user_id` → `users.id` (ON DELETE RESTRICT).
- FK `child_profile_id` → `child_profiles.id` (ON DELETE RESTRICT).
- FK `series_id` → `series.id` (ON DELETE RESTRICT).
- FK `storyline_id` → `storylines.id` (ON DELETE RESTRICT).
- FK `preview_pdf_asset_id` → `pdf_assets.id` (ON DELETE SET NULL).
- FK `final_pdf_asset_id` → `pdf_assets.id` (ON DELETE SET NULL).
- 1:N with `book_pages`, `payments`, `orders`, `regeneration_history`.

### 3.6 `book_pages`
**Purpose**: Persist rendered page content and linked illustration assets.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Page row id |
| book_id | uuid | NOT NULL, FK books(id) | Parent book |
| page_number | smallint | NOT NULL, CHECK (page_number >= 1) | Page order |
| page_text | text | NOT NULL | Page text |
| image_asset_id | uuid | NULL, FK image_assets(id) | Illustration asset |
| text_layout_json | jsonb | NOT NULL | Layout metadata |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_book_pages_book_id_page_number` (UNIQUE)
- `idx_book_pages_image_asset_id`

**Relationships**
- FK `book_id` → `books.id` (1:N, ON DELETE CASCADE).
- FK `image_asset_id` → `image_assets.id` (ON DELETE SET NULL).

### 3.7 `ai_generation_logs`
**Purpose**: Full AI audit logs for prompts/model params/cost and outputs.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Log id |
| entity_type | varchar(64) | NOT NULL | series/storyline/book/page/image |
| entity_id | uuid | NOT NULL | Referenced entity id |
| prompt_version | varchar(64) | NOT NULL | Prompt version |
| model_name | varchar(128) | NOT NULL | Model name (DALL·E for image generation) |
| temperature | numeric(4,2) | NULL | Temperature used |
| seed | varchar(64) | NULL | Seed if provided |
| input_json | jsonb | NOT NULL | Generation input payload |
| output_json | jsonb | NOT NULL | Generation output payload |
| tokens | integer | NULL | Token usage if applicable |
| cost | numeric(12,6) | NULL | Cost amount |
| duration_ms | integer | NOT NULL | Duration in milliseconds |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_ai_logs_entity`
- `idx_ai_logs_model_name`
- `idx_ai_logs_created_at`

**Relationships**
- Logical polymorphic relation via (`entity_type`, `entity_id`) managed in service layer.

### 3.8 `regeneration_history`
**Purpose**: Tracks each regeneration attempt and asset deltas.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | History id |
| book_id | uuid | NOT NULL, FK books(id) | Parent book |
| round_number | smallint | NOT NULL, CHECK (round_number >= 1) | Regeneration round |
| reason | text | NULL | User/system reason |
| triggered_by | varchar(16) | NOT NULL | USER or SYSTEM |
| previous_asset_ids | jsonb | NOT NULL | Prior assets snapshot |
| new_asset_ids | jsonb | NOT NULL | New assets snapshot |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_regeneration_history_book_round` (UNIQUE book_id + round_number)
- `idx_regeneration_history_book_id`

**Relationships**
- FK `book_id` → `books.id` (1:N, ON DELETE CASCADE).

### 3.9 `payments`
**Purpose**: Tracks payment lifecycle per book and provider.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Payment id |
| user_id | uuid | NOT NULL, FK users(id) | Paying user |
| book_id | uuid | NOT NULL, FK books(id) | Related book |
| provider | varchar(32) | NOT NULL | razorpay/paypal |
| provider_order_id | varchar(128) | NOT NULL | Provider order id |
| provider_payment_id | varchar(128) | NULL | Provider payment/transaction id |
| status | varchar(32) | NOT NULL, default 'CREATED' | Payment status machine |
| amount | numeric(12,2) | NOT NULL | Amount |
| currency | varchar(8) | NOT NULL | Currency code |
| metadata_json | jsonb | NOT NULL | Provider metadata |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_payments_user_id`
- `idx_payments_book_id`
- `idx_payments_provider_status`
- `uq_payments_provider_order_id` (provider + provider_order_id UNIQUE)
- `uq_payments_provider_payment_id` (provider + provider_payment_id UNIQUE, nullable unique)

**Relationships**
- FK `user_id` → `users.id` (ON DELETE RESTRICT).
- FK `book_id` → `books.id` (ON DELETE RESTRICT).

### 3.10 `orders`
**Purpose**: Commercial order record (PDF_ONLY or PRINT_SHIP) linked to book.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Order id |
| user_id | uuid | NOT NULL, FK users(id) | User |
| book_id | uuid | NOT NULL, FK books(id) | Book |
| fulfillment_type | varchar(32) | NOT NULL | PDF_ONLY / PRINT_SHIP |
| shipping_address_json | jsonb | NULL | Required when PRINT_SHIP |
| status | varchar(32) | NOT NULL, default 'CREATED' | CREATED/PAID/FULFILLING/FULFILLED/FAILED |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_orders_user_id`
- `idx_orders_book_id`
- `idx_orders_fulfillment_type_status`

**Relationships**
- FK `user_id` → `users.id` (ON DELETE RESTRICT).
- FK `book_id` → `books.id` (ON DELETE RESTRICT).

### 3.11 `pdf_assets`
**Purpose**: Metadata for preview/final PDF assets in S3.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Asset id |
| s3_key | varchar(512) | NOT NULL, UNIQUE | S3 object key |
| bucket | varchar(128) | NOT NULL | Bucket name |
| type | varchar(16) | NOT NULL | PREVIEW / FINAL |
| mime_type | varchar(64) | NOT NULL | application/pdf |
| size_bytes | bigint | NOT NULL | Size in bytes |
| checksum | varchar(128) | NOT NULL | Integrity checksum |
| signed_url_expires_at | timestamptz | NULL | Last signed URL expiry timestamp |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_pdf_assets_s3_key`
- `idx_pdf_assets_type`

**Relationships**
- Referenced by `books.preview_pdf_asset_id` and `books.final_pdf_asset_id`.

### 3.12 `image_assets`
**Purpose**: Metadata for child photos and generated book images.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Asset id |
| s3_key | varchar(512) | NOT NULL, UNIQUE | S3 object key |
| bucket | varchar(128) | NOT NULL | Bucket name |
| type | varchar(32) | NOT NULL | CHILD_PHOTO/BOOK_ILLUSTRATION/COVER |
| page_number | smallint | NULL | Book page number when illustration |
| style_preset | varchar(64) | NULL | Style marker |
| mime_type | varchar(64) | NOT NULL | image/* mime type |
| size_bytes | bigint | NOT NULL | Size in bytes |
| checksum | varchar(128) | NOT NULL | Integrity checksum |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_image_assets_s3_key`
- `idx_image_assets_type`
- `idx_image_assets_page_number`

**Relationships**
- Referenced by `child_profiles.photo_asset_id` and `book_pages.image_asset_id`.

### 3.13 `chat_sessions`
**Purpose**: Persist chat workflow state for repeat generation and resume.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Session id |
| user_id | uuid | NOT NULL, FK users(id) | User |
| child_profile_id | uuid | NOT NULL, FK child_profiles(id) | Child context |
| current_series_id | uuid | NULL, FK series(id) | Current selected series |
| state_json | jsonb | NOT NULL | Workflow state snapshot |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_chat_sessions_user_id`
- `idx_chat_sessions_child_profile_id`
- `idx_chat_sessions_updated_at`

**Relationships**
- FK `user_id` → `users.id` (ON DELETE CASCADE).
- FK `child_profile_id` → `child_profiles.id` (ON DELETE CASCADE).
- FK `current_series_id` → `series.id` (ON DELETE SET NULL).
- 1:N with `chat_messages`.

### 3.14 `chat_messages`
**Purpose**: Stores chat conversation messages.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Message id |
| chat_session_id | uuid | NOT NULL, FK chat_sessions(id) | Parent session |
| sender | varchar(16) | NOT NULL | USER/ASSISTANT/SYSTEM |
| message_text | text | NULL | Plain text message |
| message_json | jsonb | NULL | Structured payload |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `idx_chat_messages_chat_session_id`
- `idx_chat_messages_created_at`

**Relationships**
- FK `chat_session_id` → `chat_sessions.id` (1:N, ON DELETE CASCADE).

### 3.15 `audit_logs`
**Purpose**: Immutable append-only audit events.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Audit event id |
| user_id | uuid | NULL, FK users(id) | Actor user (optional) |
| entity_type | varchar(64) | NOT NULL | Entity class |
| entity_id | uuid | NOT NULL | Entity id |
| action | varchar(64) | NOT NULL | Action key |
| payload_json | jsonb | NOT NULL | Event details |
| correlation_id | varchar(128) | NOT NULL | Cross-system trace id |
| created_at | timestamptz | NOT NULL, default now() | Event time |
| updated_at | timestamptz | NOT NULL, default now() | Set equal to created_at |

**Indexes**
- `idx_audit_logs_entity`
- `idx_audit_logs_user_id`
- `idx_audit_logs_correlation_id`
- `idx_audit_logs_created_at`

**Relationships**
- FK `user_id` → `users.id` (ON DELETE SET NULL).
- Insert-only policy enforced at application level + DB trigger (optional).

### 3.16 `webhook_events`
**Purpose**: Stores raw webhook events and processing status with idempotency.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | uuid | PK, default gen_random_uuid() | Event row id |
| provider | varchar(32) | NOT NULL | razorpay/paypal/whatsapp |
| event_id | varchar(191) | NOT NULL, UNIQUE | Provider event id |
| event_type | varchar(128) | NOT NULL | Event type |
| payload_json | jsonb | NOT NULL | Raw webhook payload |
| received_at | timestamptz | NOT NULL, default now() | Receipt time |
| processed_at | timestamptz | NULL | Processing completion time |
| status | varchar(32) | NOT NULL, default 'RECEIVED' | RECEIVED/PROCESSED/FAILED/IGNORED |
| idempotency_key | varchar(191) | NOT NULL | Dedupe key |
| created_at | timestamptz | NOT NULL, default now() | Created time |
| updated_at | timestamptz | NOT NULL, default now() | Updated time |

**Indexes**
- `uq_webhook_events_event_id`
- `idx_webhook_events_provider_status`
- `idx_webhook_events_idempotency_key`
- `idx_webhook_events_received_at`

**Relationships**
- No strict FK; links by metadata to payments/books for resilience.

---

## 4. API Endpoints (V1)

**Common request headers for authenticated endpoints**
- `Authorization: Bearer <accessToken>`
- `X-Correlation-Id: <uuid>`
- `Content-Type: application/json` (except upload-url if multipart metadata style is used)

### AUTH (SaaS mapping)

#### POST /api/v1/auth/sync
- **Purpose**: Map SaaS user identity to local `users` row.
- **Auth required**: Yes (SaaS identity token verified server-side).
- **Request JSON**
```json
{ "saasUserId": "saas_abc123", "email": "parent@example.com", "name": "Parent Name" }
```
- **Validation rules**: `saasUserId` required max 128; email valid; name max 120.
- **Response JSON**
```json
{ "user": { "id": "uuid", "email": "parent@example.com", "role": "user" } }
```
- **Errors**: 400 invalid payload; 401 SaaS token invalid; 409 email conflict.
- **Side effects**: upsert `users`; write `audit_logs`; invalidate `books:list:{userId}` cache.
- **Caching**: none.

#### POST /api/v1/auth/session
- **Purpose**: Issue local access/refresh tokens after successful SaaS mapping.
- **Auth required**: Yes (mapped user context required).
- **Request JSON**
```json
{ "userId": "uuid" }
```
- **Validation rules**: userId must exist.
- **Response JSON**
```json
{ "accessToken": "jwt", "expiresIn": 900 }
```
- **Errors**: 401 unauthorized; 404 user not found.
- **Side effects**: set refresh cookie; store token reference in Redis.
- **Caching**: session cache write (TTL 30d).

#### POST /api/v1/auth/logout
- **Purpose**: Revoke current session.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "allDevices": false }
```
- **Validation rules**: boolean only.
- **Response JSON**
```json
{ "success": true }
```
- **Errors**: 401 unauthorized.
- **Side effects**: revoke refresh token(s) in Redis/DB; audit log insert.
- **Caching**: clear session keys.

#### POST /api/v1/auth/refresh
- **Purpose**: Rotate refresh token and issue new access token.
- **Auth required**: Refresh cookie required.
- **Request JSON**
```json
{}
```
- **Validation rules**: valid refresh token and session reference.
- **Response JSON**
```json
{ "accessToken": "jwt", "expiresIn": 900 }
```
- **Errors**: 401 invalid/expired refresh token.
- **Side effects**: rotate refresh token; audit log insert.
- **Caching**: update session key TTL.

### CHILD

#### GET /api/v1/child
- **Purpose**: Fetch single child profile for current user.
- **Auth required**: Yes.
- **Request headers**: common auth headers.
- **Request JSON**: none.
- **Validation rules**: authenticated user required.
- **Response JSON**
```json
{ "child": { "id": "uuid", "childName": "Aarav", "age": 8, "interests": ["space"], "photoAssetId": "uuid" } }
```
- **Errors**: 404 child profile not found.
- **Side effects**: none.
- **Caching**: short cache 120s keyed by user.

#### POST /api/v1/child
- **Purpose**: Create or update single child profile.
- **Auth required**: Yes.
- **Request JSON**
```json
{
  "childName": "Aarav",
  "age": 8,
  "interests": ["space", "robots"],
  "favoriteColor": "blue",
  "favoriteCartoons": ["Doraemon"],
  "hobbies": ["drawing"],
  "photoAssetId": "uuid"
}
```
- **Validation rules**: single profile per user; age range; required fields present.
- **Response JSON**
```json
{ "child": { "id": "uuid", "updated": true } }
```
- **Errors**: 400 validation; 409 duplicate profile race condition.
- **Side effects**: upsert `child_profiles`; audit log insert; invalidate child/series/books caches.
- **Caching**: invalidate `child:{userId}`, `series:list:{childId}`.

### SERIES + STORYLINES

#### POST /api/v1/series/generate
- **Purpose**: Generate exactly 5 series for child profile.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "childProfileId": "uuid", "promptVersion": "series_v1" }
```
- **Validation rules**: child belongs to user; no active generation lock.
- **Response JSON**
```json
{ "series": [{ "id": "uuid", "title": "Space Explorer", "position": 1, "status": "GENERATED" }] }
```
- **Errors**: 409 generation in progress; 422 AI output count not equal 5.
- **Side effects**: insert 5 `series`; insert `ai_generation_logs`; audit log; cache invalidation.
- **Caching**: invalidate `series:list:{childId}`.

#### GET /api/v1/series
- **Purpose**: List series for current child profile.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: child profile exists.
- **Response JSON**
```json
{ "series": [{ "id": "uuid", "title": "Space Explorer", "position": 1, "status": "GENERATED" }] }
```
- **Errors**: 404 child profile not found.
- **Side effects**: none.
- **Caching**: read-through cache TTL 300s.

#### POST /api/v1/series/:id/storylines/generate
- **Purpose**: Generate exactly 5 storylines for a selected series.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "promptVersion": "storyline_v1" }
```
- **Validation rules**: series belongs to current user child; exactly 5 outputs required.
- **Response JSON**
```json
{ "storylines": [{ "id": "uuid", "title": "Rocket Race", "position": 1, "status": "GENERATED" }] }
```
- **Errors**: 404 series; 422 invalid output count.
- **Side effects**: insert 5 `storylines`; insert `ai_generation_logs`; audit log.
- **Caching**: invalidate `storyline:list:{seriesId}`.

#### GET /api/v1/series/:id/storylines
- **Purpose**: List storylines for series.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: series ownership.
- **Response JSON**
```json
{ "storylines": [{ "id": "uuid", "title": "Rocket Race", "position": 1, "status": "GENERATED" }] }
```
- **Errors**: 404 series.
- **Side effects**: none.
- **Caching**: read-through cache TTL 300s.

### BOOK

#### POST /api/v1/books
- **Purpose**: Create book draft from selected storyline.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "seriesId": "uuid", "storylineId": "uuid", "title": "Aarav in Space" }
```
- **Validation rules**: ownership checks; storyline under series.
- **Response JSON**
```json
{ "book": { "id": "uuid", "status": "DRAFT" } }
```
- **Errors**: 400 invalid relation; 404 storyline/series not found.
- **Side effects**: insert `books`; audit log; invalidate books list cache.
- **Caching**: invalidate `books:list:{userId}`.

#### POST /api/v1/books/:id/generate
- **Purpose**: Generate story script/pages/image prompts.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "textPromptVersion": "book_text_v1", "imagePromptVersion": "book_image_v1", "model": "dall-e" }
```
- **Validation rules**: model must be DALL·E for image generation; book status must be DRAFT/FAILED.
- **Response JSON**
```json
{ "bookId": "uuid", "status": "AI_GENERATING" }
```
- **Errors**: 409 invalid state; 422 generation policy violation.
- **Side effects**: update book status; enqueue generation jobs; insert ai logs; create book_pages + image_assets.
- **Caching**: invalidate `book:{id}` and books list cache.

#### POST /api/v1/books/:id/preview
- **Purpose**: Build watermarked preview PDF.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "watermarkTemplateVersion": "wm_v1" }
```
- **Validation rules**: book must have generated pages/assets.
- **Response JSON**
```json
{ "bookId": "uuid", "status": "PREVIEW_READY", "previewPdfAssetId": "uuid" }
```
- **Errors**: 409 invalid state; 500 PDF generation failure.
- **Side effects**: create preview pdf in S3; insert `pdf_assets`; update `books.preview_pdf_asset_id` + status.
- **Caching**: invalidate `book:{id}`.

#### GET /api/v1/books/:id
- **Purpose**: Get detailed book view.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: ownership.
- **Response JSON**
```json
{ "book": { "id": "uuid", "status": "PREVIEW_READY", "regenerationCount": 1 } }
```
- **Errors**: 404 book not found.
- **Side effects**: none.
- **Caching**: 60s cache per book.

#### GET /api/v1/books
- **Purpose**: List user past books/purchases.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: authenticated user.
- **Response JSON**
```json
{ "items": [{ "id": "uuid", "status": "DELIVERED", "paid": true }], "total": 12 }
```
- **Errors**: none beyond auth errors.
- **Side effects**: none.
- **Caching**: 120s user list cache.

#### POST /api/v1/books/:id/regenerate
- **Purpose**: Regenerate content and store history.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "reason": "Need better emotional arc", "scope": "PLOT_AND_IMAGES" }
```
- **Validation rules**: max rounds not exceeded; valid scope; allowed status.
- **Response JSON**
```json
{ "bookId": "uuid", "regenerationCount": 2, "status": "AI_GENERATING" }
```
- **Errors**: 409 max rounds exceeded; 422 invalid scope.
- **Side effects**: increment book regeneration_count; insert regeneration_history; enqueue generation; ai logs.
- **Caching**: invalidate `book:{id}` and lists.

### PAYMENTS

#### POST /api/v1/payments/razorpay/create-order
- **Purpose**: Create Razorpay order for selected book.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "bookId": "uuid", "fulfillmentType": "PDF_ONLY", "amount": 499.00, "currency": "INR" }
```
- **Validation rules**: preview ready; amount matches pricing rules.
- **Response JSON**
```json
{ "provider": "razorpay", "providerOrderId": "order_123", "paymentId": "uuid" }
```
- **Errors**: 409 invalid book state; 422 pricing mismatch.
- **Side effects**: insert payments row status CREATED; insert order row; set book PAYMENT_PENDING.
- **Caching**: invalidate `book:{id}`.

#### POST /api/v1/payments/paypal/create-order
- **Purpose**: Create PayPal order for selected book.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "bookId": "uuid", "fulfillmentType": "PRINT_SHIP", "amount": 1299.00, "currency": "USD", "shippingAddress": { "line1": "..." } }
```
- **Validation rules**: shippingAddress required for PRINT_SHIP.
- **Response JSON**
```json
{ "provider": "paypal", "providerOrderId": "PAYPAL_ORDER_ID", "approvalUrl": "https://..." }
```
- **Errors**: 422 missing shipping address.
- **Side effects**: insert payments + orders; set book PAYMENT_PENDING.
- **Caching**: invalidate `book:{id}`.

#### POST /api/v1/payments/verify
- **Purpose**: Optional client-side verification fallback.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "provider": "razorpay", "providerOrderId": "order_123", "providerPaymentId": "pay_123", "signature": "..." }
```
- **Validation rules**: signature required for provider needing client callback verification.
- **Response JSON**
```json
{ "verified": true, "paymentStatus": "CAPTURED" }
```
- **Errors**: 400 invalid signature; 404 payment record not found.
- **Side effects**: update payment status; write webhook-like audit event.
- **Caching**: invalidate payment/book cache.

#### POST /api/v1/webhooks/razorpay
- **Purpose**: Receive Razorpay webhook events.
- **Auth required**: No (provider signature verified).
- **Request headers**: provider signature header.
- **Request JSON**: provider payload.
- **Validation rules**: signature verification + idempotency.
- **Response JSON**
```json
{ "received": true }
```
- **Errors**: 400 invalid signature.
- **Side effects**: insert webhook_events; update payments/orders/books statuses; enqueue final PDF unlock flow.
- **Caching**: invalidate affected book/payment/order caches.

#### POST /api/v1/webhooks/paypal
- **Purpose**: Receive PayPal webhook events.
- **Auth required**: No (provider verification).
- **Request JSON**: provider payload.
- **Validation rules**: verification + idempotency.
- **Response JSON**
```json
{ "received": true }
```
- **Errors**: 400 invalid webhook verification.
- **Side effects**: same as razorpay webhook.
- **Caching**: invalidate affected cache keys.

#### POST /api/v1/books/:id/unlock
- **Purpose**: Generate/finalize final PDF unlock after successful payment.
- **Auth required**: Yes (or internal service token if webhook-triggered).
- **Request JSON**
```json
{ "trigger": "PAYMENT_CAPTURED" }
```
- **Validation rules**: payment status must be CAPTURED.
- **Response JSON**
```json
{ "bookId": "uuid", "status": "FINAL_READY", "finalPdfAssetId": "uuid" }
```
- **Errors**: 409 payment not captured.
- **Side effects**: create final PDF in S3; insert `pdf_assets`; update book/order states; audit log.
- **Caching**: invalidate `book:{id}` + books list.

### ASSETS

#### POST /api/v1/assets/upload-url
- **Purpose**: Generate S3 presigned upload URL for child photo.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "fileName": "child.png", "mimeType": "image/png", "sizeBytes": 5242880, "type": "CHILD_PHOTO" }
```
- **Validation rules**: mime type allowlist; size <= max.
- **Response JSON**
```json
{ "uploadUrl": "https://...", "s3Key": "images/...", "expiresIn": 900 }
```
- **Errors**: 422 invalid file type/size.
- **Side effects**: optional placeholder `image_assets` row creation.
- **Caching**: none.

#### GET /api/v1/assets/:id/download-url
- **Purpose**: Get signed URL for private asset download.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: ownership/access policy.
- **Response JSON**
```json
{ "downloadUrl": "https://...", "expiresIn": 300 }
```
- **Errors**: 403 forbidden; 404 asset not found.
- **Side effects**: update signed_url_expires_at (for pdf assets).
- **Caching**: none.

### CHAT

#### POST /api/v1/chat/sessions
- **Purpose**: Start or resume chat workflow.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "childProfileId": "uuid", "resume": true }
```
- **Validation rules**: child ownership.
- **Response JSON**
```json
{ "session": { "id": "uuid", "state": { "step": "SERIES_SELECTION" } } }
```
- **Errors**: 404 child profile not found.
- **Side effects**: create/update `chat_sessions`; audit log.
- **Caching**: write temporary workflow key in Redis TTL 30m.

#### GET /api/v1/chat/sessions/:id
- **Purpose**: Fetch session state and recent messages.
- **Auth required**: Yes.
- **Request JSON**: none.
- **Validation rules**: ownership.
- **Response JSON**
```json
{ "session": { "id": "uuid", "state": { "step": "STORYLINE_SELECTION" } }, "messages": [] }
```
- **Errors**: 404 session not found.
- **Side effects**: none.
- **Caching**: session state cache TTL 300s.

#### POST /api/v1/chat/sessions/:id/messages
- **Purpose**: Append user/system/assistant message.
- **Auth required**: Yes.
- **Request JSON**
```json
{ "sender": "USER", "messageText": "Generate new storyline options", "messageJson": {} }
```
- **Validation rules**: sender enum USER/ASSISTANT/SYSTEM; at least messageText or messageJson.
- **Response JSON**
```json
{ "message": { "id": "uuid", "createdAt": "2026-02-17T10:00:00Z" } }
```
- **Errors**: 400 validation; 404 session not found.
- **Side effects**: insert chat_messages; update chat_sessions.updated_at and state_json.
- **Caching**: invalidate `chat:session:{id}`.

---

## 5. Validation Rules
- **Child name**
  - Length: 2–100 chars.
  - Trim whitespace; disallow only-symbol names.
- **Child age**
  - Integer range: 1–12.
- **Interests array**
  - Min 1, max 20 values.
  - Each value length 2–50 chars.
  - De-duplicate normalized values.
- **Upload files**
  - Types: `image/jpeg`, `image/png`, `image/webp` (if enabled).
  - Max size: 10 MB per photo.
- **Book page text**
  - Stored as text.
  - Target 2–3 lines enforced by generator policy; API checks max 320 chars/page (hard cap).
- **Regeneration max rounds**
  - Configurable env var (default 3).
  - Reject request once `books.regeneration_count >= max`.

---

## 6. Error Handling Standard

**Error envelope**
```json
{
  "success": false,
  "error": {
    "code": "BOOK_INVALID_STATE",
    "message": "Book must be PREVIEW_READY before payment.",
    "details": { "bookId": "uuid" }
  },
  "meta": {
    "correlationId": "uuid",
    "timestamp": "2026-02-17T10:00:00Z"
  }
}
```

**Error code mapping**
- `VALIDATION_ERROR` → 400
- `UNAUTHORIZED` → 401
- `FORBIDDEN` → 403
- `NOT_FOUND` → 404
- `CONFLICT` → 409
- `RATE_LIMITED` → 429
- `BOOK_INVALID_STATE` → 409
- `PAYMENT_VERIFICATION_FAILED` → 400
- `WEBHOOK_SIGNATURE_INVALID` → 400
- `GENERATION_FAILED` → 500
- `INTERNAL_ERROR` → 500

---

## 7. Caching Strategy

Redis key patterns and TTL:
- `session:{userId}:{deviceId}` → 30d
- `rate_limit:{group}:{identifier}:{window}` → 60s
- `series:list:{childProfileId}` → 300s
- `storyline:list:{seriesId}` → 300s
- `books:list:{userId}:page:{n}` → 120s
- `book:detail:{bookId}` → 60s
- `idempotency:webhook:{provider}:{eventId}` → 7d
- `idempotency:payment:{provider}:{idempotencyKey}` → 24h
- `generation:state:{bookId}` → 30m

---

## 8. Rate Limiting

- **Auth group** (`/auth/*`): 10 req/min/IP.
- **Generation group** (`/series/generate`, `/storylines/generate`, `/books/:id/generate`, `/books/:id/preview`, `/books/:id/regenerate`): 5 req/min/user.
- **Asset signed-url group** (`/assets/*`): 20 req/min/user.
- **Books list/detail group**: 60 req/min/user.
- **Payments create/verify**: 10 req/10 min/user.
- **Webhook endpoints**: 120 req/min/provider-source IP + idempotency dedupe.

**Storage strategy**
- Redis counters with sliding-window or fixed-window strategy.
- Keyed by user id where authenticated, otherwise by IP + provider identifier.

---

## 9. Database Migrations

**Prisma migration workflow (dev → staging → prod)**
1. Dev: schema change in Prisma schema.
2. Generate migration: `prisma migrate dev --name <migration_name>`.
3. Commit migration files.
4. Staging deploy: `prisma migrate deploy` + smoke tests.
5. Production deploy: `prisma migrate deploy` in maintenance-safe window.

**Rules**
- Never edit already deployed migration files.
- Forward-fix with new migration if issue is found.
- Include data backfill scripts for non-null additions.
- Rollback plan:
  - Prefer roll-forward migration.
  - If urgent rollback needed, restore DB from PITR to point before migration and redeploy previous app version.

---

## 10. Backup & Recovery

- **Backups**: Daily automated snapshots (RDS) + transaction log retention for PITR.
- **Retention**: 30 days.
- **Recovery checklist**
  1. Identify incident time and target recovery point.
  2. Restore RDS to point-in-time clone.
  3. Validate schema version and critical counts (`users`, `books`, `payments`, `orders`).
  4. Repoint app read-only first, run integrity checks.
  5. Resume writes after validation.
  6. Reconcile webhook replay for gap window.
  7. Publish incident report and corrective actions.

---

## 11. API Versioning

- Base path: `/api/v1/`
- **Deprecation policy**
  - Minimum 90-day deprecation notice for breaking changes.
  - Deprecation headers returned on deprecated endpoints.
  - New behavior shipped under `/api/v2/` without silent breaking changes in v1.
