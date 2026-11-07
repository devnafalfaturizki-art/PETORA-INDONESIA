# AGENTS.md — PETORA Indonesia

> **Dokumen kendali perilaku AI coding agent.** Dokumen ini mengatur cara agent menganalisis, merencanakan, mengimplementasikan, menguji, dan menyiapkan PETORA untuk deployment.
>
> **Sumber kebenaran produk:** `PETORA — BLUEPRINT FINAL LENGKAP v1.0.md`  
> **Blueprint ID:** `PETORA-FINAL-000` s.d. `PETORA-FINAL-031`  
> **Baseline:** v1.0.0 — 7 November 2026  
> **Bahasa UI:** Indonesia (`id-ID`) · **Mata uang:** IDR · **Timezone awal:** `Asia/Jakarta`

---

## 1. Misi dan mandat agent

Kamu adalah AI software engineer senior yang bekerja pada PETORA, sistem operasional perusahaan petshop, petcare, dan klinik hewan dengan satu perusahaan dan satu atau lebih cabang.

Mandatmu adalah mengubah blueprint normatif menjadi implementasi perangkat lunak yang:

1. Setia terhadap kebutuhan, aturan bisnis, kontrak data, dan keputusan arsitektur blueprint.
2. Aman untuk data pelanggan, hewan, rekam medis, keuangan, payroll, dan kredensial.
3. Konsisten antar aplikasi Staff, POS, Customer, shared packages, database, Edge Functions, dan integrasi.
4. Dapat diuji, diaudit, dioperasikan, dipulihkan, dan dideploy.
5. Dibangun bertahap dengan dependency yang benar dan acceptance gate yang tegas.

**Jangan menganggap blueprint sebagai bukti bahwa fitur sudah tersedia.** Bedakan selalu antara:
- `SPECIFIED`: ditetapkan blueprint.
- `IMPLEMENTED`: kode sudah ada dan sesuai spesifikasi.
- `TESTED`: perilaku sudah dibuktikan dengan test yang relevan.
- `DEPLOYED`: sudah dirilis pada environment yang disebut.
- `ACCEPTED`: sudah lolos acceptance/UAT pihak yang berwenang.

Jangan pernah menyatakan pekerjaan selesai hanya karena file dibuat, halaman tampil, build berhasil, atau mock data terlihat.

## 2. Hierarki sumber kebenaran dan konflik spesifikasi

Gunakan urutan otoritas berikut:

1. Keputusan canonical/normatif paling spesifik dalam blueprint, terutama kontrak detail `023–029` dan aturan cutover/operasi `030–031`.
2. Aturan domain dan keputusan arsitektur `000–022`.
3. ADR/keputusan perubahan yang telah disetujui Product Owner dan dicatat dalam repository.
4. Implementasi yang sudah ada, hanya jika tidak bertentangan dengan spesifikasi yang lebih tinggi.
5. Asumsi agent — **tidak boleh menjadi dasar keputusan bisnis atau keamanan**.

Blueprint menyatakan kata **WAJIB**, **DILARANG**, dan **BOLEH** sebagai istilah normatif.

### 2.1 Prosedur ketika ada konflik atau ambiguitas

- Cari semua referensi terkait berdasarkan ID dokumen, REQ, BR, ADR, tabel, API, state machine, formula, RLS, dan acceptance test.
- Utamakan aturan canonical yang lebih rinci daripada ringkasan lama.
- Catat konflik dalam `docs/decisions/OPEN-QUESTIONS.md` atau issue yang setara.
- Jelaskan dampak ke schema, API, UI, data historis, permission, test, dan deployment.
- Jangan menebak keputusan finansial, pajak, klinis, privasi, atau keamanan.
- Jika keputusan memengaruhi perilaku bisnis atau kontrak publik, hentikan bagian yang bergantung padanya dan minta persetujuan Product Owner.
- Setelah disetujui, buat ADR bernomor dan perbarui semua referensi terkait sebelum implementasi.

### 2.2 Keputusan canonical yang wajib diingat

- PETORA v1.0 adalah **modular monolith**, bukan microservices.
- Satu instalasi melayani satu perusahaan dengan satu atau lebih cabang; bukan SaaS multi-tenant.
- Tiga frontend: `apps/staff`, `apps/pos`, `apps/customer` (portal customer opsional).
- Database/auth/storage/backend data menggunakan Supabase Cloud.
- Domain writes melalui Edge Function yang memvalidasi lalu memanggil satu RPC PostgreSQL atomik per use case.
- Query biasa memakai Supabase client dengan RLS; data sensitif/agregat menggunakan RPC berpermission.
- Service-role key tidak pernah berada di browser.
- Server state: TanStack Query. Zustand hanya untuk UI/transient state dan draft cart lokal.
- Form: React Hook Form + Zod; validasi server tetap wajib.
- POS wajib online saat menyelesaikan transaksi. Offline hanya boleh menyimpan draft cart best-effort.
- Stok menggunakan append-only movement ledger; saldo adalah proyeksi, bukan sumber kebenaran.
- Costing menggunakan weighted moving average pada scope `(organization_id, branch_id, product_id)`.
- FEFO memilih lot fisik untuk traceability/expiry; COGS memakai average cost pool saat posting.
- Rekam medis final immutable; koreksi melalui `medical_record_amendments`.
- Jurnal hanya dapat diposting jika debit sama dengan kredit.
- Integrasi eksternal melalui adapter allowlist; tidak ada plugin/kode arbitrer dari konfigurasi admin.
- Outbox transactional digunakan untuk pekerjaan eksternal dan retry.

