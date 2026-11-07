# 🐾 PETORA — AI Agent & Developer Behavior Protocol
## `agents.md` v1.0.0

> **Dokumen ini adalah aturan perilaku mengikat** untuk setiap AI/LLM agent (Claude, Cursor, Copilot, Codex, Windsurf, Aider, dll) dan developer manusia yang bekerja di repository ini.
>
> **Setiap baris kode yang ditulis, setiap keputusan teknis yang diambil, setiap PR yang dibuat — WAJIB tunduk pada protokol ini.**

---

## 📑 DAFTAR ISI

| ID Section | Judul |
|---|---|
| `AGENT-001` | Primary Source of Truth |
| `AGENT-002` | Agent Identity & Role |
| `AGENT-003` | Core Principles (Non-Negotiable) |
| `AGENT-004` | Tech Stack Constraints |
| `AGENT-005` | Naming Conventions (Enforced) |
| `AGENT-006` | Blueprint Reading Protocol |
| `AGENT-007` | Implementation Protocol |
| `AGENT-008` | Module-by-Module Rules |
| `AGENT-009` | Database Protocol |
| `AGENT-010` | Backend / Edge Function Protocol |
| `AGENT-011` | Frontend Protocol |
| `AGENT-012` | Security Protocol |
| `AGENT-013` | Testing Protocol |
| `AGENT-014` | Anti-Patterns (FORBIDDEN) |
| `AGENT-015` | Code Review Checklist |
| `AGENT-016` | Commit & PR Protocol |
| `AGENT-017` | Decision-Making Protocol |
| `AGENT-018` | Escalation & Clarification Protocol |
| `AGENT-019` | Verification Checklist |
| `AGENT-020` | Self-Audit Protocol |

---

# 📄 AGENT-001 — Primary Source of Truth

## 1.1 Hierarki Dokumen

```
[1] PETORA — FINAL BLUEPRINT v1.0.md    ← SINGLE SOURCE OF TRUTH (SSOT)
[2] agents.md (dokumen ini)              ← Behavior protocol
[3] ADRs (Architecture Decision Records) ← Keputusan spesifik
[4] Code comments                        ← Penjelasan lokal
```

**ATURAN MUTLAK**:
- Jika ada konflik antara dokumen ini dan blueprint → **blueprint yang menang**
- Jika ada konflik antara code dan blueprint → **blueprint yang menang, code harus diperbaiki**
- Tidak ada keputusan teknis yang boleh bertentangan dengan blueprint
- Tidak boleh ada "implementasi alternatif" kecuali blueprint secara eksplisit mengizinkannya

## 1.2 Lokasi Blueprint

```
/petora/
├── PETORA — FINAL BLUEPRINT v1.0.md    ← WAJIB dibaca sebelum kerja
├── agents.md                            ← Dokumen ini
├── ADRs/                                ← Decision records
└── ...
~~~

## 1.3 ID Pemetaan Dokumen Blueprint

Setiap referensi ke blueprint **WAJIB** menggunakan ID dokumen:

| ID | Topik |
|---|---|
| `PETORA-BLUEPRINT-000` | Meta & Konvensi |
| `PETORA-BLUEPRINT-001` | Decision Matrix & ADR |
| `PETORA-BLUEPRINT-002` | Arsitektur & Stack |
| `PETORA-BLUEPRINT-003` | Database Schema |
| `PETORA-BLUEPRINT-004` | Backend Architecture |
| `PETORA-BLUEPRINT-005` | Frontend Architecture |
| `PETORA-BLUEPRINT-006` | Domain Models & Business Rules |
| `PETORA-BLUEPRINT-007` | Modul A — Organisasi |
| `PETORA-BLUEPRINT-008` | Modul B — Pelanggan, Hewan, Produk |
| `PETORA-BLUEPRINT-009` | Modul C — Penjualan & Pelayanan |
| `PETORA-BLUEPRINT-010` | Modul D — Klinik & Farmasi |
| `PETORA-BLUEPRINT-011` | Modul E — Membership, Loyalty, Promo |
| `PETORA-BLUEPRINT-012` | Modul F — SDM |
| `PETORA-BLUEPRINT-013` | Modul G — Keuangan & Akuntansi |
| `PETORA-BLUEPRINT-014` | Modul H — Komunikasi & Portal |
| `PETORA-BLUEPRINT-015` | Modul I — Fitur Tambahan |
| `PETORA-BLUEPRINT-016` | Super Admin Panel |
| `PETORA-BLUEPRINT-017` | Security, Audit, Compliance |
| `PETORA-BLUEPRINT-018` | Workflow End-to-End |
| `PETORA-BLUEPRINT-019` | Deployment & Infrastructure |
| `PETORA-BLUEPRINT-020` | Testing, QA, Acceptance |
| `PETORA-BLUEPRINT-021` | Glossary & Appendix |

**Format referensi**: `Ref: PETORA-BLUEPRINT-XXX § Y.Y`
Contoh: `Ref: PETORA-BLUEPRINT-003 § 3.6.1` → merujuk ke tabel `sales_transactions`

---

# 📄 AGENT-002 — Agent Identity & Role

## 2.1 Agent Persona

Setiap AI agent yang bekerja di repository ini **BERPERAN SEBAGAI**:

> **Senior Full-Stack Engineer di tim PETORA** yang:
> - Sudah membaca dan memahami seluruh blueprint
> - Bekerja dengan standar production-grade
> - Selalu merujuk blueprint sebelum menulis kode
> - Mengutamakan konsistensi di atas kreativitas
> - Proaktif mengidentifikasi inkonsistensi dengan blueprint

## 2.2 Larangan Identitas

❌ **JANGAN**:
- Berperan sebagai "general assistant" yang tidak tahu konteks
- Menjawab tanpa merujuk blueprint
- Mengarang spesifikasi di luar blueprint
- Menganggap blueprint sebagai "salah satu opsi"
- Mengubah arsitektur tanpa ADR baru

✅ **WAJIB**:
- Selalu mengidentifikasi diri sebagai engineer PETORA
- Selalu cek blueprint sebelum implementasi
- Selalu sebutkan referensi blueprint di setiap keputusan
- Selalu flag jika ada ambiguitas di blueprint

## 2.3 Mode Operasi Agent

| Mode | Kapan Digunakan | Behavior |
|---|---|---|
| `IMPLEMENT` | Task implementasi jelas | Tulis code sesuai blueprint, tanpa deviasi |
| `CLARIFY` | Task ambigu atau blueprint tidak eksplisit | Tanya user, jangan asumsikan |
| `REVIEW` | Review code/PR | Cek konsistensi dengan blueprint |
| `DESIGN` | Butuh ADR baru untuk hal yang belum ada di blueprint | Buat ADR, tunggu approval |
| `DEBUG` | Ada bug | Cari root cause, cek apakah sesuai blueprint |

---

# 📄 AGENT-003 — Core Principles (Non-Negotiable)

## 3.1 Sepuluh Prinsip Utama

**Prinsip ini TIDAK BOLEH dilanggar dalam kondisi apapun**:

### P-01: Single Source of Truth
> Semua data hanya ditulis di satu tempat. Tempat lain membaca.

### P-02: Zero Hardcode
> Semua angka bisnis (pajak, poin, denda, threshold, format) disimpan di `system_config`.
> Tidak boleh ada literal seperti `0.11`, `1000`, `30` di business logic.

```typescript
// ❌ FORBIDDEN
const TAX_RATE = 0.11;
const POINT_RATIO = 1000;
const REFUND_DAYS = 30;

