# 🐾 PETORA — FINAL BLUEPRINT v1.0
## Integrated Petshop & Pet Care Management System
### Dokumen Spesifikasi Teknis & Bisnis Final (End-to-End)

---

## 📑 DAFTAR DOKUMEN BLUEPRINT (INDEX)

| ID Dokumen | Judul | Scope |
|---|---|---|
| `PETORA-BLUEPRINT-000` | Meta & Konvensi Dokumen | Konvensi penamaan, ID, versi |
| `PETORA-BLUEPRINT-001` | Decision Matrix & Prinsip Arsitektur | Keputusan final bisnis/teknis |
| `PETORA-BLUEPRINT-002` | Arsitektur Sistem & Stack Teknologi | Stack, topologi, deployment |
| `PETORA-BLUEPRINT-003` | Database Schema & Design | Tabel, relasi, RLS, index |
| `PETORA-BLUEPRINT-004` | Backend Architecture | Edge Functions, services, API |
| `PETORA-BLUEPRINT-005` | Frontend Architecture | React, routing, state, UI kit |
| `PETORA-BLUEPRINT-006` | Domain Models & Business Rules | Entitas, enum, invariants |
| `PETORA-BLUEPRINT-007` | Modul A — Organisasi & Dashboard | Org, user, role, branch |
| `PETORA-BLUEPRINT-008` | Modul B — Pelanggan, Hewan, Produk | Customer, pet, product, inventory |
| `PETORA-BLUEPRINT-009` | Modul C — Penjualan & Pelayanan | POS, purchasing, grooming, booking, hotel, transport |
| `PETORA-BLUEPRINT-010` | Modul D — Klinik & Farmasi | Vet, medical record, pharmacy |
| `PETORA-BLUEPRINT-011` | Modul E — Membership, Loyalty, Promo | Membership, loyalty, promotion |
| `PETORA-BLUEPRINT-012` | Modul F — SDM | Staff, attendance, commission, payroll |
| `PETORA-BLUEPRINT-013` | Modul G — Keuangan & Akuntansi | Finance, accounting, consignment, report |
| `PETORA-BLUEPRINT-014` | Modul H — Komunikasi & Portal | Notification, customer portal, settings |
| `PETORA-BLUEPRINT-015` | Modul I — Fitur Tambahan | CRM, complaint, lab, surgery, asset, SOP, doc, delivery, budget, tax, backup, ecommerce |
| `PETORA-BLUEPRINT-016` | Super Admin Panel & Konfigurasi | Panel konfigurasi global |
| `PETORA-BLUEPRINT-017` | Security, Audit & Compliance | Auth, RLS, audit, enkripsi |
| `PETORA-BLUEPRINT-018` | Workflow End-to-End | Alur bisnis antar-modul |
| `PETORA-BLUEPRINT-019` | Deployment & Infrastructure | Vercel/CF, env, CI/CD |
| `PETORA-BLUEPRINT-020` | Testing, QA & Acceptance | Test strategy, acceptance criteria |
| `PETORA-BLUEPRINT-021` | Glossary & Appendix | Istilah, referensi |

---

# 📄 PETORA-BLUEPRINT-000
## Meta & Konvensi Dokumen

### 0.1 Tujuan Dokumen
Dokumen ini adalah **blueprint final** yang menjadi acuan tunggal pengembangan PETORA sampai produk siap deployment. Tidak ada fase/roadmap sementara — semua keputusan teknis, bisnis, dan arsitektur sudah ditetapkan di sini.

### 0.2 Konvensi Penamaan

| Konteks | Format | Contoh |
|---|---|---|
| ID Dokumen | `PETORA-BLUEPRINT-XXX` | `PETORA-BLUEPRINT-003` |
| ID Modul | `MOD-{KODE}` | `MOD-POS`, `MOD-VET` |
| ID Tabel DB | `snake_case` | `sales_transactions` |
| ID Kolom DB | `snake_case` | `created_at` |
| ID Edge Function | `kebab-case` | `pos-create-transaction` |
| ID React Component | `PascalCase` | `SalesTransactionForm` |
| ID React Hook | `use{Entity}{Action}` | `useCreateTransaction` |
| ID Route | `kebab-case` | `/sales/transactions` |
| ID Enum Value | `UPPER_SNAKE` | `PAYMENT_CASH` |
| ID Status | `lower_snake` | `pending_approval` |
| ID Event Bus | `domain.event.action` | `sales.transaction.created` |
| ID Konfigurasi | `dot.case` | `loyalty.point_ratio` |
| ID Permission | `module:resource:action` | `pos:transaction:create` |

### 0.3 Konvensi ID Unik Global

Setiap entitas bisnis menggunakan **UUID v7** (time-sortable) sebagai primary key. Alasan:
- Sortable by time → optimal untuk index B-tree
- Tidak perlu koordinasi antar-node
- Aman untuk publik (tidak leak urutan)

Prefix opsional untuk debugging:
- `cust_`, `pet_`, `prod_`, `trx_`, `book_`, `med_`, `inv_`, `usr_`, `brn_`, `spl_`, `emp_`, `pay_`, `vou_`

Format: `{prefix}_{uuid7}` contoh: `trx_01923a7b-8c3d-7f0e-9a1b-2c3d4e5f6a7b`

### 0.4 Konvensi Timestamp
- Semua timestamp disimpan dalam **UTC** (`timestamptz`)
- Ditampilkan ke user dalam **timezone cabang** (`branch.timezone`)
- Format ISO 8601 di API: `2026-11-07T14:30:00Z`

### 0.5 Konvensi Soft Delete
- Tidak ada hard delete untuk entitas bisnis
- Kolom `deleted_at timestamptz NULL` → soft delete
- Unique constraint menggunakan **partial index**: `WHERE deleted_at IS NULL`
- Audit log tetap mencatat sebelum soft delete

### 0.6 Konvensi Multi-Branch Scoping
Setiap tabel transaksional memiliki kolom:
- `organization_id uuid NOT NULL` — pemilik data (1 organisasi = 1 perusahaan)
- `branch_id uuid NULL` — cabang pemilik data (NULL = data pusat/global)
- `created_by uuid NOT NULL` — user pembuat
- `updated_by uuid NULL` — user terakhir mengubah
- `created_at timestamptz NOT NULL DEFAULT now()`
- `updated_at timestamptz NOT NULL DEFAULT now()`
- `deleted_at timestamptz NULL`

### 0.7 Versioning Dokumen
- Versi blueprint: `1.0.0` (final)
- Perubahan kecil (typo, klarifikasi): bump patch `1.0.x`
- Perubahan kolom/endpoint: bump minor `1.x.0` + migration
- Perubahan arsitektur: bump major `x.0.0` + ADR baru

---

# 📄 PETORA-BLUEPRINT-001
## Decision Matrix & Prinsip Arsitektur

### 1.1 Keputusan Arsitektur Final (ADR — Architecture Decision Records)

| ID | Keputusan | Alasan | Alternatif yang Ditolak |
|---|---|---|---|
| `ADR-001` | **Single-tenant, multi-branch** | Target: 1 perusahaan multi-cabang. Lebih sederhana, performa lebih baik, backup/restore per-org trivial. | Multi-tenant SaaS |
| `ADR-002` | **Modular Monolith** di Supabase Edge Functions | Satu codebase, deploy cepat, tidak perlu orkestrasi microservice. Dipisah per-domain via folder. | Microservice, serverless per-feature |
| `ADR-003` | **Supabase (PostgreSQL + RLS + Edge Functions + Storage + Realtime)** | Managed, Postgres-native, RLS built-in, realtime built-in, harga predictable. | Firebase, AWS Amplify, custom backend |
| `ADR-004` | **React 18 + TypeScript + Vite** | Ekosistem matang, typing kuat, build cepat. | Next.js (overkill untuk SPA), Vue |
| `ADR-005` | **State: Zustand + React Query (TanStack Query)** | Zustand untuk client state, RQ untuk server state + cache. | Redux (boilerplate), Context-only (re-render) |
| `ADR-006` | **UI Kit: shadcn/ui + Tailwind CSS + Radix** | Copy-paste component, customizable, accessible. | Material UI (berat), Ant Design (gaya tidak cocok) |
| `ADR-007` | **Form: React Hook Form + Zod** | Performa tinggi, validasi skema terpusat (FE+BE). | Formik, Yup |
| `ADR-008` | **Table: TanStack Table v8** | Headless, virtualized, fleksibel. | AG Grid (berbayar), MUI Table |
| `ADR-009` | **PDF: @react-pdf/renderer (client) + Edge Function Puppeteer (server)** | Client untuk preview, server untuk invoice final. | jsPDF (kurang fitur) |
| `ADR-010` | **Chart: Recharts** | React-native, cukup untuk dashboard. | Chart.js, ApexCharts |
| `ADR-011` | **Date: date-fns + dayjs** | date-fns untuk logic, dayjs untuk display. | moment (legacy) |
| `ADR-012` | **ID: UUID v7** | Time-sortable, global unique. | UUID v4, ULID, nanoid |
| `ADR-013` | **Auth: Supabase Auth + MFA TOTP** | Integrated, support MFA, magic link, SSO. | Auth0 (mahal), custom JWT |
| `ADR-014` | **Storage: Supabase Storage** | Integrated, RLS support. | S3 direct, Cloudinary |
| `ADR-015` | **Realtime: Supabase Realtime** | Built-in, Postgres CDC. | Pusher, Socket.io custom |
| `ADR-016` | **Queue/Job: Supabase pg_cron + Edge Function** | Cron native Postgres, trigger Edge Function. | Inngest, BullMQ (butuh Redis) |
| `ADR-017` | **Search: Postgres FTS + pg_trgm** | Cukup untuk skala petshop, zero extra infra. | Algolia, Meilisearch |
| `ADR-018` | **Deployment: Vercel (FE) + Supabase Cloud (BE)** | Vercel optimal untuk React SPA, Supabase managed. | Cloudflare Pages, Netlify |
| `ADR-019` | **CI/CD: GitHub Actions** | Standard, free tier cukup. | GitLab CI, CircleCI |
| `ADR-020` | **Monitoring: Sentry (FE) + Supabase Dashboard (BE)** | Error tracking + DB monitoring. | Datadog (mahal) |
| `ADR-021` | **Feature Flag: DB-driven via `system_config`** | No extra service, bisa diubah via Super Admin panel. | LaunchDarkly, Unleash |
| `ADR-022` | **Konfigurasi: DB-driven, no hardcode** | Semua config di tabel `system_config`, editable via Super Admin. | env-only, JSON file |
| `ADR-023` | **Integrasi: Plugin registry di DB** | Setiap integrasi (WA, email, payment) adalah plugin dengan config terenkripsi. | Hardcode per-provider |
| `ADR-024` | **Multi-currency: Single currency per org (IDR)** | Petshop Indonesia, tidak perlu multi-currency. | Multi-currency (kompleks) |
| `ADR-025` | **Tax: Configurable via `tax_profiles`** | PPN/PPnBM configurable, bisa multi-rate. | Hardcode 11% |
| `ADR-026` | **Inventory valuation: Moving Average (default) + FIFO (optional)** | Moving average sederhana, FIFO untuk farmasi. | LIFO (tidak umum) |
| `ADR-027` | **Stock policy: No negative stock by default (configurable)** | Mencegah data invalid. | Allow negative |
| `ADR-028` | **Document numbering: Per-branch, per-prefix, per-period** | Format: `{prefix}/{branch_code}/{YYYY}/{MM}/{sequence}` | Global sequence |
| `ADR-029` | **Audit log: Append-only table + trigger-based** | Semua perubahan penting tercatat otomatis. | Application-level only |
| `ADR-030` | **Encryption at rest: Postgres TDE (Supabase managed) + app-level untuk PII** | PII (phone, email, medical) dienkripsi app-level dengan AES-256-GCM. | DB-only encryption |

### 1.2 Prinsip Arsitektur

1. **Single Source of Truth**: Setiap data hanya ditulis di satu tempat, tempat lain membaca.
2. **Event-Driven Internal**: Perubahan state penting memicu event internal (via DB trigger + pg_notify) untuk konsistensi antar-modul.
3. **Idempotency First**: Setiap mutation endpoint menerima `idempotency_key` untuk mencegah duplikasi.
4. **Optimistic UI dengan Rollback**: UI update duluan, rollback jika server error.
5. **Fail-Safe Defaults**: Jika config tidak ditemukan, gunakan default aman (bukan crash).
6. **Zero Hardcode**: Semua angka bisnis (pajak, poin, denda, threshold) di `system_config`.
7. **Audit Everything**: Setiap mutation tercatat di `audit_logs` dengan before/after snapshot.
8. **Branch Isolation**: Data cabang terisolasi via RLS, kecuali data global (produk master, supplier).
9. **Offline-First (Best Effort)**: LocalStorage queue untuk POS saat offline, sync saat online.
10. **Type-Safe End-to-End**: Zod schema di-shared antara FE dan BE via package `@petora/schemas`.

### 1.3 Keputusan Bisnis Final

| Aspek | Keputusan |
|---|---|
| Model bisnis software | Internal tool (bukan SaaS komersial) |
| Target user | 1 perusahaan petshop/petcare, 1+ cabang |
| Bahasa UI | Indonesia (default), English (opsional via i18n) |
| Mata uang | IDR (Rupiah), format: `Rp1.000.000` |
| Format tanggal | `DD MMM YYYY` (contoh: `07 Nov 2026`) |
| Format waktu | 24-jam `HH:mm` |
| Timezone default | `Asia/Jakarta` (WIB), configurable per branch |
| Pajak default | PPN 11% (configurable) |
| Satuan default | Pcs, Kg, Liter (configurable per produk) |
| Batch tracking | Wajib untuk produk farmasi & makanan, opsional untuk aksesoris |
| Kedaluwarsa | Wajib dicatat jika produk punya FE |
| Membership | Tier-based (configurable: Bronze/Silver/Gold/Platinum) |
| Loyalty point | 1 point = Rp100 (configurable), earn 1 point per Rp1000 spent |
| Komisi | Per-staff, per-layanan, per-produk (configurable) |
| Refund | Wajib approval manager, max 30 hari dari transaksi |
| Void | Hanya sebelum payment, butuh approval supervisor |
| Diskon bertingkat | Maks 3 level (diskon item → diskon cart → voucher) |
| Pembayaran | Cash, Transfer, QRIS, Debit, Credit, Multi-payment |
| Kredit pelanggan | Opsional, butuh approval, max limit configurable |
| Pet hotel pricing | Per-hari, per-jam (after cut-off), per-jenis kamar |
| Grooming pricing | Per-jenis hewan × ukuran × layanan |
| Konsultasi dokter | Flat fee per kunjungan (configurable) |
| Rekam medis | Finalisasi wajib, revisi via snapshot |
| Obat keras | Wajib resep dokter, stok terpisah |

---

# 📄 PETORA-BLUEPRINT-002
## Arsitektur Sistem & Stack Teknologi

### 2.1 Topologi Sistem

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────┐ │
│  │  Staff Dashboard │  │  POS Terminal   │  │ Customer   │ │
│  │  (React SPA)    │  │  (React SPA)    │  │ Portal     │ │
│  │  Vercel Edge    │  │  Vercel Edge    │  │ (React SPA)│ │
│  └────────┬────────┘  └────────┬────────┘  └─────┬──────┘ │
└───────────┼────────────────────┼──────────────────┼────────┘
            │                    │                  │
            ▼                    ▼                  ▼
┌─────────────────────────────────────────────────────────────┐
│                     SUPABASE PLATFORM                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  API Gateway (PostgREST + Custom Routes)             │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────┼──────────────────────────────┐  │
│  │  Edge Functions (Deno, TypeScript)                   │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │  │
│  │  │ POS      │ │ VET      │ │ PAYROLL  │ │ NOTIFY │ │  │
│  │  │ Function │ │ Function │ │ Function │ │ Func   │ │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────┘ │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│  ┌──────────────────────┼──────────────────────────────┐  │
│  │  PostgreSQL 15 (with extensions)                     │  │
│  │  - pgcrypto, pg_trgm, pg_cron, pgjwt                 │  │
│  │  - Row Level Security (RLS)                          │  │
│  │  - Logical Replication (Realtime)                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                         │                                   │
│  ┌──────────┐ ┌─────────┴──────┐ ┌────────────────────┐  │
│  │ Storage  │ │ Realtime       │ │ Auth               │  │
│  │ (S3)     │ │ (WebSocket)    │ │ (GoTrue)           │  │
│  └──────────┘ └────────────────┘ └────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────┐
│                  EXTERNAL INTEGRATIONS                        │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────┐ │
│  │ WhatsApp│ │ Email  │ │ Payment│ │ Printer│ │ E-Commerce│ │
│  │ (Fonnte│ │ (Resend│ │ (Midtrans)│ │ (ESC/POS)│ │ (Custom) │ │
│  │  /Wablas)│ │  /SMTP)│ │        │ │        │ │          │ │
│  └────────┘ └────────┘ └────────┘ └────────┘ └──────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Stack Teknologi Final

#### Frontend
| Layer | Teknologi | Versi | Alasan |
|---|---|---|---|
| Framework | React | 18.3+ | Stable, ecosystem |
| Language | TypeScript | 5.4+ | Type safety |
| Build tool | Vite | 5.x | Fast HMR |
| Routing | React Router | 6.x | Standard SPA routing |
| State (client) | Zustand | 4.x | Lightweight |
| State (server) | TanStack Query | 5.x | Cache, refetch, optimistic |
| Forms | React Hook Form + Zod | 7.x / 3.x | Performant, shared schema |
| UI Components | shadcn/ui | latest | Copy-paste, customizable |
| Styling | Tailwind CSS | 3.4+ | Utility-first |
| Icons | Lucide React | latest | Consistent |
| Table | TanStack Table | 8.x | Headless, virtualized |
| Charts | Recharts | 2.x | React-native |
| Date | date-fns + dayjs | 3.x / 1.11 | Lightweight |
| PDF (client) | @react-pdf/renderer | 3.x | Preview |
| Barcode | JsBarcode | 3.x | Generate barcode |
| QR Code | qrcode.react | 3.x | Generate QR |
| Print (thermal) | react-thermal-printer | 1.x | ESC/POS via USB/Bluetooth |
| i18n | react-i18next | 13.x | Multi-language |
| Error tracking | Sentry | 7.x | FE error monitoring |
| Testing | Vitest + Testing Library | 1.x / 14.x | Fast, React-friendly |
| E2E | Playwright | 1.x | Cross-browser |