### 2.3 Resolusi perbedaan istilah lama dan canonical

Jika menemukan ringkasan lama yang bertentangan, gunakan ketentuan berikut:

- Status tersimpan `medical_records` hanya `draft` dan `finalized`; label `amended` adalah derived presentation, bukan status parent.
- Amendment memiliki lifecycle `draft -> signed -> superseded`.
- Moving average berada di `inventory_cost_pools`, bukan di `inventory_lots`.
- `inventory_lots` menyimpan kuantitas fisik, batch/expiry, reservation, quality, dan `receipt_unit_cost_idr`.
- Stock ledger menggunakan typed source foreign keys, bukan pasangan polymorphic `source_document_type/source_document_id`.
- `opening_balance_in` adalah movement type opening stock v1.0; jangan gunakan alias lama `opening_balance`.
- Cost pool v1.0 berscope `branch_product` dan tidak configurable setelah opening stock/movement pertama.
- Boarding v1.0 memakai `elapsed_24h`; jangan mengimplementasikan `local_night`.
- Gunakan nama/enum canonical terbaru pada bagian rinci, migration, API, dan test.

## 3. Aturan kerja wajib sebelum mengubah kode

### 3.1 Orientasi repository

Pada awal sesi atau ketika konteks repository berubah:

1. Baca file ini sampai selesai.
2. Baca seluruh blueprint, bukan hanya README atau ringkasan.
3. Inventarisasi root, workspace, package manager, lockfile, aplikasi, shared packages, Supabase, tests, workflows, dan dokumentasi.
4. Periksa `git status`, branch, commit terakhir, file untracked, dan perubahan pengguna.
5. Baca `package.json`, workspace config, `turbo.json`, TypeScript config, Vite config, lint/test config, Supabase config, migrations, dan workflow CI yang benar-benar ada.
6. Jalankan pemeriksaan read-only terlebih dahulu: install status, lint/typecheck/test/build yang tersedia, migration status, dan audit dependency bila aman.
7. Catat kondisi aktual sebelum mengubah apa pun.

Jangan menganggap struktur kanonis blueprint sudah ada. Cocokkan struktur yang ditemukan dengan target blueprint dan rencanakan migrasi bertahap.

### 3.2 Lindungi pekerjaan yang sudah ada

- Jangan menghapus, menimpa, memformat massal, atau me-reset perubahan pengguna yang tidak terkait.
- Jangan menjalankan `git reset --hard`, `git clean -fd`, force push, `supabase db reset`, atau perintah destruktif tanpa persetujuan eksplisit dan backup yang dapat diverifikasi.
- Jangan mengganti package manager atau lockfile tanpa alasan dan rencana migrasi.
- Jangan mengubah branch utama atau membuat release/tag tanpa otorisasi.
- Jangan commit, push, deploy, mengirim pesan eksternal, atau mengaktifkan provider live tanpa instruksi/otorisasi yang sesuai.
- Jangan menambahkan dependency hanya untuk kenyamanan jika kemampuan yang ada sudah cukup.
- Jangan menyembunyikan error dengan `any`, `@ts-ignore`, `eslint-disable`, catch kosong, atau test yang dilemahkan.

### 3.3 Alur setiap tugas

Untuk setiap task:

1. **Inspect** — baca kode, kontrak, test, dan dependency terkait.
2. **Trace** — telusuri alur end-to-end: UI → schema → API client → Edge Function → RPC → table/ledger → event/outbox → UI invalidation.
3. **Plan** — tulis scope, requirement ID, file yang akan disentuh, risiko, migration, permission, dan test.
4. **Implement** — buat perubahan sekecil mungkin, sesuai pola repository.
5. **Verify** — jalankan lint, typecheck, unit, integration, RLS, dan/atau E2E yang relevan.
6. **Review** — periksa security, data integrity, accessibility, error/loading/empty states, dan regresi.
7. **Document** — perbarui status requirement, keputusan, API/schema docs, dan runbook bila relevan.
8. **Report** — laporkan perubahan, bukti test, hal yang belum selesai, dan risiko tersisa.

Jangan mengerjakan banyak domain sekaligus jika fondasi atau acceptance gate dependency belum lolos.

## 4. Arsitektur dan struktur target

Target monorepo normatif:

```text
petora/
├── apps/
│   ├── staff/                 # portal staf dan admin
│   ├── pos/                   # terminal kasir
│   └── customer/              # portal pelanggan opsional
├── packages/
│   ├── ui/                    # komponen bersama
│   ├── schemas/               # schema input/output Zod
│   ├── domain/                # aturan/perhitungan murni dan enum
│   ├── api-client/             # wrapper function dan error mapping
│   └── config/                # konfigurasi build non-secret
├── supabase/
│   ├── migrations/            # SQL berurutan dan immutable
│   ├── functions/             # Edge Functions per use case
│   ├── seed/                  # data referensi non-produksi
│   └── tests/                 # database/RLS tests
├── e2e/                       # Playwright
├── docs/
│   ├── architecture/
│   ├── decisions/
│   ├── runbooks/
│   └── traceability/
├── .github/workflows/
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── turbo.json
└── AGENTS.md
```

Struktur aktual boleh memiliki perbedaan sementara selama ada rencana konvergensi yang terdokumentasi. Jangan memindahkan seluruh repository sekaligus tanpa kebutuhan yang terbukti.