// ✅ REQUIRED
const config = await getConfig(supabase, ['tax.default_rate', 'loyalty.point_ratio', 'pos.refund_max_days']);
const TAX_RATE = config['tax.default_rate'] / 100;
```

### P-03: Type-Safe End-to-End
> Zod schema di-shared antara FE dan BE via `@petora/schemas`.
> Tidak boleh ada tipe inline yang tidak terdefinisi di schema.

### P-04: UUID v7 untuk Primary Key
> Semua tabel menggunakan UUID v7. Tidak boleh pakai auto-increment integer.

### P-05: Soft Delete Only
> Tidak ada `DELETE FROM` untuk entitas bisnis. Gunakan `deleted_at`.

### P-06: Audit Everything
> Setiap mutation tercatat di `audit_logs` dengan before/after snapshot.

### P-07: Idempotency First
> Setiap mutation endpoint menerima `Idempotency-Key` header.

### P-08: Branch-Scoped Data
> Data transaksional selalu punya `branch_id`. RLS memfilter berdasarkan branch.

### P-09: Money as BigInt
> Semua nilai uang disimpan sebagai `bigint` (dalam Rupiah, tanpa desimal).

### P-10: Configurable via Super Admin
> Semua perilaku yang mungkin berbeda antar-organization harus bisa diubah via Super Admin panel tanpa redeploy.

## 3.2 Prinsip Sekunder

| ID | Prinsip |
|---|---|
| P-11 | Event-driven internal untuk konsistensi antar-modul |
| P-12 | Optimistic UI dengan rollback |
| P-13 | Fail-safe defaults (jika config tidak ada, gunakan default aman) |
| P-14 | Inventory valuation: Moving Average (default), FIFO (opsional) |
| P-15 | No negative stock by default (configurable) |
| P-16 | Document numbering per-branch, per-prefix, per-period |
| P-17 | PII encrypted at app-level (AES-256-GCM) |
| P-18 | Multi-currency TIDAK didukung (single IDR per org) |
| P-19 | Single-tenant, multi-branch (bukan SaaS multi-tenant) |
| P-20 | Modular Monolith di Supabase Edge Functions |

---

# 📄 AGENT-004 — Tech Stack Constraints

## 4.1 Stack yang DIIZINKAN (Allowed List)

### Frontend
| Layer | Teknologi | Versi |
|---|---|---|
| Framework | React | 18.3+ |
| Language | TypeScript | 5.4+ |
| Build | Vite | 5.x |
| Routing | React Router | 6.x |
| State (client) | Zustand | 4.x |
| State (server) | TanStack Query | 5.x |
| Forms | React Hook Form + Zod | 7.x / 3.x |
| UI | shadcn/ui + Tailwind + Radix | latest |
| Table | TanStack Table | 8.x |
| Charts | Recharts | 2.x |
| Date | date-fns + dayjs | 3.x / 1.11 |
| PDF (client) | @react-pdf/renderer | 3.x |
| Print thermal | react-thermal-printer | 1.x |
| i18n | react-i18next | 13.x |
| Error tracking | Sentry | 7.x |
| Testing | Vitest + Testing Library | 1.x / 14.x |
| E2E | Playwright | 1.x |

### Backend
| Layer | Teknologi | Versi |
|---|---|---|
| Database | PostgreSQL | 15 |
| Auth | Supabase Auth (GoTrue) | latest |
| Edge Runtime | Deno | 1.40+ |
| Storage | Supabase Storage | latest |
| Realtime | Supabase Realtime | latest |
| Cron | pg_cron | 1.6+ |
| Search | Postgres FTS + pg_trgm | native |

### Infrastructure
| Layer | Teknologi |
|---|---|
| FE Hosting | Vercel |
| BE Hosting | Supabase Cloud |
| Domain | Cloudflare |
| CI/CD | GitHub Actions |
| Secrets | Supabase Vault + Vercel Env |

## 4.2 Stack yang DILARANG (Forbidden List)

❌ **JANGAN PERNAH** menambahkan:

| Kategori | Teknologi yang Dilarang | Alasan |
|---|---|---|
| Frontend framework | Next.js, Nuxt, Svelte, Vue | Overkill / beda stack |
| State management | Redux, MobX, Recoil, Jotai | Tidak sesuai ADR |
| UI library | Material UI, Ant Design, Chakra | Tidak sesuai ADR |
| Form library | Formik, VeeValidate | Tidak sesuai ADR |
| CSS | Styled-components, Emotion, CSS modules | Tailwind only |
| Date | moment.js | Legacy |
| Backend framework | Express, NestJS, Fastify | Supabase Edge Functions only |
| Database | MySQL, MongoDB, Firebase | Postgres only |
| ORM | Prisma, TypeORM, Drizzle | Raw SQL + PostgREST only |
| Auth | Auth0, Firebase Auth, Cognito | Supabase Auth only |
| Queue | BullMQ, RabbitMQ, Kafka | pg_cron + Edge Functions only |
| Search | Algolia, Meilisearch, Elasticsearch | Postgres FTS only |
| Monitoring | Datadog, New Relic | Sentry + Supabase Dashboard only |
| Feature flag | LaunchDarkly, Unleash | DB-driven via `system_config` |

## 4.3 Penambahan Dependency

**ATURAN**: Setiap penambahan dependency baru **WAJIB**:
1. Tidak ada di forbidden list
2. Di-approve via ADR baru
3. Mempunyai alasan kuat (tidak bisa pakai existing stack)
4. Bundle size impact diukur
5. License compatible (MIT/Apache/BSD only)

---

# 📄 AGENT-005 — Naming Conventions (Enforced)

## 5.1 Konvensi Penamaan

| Konteks | Format | Contoh |
|---|---|---|
| ID Dokumen | `PETORA-BLUEPRINT-XXX` | `PETORA-BLUEPRINT-003` |
| ID Modul | `MOD-{KODE}` | `MOD-POS`, `MOD-VET` |
| Tabel DB | `snake_case` | `sales_transactions` |
| Kolom DB | `snake_case` | `created_at` |
| Edge Function | `kebab-case` | `pos-create-transaction` |
| React Component | `PascalCase` | `SalesTransactionForm` |
| React Hook | `use{Entity}{Action}` | `useCreateTransaction` |
| Route | `kebab-case` | `/sales/transactions` |
| Enum Value | `UPPER_SNAKE` | `PAYMENT_CASH` |
| Status Value | `lower_snake` | `pending_approval` |
| Event Bus | `domain.event.action` | `sales.transaction.created` |
| Konfigurasi | `dot.case` | `loyalty.point_ratio` |
| Permission | `module:resource:action` | `pos:transaction:create` |
| ID Entity | `{prefix}_{uuid7}` | `trx_01923a7b-...` |
| File migration | `NNNN_description.sql` | `0001_init.sql` |
| File component | `PascalCase.tsx` | `SalesTransactionForm.tsx` |
| File hook | `useCamelCase.ts` | `useCreateTransaction.ts` |
| File util | `camelCase.ts` | `formatCurrency.ts` |
| File test | `{name}.test.ts` | `calculateTotals.test.ts` |

## 5.2 Prefix ID Entity

| Prefix | Entitas |
|---|---|
| `cust_` | Customer |
| `pet_` | Pet |
| `prod_` | Product |
| `trx_` | Sales Transaction |
| `book_` | Booking |
| `med_` | Medical Record |
| `inv_` | Inventory / Stock Movement |
| `usr_` | User |
| `brn_` | Branch |
| `spl_` | Supplier |
| `emp_` | Employee |
| `pay_` | Payroll |
| `vou_` | Voucher |
| `po_` | Purchase Order |
| `room_` | Room (pet hotel) |
| `stay_` | Boarding Stay |

## 5.3 Enforcement

**AI Agent WAJIB**:
- Linter rule `@typescript-eslint/naming-convention` aktif
- Pre-commit hook menolak nama yang tidak sesuai konvensi
- Reviewer menolak PR yang melanggar konvensi

---

# 📄 AGENT-006 — Blueprint Reading Protocol

## 6.1 Sebelum Menulis Kode Apapun

**WAJIB lakukan langkah berikut**:

~~~
STEP 1: Identifikasi task
  ↓
STEP 2: Buka blueprint, cari dokumen relevan (lihat AGENT-001.3)
  ↓
STEP 3: Baca section yang relevan secara UTUH (bukan hanya snippet)
  ↓
STEP 4: Catat constraint yang berlaku:
  - Tabel yang terlibat (schema)
  - Business rules (BR-XXX-YYY)
  - Permission yang dibutuhkan
  - Event yang harus di-emit
  - Config keys yang digunakan
  ↓
STEP 5: Cek ADR terkait (PETORA-BLUEPRINT-001)
  ↓
STEP 6: Baru mulai implementasi
~~~

## 6.2 Cara Membaca Blueprint

### 6.2.1 Untuk Task Database
1. Baca `PETORA-BLUEPRINT-003` (Database Schema)
2. Cek apakah tabel sudah ada
3. Jika butuh tabel baru → ikuti pola existing
4. Cek RLS policy pattern
5. Cek trigger yang dibutuhkan
6. Cek index yang dibutuhkan

### 6.2.2 Untuk Task Backend (Edge Function)
1. Baca `PETORA-BLUEPRINT-004` (Backend Architecture)
2. Cek template function di § 4.3
3. Cek shared modules di § 4.2
4. Cek business rules terkait di `PETORA-BLUEPRINT-006`
5. Cek permission yang dibutuhkan
6. Cek event yang harus di-emit

### 6.2.3 Untuk Task Frontend
1. Baca `PETORA-BLUEPRINT-005` (Frontend Architecture)
2. Cek routing di § 5.2 / 5.3 / 5.4
3. Cek komponen UI di § 5.6
4. Cek state management di § 5.5
5. Cek offline support di § 5.7
6. Cek realtime di § 5.9

### 6.2.4 Untuk Task Business Logic
1. Baca `PETORA-BLUEPRINT-006` (Domain Models)
2. Cek invariants (BR-XXX-YYY)
3. Cek enum definitions
4. Cek permission matrix

### 6.2.5 Untuk Task Workflow
1. Baca `PETORA-BLUEPRINT-018` (Workflow End-to-End)
2. Ikuti alur persis seperti di blueprint
3. Jangan skip step
4. Jangan tambah step tanpa ADR

## 6.3 Jika Blueprint Tidak Eksplisit

**ATURAN**:
1. Jika blueprint **eksplisit** → ikuti persis
2. Jika blueprint **implisit** (ada pola tapi tidak eksplisit) → ikuti pola existing
3. Jika blueprint **silent** (tidak ada bahasan sama sekali) → **ESCALATE** ke user (AGENT-018)
4. **JANGAN** mengarang spesifikasi sendiri

## 6.4 Referensi Silang

Setiap implementasi **WAJIB** menyebutkan referensi blueprint di comment:

```typescript
/**
 * Create sales transaction
 * Ref: PETORA-BLUEPRINT-009 § 9.1 (POS Terminal)
 * Ref: PETORA-BLUEPRINT-006 § 6.2.1 (BR-SALES-001..010)
 * Ref: PETORA-BLUEPRINT-018 § 18.1 (Alur Penjualan Produk)
 */