#### Backend (Supabase)
| Layer | Teknologi | Versi | Alasan |
|---|---|---|---|
| Database | PostgreSQL | 15 | Stable, extension-rich |
| Auth | Supabase Auth (GoTrue) | latest | Integrated |
| API | PostgREST (auto) + Edge Functions | latest | Auto CRUD + custom logic |
| Edge Runtime | Deno | 1.40+ | TypeScript-native, secure |
| Storage | Supabase Storage | latest | Integrated, RLS |
| Realtime | Supabase Realtime | latest | Postgres CDC |
| Cron | pg_cron | 1.6+ | Scheduled jobs |
| Search | Postgres FTS + pg_trgm | native | Zero extra infra |

#### Shared Package
| Package | Isi |
|---|---|
| `@petora/schemas` | Zod schemas (FE+BE shared) |
| `@petora/types` | TypeScript types (generated from Zod) |
| `@petora/constants` | Enums, constants |
| `@petora/utils` | Utility functions (format, validate) |

#### Infrastructure
| Layer | Teknologi | Alasan |
|---|---|---|
| FE Hosting | Vercel | Optimal untuk React SPA |
| BE Hosting | Supabase Cloud | Managed Postgres |
| Domain | Cloudflare | DNS + CDN + DDoS protection |
| CI/CD | GitHub Actions | Standard |
| Secret management | Supabase Vault + Vercel Env | Encrypted |
| Monitoring | Sentry (FE) + Supabase Dashboard (BE) | Error + DB |
| Backup | Supabase automatic + daily dump to R2 | Redundant |

### 2.3 Struktur Folder Monorepo

```
petora/
├── apps/
│   ├── web-staff/              # Staff dashboard (Vite + React)
│   │   ├── src/
│   │   │   ├── app/            # App shell, providers
│   │   │   ├── pages/          # Route-based pages
│   │   │   ├── features/       # Feature modules (domain-based)
│   │   │   │   ├── pos/
│   │   │   │   ├── inventory/
│   │   │   │   ├── grooming/
│   │   │   │   ├── vet/
│   │   │   │   └── ...
│   │   │   ├── components/     # Shared components
│   │   │   ├── hooks/          # Shared hooks
│   │   │   ├── lib/            # Utils, API client
│   │   │   ├── stores/         # Zustand stores
│   │   │   └── styles/         # Global CSS
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   ├── web-pos/                # POS terminal (optimized for touchscreen)
│   │   └── ...
│   │
│   └── web-customer/           # Customer portal
│       └── ...
│
├── packages/
│   ├── schemas/                # Zod schemas (shared)
│   ├── types/                  # Generated TS types
│   ├── constants/              # Enums, constants
│   ├── utils/                  # Shared utilities
│   └── ui/                     # Shared UI components (shadcn-based)
│
├── supabase/
│   ├── migrations/             # SQL migrations (numbered)
│   │   ├── 0001_init.sql
│   │   ├── 0002_rls.sql
│   │   └── ...
│   ├── functions/              # Edge Functions
│   │   ├── _shared/            # Shared code
│   │   │   ├── database.ts
│   │   │   ├── auth.ts
│   │   │   ├── config.ts
│   │   │   └── events.ts
│   │   ├── pos-create-transaction/
│   │   │   └── index.ts
│   │   ├── vet-finalize-record/
│   │   │   └── index.ts
│   │   └── ...
│   ├── seed/                   # Seed data
│   └── config.toml
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
│
├── turbo.json                  # Turborepo config
├── package.json
└── README.md
```

### 2.4 Konvensi Edge Function

Setiap Edge Function mengikuti struktur:

```typescript
// supabase/functions/{function-name}/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "@supabase/supabase-js";
import { z } from "zod";
import { corsHeaders } from "../_shared/cors.ts";
import { authenticate } from "../_shared/auth.ts";
import { getConfig } from "../_shared/config.ts";
import { emitEvent } from "../_shared/events.ts";
import { InputSchema, OutputSchema } from "./schema.ts";

serve(async (req) => {
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    // 1. Auth
    const { user, supabase } = await authenticate(req, {
      requirePermission: "pos:transaction:create",
    });

    // 2. Parse & validate input
    const body = await req.json();
    const input = InputSchema.parse(body);

    // 3. Load config
    const config = await getConfig(supabase, [
      "pos.require_customer",
      "tax.default_rate",
      "loyalty.enabled",
    ]);

    // 4. Business logic (in transaction)
    const result = await supabase.rpc("pos_create_transaction", {
      p_payload: input,
      p_user_id: user.id,
    });

    if (result.error) throw result.error;

    // 5. Emit events
    await emitEvent(supabase, "sales.transaction.created", {
      transaction_id: result.data.id,
      branch_id: result.data.branch_id,
    });

    // 6. Return
    const output = OutputSchema.parse(result.data);
    return new Response(JSON.stringify({ success: true, data: output }), {
      headers: { ...corsHeaders, "Content-Type": "application/json" },
      status: 200,
    });
  } catch (err) {
    return handleError(err);
  }
});
```

### 2.5 Konvensi API Response

**Success**:
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

**Error**:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input validation failed",
    "details": [
      { "field": "amount", "message": "Must be positive" }
    ],
    "trace_id": "abc123"
  }
}
```

**Error Codes**:
| Code | HTTP | Arti |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Input invalid |
| `UNAUTHORIZED` | 401 | Belum login |
| `FORBIDDEN` | 403 | Tidak punya permission |
| `NOT_FOUND` | 404 | Resource tidak ada |
| `CONFLICT` | 409 | Duplikasi / unique violation |
| `IDEMPOTENCY_REPLAY` | 200 | Request sama sudah diproses |
| `BUSINESS_RULE_VIOLATION` | 422 | Aturan bisnis dilanggar |
| `STOCK_INSUFFICIENT` | 422 | Stok tidak cukup |
| `SCHEDULE_CONFLICT` | 422 | Jadwal bentrok |
| `RATE_LIMIT` | 429 | Terlalu banyak request |
| `INTERNAL_ERROR` | 500 | Error server |

---

# 📄 PETORA-BLUEPRINT-003
## Database Schema & Design

### 3.1 Prinsip Database

1. **UUID v7** sebagai primary key semua tabel
2. **Soft delete** via `deleted_at`
3. **Audit columns** wajib di semua tabel bisnis
4. **RLS** wajib di semua tabel (kecuali `system_config` global)
5. **Index** pada kolom yang sering di-filter/join
6. **Enum** disimpan sebagai `text` + `CHECK` constraint (bukan Postgres enum) agar mudah dimigrate
7. **Money** disimpan sebagai `bigint` (dalam satuan terkecil, Rupiah = satuan) untuk presisi
8. **JSONB** hanya untuk data yang benar-benar dinamis (metadata, custom fields)
9. **Foreign key** wajib untuk relasi bisnis
10. **Trigger** untuk audit log, stock movement, auto-update `updated_at`

### 3.2 Extensions PostgreSQL

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";    -- gen_random_uuid
CREATE EXTENSION IF NOT EXISTS "pg_trgm";     -- trigram search
CREATE EXTENSION IF NOT EXISTS "pg_cron";     -- scheduled jobs
CREATE EXTENSION IF NOT EXISTS "pgjwt";       -- JWT handling
CREATE EXTENSION IF NOT EXISTS "moddatetime"; -- auto updated_at
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";   -- UUID generation
```

### 3.3 Schema Organisasi (Core)

#### 3.3.1 `organizations` — Data perusahaan
```sql
CREATE TABLE organizations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  legal_name text,
  logo_url text,
  address text,
  city text,
  province text,
  postal_code text,
  country text DEFAULT 'ID',
  phone text,
  email text,
  website text,
  tax_number text,                 -- NPWP
  business_type text NOT NULL CHECK (business_type IN ('petshop','petcare','clinic','mixed')),
  default_currency text DEFAULT 'IDR',
  default_timezone text DEFAULT 'Asia/Jakarta',
  default_language text DEFAULT 'id',
  fiscal_year_start int DEFAULT 1, -- bulan (1=Jan)
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);
```

#### 3.3.2 `branches` — Data cabang
```sql
CREATE TABLE branches (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  code text NOT NULL,              -- unik per org, contoh: "JKT01"
  name text NOT NULL,
  address text,
  city text,
  phone text,
  email text,
  timezone text DEFAULT 'Asia/Jakarta',
  latitude numeric(10,7),
  longitude numeric(10,7),
  operating_hours jsonb,           -- {"mon":{"open":"08:00","close":"21:00"},...}
  is_active boolean DEFAULT true,
  is_default boolean DEFAULT false,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, code)
);
```

#### 3.3.3 `users` — Akun login
```sql
CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  employee_id uuid REFERENCES employees(id),   -- nullable untuk customer
  customer_id uuid REFERENCES customers(id),   -- nullable untuk staff
  email text UNIQUE,
  phone text,
  full_name text NOT NULL,
  avatar_url text,
  password_hash text,              -- dari Supabase Auth
  mfa_enabled boolean DEFAULT false,
  mfa_secret text,                 -- encrypted
  is_active boolean DEFAULT true,
  last_login_at timestamptz,
  email_verified_at timestamptz,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz
);
```

#### 3.3.4 `roles` & `permissions`
```sql
CREATE TABLE roles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  code text NOT NULL,              -- super_admin, branch_manager, cashier, ...
  name text NOT NULL,
  description text,
  is_system boolean DEFAULT false, -- system role tidak bisa dihapus
  created_at timestamptz DEFAULT now(),
  UNIQUE (organization_id, code)
);

CREATE TABLE permissions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  code text UNIQUE NOT NULL,       -- contoh: pos:transaction:create
  name text NOT NULL,
  module text NOT NULL,            -- pos, inventory, vet, ...
  resource text NOT NULL,          -- transaction, product, ...
  action text NOT NULL             -- create, read, update, delete, approve
);

CREATE TABLE role_permissions (
  role_id uuid NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  permission_id uuid NOT NULL REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
  user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role_id uuid NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
  branch_id uuid REFERENCES branches(id), -- NULL = semua cabang
  PRIMARY KEY (user_id, role_id, branch_id)
);
```

#### 3.3.5 `employees` — Data karyawan
```sql
CREATE TABLE employees (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  employee_number text NOT NULL,
  full_name text NOT NULL,
  nick_name text,
  id_number text,                  -- KTP (encrypted)
  birth_place text,
  birth_date date,
  gender text CHECK (gender IN ('male','female','other')),
  religion text,
  marital_status text,
  address text,
  phone text,
  email text,
  emergency_contact jsonb,
  photo_url text,
  position_id uuid REFERENCES positions(id),
  department_id uuid REFERENCES departments(id),
  branch_id uuid REFERENCES branches(id), -- primary branch
  employment_type text CHECK (employment_type IN ('full_time','part_time','contract','intern')),
  join_date date NOT NULL,
  end_date date,
  status text DEFAULT 'active' CHECK (status IN ('active','on_leave','resigned','terminated')),
  bank_account jsonb,              -- {bank, account_number, account_name}
  base_salary bigint,              -- dalam Rupiah
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, employee_number)
);
```

### 3.4 Schema Pelanggan & Hewan

#### 3.4.1 `customers`
```sql
CREATE TABLE customers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  customer_number text NOT NULL,   -- auto-generated
  user_id uuid REFERENCES users(id), -- linked account, nullable
  full_name text NOT NULL,
  nick_name text,
  gender text CHECK (gender IN ('male','female','other')),
  birth_date date,
  phone text NOT NULL,             -- encrypted
  phone_normalized text NOT NULL,  -- +62xxx, untuk lookup
  email text,                      -- encrypted
  address text,
  city text,
  photo_url text,
  membership_id uuid REFERENCES memberships(id),
  membership_tier text,            -- bronze/silver/gold/platinum
  membership_expires_at date,
  loyalty_points bigint DEFAULT 0,
  total_spent bigint DEFAULT 0,
  total_visits int DEFAULT 0,
  last_visit_at timestamptz,
  notes text,
  is_blacklisted boolean DEFAULT false,
  blacklist_reason text,
  preferred_contact_method text DEFAULT 'whatsapp',
  marketing_consent boolean DEFAULT false,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, customer_number),
  UNIQUE (organization_id, phone_normalized)
);
CREATE INDEX idx_customers_phone ON customers USING gin(phone_normalized gin_trgm_ops);
CREATE INDEX idx_customers_name ON customers USING gin(full_name gin_trgm_ops);
```

#### 3.4.2 `pets`
```sql
CREATE TABLE pets (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  pet_code text NOT NULL,
  name text NOT NULL,
  species text NOT NULL,           -- dog, cat, bird, rabbit, fish, reptile, other
  breed text,
  gender text CHECK (gender IN ('male','female','unknown')),
  birth_date date,
  age_text text,                   -- jika birth_date tidak pasti
  color text,
  markings text,                   -- ciri khusus
  weight_kg numeric(5,2),
  microchip_number text,
  is_sterilized boolean DEFAULT false,
  photo_url text,
  notes text,
  allergies jsonb,                 -- [{type:'food'|'medication', name:'...'}]
  chronic_conditions jsonb,        -- [{condition:'...', since:'...', notes:''}]
  behavior_notes text,             -- agresif, takut, dll
  is_deceased boolean DEFAULT false,
  deceased_at timestamptz,
  deceased_reason text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, pet_code)
);

CREATE TABLE pet_owners (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  pet_id uuid NOT NULL REFERENCES pets(id) ON DELETE CASCADE,
  customer_id uuid NOT NULL REFERENCES customers(id),
  relationship text DEFAULT 'owner' CHECK (relationship IN ('owner','co_owner','caretaker')),
  is_primary boolean DEFAULT false,
  notes text,
  created_at timestamptz DEFAULT now(),
  UNIQUE (pet_id, customer_id)
);
CREATE INDEX idx_pet_owners_primary ON pet_owners(pet_id) WHERE is_primary = true;
```

#### 3.4.3 `pet_vaccinations`
```sql
CREATE TABLE pet_vaccinations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  pet_id uuid NOT NULL REFERENCES pets(id),
  vaccine_name text NOT NULL,
  batch_number text,
  manufacturer text,
  vaccination_date date NOT NULL,
  next_due_date date,
  veterinarian_id uuid REFERENCES employees(id),
  clinic_name text,
  certificate_url text,
  notes text,
  created_at timestamptz DEFAULT now()
);
```

### 3.5 Schema Produk & Inventory

#### 3.5.1 `products`
```sql
CREATE TABLE products (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  sku text NOT NULL,
  barcode text,                    -- EAN-13 / Code-128
  name text NOT NULL,
  short_name text,
  description text,
  category_id uuid REFERENCES product_categories(id),
  brand_id uuid REFERENCES brands(id),
  product_type text NOT NULL CHECK (product_type IN ('single','bundle','service','consignment')),
  unit_id uuid NOT NULL REFERENCES units(id), -- base unit
  purchase_unit_id uuid REFERENCES units(id), -- satuan beli (beda dari jual)
  purchase_conversion numeric(10,4) DEFAULT 1, -- 1 karton = 12 pcs
  cost_price bigint,               -- HPP (moving average)
  base_price bigint NOT NULL,      -- harga jual default
  wholesale_price bigint,          -- harga grosir
  wholesale_min_qty int,
  minimum_price bigint,            -- batas bawah diskon
  weight_gram int,
  length_mm int,
  width_mm int,
  height_mm int,
  has_batch boolean DEFAULT false,
  has_expiry boolean DEFAULT false,
  is_active boolean DEFAULT true,
  is_sellable boolean DEFAULT true,
  is_purchasable boolean DEFAULT true,
  requires_prescription boolean DEFAULT false, -- obat keras
  image_url text,
  metadata jsonb DEFAULT '{}',
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, sku),
  UNIQUE (organization_id, barcode) WHERE barcode IS NOT NULL
);
```

#### 3.5.2 `product_prices` — Harga per cabang
```sql
CREATE TABLE product_prices (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id uuid NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  branch_id uuid NOT NULL REFERENCES branches(id),
  price bigint NOT NULL,
  wholesale_price bigint,
  effective_from date NOT NULL DEFAULT CURRENT_DATE,
  effective_to date,
  created_at timestamptz DEFAULT now(),
  UNIQUE (product_id, branch_id, effective_from)
);
```

#### 3.5.3 `product_bundles`
```sql
CREATE TABLE product_bundles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  bundle_product_id uuid NOT NULL REFERENCES products(id),
  component_product_id uuid NOT NULL REFERENCES products(id),
  quantity numeric(10,4) NOT NULL DEFAULT 1,
  is_optional boolean DEFAULT false,
  created_at timestamptz DEFAULT now(),
  UNIQUE (bundle_product_id, component_product_id)
);
```

#### 3.5.4 `warehouses`
```sql
CREATE TABLE warehouses (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id),
  branch_id uuid NOT NULL REFERENCES branches(id),
  code text NOT NULL,
  name text NOT NULL,
  warehouse_type text DEFAULT 'main' CHECK (warehouse_type IN ('main','transit','quarantine','damaged','consignment')),
  address text,
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, code)
);
```