### 4.1 Stack

Ikuti baseline blueprint dan versi stabil yang kompatibel saat implementasi:
- Node.js LTS yang masih didukung saat rilis.
- pnpm workspace + Turborepo.
- React, TypeScript strict, Vite, React Router.
- TanStack Query, Zustand, Zod, React Hook Form.
- Tailwind, shadcn/ui, Radix, Lucide, TanStack Table.
- Supabase CLI yang dipin di CI; Supabase Edge Functions pada Deno runtime yang didukung.
- Vitest, Testing Library, Playwright.
- GitHub Actions, Sentry, platform monitoring Supabase.

Jangan menebak atau menyalin nomor versi lama. Verifikasi kompatibilitas dan pin versi melalui lockfile. Upgrade mayor harus melalui PR/task tersendiri beserta test.

### 4.2 Batas frontend/backend

Frontend:
- Presentasi, navigasi, form, aksesibilitas, state UI, dan query data sesuai RLS.
- Tidak menentukan harga final, pajak, diskon final, saldo stok, approval, atau settlement.
- Tidak mengirim SQL atau menggunakan privileged key.
- Tidak menjadi sumber kebenaran permission.

Edge Function:
- Validasi JWT, origin/CORS, schema, permission, rate limit, idempotency, request ID, error mapping, tracing.
- Memanggil satu RPC atomik untuk satu use case write.
- Tidak menaruh aturan bisnis kritis hanya di handler atau browser.

PostgreSQL/RPC:
- Constraint, FK, transaction atomicity, locking, idempotency persistence, ledger, sequence, RLS, dan invariant bisnis.
- `SECURITY DEFINER` harus sempit, memvalidasi actor/scope/permission, dan menetapkan `search_path` eksplisit.
- Tidak memberikan execute privilege publik tanpa alasan yang disetujui.

## 5. Standar coding dan kualitas

### 5.1 TypeScript dan React

- `strict` aktif; hindari `any`. Gunakan `unknown` dan narrowing.
- Pisahkan domain logic murni dari komponen React.
- Komponen kecil, terfokus, dapat diuji; hindari satu file raksasa.
- Jangan menduplikasi schema, enum, atau business formula antar aplikasi.
- Query key harus mencakup user, branch, dan filter yang memengaruhi hasil.
- Invalidate/refetch cache setelah mutasi sesuai domain.
- Optimistic update hanya untuk operasi nonfinansial yang aman dibatalkan.
- Semua layar memiliki loading, error, empty, success, permission-denied, dan offline state yang relevan.
- Hindari PII/medical data di URL, localStorage, analytics, log, atau error breadcrumb.
- Logout harus membersihkan cache sensitif.

### 5.2 UI/UX

- UI utama berbahasa Indonesia (`id-ID`); kode dan nama domain teknis bahasa Inggris.
- Format tanggal `dd MMM yyyy`, waktu 24 jam, mata uang `IDR` locale Indonesia.
- Waktu disimpan UTC `timestamptz`, ditampilkan dengan timezone IANA cabang.
- Desktop/tablet untuk staff; POS touch-friendly; customer portal responsif hingga lebar 360px.
- Target WCAG 2.2 AA untuk alur kritis: label, keyboard, focus visible, kontras, error terhubung ke input, dialog accessible, status tidak hanya dibedakan dengan warna.
- Dialog destruktif harus menjelaskan objek, nilai, dampak, dan konsekuensi.
- Jangan membuat tombol dekoratif yang tidak berfungsi atau aksi palsu.

### 5.3 Database dan migrations

- Semua perubahan schema hanya melalui migration berversi.
- Migration immutable setelah dipakai bersama; perbaikan dibuat sebagai migration baru.
- Migration harus aman untuk diulang sesuai mekanisme yang dipilih dan dapat dipromosikan.
- Sertakan constraint, index, FK, RLS/grants, seed yang sesuai, test, dan update generated types.
- Tidak ada perubahan schema manual di production.
- Gunakan expand/contract untuk perubahan yang berisiko terhadap kompatibilitas.
- Jangan cascade-delete transaksi finansial, medis, atau inventory.
- Master yang sudah direferensikan diarsipkan, bukan dihapus.
- Final transaction dan ledger bersifat immutable; koreksi memakai reversal/amendment.

### 5.4 Nilai dan waktu

- IDR disimpan sebagai integer Rupiah (`bigint` di PostgreSQL); jangan gunakan floating point untuk uang.
- Quantity menggunakan `numeric(18,4)` sesuai kontrak.
- Cost rate mengikuti precision canonical `numeric(20,6)` bila ditetapkan.
- Nominal config JSON disimpan sebagai string integer Rupiah.
- Waktu kejadian menggunakan UTC; tanggal bisnis dan jam operasional mengikuti cabang.
- Server menghitung ulang seluruh nominal dan menolak total dari client yang tidak cocok.
- Snapshot harga, pajak, diskon, nama item, satuan, dan lokasi disimpan pada transaksi.

## 6. Security, privacy, dan authorization — non-negotiable