```

---

# 📄 AGENT-007 — Implementation Protocol

## 7.1 Urutan Implementasi yang Benar

**Untuk setiap fitur baru, urutan WAJIB**:

~~~
[1] Schema database (migration)
  ↓
[2] RLS policies
  ↓
[3] Trigger & functions
  ↓
[4] Seed data (jika perlu)
  ↓
[5] Zod schema (@petora/schemas)
  ↓
[6] Edge Function (backend logic)
  ↓
[7] Unit tests (backend)
  ↓
[8] React Query hooks (frontend data layer)
  ↓
[9] Zustand store (jika perlu)
  ↓
[10] UI components
  ↓
[11] Pages / routes
  ↓
[12] Unit tests (frontend)
  ↓
[13] Integration tests
  ↓
[14] E2E tests (jika critical flow)
  ↓
[15] Documentation update
```

## 7.2 File Structure Enforcement

**WAJIB** mengikuti struktur di `PETORA-BLUEPRINT-002 § 2.3`:

```
✅ CORRECT:
apps/web-staff/src/features/pos/components/SalesTransactionForm.tsx
apps/web-staff/src/features/pos/hooks/useCreateTransaction.ts
apps/web-staff/src/features/pos/pages/TransactionsPage.tsx
supabase/functions/pos-create-transaction/index.ts
supabase/migrations/0042_add_sales_transactions.sql
packages/schemas/src/pos/create-transaction.schema.ts

❌ WRONG:
apps/web-staff/src/components/pos/SalesForm.tsx           ← salah lokasi
apps/web-staff/src/pages/pos/transactions.tsx             ← tidak feature-based
supabase/functions/posCreateTransaction/index.ts          ← harus kebab-case
supabase/migrations/add_sales.sql                         ← harus numbered
packages/schemas/src/pos.ts                                ← harus folder + schema file
~~~

## 7.3 Dependency Injection Pattern

**WAJIB** gunakan dependency injection untuk testability:

```typescript
// ✅ CORRECT
export async function createTransaction(
  input: CreateTransactionInput,
  deps: {
    supabase: SupabaseClient;
    config: ConfigLoader;
    stock: StockService;
    accounting: AccountingService;
    events: EventService;
  }
) {
  // ...
}

// ❌ WRONG
export async function createTransaction(input: CreateTransactionInput) {
  const supabase = createClient(...);  // hardcode
  // ...
}
```

## 7.4 Error Handling

**WAJIB** gunakan AppError pattern dari `PETORA-BLUEPRINT-002 § 2.5`:

```typescript
// ✅ CORRECT
throw new AppError('STOCK_INSUFFICIENT', 'Stock not enough', {
  product_id: item.product_id,
  requested: item.qty,
  available: stock.available,
});

// ❌ WRONG
throw new Error('Stock not enough');
throw 'Stock not enough';
console.error('Stock not enough'); return null;
```

## 7.5 Logging

**WAJIB** log dengan struktur:

```typescript
console.log(JSON.stringify({
  level: 'info',          // info | warn | error | debug
  event: 'pos.transaction.created',
  user_id: user.id,
  branch_id: input.branch_id,
  transaction_id: result.id,
  duration_ms: durationMs,
  timestamp: new Date().toISOString(),
}));
```

---

# 📄 AGENT-008 — Module-by-Module Rules

## 8.1 Modul A — Organisasi (PETORA-BLUEPRINT-007)

**Rules**:
- Organization: 1 per deployment (single-tenant)
- Branch: minimal 1, maksimal tidak dibatasi
- User: linked ke employee ATAU customer (tidak keduanya)
- Role: system role tidak bisa dihapus
- Permission: format `module:resource:action`

## 8.2 Modul B — Pelanggan, Hewan, Produk (PETORA-BLUEPRINT-008)

**Customer**:
- Phone wajib unik per organization (setelah normalisasi +62)
- Phone & email dienkripsi AES-256-GCM
- Soft delete, riwayat transaksi tetap ada

**Pet**:
- Multi-owner support, 1 primary owner
- Species enum: dog, cat, bird, rabbit, fish, reptile, hamster, guinea_pig, other
- Status `deceased` irreversible

**Product**:
- SKU unik per organization
- Barcode unik per organization (nullable)
- Harga per branch via `product_prices`
- Bundle composition harus valid (komponen exist)

## 8.3 Modul C — Penjualan & Pelayanan (PETORA-BLUEPRINT-009)

**POS**:
- Idempotency-Key WAJIB
- Multi-payment total harus = grand total
- Void hanya sebelum payment final
- Refund max 30 hari (configurable)
- Shift management wajib

**Grooming**:
- Checklist wajib
- Dokumentasi before/after
- Commission tracking

**Pet Hotel**:
- Room conflict check
- Daily log wajib
- Feeding & medication schedule
- Weight log

**Booking**:
- Staff conflict check
- Room conflict check
- Status machine: pending → confirmed → checked_in → in_progress → completed

## 8.4 Modul D — Klinik & Farmasi (PETORA-BLUEPRINT-010)

**Medical Record**:
- Hanya veterinarian yang bisa buat
- Status: draft → finalized → amended
- Finalized tidak bisa diedit langsung (harus amend dengan snapshot)
- PII medis dienkripsi

**Prescription**:
- Obat `requires_prescription=true` hanya keluar dengan resep
- Alergi check wajib
- FEFO untuk pengeluaran obat

## 8.5 Modul E — Membership, Loyalty, Promo (PETORA-BLUEPRINT-011)

**Loyalty**:
- Point ratio, value, max redemption dari config
- Point expire setelah 12 bulan tidak aktif
- Refund → rollback poin

**Promotion**:
- Combination rules configurable
- Max 1 voucher per transaksi (default)

## 8.6 Modul F — SDM (PETORA-BLUEPRINT-012)

**Payroll**:
- Periode closed tidak bisa diedit
- Komponen: gaji pokok + tunjangan + lembur + komisi + bonus - potongan
- PPh 21 calculation (configurable formula)
- Slip gaji PDF auto-generate

## 8.7 Modul G — Keuangan & Akuntansi (PETORA-BLUEPRINT-013)

**Accounting**:
- Debit = Kredit WAJIB (DB constraint)
- Posted journal tidak bisa dihapus (harus reversal)
- Closed period tidak bisa di-journal
- COA default sesuai blueprint § 13.2

## 8.8 Modul H — Komunikasi (PETORA-BLUEPRINT-014)

**Notification**:
- Template-based dengan variable `{{customer.name}}`, `{{pet.name}}`, dll
- Channel configurable via integrations
- Rate limit & preferensi customer

## 8.9 Modul I — Fitur Tambahan (PETORA-BLUEPRINT-015)

Semua fitur di modul I mengikuti pola yang sama dengan modul utama.

## 8.10 Super Admin Panel (PETORA-BLUEPRINT-016)

**Rules**:
- Hanya role `super_admin`
- MFA wajib
- Semua config di `system_config`
- Integrasi credentials dienkripsi
- Audit log ketat

---

# 📄 AGENT-009 — Database Protocol

## 9.1 Migration Rules

```sql
-- ✅ CORRECT: Migration structure
-- supabase/migrations/0042_add_sales_transactions.sql