#### 3.5.5 `stock_items` — Stok per produk × gudang × batch
```sql
CREATE TABLE stock_items (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id uuid NOT NULL REFERENCES products(id),
  warehouse_id uuid NOT NULL REFERENCES warehouses(id),
  batch_number text,
  expiry_date date,
  quantity numeric(15,4) NOT NULL DEFAULT 0,
  reserved_quantity numeric(15,4) NOT NULL DEFAULT 0, -- untuk booking/PO
  cost_price bigint,                -- HPP per batch
  last_movement_at timestamptz,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (product_id, warehouse_id, batch_number)
);
CREATE INDEX idx_stock_expiry ON stock_items(expiry_date) WHERE expiry_date IS NOT NULL;
```

#### 3.5.6 `stock_movements` — Kardex / kartu stok
```sql
CREATE TABLE stock_movements (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  product_id uuid NOT NULL REFERENCES products(id),
  warehouse_id uuid NOT NULL REFERENCES warehouses(id),
  batch_number text,
  movement_type text NOT NULL CHECK (movement_type IN (
    'purchase_in','sales_out','adjustment_in','adjustment_out',
    'transfer_in','transfer_out','return_in','return_out',
    'damage_out','loss_out','production_in','consumption_out',
    'opname_adjust','consignment_in','consignment_out'
  )),
  quantity numeric(15,4) NOT NULL, -- positif = masuk, negatif = keluar
  unit_cost bigint,                -- HPP saat movement
  reference_type text,             -- 'purchase_order','sales_transaction',...
  reference_id uuid,               -- ID dokumen referensi
  reference_number text,
  notes text,
  performed_by uuid NOT NULL REFERENCES users(id),
  performed_at timestamptz DEFAULT now(),
  created_at timestamptz DEFAULT now()
);
CREATE INDEX idx_stock_movements_product ON stock_movements(product_id, performed_at DESC);
CREATE INDEX idx_stock_movements_ref ON stock_movements(reference_type, reference_id);
```

### 3.6 Schema Penjualan (POS)

#### 3.6.1 `sales_transactions`
```sql
CREATE TABLE sales_transactions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  transaction_number text NOT NULL, -- TRX/JKT01/2026/11/000001
  idempotency_key text UNIQUE,
  transaction_type text DEFAULT 'sales' CHECK (transaction_type IN ('sales','refund','void')),
  original_transaction_id uuid REFERENCES sales_transactions(id), -- untuk refund/void
  customer_id uuid REFERENCES customers(id),
  pet_id uuid REFERENCES pets(id),
  cashier_id uuid NOT NULL REFERENCES users(id),
  shift_id uuid REFERENCES cashier_shifts(id),
  transaction_date date NOT NULL DEFAULT CURRENT_DATE,
  transaction_time timestamptz NOT NULL DEFAULT now(),
  
  -- Subtotal
  subtotal bigint NOT NULL,
  discount_item bigint DEFAULT 0,
  discount_cart bigint DEFAULT 0,
  discount_voucher bigint DEFAULT 0,
  total_discount bigint DEFAULT 0,
  taxable_amount bigint NOT NULL,
  tax_amount bigint DEFAULT 0,
  tax_rate numeric(5,2) DEFAULT 0,
  service_fee bigint DEFAULT 0,
  rounding bigint DEFAULT 0,
  grand_total bigint NOT NULL,
  
  -- Payment
  paid_amount bigint NOT NULL,
  change_amount bigint DEFAULT 0,
  payment_status text DEFAULT 'paid' CHECK (payment_status IN ('paid','partial','unpaid','refunded')),
  
  -- Loyalty
  loyalty_points_earned int DEFAULT 0,
  loyalty_points_redeemed int DEFAULT 0,
  loyalty_value bigint DEFAULT 0,
  
  -- Voucher
  voucher_id uuid REFERENCES vouchers(id),
  
  -- Status
  status text DEFAULT 'completed' CHECK (status IN ('draft','completed','voided','refunded','pending')),
  notes text,
  
  -- Metadata
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (branch_id, transaction_number)
);
CREATE INDEX idx_sales_date ON sales_transactions(branch_id, transaction_date DESC);
CREATE INDEX idx_sales_customer ON sales_transactions(customer_id);
```

#### 3.6.2 `sales_transaction_items`
```sql
CREATE TABLE sales_transaction_items (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id uuid NOT NULL REFERENCES sales_transactions(id) ON DELETE CASCADE,
  product_id uuid NOT NULL REFERENCES products(id),
  item_type text DEFAULT 'product' CHECK (item_type IN ('product','service','bundle')),
  description text NOT NULL,
  quantity numeric(15,4) NOT NULL,
  unit_id uuid NOT NULL REFERENCES units(id),
  unit_price bigint NOT NULL,
  discount_amount bigint DEFAULT 0,
  tax_rate numeric(5,2) DEFAULT 0,
  tax_amount bigint DEFAULT 0,
  subtotal bigint NOT NULL,
  total bigint NOT NULL,
  batch_number text,
  warehouse_id uuid REFERENCES warehouses(id),
  staff_commission_id uuid REFERENCES commission_rules(id),
  notes text,
  sort_order int DEFAULT 0,
  metadata jsonb DEFAULT '{}'
);
```

#### 3.6.3 `sales_payments`
```sql
CREATE TABLE sales_payments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id uuid NOT NULL REFERENCES sales_transactions(id) ON DELETE CASCADE,
  payment_method text NOT NULL CHECK (payment_method IN (
    'cash','debit_card','credit_card','qris','bank_transfer','e_wallet','voucher','loyalty_point','credit_account','other'
  )),
  payment_provider text,           -- 'midtrans','xendit',...
  provider_reference text,         -- ID dari payment gateway
  amount bigint NOT NULL,
  reference_number text,           -- nomor referensi manual
  card_number_last4 text,
  status text DEFAULT 'success' CHECK (status IN ('pending','success','failed','refunded')),
  paid_at timestamptz DEFAULT now(),
  metadata jsonb DEFAULT '{}'
);
```

### 3.7 Schema Pembelian

#### 3.7.1 `suppliers`
```sql
CREATE TABLE suppliers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  code text NOT NULL,
  name text NOT NULL,
  contact_person text,
  phone text,
  email text,
  address text,
  city text,
  tax_number text,
  bank_account jsonb,
  payment_terms int DEFAULT 0,     -- hari (0=COD)
  currency text DEFAULT 'IDR',
  rating numeric(2,1),
  is_active boolean DEFAULT true,
  notes text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  deleted_at timestamptz,
  UNIQUE (organization_id, code)
);
```

#### 3.7.2 `purchase_orders`
```sql
CREATE TABLE purchase_orders (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  po_number text NOT NULL,
  supplier_id uuid NOT NULL REFERENCES suppliers(id),
  order_date date NOT NULL,
  expected_date date,
  warehouse_id uuid NOT NULL REFERENCES warehouses(id),
  subtotal bigint NOT NULL,
  discount_amount bigint DEFAULT 0,
  tax_amount bigint DEFAULT 0,
  shipping_cost bigint DEFAULT 0,
  grand_total bigint NOT NULL,
  paid_amount bigint DEFAULT 0,
  remaining_amount bigint NOT NULL,
  status text DEFAULT 'draft' CHECK (status IN ('draft','pending_approval','approved','ordered','partial_received','received','closed','cancelled')),
  approval_status text DEFAULT 'pending' CHECK (approval_status IN ('pending','approved','rejected')),
  approved_by uuid REFERENCES users(id),
  approved_at timestamptz,
  notes text,
  created_by uuid NOT NULL REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, po_number)
);
```

### 3.8 Schema Pelayanan (Grooming, Booking, Hotel)

#### 3.8.1 `service_categories` & `services`
```sql
CREATE TABLE service_categories (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  name text NOT NULL,
  code text NOT NULL,
  icon text,
  sort_order int DEFAULT 0,
  is_active boolean DEFAULT true,
  UNIQUE (organization_id, code)
);

CREATE TABLE services (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  category_id uuid NOT NULL REFERENCES service_categories(id),
  code text NOT NULL,
  name text NOT NULL,
  description text,
  service_type text NOT NULL CHECK (service_type IN ('grooming','boarding','daycare','consultation','medical','transport','home_service','other')),
  base_duration_minutes int NOT NULL,
  buffer_minutes int DEFAULT 0,
  base_price bigint NOT NULL,
  requires_pet boolean DEFAULT true,
  requires_staff boolean DEFAULT true,
  requires_room boolean DEFAULT false,
  max_concurrent int DEFAULT 1,
  is_active boolean DEFAULT true,
  image_url text,
  UNIQUE (organization_id, code)
);

CREATE TABLE service_prices (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  service_id uuid NOT NULL REFERENCES services(id) ON DELETE CASCADE,
  branch_id uuid NOT NULL REFERENCES branches(id),
  species text,                    -- dog/cat/...
  size text,                       -- small/medium/large/xlarge
  price bigint NOT NULL,
  duration_minutes int,
  UNIQUE (service_id, branch_id, species, size)
);
```

#### 3.8.2 `bookings`
```sql
CREATE TABLE bookings (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  booking_number text NOT NULL,
  customer_id uuid NOT NULL REFERENCES customers(id),
  pet_id uuid NOT NULL REFERENCES pets(id),
  service_id uuid NOT NULL REFERENCES services(id),
  staff_id uuid REFERENCES employees(id),
  room_id uuid REFERENCES rooms(id),
  booking_date date NOT NULL,
  start_time timestamptz NOT NULL,
  end_time timestamptz NOT NULL,
  duration_minutes int NOT NULL,
  estimated_price bigint,
  final_price bigint,
  deposit_amount bigint DEFAULT 0,
  status text DEFAULT 'pending' CHECK (status IN (
    'pending','confirmed','checked_in','in_progress','completed','cancelled','no_show','rescheduled'
  )),
  source text DEFAULT 'staff' CHECK (source IN ('staff','customer_portal','walk_in','phone','whatsapp')),
  notes text,
  customer_notes text,
  cancellation_reason text,
  cancelled_at timestamptz,
  cancelled_by uuid REFERENCES users(id),
  created_by uuid NOT NULL REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, booking_number)
);
CREATE INDEX idx_bookings_date ON bookings(branch_id, booking_date);
CREATE INDEX idx_bookings_staff ON bookings(staff_id, start_time, end_time);
CREATE INDEX idx_bookings_room ON bookings(room_id, booking_date);
```

#### 3.8.3 `rooms` (untuk pet hotel)
```sql
CREATE TABLE rooms (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  code text NOT NULL,
  name text NOT NULL,
  room_type text NOT NULL CHECK (room_type IN ('standard','deluxe','vip','cat_room','isolasi')),
  capacity int DEFAULT 1,
  size_category text CHECK (size_category IN ('small','medium','large','xlarge','any')),
  daily_rate bigint NOT NULL,
  facilities jsonb DEFAULT '[]',
  status text DEFAULT 'available' CHECK (status IN ('available','occupied','maintenance','cleaning')),
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, code)
);

CREATE TABLE boarding_stays (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  stay_number text NOT NULL,
  booking_id uuid REFERENCES bookings(id),
  customer_id uuid NOT NULL REFERENCES customers(id),
  pet_id uuid NOT NULL REFERENCES pets(id),
  room_id uuid NOT NULL REFERENCES rooms(id),
  check_in_at timestamptz NOT NULL,
  check_out_at timestamptz,
  expected_check_out timestamptz NOT NULL,
  daily_rate bigint NOT NULL,
  total_days int NOT NULL,
  subtotal bigint NOT NULL,
  additional_charges bigint DEFAULT 0,
  discount_amount bigint DEFAULT 0,
  grand_total bigint NOT NULL,
  paid_amount bigint DEFAULT 0,
  status text DEFAULT 'active' CHECK (status IN ('reserved','active','checked_out','cancelled')),
  feeding_schedule jsonb,          -- [{time:'08:00', food:'...', amount:'...'}]
  medication_schedule jsonb,
  special_instructions text,
  weight_log jsonb DEFAULT '[]',   -- [{date, weight_kg}]
  notes text,
  created_by uuid NOT NULL REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, stay_number)
);
```

### 3.9 Schema Klinik & Farmasi

#### 3.9.1 `medical_records`
```sql
CREATE TABLE medical_records (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid NOT NULL,
  record_number text NOT NULL,
  pet_id uuid NOT NULL REFERENCES pets(id),
  customer_id uuid NOT NULL REFERENCES customers(id),
  veterinarian_id uuid NOT NULL REFERENCES employees(id),
  visit_date timestamptz NOT NULL,
  visit_type text NOT NULL CHECK (visit_type IN ('consultation','vaccination','checkup','emergency','surgery','follow_up','grooming_medical')),
  appointment_id uuid REFERENCES bookings(id),
  
  -- Anamnesa
  chief_complaint text,
  history_of_present_illness text,
  previous_treatment text,
  
  -- Pemeriksaan fisik
  vital_signs jsonb,               -- {temp_c, heart_rate, resp_rate, bcs, mucous_color, crt}
  physical_exam jsonb,             -- {eyes, ears, nose, mouth, skin, coat, ...}
  body_weight_kg numeric(5,2),
  body_temperature_c numeric(4,1),
  
  -- Diagnosis
  diagnosis text,
  icd_code text,                   -- kode diagnosis
  differential_diagnosis text,
  
  -- Tindakan
  treatment_plan text,
  procedures_performed jsonb DEFAULT '[]',
  
  -- Rekomendasi
  recommendations text,
  follow_up_date date,
  next_vaccination_date date,
  
  -- Status
  status text DEFAULT 'draft' CHECK (status IN ('draft','finalized','amended')),
  finalized_at timestamptz,
  finalized_by uuid REFERENCES users(id),
  
  notes text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (branch_id, record_number)
);
```

#### 3.9.2 `prescriptions` & `prescription_items`
```sql
CREATE TABLE prescriptions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  medical_record_id uuid NOT NULL REFERENCES medical_records(id),
  prescription_number text NOT NULL,
  veterinarian_id uuid NOT NULL REFERENCES employees(id),
  pet_id uuid NOT NULL REFERENCES pets(id),
  customer_id uuid NOT NULL REFERENCES customers(id),
  prescription_date date NOT NULL,
  instructions text,
  refills_allowed int DEFAULT 0,
  status text DEFAULT 'active' CHECK (status IN ('active','dispensed','completed','cancelled')),
  created_at timestamptz DEFAULT now()
);

CREATE TABLE prescription_items (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  prescription_id uuid NOT NULL REFERENCES prescriptions(id) ON DELETE CASCADE,
  product_id uuid NOT NULL REFERENCES products(id),
  dosage text NOT NULL,            -- contoh: "1 tablet, 2x sehari"
  frequency text,                  -- "2x daily"
  duration_days int,
  quantity_prescribed numeric(10,4) NOT NULL,
  quantity_dispensed numeric(10,4) DEFAULT 0,
  notes text,
  UNIQUE (prescription_id, product_id)
);
```

### 3.10 Schema Keuangan

#### 3.1.1 `chart_of_accounts`
```sql
CREATE TABLE chart_of_accounts (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  account_code text NOT NULL,      -- contoh: '1-1001'
  account_name text NOT NULL,
  account_type text NOT NULL CHECK (account_type IN ('asset','liability','equity','revenue','expense')),
  parent_id uuid REFERENCES chart_of_accounts(id),
  level int NOT NULL,              -- 1=header, 2=group, 3=detail
  is_postable boolean DEFAULT true, -- bisa di-journal?
  is_cash_account boolean DEFAULT false,
  is_bank_account boolean DEFAULT false,
  normal_balance text CHECK (normal_balance IN ('debit','credit')),
  description text,
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  UNIQUE (organization_id, account_code)
);
```

#### 3.10.2 `journal_entries`
```sql
CREATE TABLE journal_entries (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid,
  journal_number text NOT NULL,
  journal_date date NOT NULL,
  period_id uuid REFERENCES accounting_periods(id),
  reference_type text,             -- 'sales','purchase','adjustment',...
  reference_id uuid,
  description text NOT NULL,
  total_debit bigint NOT NULL,
  total_credit bigint NOT NULL,
  status text DEFAULT 'posted' CHECK (status IN ('draft','posted','reversed','void')),
  created_by uuid NOT NULL REFERENCES users(id),
  approved_by uuid REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  CHECK (total_debit = total_credit)
);

CREATE TABLE journal_entry_lines (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  journal_id uuid NOT NULL REFERENCES journal_entries(id) ON DELETE CASCADE,
  account_id uuid NOT NULL REFERENCES chart_of_accounts(id),
  description text,
  debit bigint DEFAULT 0,
  credit bigint DEFAULT 0,
  branch_id uuid REFERENCES branches(id),
  CHECK (NOT (debit > 0 AND credit > 0)),
  CHECK (debit > 0 OR credit > 0)
);
```

### 3.11 Schema Konfigurasi

#### 3.11.1 `system_config`
```sql
CREATE TABLE system_config (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  branch_id uuid,                  -- NULL = global org
  config_key text NOT NULL,        -- contoh: 'loyalty.point_ratio'
  config_value jsonb NOT NULL,
  value_type text NOT NULL CHECK (value_type IN ('string','number','boolean','json','encrypted')),
  description text,
  is_secret boolean DEFAULT false,
  is_editable boolean DEFAULT true,
  updated_by uuid REFERENCES users(id),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (organization_id, branch_id, config_key)
);
```

**Contoh default config**:
```
loyalty.enabled                 = true
loyalty.point_ratio             = 1000    (1 poin per Rp1000)
loyalty.point_value             = 100     (1 poin = Rp100)
loyalty.max_redemption_percent  = 50      (max 50% dari total)
tax.default_rate                = 11
tax.enabled                     = true
pos.require_customer            = false
pos.allow_negative_stock        = false
pos.max_discount_percent        = 50
pos.refund_max_days             = 30
pos.receipt_footer              = "Terima kasih"
inventory.valuation_method      = moving_average
inventory.negative_stock_alert  = true
notification.whatsapp_enabled   = true
notification.booking_reminder_h = 24
membership.tiers                = [{"code":"bronze","min_spent":0,"discount":0}, ...]
```