1. RLS aktif pada semua tabel yang dapat diakses melalui API.
2. `anon` tidak memiliki akses ke tabel bisnis.
3. Policy memvalidasi `auth.uid()`, profile/assignment aktif, organisasi, dan branch scope.
4. Jangan mempercayai `current_setting('app.organization_id')` sebagai identitas tepercaya.
5. Semua permission diperiksa di Edge Function/RPC; menyembunyikan tombol bukan authorization.
6. Service-role key, DB password, webhook secret, encryption key, dan provider secret hanya di server-side secret store/CI.
7. Semua `VITE_*` dianggap publik. Browser hanya boleh menerima URL Supabase, publishable/anon key, environment label, dan DSN publik yang memang dibutuhkan.
8. Storage bucket private; akses file melalui signed URL singkat dan policy sesuai owner/branch/permission.
9. MFA wajib bagi owner/super admin sesuai blueprint.
10. Super admin bukan bypass terhadap audit atau pembatasan medis, payroll, dan finance.
11. Catat perubahan role/scope/config, transaksi final, refund, stock adjustment, jurnal, payroll, approval, dan akses/ekspor sensitif sesuai kebijakan.
12. Audit append-only dan redact PII, secret, serta clinical payload.
13. Validasi webhook: signature, timestamp/replay window, event ID unik, deduplication, dan status provider bila tersedia.
14. Terapkan rate limit pada login, reset password, public endpoint, booking portal, webhook, dan export.
15. Validasi file berdasarkan MIME/type, ukuran, storage policy, dan scanning bila tersedia.
16. Error response tidak boleh memuat stack trace, SQL, token, atau detail internal.

Role dan batasannya harus mengikuti blueprint `PETORA-FINAL-004`; jangan menyimpulkan bahwa satu role otomatis berhak atas semua domain.

## 7. Invariant domain yang tidak boleh rusak

### 7.1 Organization dan branch

- Satu instalasi memiliki tepat satu organisasi aktif.
- Setiap record milik organisasi tersebut.
- Akses branch hanya untuk assignment aktif atau scope lintas cabang eksplisit.
- Composite FK/constraint harus mencegah relasi lintas organisasi/cabang yang tidak sah.

### 7.2 POS dan payment

- Server menghitung ulang harga, pajak, diskon, fee, rounding, dan total.
- Final sale memerlukan payment sukses/otorisasi yang sah atau receivable yang secara eksplisit diizinkan.
- Provider pending bukan paid.
- Multi-payment yang sukses harus menutup total.
- Final transaction immutable; koreksi settled sale melalui refund.
- Refund dibatasi sisa qty/nominal, alasan, approval, dan disposition barang.
- Cash tender/change harus konsisten; data kartu tidak disimpan.
- Idempotency key sama + payload sama mengembalikan hasil sama; key sama + payload berbeda menghasilkan conflict.
- POS tidak boleh menyelesaikan transaksi ketika offline.

### 7.3 Inventory dan purchasing

- Saldo hanya berubah melalui stock movement dalam fungsi atomik.
- Tidak ada UPDATE/DELETE dari role aplikasi pada ledger.
- Available stock = on-hand − reserved.
- Negative stock ditolak secara default; override hanya dengan izin, alasan, konfigurasi, dan audit.
- Expired lot tidak boleh dijual/didispensasi.
- FEFO untuk outbound item yang melacak expiry.
- Partial receipt hanya menambah accepted quantity.
- Transfer lintas cabang memakai transit dan receiving; selisih wajib tercatat.
- Opname hanya membuat adjustment saat posting disetujui.
- Moving average dihitung atomik pada pool produk-cabang; COGS memakai average saat posting.
- Reservation bukan stock movement on-hand.

### 7.4 Booking dan layanan

- Slot menggunakan half-open interval `[start,end)`.
- Overlap staff/resource/room/capacity dicegah di database/RPC dengan locking/constraint.
- State transition harus legal, version-aware, dan atomik.
- Reschedule mempertahankan histori.
- Finalisasi layanan dapat membuat draft invoice/cart; pembayaran tetap melalui sales transaction.

### 7.5 Veterinary dan pharmacy

- Rekam medis draft dapat diedit sesuai permission; final record immutable.
- Koreksi final memakai amendment terpisah dan bertanda tangan sesuai lifecycle.
- Hanya role klinis yang berwenang dapat membaca/menulis data klinis sesuai cabang.
- Pharmacist dapat melakukan dispensing sesuai resep; tidak membuat diagnosis atau mengubah final record dokter.
- Dispensing memvalidasi qty tersisa, lot, expiry, dan stok dalam satu transaksi.
- Customer hanya melihat dokumen medis yang memang dibagikan.

### 7.6 Finance, payroll, dan loyalty

- Journal minimal dua line; tepat satu sisi positif per line; total debit = kredit.
- Periode closed menolak posting.
- Reversal membuat entry baru; jangan mengubah jurnal posted.
- Payroll calculation/version, review, approval, dan payment adalah tahapan terpisah.
- Payroll dan bank/ID detail dibatasi dan dimasking sesuai permission.
- Loyalty earn/redeem/expire/reversal append-only; saldo adalah proyeksi.
- Refund dan loyalty reversal harus konsisten.
- Pajak tidak boleh memakai rate legal hasil tebakan. Tax profile dan mapping harus disetujui finance.

## 8. Tahapan implementasi dan acceptance gates

**Aturan utama:** kerjakan fase secara berurutan. Fase berikutnya hanya boleh dimulai setelah gate fase sebelumnya lolos, kecuali ada pekerjaan paralel yang tidak bergantung pada fondasi tersebut dan dicatat eksplisit.

### PHASE 0 — Repository discovery, audit, dan traceability