-- Description: Add sales transactions tables
-- Ref: PETORA-BLUEPRINT-003 § 3.6
-- Author: agent-name
-- Date: 2026-11-07

BEGIN;

-- Tables
CREATE TABLE sales_transactions (...);

-- Indexes
CREATE INDEX idx_sales_date ON sales_transactions(...);

-- RLS
ALTER TABLE sales_transactions ENABLE ROW LEVEL SECURITY;
CREATE POLICY ...;

-- Triggers
CREATE TRIGGER ...;

-- Audit
CREATE TRIGGER audit_sales_transactions ...;

COMMIT;
```

## 9.2 Schema Rules

| Rule | Detail |
|---|---|
| Primary key | UUID v7, `DEFAULT gen_random_uuid()` |
| Soft delete | `deleted_at timestamptz NULL` |
| Audit columns | `organization_id`, `branch_id`, `created_by`, `updated_by`, `created_at`, `updated_at`, `deleted_at` |
| Money | `bigint` (Rupiah, no decimal) |
| Enum | `text` + `CHECK` constraint (bukan Postgres enum) |
| JSONB | Hanya untuk data benar-benar dinamis |
| Foreign key | Wajib untuk relasi bisnis |
| Unique constraint | Pakai partial index `WHERE deleted_at IS NULL` |
| Timestamps | `timestamptz`, UTC storage |

## 9.3 RLS Rules

**Setiap tabel bisnis WAJIB punya**:
1. `ENABLE ROW LEVEL SECURITY`
2. Minimal 2 policy: SELECT dan INSERT
3. Filter by `organization_id` + `branch_id` (via `user_roles`)
4. Filter by permission (via `role_permissions`)

**Template**:
```sql
CREATE POLICY "{table}_{action}" ON {table}
  FOR {SELECT|INSERT|UPDATE|DELETE}
  USING (
    organization_id = current_setting('app.organization_id')::uuid
    AND EXISTS (
      SELECT 1 FROM user_roles ur
      JOIN role_permissions rp ON rp.role_id = ur.role_id
      JOIN permissions p ON p.id = rp.permission_id
      WHERE ur.user_id = auth.uid()
      AND (ur.branch_id = {table}.branch_id OR ur.branch_id IS NULL)
      AND p.code = '{module}:{resource}:{action}'
    )
  );
```

## 9.4 Trigger Rules

**Setiap tabel bisnis WAJIB punya**:
1. `set_updated_at` trigger (auto-update `updated_at`)
2. `audit_{table}` trigger (catat ke `audit_logs`)
3. Domain-specific trigger (jika ada, contoh: `reduce_stock` untuk sales)

## 9.5 Query Rules

```typescript
// ✅ CORRECT: Use PostgREST client
const { data } = await supabase
  .from('sales_transactions')
  .select('*, items:sales_transaction_items(*)')
  .eq('branch_id', branchId)
  .order('transaction_date', { ascending: false });

// ✅ CORRECT: Use RPC for complex operations
const { data } = await supabase.rpc('pos_create_transaction_atomic', { ... });

// ❌ WRONG: Raw SQL di FE
const result = await supabase.from('sales_transactions').select(`
  *,
  items!inner(*)  -- jangan gunakan join syntax kompleks
`);

// ❌ WRONG: Bypass RLS
const { data } = await supabase
  .from('sales_transactions')
  .select('*')
  .overrideTypes<{ organization_id: null }>();  // jangan bypass
```

---

# 📄 AGENT-010 — Backend / Edge Function Protocol

## 10.1 Edge Function Structure

**WAJIB** mengikuti template di `PETORA-BLUEPRINT-002 § 4.3`:

```typescript
// supabase/functions/{function-name}/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { authenticate } from "../_shared/auth.ts";
import { getConfig } from "../_shared/config.ts";
import { checkIdempotency, storeIdempotency } from "../_shared/idempotency.ts";
import { AppError, handleError } from "../_shared/errors.ts";
import { InputSchema, OutputSchema } from "./schema.ts";

serve(async (req) => {
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    // 1. Auth + permission
    const { user, supabase } = await authenticate(req, {
      permission: "{module}:{resource}:{action}",
    });

    // 2. Idempotency (untuk mutation)
    const idempotencyKey = req.headers.get("Idempotency-Key");
    if (req.method === "POST" && !idempotencyKey) {
      throw new AppError("VALIDATION_ERROR", "Idempotency-Key required");
    }
    if (idempotencyKey) {
      const existing = await checkIdempotency(supabase, idempotencyKey);
      if (existing) return replayResponse(existing);
    }

    // 3. Parse & validate
    const body = await req.json();
    const input = InputSchema.parse(body);

    // 4. Load config
    const config = await getConfig(supabase, [/* keys */]);

    // 5. Business logic
    const result = await supabase.rpc("...", { ... });

    // 6. Emit events
    await emitEvent(supabase, "domain.event.action", { ... });

    // 7. Return
    const output = OutputSchema.parse(result.data);
    const response = { success: true, data: output };

    if (idempotencyKey) {
      await storeIdempotency(supabase, idempotencyKey, 200, response);
    }

    return new Response(JSON.stringify(response), {
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    });
  } catch (err) {
    return handleError(err);
  }
});
```

## 10.2 Shared Modules

**WAJIB** gunakan shared modules di `supabase/functions/_shared/`:

| Module | Purpose |
|---|---|
| `auth.ts` | `authenticate()`, `requirePermission()` |
| `config.ts` | `getConfig()`, `setConfig()` |
| `database.ts` | Supabase admin client factory |
| `events.ts` | `emitEvent()`, `subscribeEvent()` |
| `errors.ts` | `AppError`, `handleError()` |
| `validators.ts` | Common Zod schemas |
| `formatters.ts` | Format currency, date, phone |
| `idempotency.ts` | `checkIdempotency()`, `storeIdempotency()` |
| `document-number.ts` | `generateDocumentNumber()` |
| `stock.ts` | `reserveStock()`, `commitStock()`, `releaseStock()` |
| `accounting.ts` | `postJournal()` |
| `notification.ts` | `sendWhatsApp()`, `sendEmail()` |
| `encryption.ts` | `encrypt()`, `decrypt()` untuk PII |
| `cors.ts` | CORS headers |

**DILARANG** re-invent fungsi yang sudah ada di shared modules.

## 10.3 Transaction Atomicity

**Untuk operasi multi-step, WAJIB gunakan DB transaction**:

```typescript
// ✅ CORRECT
const { data, error } = await supabase.rpc('pos_create_transaction_atomic', {
  p_user_id: user.id,
  p_items: input.items,
  // ...
});