#### 3.11.2 `integrations`
```sql
CREATE TABLE integrations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  code text NOT NULL,              -- 'whatsapp_fonnte','email_resend','payment_midtrans',...
  name text NOT NULL,
  provider text NOT NULL,
  is_active boolean DEFAULT false,
  config jsonb NOT NULL,           -- encrypted credentials
  webhook_url text,
  last_tested_at timestamptz,
  last_test_status text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE (organization_id, code)
);
```

### 3.12 Schema Audit & Event

#### 3.12.1 `audit_logs`
```sql
CREATE TABLE audit_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL,
  user_id uuid,                    -- null jika system
  action text NOT NULL,            -- 'INSERT','UPDATE','DELETE','LOGIN','LOGOUT',...
  table_name text NOT NULL,
  record_id uuid,
  old_values jsonb,
  new_values jsonb,
  ip_address inet,
  user_agent text,
  branch_id uuid,
  created_at timestamptz DEFAULT now()
);
CREATE INDEX idx_audit_user ON audit_logs(user_id, created_at DESC);
CREATE INDEX idx_audit_record ON audit_logs(table_name, record_id);
```

#### 3.12.2 `idempotency_keys`
```sql
CREATE TABLE idempotency_keys (
  key text PRIMARY KEY,
  user_id uuid NOT NULL,
  response_status int NOT NULL,
  response_body jsonb NOT NULL,
  created_at timestamptz DEFAULT now(),
  expires_at timestamptz NOT NULL
);
```

### 3.13 Row Level Security (RLS)

**Policy pattern**:
```sql
-- Contoh untuk sales_transactions
ALTER TABLE sales_transactions ENABLE ROW LEVEL SECURITY;

-- User bisa baca transaksi di cabang yang di-assign
CREATE POLICY "sales_transactions_read" ON sales_transactions
  FOR SELECT
  USING (
    organization_id = current_setting('app.organization_id')::uuid
    AND (
      EXISTS (
        SELECT 1 FROM user_roles ur
        WHERE ur.user_id = auth.uid()
        AND ur.branch_id = sales_transactions.branch_id
      )
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        WHERE ur.user_id = auth.uid()
        AND ur.branch_id IS NULL  -- akses semua cabang
      )
    )
  );

-- User hanya bisa buat transaksi di cabang yang di-assign
CREATE POLICY "sales_transactions_insert" ON sales_transactions
  FOR INSERT
  WITH CHECK (
    organization_id = current_setting('app.organization_id')::uuid
    AND branch_id IN (
      SELECT branch_id FROM user_roles
      WHERE user_id = auth.uid() AND (branch_id IS NULL OR branch_id = sales_transactions.branch_id)
    )
    AND EXISTS (
      SELECT 1 FROM role_permissions rp
      JOIN user_roles ur ON ur.role_id = rp.role_id
      WHERE ur.user_id = auth.uid()
      AND rp.permission_id IN (SELECT id FROM permissions WHERE code = 'pos:transaction:create')
    )
  );
```

### 3.14 Trigger & Functions

```sql
-- Auto-update updated_at
CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON sales_transactions
  FOR EACH ROW
  EXECUTE FUNCTION moddatetime(updated_at);

-- Audit log trigger
CREATE OR REPLACE FUNCTION audit_log_trigger()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_logs (organization_id, user_id, action, table_name, record_id, old_values, new_values, branch_id)
  VALUES (
    COALESCE(NEW.organization_id, OLD.organization_id),
    auth.uid(),
    TG_OP,
    TG_TABLE_NAME,
    COALESCE(NEW.id, OLD.id),
    to_jsonb(OLD),
    to_jsonb(NEW),
    COALESCE(NEW.branch_id, OLD.branch_id)
  );
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Attach ke semua tabel bisnis
CREATE TRIGGER audit_sales_transactions
  AFTER INSERT OR UPDATE OR DELETE ON sales_transactions
  FOR EACH ROW EXECUTE FUNCTION audit_log_trigger();

-- Auto stock movement on sales
CREATE OR REPLACE FUNCTION fn_sales_reduce_stock()
RETURNS TRIGGER AS $$
BEGIN
  -- Untuk setiap item, kurangi stok di warehouse terkait
  -- (dipanggil setelah INSERT sales_transaction_items via Edge Function)
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### 3.15 Views untuk Reporting

```sql
CREATE VIEW v_daily_sales_summary AS
SELECT
  st.branch_id,
  st.transaction_date,
  COUNT(*) AS transaction_count,
  SUM(st.grand_total) AS total_revenue,
  SUM(st.total_discount) AS total_discount,
  SUM(st.tax_amount) AS total_tax,
  SUM(CASE WHEN st.payment_status='paid' THEN st.grand_total ELSE 0 END) AS paid_revenue
FROM sales_transactions st
WHERE st.status = 'completed'
GROUP BY st.branch_id, st.transaction_date;

CREATE VIEW v_stock_status AS
SELECT
  p.id AS product_id,
  p.sku,
  p.name,
  w.branch_id,
  w.id AS warehouse_id,
  SUM(si.quantity) AS total_quantity,
  SUM(si.reserved_quantity) AS reserved,
  SUM(si.quantity) - SUM(si.reserved_quantity) AS available,
  MIN(si.expiry_date) AS nearest_expiry,
  p.minimum_stock,
  CASE WHEN SUM(si.quantity) < p.minimum_stock THEN true ELSE false END AS is_low_stock
FROM products p
JOIN stock_items si ON si.product_id = p.id
JOIN warehouses w ON w.id = si.warehouse_id
WHERE p.deleted_at IS NULL
GROUP BY p.id, w.branch_id, w.id;
```

---

# 📄 PETORA-BLUEPRINT-004
## Backend Architecture (Edge Functions)

### 4.1 Daftar Edge Functions

| ID Function | Method | Deskripsi |
|---|---|---|
| `pos-create-transaction` | POST | Buat transaksi POS |
| `pos-void-transaction` | POST | Void transaksi |
| `pos-refund-transaction` | POST | Refund transaksi |
| `pos-close-shift` | POST | Tutup shift kasir |
| `inventory-adjust-stock` | POST | Adjustment stok |
| `inventory-transfer` | POST | Transfer antar gudang |
| `inventory-stock-opname` | POST | Submit hasil opname |
| `purchase-create-order` | POST | Buat purchase order |
| `purchase-receive` | POST | Terima barang |
| `booking-create` | POST | Buat booking |
| `booking-confirm` | POST | Konfirmasi booking |
| `booking-checkin` | POST | Check-in booking |
| `booking-complete` | POST | Selesaikan booking |
| `grooming-start` | POST | Mulai grooming |
| `grooming-complete` | POST | Selesai grooming |
| `boarding-checkin` | POST | Check-in pet hotel |
| `boarding-checkout` | POST | Check-out pet hotel |
| `boarding-daily-log` | POST | Log harian penitipan |
| `vet-create-record` | POST | Buat rekam medis |
| `vet-finalize-record` | POST | Finalisasi rekam medis |
| `vet-amend-record` | POST | Revisi rekam medis |
| `pharmacy-dispense` | POST | Keluarkan obat |
| `payroll-calculate` | POST | Hitung gaji periode |
| `payroll-approve` | POST | Approve payroll |
| `finance-record-journal` | POST | Catat jurnal manual |
| `notification-send` | POST | Kirim notifikasi |
| `integration-test` | POST | Test koneksi integrasi |
| `report-generate` | POST | Generate laporan |
| `backup-create` | POST | Trigger backup |

### 4.2 Shared Modules (supabase/functions/_shared/)

```
_shared/
├── auth.ts              # authenticate(), requirePermission()
├── config.ts            # getConfig(), setConfig()
├── database.ts          # supabase admin client factory
├── events.ts            # emitEvent(), subscribeEvent()
├── errors.ts            # AppError, handleError()
├── validators.ts        # Common Zod schemas
├── formatters.ts        # Format currency, date, phone
├── idempotency.ts       # checkIdempotency(), storeIdempotency()
├── document-number.ts   # generateDocumentNumber()
├── stock.ts             # reserveStock(), commitStock(), releaseStock()
├── accounting.ts        # postJournal()
├── notification.ts      # sendWhatsApp(), sendEmail()
├── encryption.ts        # encrypt(), decrypt() untuk PII
└── cors.ts              # CORS headers
```

### 4.3 Contoh Implementasi: `pos-create-transaction`

```typescript
// supabase/functions/pos-create-transaction/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { authenticate } from "../_shared/auth.ts";
import { getConfig } from "../_shared/config.ts";
import { checkIdempotency, storeIdempotency } from "../_shared/idempotency.ts";
import { generateDocumentNumber } from "../_shared/document-number.ts";
import { reserveStock, commitStock, releaseStock } from "../_shared/stock.ts";
import { postJournal } from "../_shared/accounting.ts";
import { emitEvent } from "../_shared/events.ts";
import { CreateTransactionSchema } from "./schema.ts";
import { calculateTotals } from "./calculations.ts";

serve(async (req) => {
  try {
    // 1. Auth + permission
    const { user, supabase, session } = await authenticate(req, {
      permission: "pos:transaction:create",
    });

    // 2. Idempotency check
    const idempotencyKey = req.headers.get("Idempotency-Key");
    if (!idempotencyKey) {
      throw new AppError("VALIDATION_ERROR", "Idempotency-Key header required");
    }
    const existing = await checkIdempotency(supabase, idempotencyKey);
    if (existing) {
      return new Response(JSON.stringify(existing.response_body), {
        status: existing.response_status,
        headers: { "Idempotent-Replayed": "true" },
      });
    }

    // 3. Parse & validate input
    const body = await req.json();
    const input = CreateTransactionSchema.parse(body);

    // 4. Load config
    const config = await getConfig(supabase, [
      "pos.require_customer",
      "pos.allow_negative_stock",
      "tax.enabled",
      "tax.default_rate",
      "loyalty.enabled",
      "loyalty.point_ratio",
    ]);

    // 5. Business rules validation
    if (config["pos.require_customer"] && !input.customer_id) {
      throw new AppError("BUSINESS_RULE_VIOLATION", "Customer is required");
    }

    // 6. Calculate totals
    const totals = await calculateTotals(supabase, input, config);

    // 7. Reserve stock (check availability)
    const stockReservation = await reserveStock(supabase, {
      branch_id: input.branch_id,
      items: input.items,
      allow_negative: config["pos.allow_negative_stock"],
    });

    // 8. Begin DB transaction
    const { data: transaction, error } = await supabase.rpc("pos_create_transaction_atomic", {
      p_user_id: user.id,
      p_branch_id: input.branch_id,
      p_customer_id: input.customer_id,
      p_pet_id: input.pet_id,
      p_items: input.items,
      p_payments: input.payments,
      p_discounts: input.discounts,
      p_voucher_id: input.voucher_id,
      p_loyalty_redeem: input.loyalty_points_redeemed,
      p_notes: input.notes,
      p_totals: totals,
    });

    if (error) {
      await releaseStock(supabase, stockReservation.id);
      throw error;
    }

    // 9. Commit stock
    await commitStock(supabase, stockReservation.id, transaction.id);

    // 10. Post accounting journal
    await postJournal(supabase, {
      type: "sales",
      reference_id: transaction.id,
      branch_id: input.branch_id,
      lines: buildSalesJournalLines(transaction, config),
    });

    // 11. Update loyalty points
    if (config["loyalty.enabled"] && input.customer_id) {
      await supabase.rpc("loyalty_add_points", {
        p_customer_id: input.customer_id,
        p_points: transaction.loyalty_points_earned,
        p_source: "sales",
        p_reference_id: transaction.id,
      });
    }

    // 12. Emit event
    await emitEvent(supabase, "sales.transaction.created", {
      transaction_id: transaction.id,
      branch_id: input.branch_id,
      total: transaction.grand_total,
    });

    // 13. Store idempotency
    const response = { success: true, data: transaction };
    await storeIdempotency(supabase, idempotencyKey, 200, response);

    return new Response(JSON.stringify(response), {
      headers: { "Content-Type": "application/json", "Idempotent-Key": idempotencyKey },
    });
  } catch (err) {
    return handleError(err);
  }
});
```

### 4.4 RPC Function: `pos_create_transaction_atomic`

```sql
CREATE OR REPLACE FUNCTION pos_create_transaction_atomic(
  p_user_id uuid,
  p_branch_id uuid,
  p_customer_id uuid,
  p_pet_id uuid,
  p_items jsonb,
  p_payments jsonb,
  p_discounts jsonb,
  p_voucher_id uuid,
  p_loyalty_redeem int,
  p_notes text,
  p_totals jsonb
) RETURNS jsonb
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_transaction_id uuid;
  v_transaction_number text;
  v_item jsonb;
  v_payment jsonb;
BEGIN
  -- Generate document number
  v_transaction_number := generate_document_number(p_branch_id, 'TRX');
  
  -- Insert transaction
  INSERT INTO sales_transactions (
    organization_id, branch_id, transaction_number, transaction_type,
    customer_id, pet_id, cashier_id, transaction_date,
    subtotal, discount_item, discount_cart, discount_voucher, total_discount,
    taxable_amount, tax_amount, tax_rate, service_fee, grand_total,
    paid_amount, change_amount, payment_status,
    loyalty_points_earned, loyalty_points_redeemed, loyalty_value,
    voucher_id, status, notes
  ) VALUES (
    (SELECT branch_id FROM branches WHERE id = p_branch_id).organization_id,
    p_branch_id, v_transaction_number, 'sales',
    p_customer_id, p_pet_id, p_user_id, CURRENT_DATE,
    (p_totals->>'subtotal')::bigint,
    (p_totals->>'discount_item')::bigint,
    (p_totals->>'discount_cart')::bigint,
    (p_totals->>'discount_voucher')::bigint,
    (p_totals->>'total_discount')::bigint,
    (p_totals->>'taxable_amount')::bigint,
    (p_totals->>'tax_amount')::bigint,
    (p_totals->>'tax_rate')::numeric,
    (p_totals->>'service_fee')::bigint,
    (p_totals->>'grand_total')::bigint,
    (p_totals->>'paid_amount')::bigint,
    (p_totals->>'change_amount')::bigint,
    CASE WHEN (p_totals->>'paid_amount')::bigint >= (p_totals->>'grand_total')::bigint THEN 'paid' ELSE 'partial' END,
    (p_totals->>'loyalty_earned')::int,
    p_loyalty_redeem,
    (p_totals->>'loyalty_value')::bigint,
    p_voucher_id, 'completed', p_notes
  ) RETURNING id INTO v_transaction_id;
  
  -- Insert items
  FOR v_item IN SELECT * FROM jsonb_array_elements(p_items) LOOP
    INSERT INTO sales_transaction_items (
      transaction_id, product_id, item_type, description, quantity, unit_id,
      unit_price, discount_amount, tax_rate, tax_amount, subtotal, total,
      batch_number, warehouse_id, staff_commission_id, notes, sort_order
    ) VALUES (
      v_transaction_id,
      (v_item->>'product_id')::uuid,
      COALESCE(v_item->>'item_type', 'product'),
      v_item->>'description',
      (v_item->>'quantity')::numeric,
      (v_item->>'unit_id')::uuid,
      (v_item->>'unit_price')::bigint,
      COALESCE((v_item->>'discount_amount')::bigint, 0),
      COALESCE((v_item->>'tax_rate')::numeric, 0),
      COALESCE((v_item->>'tax_amount')::bigint, 0),
      (v_item->>'subtotal')::bigint,
      (v_item->>'total')::bigint,
      v_item->>'batch_number',
      (v_item->>'warehouse_id')::uuid,
      (v_item->>'staff_commission_id')::uuid,
      v_item->>'notes',
      COALESCE((v_item->>'sort_order')::int, 0)
    );
  END LOOP;
  
  -- Insert payments
  FOR v_payment IN SELECT * FROM jsonb_array_elements(p_payments) LOOP
    INSERT INTO sales_payments (
      transaction_id, payment_method, payment_provider, provider_reference,
      amount, reference_number, status
    ) VALUES (
      v_transaction_id,
      v_payment->>'payment_method',
      v_payment->>'payment_provider',
      v_payment->>'provider_reference',
      (v_payment->>'amount')::bigint,
      v_payment->>'reference_number',
      COALESCE(v_payment->>'status', 'success')
    );
  END LOOP;
  
  -- Return full transaction
  RETURN (SELECT to_jsonb(st) FROM sales_transactions st WHERE id = v_transaction_id);
END;
$$;
```

### 4.5 Scheduled Jobs (pg_cron)

```sql
-- Reminder booking H-1
SELECT cron.schedule(
  'booking-reminder-daily',
  '0 8 * * *',  -- setiap hari jam 8 pagi
  $$SELECT net.http_post(
    url := current_setting('app.edge_url') || '/notification-booking-reminder',
    headers := '{"Authorization": "Bearer ' || current_setting('app.service_role_key') || '"}'::jsonb
  );$$
);

-- Reminder kedaluwarsa produk (H-30, H-7, H-1)
SELECT cron.schedule(
  'expiry-check-daily',
  '0 6 * * *',
  $$SELECT net.http_post(
    url := current_setting('app.edge_url') || '/notification-expiry-alert',
    headers := '{"Authorization": "Bearer ' || current_setting('app.service_role_key') || '"}'::jsonb
  );$$
);

-- Rekonsiliasi stok harian
SELECT cron.schedule(
  'stock-reconciliation-daily',
  '0 23 * * *',
  $$SELECT net.http_post(
    url := current_setting('app.edge_url') || '/inventory-reconcile',
    headers := '{"Authorization": "Bearer ' || current_setting('app.service_role_key') || '"}'::jsonb
  );$$
);

-- Backup database harian
SELECT cron.schedule(
  'backup-daily',
  '0 2 * * *',
  $$SELECT net.http_post(
    url := current_setting('app.edge_url') || '/backup-create',
    headers := '{"Authorization": "Bearer ' || current_setting('app.service_role_key') || '"}'::jsonb
  );$$
);