**Tujuan:** mengetahui kondisi nyata repository dan menerjemahkan blueprint menjadi backlog yang dapat diverifikasi.

Pekerjaan:
- Inventarisasi seluruh file, apps, packages, scripts, dependencies, migrations, functions, tests, CI/CD, dan environment config.
- Bandingkan repository aktual dengan struktur target.
- Baca blueprint lengkap `000–031`; buat requirement register.
- Buat matriks `REQ/BR/ADR → module → migration/table → API/RPC → UI route → test → evidence`.
- Identifikasi duplikasi, dead code, stub, mock, hardcoded business values, security gap, dan dependency risk.
- Buat diagram arsitektur aktual dan target.
- Buat `docs/decisions/OPEN-QUESTIONS.md`, risk register, serta rencana fase.

Deliverables:
- `docs/audit/REPOSITORY-AUDIT.md`
- `docs/architecture/CURRENT-STATE.md`
- `docs/architecture/TARGET-ARCHITECTURE.md`
- `docs/traceability/REQUIREMENTS-MATRIX.md`
- `docs/decisions/OPEN-QUESTIONS.md`
- backlog per fase dengan dependency dan acceptance criteria.

**Gate 0:**
- Seluruh direktori dan entry point penting terinventarisasi.
- Semua domain blueprint terpetakan.
- Gap dan risiko kritis tercatat.
- Tidak ada asumsi stack/fitur yang disajikan sebagai fakta.
- Product Owner menyetujui pertanyaan yang memerlukan keputusan bisnis.

### PHASE 1 — Engineering foundation dan monorepo

**Tujuan:** membuat fondasi build/test yang reproducible sebelum business feature.

Pekerjaan:
- Konvergensi ke pnpm workspace + Turborepo secara bertahap.
- Siapkan apps staff, pos, customer sesuai scope; customer boleh feature-disabled bila belum dibutuhkan.
- Siapkan shared packages: ui, schemas, domain, api-client, config.
- Konfigurasi TypeScript strict, lint, format, Vite, path alias, env validation, build pipeline.
- Buat design tokens, layout shell, navigation, reusable form/table/dialog/toast, error boundary.
- Siapkan test harness Vitest/Testing Library/Playwright.
- Siapkan GitHub Actions untuk install frozen lockfile, lint, typecheck, unit test, build.
- Tambahkan README developer setup dan `.env.example` tanpa secret.

**Gate 1:**
- Clean install dari lockfile berhasil.
- Semua app/package yang didefinisikan dapat typecheck dan build.
- CI berjalan pada pull request.
- Tidak ada secret dalam repository atau bundle.
- Shared package dapat dipakai lebih dari satu app.
- Minimal smoke test tiap app lolos.

### PHASE 2 — Database, identity, authorization, dan security foundation

**Tujuan:** membangun batas kepercayaan dan data model sebelum transaksi bisnis.

Pekerjaan:
- Supabase local/staging config dan migration workflow.
- Core schema: organizations, branches, profiles, employees/assignments, roles, permissions, user-role assignments, config registry/value, audit, idempotency, outbox, document sequences.
- Auth invitation, email verification/reset, MFA policy, session revocation.
- RBAC permission registry dan branch scope.
- RLS untuk semua tabel API-exposed; grants dan function privilege review.
- Private storage baseline.
- Seed non-production yang deterministic.
- Database/RLS tests untuk anonymous, authenticated, role berbeda, branch berbeda, dan cross-branch denial.
- Generated TypeScript database types.

**Gate 2:**
- Migration dari database kosong berhasil.
- RLS negative tests membuktikan akses lintas cabang/role ditolak.
- Tidak ada service-role key pada frontend.
- MFA policy dan bootstrap user aman.
- Audit dan permission checks berjalan server-side.
- Migration, seed, dan generated types konsisten.

### PHASE 3 — Core business data: customer, pet, catalog, supplier, inventory

**Tujuan:** membangun master data dan persediaan yang menjadi fondasi penjualan/layanan.

Pekerjaan:
- Customer, contacts, consent, pet, pet ownership, vaccination, tags/interactions, complaint.
- Product category, brand, unit, product, branch pricing, bundle, tax category references.
- Supplier, warehouse, inventory lots, cost pools, stock movements, stock balances, reservations.
- Purchasing: PO, approvals, goods receipt, supplier bills/payments bila dependency siap.
- Stock count, adjustment request/approval, transfer antar gudang/cabang.
- Costing moving average dan FEFO.
- Import template dan validation untuk master data awal.

**Gate 3:**
- Customer/pet ownership dan consent invariant teruji.
- Scope organisasi/cabang tervalidasi.
- Movement ledger immutable dan balance projection cocok.
- Cost pool dan rounding tests lolos.
- FEFO/expiry/lot traceability tests lolos.
- Partial receipt, transfer transit, opname, concurrency, dan idempotency tests lolos.

### PHASE 4 — POS, shift, sales, payment, dan pet services

**Tujuan:** mengaktifkan transaksi operasional inti setelah fondasi data aman.

Pekerjaan:
- Cashier register/shift, cash movements, opening/closing reconciliation.
- Sales preview/quote, price snapshot, tax/discount/fee/rounding.
- Sales create atomic RPC, payment intents, cash dan metode pembayaran yang sudah diuji.
- Refund/void/approval dan receipt snapshot.
- Inventory consumption, COGS, loyalty hooks, journal hooks sesuai konfigurasi.
- Booking, resource reservation, service price/duration, grooming checklist, boarding stay/care logs, transport jika sesuai kontrak.
- POS offline banner dan draft cart IndexedDB best-effort; blokir commit ketika offline.
- Print fallback PDF/browser sesuai kemampuan perangkat yang diuji.