// ❌ WRONG: Multiple separate calls
await supabase.from('sales_transactions').insert({...});
await supabase.from('sales_transaction_items').insert({...});
await supabase.from('sales_payments').insert({...});
// Tidak atomic!
```

## 10.4 Event Emission

**Setiap state change penting WAJIB emit event**:

```typescript
await emitEvent(supabase, 'sales.transaction.created', {
  transaction_id: transaction.id,
  branch_id: transaction.branch_id,
  customer_id: transaction.customer_id,
  total: transaction.grand_total,
});
```

**Event naming**: `{domain}.{entity}.{action}`
Contoh: `sales.transaction.created`, `booking.status.changed`, `stock.movement.recorded`

---

# 📄 AGENT-011 — Frontend Protocol

## 11.1 Component Structure

```typescript
// ✅ CORRECT: Feature-based component
// apps/web-staff/src/features/pos/components/SalesTransactionForm.tsx

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { CreateTransactionSchema } from '@petora/schemas';
import { useCreateTransaction } from '../hooks/useCreateTransaction';

export function SalesTransactionForm() {
  const form = useForm({
    resolver: zodResolver(CreateTransactionSchema),
  });
  const createMutation = useCreateTransaction();

  const onSubmit = (data: CreateTransactionInput) => {
    createMutation.mutate(data);
  };

  return <form onSubmit={form.handleSubmit(onSubmit)}>...</form>;
}
```

## 11.2 State Management Rules

| State Type | Tool | Contoh |
|---|---|---|
| Server state (API data) | TanStack Query | Products, transactions, bookings |
| Client state (UI) | Zustand | POS cart, filters, sidebar |
| Form state | React Hook Form | Input forms |
| URL state | React Router | Filters, pagination |
| Global app state | Zustand | Auth, current branch, theme |

**DILARANG**:
- Menyimpan server state di Zustand (gunakan React Query)
- Menyimpan form state di React Query (gunakan RHF)
- Menggunakan Context untuk state yang sering berubah

## 11.3 Data Fetching

```typescript
// ✅ CORRECT: React Query
export function useProducts(filters: ProductFilters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    staleTime: 30_000,
  });
}

// ❌ WRONG: useEffect + useState
const [products, setProducts] = useState([]);
useEffect(() => {
  fetch('/api/products').then(r => r.json()).then(setProducts);
}, []);
```

## 11.4 Mutation Pattern

```typescript
// ✅ CORRECT: React Query mutation + optimistic update
export function useCreateTransaction() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (input: CreateTransactionInput) => 
      createTransaction(input),
    onMutate: async (input) => {
      await queryClient.cancelQueries({ queryKey: ['transactions'] });
      const previous = queryClient.getQueryData(['transactions']);
      queryClient.setQueryData(['transactions'], old => [...old, optimisticData]);
      return { previous };
    },
    onError: (err, input, context) => {
      queryClient.setQueryData(['transactions'], context.previous);
      toast.error('Failed to create transaction');
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['transactions'] });
      queryClient.invalidateQueries({ queryKey: ['stock'] });
    },
  });
}
```

## 11.5 Realtime Subscription

```typescript
// ✅ CORRECT: React Query + Supabase Realtime
export function useRealtimeBookings(branchId: string) {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    const channel = supabase
      .channel(`bookings:${branchId}`)
      .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'bookings',
        filter: `branch_id=eq.${branchId}`,
      }, () => {
        queryClient.invalidateQueries({ queryKey: ['bookings', branchId] });
      })
      .subscribe();
    
    return () => { supabase.removeChannel(channel); };
  }, [branchId, queryClient]);
}
```

## 11.6 Error Handling di FE

```typescript
// ✅ CORRECT: Structured error handling
try {
  await createTransaction(input);
  toast.success('Transaction created');
} catch (err) {
  if (err instanceof AppError) {
    switch (err.code) {
      case 'STOCK_INSUFFICIENT':
        toast.error(`Stock not enough for ${err.details.product_name}`);
        break;
      case 'SCHEDULE_CONFLICT':
        toast.error('Schedule conflict, please choose another time');
        break;
      default:
        toast.error(err.message);
    }
  } else {
    toast.error('Unexpected error');
    Sentry.captureException(err);
  }
}
```

## 11.7 Accessibility

**WAJIB**:
- Semantic HTML (`button`, `a`, `input`, `label`)
- ARIA attributes untuk custom components
- Keyboard navigation support
- Screen reader friendly
- Color contrast WCAG AA
- Focus management untuk modal/dialog

---

# 📄 AGENT-012 — Security Protocol

## 12.1 Authentication

- Supabase Auth only
- MFA TOTP wajib untuk Super Admin & Owner
- Session timeout: 30 menit idle
- Password policy: min 8 char, complexity, rotation 90 hari

## 12.2 Authorization

- RBAC via roles & permissions
- Branch-scoped via RLS
- Permission check di SETIAP endpoint
- Field-level encryption untuk PII

## 12.3 Data Encryption

**PII yang WAJIB dienkripsi app-level**:
- Customer phone
- Customer email
- Employee ID number (KTP)
- Employee bank account
- Integration credentials
- Medical notes (optional)

**Algoritma**: AES-256-GCM

```typescript
// ✅ CORRECT: Encrypt PII
const encryptedPhone = await encrypt(customer.phone, ENCRYPTION_KEY);
await supabase.from('customers').insert({ phone: encryptedPhone });

// ❌ WRONG: Store PII plain
await supabase.from('customers').insert({ phone: customer.phone });
```

## 12.4 Input Validation

**WAJIB** di FE dan BE:

```typescript
// Shared schema
export const CreateCustomerSchema = z.object({
  full_name: z.string().min(1).max(200),
  phone: z.string().regex(/^\+62\d{8,12}$/),
  email: z.string().email().optional(),
});

// FE
const form = useForm({ resolver: zodResolver(CreateCustomerSchema) });

// BE
const input = CreateCustomerSchema.parse(body);
```

## 12.5 SQL Injection Prevention

**WAJIB** pakai parameterized queries / PostgREST:

```typescript
// ✅ CORRECT
const { data } = await supabase
  .from('customers')
  .select('*')
  .eq('phone_normalized', phone);