-- Cleanup idempotency keys expired
SELECT cron.schedule(
  'cleanup-idempotency',
  '0 * * * *',
  $$DELETE FROM idempotency_keys WHERE expires_at < now();$$
);
```

---

# 📄 PETORA-BLUEPRINT-005
## Frontend Architecture

### 5.1 Struktur Aplikasi

**3 aplikasi terpisah**:
1. `web-staff` — Dashboard staf (admin, manager, groomer, vet, kasir non-POS)
2. `web-pos` — Terminal POS (optimized untuk touchscreen, fullscreen)
3. `web-customer` — Portal pelanggan

### 5.2 Routing `web-staff`

```
/                                  → Dashboard
/login                             → Login
/forgot-password                   → Forgot password
/reset-password                    → Reset password

/customers                         → List pelanggan
/customers/:id                     → Detail pelanggan
/customers/:id/pets                → Hewan pelanggan
/customers/:id/transactions        → Transaksi pelanggan

/pets                              → List hewan
/pets/:id                          → Detail hewan
/pets/:id/medical-records          → Rekam medis
/pets/:id/vaccinations             → Vaksinasi

/products                          → List produk
/products/create                   → Tambah produk
/products/:id                      → Detail produk
/products/:id/stock                → Stok produk
/products/categories               → Kategori produk
/products/brands                   → Brand
/products/units                    → Satuan

/inventory                         → Dashboard inventory
/inventory/stock                   → Stok semua produk
/inventory/movements               → Pergerakan stok
/inventory/adjustments             → Adjustment
/inventory/transfers               → Transfer
/inventory/stock-opname            → Stock opname
/inventory/warehouses              → Gudang
/inventory/expiry                  → Produk kedaluwarsa
/inventory/low-stock               → Stok minimum

/sales                             → Dashboard penjualan
/sales/transactions                → List transaksi
/sales/transactions/:id            → Detail transaksi
/sales/returns                     → Retur
/sales/shifts                      → Shift kasir

/purchases                         → Dashboard pembelian
/purchases/suppliers               → Supplier
/purchases/orders                  → Purchase orders
/purchases/orders/create           → Buat PO
/purchases/orders/:id              → Detail PO
/purchases/receiving               → Penerimaan barang
/purchases/payments                → Pembayaran supplier

/services                          → Layanan
/services/categories               → Kategori layanan
/services/list                     → List layanan
/services/pricing                  → Harga per cabang

/booking                           → Kalender booking
/booking/list                      → List booking
/booking/create                    → Buat booking
/booking/:id                       → Detail booking

/grooming                          → Dashboard grooming
/grooming/today                    → Antrian hari ini
/grooming/schedule                 → Jadwal groomer
/grooming/history                  → Riwayat
/grooming/staff-performance        → Performa groomer

/pet-hotel                         → Dashboard pet hotel
/pet-hotel/rooms                   → Kamar
/pet-hotel/occupancy               → Okupansi
/pet-hotel/stays                   → Penitipan aktif
/pet-hotel/stays/:id               → Detail penitipan
/pet-hotel/history                 → Riwayat

/veterinary                        → Dashboard klinik
/veterinary/queue                  → Antrian
/veterinary/appointments           → Jadwal konsultasi
/veterinary/consultations          → Konsultasi
/veterinary/consultations/new      → Buat rekam medis
/veterinary/consultations/:id      → Detail rekam medis
/veterinary/medical-records        → Semua rekam medis
/veterinary/doctors                → Dokter hewan
/veterinary/laboratory             → Lab
/veterinary/surgery                → Operasi
/veterinary/hospitalization        → Rawat inap

/pharmacy                          → Dashboard farmasi
/pharmacy/stock                    → Stok obat
/pharmacy/prescriptions            → Resep aktif
/pharmacy/dispensing               → Pengeluaran obat
/pharmacy/controlled               → Obat terkontrol

/membership                        → Dashboard membership
/membership/members                → Anggota
/membership/tiers                  → Tier
/membership/benefits               → Benefit

/loyalty                           → Dashboard loyalty
/loyalty/members                   → Pemilik poin
/loyalty/transactions              → Transaksi poin
/loyalty/rewards                   → Reward catalog

/promotions                        → Dashboard promo
/promotions/vouchers               → Voucher
/promotions/discounts              → Diskon
/promotions/bundles                → Bundling
/promotions/campaigns              → Kampanye

/employees                         → Karyawan
/employees/list                    → List
/employees/attendance              → Absensi
/employees/schedule                → Jadwal kerja
/employees/commission              → Komisi
/employees/payroll                 → Penggajian
/employees/payroll/:period         → Detail payroll
/employees/positions               → Jabatan
/employees/departments             → Departemen

/finance                           → Dashboard keuangan
/finance/cash                      → Kas
/finance/cash-flow                 → Arus kas
/finance/banks                     → Rekening bank
/finance/receivables               → Piutang
/finance/payables                  → Hutang
/finance/budget                    → Anggaran

/accounting                        → Akuntansi
/accounting/chart-of-accounts      → COA
/accounting/journals               → Jurnal
/accounting/journals/create        → Buat jurnal
/accounting/periods                → Periode
/accounting/trial-balance          → Neraca saldo
/accounting/profit-loss            → Laba rugi
/accounting/balance-sheet          → Neraca
/accounting/cash-flow-statement    → Arus kas

/reports                           → Laporan
/reports/sales                     → Penjualan
/reports/purchases                 → Pembelian
/reports/inventory                 → Inventory
/reports/profit-loss               → Laba rugi
/reports/tax                       → Pajak
/reports/customers                 → Pelanggan
/reports/employees                 → Karyawan

/crm                               → CRM
/crm/customers                     → Pelanggan
/crm/segments                      → Segmentasi
/crm/campaigns                     → Kampanye
/crm/complaints                    → Keluhan
/crm/complaints/:id                → Detail keluhan

/operations                        → Operasional
/operations/checklists             → Checklist
/operations/sop                    → SOP
/operations/assets                 → Aset
/operations/documents              → Dokumen

/settings                          → Pengaturan
/settings/organization             → Organisasi
/settings/branches                 → Cabang
/settings/users                    → User
/settings/roles                    → Role
/settings/integrations             → Integrasi
/settings/notifications            → Notifikasi
/settings/tax                      → Pajak
/settings/backup                   → Backup

/super-admin                       → Super Admin (khusus)
/super-admin/config                → Konfigurasi global
/super-admin/integrations          → Plugin integrasi
/super-admin/audit-logs            → Audit log
/super-admin/system-health         → Kesehatan sistem
```

### 5.3 Routing `web-pos`

```
/                       → Login POS / pilih kasir
/pos                    → Layar POS utama
/pos/hold               → Transaksi tertunda
/pos/transactions       → Transaksi shift ini
/pos/close-shift        → Tutup shift
/pos/settings           → Pengaturan POS
```

### 5.4 Routing `web-customer`

```
/                       → Landing
/login                  → Login
/register               → Register
/dashboard              → Dashboard
/pets                   → Hewan saya
/pets/:id               → Detail hewan
/pets/:id/medical       → Rekam medis
/bookings               → Booking saya
/bookings/create        → Buat booking
/bookings/:id           → Detail booking
/transactions           → Transaksi saya
/membership             → Membership
/loyalty                → Loyalty
/profile                → Profil
```

### 5.5 State Management

```typescript
// stores/authStore.ts
interface AuthState {
  user: User | null;
  session: Session | null;
  permissions: Set<string>;
  currentBranch: Branch | null;
  availableBranches: Branch[];
  
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  switchBranch: (branchId: string) => void;
  hasPermission: (code: string) => boolean;
}

// stores/posStore.ts
interface PosState {
  currentTransaction: DraftTransaction;
  heldTransactions: DraftTransaction[];
  selectedCustomer: Customer | null;
  selectedPet: Pet | null;
  
  addItem: (product: Product, qty: number) => void;
  removeItem: (itemId: string) => void;
  updateItemQty: (itemId: string, qty: number) => void;
  applyDiscount: (discount: Discount) => void;
  applyVoucher: (voucher: Voucher) => void;
  redeemPoints: (points: number) => void;
  holdTransaction: () => void;
  recallTransaction: (id: string) => void;
  clearTransaction: () => void;
  calculateTotals: () => TransactionTotals;
}

// Server state via React Query
// Contoh:
const { data: products } = useQuery({
  queryKey: ['products', { branchId, categoryId, search }],
  queryFn: () => fetchProducts({ branchId, categoryId, search }),
  staleTime: 30_000,
});
```

### 5.6 Komponen UI Utama

**Layout**:
- `AppShell` — Sidebar + Topbar + Main content
- `Sidebar` — Navigation, collapsible
- `Topbar` — User menu, branch selector, notifications
- `PageHeader` — Title, breadcrumb, actions

**Form**:
- `FormField` — Label + input + error
- `FormSelect`, `FormDatePicker`, `FormAutocomplete`
- `SearchInput` — Debounced search
- `FileUpload` — Drag & drop

**Data Display**:
- `DataTable` — TanStack Table + virtualization
- `DataGrid` — Untuk POS (touch-optimized)
- `StatCard` — Metric card
- `Timeline` — Untuk riwayat
- `KanbanBoard` — Untuk status workflow

**Feedback**:
- `Toast` — Notifikasi singkat
- `Dialog` — Modal konfirmasi
- `Drawer` — Side panel
- `AlertBanner` — Peringatan

**POS-specific**:
- `ProductGrid` — Grid produk dengan gambar
- `Numpad` — Numeric keypad
- `PaymentModal` — Multi-payment dialog
- `ReceiptPreview` — Preview struk

### 5.7 Offline Support (POS)

```typescript
// lib/offlineQueue.ts
class OfflineQueue {
  private queue: QueuedRequest[] = [];
  
  async enqueue(request: QueuedRequest): Promise<void> {
    this.queue.push(request);
    await this.persist();
  }
  
  async sync(): Promise<void> {
    while (this.queue.length > 0) {
      const req = this.queue[0];
      try {
        await fetch(req.url, req.options);
        this.queue.shift();
      } catch (err) {
        if (err instanceof NetworkError) break;
        this.queue.shift(); // skip invalid
      }
    }
    await this.persist();
  }
  
  private async persist(): Promise<void> {
    localStorage.setItem('offline_queue', JSON.stringify(this.queue));
  }
}

// Hook
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

### 5.8 Print (Struk Thermal)

```typescript
// lib/thermalPrinter.ts
import { ThermalPrinter } from 'react-thermal-printer';

export async function printReceipt(transaction: SalesTransaction) {
  const config = await getConfig('pos.receipt');
  
  const element = (
    <ThermalPrinter width={58}>
      <Line text={config.store_name} align="center" bold />
      <Line text={config.store_address} align="center" />
      <Line text={`#${transaction.transaction_number}`} align="center" />
      <Line text={formatDate(transaction.transaction_time)} align="center" />
      <Divider />
      <Line text={`Kasir: ${transaction.cashier.name}`} />
      {transaction.customer && <Line text={`Customer: ${transaction.customer.name}`}/> }
      <Divider />
      {transaction.items.map(item => (
        <Line key={item.id} left={item.description} right={formatCurrency(item.total)} />
      ))}
      <Divider />
      <Line left="Subtotal" right={formatCurrency(transaction.subtotal)} />
      <Line left="Diskon" right={formatCurrency(transaction.total_discount)} />
      <Line left="Pajak" right={formatCurrency(transaction.tax_amount)} />
      <Line left="TOTAL" right={formatCurrency(transaction.grand_total)} bold />
      <Line left="Bayar" right={formatCurrency(transaction.paid_amount)} />
      <Line left="Kembali" right={formatCurrency(transaction.change_amount)} />
      <Divider />
      <Line text={config.footer_text} align="center" />
    </ThermalPrinter>
  );
  
  await print(element);
}
```

### 5.9 Realtime Updates

```typescript
// hooks/useRealtimeBookings.ts
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
      }, (payload) => {
        queryClient.invalidateQueries({ queryKey: ['bookings', branchId] });
      })
      .subscribe();
    
    return () => { supabase.removeChannel(channel); };
  }, [branchId]);
}
```

---

# 📄 PETORA-BLUEPRINT-006
## Domain Models & Business Rules

### 6.1 Domain Entities

```
┌─────────────────────────────────────────────────────────────┐
│                    ORGANIZATION DOMAIN                        │
│  Organization → Branch → Department → Position → Employee   │
│                 ↓                                           │
│              User → Role → Permission                       │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOMER DOMAIN                            │
│  Customer → Pet → Vaccination → MedicalRecord               │
│           ↓                                                 │
│       Membership → LoyaltyAccount → RewardRedemption        │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    PRODUCT DOMAIN                             │
│  Product → Category / Brand / Unit                          │
│         ↓                                                   │
│  StockItem → StockMovement                                  │
│         ↓                                                   │
│  Warehouse                                                  │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    SALES DOMAIN                               │
│  SalesTransaction → Item / Payment                          │
│                   ↓                                         │
│  CashierShift → SalesReturn                                 │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    PURCHASE DOMAIN                            │
│  Supplier → PurchaseOrder → Item                            │
│           → PurchaseReceipt → PurchasePayment               │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    SERVICE DOMAIN                             │
│  Service → Category → Price                                 │
│         ↓                                                   │
│  Booking → GroomingSession / BoardinStay / Consultation     │
│         ↓                                                   │
│  Room (hotel) / Staff (groomer) / Doctor (vet)              │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    FINANCE DOMAIN                             │
│  ChartOfAccount → JournalEntry → Line                       │
│  CashFlow / Receivable / Payable                            │
│  PayrollPeriod → Payroll → PayrollComponent                 │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Business Rules (Invariants)

#### 6.2.1 Sales
| ID | Rule | Enforcement |
|---|---|---|
| `BR-SALES-001` | Total pembayaran harus ≥ grand total (kecuali kredit diizinkan) | DB constraint + FE validation |
| `BR-SALES-002` | Stok tidak boleh negatif saat penjualan (kecuali config allow_negative) | Pre-check di Edge Function |
| `BR-SALES-003` | Void hanya bisa sebelum pembayaran final | Status check |
| `BR-SALES-004` | Refund maksimal 30 hari dari transaksi asli (configurable) | Date check |
| `BR-SALES-005` | Refund butuh approval role `pos.refund.approve` | Permission check |
| `BR-SALES-006` | Diskon item tidak boleh melebihi `pos.max_discount_percent` | Validation |
| `BR-SALES-007` | Voucher tidak bisa digunakan melebihi `max_usage` | Stock check |
| `BR-SALES-008` | Multi-payment total harus = grand total | Validation |
| `BR-SALES-009` | Transaksi yang sudah finalized tidak bisa diedit | Status check |
| `BR-SALES-010` | Setiap transaksi harus punya minimal 1 payment method | Validation |

#### 6.2.2 Inventory
| ID | Rule | Enforcement |
|---|---|---|
| `BR-INV-001` | Stok tidak bisa diubah langsung, harus via movement | RLS + trigger |
| `BR-INV-002` | Transfer harus approval jika value > threshold | Workflow |
| `BR-INV-003` | Stock opname harus ada selisih yang di-approve | Workflow |
| `BR-INV-004` | Batch dengan expiry date wajib untuk produk `has_expiry=true` | Validation |
| `BR-INV-005` | FEFO: saat sales, ambil batch dengan expiry terdekat | Algorithm |
| `BR-INV-006` | Produk bundled tidak bisa dijual jika komponen < kebutuhan | Stock check |

#### 6.2.3 Booking
| ID | Rule | Enforcement |
|---|---|---|
| `BR-BOOK-001` | 1 staff tidak bisa handle 2 booking di waktu sama | Schedule conflict check |
| `BR-BOOK-002` | 1 room tidak bisa ditempati 2 hewan di waktu sama | Room conflict check |
| `BR-BOOK-003` | Booking harus minimal 1 jam dari sekarang (configurable) | Time check |
| `BR-BOOK-004` | Cancel setelah confirmed butuh alasan | Validation |
| `BR-BOOK-005` | No-show otomatis dibatalkan setelah 30 menit dari jadwal | Cron job |

#### 6.2.4 Veterinary
| ID | Rule | Enforcement |
|---|---|---|
| `BR-VET-001` | Rekam medis hanya bisa dibuat oleh veterinarian | Permission check |
| `BR-VET-002` | Rekam medis finalized tidak bisa diedit langsung | Status check |
| `BR-VET-003` | Revisi harus menyimpan snapshot sebelumnya | Audit trigger |
| `BR-VET-004` | Obat `requires_prescription=true` hanya bisa keluar dengan resep | Validation |
| `BR-VET-005` | Dokter tidak bisa meresepkan untuk hewan yang tidak diperiksa | Validation |
| `BR-VET-006` | Peringatan alergi obat jika pet punya alergi yang sama | Alert |

#### 6.2.5 Finance
| ID | Rule | Enforcement |
|---|---|---|
| `BR-FIN-001` | Total debit harus = total kredit | DB constraint |
| `BR-FIN-002` | Jurnal yang sudah posted tidak bisa dihapus, harus reversal | Status check |
| `BR-FIN-003` | Periode yang sudah closed tidak bisa di-journal | Period check |
| `BR-FIN-004` | Pembayaran supplier tidak boleh melebihi hutang | Validation |

#### 6.2.6 Payroll
| ID | Rule | Enforcement |
|---|---|---|
| `BR-PAY-001` | Payroll tidak bisa dihitung jika absensi belum approved | Validation |
| `BR-PAY-002` | Payroll yang sudah approved tidak bisa diedit | Status check |
| `BR-PAY-003` | 1 karyawan tidak bisa punya 2 payroll aktif di periode sama | Validation |

### 6.3 Enum Definitions