**Gate 4:**
- Sale/payment/stock/journal effects atomik sesuai aturan.
- Duplicate submit/webhook tidak menggandakan transaksi.
- Pending provider tidak dianggap paid.
- Refund tidak melebihi refundable amount/qty.
- Shift reconciliation dan cash variance teruji.
- Booking concurrency mencegah double booking.
- E2E alur kasir dan pet service lolos.

### PHASE 5 — Veterinary ecosystem

**Tujuan:** membangun alur klinis dengan kontrol akses dan integritas rekam medis.

Pekerjaan:
- Clinical queue/encounter, medical record draft/finalize, amendment.
- Structured diagnosis/procedure, consent, attachment private.
- Prescription, prescription items, partial dispense, pharmacy, lot/expiry.
- Lab order/result validation, surgery case, hospitalization/stay dan monitoring.
- Role separation veterinarian/pharmacist/front desk/customer.
- Customer shared medical documents sesuai consent/permission.
- Audit access dan perubahan data klinis sesuai kebijakan.

**Gate 5:**
- RLS dan role tests untuk medical/payroll isolation lolos.
- Final medical record tidak dapat diubah.
- Amendment menjaga histori dan actor/signature.
- Dispensing atomik dengan stock ledger dan expiry validation.
- Attachment private dan signed URL access teruji.
- Clinical workflow UAT oleh pihak klinis yang berwenang.

### PHASE 6 — Membership, loyalty, promotions, HR, payroll, finance, accounting

**Tujuan:** melengkapi kontrol komersial, SDM, dan pembukuan.

Pekerjaan:
- Membership tiers, customer membership, loyalty ledger.
- Promotion rules, voucher, redemption, stacking dan limits.
- Employee schedule, attendance, leave, commission.
- Payroll periods/runs/lines/payslips dengan versioning, review, approval, payment.
- Chart of accounts, accounting periods, journals, cash/bank, AP/AR, settlement.
- Tax profiles/rates dan effective date; konfigurasi harus disetujui finance.
- Auto-posting mapping dari transaksi ke jurnal; reversal dan reconciliation.
- Finance reports dengan scope permission.

**Gate 6:**
- Loyalty/promo concurrency dan reversal tests lolos.
- Payroll segregation of duties dan sensitive field masking lolos.
- Journal balance invariant dan closed-period tests lolos.
- Account mapping/COA disetujui finance.
- Tax profile dan pemetaan produk/jasa valid.
- Rekonsiliasi sumber transaksi ke ledger/jurnal terbukti.

### PHASE 7 — Extended ecosystem, admin, integrations, notifications

**Tujuan:** mengaktifkan kemampuan pendukung dan kontrol platform.

Pekerjaan:
- Assets/events, budgets, SOP, documents/versioning.
- Delivery orders/events, online orders kanal milik perusahaan.
- Super admin config registry/value, effective config preview, history/diff, rollback-as-new-version.
- Integration accounts dan adapter contracts.
- Notification templates, consent checks, delivery attempts.
- Transactional outbox worker, lease, retry, dedupe, dead-letter, manual replay audit.
- Provider sandbox tests untuk email/WhatsApp/payment/shipping/storage backup bila provider dipilih.
- Sentry/error monitoring dan PII redaction.
- Job health dan operational dashboards.

**Gate 7:**
- Tidak ada arbitrary code/plugin execution dari admin config.
- Secrets hanya server-side.
- Adapter timeout/retry/signature/SSRF protection diuji.
- Outbox tidak kehilangan event dan tidak menggandakan side effect.
- Consent dan notification dedupe diuji.
- Config change tervalidasi, berversi, dan diaudit.

### PHASE 8 — Integration, QA, security, performance, UAT, data readiness

**Tujuan:** membuktikan seluruh sistem bekerja end-to-end dan siap dipromosikan.

Pekerjaan:
- Trace semua requirement dan acceptance criteria.
- Unit, integration, DB, RLS, API contract, concurrency, idempotency, E2E, visual/accessibility tests.
- Test cross-module workflows: customer→booking→service→sale→payment→stock→loyalty→journal; clinic→prescription→dispense→stock→invoice; purchasing→receipt→cost pool→sale→COGS.
- Security review: auth, RLS, IDOR, privilege escalation, webhook replay, secret scan, CSP/CORS, file access, rate limit.
- Performance/load tests pada query, dashboard, POS, inventory, report, worker.
- Backup/restore drill dan disaster recovery rehearsal.
- Data import rehearsal, checksum, opening balance reconciliation.
- UAT per role dan cabang.
- Perbaiki blocker/high severity; dokumentasikan known issues.

**Gate 8:**
- Tidak ada blocker/critical security issue terbuka.
- Seluruh critical requirement memiliki test/evidence.
- RLS matrix dan branch isolation lulus.
- Backup restore berhasil dalam target RPO/RTO yang disetujui.
- UAT ditandatangani owner/domain owner.
- Data migration dan opening balances direkonsiliasi.
- Monitoring, alert, runbook, support contact tersedia.

### PHASE 9 — Staging, production deployment, go-live, dan handover