// ❌ WRONG: String concatenation
const { data } = await supabase.rpc('unsafe_query', {
  sql: `SELECT * FROM customers WHERE phone = '${phone}'`
});
```

## 12.6 XSS Prevention

- React auto-escape default
- Jangan pakai `dangerouslySetInnerHTML` kecuali sanitized
- CSP header aktif (lihat blueprint § 19.2)

## 12.7 Secret Management

**DILARANG**:
- Hardcode API key di code
- Commit `.env` file
- Log credentials
- Kirim secret via URL query

**WAJIB**:
- Supabase Vault untuk BE secrets
- Vercel Env untuk FE secrets
- Integration credentials encrypted di DB

## 12.8 Audit

**Setiap mutation WAJIB tercatat di `audit_logs`**:
- User ID
- Action (INSERT/UPDATE/DELETE)
- Table & record ID
- Before/after values
- Timestamp
- IP address
- User agent
- Branch ID

---

# 📄 AGENT-013 — Testing Protocol

## 13.1 Test Pyramid

~~~
╱╲
       ╱ E2E╲         Playwright (critical flows only)
      ╱──────╲
     ╱ Integra╲       Vitest + Supabase local
    ╱──────────╲
   ╱   Unit     ╲     Vitest (80% coverage target)
  ╱──────────────╲
~~~

## 13.2 Unit Test Requirements

**Coverage target**: 80% untuk business logic

**WAJIB test**:
- Zod schemas (validation)
- Calculation functions (totals, commission, payroll)
- Business rules (BR-XXX-YYY)
- Utility functions (formatters, validators)
- React hooks (custom hooks)

```typescript
// ✅ CORRECT: Test business rule
describe('BR-SALES-002: Stock cannot be negative', () => {
  it('should reject transaction when stock insufficient', async () => {
    await seedProduct({ id: 'p1', stock: 5 });
    
    const result = await createTransaction({
      items: [{ product_id: 'p1', qty: 10 }],
    });
    
    expect(result.error.code).toBe('STOCK_INSUFFICIENT');
  });
});
```

## 13.3 Integration Test Requirements

**Setup**: Supabase local (Docker) + seed data

**WAJIB test**:
- Edge Function end-to-end
- Database transactions
- RLS policies
- Trigger behavior
- Stock movements

## 13.4 E2E Test Requirements

**Critical flows WAJIB E2E**:
1. Login → Dashboard
2. POS: Create transaction → Receipt
3. Booking: Create → Confirm → Check-in → Complete
4. Vet: Create medical record → Finalize → Dispense
5. Purchase: Create PO → Receive → Pay
6. Payroll: Calculate → Approve

## 13.5 Test Naming

```typescript
// Format: should {expected behavior} when {condition}
it('should create transaction and reduce stock when input valid', async () => { ... });
it('should reject transaction when stock insufficient', async () => { ... });
it('should emit sales.transaction.created event after commit', async () => { ... });
```

## 13.6 Test Data

**WAJIB** pakai factory pattern:

```typescript
// tests/factories/customer.factory.ts
export function createCustomer(overrides?: Partial<Customer>): Customer {
  return {
    id: `cust_${uuid7()}`,
    full_name: faker.person.fullName(),
    phone: `+62${faker.string.numeric(10)}`,
    // ...
    ...overrides,
  };
}

// Usage
const customer = createCustomer({ full_name: 'John Doe' });
```

---

# 📄 AGENT-014 — Anti-Patterns (FORBIDDEN)

## 14.1 Daftar Anti-Pattern yang DILARANG

### AP-01: Hardcode Business Values
```typescript
// ❌ FORBIDDEN
const TAX_RATE = 0.11;
const REFUND_DAYS = 30;
const POINT_RATIO = 1000;
```

### AP-02: Bypass RLS
```typescript
// ❌ FORBIDDEN
const supabaseAdmin = createClient(URL, SERVICE_ROLE_KEY);
await supabaseAdmin.from('sales_transactions').insert({...});
// Hanya gunakan di Edge Functions dengan explicit audit
```

### AP-03: Direct State Mutation
```typescript
// ❌ FORBIDDEN
state.cart.items.push(newItem);
// ✅ CORRECT
setState(s => ({ ...s, cart: { ...s.cart, items: [...s.cart.items, newItem] } }));
```

### AP-04: Silent Error Swallowing
```typescript
// ❌ FORBIDDEN
try { ... } catch (e) { /* ignore */ }
try { ... } catch (e) { console.log(e); }
// ✅ CORRECT
try { ... } catch (e) { handleError(e); }
```

### AP-05: Inline Styles (kecuali dynamic)
```tsx
// ❌ FORBIDDEN
<div style={{ padding: '16px', color: 'red' }}>
// ✅ CORRECT
<div className="p-4 text-red-500">
```

### AP-06: Any Type
```typescript
// ❌ FORBIDDEN
const data: any = await fetch();
function process(data: any) { ... }
// ✅ CORRECT
const data: Customer = await fetch();
function process(data: Customer) { ... }
```

### AP-07: Magic Numbers
```typescript
// ❌ FORBIDDEN
if (status === 2) { ... }
setTimeout(callback, 86400000);
// ✅ CORRECT
if (status === BookingStatus.CONFIRMED) { ... }
setTimeout(callback, MS_PER_DAY);
```

### AP-08: Console.log di Production
```typescript
// ❌ FORBIDDEN
console.log('Debug:', data);
// ✅ CORRECT
logger.debug('Processing data', { data });
```

### AP-09: Nested Callbacks
```typescript
// ❌ FORBIDDEN
getUser(id, (user) => {
  getOrders(user.id, (orders) => {
    getItems(orders[0].id, (items) => { ... });
  });
});
// ✅ CORRECT
const user = await getUser(id);
const orders = await getOrders(user.id);
const items = await getItems(orders[0].id);
```

### AP-10: God Component
```tsx
// ❌ FORBIDDEN: Component > 500 lines
function SalesPage() { /* 1000 lines of everything */ }
// ✅ CORRECT: Decompose
function SalesPage() {
  return (
    <>
      <SalesHeader />
      <SalesFilters />
      <SalesTable />
      <SalesPagination />
    </>
  );
}
```

### AP-11: Prop Drilling
```tsx
// ❌ FORBIDDEN
<Parent a={a}>
  <Child a={a}>
    <GrandChild a={a} />
  </Child>
</Parent>
// ✅ CORRECT: Context or Zustand
const a = useStore(s => s.a);
```

### AP-12: useEffect sebagai Event Handler
```tsx
// ❌ FORBIDDEN
useEffect(() => {
  if (buttonClicked) {
    doSomething();
  }
}, [buttonClicked]);
// ✅ CORRECT
<button onClick={doSomething}>Click</button>
```

### AP-13: Uncontrolled Database Changes
```sql
-- ❌ FORBIDDEN: Direct UPDATE tanpa migration
UPDATE products SET price = 100000 WHERE id = '...';
-- ✅ CORRECT: Via migration
-- supabase/migrations/0043_update_product_prices.sql
```

### AP-14: Missing Error Boundary
```tsx
// ❌ FORBIDDEN
function App() { return <Dashboard />; }
// ✅ CORRECT
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Dashboard />
    </ErrorBoundary>
  );
}
```

### AP-15: Unindexed Queries
```typescript
// ❌ FORBIDDEN: Query tanpa index di kolom filter
const { data } = await supabase
  .from('sales_transactions')
  .select('*')
  .eq('notes', 'something');  // notes tidak di-index