```typescript
// packages/constants/src/enums.ts

export const TransactionStatus = {
  DRAFT: 'draft',
  COMPLETED: 'completed',
  VOIDED: 'voided',
  REFUNDED: 'refunded',
  PENDING: 'pending',
} as const;

export const PaymentMethod = {
  CASH: 'cash',
  DEBIT_CARD: 'debit_card',
  CREDIT_CARD: 'credit_card',
  QRIS: 'qris',
  BANK_TRANSFER: 'bank_transfer',
  E_WALLET: 'e_wallet',
  VOUCHER: 'voucher',
  LOYALTY_POINT: 'loyalty_point',
  CREDIT_ACCOUNT: 'credit_account',
  OTHER: 'other',
} as const;

export const BookingStatus = {
  PENDING: 'pending',
  CONFIRMED: 'confirmed',
  CHECKED_IN: 'checked_in',
  IN_PROGRESS: 'in_progress',
  COMPLETED: 'completed',
  CANCELLED: 'cancelled',
  NO_SHOW: 'no_show',
  RESCHEDULED: 'rescheduled',
} as const;

export const MedicalRecordStatus = {
  DRAFT: 'draft',
  FINALIZED: 'finalized',
  AMENDED: 'amended',
} as const;

export const StockMovementType = {
  PURCHASE_IN: 'purchase_in',
  SALES_OUT: 'sales_out',
  ADJUSTMENT_IN: 'adjustment_in',
  ADJUSTMENT_OUT: 'adjustment_out',
  TRANSFER_IN: 'transfer_in',
  TRANSFER_OUT: 'transfer_out',
  RETURN_IN: 'return_in',
  RETURN_OUT: 'return_out',
  DAMAGE_OUT: 'damage_out',
  LOSS_OUT: 'loss_out',
  PRODUCTION_IN: 'production_in',
  CONSUMPTION_OUT: 'consumption_out',
  OPNAME_ADJUST: 'opname_adjust',
  CONSIGNMENT_IN: 'consignment_in',
  CONSIGNMENT_OUT: 'consignment_out',
} as const;

export const PetSpecies = {
  DOG: 'dog',
  CAT: 'cat',
  BIRD: 'bird',
  RABBIT: 'rabbit',
  FISH: 'fish',
  REPTILE: 'reptile',
  HAMSTER: 'hamster',
  GUINEA_PIG: 'guinea_pig',
  OTHER: 'other',
} as const;

export const PetSize = {
  SMALL: 'small',       // < 10kg
  MEDIUM: 'medium',     // 10-25kg
  LARGE: 'large',       // 25-45kg
  XLARGE: 'xlarge',     // > 45kg
} as const;

export const ServiceType = {
  GROOMING: 'grooming',
  BOARDING: 'boarding',
  DAYCARE: 'daycare',
  CONSULTATION: 'consultation',
  MEDICAL: 'medical',
  TRANSPORT: 'transport',
  HOME_SERVICE: 'home_service',
  OTHER: 'other',
} as const;

export const AccountType = {
  ASSET: 'asset',
  LIABILITY: 'liability',
  EQUITY: 'equity',
  REVENUE: 'revenue',
  EXPENSE: 'expense',
} as const;
```

### 6.4 Permission Matrix (Default)

| Permission | Super Admin | Owner | Branch Mgr | Admin | Kasir | Groomer | Vet | Inventory |
|---|---|---|---|---|---|---|---|---|
| `org:organization:*` | ✓ | ✓ | — | — | — | — | — | — |
| `org:branch:*` | ✓ | ✓ | read | read | — | — | — | — |
| `org:user:*` | ✓ | ✓ | read(branch) | — | — | — | — | — |
| `org:role:*` | ✓ | ✓ | — | — | — | — | — | — |
| `customer:customer:*` | ✓ | ✓ | ✓(branch) | ✓(branch) | read(branch) | read(branch) | read(branch) | read(branch) |
| `pet:pet:*` | ✓ | ✓ | ✓(branch) | ✓(branch) | read(branch) | read(branch) | ✓(branch) | read(branch) |
| `product:product:*` | ✓ | ✓ | ✓(branch) | ✓ | read(branch) | read(branch) | read(branch) | ✓(branch) |
| `inventory:stock:read` | ✓ | ✓ | ✓(branch) | ✓(branch) | read(branch) | read(branch) | read(branch) | ✓(branch) |
| `inventory:stock:adjust` | ✓ | ✓ | ✓(branch) | ✓(branch) | — | — | — | ✓(branch) |
| `pos:transaction:create` | ✓ | ✓ | ✓(branch) | ✓(branch) | ✓(branch) | — | — | — |
| `pos:transaction:void` | ✓ | ✓ | ✓(branch) | — | — | — | — | — |
| `pos:transaction:refund` | ✓ | ✓ | ✓(branch) | — | — | — | — | — |
| `purchase:order:*` | ✓ | ✓ | ✓(branch) | ✓(branch) | — | — | — | ✓(branch) |
| `purchase:order:approve` | ✓ | ✓ | ✓(branch) | — | — | — | — | — |
| `booking:booking:create` | ✓ | ✓ | ✓(branch) | ✓(branch) | ✓(branch) | read(branch) | read(branch) | — |
| `grooming:session:*` | ✓ | ✓ | ✓(branch) | ✓(branch) | read(branch) | ✓(own) | — | — |
| `vet:medical_record:create` | ✓ | ✓ | ✓(branch) | read(branch) | — | — | ✓(branch) | — |
| `vet:medical_record:finalize` | ✓ | ✓ | ✓(branch) | — | — | — | ✓(branch) | — |
| `pharmacy:dispense` | ✓ | ✓ | ✓(branch) | ✓(branch) | — | — | ✓(branch) | — |
| `finance:journal:create` | ✓ | ✓ | — | — | — | — | — | — |
| `finance:journal:approve` | ✓ | ✓ | — | — | — | — | — | — |
| `hr:employee:*` | ✓ | ✓ | read(branch) | ✓(branch) | — | — | — | — |
| `hr:payroll:*` | ✓ | ✓ | — | — | — | — | — | — |
| `report:*:read` | ✓ | ✓ | ✓(branch) | ✓(branch) | read(branch) | read(own) | read(own) | read(branch) |
| `settings:config:*` | ✓ | ✓ | — | — | — | — | — | — |
| `settings:integration:*` | ✓ | — | — | — | — | — | — | — |
| `super_admin:*` | ✓ | — | — | — | — | — | — | — |

---

# 📄 PETORA-BLUEPRINT-007
## Modul A — Organisasi & Dashboard

### 7.1 Dashboard Management

**Endpoint**: `GET /dashboard/summary`

**Response**:
```json
{
  "period": { "start": "2026-11-01", "end": "2026-11-07" },
  "revenue": {
    "today": 5500000,
    "this_week": 35000000,
    "this_month": 150000000,
    "this_year": 1500000000,
    "vs_last_period_percent": 12.5
  },
  "transactions": {
    "today_count": 45,
    "this_month_count": 1200,
    "average_value": 125000
  },
  "customers": {
    "total_registered": 2500,
    "new_this_month": 85,
    "active_this_month": 450
  },
  "top_products": [
    { "product_id": "...", "name": "Royal Canin Adult 2kg", "qty_sold": 120, "revenue": 12000000 }
  ],
  "top_services": [
    { "service_id": "...", "name": "Full Grooming Dog M", "count": 85, "revenue": 8500000 }
  ],
  "alerts": [
    { "type": "low_stock", "count": 15, "severity": "warning" },
    { "type": "expiry_soon", "count": 8, "severity": "warning" },
    { "type": "pending_approval", "count": 3, "severity": "info" }
  ],
  "branch_performance": [
    { "branch_id": "...", "name": "Jakarta Pusat", "revenue": 80000000, "transactions": 650 }
  ]
}
```

### 7.2 User Management

**Fitur**:
- Create user dengan email + temporary password
- Assign role(s) + branch scope
- Activate / deactivate user
- Reset password
- Force logout
- View audit log user

**Workflow create user**:
1. Admin isi form: email, name, phone, role, branch
2. System generate temporary password
3. Email dikirim ke user dengan link aktivasi
4. User set password sendiri saat first login
5. MFA opsional diaktifkan

### 7.3 Role & Permission

**Default system roles** (tidak bisa dihapus):
- `super_admin` — full access
- `owner` — all except super_admin
- `branch_manager` — branch-scoped full access
- `administrator` — operational admin
- `cashier` — POS only
- `groomer` — grooming only
- `veterinarian` — vet + medical record
- `pharmacist` — pharmacy only
- `inventory_staff` — inventory only
- `hr_staff` — employee + payroll
- `finance_staff` — finance + accounting
- `customer_service` — customer + complaint
- `customer` — portal only

Custom role bisa dibuat dengan subset permission.

### 7.4 Branch Management

**Fitur**:
- Create branch dengan code unik
- Set operating hours per hari
- Set services available per branch
- Set prices per branch (override global)
- Assign employees ke branch
- Activate / deactivate branch
- Transfer stock antar branch

---

# 📄 PETORA-BLUEPRINT-008
## Modul B — Pelanggan, Hewan, Produk

### 8.1 Customer Management

**Form fields**:
- Nama lengkap (wajib)
- Nama panggilan
- Gender
- Tanggal lahir
- No HP (wajib, unik per org, encrypted)
- Email (encrypted)
- Alamat
- Kota
- Foto
- Membership
- Notes
- Marketing consent

**Fitur**:
- Search by name/phone (fuzzy via pg_trgm)
- Filter by membership tier, city, blacklist
- View: detail, transactions, pets, bookings, medical records
- Merge duplicate customers
- Export to CSV
- Blacklist management

### 8.2 Pet Management

**Form fields**:
- Nama (wajib)
- Species (wajib)
- Breed
- Gender
- Tanggal lahir / usia
- Warna
- Ciri khusus
- Berat
- Microchip
- Steril
- Foto
- Alergi (multi)
- Kondisi kronis (multi)
- Catatan perilaku
- Owner(s) (multi, 1 primary)

**Fitur**:
- Register pet via customer profile atau standalone
- Multi-owner support
- Timeline view: semua aktivitas (transaksi, grooming, vet, hotel)
- Vaccination schedule reminder
- Weight chart
- Deceased marking

### 8.3 Product Management

**Form fields**:
- SKU (wajib, unik)
- Barcode
- Nama (wajib)
- Kategori
- Brand
- Tipe: single / bundle / service / consignment
- Satuan jual (wajib)
- Satuan beli
- Konversi (contoh: 1 karton = 12 pcs)
- Harga modal
- Harga jual (wajib)
- Harga grosir
- Min qty grosir
- Harga minimum (batas diskon)
- Berat / dimensi
- Batch tracking
- Expiry tracking
- Requires prescription
- Foto
- Deskripsi

**Fitur**:
- Bulk import via Excel
- Bulk export
- Bulk price update
- Duplicate product
- Archive (soft delete)
- Barcode label print
- Variant management (warna, ukuran)
- Bundle composition

### 8.4 Inventory Management

**Sub-modul**:
1. **Stock Overview** — Grid produk × gudang, filter by category/brand/low stock
2. **Stock Card** — Detail pergerakan per produk (kardex)
3. **Stock Adjustment** — Penyesuaian stok (rusak, hilang, koreksi)
4. **Stock Transfer** — Transfer antar gudang/cabang
5. **Stock Opname** — Cicil opname, rekam selisih, approve adjustment
6. **Expiry Monitoring** — List produk mendekati kedaluwarsa
7. **Low Stock Alert** — List produk di bawah minimum
8. **Warehouse Management** — CRUD gudang

**Stock movement workflow**:
```
Adjustment:
  Staff → Create adjustment → Approval (if > threshold) → Commit movement → Update stock

Transfer:
  Staff → Create transfer request → Approval → Source warehouse: reserve → 
  In-transit status → Destination: receive → Commit both sides

Stock Opname:
  Staff → Create opname session → Count items → System calculate variance →
  Approval → Commit adjustment movements
```

---

# 📄 PETORA-BLUEPRINT-009
## Modul C — Penjualan & Pelayanan

### 9.1 POS Terminal

**Layout**:
```
┌─────────────────────────────────────────────────────────────┐
│ [Branch: JKT01] [Kasir: Budi] [Shift: Pagi]    [⚙] [⎋]    │
├─────────────────────────────┬───────────────────────────────┤
│ [Search...] [Cat: All ▼]    │ Customer: [Pilih]  Pet: [-]  │
│                             │───────────────────────────────│
│ ┌─────┐ ┌─────┐ ┌─────┐   │ # | Item            | Qty | $ │
│ │ img │ │ img │ │ img │   │ 1 | RC Adult 2kg    |  1  │100│
│ │ RC  │ │ Whi │ │ Ped │   │ 2 | Grooming Dog M  |  1  │ 80│
│ │100k │ │ 50k │ │ 30k │   │───────────────────────────────│
│ └─────┘ └─────┘ └─────┘   │ Subtotal              180.000 │
│ ┌─────┐ ┌─────┐ ┌─────┐   │ Diskon Item           -10.000 │
│ │ ... │ │ ... │ │ ... │   │ Diskon Cart            -5.000 │
│ └─────┘ └─────┘ └─────┘   │ Voucher               -10.000 │
│                             │ Poin (50)              -5.000 │
│                             │ Pajak (11%)            16.500 │
│                             │───────────────────────────────│
│                             │ TOTAL                 186.500 │
│                             │───────────────────────────────│
│                             │ [Hold] [Disc] [Vouch] [Pay ▶]│
└─────────────────────────────┴───────────────────────────────┘
```

**Workflow POS**:
1. Kasir login → pilih shift
2. Pilih customer (opsional) → pilih pet
3. Scan barcode atau search produk → add to cart
4. Tambah layanan (grooming, dll)
5. Apply diskon / voucher / poin
6. Klik Pay → pilih payment method(s)
7. Process payment → print receipt
8. System: reduce stock, add loyalty, post journal, emit event

**Payment flow**:
```
[Pay] → Modal:
  ┌─────────────────────────────────────┐
  │ Total: Rp186.500                    │
  │─────────────────────────────────────│
  │ Cash     [___] [100k] [200k]       │
  │ QRIS     [___] [Scan]              │
  │ Debit    [___] [EDC]               │
  │ Transfer [___] [Confirm]           │
  │─────────────────────────────────────│
  │ Total Paid: Rp200.000               │
  │ Change: Rp13.500                    │
  │ [Cancel] [Process ✓]                │
  └─────────────────────────────────────┘
```

### 9.2 Grooming Management

**Workflow**:
1. Booking dibuat (oleh staff/customer)
2. Groomer check jadwal
3. Hewan check-in → pemeriksaan kondisi awal (foto, catatan kulit/bulu)
4. Status: `checked_in`
5. Groomer mulai kerja → status: `in_progress`
6. Dokumentasi progress
7. Selesai → status: `completed` → foto after
8. Customer notified → pick up
9. Payment di POS atau langsung

**Checklist grooming**:
- [ ] Pemeriksaan kutu
- [ ] Mandi
- [ ] Blow dry
- [ ] Brushing
- [ ] Gunting kuku
- [ ] Cleaning telinga
- [ ] Gunting bulu (style)
- [ ] Final check

### 9.3 Pet Hotel

**Workflow**:
1. Booking kamar (pilih kamar, tanggal check-in/out)
2. Check-in:
   - Verifikasi booking
   - Pencatatan kondisi awal (foto, berat, perilaku)
   - Serah terima makanan/obat dari owner
   - Input feeding & medication schedule
3. Daily monitoring:
   - Log makan/minum
   - Log buang air
   - Log aktivitas/perilaku
   - Foto perkembangan
   - Timbang berat (jika perlu)
4. Owner update via portal/WA
5. Check-out:
   - Final condition check
   - Calculate total (daily rate × days + additional)
   - Payment
   - Foto serah terima

**Room status board**:
```
┌────────────────────────────────────────────────────────────┐
│ Pet Hotel - Jakarta Pusat                                  │
├────────────────────────────────────────────────────────────┤
│ Standard A1: [🟢 Available]                                │
│ Standard A2: [🔵 Occupied - Buddy 🐕 (2/5 days)]          │
│ Standard A3: [🔵 Occupied - Mimi 🐈 (1/3 days)]           │
│ Deluxe B1:   [🟡 Cleaning]                                 │
│ Deluxe B2:   [🔵 Occupied - Max 🐕 (4/7 days)]            │
│ VIP C1:      [🟢 Available]                                │
│ Cat Room D1: [🔵 Occupied - Luna 🐈 (3/5 days)]           │
└────────────────────────────────────────────────────────────┘
```

### 9.4 Booking & Reservation

**Calendar view**:
- Day / Week / Month view
- Filter by staff / room / service
- Drag & drop untuk reschedule
- Color by status

**Conflict detection**:
```typescript
function checkBookingConflict(newBooking: BookingInput): ConflictResult {
  const staffConflict = db.query(bookings).where({
    staff_id: newBooking.staff_id,
    status: ['confirmed', 'checked_in', 'in_progress'],
    start_time: lt(newBooking.end_time),
    end_time: gt(newBooking.start_time),
  });
  
  const roomConflict = newBooking.room_id ? db.query(bookings).where({
    room_id: newBooking.room_id,
    ...
  }) : [];
  
  return { staffConflict, roomConflict };
}
```

---

# 📄 PETORA-BLUEPRINT-010
## Modul D — Klinik & Farmasi

### 10.1 Veterinary Consultation

**Workflow**:
1. Pasien datang → registrasi / pilih existing
2. Triage (opsional) → prioritas
3. Masuk antrian dokter
4. Dokter panggil pasien
5. Anamnesa (keluhan, riwayat)
6. Pemeriksaan fisik (vital signs, fisik)
7. Diagnosis
8. Tindakan (jika ada)
9. Resep / rekomendasi
10. Finalisasi rekam medis
11. Pasien ke farmasi (jika ada obat)
12. Pembayaran