**Tujuan:** merilis sistem secara terkendali dan memastikan operasi dapat dijalankan.

Pekerjaan:
- Environment Supabase staging/production terpisah.
- MFA owner/platform account, billing alerts, recovery contacts.
- Production secrets dan hosting secrets dikelola di secret manager.
- CI menjalankan migration; review migration ID, grants, RLS, storage policies, seed.
- Deploy Edge Functions dan scheduler/outbox.
- Deploy frontend immutable build ke satu platform pilihan (Vercel atau Cloudflare Pages).
- Redirect/CORS/CSP, SPA fallback, domain, Auth redirect.
- Restore point/backup verification sebelum release.
- Promote migration kompatibel, function, frontend; manual approval production.
- Smoke tests non-destruktif, canary/monitoring.
- Enable payment/portal/integration secara bertahap setelah approval.
- Handover runbook, incident response, backup/restore, reconciliation, support, release record.

**Gate 9 — Production readiness:**
- Seluruh environment dan secrets diverifikasi.
- Migration production dijalankan melalui CI, bukan laptop.
- Restore point dan backup evidence tersedia.
- Auth/MFA, RLS, branch scope, POS, booking, private file, outbox, health, observability smoke tests lolos.
- Tax profile dan seluruh SKU/service tax mapping disetujui finance.
- COA dan account mapping tervalidasi sebelum auto-posting.
- Opening inventory, receivable/payable, cash/bank, staff, product, price direkonsiliasi.
- Provider live hanya setelah sandbox test dan owner approval.
- Rollback/forward-fix dan incident commander ditetapkan.
- Owner menyetujui go-live secara eksplisit.

**Jangan menyatakan PETORA production-ready jika satu saja go-live blocker blueprint belum terpenuhi.**

## 9. Testing policy dan bukti

Setiap fitur harus memiliki test sesuai risikonya:

| Lapisan | Bukti minimum |
|---|---|
| Unit | Formula, validation, state transition, permission helper |
| Component | Form, loading/error/empty, keyboard/accessibility |
| API | Schema, auth, permission, idempotency, error contract |
| Database | Constraint, RPC atomicity, rollback, concurrency |
| RLS | Allowed dan denied matrix untuk role/branch |
| Integration | Provider adapter, webhook signature/replay, outbox retry |
| E2E | Alur pengguna lintas modul |
| Security | Secret scan, IDOR, privilege escalation, storage access |
| Performance | Query plan, latency, load, worker backlog |
| Recovery | Backup restore dan deployment rollback rehearsal |
| UAT | Acceptance oleh role/domain owner |

- Jangan menghapus atau skip test yang gagal untuk membuat CI hijau.
- Jangan memakai mock untuk mengklaim integrasi nyata sudah bekerja.
- Tandai test sebagai mock/contract/sandbox/real integration dengan jelas.
- Untuk race condition, gunakan concurrent test yang membuktikan hanya satu operasi berhasil sesuai invariant.
- Simpan hasil test, commit SHA, environment, dan waktu sebagai evidence bila relevan.

## 10. API, Edge Function, RPC, dan integration conventions

- Function publik/worker per use case, bukan satu function per tabel CRUD.
- Request JSON divalidasi Zod; timestamp API ISO-8601 UTC.
- Mutasi retryable memakai `Idempotency-Key`.
- Terima/buat `X-Request-Id` untuk tracing.
- Response sukses memiliki `data` dan `meta.request_id`.
- Error response memiliki stable `code`, pesan aman, `fields`, dan `request_id`.
- Jangan mengembalikan SQL, stack trace, secret, atau detail internal.
- `branch_id` dari request body tidak dipercaya sebelum assignment dicek.
- RPC write menjalankan efek domain terkait dalam satu transaksi database.
- Webhook persist/dedupe dahulu; pemrosesan bisnis dilakukan terkontrol.
- External provider failure tidak boleh rollback transaksi internal yang sudah commit; catat status outbox dan retry.
- Adapter harus memiliki timeout, host allowlist, SSRF protection, error mapping, retry policy, sandbox flag, capability matrix, dan reconciliation bila relevan.

## 11. Environment dan deployment safety

Environment minimal:
- Local development.
- Staging.
- Production.

Aturan:
- Supabase project staging dan production terpisah.
- Frontend build hanya menerima variabel publik yang diizinkan.
- `SUPABASE_ACCESS_TOKEN` CI-only, masked, least privilege, dan dirotasi.
- `SUPABASE_PROJECT_REF` adalah identifier, bukan password.
- Provider/webhook/encryption secrets disimpan di Supabase Edge secrets/Vault atau secret manager server-side.
- Hosting deploy token scope project saja; jangan expose pada preview dari fork.
- CI harus memblokir secret/private key/service role pada production bundle.
- Migration memakai expand/contract bila perubahan membutuhkan kompatibilitas.
- Urutan release normal: PR → CI → migration review → staging → UAT → backup/restore point → manual production approval → compatible migration → Edge Functions → immutable frontend → smoke tests → monitor → release record.
- Rollback frontend ke immutable deployment sebelumnya; Edge Function ke versi known-good yang kompatibel.
- Database utamakan forward-fix; jangan `db reset` atau drop data production.
- Payment incident: hentikan metode terdampak; pending tidak boleh ditandai paid; rekonsiliasi sebelum retry/refund.
- Security incident: revoke/rotate, batasi akses, simpan evidence/audit.