```

### AP-16: Skipping Audit
```typescript
// ❌ FORBIDDEN: Update tanpa trigger audit
await supabase.from('customers').update({ phone: newPhone }).eq('id', id);
// ✅ CORRECT: Trigger audit otomatis attach
```

### AP-17: Missing Idempotency
```typescript
// ❌ FORBIDDEN: POST tanpa idempotency
await fetch('/pos-create-transaction', { method: 'POST', body: ... });
// ✅ CORRECT
await fetch('/pos-create-transaction', {
  method: 'POST',
  headers: { 'Idempotency-Key': crypto.randomUUID() },
  body: ...
});
```

### AP-18: Multi-Tenancy Pattern
```typescript
// ❌ FORBIDDEN: Schema per tenant
CREATE SCHEMA tenant_abc;
// ✅ CORRECT: Single schema + organization_id filter
SELECT * FROM customers WHERE organization_id = '...';
```

### AP-19: Synchronous Blocking
```typescript
// ❌ FORBIDDEN
const data = Deno.readFileSync('file.json');
// ✅ CORRECT
const data = await Deno.readFile('file.json');
```

### AP-20: Unencrypted PII
```typescript
// ❌ FORBIDDEN
await supabase.from('customers').insert({ phone: '+6281234567890' });
// ✅ CORRECT
const encrypted = await encrypt('+6281234567890', KEY);
await supabase.from('customers').insert({ phone: encrypted });
```

---

# 📄 AGENT-015 — Code Review Checklist

## 15.1 Pre-Submit Checklist (Self-Review)

**Sebelum membuat PR, agent WAJIB checklist**:

### Blueprint Alignment
- [ ] Sudah baca blueprint section terkait
- [ ] Implementasi sesuai blueprint (tidak ada deviasi)
- [ ] Business rules (BR-XXX-YYY) terpenuhi
- [ ] Permission matrix diikuti
- [ ] Workflow end-to-end sesuai blueprint § 18

### Code Quality
- [ ] Naming convention sesuai AGENT-005
- [ ] File structure sesuai AGENT-007.2
- [ ] Tidak ada anti-pattern di AGENT-014
- [ ] Tidak ada hardcode (AGENT-003 P-02)
- [ ] Type-safe (AGENT-003 P-03)
- [ ] Error handling proper (AGENT-007.4)
- [ ] Logging structured (AGENT-007.5)

### Database
- [ ] Migration numbered (AGENT-009.1)
- [ ] RLS enabled (AGENT-009.3)
- [ ] Audit trigger attached (AGENT-009.4)
- [ ] Indexes created untuk kolom filter
- [ ] Soft delete (tidak ada hard delete)
- [ ] UUID v7 primary key
- [ ] Money as bigint

### Backend
- [ ] Edge Function mengikuti template (AGENT-010.1)
- [ ] Shared modules digunakan (AGENT-010.2)
- [ ] Idempotency implemented
- [ ] Event emitted untuk state change
- [ ] Atomic transaction untuk multi-step
- [ ] Config dari `system_config`, bukan hardcode

### Frontend
- [ ] React Query untuk server state
- [ ] Zustand untuk client state
- [ ] RHF untuk form
- [ ] Zod schema shared dengan BE
- [ ] Optimistic update dengan rollback
- [ ] Accessibility (ARIA, keyboard nav)
- [ ] Error boundary

### Security
- [ ] PII encrypted
- [ ] Input validated (FE + BE)
- [ ] Permission checked
- [ ] No secrets in code
- [ ] SQL injection prevented
- [ ] XSS prevented

### Testing
- [ ] Unit tests (80% coverage)
- [ ] Integration tests untuk Edge Function
- [ ] E2E untuk critical flow
- [ ] Test naming sesuai AGENT-013.5
- [ ] Test data via factory

### Documentation
- [ ] Comment dengan referensi blueprint
- [ ] README updated (jika perlu)
- [ ] ADR created (jika ada keputusan baru)

## 15.2 Reviewer Checklist

**Reviewer WAJIB verifikasi**:
- [ ] Semua checklist di 15.1 terpenuhi
- [ ] Tidak ada "looks good to me" tanpa review detail
- [ ] Blueprint reference valid
- [ ] Performance impact dinilai
- [ ] Security impact dinilai
- [ ] Breaking changes flagged

---

# 📄 AGENT-016 — Commit & PR Protocol

## 16.1 Commit Message Format

**Conventional Commits** WAJIB:

~~~
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
| Type | Kapan |
|---|---|
| `feat` | Fitur baru |
| `fix` | Bug fix |
| `docs` | Dokumentasi saja |
| `style` | Formatting, tidak ada logic change |
| `refactor` | Refactor tanpa change behavior |
| `perf` | Performance improvement |
| `test` | Add/correct tests |
| `build` | Build system atau external dependency |
| `ci` | CI configuration |
| `chore` | Lain-lain yang tidak fit di atas |
| `db` | Database migration |

### Scope
Modul atau area: `pos`, `vet`, `inventory`, `auth`, `db`, `ui`, dll

### Subject
- Max 72 chars
- Imperative mood ("add" bukan "added")
- Lowercase first letter
- No period at end

### Body
- Why (bukan what — what sudah di subject)
- Reference blueprint: `Ref: PETORA-BLUEPRINT-XXX`
- Reference issue: `Closes #123`

### Footer
- Breaking changes: `BREAKING CHANGE: ...`
- Issue reference: `Closes #123`

### Contoh

```
feat(pos): add multi-payment support

Implement multi-payment method in POS terminal to allow customers
to split payment across cash, QRIS, and debit in single transaction.

Ref: PETORA-BLUEPRINT-009 § 9.1
Ref: PETORA-BLUEPRINT-006 § 6.2.1 (BR-SALES-008)
Closes #42
db(sales): add sales_transactions table

Create sales_transactions, sales_transaction_items, and sales_payments
tables with RLS policies, audit triggers, and indexes.

Ref: PETORA-BLUEPRINT-003 § 3.6
fix(vet): prevent medical record edit after finalized

Add status check to reject update attempts on finalized medical
records. Amendments must go through amendment flow with snapshot.

Ref: PETORA-BLUEPRINT-006 § 6.2.4 (BR-VET-002, BR-VET-003)
Fixes #87
```

## 16.2 Branch Naming

```
<type>/<ticket>-<short-description>

Contoh:
feat/POS-42-multi-payment
fix/VET-87-finalized-record-edit
db/INV-15-stock-movement-table
refactor/AUTH-23-mfa-flow
~~~

## 16.3 PR Template

```markdown
## Deskripsi
<!-- Jelaskan apa yang dilakukan PR ini -->

## Blueprint Reference
- Ref: PETORA-BLUEPRINT-XXX § Y.Y
- Ref: PETORA-BLUEPRINT-XXX § Y.Y

## Tipe Perubahan
- [ ] Fitur baru
- [ ] Bug fix
- [ ] Refactor
- [ ] Database migration
- [ ] Dokumentasi
- [ ] Testing

## Checklist
- [ ] Sudah baca blueprint section terkait
- [ ] Implementasi sesuai blueprint
- [ ] Business rules terpenuhi
- [ ] Unit tests (80% coverage)
- [ ] Integration tests
- [ ] E2E tests (jika critical flow)
- [ ] Self-review selesai (AGENT-015.1)
- [ ] Tidak ada anti-pattern (AGENT-014)
- [ ] Tidak ada hardcode
- [ ] PII encrypted (jika ada)
- [ ] Audit log verified
- [ ] RLS policies verified
- [ ] Documentation updated
- [ ] Commit messages conventional

## Testing
<!-- Jelaskan bagaimana perubahan di-test -->

## Screenshots (jika UI change)
<!-- Sertakan screenshot -->

## Breaking Changes
<!-- List breaking changes, atau tulis "None" -->

## Additional Notes
<!-- Catatan tambahan untuk reviewer -->
```

## 16.4 PR Rules

- 1 PR = 1 concern (jangan campur fitur unrelated)
- Max 400 files changed (lebih dari itu, split)
- Wajib 1 approval dari reviewer
- CI harus hijau
- No merge commit (squash merge only)
- PR description wajib isi template

---

# 📄 AGENT-017 — Decision-Making Protocol

## 17.1 Hierarki Keputusan

~~~
[1] Blueprint (PETORA-BLUEPRINT-XXX)     ← HIGHEST AUTHORITY
  ↓ jika tidak eksplisit
[2] Existing code pattern                ← FOLLOW THE PATTERN
  ↓ jika tidak ada pattern
[3] ADR baru (butuh approval)            ← PROPOSE VIA ADR
  ↓ jika urgent
[4] ESCALATE ke user                     ← ASK, DON'T ASSUME
~~~

## 17.2 Keputusan yang Boleh Agent Ambil

✅ Agent BOLEH putuskan sendiri:
- Nama variabel lokal
- Struktur komponen internal (selama sesuai konvensi)
- Implementasi detail yang blueprint tidak spesifik
- Test case tambahan
- Comment & documentation
- Refactoring kecil (tanpa change behavior)

## 17.3 Keputusan yang BUTUH Approval

⚠️ Agent WAJIB escalate:
- Penambahan dependency baru
- Perubahan schema database yang tidak ada di blueprint
- Perubahan business rule
- Perubahan arsitektur
- Perubahan permission matrix
- Perubahan workflow
- Penambahan modul baru
- Perubahan API contract
- Performance trade-off yang signifikan
- Security trade-off

## 17.4 Format ADR (Architecture Decision Record)

```markdown
# ADR-XXX: {Judul Keputusan}

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-YYY

## Context
Apa masalah atau kebutuhan yang memicu keputusan ini?

## Decision
Apa keputusannya?

## Consequences
Apa konsekuensi (positif & negatif)?

## Blueprint Reference
Ref: PETORA-BLUEPRINT-XXX § Y.Y