**Form rekam medis**:
```
┌─────────────────────────────────────────────────────────────┐
│ Rekam Medis - #RM/2026/11/000123                           │
├─────────────────────────────────────────────────────────────┤
│ Pasien: Buddy (Dog, Male, 3y) | Owner: John                │
│ Dokter: drh. Sarah | Tanggal: 07 Nov 2026                  │
├─────────────────────────────────────────────────────────────┤
│ [Anamnesa]                                                  │
│ Keluhan utama: _____________                                │
│ Riwayat: _______________________________________________    │
├─────────────────────────────────────────────────────────────┤
│ [Pemeriksaan Fisik]                                         │
│ Suhu: [__]°C  HR: [__]bpm  RR: [__]x/m  BB: [__]kg        │
│ Mata: [normal/abnormal]  Telinga: [n/a]  Hidung: [n/a]     │
│ Mulut: [n/a]  Kulit: [n/a]  Bulu: [n/a]                    │
├─────────────────────────────────────────────────────────────┤
│ [Diagnosis]                                                 │
│ Diagnosis: ___________________________________              │
│ ICD Code: [____]                                            │
├─────────────────────────────────────────────────────────────┤
│ [Tindakan]                                                  │
│ + Tambah tindakan (vaksin, injeksi, dll)                   │
├─────────────────────────────────────────────────────────────┤
│ [Resep]                                                     │
│ + Tambah obat                                               │
│   Amoxicillin 500mg - 1 tablet, 2x sehari, 7 hari          │
├─────────────────────────────────────────────────────────────┤
│ [Rekomendasi]                                               │
│ _______________________________________________             │
│ Follow up: [____]                                           │
├─────────────────────────────────────────────────────────────┤
│ [Draft] [Finalize ✓]                                        │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 Pharmacy

**Workflow pengeluaran obat**:
1. Resep dari dokter (status: `active`)
2. Pharmacist pilih resep
3. System cek stok
4. Pharmacist input qty dispensed
5. System reduce stok + create movement
6. Print label obat
7. Status resep → `dispensed`
8. Payment (via POS atau langsung)

**Alert**:
- Stok obat hampir habis
- Obat mendekati kedaluwarsa
- Interaksi obat (jika ada referensi)
- Alergi pasien vs obat

### 10.3 Laboratory

**Workflow**:
1. Dokter request pemeriksaan
2. Sample diambil → label
3. Pemeriksaan dilakukan
4. Input hasil
5. Validasi oleh dokter/penanggung jawab
6. Hasil masuk rekam medis
7. Notifikasi ke customer (jika diizinkan)

### 10.4 Surgery & Hospitalization

**Workflow operasi**:
1. Dokter indikasi operasi
2. Persetujuan owner (informed consent digital)
3. Jadwal operasi
4. Pre-op check
5. Operasi (catat tim, bahan, anestesi)
6. Post-op monitoring
7. Rawat inap (jika perlu)
8. Discharge

---

# 📄 PETORA-BLUEPRINT-011
## Modul E — Membership, Loyalty, Promo

### 11.1 Membership

**Tier structure (configurable)**:
```json
{
  "tiers": [
    { "code": "bronze", "name": "Bronze", "min_spent": 0, "discount_percent": 0, "point_multiplier": 1 },
    { "code": "silver", "name": "Silver", "min_spent": 5000000, "discount_percent": 5, "point_multiplier": 1.2 },
    { "code": "gold", "name": "Gold", "min_spent": 15000000, "discount_percent": 10, "point_multiplier": 1.5 },
    { "code": "platinum", "name": "Platinum", "min_spent": 30000000, "discount_percent": 15, "point_multiplier": 2 }
  ]
}
```

**Benefits per tier**:
- Discount produk/layanan
- Point multiplier
- Free grooming per bulan
- Priority booking
- Free transport
- Birthday reward

### 11.2 Loyalty

**Earning**:
```
points_earned = floor(grand_total / point_ratio) × point_multiplier
```

**Redemption**:
```
value = points_redeemed × point_value
max_redemption = min(grand_total × max_redemption_percent, available_points × point_value)
```

**Rules**:
- Point expire setelah 12 bulan tidak aktif (configurable)
- Refund → point dikembalikan
- Point tidak bisa ditransfer antar customer

### 11.3 Promotion

**Types**:
1. **Product discount** — diskon per produk
2. **Cart discount** — diskon total belanja
3. **Voucher** — kode unik, limited use
4. **Bundle** — beli paket lebih murah
5. **BOGO** — buy 1 get 1
6. **Tier-based** — khusus membership tier
7. **Birthday** — otomatis di bulan ulang tahun
8. **Seasonal** — periode tertentu

**Combination rules** (configurable):
- Max 1 voucher per transaksi
- Voucher bisa digabung dengan diskon membership
- Diskon item + diskon cart + voucher = stackable (configurable)

---

# 📄 PETORA-BLUEPRINT-012
## Modul F — SDM

### 12.1 Employee Management

**Data karyawan**:
- Data pribadi (encrypted: KTP, bank)
- Data kepegawaian (jabatan, departemen, cabang, status)
- Data kontrak
- Data penggajian
- Dokumen (upload)

### 12.2 Attendance

**Clock in/out**:
- Via web (dengan geolocation validation)
- Via mobile (future)
- Via fingerprint/face (integration)

**Schedule**:
- Shift pattern (pagi/siang/malam)
- Weekly schedule
- Swap shift (butuh approval)

**Rekap**:
- Hadir, terlambat, izin, cuti, alpha
- Lembur
- Export untuk payroll

### 12.3 Commission

**Rules** (configurable per employee/service/product):
```json
{
  "rules": [
    {
      "type": "service",
      "service_id": "*",
      "staff_id": "groomer_x",
      "percent": 20
    },
    {
      "type": "product",
      "product_category": "premium_food",
      "staff_id": "*",
      "percent": 5
    }
  ]
}
```

**Calculation**:
```
commission = sum(item.total × commission_percent)
```

### 12.4 Payroll

**Components**:
- Gaji pokok
- Tunjangan (tetap, transport, makan)
- Lembur
- Komisi
- Bonus
- Potongan (kasbon, keterlambatan, absen)
- BPJS (jika ada)
- PPh 21

**Workflow**:
1. HR create payroll period (bulan X)
2. System auto-calculate dari attendance + commission
3. HR review & adjust manual
4. Approve oleh manager
5. Generate slip gaji
6. Disburse (manual atau integration)
7. Journal entry otomatis

---

# 📄 PETORA-BLUEPRINT-013
## Modul G — Keuangan & Akuntansi

### 13.1 Finance

**Cash management**:
- Kas per cabang
- Kas masuk (dari penjualan, dll)
- Kas keluar (biaya operasional, pembelian)
- Transfer antar kas
- Cash closing (akhir shift/hari)

**Receivable**:
- Piutang customer (kredit)
- Aging report
- Reminder jatuh tempo

**Payable**:
- Hutang supplier
- Aging report
- Payment scheduling

### 13.2 Accounting

**Chart of Accounts (default)**:
```
1-0000 ASET
  1-1000 Aset Lancar
    1-1001 Kas
    1-1002 Bank BCA
    1-1003 Piutang Usaha
    1-1004 Persediaan
  1-2000 Aset Tetap
    1-2001 Peralatan
    1-2002 Kendaraan
    1-2003 Akumulasi Penyusutan
2-0000 KEWAJIBAN
  2-1000 Kewajiban Lancar
    2-1001 Hutang Usaha
    2-1002 Pajak Terhutang
    2-1003 Uang Muka Pelanggan
3-0000 EKUITAS
  3-1001 Modal
  3-1002 Laba Ditahan
4-0000 PENDAPATAN
  4-1001 Penjualan Produk
  4-1002 Penjualan Layanan
  4-1003 Pendapatan Lain
5-0000 BEBAN
  5-1001 HPP
  5-1002 Gaji
  5-1003 Sewa
  5-1004 Utilitas
  5-1005 Pemasaran
  5-1006 Penyusutan
  5-1007 Beban Lain
```

**Auto journal entries**:
- Sales → Dr Kas/Piutang, Cr Penjualan, Cr Pajak
- Purchase → Dr Persediaan, Cr Hutang/Kas
- COGS → Dr HPP, Cr Persediaan
- Payroll → Dr Gaji, Cr Kas
- Adjustment → Dr/Loss atau Cr/Gain, Cr/Dr Persediaan

### 13.3 Reports

**Laporan utama**:
1. **Laba Rugi** — Revenue - COGS - Expenses = Net Income
2. **Neraca** — Assets = Liabilities + Equity
3. **Arus Kas** — Operating, Investing, Financing
4. **Penjualan** — By product, category, staff, customer
5. **Inventory** — Stock value, movement, expiry
6. **Piutang/Hutang** — Aging
7. **Pajak** — PPN keluaran & masukan
8. **Payroll** — Per employee, per period
9. **Commission** — Per staff, per period

---

# 📄 PETORA-BLUEPRINT-014
## Modul H — Komunikasi & Portal

### 14.1 Notification

**Channels** (configurable via integrations):
- WhatsApp (Fonnte / Wablas)
- Email (Resend / SMTP)
- SMS (Twilio / local)
- In-app notification

**Events**:
- Booking reminder (H-24, H-1)
- Booking confirmed
- Booking cancelled
- Pet hotel daily update
- Vaccination due
- Medical record finalized
- Prescription ready
- Transaction receipt
- Payment received
- Loyalty point earned
- Membership expiring
- Low stock alert (internal)
- Approval required (internal)

**Template system**:
```
Halo {{customer.name}},

Booking Anda untuk {{pet.name}} pada {{booking.date}} pukul {{booking.time}} 
telah dikonfirmasi.

Layanan: {{service.name}}
Staff: {{staff.name}}
Cabang: {{branch.name}}

Sampai jumpa!
{{organization.name}}
```

### 14.2 Customer Portal

**Fitur**:
- Login / register
- Profile management
- Pet management (add/edit)
- Booking (create, view, cancel)
- Transaction history
- Membership info
- Loyalty balance
- Medical records (yang diizinkan)
- Vaccination history
- Upload dokumen
- Notifications preferences

### 14.3 System Settings

**Konfigurasi global**:
- Identitas organisasi
- Default currency, timezone, language
- Pajak
- Harga (pembulatan, minimum)
- Inventory (negative stock, valuation)
- Loyalty (ratio, value, expiry)
- Document numbering format
- Operating hours
- Notification preferences

---

# 📄 PETORA-BLUEPRINT-015
## Modul I — Fitur Tambahan

### 15.1 CRM & Customer Relationship

**Fitur**:
- Segmentasi otomatis (by spending, frequency, tier)
- Tag customer
- Timeline interaksi
- Follow-up task
- Campaign (email/WA blast)
- Churn analysis
- Customer lifetime value

### 15.2 Complaint Management

**Workflow**:
1. Customer complain (via portal/WA/in-person)
2. CS create ticket
3. Assign to relevant person
4. Investigation
5. Resolution
6. Customer confirmation
7. Satisfaction survey
8. Report

**SLA** (configurable):
- Low priority: 72 jam
- Medium: 48 jam
- High: 24 jam
- Critical: 4 jam

### 15.3 Asset Management

**Data aset**:
- Kode aset
- Kategori (peralatan grooming, kendaraan, komputer, dll)
- Lokasi
- Penanggung jawab
- Nilai perolehan
- Tanggal perolehan
- Umur ekonomis
- Penyusutan (linear)
- Kondisi
- Jadwal maintenance
- Riwayat perbaikan

### 15.4 SOP & Checklist

**Template checklist**:
- Opening store
- Closing store
- Cleaning cage
- Grooming procedure
- Receiving goods
- Handover hewan

**Execution**:
- Staff execute → checklist terstamp waktu + foto
- Supervisor verify
- Report deviation

### 15.5 Document Management

**Kategori dokumen**:
- Dokumen supplier (kontrak, NPWP)
- Dokumen karyawan (KTP, kontrak)
- Dokumen pelanggan (vaksin, steril)
- Dokumen medis (sertifikat, lab)
- Invoice & bukti bayar
- Legal (SIUP, izin)

**Fitur**:
- Upload, categorize, tag
- Expiry reminder
- Access control
- Version history

### 15.6 Delivery Management

**Workflow**:
1. Customer pilih delivery saat checkout
2. System pilih kurir (internal / integration)
3. Generate resi
4. Kurir pick up
5. Tracking
6. POD (proof of delivery)
7. Complete

### 15.7 Budget & Expense Control

**Fitur**:
- Budget per cabang/departemen/akun
- Approval workflow
- Realisasi tracking
- Variance analysis
- Alert over-budget

### 15.8 Tax Management

**Fitur**:
- Konfigurasi tarif pajak (PPN, PPnBM)
- Mapping pajak per produk/layanan
- Rekap PPN keluaran (sales)
- Rekap PPN masukan (purchase)
- Generate form pajak
- Export untuk e-Faktur

### 15.9 Backup & Disaster Recovery

**Strategi**:
- Daily backup otomatis (Supabase managed)
- Weekly dump ke Cloudflare R2
- Monthly archive
- Test restore quarterly
- Retention: 30 harian, 12 mingguan, 12 bulanan

### 15.10 Online Store (E-Commerce)

**Fitur**:
- Katalog online (subset produk)
- Shopping cart
- Checkout
- Payment gateway integration
- Order management
- Shipping integration
- Stock sync dengan POS
- Customer account

---

# 📄 PETORA-BLUEPRINT-016
## Super Admin Panel & Konfigurasi

### 16.1 Tujuan Panel
Panel Super Admin adalah antarmuka khusus untuk mengelola konfigurasi sistem, integrasi, dan monitoring — **tanpa hardcode**. Semua perubahan langsung apply tanpa redeploy.

### 16.2 Akses
- Hanya role `super_admin`
- MFA wajib
- Audit log ketat

### 16.3 Fitur Panel

#### 16.3.1 Configuration Manager
**UI**:
```
┌────────────────────────────────────────────────────────────┐
│ Configuration Manager                                      │
├────────────────────────────────────────────────────────────┤
│ Search: [___________________]  [All ▼] [All Branches ▼]   │
├────────────────────────────────────────────────────────────┤
│ Key                              │ Value      │ Branch │ ⚙ │
│──────────────────────────────────┼────────────┼────────┼───│
│ loyalty.enabled                  │ true       │ Global │ ✎ │
│ loyalty.point_ratio              │ 1000       │ Global │ ✎ │
│ tax.default_rate                 │ 11         │ Global │ ✎ │
│ pos.require_customer             │ false      │ Global │ ✎ │
│ pos.receipt_footer               │ "Thanks"   │ Global │ ✎ │
│ notification.whatsapp_enabled    │ true       │ Global │ ✎ │
│ inventory.valuation_method       │ moving_avg │ Global │ ✎ │
└────────────────────────────────────────────────────────────┘
```

**Edit modal**:
- Input type auto-detect dari `value_type`
- Encrypted value: masked input
- History perubahan
- Rollback to previous value

#### 16.3.2 Integration Manager
**Supported integrations**:
| Code | Provider | Purpose |
|---|---|---|
| `whatsapp_fonnte` | Fonnte | WhatsApp messaging |
| `whatsapp_wablas` | Wablas | WhatsApp messaging |
| `email_resend` | Resend | Email sending |
| `email_smtp` | Custom SMTP | Email sending |
| `sms_twilio` | Twilio | SMS |
| `payment_midtrans` | Midtrans | Payment gateway |
| `payment_xendit` | Xendit | Payment gateway |
| `shipping_jne` | JNE | Shipping |
| `shipping_rajaongkir` | Raja Ongkir | Shipping cost |
| `accounting_onlinepajak` | Online Pajak | Tax reporting |
| `ecommerce_custom` | Custom API | Online store |

**UI per integration**:
- Enable/disable toggle
- Config form (API key, secret, webhook URL)
- Test connection button
- Last test status
- Logs

**Encrypted storage**:
```typescript
// Semua credentials dienkripsi AES-256-GCM
const ENCRYPTION_KEY = process.env.INTEGRATION_ENCRYPTION_KEY; // from Vault

async function encryptConfig(config: object): Promise<string> {
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const key = await crypto.subtle.importKey(...);
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    new TextEncoder().encode(JSON.stringify(config))
  );
  return btoa(String.fromCharCode(...iv)) + ':' + btoa(String.fromCharCode(...encrypted));
}
```

#### 16.3.3 Audit Log Viewer
- Filter by user, action, table, date range
- View before/after values
- Export to CSV
- Detail view

#### 16.3.4 System Health
- DB connection status
- Edge Function latency
- Storage usage
- Failed jobs
- Error rate

#### 16.3.5 Feature Flags
- Toggle features on/off
- Per-branch override
- Percentage rollout (future)

### 16.4 Konfigurasi via DB (tidak ada hardcode)

**Contoh penggunaan di Edge Function**:
```typescript
// ❌ BAD: Hardcode
const POINT_RATIO = 1000;
const TAX_RATE = 0.11;

// ✅ GOOD: From config
const config = await getConfig(supabase, ['loyalty.point_ratio', 'tax.default_rate']);
const POINT_RATIO = config['loyalty.point_ratio'];
const TAX_RATE = config['tax.default_rate'] / 100;
```

---

# 📄 PETORA-BLUEPRINT-017
## Security, Audit & Compliance

### 17.1 Authentication
- Supabase Auth (email/password, magic link)
- MFA TOTP wajib untuk Super Admin & Owner
- Session management (JWT, refresh token)
- Password policy: min 8 char, complexity, rotation 90 hari

### 17.2 Authorization
- RBAC via roles & permissions
- Branch-scoped data access via RLS
- Field-level encryption untuk PII
- Audit semua akses data sensitif

### 17.3 Data Encryption
**At rest**:
- Supabase managed TDE (database)
- App-level AES-256-GCM untuk:
  - Customer phone
  - Customer email
  - Employee ID number
  - Employee bank account
  - Integration credentials
  - Medical notes (optional)

**In transit**:
- HTTPS only (enforced by Vercel/Cloudflare)
- HSTS enabled
- TLS 1.3

### 17.4 Audit Trail
**Every mutation logged**:
- User ID
- Action (INSERT/UPDATE/DELETE)
- Table & record
- Before/after values
- Timestamp
- IP address
- User agent
- Branch

**Retention**:
- Audit log: 7 tahun (compliance)
- Archive ke cold storage setelah 1 tahun

### 17.5 Input Validation
- Zod schema di FE & BE
- SQL injection prevention (parameterized queries)
- XSS prevention (React default + CSP)
- CSRF protection (SameSite cookie)

### 17.6 Rate Limiting
- Per-IP: 100 req/min
- Per-user: 300 req/min
- Login: 5 attempts / 15 min
- OTP: 3 attempts / 10 min

### 17.7 Backup & Recovery
- Daily backup (Supabase managed)
- Weekly dump ke R2
- RPO: 24 jam
- RTO: 4 jam
- Test restore quarterly

### 17.8 Compliance Checklist
- [ ] PII encrypted at rest
- [ ] Audit log enabled
- [ ] MFA for admin
- [ ] Password policy enforced
- [ ] Session timeout (30 min idle)
- [ ] HTTPS only
- [ ] CSP headers
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] Backup tested
- [ ] Data retention policy
- [ ] Privacy policy published

---

# 📄 PETORA-BLUEPRINT-018
## Workflow End-to-End

### 18.1 Alur Penjualan Produk (POS)
```
[1] Customer datang
 ↓