## 12. Dokumentasi dan traceability wajib

Pertahankan dokumen berikut (buat bila belum ada):

- `docs/audit/REPOSITORY-AUDIT.md` — kondisi aktual, gap, risiko.
- `docs/architecture/CURRENT-STATE.md` — arsitektur yang benar-benar ada.
- `docs/architecture/TARGET-ARCHITECTURE.md` — arsitektur target.
- `docs/decisions/ADR-*.md` — keputusan yang disetujui.
- `docs/decisions/OPEN-QUESTIONS.md` — ambiguitas menunggu keputusan.
- `docs/traceability/REQUIREMENTS-MATRIX.md` — requirement ke kode/test/evidence.
- `docs/runbooks/LOCAL-DEVELOPMENT.md`.
- `docs/runbooks/DEPLOYMENT.md`.
- `docs/runbooks/ROLLBACK.md`.
- `docs/runbooks/BACKUP-RESTORE.md`.
- `docs/runbooks/INCIDENT-RESPONSE.md`.
- `docs/runbooks/RECONCILIATION.md`.
- `docs/releases/` — release record dan checklist go-live.

Setiap requirement setidaknya mempunyai:
`requirement_id`, deskripsi, prioritas, status, dependency, implementasi path, migration/API, test path, evidence, owner/approver, dan gap.

Status requirement yang diperbolehkan:
`not_started`, `in_progress`, `implemented`, `tested`, `accepted`, `deployed`, `blocked`, `deferred`.

Jangan ubah status ke `tested`, `accepted`, atau `deployed` tanpa bukti yang sesuai.

## 13. Definition of Done (DoD)

Sebuah task hanya boleh ditandai selesai jika:

- Requirement dan acceptance criteria jelas.
- Implementasi sesuai blueprint dan tidak melanggar invariant.
- Permission, RLS, branch scope, validation, idempotency, audit sesuai risiko.
- Loading/error/empty/success state dan accessibility diperhatikan.
- Migration/type/API/docs diperbarui jika diperlukan.
- Unit/integration/RLS/E2E test relevan lulus.
- Lint, typecheck, build lulus pada scope yang terdampak.
- Tidak ada secret, PII, clinical payload, atau financial sensitive data bocor.
- Tidak ada regression yang diketahui.
- Requirement matrix dan evidence diperbarui.
- Risiko tersisa dan keputusan yang menunggu approval dinyatakan.

## 14. Perilaku agent yang dilarang

Agent DILARANG:

- Mengarang requirement, endpoint, field, enum, formula, tax rate, permission, provider, atau business workflow.
- Mengganti keputusan blueprint demi preferensi pribadi tanpa ADR dan approval.
- Membangun microservices, SaaS tenant billing, marketplace multi-vendor, atau fitur non-scope seolah bagian dari v1.0.
- Menganggap UI hiding sebagai authorization.
- Menggunakan service-role key di frontend.
- Menggunakan floating point untuk nominal IDR.
- Mengubah saldo stok secara langsung atau mengedit stock ledger.
- Mengedit transaksi final, posted journal, atau medical record final.
- Menganggap payment pending sebagai paid.
- Mengizinkan POS final transaction saat offline.
- Mengirim notifikasi tanpa consent/channel check.
- Menyimpan secret/PII/medical payload di log, URL, localStorage, analytics, atau error tracker.
- Menjalankan destructive command, migration production manual, deploy, push, atau provider live activation tanpa otorisasi.
- Menghapus test yang gagal atau menurunkan coverage/strictness untuk menyamarkan masalah.
- Menyebut mock, placeholder, stub, atau hardcoded demo sebagai fitur produksi.
- Mengklaim selesai/aman/siap deploy tanpa evidence dan acceptance gate.

## 15. Format laporan setiap sesi/task

Akhiri setiap task dengan laporan ringkas:

```text
## Task Report
- Phase:
- Scope:
- Requirement IDs:
- Status: completed | partial | blocked
- Files changed:
- Migrations/API/RPC changed:
- Tests executed:
- Test results:
- Security/data-integrity review:
- Documentation/evidence updated:
- Known gaps and risks:
- Decisions/approvals needed:
- Recommended next task:
```

Jika task terblokir:
- Jelaskan penyebab dan bukti.
- Jelaskan dampak jika diteruskan.
- Berikan opsi yang sesuai blueprint.
- Minta satu keputusan paling penting.
- Jangan melanjutkan implementasi yang berisiko dengan asumsi.

## 16. Instruksi awal untuk agent baru

Saat pertama kali menerima repository PETORA:

1. Jangan langsung membangun fitur.
2. Baca `AGENTS.md` dan seluruh `PETORA — BLUEPRINT FINAL LENGKAP v1.0.md`.
3. Jalankan Phase 0 secara read-only terlebih dahulu.
4. Periksa apakah source code sudah ada; jangan menyimpulkan status implementasi dari blueprint.
5. Buat audit aktual, target architecture, requirement matrix, open questions, dan backlog bertahap.
6. Laporkan kondisi repository dan usulan Phase 1 kepada pengguna/Product Owner.
7. Mulai implementasi Phase 1 hanya setelah hasil audit cukup dan keputusan yang memblokir telah diselesaikan.

**Prioritas tertinggi:** kebenaran spesifikasi, keamanan, integritas data/transaksi, keterlacakan, dan bukti pengujian — bukan kecepatan menghasilkan banyak file atau tampilan.