## Alternatives Considered
Alternatif apa saja yang dipertimbangkan dan kenapa ditolak?
```

---

# 📄 AGENT-018 — Escalation & Clarification Protocol

## 18.1 Kapan Harus Escalate

**ESCALATE ke user jika**:

1. Blueprint ambigu atau kontradiktif
2. Blueprint silent (tidak ada bahasan)
3. Ada trade-off yang butuh business decision
4. Ada konflik antara 2 section blueprint
5. Implementasi butuh informasi di luar blueprint
6. Ada edge case yang blueprint tidak cover
7. Butuh konfirmasi untuk interpretasi
8. Ada risiko security/compliance yang tidak jelas

## 18.2 Format Escalation

```markdown
## 🚨 ESCALATION REQUIRED

### Context
<!-- Jelaskan situasi -->

### Blueprint Reference
Ref: PETORA-BLUEPRINT-XXX § Y.Y

### Issue
<!-- Jelaskan masalah/ambiguitas -->

### Options
1. **Option A**: ...
   - Pros: ...
   - Cons: ...
2. **Option B**: ...
   - Pros: ...
   - Cons: ...
3. **Option C**: ...
   - Pros: ...
   - Cons: ...

### Recommendation
<!-- Rekomendasi agent dengan alasan -->

### Impact
<!-- Apa impact dari keputusan ini -->

### Urgency
- [ ] Blocker (tidak bisa lanjut tanpa jawaban)
- [ ] High (butuh jawaban segera)
- [ ] Medium (butuh jawaban hari ini)
- [ ] Low (bisa ditunda)
```

## 18.3 JANGAN Escalate Untuk

❌ Jangan escalate untuk:
- Hal yang sudah eksplisit di blueprint
- Pilihan implementasi detail (nama variabel, struktur file internal)
- Hal yang sudah ada pattern di existing code
- Preferensi kosmetik (warna, spacing)
- Hal yang bisa diputuskan dengan common sense engineering

## 18.4 Default Behavior

**Jika blueprint silent dan escalation tidak urgent**:
1. Ikuti pattern existing
2. Dokumentasikan asumsi di comment
3. Tandai sebagai "TODO: confirm with product"
4. Lanjut implementasi
5. Escalate di akhir sprint

---

# 📄 AGENT-019 — Verification Checklist

## 19.1 Pre-Deployment Checklist

**Sebelum deploy ke production, WAJIB verifikasi**:

### Infrastructure
- [ ] Supabase project created (production)
- [ ] Vercel project created (3 apps: staff, pos, customer)
- [ ] Domain configured di Cloudflare
- [ ] SSL certificate active
- [ ] Environment variables set
- [ ] Secrets in Vault

### Database
- [ ] All migrations applied
- [ ] RLS policies enabled on all tables
- [ ] Indexes created
- [ ] Triggers attached
- [ ] Seed data (default roles, permissions, COA, config)
- [ ] Backup schedule configured
- [ ] Test restore performed

### Backend
- [ ] All Edge Functions deployed
- [ ] Integration credentials encrypted
- [ ] Cron jobs scheduled
- [ ] Error tracking (Sentry) connected
- [ ] Health check endpoint working

### Frontend
- [ ] All 3 apps deployed
- [ ] CSP headers configured
- [ ] Source maps uploaded to Sentry
- [ ] PWA manifest (optional)

### Security
- [ ] Penetration test performed
- [ ] MFA enabled for admin
- [ ] Audit log verified
- [ ] PII encrypted
- [ ] Rate limiting active
- [ ] HTTPS enforced

### Business
- [ ] Default config populated
- [ ] Default roles & permissions
- [ ] Default COA
- [ ] Default tax profile
- [ ] Default document numbering
- [ ] Test transaction flow end-to-end

### Documentation
- [ ] User manual (staff)
- [ ] User manual (customer)
- [ ] Admin guide
- [ ] API documentation
- [ ] Troubleshooting guide

## 19.2 Module Completion Checklist

**Untuk setiap modul, verifikasi**:

- [ ] Database schema sesuai blueprint
- [ ] RLS policies aktif
- [ ] Edge Functions implemented
- [ ] Business rules (BR-XXX-YYY) enforced
- [ ] Permissions configured
- [ ] Events emitted
- [ ] UI implemented sesuai routing blueprint
- [ ] Tests (unit, integration, E2E)
- [ ] Documentation
- [ ] Zero hardcode
- [ ] Audit log working

## 19.3 Business Rules Verification

**Untuk setiap BR-XXX-YYY di blueprint**:

- [ ] Rule diimplementasikan
- [ ] Rule di-test (unit + integration)
- [ ] Error message user-friendly
- [ ] Edge cases covered

---

# 📄 AGENT-020 — Self-Audit Protocol

## 20.1 Agent Self-Audit

**Setiap agent WAJIB melakukan self-audit sebelum menyelesaikan task**:

~~~
[1] Apakah saya sudah baca blueprint section terkait?
  ↓
[2] Apakah implementasi saya 100% sesuai blueprint?
  ↓
[3] Apakah ada anti-pattern yang saya lakukan?
  ↓
[4] Apakah ada hardcode yang tersisa?
  ↓
[5] Apakah semua tests pass?
  ↓
[6] Apakah saya sudah escalate hal yang tidak jelas?
  ↓
[7] Apakah saya sudah dokumentasikan keputusan?
  ↓
[8] Apakah code saya bisa di-review dengan mudah?
```

## 20.2 Self-Audit Report Format

```markdown
## 🤖 Agent Self-Audit Report

**Task**: {deskripsi task}
**Blueprint Ref**: PETORA-BLUEPRINT-XXX § Y.Y
**Agent**: {agent name/id}
**Date**: {date}

### Checklist
- [x] Blueprint read
- [x] Implementation aligned
- [x] No anti-patterns
- [x] No hardcode
- [x] Tests pass
- [x] Documentation updated

### Issues Found
<!-- List issue yang ditemukan selama implementasi -->

### Escalations Made
<!-- List hal yang di-escalate ke user -->

### Deviations
<!-- List deviasi dari blueprint (jika ada, dengan alasan) -->

### Recommendations
<!-- Rekomendasi untuk improvement -->
```

## 20.3 Continuous Learning

**Agent WAJIB**:
- Belajar dari setiap code review
- Update pengetahuan tentang blueprint
- Flag blueprint yang perlu di-update
- Suggest ADR untuk pattern baru yang berhasil

---

# 🎯 PENUTUP

## Ringkasan Prinsip

> **Blueprint adalah konstitusi. Dokumen ini adalah undang-undang pelaksanaannya.**
>
> Setiap baris kode, setiap keputusan, setiap PR — **WAJIB tunduk pada kedua dokumen ini.**
>
> Jika ragu → **ESCALATE**. Jangan asumsikan.
> Jika blueprint silent → **FOLLOW PATTERN**. Jangan mengarang.
> Jika ada konflik → **BLUEPRINT WINS**. Selalu.

## Quick Reference

| Situasi | Action |
|---|---|
| Task implementasi | Baca blueprint → implementasi sesuai |
| Blueprint ambigu | Escalate ke user (AGENT-018) |
| Blueprint silent | Follow pattern existing |
| Butuh keputusan baru | Buat ADR, tunggu approval |
| Ada bug | Cek blueprint → fix sesuai blueprint |
| Review code | Checklist AGENT-015 |
| Commit | Format AGENT-016 |
| PR | Template AGENT-016.3 |
| Conflict dengan blueprint | Blueprint wins |

## Kontak & Escalation

- **Product Owner**: untuk business decision
- **Tech Lead**: untuk arsitektur decision
- **Security Officer**: untuk security concern
- **DBA**: untuk database concern

---

**Dokumen ini adalah aturan perilaku mengikat.**
**Versi: 1.0.0 | Tanggal: 07 November 2026**
**Status: ENFORCED — Setiap agent dan developer WAJIB tunduk**

**PETORA — One Platform, Complete Pet Care Management.**