[2] Kasir login → buka POS
 ↓
[3] Pilih customer (atau create new)
 ↓
[4] Scan/search produk → add to cart
 ↓
[5] Apply diskon/voucher/poin
 ↓
[6] Klik Pay → pilih payment method
 ↓
[7] Edge Function: pos-create-transaction
    ├─ Validate input
    ├─ Reserve stock
    ├─ Create sales_transaction
    ├─ Create sales_transaction_items
    ├─ Create sales_payments
    ├─ Commit stock (create stock_movements)
    ├─ Post journal (Dr Kas, Cr Penjualan, Cr Pajak)
    ├─ Add loyalty points
    └─ Emit event: sales.transaction.created
 ↓
[8] Print receipt
 ↓
[9] Kirim notifikasi (WA/email) ke customer
 ↓
[10] Dashboard terupdate real-time
```

### 18.2 Alur Grooming
```
[1] Customer booking (portal/staff/WA)
 ↓
[2] System cek jadwal groomer + kapasitas
 ↓
[3] Create booking (status: pending)
 ↓
[4] Staff confirm → status: confirmed
 ↓
[5] Reminder H-1 via WA
 ↓
[6] Customer datang → check-in
    ├─ Status: checked_in
    ├─ Foto kondisi awal
    └─ Checklist pemeriksaan
 ↓
[7] Groomer mulai → status: in_progress
    ├─ Tracking progress
    └─ Foto progress
 ↓
[8] Selesai → status: completed
    ├─ Foto hasil
    └─ Customer notified
 ↓
[9] Customer pick up + payment di POS
    ├─ Create sales_transaction (service)
    ├─ Commission recorded untuk groomer
    └─ Loyalty points added
 ↓
[10] Follow-up (optional) via WA
```

### 18.3 Alur Pembelian
```
[1] Inventory staff: low stock alert
 ↓
[2] Create purchase request
 ↓
[3] Manager approve
 ↓
[4] Create purchase order → send to supplier
 ↓
[5] Barang datang
 ↓
[6] Inventory staff: receive goods
    ├─ Check qty + quality
    ├─ Input batch + expiry
    ├─ Create stock_movement: purchase_in
    └─ Update stock_items
 ↓
[7] Create invoice matching
 ↓
[8] Finance: approve payment
 ↓
[9] Pay supplier
    ├─ Create journal: Dr Hutang, Cr Kas/Bank
    └─ Update payable
 ↓
[10] Close PO
```

### 18.4 Alur Konsultasi Dokter
```
[1] Customer booking konsultasi
 ↓
[2] Confirm → masuk antrian
 ↓
[3] Pasien datang → check-in
 ↓
[4] Dokter panggil
 ↓
[5] Anamnesa + pemeriksaan fisik
 ↓
[6] Diagnosis + tindakan
 ↓
[7] Buat resep (jika perlu)
 ↓
[8] Finalisasi rekam medis
    ├─ Status: finalized
    ├─ Snapshot tersimpan
    └─ Emit event: medical_record.finalized
 ↓
[9] Patient ke farmasi
    ├─ Pharmacist terima resep
    ├─ Dispense obat (reduce stock)
    └─ Print label
 ↓
[10] Payment
 ↓
[11] Notifikasi ke customer: hasil konsultasi (via portal)
```

### 18.5 Alur Pet Hotel
```
[1] Booking kamar (pilih kamar, tanggal)
 ↓
[2] Confirm → status: reserved
 ↓
[3] Check-in
    ├─ Verifikasi booking
    ├─ Foto kondisi awal
    ├─ Input feeding/medication schedule
    ├─ Status: active
    └─ Room status: occupied
 ↓
[4] Daily care
    ├─ Log makan/minum
    ├─ Log buang air
    ├─ Foto perkembangan
    └─ Kirim update ke owner (WA)
 ↓
[5] Check-out
    ├─ Final condition check
    ├─ Calculate total (daily rate × days + additional)
    ├─ Payment
    ├─ Room status: cleaning → available
    └─ Foto serah terima
 ↓
[6] Follow-up
```

### 18.6 Alur Payroll
```
[1] HR: buka periode payroll (bulan X)
 ↓
[2] System auto-calculate:
    ├─ Gaji pokok
    ├─ Attendance (hadir, terlambat, alpha)
    ├─ Lembur
    ├─ Komisi (dari transaksi)
    ├─ Tunjangan
    └─ Potongan
 ↓
[3] HR review + adjust manual
 ↓
[4] Manager approve
 ↓
[5] Generate slip gaji (PDF)
 ↓
[6] Disburse (manual/transfer)
 ↓
[7] Journal entry otomatis:
    ├─ Dr Beban Gaji
    ├─ Dr Beban Komisi
    ├─ Cr Kas/Bank
    └─ Cr BPJS (jika ada)
 ↓
[8] Period closed
```

---

# 📄 PETORA-BLUEPRINT-019
## Deployment & Infrastructure

### 19.1 Environment

**Environments**:
- `local` — development
- `staging` — testing
- `production` — live

### 19.2 Vercel (Frontend)

**vercel.json** (web-staff):
```json
{
  "framework": "vite",
  "buildCommand": "turbo run build --filter=web-staff",
  "outputDirectory": "apps/web-staff/dist",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        { "key": "Content-Security-Policy", "value": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://*.supabase.co wss://*.supabase.co;" }
      ]
    }
  ]
}
```

### 19.3 Supabase (Backend)

**supabase/config.toml**:
```toml
[api]
enabled = true
port = 54321

[db]
port = 54322

[studio]
enabled = true
port = 54323

[auth]
enabled = true
site_url = "https://staff.petora.app"
additional_redirect_urls = ["https://pos.petora.app", "https://customer.petora.app"]
jwt_expiry = 3600
enable_refresh_token_rotation = true
refresh_token_reuse_interval = 10
enable_signup = true
enable_anonymous_sign_ins = false

[auth.email]
enable_signup = true
double_confirm_changes = true
enable_confirmations = true

[auth.mfa]
enable_totp = true

[storage]
enabled = true
file_size_limit = "50MB"

[realtime]
enabled = true

[edge_runtime]
policy = "per_worker"
```

### 19.4 Environment Variables

**Vercel**:
```
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
VITE_ENVIRONMENT=production
```

**Supabase Edge Functions**:
```
SUPABASE_URL (auto)
SUPabase_SERVICE_ROLE_KEY (auto)
INTEGRATION_ENCRYPTION_KEY=xxx (from Vault)
SENTRY_DSN=xxx
```

### 19.5 CI/CD Pipeline

**`.github/workflows/ci.yml`**:
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test
      - run: npm run test:e2e
  
  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - run: supabase link --project-ref $STAGING_PROJECT_REF
      - run: supabase db push
      - run: supabase functions deploy
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID_STAGING }}
  
  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - run: supabase link --project-ref $PROD_PROJECT_REF
      - run: supabase db push
      - run: supabase functions deploy
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID_PROD }}
          vercel-args: '--prod'
```

### 19.6 Domain & DNS

```
petora.app                    → Cloudflare (DNS)
├── staff.petora.app          → Vercel (web-staff)
├── pos.petora.app            → Vercel (web-pos)
├── customer.petora.app       → Vercel (web-customer)
├── api.petora.app            → Supabase (custom domain)
└── storage.petora.app        → Supabase Storage
```

### 19.7 Monitoring

**Sentry** (FE):
- Error tracking
- Performance monitoring
- Release tracking

**Supabase Dashboard** (BE):
- Query performance
- API usage
- Auth events
- Storage usage

**Custom health check endpoint**:
```
GET /health
→ { status: 'ok', db: 'ok', edge: 'ok', timestamp: '...' }
```

---

# 📄 PETORA-BLUEPRINT-020
## Testing, QA & Acceptance

### 20.1 Test Strategy

**Pyramid**:
```
        ╱╲
       ╱ E2E╲         Playwright (critical flows)
      ╱──────╲
     ╱ Integra╲       Vitest + Supabase local
    ╱──────────╲
   ╱   Unit     ╲     Vitest (functions, utils, hooks)
  ╱──────────────╲
```

### 20.2 Unit Tests

**Coverage target**: 80% untuk business logic

**Contoh**:
```typescript
// packages/utils/src/__tests__/calculateTotals.test.ts
describe('calculateTotals', () => {
  it('should apply item discount correctly', () => {
    const items = [{ price: 100000, qty: 2, discount: 10000 }];
    const result = calculateTotals(items, { taxRate: 0.11 });
    expect(result.subtotal).toBe(200000);
    expect(result.discountItem).toBe(10000);
    expect(result.taxableAmount).toBe(190000);
    expect(result.taxAmount).toBe(20900);
  });
  
  it('should reject negative quantities', () => {
    expect(() => calculateTotals([{ qty: -1 }])).toThrow();
  });
});
```

### 20.3 Integration Tests

**Setup**: Supabase local (Docker) + seed data

**Contoh**:
```typescript
// supabase/functions/__tests__/pos-create-transaction.test.ts
describe('pos-create-transaction', () => {
  beforeEach(async () => {
    await resetDatabase();
    await seedTestData();
  });
  
  it('should create transaction and reduce stock', async () => {
    const initialStock = await getStock('product_1');
    
    const response = await callFunction('pos-create-transaction', {
      items: [{ product_id: 'product_1', qty: 2 }],
      payments: [{ method: 'cash', amount: 200000 }],
    });
    
    expect(response.status).toBe(200);
    const finalStock = await getStock('product_1');
    expect(finalStock.quantity).toBe(initialStock.quantity - 2);
    
    const movement = await getStockMovement({ reference_id: response.data.id });
    expect(movement.movement_type).toBe('sales_out');
  });
  
  it('should reject when stock insufficient', async () => {
    const response = await callFunction('pos-create-transaction', {
      items: [{ product_id: 'product_1', qty: 9999 }],
    });
    expect(response.status).toBe(422);
    expect(response.error.code).toBe('STOCK_INSUFFICIENT');
  });
});
```

### 20.4 E2E Tests

**Critical flows** (Playwright):
1. Login → Dashboard
2. POS: Create transaction → Receipt printed
3. Booking: Create → Confirm → Check-in → Complete
4. Vet: Create medical record → Finalize → Dispense
5. Purchase: Create PO → Receive → Pay
6. Payroll: Calculate → Approve

**Contoh**:
```typescript
// e2e/pos-transaction.spec.ts
test('complete POS transaction flow', async ({ page }) => {
  await page.goto('/pos');
  await page.fill('[data-testid=email]', 'cashier@test.com');
  await page.fill('[data-testid=password]', 'password');
  await page.click('[data-testid=login-button]');
  
  // Add product
  await page.click('[data-testid=product-grid] >> text=Royal Canin');
  await expect(page.locator('[data-testid=cart]')).toContainText('Royal Canin');
  
  // Pay
  await page.click('[data-testid=pay-button]');
  await page.click('[data-testid=payment-cash]');
  await page.fill('[data-testid=cash-amount]', '200000');
  await page.click('[data-testid=process-payment]');
  
  // Verify
  await expect(page.locator('[data-testid=success-modal]')).toBeVisible();
  await expect(page.locator('[data-testid=receipt]')).toContainText('TRX/');
});
```

### 20.5 Acceptance Criteria

**Definition of Done per feature**:
- [ ] Code reviewed
- [ ] Unit tests pass (80% coverage)
- [ ] Integration tests pass
- [ ] E2E tests pass (for critical flows)
- [ ] Manual QA on staging
- [ ] Performance tested (response < 500ms p95)
- [ ] Security checklist verified
- [ ] Documentation updated
- [ ] Audit log verified
- [ ] RLS policies verified
- [ ] Config-driven (no hardcode)

### 20.6 Performance Targets

| Metric | Target |
|---|---|
| FE First Contentful Paint | < 1.5s |
| FE Time to Interactive | < 3s |
| API response (p95) | < 500ms |
| API response (p99) | < 1500ms |
| DB query (p95) | < 100ms |
| Concurrent users | 500 per branch |
| Transactions per day | 10,000 per branch |

### 20.7 Load Testing

**Scenario**:
- 100 kasir concurrent di 10 cabang
- 1000 transaksi/jam
- 5000 produk in catalog
- 10,000 customers

**Tool**: k6 / Artillery

---

# 📄 PETORA-BLUEPRINT-021
## Glossary & Appendix

### 21.1 Glossary

| Istilah | Definisi |
|---|---|
| **SKU** | Stock Keeping Unit — kode unik produk |
| **FEFO** | First Expired, First Out — metode pengeluaran stok berdasarkan kedaluwarsa |
| **FIFO** | First In, First Out — metode pengeluaran stok berdasarkan tanggal masuk |
| **HPP** | Harga Pokok Penjualan — COGS |
| **RLS** | Row Level Security — keamanan level baris di Postgres |
| **PII** | Personally Identifiable Information |
| **PO** | Purchase Order |
| **DO** | Delivery Order |
| **SO** | Sales Order |
| **GR** | Goods Receipt |
| **OP** | Stock Opname |
| **DOA** | Date of Arrival |
| **ED** | Expiry Date |
| **MOQ** | Minimum Order Quantity |
| **ROP** | Reorder Point |
| **TDE** | Transparent Data Encryption |
| **MFA** | Multi-Factor Authentication |
| **RBAC** | Role-Based Access Control |
| **ADR** | Architecture Decision Record |
| **RPO** | Recovery Point Objective |
| **RTO** | Recovery Time Objective |
| **SLA** | Service Level Agreement |

### 21.2 Referensi

- [Supabase Docs](https://supabase.com/docs)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [React Docs](https://react.dev/)
- [TanStack Query](https://tanstack.com/query)
- [shadcn/ui](https://ui.shadcn.com/)
- [Zod](https://zod.dev/)

### 21.3 Changelog Blueprint

| Versi | Tanggal | Perubahan |
|---|---|---|
| 1.0.0 | 07 Nov 2026 | Initial final blueprint |

---

# ✅ CHECKLIST IMPLEMENTASI

Sebelum deployment production, pastikan semua berikut terpenuhi:

## Infrastructure
- [ ] Supabase project created (production)
- [ ] Vercel project created (3 apps)
- [ ] Domain configured di Cloudflare
- [ ] SSL certificate active
- [ ] Environment variables set
- [ ] Secrets in Vault

## Database
- [ ] All migrations applied
- [ ] RLS policies enabled on all tables
- [ ] Indexes created
- [ ] Triggers attached
- [ ] Seed data (default roles, permissions, COA, config)
- [ ] Backup schedule configured
- [ ] Test restore performed

## Backend
- [ ] All Edge Functions deployed
- [ ] Integration credentials encrypted
- [ ] Cron jobs scheduled
- [ ] Error tracking (Sentry) connected
- [ ] Health check endpoint working

## Frontend
- [ ] All 3 apps deployed
- [ ] CSP headers configured
- [ ] Source maps uploaded to Sentry
- [ ] PWA manifest (optional)

## Security
- [ ] Penetration test performed
- [ ] MFA enabled for admin
- [ ] Audit log verified
- [ ] PII encrypted
- [ ] Rate limiting active
- [ ] HTTPS enforced

## Business
- [ ] Default config populated
- [ ] Default roles & permissions
- [ ] Default COA
- [ ] Default tax profile
- [ ] Default document numbering
- [ ] Test transaction flow end-to-end

## Documentation
- [ ] User manual (staff)
- [ ] User manual (customer)
- [ ] Admin guide
- [ ] API documentation
- [ ] Troubleshooting guide

---

# 🎯 PENUTUP

Dokumen **PETORA-BLUEPRINT v1.0.0** ini adalah **acuan final** pengembangan sistem PETORA. Semua keputusan teknis, bisnis, dan arsitektur telah ditetapkan. Tidak ada roadmap bertahap — semua spesifikasi di sini harus diimplementasikan untuk mencapai produk yang **production-ready**.

**Prinsip utama**:
1. ✅ **Single-tenant, multi-branch** — bukan SaaS multi-tenant
2. ✅ **Modular Monolith** di Supabase Edge Functions
3. ✅ **Zero hardcode** — semua config via Super Admin panel
4. ✅ **Type-safe end-to-end** — Zod schemas shared FE+BE
5. ✅ **Event-driven internal** — konsistensi antar-modul
6. ✅ **Audit everything** — traceability penuh
7. ✅ **RLS-based security** — data isolation per branch
8. ✅ **Idempotency first** — no duplicate transactions
9. ✅ **Offline-first (POS)** — best effort queue
10. ✅ **Configurable** — semua angka bisnis bisa diubah tanpa redeploy

**PETORA — One Platform, Complete Pet Care Management.**

---

**Dokumen ini disusun sebagai blueprint final.**  
**Versi: 1.0.0 | Tanggal: 07 November 2026**  
**Status: FINAL — Ready for Implementation**
