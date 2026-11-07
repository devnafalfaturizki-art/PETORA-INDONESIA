# PETORA - BLUEPRINT FINAL LENGKAP v1.0

**ID Dokumen:** `PETORA-FINAL-BLUEPRINT-000`  
**Versi:** 1.0.0
**Status:** Baseline spesifikasi normatif untuk implementasi penuh  
**Tanggal:** 7 November 2026  
**Produk:** Sistem operasional perusahaan petshop, petcare, dan klinik hewan dengan satu perusahaan dan satu atau lebih cabang.

Dokumen ini adalah satu-satunya acuan blueprint PETORA. Kata **WAJIB**, **DILARANG**, dan **BOLEH** memiliki makna normatif. Dokumen ini menetapkan rancangan sampai kriteria siap produksi; status blueprint tidak berarti produk telah dibangun, diuji, atau dideploy.

## Indeks Dokumen

| ID                 | Bagian                                    | Isi                                                              |
| ------------------ | ----------------------------------------- | ---------------------------------------------------------------- |
| `PETORA-FINAL-000` | Tata Kelola Spesifikasi                   | Tujuan, istilah, aturan perubahan, keputusan mengikat            |
| `PETORA-FINAL-001` | Ruang Lingkup Produk                      | Pengguna, konteks, modul, batas sistem                           |
| `PETORA-FINAL-002` | Keputusan Arsitektur                      | ADR, stack, deployment Vercel/Cloudflare Pages/Supabase          |
| `PETORA-FINAL-003` | Domain dan Data                           | Entitas, relasi, integritas, skema inti                          |
| `PETORA-FINAL-004` | Keamanan dan Akses                        | Auth, RBAC, cabang, RLS, privasi, audit                          |
| `PETORA-FINAL-005` | Backend dan API                           | Batas layanan, transaksi atomik, kontrak API, idempotensi        |
| `PETORA-FINAL-006` | Frontend dan UX                           | Aplikasi, navigasi, state, aksesibilitas, offline, cetak         |
| `PETORA-FINAL-007` | Aturan Bisnis                             | POS, stok, harga, booking, klinik, keuangan, SDM                 |
| `PETORA-FINAL-008` | Modul Organisasi dan Pengguna             | Perusahaan, cabang, user, role, dashboard                        |
| `PETORA-FINAL-009` | Modul Pelanggan, Hewan, CRM               | Profil, kepemilikan, consent, komunikasi, keluhan                |
| `PETORA-FINAL-010` | Modul Katalog dan Persediaan              | Produk, gudang, pembelian, stok, batch, transfer                 |
| `PETORA-FINAL-011` | Modul Penjualan dan Pembayaran            | POS, shift, refund, voucher, retur, rekonsiliasi                 |
| `PETORA-FINAL-012` | Modul Layanan dan Pet Hotel               | Booking, grooming, daycare, boarding, transport                  |
| `PETORA-FINAL-013` | Modul Klinik dan Farmasi                  | Rekam medis, resep, lab, tindakan, rawat inap                    |
| `PETORA-FINAL-014` | Modul Membership dan Promosi              | Tier, poin, kupon, campaign, aturan stacking                     |
| `PETORA-FINAL-015` | Modul SDM                                 | Karyawan, jadwal, absensi, komisi, payroll                       |
| `PETORA-FINAL-016` | Modul Keuangan dan Akuntansi              | Kas, piutang, utang, jurnal, pajak, laporan                      |
| `PETORA-FINAL-017` | Modul Operasional Tambahan                | Aset, SOP, dokumen, pengiriman, toko online                      |
| `PETORA-FINAL-018` | Super Admin dan Konfigurasi               | Config registry, integrasi, rahasia, feature access              |
| `PETORA-FINAL-019` | Workflow End-to-End                       | Urutan status dan efek transaksi antarmodul                      |
| `PETORA-FINAL-020` | Operasional dan Deployment                | Lingkungan, CI/CD, domain, backup, observabilitas                |
| `PETORA-FINAL-021` | QA dan Kriteria Penerimaan                | Test, security gate, performa, readiness produksi                |
| `PETORA-FINAL-022` | Kamus, Register ID, dan Keputusan Terbuka | Terminologi, traceability, daftar larangan                       |
| `PETORA-FINAL-023` | Kontrak Implementasi Terperinci           | Field dictionary, state machine, API, layar, event, laporan, UAT |
| `PETORA-FINAL-024` | Formula dan Kalkulasi Domain              | Harga, pajak, promo, costing, loyalty, komisi, layanan           |
| `PETORA-FINAL-025` | Model Otorisasi dan RLS                   | Permission code, cabang, klasifikasi tabel, policy, storage      |
| `PETORA-FINAL-026` | Runbook Deployment Produksi               | Hosting, migrasi, rollout, rollback, secrets, smoke test         |
| `PETORA-FINAL-027` | Kontrak Modul, Fungsi, dan Workflow       | Aktor, fungsi, transisi, integrasi, validasi, acceptance         |

---

## PETORA-FINAL-000 - Tata Kelola Spesifikasi

### 000.1 Tujuan dan status

Dokumen ini adalah acuan desain dan kontrak perilaku yang konsisten bagi product owner, developer, QA, keamanan, operator, serta pihak deployment. Semua bagian menggunakan keputusan yang sama; jika ada perubahan kebutuhan, perubahan harus dicatat sebagai keputusan bernomor, disetujui pemilik produk, dan memperbarui semua referensi terkait sebelum implementasi.

### 000.2 Konvensi ID

| Objek                | Pola                      | Contoh                        |
| -------------------- | ------------------------- | ----------------------------- |
| Dokumen              | `PETORA-FINAL-NNN`        | `PETORA-FINAL-003`            |
| Keputusan arsitektur | `ADR-PET-001`             | hosting frontend              |
| Kebutuhan            | `REQ-PET-NNN`             | `REQ-PET-014`                 |
| Aturan bisnis        | `BR-PET-DOM-NNN`          | `BR-PET-POS-001`              |
| Tabel                | `snake_case`              | `sales_transactions`          |
| Route                | lowercase kebab           | `/inventory/stock`            |
| Permission           | `domain:resource:action`  | `inventory:stock:adjust`      |
| Config key           | `domain.setting_name`     | `pos.refund_window_days`      |
| Event                | `domain.aggregate.action` | `sales.transaction.completed` |
| Edge Function        | lowercase kebab           | `sales-create`                |

Semua tabel transaksional memiliki UUID sebagai PK dan timestamps UTC. Nomor dokumen yang ditampilkan manusia bukan PK, tidak boleh dipakai sebagai FK. Prefix contoh pada blueprint tidak menjadi format ID internal.

### 000.3 Bahasa, format, dan zona waktu

UI utama berbahasa Indonesia (`id-ID`); nama domain/kode teknis tetap bahasa Inggris. Mata uang operasional tunggal adalah IDR, disimpan sebagai bilangan bulat Rupiah. Waktu disimpan sebagai `timestamptz` UTC dan ditampilkan dalam zona waktu IANA cabang. Format UI: tanggal `dd MMM yyyy`, waktu 24 jam, mata uang mengikuti `id-ID`. Zona waktu cabang wajib valid dari database IANA; nilai awal `Asia/Jakarta`.

### 000.4 Klasifikasi data

- **Master:** produk, layanan, customer, supplier, pegawai, akun COA; dapat diarsipkan, bukan dihapus jika sudah direferensikan.
- **Transaksi/fakta:** penjualan, pembayaran, movement stok, jurnal, rekam medis final; immutable setelah finalisasi, koreksi memakai reversal/amendment.
- **Konfigurasi:** nilai bertipe dari registry yang disetujui; perubahan memiliki versi dan audit.
- **Rahasia:** kredensial provider dan key; tidak pernah dikirim ke browser, log, atau respons API.
- **Cache/proyeksi:** boleh dibangun ulang dari sumber transaksi yang otoritatif.

### 000.5 Prinsip perubahan

Migrasi database hanya maju dan dapat diulang dengan aman; data historis tidak diubah diam-diam. Setiap perubahan schema menyertakan migration, kebijakan RLS, tipe frontend yang dihasilkan, test, dan pembaruan dokumen. Tidak ada perubahan langsung di produksi tanpa migration berversi.

---

## PETORA-FINAL-001 - Ruang Lingkup Produk

### 001.1 Model bisnis dan batas tenant

`REQ-PET-001`: satu instalasi PETORA melayani **satu perusahaan** dan satu atau lebih cabang. Aplikasi bukan SaaS multi-tenant, tidak ada pendaftaran organisasi mandiri, billing SaaS, atau isolasi pelanggan antarbisnis. `organization_id` tetap disimpan untuk integritas dan kesiapan ekspor, tetapi seluruh instalasi memiliki tepat satu organisasi aktif. Data cabang dapat dibatasi per pengguna.

### 001.2 Persona

- Pemilik/direktur: laporan lintas cabang, konfigurasi, approval, kontrol keuangan.
- Super admin operasional: user, role, config, integrasi, audit dan health; tidak otomatis berhak membaca seluruh rekam medis.
- Manajer cabang: operasional, staf, harga lokal, approval cabang.
- Kasir: POS, pembayaran, shift, struk; tidak boleh mengubah transaksi final.
- Staf gudang/purchasing: PO, penerimaan, transfer, opname, stok.
- Groomer/front desk/pet hotel: booking dan perawatan sesuai tugas.
- Dokter hewan: antrean, rekam medis, resep, persetujuan medis.
- Apoteker/petugas farmasi: validasi dan dispensing resep.
- Finance/HR: jurnal, pembayaran, payroll sesuai pembatasan data.
- Customer: portal opsional, hanya data dirinya dan hewan yang menjadi tanggung jawabnya.

### 001.3 Cakupan produk

Semua domain dalam indeks adalah bagian dari desain produk. Modul yang tidak dipakai perusahaan dapat dinonaktifkan melalui hak akses/feature setting, namun skema, kontrak, dan interaksi yang disebut wajib konsisten. Toko online berarti kanal order milik perusahaan yang memakai katalog/stok internal; bukan marketplace multi-vendor.

### 001.4 Batas eksternal

PETORA tidak menggantikan sistem resmi bank, gateway, penyedia WhatsApp/email/SMS, sistem pajak pemerintah, perangkat absensi biometrik, atau nasihat dokter/akuntan. Koneksi dilakukan melalui adapter yang dikendalikan admin. Kegagalan provider tidak boleh merusak transaksi internal yang telah commit; status pengiriman/integrasi dicatat dan dapat diproses ulang.

### 001.5 Kebutuhan nonfungsional

- `REQ-PET-002`: UI desktop/tablet, layout kasir touch-friendly, customer portal responsif mobile.
- `REQ-PET-003`: seluruh operasi tulis penting memiliki audit trail, permission server-side, dan idempotensi.
- `REQ-PET-004`: tidak ada penjualan final saat browser offline; draft keranjang boleh disimpan lokal dan harus dikonfirmasi ulang saat online.
- `REQ-PET-005`: dukung cabang dengan data dan laporan tersaring; dashboard lintas cabang hanya bagi permission yang sesuai.
- `REQ-PET-006`: data klinis dan payroll memiliki akses paling ketat sesuai kebutuhan kerja.

---

## PETORA-FINAL-002 - Keputusan Arsitektur

### 002.1 Register keputusan final

| ID            | Keputusan normatif                                                                                                                                                                       | Konsekuensi                                                                                                                   |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `ADR-PET-001` | Modular monolith, satu monorepo; bukan microservices.                                                                                                                                    | Domain dipisah per modul, deploy tetap sedikit dan transaksi dapat atomik.                                                    |
| `ADR-PET-002` | Frontend React + TypeScript + Vite; tiga entry app staff, POS, customer pada monorepo.                                                                                                   | Aplikasi dapat dirilis terpisah, memakai UI dan schema bersama.                                                               |
| `ADR-PET-003` | Hosting frontend mendukung Vercel **atau** Cloudflare Pages. Vercel menjadi konfigurasi referensi; satu platform dipilih per environment, tidak menjalankan dua frontend produksi aktif. | Semua app SPA build statis, tanpa ketergantungan runtime khusus provider hosting.                                             |
| `ADR-PET-004` | Backend data/auth/storage Supabase Cloud (PostgreSQL, Auth, Storage, Realtime, Edge Functions).                                                                                          | Supabase adalah sumber data dan batas kepercayaan backend.                                                                    |
| `ADR-PET-005` | Domain writes dieksekusi melalui Edge Function yang memvalidasi lalu memanggil satu RPC PostgreSQL atomik per use case.                                                                  | Tidak ada rangkaian REST writes yang dapat meninggalkan setengah transaksi.                                                   |
| `ADR-PET-006` | Query data biasa memakai Supabase client dengan RLS; laporan sensitif/kompleks memakai fungsi query/RPC berpermission.                                                                   | Tidak ada service-role key di frontend.                                                                                       |
| `ADR-PET-007` | PostgreSQL `uuid` PK dengan `gen_random_uuid()` (UUID v4).                                                                                                                               | Tidak mengklaim UUID v7 bila generator DB standar tidak menjaminnya; UUID tidak mengandung makna bisnis.                      |
| `ADR-PET-008` | TanStack Query untuk server state; Zustand hanya untuk state UI/keranjang lokal yang belum commit.                                                                                       | Cache server tidak menjadi sumber kebenaran.                                                                                  |
| `ADR-PET-009` | React Hook Form + Zod; TypeScript types database dihasilkan dari migration/schema dan validasi request berada di backend.                                                                | Validasi browser bukan pengganti validasi server.                                                                             |
| `ADR-PET-010` | UI shadcn/ui + Radix + Tailwind, Lucide; tabel TanStack Table.                                                                                                                           | Aksesibilitas dan konsistensi komponen wajib diuji.                                                                           |
| `ADR-PET-011` | Supabase Auth email/password, MFA wajib untuk super admin/owner, opsional sesuai kebijakan role lain.                                                                                    | Password tidak disimpan di tabel aplikasi.                                                                                    |
| `ADR-PET-012` | Outbox transactional di PostgreSQL untuk event/pekerjaan eksternal; worker terjadwal memproses dan retry.                                                                                | Event tidak hilang jika provider sedang gagal.                                                                                |
| `ADR-PET-013` | Stok memakai movement ledger append-only, saldo terproyeksi diperbarui dalam transaksi DB; valuasi moving average. Lot FEFO untuk item bertanggal kedaluwarsa.                           | Saldo tidak boleh diedit langsung.                                                                                            |
| `ADR-PET-014` | Penjualan final, pembayaran internal, konsumsi stok, loyalty, dan jurnal yang diwajibkan dilakukan dalam transaksi DB tunggal.                                                           | Pembayaran gateway asynchronous menjadi pending intent sampai webhook terverifikasi.                                          |
| `ADR-PET-015` | Konfigurasi operasional bertipe di DB dan dapat dikelola di panel berdasarkan allowlist. Rahasia integrasi di Vault/secret store server-side.                                            | “Tanpa hardcode” berlaku untuk nilai bisnis dan endpoint provider yang didukung, bukan secrets, schema, atau aturan keamanan. |
| `ADR-PET-016` | GitHub Actions untuk CI/CD; Supabase migrations sebagai satu-satunya jalur perubahan schema.                                                                                             | Build dan migration dapat diaudit dan dipromosikan.                                                                           |
| `ADR-PET-017` | Supabase Realtime hanya untuk invalidasi/pemberitahuan perubahan yang sudah commit.                                                                                                      | Realtime bukan event ledger atau otorisasi.                                                                                   |
| `ADR-PET-018` | POS online-required untuk menyelesaikan transaksi. Draft keranjang dapat di-cache lokal; tidak ada antrean pembayaran/penjualan final offline.                                           | Menghindari double sale, oversell, dan pembayaran tanpa commit.                                                               |
| `ADR-PET-019` | Integrasi hanya adapter provider yang telah diimplementasikan; panel memilih provider/config, bukan mengunggah kode/plugin arbitrer.                                                     | Tidak ada eksekusi kode dari konfigurasi admin.                                                                               |
| `ADR-PET-020` | Sentry untuk FE/function error dengan redaksi PII; platform dashboard untuk database/runtime.                                                                                            | Error memiliki correlation/request ID tanpa membocorkan isi klinis/rahasia.                                                   |

### 002.2 Stack versi dan dependensi

Versi mayor dikunci di lockfile; pembaruan dilakukan melalui PR dan test. Baseline implementasi: Node.js LTS yang masih didukung saat rilis, pnpm workspace + Turborepo; React 19 stabil, TypeScript strict, Vite stabil; React Router; TanStack Query; Zustand; Zod; Tailwind; shadcn/ui/Radix; Lucide; Vitest/Testing Library; Playwright. Backend memakai Supabase CLI yang dipin di CI, Edge Functions Deno runtime yang didukung Supabase, PostgreSQL versi yang tersedia pada project target. Jangan menyalin nomor versi lama dari blueprint; CI adalah sumber versi aktual.

### 002.3 Struktur monorepo kanonis

```text
petora/
  apps/
    staff/                 # portal staf dan admin
    pos/                   # terminal kasir
    customer/              # portal pelanggan opsional
  packages/
    ui/                    # komponen bersama
    schemas/               # schema input/output Zod
    domain/                # perhitungan murni dan enum domain
    api-client/            # wrapper function dan error mapping
    config/                # konfigurasi build non-secret
  supabase/
    migrations/            # SQL berurutan dan immutable
    functions/             # Edge Functions per use case
    seed/                  # data referensi non-produksi
    tests/                 # database/RLS tests
  e2e/                     # Playwright
  .github/workflows/       # CI dan deploy
  package.json
  pnpm-lock.yaml
  turbo.json
```

### 002.4 Konfigurasi build vs konfigurasi runtime

Browser hanya menerima `SUPABASE_URL`, publishable/anon key, nama environment non-rahasia, dan Sentry DSN yang memang publik. Semua prefiks `VITE_` dianggap terbuka. Service role, DB password, webhook signing secret, encryption key, dan provider API secret hanya ada di secret store server/CI. URL, domain, CORS allowlist, redirect Auth, dan identifier project adalah konfigurasi deployment; bukan nilai yang dapat diubah bebas dari panel bisnis.

---

## PETORA-FINAL-003 - Domain dan Data

### 003.1 Aturan model data

1. Semua uang IDR menggunakan `bigint` Rupiah; jumlah barang/jasa menggunakan `numeric(18,4)`; persentase/rate menggunakan `numeric` dengan skala eksplisit.
2. Waktu memakai `timestamptz`; tanggal bisnis memakai `date`; jam buka memakai waktu lokal dan zona cabang.
3. Tidak ada FK yang mengandalkan polymorphic `reference_type/reference_id` untuk transaksi inti. Sumber referensi utama memakai FK eksplisit; outbox/audit dapat menyimpan tipe dan UUID sebagai konteks.
4. Semua tabel organisasi menyimpan `organization_id`; tabel yang dimiliki cabang menyimpan `branch_id`. Constraint FK gabungan memastikan relasi berada pada organisasi/cabang yang benar.
5. Master yang telah dipakai memakai `archived_at`; transaksi final tidak dihapus. Hard delete hanya untuk data konfigurasi/draft yang tidak direferensikan dan sesuai kebijakan retensi.
6. Nilai total pada dokumen dan item adalah snapshot pada saat transaksi. Perubahan katalog di masa depan tidak mengubah histori.
7. Kolom `created_at`, `created_by`, `updated_at`, `updated_by` berlaku untuk tabel mutable; data ledger memiliki waktu/pelaku kejadian dan tidak boleh diubah.
8. Status disimpan sebagai `text` dengan `CHECK` atau lookup terkontrol; nilai status di UI dan DB bersumber dari schema bersama.

### 003.2 Peta tabel kanonis

Nama di bawah adalah nama tabel implementasi; tabel detail dan indeks tambahan dibuat melalui migrations. Setiap tabel punya PK UUID, FK, constraint organisasi, indeks sesuai filter nyata, serta RLS kecuali dinyatakan sebagai internal-only.

#### Organisasi, akses, dan sistem

| Tabel                                      | Tujuan dan field kanonis penting                                                                                            |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `organizations`                            | Singleton organisasi: nama legal/brand, kontak, NPWP opsional, locale, status.                                              |
| `branches`                                 | `organization_id`, code unik, nama, alamat, timezone, jam operasional, status.                                              |
| `profiles`                                 | Satu-ke-satu `auth.users`: nama tampilan, kontak, avatar, status; bukan password.                                           |
| `employees`                                | Nomor pegawai, profil, jabatan, status kerja, tanggal masuk/keluar, cabang utama. PII sensitif dipisahkan/diakses terbatas. |
| `employee_branch_assignments`              | Relasi karyawan-cabang dan rentang aktif.                                                                                   |
| `roles`, `permissions`, `role_permissions` | Role scope perusahaan dan permission code stabil.                                                                           |
| `user_role_assignments`                    | User, role, scope cabang nullable hanya untuk akses semua cabang yang disetujui.                                            |
| `user_sessions_audit`                      | Peristiwa keamanan yang relevan; token/session secret tidak disimpan.                                                       |
| `system_config_registry`                   | Daftar key, tipe, schema, scope yang diizinkan, validasi, dampak, nilai default. Dibuat migration, bukan admin bebas.       |
| `system_config_values`                     | Nilai konfigurasi per organisasi/cabang, versi, actor, waktu; tidak menyimpan secret.                                       |
| `integration_accounts`                     | Provider, status, referensi secret Vault, scope, status test, metadata nonsecret.                                           |
| `audit_logs`                               | Append-only actor/action/resource, before/after yang sudah disaring, cabang, request_id, waktu.                             |
| `idempotency_records`                      | Scope, key, payload hash, status, response aman, expiry. Unique per user/use case/key.                                      |
| `outbox_events`, `job_runs`                | Event eksternal, percobaan, jadwal retry, hasil/error redacted.                                                             |
| `document_sequences`                       | Sequence transactional per cabang, jenis dokumen, periode.                                                                  |

#### Pelanggan dan hewan

| Tabel                                     | Tujuan dan field kanonis penting                                                                                         |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `customers`                               | Nomor pelanggan, nama, kontak, alamat, status, consent, preferensi komunikasi. Kontak normalisasi dipisah dari tampilan. |
| `customer_contacts`                       | Email/telepon, nilai terenkripsi bila perlu, nilai normalized untuk lookup, verified_at, primary flag.                   |
| `customer_consents`                       | Jenis consent, status, sumber, versi notice, waktu dan actor; withdrawal dipertahankan.                                  |
| `pets`                                    | Nama, spesies, breed, jenis kelamin, DOB/perkiraan, karakteristik, status hidup, catatan yang diizinkan.                 |
| `pet_owners`                              | Pet/customer, relasi, primary, masa berlaku; minimal satu pemilik aktif untuk layanan pelanggan.                         |
| `pet_vaccinations`                        | Vaksin, tanggal, batch, pemberi, tanggal pengingat, lampiran.                                                            |
| `customer_tags`, `customer_interactions`  | Segmentasi dan timeline komunikasi; campaign harus memeriksa consent.                                                    |
| `complaint_tickets`, `complaint_messages` | Status, prioritas, assignee, SLA, resolusi, komunikasi.                                                                  |

#### Katalog, pemasok, dan persediaan

| Tabel                                     | Tujuan dan field kanonis penting                                                                                                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `product_categories`, `brands`, `units`   | Referensi katalog dengan kode unik organisasi.                                                                                        |
| `products`                                | SKU unik organisasi, barcode, nama, kategori, unit dasar, tipe, tax profile, harga referensi, track batch/expiry, resep wajib, arsip. |
| `product_branch_settings`                 | Harga jual/threshold stok per cabang; efektif mulai/akhir.                                                                            |
| `product_bundle_components`               | Produk induk dan komponen/qty; dilarang siklus bundling.                                                                              |
| `suppliers`                               | Kode, kontak, termin, legal info, status.                                                                                             |
| `warehouses`                              | Cabang, tipe gudang (jual, transit, karantina, rusak, titipan), status.                                                               |
| `inventory_lots`                          | Produk, gudang, lot/batch, tanggal expiry, qty tersedia/reserved, biaya unit. Unique dengan penanganan batch null yang eksplisit.     |
| `stock_movements`                         | Ledger immutable: tipe, qty bertanda, biaya, lot, referensi FK, actor, waktu.                                                         |
| `stock_balances`                          | Proyeksi saldo per produk/gudang/lot, diperbarui hanya fungsi DB yang terotorisasi.                                                   |
| `stock_reservations`                      | Reservasi booking/order, lot, qty, expiry; release/commit atomik.                                                                     |
| `stock_counts`, `stock_count_lines`       | Opname snapshot, hitungan, selisih, approval, movement adjustment saat posting.                                                       |
| `purchase_orders`, `purchase_order_items` | PO draft/approval, qty/harga/tax snapshot dan total.                                                                                  |
| `goods_receipts`, `goods_receipt_items`   | Penerimaan parsial, lot/expiry, qty diterima, quality hold.                                                                           |
| `supplier_bills`, `supplier_payments`     | Hutang dan pembayaran beserta alokasi.                                                                                                |
| `stock_transfers`, `stock_transfer_items` | Transfer antar gudang dengan status transit dan penerimaan tujuan.                                                                    |

#### Penjualan, pelayanan, klinik

| Tabel                                                          | Tujuan dan field kanonis penting                                                                              |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `cashier_shifts`, `cash_movements`                             | Kasir/cabang/register, saldo awal, cash-in/out, hasil hitung dan selisih tutup shift.                         |
| `sales_transactions`                                           | Nomor, cabang, customer optional sesuai config, status, snapshot totals/tax, cashier, shift, idempotency ref. |
| `sales_items`                                                  | Product/service snapshot, qty, harga, discount, tax, line total, lot, staff, pet opsional.                    |
| `sales_payments`                                               | Metode, amount, status, provider/ref, paid/refunded amount.                                                   |
| `refunds`, `refund_items`                                      | Referensi transaksi/item asal, qty/nilai maksimal belum direfund, approval, alasan, payment result.           |
| `services`, `service_branch_prices`                            | Jenis layanan, durasi, buffer, kapasitas, harga dan aturan/species-size per cabang.                           |
| `bookings`, `booking_services`, `booking_assignments`          | Pelanggan/pet, slot, cabang, staff/resource, estimate/deposit, status, sumber.                                |
| `grooming_sessions`, `grooming_checklists`                     | Kondisi awal/akhir, foto dengan akses private, checklist, staff, status.                                      |
| `rooms`, `boarding_stays`, `boarding_care_logs`                | Kapasitas, okupansi rentang waktu, check-in/out, feeding/medication, observasi harian.                        |
| `medical_records`, `medical_record_amendments`                 | Rekam klinis dengan status draft/final/amended, pencatat dan snapshot perubahan.                              |
| `medical_diagnoses`, `medical_procedures`                      | Data terstruktur bila diperlukan, selain narasi klinis.                                                       |
| `prescriptions`, `prescription_items`, `dispensations`         | Dokter, rekam final, aturan dosis, qty, lot yang diserahkan, partial dispense/refill.                         |
| `lab_orders`, `lab_results`, `surgery_cases`, `hospital_stays` | Order/pemeriksaan, hasil tervalidasi, consent, operasi, monitoring/rawat inap.                                |

#### Membership, SDM, dan keuangan

| Tabel                                                                      | Tujuan dan field kanonis penting                                                       |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `membership_tiers`, `customer_memberships`                                 | Tier/benefit versi efektif, status dan periode customer.                               |
| `loyalty_ledger`                                                           | Earn/redeem/expire/reversal append-only; saldo adalah proyeksi.                        |
| `promotions`, `promotion_rules`, `vouchers`, `voucher_redemptions`         | Periode, scope, syarat, batas, stacking, penggunaan atomik.                            |
| `employee_schedules`, `attendance_records`, `leave_requests`               | Jadwal, clock events, approval, geofence bila kebijakan mengaktifkan.                  |
| `commission_rules`, `commission_entries`                                   | Aturan efektif dan snapshot komisi per transaksi/layanan.                              |
| `payroll_periods`, `payroll_runs`, `payroll_lines`, `payslips`             | Snapshot komponen, perhitungan, review, approval, payment status.                      |
| `chart_of_accounts`, `accounting_periods`                                  | COA hirarkis dan periode open/closed.                                                  |
| `journal_entries`, `journal_lines`                                         | Header dan baris double-entry, referensi sumber, post/reversal.                        |
| `cash_accounts`, `cash_expenses`, `receivables`, `payables`, `settlements` | Saldo, jatuh tempo, alokasi pembayaran dan rekonsiliasi.                               |
| `tax_profiles`, `tax_rates`                                                | Jenis pajak dan rentang berlaku; nilai harus ditinjau finance sesuai regulasi berlaku. |
| `assets`, `asset_events`, `budgets`, `budget_lines`                        | Aset dan perawatan, budget, realisasi/approval.                                        |
| `documents`, `document_versions`                                           | Metadata, storage path private, klasifikasi, akses dan masa retensi.                   |
| `notifications`, `notification_attempts`, `message_templates`              | In-app dan provider delivery dengan status/retry.                                      |
| `delivery_orders`, `delivery_events`, `online_orders`                      | Order milik perusahaan, pemenuhan dan tracking.                                        |

### 003.3 Constraint lintas cabang

Foreign keys untuk data cabang menggunakan composite key yang memuat `organization_id` dan, bila relevan, `branch_id`. Satu cabang tidak boleh mereferensikan gudang, user scope, atau service milik cabang/organisasi lain. Database function yang memakai `SECURITY DEFINER` harus memvalidasi actor, organisasi, cabang, permission, dan `search_path` eksplisit.

### 003.4 Angka transaksi dan snapshot

Rumus dibulatkan per baris sesuai konfigurasi mata uang/pajak yang terversi. Jumlah nominal yang disimpan di header harus sama dengan agregasi baris + fee - discount + tax + rounding. Database function menghitung ulang dan menolak total yang dikirim client tidak cocok. Harga, diskon, rate pajak, nama item, satuan, dan identitas lokasi disimpan sebagai snapshot pada item transaksi.

### 003.5 Nomor dokumen

Nomor manusia dibangkitkan di DB dalam transaksi dengan row lock pada `document_sequences`, format default `{PREFIX}/{BRANCH}/{YYYY}/{MM}/{SEQ:6}`. Format dan reset periodenya dapat diatur lewat registry yang dibatasi; nomor yang sudah diterbitkan tidak pernah digunakan ulang. Tidak menggunakan `MAX()+1` atau sequence yang dapat menghasilkan duplikasi cabang.

### 003.6 Indeks dan retensi

Index wajib mencakup seluruh FK, `(branch_id, business_date)`, lookup SKU/barcode, status+tanggal antrean, expiry lot aktif, serta reference transaksi ledger. Gunakan partial index untuk data aktif jika sesuai query. Audit, finansial, medis, dan payroll mengikuti jadwal retensi yang disetujui organisasi dan regulasi; penghapusan permanen melalui prosedur terkontrol, bukan tombol rutin.

### 003.7 Field-level contract untuk transaksi inti

Semua field `NOT NULL` di bawah harus memiliki constraint eksplisit; nilai nullable harus mempunyai makna bisnis. Nama dapat disesuaikan saat migration hanya jika kontrak API dan test ikut diperbarui.

| Tabel                                             | Kolom inti dan kontrak                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sales_transactions`                              | `id uuid PK`; `organization_id`, `branch_id`; `transaction_number`; `business_date`; `status` (`completed`, `voided`, `partially_refunded`, `refunded`); `payment_status` (`pending`, `partial`, `paid`, `failed`, `refunded`); `customer_id NULL`; `cashier_user_id`; `shift_id`; `currency='IDR'`; `subtotal`, `discount_total`, `taxable_total`, `tax_total`, `fee_total`, `rounding_total`, `grand_total`, `paid_total`, `change_total bigint`; `tax_snapshot jsonb`; `completed_at`; `created_at`; unique `(branch_id, transaction_number)`. Total non-negatif; `grand_total` sama dengan formula header. |
| `sales_items`                                     | `id`; `transaction_id FK`; `line_no`; `item_kind` (`product`, `service`, `bundle`); nullable `product_id`/`service_id` dengan tepat satu sumber valid kecuali bundle; `sku_snapshot`, `name_snapshot`, `unit_snapshot`; `quantity numeric(18,4)>0`; `unit_price`, `gross_amount`, `discount_amount`, `taxable_amount`, `tax_amount`, `line_total bigint`; `tax_profile_snapshot`; `warehouse_id NULL`; `pet_id NULL`; `assigned_employee_id NULL`; unique `(transaction_id,line_no)`.                                                                                                                          |
| `sales_payments`                                  | `id`; `transaction_id FK`; `method`; `amount bigint>0`; `status` (`pending`, `succeeded`, `failed`, `cancelled`, `refunded`); `provider_code/ref` nullable; `idempotency/provider_event_ref` unique jika tersedia; `paid_at`; `refunded_amount bigint` dengan `0<=refunded_amount<=amount`; payment card data tidak disimpan.                                                                                                                                                                                                                                                                                  |
| `inventory_lots`                                  | `id`; `organization_id`, `branch_id`, `warehouse_id`, `product_id`; `batch_number NULL` hanya bila produk tidak melacak batch; `expiry_date NULL` hanya bila produk tidak mewajibkan expiry; `on_hand_qty`, `reserved_qty numeric(18,4)`; `average_unit_cost bigint`; `received_at`; `archived_at`; check `on_hand_qty>=0`, `reserved_qty>=0`, `reserved_qty<=on_hand_qty` kecuali override resmi; unique index lot aktif dengan NULL semantics yang eksplisit.                                                                                                                                                |
| `stock_movements`                                 | `id`; scope organisasi/cabang; `warehouse_id`, `product_id`, `inventory_lot_id NULL`; `movement_type`; `quantity_delta numeric(18,4)<>0` bertanda; `unit_cost bigint>=0`; `source_document_type`, `source_document_id`, `source_line_id`; `actor_user_id`; `occurred_at`; `reversal_of_movement_id NULL`; unique sumber untuk mencegah posting ganda. Tidak ada UPDATE/DELETE dari role aplikasi.                                                                                                                                                                                                              |
| `bookings`                                        | `id`; organisasi/cabang; `booking_number`; customer/pet; `status`; `starts_at`, `ends_at` (`ends_at>starts_at`); `timezone_snapshot`; `service_price_snapshot bigint`; `deposit_due`, `deposit_paid bigint`; source, notes terpisah internal/customer; cancellation actor/reason; created/updated actor. Unique nomor per cabang.                                                                                                                                                                                                                                                                              |
| `booking_resource_reservations`                   | `id`; booking; resource type/id (staff, room, equipment); `slot tstzrange` `[start,end)`; status aktif/dirilis; exclusion constraint untuk mencegah overlap resource tunggal. Kapasitas lebih dari satu dikontrol lock + counter dalam RPC.                                                                                                                                                                                                                                                                                                                                                                    |
| `medical_records`                                 | `id`; organisasi/cabang; `record_number`; pet/customer; attending employee; encounter time; status (`draft`, `finalized`, `amended`); `version`; clinical payload terstruktur tervalidasi; `finalized_at/by`; `amends_record_id NULL`; created actor. Rekam final immutable; versi revisi tersimpan di `medical_record_amendments`.                                                                                                                                                                                                                                                                            |
| `journal_entries` / `journal_lines`               | Header: scope, `journal_number`, `journal_date`, period, source type/id, status, description, posted/reversal refs, actor/approver. Lines: account, branch, description, `debit bigint>=0`, `credit bigint>=0`; tepat satu sisi positif. RPC memvalidasi minimal dua line dan total debit=credit sebelum post.                                                                                                                                                                                                                                                                                                 |
| `system_config_registry` / `system_config_values` | Registry: `config_key PK`, `value_type`, `schema jsonb`, `default_value`, `allowed_scope`, `edit_permission`, `sensitivity`, `effective_dated`. Values: scope/org/branch, `config_key FK`, `value jsonb`, `version`, `effective_from/to`, actor/time, reason; unique key + scope + effective start. Rahasia dilarang pada tabel ini.                                                                                                                                                                                                                                                                           |

### 003.8 Constraint database wajib

- Check constraint untuk nominal non-negatif, quantity positif pada item dan status yang valid; ledger quantity menggunakan delta bertanda.
- FK transaksi tidak boleh cascade-delete keuangan/medis/stock. Parent yang sudah dipakai tidak dapat dihapus; foreign key historis tetap tersedia.
- Setiap tabel child transaksi membawa `organization_id` atau divalidasi melalui FK parent; trigger/RPC menolak cabang yang tidak cocok.
- `stock_balances` dan saldo loyalty adalah proyeksi; update hanya lewat function `SECURITY DEFINER` yang sempit, audit actor dan validasi konsistensi terhadap ledger.
- Reservation, voucher usage, document sequence, shift aktif, alokasi refund, dan jurnal sumber memiliki constraint/lock di database, bukan hanya pengecekan UI.
- Gunakan `btree_gist` dan exclusion constraint hanya pada tabel resource reservation jika extension tersedia di project; jika tidak, implementasikan lock resource di RPC dan test concurrency yang setara.
- Trigger `updated_at` hanya untuk tabel mutable; trigger audit tidak boleh merekam secret/raw medical payload secara tidak terfilter.

---

## PETORA-FINAL-004 - Keamanan dan Akses

### 004.1 Authentication

Supabase Auth menangani password, reset, verifikasi, refresh token, dan MFA. Akun staff dibuat melalui undangan admin; customer signup hanya boleh jika portal diaktifkan dan harus diverifikasi/link ke customer tanpa membuka akun customer lain. Login disable/employee terminated harus mencabut akses dengan memperbarui status profil/role dan mencabut sesi.

### 004.2 RBAC dan cakupan cabang

Permission format `domain:resource:action`. Role adalah kumpulan permission; assignment memiliki scope perusahaan atau cabang tertentu. Akses ke semua cabang hanya melalui assignment eksplisit. Permission selalu dicek di Edge Function/RPC; menyembunyikan tombol bukan kontrol akses. Super admin bukan bypass terhadap audit, MFA, atau batas akses medis yang terpisah.

### 004.3 RLS wajib

RLS diaktifkan untuk semua tabel yang dapat diakses melalui API. Policy menggunakan `auth.uid()` menuju profile/assignment aktif, membership organisasi tunggal dan branch scope. Jangan menggunakan `current_setting('app.organization_id')` sebagai identitas tepercaya karena client dapat memanipulasi session setting. `anon` tidak memiliki akses tabel bisnis. Storage bucket private dan policy mengikuti owner/branch/permission; file diakses lewat signed URL berdurasi pendek.

### 004.4 Jalur service role

Service-role key hanya digunakan di Edge Function/internal worker, tidak pernah dari browser. Setiap handler memvalidasi JWT user dan membentuk client yang meneruskan JWT untuk operasi ber-RLS. Penggunaan elevated client dibatasi pada tindakan sistem yang memerlukan privilege; actor dan permission diperiksa sebelum RPC. Function definer tidak terbuka untuk `public` tanpa alasan.

### 004.5 PII dan data medis

Gunakan HTTPS/TLS dan enkripsi managed provider at rest. Field sensitif seperti identifier resmi, rekening, dan kredensial provider disimpan terpisah dan dienkripsi di server bila dibutuhkan; lookup menggunakan normalized/hash yang sesuai, bukan plaintext log. Nomor telepon untuk pencarian boleh memakai HMAC index terpisah. Rekam medis, payroll, dokumen, dan catatan operasional memiliki permission terpisah. Jangan klaim enkripsi aplikasi otomatis untuk semua kolom tanpa key management, rotasi, recovery, dan migrasi yang diuji.

### 004.6 Audit dan anti-abuse

Catat login penting, perubahan role/config/integrasi, pembacaan/ekspor data sensitif bila diwajibkan, approval, transaksi final, refund, stock adjustment, jurnal, payroll, dan download dokumen. Audit append-only, akses baca terbatas, PII dihapus/redacted dari snapshot. Rate limit login, reset password, endpoint public, booking portal, webhook dan export. Terapkan CSRF bila memakai cookie session sendiri; pada Supabase bearer token tetap validasi origin dan CORS allowlist.

### 004.7 Webhook

Verifikasi signature provider, timestamp/replay window, event ID unik, dan status provider melalui API bila tersedia. Webhook tidak mempercayai redirect browser. Simpan payload minimum yang sudah disaring, tangani duplikasi idempotent, dan balas cepat setelah persist; pemrosesan bisnis berjalan terkontrol.

### 004.8 Role dan permission default

Scope cabang selalu membatasi akses meskipun role mengandung permission. `Owner` adalah pemilik bisnis; `Super Admin` mengelola sistem tetapi tidak otomatis memiliki permission klinis/finance; setiap role tambahan mengikuti least privilege.

| Role                     | Scope default                            | Permission bundle                                                                                      | Batasan eksplisit                                                                                         |
| ------------------------ | ---------------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| `owner`                  | semua cabang                             | laporan, konfigurasi bisnis, user/role, approval dan operasi; akses finansial sesuai penugasan owner   | MFA wajib; perubahan sensitif tetap diaudit; tidak membaca rahasia provider                               |
| `super_admin`            | semua cabang untuk health/config         | `settings:*`, integrasi, audit, job health, user support                                               | Tidak otomatis `vet:*`, payroll detail, bank detail atau secret read                                      |
| `branch_manager`         | cabang assignment                        | dashboard cabang, customer, staff operasional, booking, approval PO/refund/adjustment sesuai threshold | Tidak dapat melihat cabang lain, secret, atau mengubah permission global                                  |
| `cashier`                | cabang/register assignment               | customer lookup terbatas, POS sale, shift open/close, receipt, booking create                          | Tidak boleh edit final transaction, refund/void sendiri di atas limit, lihat medical detail, payroll, COA |
| `inventory_staff`        | cabang assignment                        | product read, stock read, receiving, transfer, count dan adjustment request                            | Adjustment final/selisih material memerlukan approver lain                                                |
| `purchasing_staff`       | cabang assignment                        | supplier, PO draft, receipt draft, bill matching view                                                  | Tidak menyetujui PO sendiri di atas threshold atau melakukan pembayaran                                   |
| `groomer` / `front_desk` | cabang assignment                        | booking, check-in, grooming checklist, care logs, customer contact yang diperlukan                     | Tidak melihat rekam medis penuh, harga pokok, payroll                                                     |
| `veterinarian`           | cabang klinik assignment                 | antrean, rekam medis, diagnosis, tindakan, resep, hasil klinik                                         | Tidak mengubah transaksi/pembayaran atau payroll; final record memiliki audit                             |
| `pharmacist`             | cabang klinik assignment                 | resep, dispensing, lot obat, stock read untuk farmasi                                                  | Tidak membuat/mengubah diagnosis atau final record dokter                                                 |
| `finance_staff`          | cabang atau perusahaan sesuai assignment | kas/bank, AP/AR, jurnal, tax setup, finance reports                                                    | Data klinis hanya minimum untuk referensi invoice; payment approval terpisah                              |
| `hr_staff`               | perusahaan/assigned branches             | employee administration, attendance, payroll draft, payslip administration                             | Bank/ID data masked; payroll approval dipisah bila memungkinkan                                           |
| `customer_service`       | cabang assignment                        | profil pelanggan, pet basic, complaint, booking, consent, loyalty support                              | Tidak melihat payroll/COA/medical notes internal                                                          |
| `customer`               | record milik sendiri                     | portal profile, pets yang terhubung, booking, transaksi sendiri, shared medical documents              | Tidak dapat memilih customer_id lain atau membaca internal note                                           |

Permission inti harus di-seed eksplisit, termasuk `read`, `create`, `update`, `approve`, `post`, `export`, dan `admin` bila operasi tersebut berbeda. Wildcard hanya boleh digunakan pada seed role yang direview; custom role menyimpan permission individual dan tidak boleh membuat permission baru.

---

## PETORA-FINAL-005 - Backend dan API

### 005.1 Boundary layanan

Frontend tidak boleh mengirim SQL, memakai key privileged, atau menjadi penentu harga/pajak/stok. Edge Function menangani autentikasi, validasi schema, rate limit, idempotensi, pemetaan error, lalu memanggil RPC yang menjalankan seluruh perubahan terkait dalam satu transaksi DB. Query read sederhana lewat RLS; data agregasi/privileged via RPC read-only.

### 005.2 Bentuk fungsi

```text
supabase/functions/
  _shared/         # auth, permission, cors, errors, schemas, tracing, provider adapters
  sales-create/
  sales-refund/
  shifts-close/
  inventory-adjust/
  inventory-transfer/
  purchasing-receive/
  booking-transition/
  medical-record-finalize/
  pharmacy-dispense/
  payroll-run/
  integration-webhook-{provider}/
  jobs-process-outbox/
```

Gunakan satu fungsi per use case publik/worker yang jelas, bukan satu function per tabel CRUD. Endpoint hanya metode yang diperlukan; CORS allowlist environment-specific. Semua handler punya timeout dan batas ukuran request.

### 005.3 Kontrak umum API

Request JSON tervalidasi Zod; gunakan ISO-8601 UTC. Header wajib untuk mutasi retryable: `Idempotency-Key`; `X-Request-Id` dapat diterima/dibuat server. Respons sukses:

```json
{
  "data": {},
  "meta": { "request_id": "uuid" }
}
```

Respons gagal:

```json
{
  "error": {
    "code": "STOCK_INSUFFICIENT",
    "message": "Stok tidak mencukupi.",
    "fields": [],
    "request_id": "uuid"
  }
}
```

Kode umum: `VALIDATION_ERROR` 400, `UNAUTHENTICATED` 401, `FORBIDDEN` 403, `NOT_FOUND` 404, `CONFLICT` 409, `BUSINESS_RULE_VIOLATION` 422, `RATE_LIMITED` 429, `INTEGRATION_UNAVAILABLE` 503, `INTERNAL_ERROR` 500. Jangan mengirim stack trace atau detail SQL kepada client.

### 005.4 Idempotensi

Scope unique adalah `(actor_id, use_case, idempotency_key)`. Simpan hash canonical request dan hasil final. Request ulang dengan key dan payload sama mengembalikan hasil identik; key sama dengan payload berbeda menghasilkan `409 IDEMPOTENCY_KEY_REUSED`. Catat record idempotensi di transaksi yang sama dengan efek bisnis. TTL untuk respons sesuai use case; transaksi ledger tetap disimpan.

### 005.5 Transaksi atomik

Mutation kritis mempunyai satu fungsi SQL/RPC per use case. Contoh `sales_create_atomic`: lock lot/balance yang dipakai, validasi harga/izin/stock/shift/voucher/loyalty, hitung ulang total, alokasikan lot FEFO, insert header/items/payments, insert stock movements, update balance, ledger loyalty, journal, outbox, sequence, dan idempotency response, lalu commit. Error di langkah mana pun rollback semuanya. Jangan melakukan reserve via request terpisah lalu mencoba kompensasi di aplikasi.

### 005.6 Event dan pekerjaan asynchronous

Outbox ditulis dalam transaksi bisnis yang sama. Worker menggunakan claim/lease dan retry exponential dengan batas, dead-letter status dan tombol retry permission-limited. Side effects seperti email, WhatsApp, PDF, export, backup verification, dan notifikasi tidak menunda commit penjualan. Setiap handler downstream idempotent.

### 005.7 Pagination, filter, dan pencarian

List API menggunakan pagination cursor untuk dataset besar; page/limit dibatasi (maksimum 100). Sort key harus allowlist. Search hanya pada kolom terindeks dan scope permission. Export besar berjalan sebagai job, file hasil private, signed link kadaluarsa, dan tercatat audit.

### 005.8 API use case minimum

| Function / RPC                                     | I/O utama                                        | Permission                     |
| -------------------------------------------------- | ------------------------------------------------ | ------------------------------ |
| `sales-create` / `sales_create_atomic`             | cart, pembayaran, customer/pet, key -> transaksi | `pos:transaction:create`       |
| `sales-refund` / `sales_refund_atomic`             | item/qty/refund reason/approval -> refund        | `pos:refund:create` + approval |
| `shifts-open`, `shifts-close`                      | register/opening count/closing count -> shift    | `pos:shift:*`                  |
| `inventory-adjust` / `inventory_adjust_atomic`     | gudang, lot, qty, alasan -> movement             | `inventory:stock:adjust`       |
| `inventory-transfer` / `inventory_transfer_atomic` | source/target/items -> transfer                  | `inventory:transfer:create`    |
| `purchasing-receive` / `purchase_receive_atomic`   | PO, accepted qty, batch/expiry -> GR             | `purchase:receipt:create`      |
| `booking-create`, `booking-transition`             | service, slot, pet, staff -> booking             | `booking:booking:*`            |
| `medical-record-finalize`                          | draft version -> immutable final record          | `vet:medical_record:finalize`  |
| `pharmacy-dispense`                                | prescription/items/lot -> dispense               | `pharmacy:dispense`            |
| `payroll-run`, `payroll-approve`                   | period -> snapshot/approval                      | `hr:payroll:*`                 |
| `config-update`                                    | config key/value/version -> audited version      | `settings:config:update`       |
| `integration-test`                                 | provider reference -> connection result          | `settings:integration:test`    |
| `jobs-process-outbox`                              | bounded batch -> processing report               | internal scheduler only        |

---

## PETORA-FINAL-006 - Frontend dan UX

### 006.1 Aplikasi dan navigasi

**Staff app**: dashboard, organisasi, pelanggan/hewan, katalog/stok, sales, purchasing, layanan/booking, klinik/farmasi, membership/promo, SDM, keuangan/akuntansi, laporan, CRM/keluhan, operasional, dokumen, pengaturan.  
**POS app**: login/branch/register selection, shift, transaksi, hold draft lokal, riwayat shift, refund sesuai permission, tutup kasir.  
**Customer app**: login/consent, profil, hewan, booking, histori transaksi, membership/poin, notifikasi dan dokumen yang dibagikan. App pelanggan tidak mengakses data medis internal penuh.

### 006.2 Route map

```text
/staff/login                         /staff/dashboard
/staff/organization/branches         /staff/organization/users
/staff/organization/roles            /staff/customers
/staff/customers/:id                 /staff/pets/:id
/staff/catalog/products              /staff/catalog/services
/staff/inventory/stock               /staff/inventory/movements
/staff/inventory/transfers           /staff/inventory/counts
/staff/purchasing/orders             /staff/purchasing/receipts
/staff/sales/transactions            /staff/sales/refunds
/staff/booking/calendar              /staff/booking/:id
/staff/grooming/queue                /staff/boarding/stays
/staff/clinic/queue                  /staff/clinic/records/:id
/staff/pharmacy/prescriptions        /staff/hr/payroll
/staff/finance/journals              /staff/finance/reports
/staff/crm/complaints                /staff/operations/checklists
/staff/reports                       /staff/settings/config
/staff/settings/integrations         /staff/super-admin/audit
/pos/login                           /pos/register
/pos/sale                            /pos/transactions
/pos/shifts/:id/close                /customer/login
/customer/dashboard                  /customer/pets
/customer/bookings                   /customer/transactions
/customer/membership                 /customer/profile
```

Setiap route memiliki permission requirement, branch context, loading/empty/error/unauthorized state, breadcrumb, dan filter URL yang dapat dibagikan jika data tidak sensitif.

### 006.3 Shell dan komponen

Staff: sidebar dengan modul sesuai permission, header dengan branch selector dan profil, konten utama padat dan mudah dipindai. POS: fokus input barcode/cart/total/payment dengan dukungan keyboard dan touch. Customer: navigasi mobile-first yang sederhana. Komponen bersama: `AppShell`, `PageHeader`, `BranchSelector`, `PermissionGate`, `DataTable`, `FormField`, `SearchSelect`, `ConfirmDialog`, `Toast`, `StatusBadge`, `AuditTimeline`, `MoneyInput`, `DateTimeInput`, `FilePicker`.

### 006.4 State dan validasi UI

TanStack Query untuk server data dan invalidation setelah mutasi; query key wajib mencakup user/branch/filter yang memengaruhi hasil. Zustand hanya untuk keranjang draft, modal, dan state transient. Jangan menyimpan permission sebagai otoritas; refresh dari server. Form memakai schema yang berbagi aturan bentuk dengan backend, tetapi server tetap mengulang semua validasi. Optimistic update hanya untuk perubahan nonfinansial yang dapat dibatalkan dengan aman.

### 006.5 POS dan offline

Ketika jaringan putus, tampilkan banner offline dan hentikan proses pembayaran/commit. Draft keranjang dapat disimpan di IndexedDB secara best-effort dengan data minimum, tidak menyimpan kredensial/kartu/payment data. Saat online kembali, refresh harga, pajak, promo, customer, stok dan minta kasir memeriksa ulang sebelum submit. Setiap transaksi final memerlukan respons commit backend. Printer gagal tidak membatalkan transaksi; struk dapat dicetak ulang.

### 006.6 Cetak dan file

Struk/invoice dibentuk dari snapshot transaksi yang sudah tersimpan. Print thermal mengandalkan browser print/WebUSB atau bridge yang dipilih saat implementasi, mendeteksi dukungan perangkat dan memberi fallback PDF; jangan menjanjikan akses USB lintas browser tanpa uji perangkat. Dokumen klinis/HR memakai bucket private, signed URL pendek, tipe dan ukuran file allowlist, scanning bila disediakan layanan, serta validasi server.

### 006.7 Aksesibilitas dan kualitas UX

Target WCAG 2.2 AA untuk alur kritis: label, fokus terlihat, keyboard, kontras, error terkait input, dialog dapat ditutup, status tidak hanya dibedakan warna. Semua dialog konfirmasi untuk transaksi destruktif menjelaskan objek/nilai. Tabel mendukung pencarian, sorting, filter, loading dan empty state. Layak digunakan pada lebar 360px untuk portal; alur staff di layar kecil harus tetap dapat menyelesaikan tugas penting tanpa overflow horizontal yang tidak perlu.

### 006.8 Keamanan frontend

CSP mengikuti host aktual; hindari `unsafe-eval`, inline script, dan wildcard connect-src. Sanitasi konten user, jangan render HTML template pesan mentah. Logout membersihkan cache sensitif. Data PII/medis tidak masuk localStorage, analytics, URL query, atau error breadcrumb.

---

## PETORA-FINAL-007 - Aturan Bisnis

### 007.1 Identitas dan hak akses

- `BR-PET-CORE-001`: setiap record milik tepat satu organisasi aktif.
- `BR-PET-CORE-002`: actor hanya memproses cabang yang ada di assignment aktif atau scope lintas cabang.
- `BR-PET-CORE-003`: perubahan role, scope, config sensitif, harga, payroll, refund, dan jurnal wajib mencatat actor dan alasan jika diperlukan.
- `BR-PET-CORE-004`: reference historis tidak hilang karena archive master; label snapshot tetap terlihat.

### 007.2 Sales dan pembayaran

- `BR-PET-POS-001`: total server dihitung dari item qty × harga snapshot, diskon, pajak, fee, dan rounding; nilai client hanya input/indikasi.
- `BR-PET-POS-002`: transaksi final harus memiliki minimal satu payment sukses atau saldo piutang yang diizinkan.
- `BR-PET-POS-003`: pembayaran tunai dapat lebih besar dari total; kembalian hanya tunai dan perubahan payment total harus seimbang.
- `BR-PET-POS-004`: multi-payment yang sukses/diotorisasi harus menutup total; provider pending tidak dianggap lunas.
- `BR-PET-POS-005`: transaksi final immutable. Void hanya draft atau transaksi sebelum settlement sesuai status; transaksi settled dikoreksi melalui refund.
- `BR-PET-POS-006`: refund tidak melebihi qty/nominal item yang belum direfund, jendela default ditentukan config yang disetujui, wajib alasan dan approval sesuai limit.
- `BR-PET-POS-007`: refund mengembalikan stok hanya jika barang secara fisik diterima dan lolos kondisi; pilih gudang/karantina; refund jasa tidak menciptakan stok.
- `BR-PET-POS-008`: diskon tidak boleh membuat harga di bawah floor kecuali permission override dan alasan tercatat.
- `BR-PET-POS-009`: saldo shift, kas yang diharapkan, hitungan aktual, dan selisih disimpan; tutup shift tidak mengubah transaksi penjualan.

### 007.3 Inventory dan purchasing

- `BR-PET-INV-001`: saldo hanya berubah melalui `stock_movements` dalam fungsi atomik.
- `BR-PET-INV-002`: stok tersedia = on_hand - reserved; negatif ditolak secara default, dan kebijakan negatif jika diaktifkan hanya melalui approval/konfigurasi eksplisit.
- `BR-PET-INV-003`: outbound lot expiry memakai FEFO; tanpa expiry memakai kebijakan biaya dan penerimaan tercatat.
- `BR-PET-INV-004`: produk tracked expiry harus menerima lot dan expiry; barang expired tidak dapat dijual/didispensasi.
- `BR-PET-INV-005`: penerimaan PO mendukung partial receipt, lot, expiry, qty rejected; hanya accepted qty menaikkan stok.
- `BR-PET-INV-006`: transfer antar cabang melalui gudang transit: dispatch menurunkan sumber dan membuat transit; receiving menambah tujuan; selisih wajib dicatat.
- `BR-PET-INV-007`: opname menyimpan snapshot saat mulai, hitung aktual, selisih dan approval; hanya posting membuat adjustment.
- `BR-PET-INV-008`: COGS memakai moving average yang dihitung/diupdate atomik; metode tidak dapat berubah untuk histori tanpa prosedur migrasi akuntansi.

### 007.4 Booking dan pelayanan

- `BR-PET-SVC-001`: booking memakai rentang waktu half-open `[start,end)`; overlap untuk staff/resource/room/kapasitas dicegah atomik.
- `BR-PET-SVC-002`: aturan slot, lead time, durasi, buffer, deposit, pembatalan dan no-show berasal dari config/service version.
- `BR-PET-SVC-003`: status booking legal: `requested -> confirmed -> checked_in -> in_progress -> completed`; pembatalan/no-show menjadi terminal; reschedule membuat perubahan tercatat, bukan menghapus histori.
- `BR-PET-SVC-004`: finalisasi layanan dapat membuat draft invoice/cart, tetapi hanya sales transaction mencatat penerimaan pembayaran.
- `BR-PET-SVC-005`: boarding tidak dapat menempati kapasitas melebihi room capacity; waktu check-out dan late fee dihitung dari aturan versi yang diterapkan saat stay.

### 007.5 Klinik dan farmasi

- `BR-PET-VET-001`: rekam medis hanya dibuat/diubah oleh petugas berwenang; dokter yang berlisensi dicatat dan status credential dikelola organisasi.
- `BR-PET-VET-002`: finalisasi membuat snapshot tidak dapat diedit; koreksi adalah amendment beralasan dan mempertahankan versi sebelumnya.
- `BR-PET-VET-003`: resep terkontrol memerlukan medical record final dan prescriber yang berwenang; dispensing memeriksa qty tersisa, lot/expiry, alergi dan interaksi yang tersedia.
- `BR-PET-VET-004`: warning alergi harus eksplisit dan acknowledgment tercatat; warning bukan diagnosis otomatis.
- `BR-PET-VET-005`: data portal hanya yang dibagikan dan relevan; catatan internal tidak otomatis tampil ke customer.
- `BR-PET-VET-006`: tindakan klinis berbayar masuk invoice melalui layanan/produk dan tidak menggandakan revenue saat jurnal.

### 007.6 Loyalty, promo, pajak

- `BR-PET-LOY-001`: poin disimpan sebagai ledger earn/redeem/expire/reversal; tidak mengedit saldo historis.
- `BR-PET-LOY-002`: poin dihitung dari eligible net spend setelah diskon dan sebelum/atau sesudah pajak sesuai setting program yang berversi.
- `BR-PET-PROMO-001`: engine promo mengevaluasi tanggal, cabang, channel, kategori, tier, minimum belanja, kuota dan stacking deterministik.
- `BR-PET-PROMO-002`: voucher usage/reservation diklaim atomik agar tidak oversubscribe.
- `BR-PET-TAX-001`: tax profile memiliki effective dates, inclusive/exclusive mode, tax category, rounding dan mapping akun. Rate produksi harus dikonfirmasi finance sesuai regulasi berjalan; tidak ada angka tarif legal permanen yang ditanam di kode.

### 007.7 Akuntansi dan payroll

- `BR-PET-FIN-001`: jurnal posted seimbang dan tidak dapat dihapus; koreksi melalui reversal dan jurnal pengganti.
- `BR-PET-FIN-002`: periode closed menolak posting mundur kecuali dibuka oleh permission khusus dan dicatat audit.
- `BR-PET-FIN-003`: setiap transaksi keuangan hanya memiliki satu sumber posting; laporan memakai posted journal atau definisi sales yang dinyatakan, tidak menjumlah keduanya.
- `BR-PET-HR-001`: payroll disusun dari snapshot data attendance, salary, commission dan komponen periode; perubahan sumber setelah lock tidak mengubah payroll.
- `BR-PET-HR-002`: perhitungan pajak/potongan statutori harus memakai aturan berversi dan dikonfirmasi HR/finance; sistem tidak mengklaim pengganti layanan payroll/akuntan resmi.
- `BR-PET-HR-003`: satu karyawan memiliki maksimal satu payroll final per periode; edit setelah approve lewat reopen berizin dan audit.

---

## PETORA-FINAL-008 - Modul Organisasi dan Pengguna

### 008.1 Setup perusahaan/cabang

Onboarding mengisi profil legal, alamat, timezone, mata uang, periode fiskal, cabang, jam operasional, gudang awal, register POS, tax profile yang telah disetujui, COA dan metode pembayaran. Satu cabang dapat dinonaktifkan jika tidak memiliki transaksi/booking aktif; histori tetap dapat dibaca sesuai permission.

### 008.2 User, role, assignment

Admin mengundang melalui email, memilih profil karyawan/customer, role dan cakupan cabang. Undangan memiliki expiry; pengguna menetapkan password sendiri. Deaktivasi menolak login dan mutasi baru namun mempertahankan audit/history. Role default: Owner, Super Admin, Branch Manager, Cashier, Front Desk, Groomer, Veterinarian, Pharmacist, Inventory, Purchasing, Finance, HR, Customer Service, Customer. Role adalah template; matrix permission aktual berasal dari seeds yang dapat direview dan custom role boleh dibuat dengan prinsip least privilege.

### 008.3 Dashboard

Dashboard menerima filter tanggal/cabang; nilai finansial harus definisinya jelas (gross/net, refunded, tax). Widget: revenue/transactions, payment mix, booking/service queue, stock alert/expiry, PO approval, cash shift, piutang/utang, klinik queue sesuai permission. Dashboard lintas cabang hanya untuk role berizin. KPI disajikan dengan waktu update dan sumber data; jangan membuat query dashboard memindai seluruh ledger tanpa agregasi/index.

---

## PETORA-FINAL-009 - Modul Pelanggan, Hewan, CRM

### 009.1 Customer profile

Customer memiliki nomor internal, nama, kontak normalized dan verified status, alamat, consent marketing, preferensi kanal, catatan layanan, tag, riwayat transaksi dan membership. Duplikasi dicari berdasarkan exact normalized phone/email yang diizinkan; merge customer hanya melalui workflow khusus yang memetakan pet, ledger, transaksi, consent, dan audit.

### 009.2 Pet profile

Dukung spesies/breed yang dapat dikelola, tanggal lahir atau perkiraan usia, gender, berat dengan riwayat ukur, microchip opsional, steril, alergi, kondisi kronis, vaksin, foto, pemilik lebih dari satu dan caretaker. Penghapusan pemilik/hewannya tidak menghapus rekam medis.

### 009.3 CRM, campaign, keluhan

Segmentasi berbasis rule yang dapat dihitung ulang (kunjungan, spend, layanan, inactivity, tier). Campaign hanya ke kontak yang consent-nya valid pada saat kirim; ada suppression, preview, test send, batas frekuensi, unsubscribe dan delivery log. Keluhan memiliki SLA per prioritas, assignee, status, catatan internal terpisah dari pesan customer dan audit resolusi.

---

## PETORA-FINAL-010 - Modul Katalog dan Persediaan

### 010.1 Produk dan harga

Produk memiliki SKU, barcode unique, variasi bila dibutuhkan, unit dasar, konversi unit beli, kategori, brand, supplier default, tax category, harga/discount floor, status jual/beli, prescription flag, tracking batch/expiry, dan gambar. Harga cabang efektif pada rentang tanggal; transaksi menggunakan snapshot. Service dipisah dari product tetapi dapat menjadi sales item dengan model pajak/komisi yang sama.

### 010.2 Persediaan

Gudang cabang dapat bertipe sale, backroom, transit, quarantine, damaged, consignment. Kartu stok menampilkan opening, inbound, outbound, balance, lot, biaya, referensi dan actor. Reservasi dipakai untuk order/booking bila barang benar-benar dijanjikan; expirynya dibersihkan/release otomatis secara idempotent. Batch expired dipindah/ditahan, tidak dijual.

### 010.3 Pembelian

Alur: permintaan pembelian opsional -> PO draft -> review/approval berdasar nilai -> kirim supplier -> penerimaan parsial -> quality hold/reject -> supplier bill -> alokasi pembayaran -> close. Match harga/qty terhadap PO; variance di atas threshold meminta approval. PO tidak menaikkan stok sebelum goods receipt.

### 010.4 Stock transfer dan opname

Transfer memiliki dispatch, transit, receiving, selisih, dan close. Opname dapat full/cycle count; snapshot awal dan freeze policy ditetapkan per gudang. Approval berbeda dari pelaksana untuk adjustment di atas batas. Semua adjustment memerlukan kode alasan (rusak, hilang, koreksi, expired) dan bukti bila diwajibkan.

### 010.5 Titipan/consignment

Barang titipan diidentifikasi pemilik dan saldo titipannya terpisah dari inventory milik perusahaan. Penjualan membukukan payable ke consignor sesuai perjanjian; retur/settlement dapat direkonsiliasi. Jangan menganggap barang titipan sebagai aset perusahaan untuk neraca.

---

## PETORA-FINAL-011 - Modul Penjualan dan Pembayaran

### 011.1 POS workflow

Pilih cabang/register dan buka shift -> scan/search item/service -> pilih customer/pet bila diperlukan -> server menghitung harga/promo/tax preview -> kasir konfirmasi -> pilih metode pembayaran -> server commit -> struk/notification diproses via outbox. UI menampilkan status `processing`, `completed`, atau `unknown`; untuk timeout, client mengulang dengan idempotency key yang sama atau mengambil status, tidak membuat key baru.

### 011.2 Shift dan kas

Shift menyimpan cashier, register, opening float, waktu buka/tutup, penjualan per payment, cash movement, expected cash, counted cash, variance, reviewer. Satu kasir/register hanya punya shift aktif sesuai aturan organisasi. Cash movement non-sale (petty cash/float) memerlukan kategori dan approval.

### 011.3 Pembayaran

Metode internal: cash, bank transfer, debit/credit, QRIS/e-wallet, voucher, loyalty, account receivable, lainnya. Integrasi gateway mendukung `created/pending/succeeded/failed/expired/refunded`; callback signature valid. Reconciliation membandingkan payment provider/settlement dengan transaksi internal. Data kartu sensitif tidak disimpan; hanya token/reference dan masked metadata jika diizinkan.

### 011.4 Retur/refund/void

Kasir mencari transaksi, pilih item dan qty yang belum direfund, alasan, kondisi barang dan refund method. Aturan approval dapat memakai threshold nominal dan role; pembuat tidak menyetujui refund sendiri di atas threshold. Refund payment asynchronous menunggu konfirmasi gateway; refund inventory dan jurnal mengikuti hasil penerimaan fisik dan hasil payment. Audit mencatat before/after.

### 011.5 Struk dan invoice

Nomor transaksi unik per cabang/periode. Struk menampilkan identitas perusahaan/cabang, nomor/waktu lokal, kasir, item, qty, harga, diskon, tax profile/rate yang berlaku, total, pembayaran, change, membership points jika relevan, serta footer config. Reprint memiliki penanda copy dan tidak membuat transaksi baru.

---

## PETORA-FINAL-012 - Modul Layanan dan Pet Hotel

### 012.1 Katalog layanan

Service memiliki tipe, cabang tersedia, species/size, harga, durasi, buffer, kapasitas, resource/staff requirement, deposit, cancellation rule, tax category dan komisi. Perubahan harga berlaku ke booking/transaksi baru; booking lama mempertahankan snapshot.

### 012.2 Booking

Sumber staff, portal, telepon/WA (dicatat oleh staff), walk-in. Cek ownership/consent pet, service capability, branch hours, slot, kapasitas staff/room, blackout, deposit dan conflict dalam transaksi. Booking punya confirmation, reminder, reschedule/cancel dengan alasan, check-in, in-progress, completion dan no-show. Reminder dikirim via outbox dan menghormati consent/channel.

### 012.3 Grooming/daycare

Check-in mengambil kondisi awal/foto, instruksi dan persetujuan layanan tambahan. Groomer memulai sesi, checklist tahap, catatan kondisi/insiden dan foto, lalu supervisor/customer handover. Add-on memerlukan persetujuan harga sebelum dilakukan kecuali tindakan keselamatan mendesak yang dicatat.

### 012.4 Boarding/pet hotel

Reservasi mengunci kapasitas kamar sesuai jenis/species/ukuran. Check-in mencatat kondisi dan barang bawaan, feeding/medication plan, kontak darurat dan consent. Log perawatan (makan, obat, toileting, berat, foto, insiden) bertimestamp dan actor. Check-out menghitung malam/jam dari policy snapshot, tambahan, deposit/refund dan invoice. Room masuk `cleaning` sebelum `available`; kamar isolasi hanya untuk status dan prosedur yang sesuai.

### 012.5 Transport/home service

Tentukan pickup/dropoff, rute/biaya, pet safety/kennel, driver/vehicle, status dispatch dan proof-of-delivery. Lokasi sensitif dibatasi akses. Estimasi ongkir provider tidak final sampai dikonfirmasi adapter.

---

## PETORA-FINAL-013 - Modul Klinik dan Farmasi

### 013.1 Klinik

Antrean dan triage -> konsultasi -> anamnesis/vital/pemeriksaan -> diagnosis/prosedur -> rencana/consent -> resep/lab -> finalisasi -> billing -> follow-up. Dokter dapat membuat draft autosave; finalisasi mengunci snapshot dan mencatat prescriber. Rekam klinis menyimpan satuan, timestamp dan author. Template form dapat dikonfigurasi dengan schema terkontrol, bukan arbitrary code.

### 013.2 Resep dan farmasi

Resep memiliki item produk/obat, dosis, route, frequency, duration, qty, refill, warning dan dokter. Dispense penuh/parsial memilih lot FEFO, memvalidasi expiry, alergi, jumlah sisa dan status resep; transaksi stock movement dan dispensation atomik. Label mencantumkan instruksi yang disetujui. Tidak ada pengeluaran obat prescription-required tanpa resep valid atau override berizin dengan alasan dan audit.

### 013.3 Lab, operasi, rawat inap

Lab order terkait rekam medis, sample ID, status collection/processing/result/verified dan unit/reference range; hasil hanya final setelah validasi petugas yang berwenang. Surgery case mencatat informed consent, tim, anesthesia, bahan, timeline, monitoring dan outcome. Hospital stay mencatat admission, observasi, obat, biaya, discharge dan serah terima. Data dapat dikoreksi via amendment.

### 013.4 Data customer dan safety

Customer portal menampilkan ringkasan/hasil yang secara eksplisit dibagikan oleh petugas; lampiran mempunyai permission terpisah. Aplikasi membantu dokumentasi, tidak mengeluarkan diagnosis atau saran medis otomatis. Peringatan obat/alergi harus menyatakan sumber dan status verifikasi.

---

## PETORA-FINAL-014 - Modul Membership dan Promosi

### 014.1 Membership

Tier memiliki ambang spend/benefit/periode dan versi efektif. Upgrade/downgrade terjadi dari aturan periodik atau manual approval; benefit yang sudah dipakai tidak dibatalkan diam-diam. Customer dapat melihat tier aktif dan syaratnya.

### 014.2 Loyalty

Ledger event `earn`, `redeem`, `expire`, `reversal`, `adjustment`; pembulatan dan expiry berdasarkan config versi. Redeem tidak dapat menghasilkan saldo negatif. Refund membalik earning yang terkait sesuai sisa saldo; bila poin telah dibelanjakan, kebijakan debt/penolakan refund ditetapkan config dan tampak ke operator.

### 014.3 Promo engine

Jenis: item/cart discount, voucher, bundle, BOGO, membership, birthday, campaign. Prioritas deterministik; setiap promo menyimpan versi rule yang diterapkan. Default: satu voucher per transaksi; diskon item + membership dapat digabung hanya sesuai rule; diskon bertingkat tidak boleh melebihi ceiling dan tidak menembus harga minimum tanpa override. UI menampilkan breakdown dan alasan promo tidak berlaku.

---

## PETORA-FINAL-015 - Modul SDM

### 015.1 Employee dan jadwal

Data pegawai, posisi/departemen, kontrak, penempatan cabang, dokumen, status dan emergency contact. Akses salary/bank/KTP terpisah. Jadwal shift, cuti, tukar shift, hari libur dan approval disimpan dengan histori versi.

### 015.2 Absensi

Clock-in/out melalui app dengan waktu server; geolocation hanya jika consent/policy mengaktifkan dan data lokasi disimpan minimum. Koreksi attendance perlu alasan/approval. Import perangkat eksternal mencatat batch dan sumber. Tidak menggunakan browser clock sebagai waktu otoritatif.

### 015.3 Komisi dan payroll

Commission rule berlaku efektif per staff/service/product/category dengan basis amount yang ditentukan (net sales sebelum/selepas pajak); snapshot entry dibuat saat sumber final. Payroll period: draft -> calculated -> reviewed -> approved -> paid/closed. Sistem menghitung komponen yang dikonfigurasi, menyimpan input snapshot dan formula/version, menghasilkan slip private. Approval terpisah dari payroll preparer. Integrasi transfer bank opsional dan bukan prasyarat jurnal.

### 015.4 Kepatuhan

Aturan statutori, BPJS, potongan, pajak, lembur dan format slip harus dapat diperbarui melalui versi konfigurasi/logic rilis dan ditinjau HR/finance. Konfigurasi bebas tidak boleh mengubah formula legal tanpa audit dan test.

---

## PETORA-FINAL-016 - Modul Keuangan dan Akuntansi

### 016.1 Double-entry

COA organisasi dapat di-seed dari template dan disesuaikan akuntan. Journal mempunyai source reference, branch, period, tanggal, status draft/posted/reversed; baris debit/credit non-negatif dan total seimbang. Posting memeriksa periode terbuka, akun postable, organisasi dan cabang. Manual journal memerlukan attachment/alasan dan approval sesuai threshold.

### 016.2 Mapping posting

- Sales settled: Dr kas/bank/clearing/AR sesuai payment, Cr revenue per kategori, Cr tax payable bila berlaku.
- COGS: Dr HPP, Cr inventory berdasarkan costing transaction.
- Refund: reversal proporsional revenue/tax/payment dan COGS/inventory sesuai kondisi barang.
- Goods receipt: Dr inventory/quality hold, Cr GRNI/AP berdasarkan kebijakan invoice matching.
- Supplier payment: Dr AP, Cr kas/bank.
- Payroll: Dr beban payroll per komponen, Cr payable/kas sesuai status pembayaran.
- Loyalty liability, voucher liability, customer deposit dan consignment payable dibukukan sesuai mapping COA.

Semua mapping akun disimpan di config registry bertipe dan harus lengkap sebelum sumber transaksi dapat auto-post. Jangan membuat jurnal dari job terpisah yang dapat terlewat tanpa outbox/reconciliation.

### 016.3 Piutang, utang, kas, rekonsiliasi

Credit account customer hanya jika diaktifkan, ada limit, due date, approval dan aging. Supplier bill dapat dicocokkan dengan PO/receipt dan partial payment. Bank/cash account memiliki statement import/settlement matching, unmatched queue dan rekonsiliasi periodik. Closing kasir bukan pengganti rekonsiliasi bank.

### 016.4 Pajak

Tax profile dan rate memiliki tanggal berlaku dan tipe inclusive/exclusive/exempt; tiap barang/service dapat dipetakan. Sistem menyimpan basis dan nominal pada snapshot. Tarif dan kewajiban legal tidak ditanam sebagai angka tetap; production launch diblokir sampai finance mengonfirmasi konfigurasi terbaru dan hasil uji pembulatan/laporan.

### 016.5 Laporan

Laporan utama: laba rugi, neraca, arus kas (metode yang dipilih/diungkapkan), trial balance, sales, tax, inventory valuation, stock aging/expiry, purchase, AR/AP aging, payroll, commission, booking/service, customer cohort. Semua laporan menyebut rentang tanggal, basis status, cabang, timezone, waktu generate dan sumber. Finance reports memakai posted journal; operational sales reports menyebut transaksi final/refund sesuai definisi.

---

## PETORA-FINAL-017 - Modul Operasional Tambahan

### 017.1 Aset dan biaya

Aset memiliki tag, lokasi, custodian, nilai perolehan, tanggal, umur, kondisi, jadwal maintenance dan ledger depresiasi bila akuntansi mengaktifkan. Pengeluaran memiliki kategori/akun, bukti, cabang, budget, submit/approval/pay status.

### 017.2 SOP/checklist

Template berversi, applicability cabang/role, jadwal, required response/photo, checklist execution dengan waktu server, pelaksana dan reviewer. Perubahan SOP tidak mengubah eksekusi historis.

### 017.3 Dokumen

Kategori, pemilik, akses, versi, expiry, retention dan storage path. File private; unduh melalui signed URL. Dokumen legal, karyawan, klinik dan customer memiliki role-specific policies.

### 017.4 Delivery dan online order

Order online hanya menampilkan katalog publik yang disetujui, harga/stok per cabang, terms dan consent. Checkout menciptakan order pending; stok direservasi dengan TTL. Payment webhook terverifikasi mengubah order menjadi confirmed; kegagalan/expiry melepaskan reservation. Fulfillment, packing, courier tracking, delivery proof, return dan refund mengikuti flow POS/purchasing yang sama; tidak ada stok kedua untuk kanal online.

### 017.5 Budget

Budget per periode/cabang/departemen/akun memiliki draft/review/approved/closed, actual dihitung dari posting finance, alert variance dan approval threshold. Perubahan budget setelah approved tercatat versi.

---

## PETORA-FINAL-018 - Super Admin dan Konfigurasi

### 018.1 Dua tipe admin

`Super Admin` adalah role operasional dengan akses ke health/config/integrations yang diberikan. `Platform secret owner` adalah tanggung jawab terbatas untuk secret/deployment dan tidak otomatis merupakan role database tambahan yang dapat melewati RLS. Semua tindakan admin memerlukan MFA dan audit.

### 018.2 Config registry dan resolusi

Setiap config key memiliki registry migration-seeded: key, tipe (boolean/integer/decimal/string/enum/json schema), default, min/max/enum, scope (org/cabang), unit, effective date bila perlu, dampak restart, sensitivity, modul, permission edit, dan validator. Urutan nilai: branch override -> organization value -> registry default. Key tak dikenal ditolak. Cache server maksimal 60 detik dengan version/invalidation; operasi finansial membaca nilai dan versinya dalam transaksi.

Nilai bisnis yang dapat dikelola panel mencakup: lead time/cancel/no-show, loyalty earning/redemption, promo policy, diskon ceiling, inventory alert/negative-stock policy, metode bayar aktif, tax profile reference, nomor dokumen, jam operasional, reminder, receipt content, approval thresholds, dan feature availability. Perubahan nilai yang mengubah akuntansi/stock/medical harus memerlukan permission khusus dan effective date.

### 018.3 Larangan config panel

Tidak dapat diedit dari panel: SQL/function code, RLS, daftar permission inti, origin/CORS, redirect Auth, service-role key, webhook verification logic, encryption keys, bucket policy, database URL, arbitrary HTML/JS, arbitrary network URL/webhook, dan batas keamanan sistem. Nilai-nilai tersebut berada di migration/deployment secret. Config tidak dipakai untuk menyimpan password atau API secret plaintext.

### 018.4 Integrasi dan rahasia

Panel menampilkan adapter/provider yang sudah didukung, field form terdefinisi, scope cabang, status aktif, test connection, last success/error redacted, webhook health dan rotasi credential. Secret ditulis melalui Edge Function terautentikasi ke Supabase Vault atau secret store yang disetujui; database menyimpan reference/metadata. Secret hanya dapat ditulis/rotate, tidak dapat dibaca ulang; UI menunjukkan masked value dan waktu rotasi. Adapter memakai allowlist host/provider, timeout, signature verification, dan proteksi SSRF. Credential environment-level (Supabase, hosting, encryption root key) hanya dikelola melalui secret manager/dashboard/CI, bukan panel bisnis.

### 018.5 Panel layar dan workflow

Menu: Configuration registry values, effective config preview by branch, history/diff/rollback as new version, Integrations, test logs, Feature access, audit log, job/outbox health, environment health, data export/restore status. Perubahan mempunyai preview nilai efektif, validasi schema, alasan, konfirmasi, audit, serta tombol rollback yang membuat versi baru. Panel menolak tindakan destructive/ambiguous.

### 018.6 Integrasi yang didukung

Provider dipilih saat implementasi berdasarkan kebutuhan dan ketersediaan akun: email provider (contoh Resend/SMTP), WhatsApp provider lokal yang disetujui, payment gateway (contoh Midtrans/Xendit), shipping, accounting export, dan storage backup R2. Tidak ada jaminan semua provider aktif secara default. Adapter contract: `send/test/webhook/reconcile`, error mapping, retry policy, sandbox flag dan capability matrix.

### 018.7 Config baseline dan syarat aktivasi produksi

Nilai di bawah disemai melalui migration. Perubahan nilai dilakukan lewat config service, berversi dan diaudit. Nominal uang config disimpan sebagai string integer Rupiah di JSON agar tidak terkena presisi floating point JavaScript.

| Config key                            | Tipe / scope                     | Default                                 | Aturan                                                                                                                                                                                             |
| ------------------------------------- | -------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organization.default_timezone`       | IANA timezone / organisasi       | `Asia/Jakarta`                          | Tiap cabang boleh override dengan timezone valid.                                                                                                                                                  |
| `pos.require_customer`                | boolean / organisasi-cabang      | `false`                                 | Modul klinik dapat mewajibkan customer dan pet terdaftar secara terpisah.                                                                                                                          |
| `pos.allow_negative_stock`            | boolean / organisasi             | `false`                                 | Override perlu permission owner, alasan, alert dan audit; jangan izinkan obat/lot expired.                                                                                                         |
| `pos.cashier_discount_limit_percent`  | decimal / role-cabang            | `0`                                     | Nilai lebih dari nol harus disetujui owner; floor price tetap berlaku.                                                                                                                             |
| `pos.refund_window_days`              | integer hari / organisasi        | `30`                                    | Perubahan hanya memengaruhi request refund baru.                                                                                                                                                   |
| `pos.refund_requires_approval`        | boolean / organisasi             | `true`                                  | Default semua refund perlu approval; batas nominal dapat memperketat, bukan menonaktifkan audit.                                                                                                   |
| `inventory.valuation_method`          | enum / organisasi                | `moving_average`                        | Tidak dapat diganti setelah transaksi pertama tanpa migrasi costing dan persetujuan finance.                                                                                                       |
| `inventory.expiry_alert_days`         | array integer / organisasi       | `[30,7,1]`                              | Alert hanya; expiry blocking tetap enforced saat penjualan/dispense.                                                                                                                               |
| `booking.minimum_lead_time_minutes`   | integer / cabang                 | `60`                                    | Service dapat memiliki nilai yang lebih ketat.                                                                                                                                                     |
| `booking.no_show_grace_minutes`       | integer / cabang                 | `30`                                    | Job menandai no-show setelah waktu lokal booking + grace.                                                                                                                                          |
| `booking.reminder_offsets_minutes`    | array integer / cabang           | `[1440,60]`                             | Notifikasi hanya jika consent/channel valid; dedupe key per booking+offset.                                                                                                                        |
| `boarding.billing_method`             | enum / cabang                    | `elapsed_24h`                           | Opsi `elapsed_24h` atau `local_night`; metode disimpan pada booking/stay snapshot dan terkunci saat check-in.                                                                                      |
| `boarding.billing_unit_hours`         | integer jam / cabang             | `24`                                    | Dipakai hanya untuk `elapsed_24h`; nilai positif, effective date berlaku untuk stay baru.                                                                                                          |
| `boarding.checkout_grace_minutes`     | integer menit / cabang           | `120`                                   | Dikurangkan sebelum pembulatan unit; nilai disimpan pada stay saat check-in.                                                                                                                       |
| `loyalty.enabled`                     | boolean / organisasi             | `false`                                 | Sebelum enable, konfigurasi ratio earning, value, expiry, rounding, eligible spend dan refund policy.                                                                                              |
| `promotion.max_vouchers_per_sale`     | integer / organisasi             | `1`                                     | Penggunaan voucher atomik; stacking promo mengikuti rule version.                                                                                                                                  |
| `tax.default_profile_id`              | UUID nullable / organisasi       | `null`                                  | **Go-live blocker** sampai finance mengisi profile dan memetakan setiap SKU/service ke tax category yang sah. Tidak ada rate legal bawaan.                                                         |
| `accounting.post_sales_automatically` | boolean / organisasi             | `true` setelah setup                    | Tidak boleh enable sebelum COA dan seluruh account mapping tervalidasi. Jika auto-post gagal, transaksi tetap rollback atau masuk flow pending yang eksplisit; tidak boleh diam-diam tanpa jurnal. |
| `documents.number_format`             | template allowlist / cabang-tipe | `{PREFIX}/{BRANCH}/{YYYY}/{MM}/{SEQ:6}` | Token allowlist saja; uniqueness tetap dijamin DB.                                                                                                                                                 |
| `notifications.enabled_channels`      | enum array / organisasi          | `[]`                                    | Provider off sampai credentials diuji; in-app internal dapat aktif tanpa external provider.                                                                                                        |
| `integrations.*.enabled`              | boolean / integrasi              | `false`                                 | Hanya adapter yang terdaftar; production credential terpisah dari sandbox.                                                                                                                         |
| `security.customer_portal_enabled`    | boolean / organisasi             | `false`                                 | Enable setelah Auth redirect, consent, RLS dan customer-link flow lulus test.                                                                                                                      |

Sebelum produksi, checklist konfigurasi wajib berstatus valid untuk timezone/cabang, sequence, role seed, COA/mapping, tax profile, batas discount/refund, payment method, inventory policy, receipt, consent/template, dan integration sandbox/production. Panel menampilkan blocker dan tidak menyediakan tombol bypass.

---

## PETORA-FINAL-019 - Workflow End-to-End

### 019.1 Penjualan kasir

1. Kasir login/MFA sesuai kebijakan, memilih cabang/register dan membuka shift dengan opening float.
2. Scan produk; server lookup memastikan product aktif, harga cabang dan stok; UI menunjukkan lot/expiry warning jika relevan.
3. Pilih customer/pet opsional, promo dan poin. Server preview menghitung ulang menggunakan versi config.
4. Kasir memilih metode. Untuk cash, amount/cashback divalidasi; untuk gateway, buat payment intent dan status pending.
5. Untuk penjualan internal/cash, `sales_create_atomic` commit transaksi, item snapshot, payment, stock movements/balances, loyalty, jurnal dan outbox.
6. Server mengembalikan transaction ID/number; UI hanya menyatakan sukses setelah commit. Receipt/notification diproses asynchronous dan dapat dicoba ulang.
7. Timeout: kasir mencari hasil memakai idempotency key; tidak submit transaksi baru.

### 019.2 Pembelian sampai pembayaran supplier

1. Reorder alert atau user membuat PO draft.
2. Validasi supplier, harga, tax, cabang/gudang dan budget; threshold memerlukan approver berbeda.
3. PO approved dikirim supplier; status ordered.
4. Goods receipt mencatat diterima, ditolak, lot, expiry, biaya dan quality hold; posting satu transaksi atomik memperbarui stok ledger dan GRNI/AP.
5. Invoice supplier dicocokkan PO/receipt; selisih di atas threshold ditahan untuk review.
6. Pembayaran dialokasikan ke bill; jurnal AP/kas; PO ditutup setelah seluruh qty/finance status diselesaikan.

### 019.3 Transfer dan stock count

Transfer dibuat -> approval bila perlu -> dispatch source (ledger keluar, stock in transit) -> receiving target -> variance claim/adjustment -> close. Stock count membuat snapshot -> freeze/track pergerakan saat count -> input actual -> reviewer menyetujui -> posting selisih sebagai movement -> tutup count. Retry tidak menggandakan movement.

### 019.4 Booking ke layanan dan checkout

Staff/portal mengirim kebutuhan -> cek kepemilikan, slot, kapasitas, harga -> booking pending/confirmed sesuai sumber -> deposit intent -> reminder -> check-in dan consent -> session/progress -> add-on approval -> complete -> invoice draft/POS transaction -> payment -> follow-up. Cancellation/no-show mengikuti deadline/fee snapshot.

### 019.5 Klinik ke farmasi

Registrasi/triage -> queue -> draft consult -> pemeriksaan dan diagnosis -> order lab/procedure/prescription -> validasi alergi dan informed consent -> dokter finalisasi (immutable snapshot) -> farmasi dispense sebagian/penuh dengan lock stok dan lot -> invoice/payment -> hasil dibagikan sesuai izin -> follow-up. Amend hanya oleh role berwenang dengan alasan.

### 019.6 Pet hotel

Booking kapasitas kamar -> deposit -> check-in dengan kondisi awal dan care plan -> log pemberian makan/obat/observasi -> incident/owner notice bila perlu -> check-out dan hitung tagihan menurut rate snapshot -> payment/refund deposit -> room cleaning -> release ke available setelah checklist.

### 019.7 Payroll

HR membuka period -> cutoff attendance/commission -> calculate menghasilkan input snapshot dan exceptions -> HR review/adjust beralasan -> approver lain menyetujui -> journal/payable -> disbursement/reconciliation -> slip dikirim private -> period closed. Re-run menghasilkan version baru hanya sebelum final; sesudah final perlu reopen berizin.

### 019.8 Konfigurasi/integrasi

Admin memilih key/provider -> panel menampilkan scope dan dampak -> validasi schema/test -> approval bila diperlukan -> secret write-only ke Vault -> active setelah test berhasil -> perubahan dicatat -> worker memakai versi baru. Gagal koneksi menandai degraded, bukan menghapus konfigurasi aktif yang terakhir valid.

---

## PETORA-FINAL-020 - Operasional dan Deployment

### 020.1 Lingkungan

Tiga lingkungan terpisah: local (Supabase CLI/Docker), staging, production. Setiap environment mempunyai project Supabase, hosting app, Auth redirect, bucket, secrets, analytics dan data terpisah. Production tidak menggunakan seed akun/data demo. Staging data sintetis/anonymized saja.

### 020.2 Hosting Vercel dan Cloudflare Pages

Keduanya merupakan target resmi untuk static Vite apps. Build command dan output directory sama; SPA fallback ke `index.html`; header security/CSP disesuaikan domain Supabase dan provider adapter. Vercel menjadi referensi konfigurasi CI; Cloudflare Pages dapat dipilih tanpa perubahan backend/data contract. Tiap environment menunjuk ke satu host utama; jika berpindah, update domain, CORS, Auth redirect, CSP, Sentry release dan smoke test. Jangan menempatkan rahasia di build-time public env.

### 020.3 Domain dan jaringan

Contoh domain: `staff.petora.example`, `pos.petora.example`, `app.petora.example`; API memakai domain Supabase default/custom yang didukung. DNS/TLS dikelola provider DNS. Redirect Auth allowlist hanya origin yang diperlukan. CORS allowlist tepat untuk app env. Webhook endpoint tidak berbagi rahasia dengan client. Hindari mengklaim proxy database langsung melalui Cloudflare/Vercel kecuali memang ada implementasi yang diuji.

### 020.4 Migrations dan CI/CD

Pull request wajib menjalankan: format/lint, typecheck, unit, DB migration test dari database kosong dan upgrade staging snapshot, RLS tests, build tiga app, Edge Function tests, dependency/security scan. Merge ke branch deployment menjalankan staging. Production deploy manual approval setelah staging acceptance, backup/restore point tersedia, migrations backward-compatible dan smoke tests. Edge Functions di-deploy melalui Supabase CLI; migration tidak diedit setelah dirilis. Untuk schema breaking, gunakan expand-migrate-contract.

### 020.5 Environment secret checklist

| Lokasi                 | Nilai                                                                                                                        | Eksposur                                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Frontend build         | Supabase URL, public/anon key, public Sentry DSN, env name                                                                   | boleh tampil di bundle                                      |
| Supabase Edge secrets  | service role (auto-managed), integration encryption/root secret jika diperlukan, Sentry server DSN, provider webhook secrets | server only                                                 |
| GitHub Actions secrets | deploy token/project refs dan hosting deploy token                                                                           | CI only, least privilege                                    |
| Supabase Vault         | integration secrets yang dikelola panel                                                                                      | database/service only, tidak dapat dibaca ulang via browser |

Tidak menaruh DB password, service key, provider key, token atau PII di git, issue, build log, URL, atau Sentry. Rotasi credentials dan prosedur revoke wajib tersedia.

### 020.6 Backup dan disaster recovery

Gunakan backup otomatis Supabase sesuai paket dan retensi yang dikonfirmasi. Atur logical backup terenkripsi ke object storage terpisah (mis. Cloudflare R2) dengan credential write-only/rotated, object lock bila tersedia, retention policy dan audit. Backup bukan dianggap valid sebelum restore diuji ke project terisolasi. Target awal operasional: RPO <= 24 jam dan RTO <= 8 jam, kecuali bisnis menyetujui target lain dan paket/platform mendukungnya. Uji restore sekurangnya per kuartal dan setelah perubahan kritis; dokumentasikan hasil, owner, waktu dan gap. Backup file Storage dan database harus dipulihkan konsisten.

### 020.7 Scheduled jobs

Jadwalkan outbox retry, reminder, expiry alert, reservation expiry, membership/points expiry, no-show, payroll period tasks, reconciliation dan backup verification memakai Supabase Cron/worker yang didukung project. Secret tidak disimpan di SQL migration/plaintext cron command. Job mempunyai lock/lease, idempotensi, timezone eksplisit, batas batch, retry dan `job_runs`. Jangan menjadwalkan ulang schema backup melalui Edge Function yang tidak punya mekanisme restore teruji.

### 020.8 Observability dan incident response

Log JSON dengan `request_id`, function/use case, duration, status, actor ID pseudonymous, branch ID dan error code. Redact email/telepon, medis, payroll, token, body payment. Monitor error rate, latency p95/p99, DB saturation/slow query, Edge invocations, storage, failed outbox, webhook failures, auth anomalies, backup age. Alert punya owner dan runbook untuk payment stuck, outbox backlog, stock discrepancy, suspected breach, restore, provider outage dan deployment rollback.

### 020.9 Prosedur go-live

Sebelum traffic produksi: verifikasi domain/TLS, Auth email/redirect/MFA, admin awal dibuat dengan prosedur aman, cabang dan timezone, tax/COA/config validasi finance, role matrix review, metode pembayaran/gateway sandbox-to-live, printer/perangkat per cabang, inventory opening balances dengan approval, template receipt, notification consent, migration dry-run, backup+restore proof, dashboard/alerts, support contacts dan smoke test transaksi/refund/receipt/booking. Opening stock dan saldo awal merupakan import yang direkonsiliasi dan diaudit.

---

## PETORA-FINAL-021 - QA dan Kriteria Penerimaan

### 021.1 Jenis test wajib

- Unit: money/tax rounding, promo precedence, loyalty, FEFO, booking overlap, payroll calculation, state machine.
- Database integration: migration kosong/upgrade, constraints, atomic rollback, sequence concurrency, idempotency, ledger balance, outbox.
- RLS/security: setiap role/cabang, cross-branch read/write, storage signed URL, unauthenticated access, direct RPC abuse, definer grants.
- Edge Function contract: malformed input, missing permission, duplicate key, payload mismatch, timeout/provider outage, webhook replay/signature invalid.
- E2E Playwright: login/MFA, open-close shift, POS success + retry + refund, purchase partial receipt, transfer, booking/grooming, medical finalize/dispense, payroll approve, config/integration update, portal consent.
- Accessibility: keyboard and automated axe on core routes.
- Performance/load: POS commit, booking concurrency, inventory list, dashboard/report; test dengan workload representatif dan batas platform project target.

### 021.2 Acceptance criteria per use case

Sebuah fitur dinyatakan diterima bila: permission server-side benar; branch scope teruji; input/output terdokumentasi; state transitions dan error dapat dipulihkan; audit/outbox dibuat tepat sekali; retry tidak menggandakan efek; data sensitif tidak bocor; loading/empty/error states ada; migration/rollback strategy dijelaskan; test lulus pada CI dan staging.

### 021.3 Acceptance gate produksi

Produk dinyatakan siap deploy hanya ketika seluruh syarat berikut terpenuhi:

1. Semua modul yang diaktifkan memiliki UI, backend, database, permission, audit, test, dan dokumentasi operator.
2. Tidak ada RLS table/access route yang belum direview; hasil test membuktikan isolasi cabang dan batas role.
3. POS, payment callback, inventory, refund, booking, medical, payroll dan journal flow atomik/idempotent sesuai kontrak.
4. Integrasi produksi berhasil diuji; retry, revoke/rotate dan outage behavior teruji.
5. Migrations dapat diterapkan dari database kosong dan staging ke production tanpa edit manual.
6. Backup dan pemulihan database serta storage telah dibuktikan dalam environment terisolasi sesuai target RPO/RTO.
7. Tidak ada critical/high security finding yang belum diterima tertulis oleh pemilik risiko.
8. Finance menyetujui pajak, COA, opening balances, report reconciliation dan mapping jurnal; HR menyetujui formula payroll; dokter penanggung jawab menyetujui workflow klinis.
9. Smoke test di domain/akun produksi tidak memakai data nyata yang tidak perlu; receipt dan timezone benar.
10. Runbook, support owner, alert, rotasi secret dan prosedur incident tersedia.

### 021.4 Target performa

Target awal pada staging dengan data realistis: interaksi biasa p95 <= 500 ms untuk API read sederhana, mutation bisnis p95 <= 1.5 s tanpa waktu provider eksternal, p99 <= 3 s; initial route usable <= 3 s pada koneksi kantor yang disepakati. Provider external, report besar dan PDF job asynchronous. Target final harus diuji terhadap tier Supabase/hosting yang dipilih; jangan menjanjikan jumlah concurrent/transaction yang tidak divalidasi oleh load test.

### 021.5 Data uji dan UAT

Gunakan seed deterministik dengan user role, cabang, produk batch, saldo awal, customer/pet, promo, PO dan transaksi yang konsisten. Dilarang memakai data pasien/customer nyata di CI. UAT ditandatangani per kelompok: owner/manager, kasir, inventory, layanan/klinik, finance/HR, super admin. Defect severity dan owner dicatat; seluruh blocker dan critical ditutup sebelum release.

---

## PETORA-FINAL-022 - Kamus, Register ID, dan Keputusan Terbuka

### 022.1 Definisi

| Istilah        | Definisi                                                                                      |
| -------------- | --------------------------------------------------------------------------------------------- |
| RLS            | PostgreSQL Row Level Security untuk batas akses per record.                                   |
| FEFO           | First Expired, First Out untuk lot yang memiliki expiry.                                      |
| Moving average | HPP rata-rata tertimbang yang diperbarui saat inbound secara atomik.                          |
| Outbox         | Catatan event/job yang commit bersama transaksi bisnis untuk pemrosesan asynchronous.         |
| Idempotency    | Mekanisme agar retry request yang sama tidak menggandakan efek.                               |
| Snapshot       | Nilai historis item/rule yang disalin saat transaksi agar tidak berubah karena master update. |
| Vault          | Penyimpanan secret terenkripsi yang hanya dapat diakses server sesuai izin.                   |
| Single-tenant  | Satu instalasi dan satu organisasi bisnis; tetap dapat memiliki banyak cabang.                |
| Reversal       | Transaksi koreksi bernilai lawan yang mempertahankan jejak sumber.                            |
| Signed URL     | URL sementara untuk akses file private dengan masa berlaku terbatas.                          |

### 022.2 Register kebutuhan utama

| ID            | Kebutuhan                                                | Verifikasi                     |
| ------------- | -------------------------------------------------------- | ------------------------------ |
| `REQ-PET-001` | Satu perusahaan, banyak cabang; bukan SaaS multi-tenant. | DB invariant + RLS test        |
| `REQ-PET-002` | UI staff/POS/customer sesuai perangkat.                  | Responsive/accessibility E2E   |
| `REQ-PET-003` | Mutasi kritis berpermission, audit, idempotensi.         | API dan DB integration tests   |
| `REQ-PET-004` | POS hanya final saat online; draft offline aman.         | Offline E2E                    |
| `REQ-PET-005` | Akses dan laporan sesuai cabang.                         | RLS matrix tests               |
| `REQ-PET-006` | PII/medis/payroll terlindungi.                           | Security review + access tests |

### 022.3 Keputusan yang sengaja tidak dibiarkan mengambang

- Frontend memilih satu provider per environment: Vercel atau Cloudflare Pages; stack tidak mengunci backend ke salah satunya.
- UUID v4 dipakai konsisten dengan generator PostgreSQL; tidak ada klaim UUID v7 tanpa generator terverifikasi.
- Config panel mengelola nilai bisnis terdaftar, bukan kode, secrets deployment, atau kontrol keamanan.
- POS final tidak berjalan offline.
- Tax rate dan kewajiban payroll bukan angka tetap di kode; finance/HR bertanggung jawab mengonfirmasi aturan yang berlaku sebelum produksi.
- Payment gateway, WhatsApp, email, shipping, printer dan accounting export memakai adapter yang telah diimplementasikan dan diuji, bukan plugin kode bebas.
- Penjualan, inventory, payment internal, loyalty dan accounting effects utama atomik di PostgreSQL; network provider dilakukan asynchronous melalui outbox/webhook.
- Data transaksi tidak dihapus; koreksi menggunakan reversal/amendment.

### 022.4 Checklist implementasi lintas dokumen

- [ ] Semua requirement/aturan mempunyai test atau alasan pengecualian.
- [ ] Setiap tabel/API/route memiliki ID atau rujukan ID stabil pada katalog implementasi.
- [ ] Seluruh tabel bisnis memiliki RLS, policy yang direview, indeks dan migration.
- [ ] Seluruh role default dan permission seeded, direview owner, diuji per cabang.
- [ ] Config registry berisi schema/default/scope/permission; UI panel menghasilkan audit/version.
- [ ] Integrasi secret write-only, rotasi, provider test, webhook verify dan retry berjalan.
- [ ] Posisi opening balance, inventory, COA, tax, document sequence sudah direkonsiliasi.
- [ ] CI, staging, deployment rollback, backup restore, alert dan runbook teruji.
- [ ] UAT dan sign-off owner domain (operasional, klinik, finance, HR) lengkap.

---

## Penetapan

`PETORA-FINAL-BLUEPRINT-000` adalah spesifikasi normatif produk dan deployment PETORA. Pengembangan harus mengikuti keputusan dan invariant di atas. Pengecualian hanya boleh dilakukan melalui keputusan perubahan terdokumentasi yang memperbarui bagian, requirement, database, API, UI, test dan prosedur operasi terkait. Spesifikasi ini menetapkan target readiness; implementasi tetap harus memenuhi seluruh acceptance gate sebelum dinyatakan siap produksi.

---

## PETORA-FINAL-023 - Kontrak Implementasi Terperinci

Bagian ini melengkapi bagian sebelumnya dengan nama field inti, state machine, kontrak request/response, perilaku layar, event, definisi laporan, dan contoh penerimaan yang mengikat. Jika suatu kolom diberi tipe atau constraint di sini, implementasi wajib merepresentasikannya di migration dan test; daftar ini tidak menggantikan RLS atau aturan bisnis pada bagian lain.

### 023.1 Konvensi field dan tipe database

Tabel bisnis selain ledger menggunakan kolom umum berikut kecuali ada pengecualian: `id uuid primary key default gen_random_uuid()`, `organization_id uuid not null`, `created_at timestamptz not null default now()`, `created_by uuid not null references auth.users(id)`, `updated_at timestamptz not null default now()`, `updated_by uuid null references auth.users(id)`, `archived_at timestamptz null`. Tabel scoped cabang menambah `branch_id uuid not null references branches(id)`. Jangan menambahkan kolom umum tersebut pada tabel append-only; gunakan field actor/waktu spesifik sesuai kejadian.

| Domain / field    | Tipe dan constraint normatif                                                                                                                                                            |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PK dan FK         | UUID, FK wajib; gunakan composite FK yang mencakup organisasi/cabang saat relasi berscope. Tidak ada FK ke label yang berubah.                                                          |
| Nominal IDR       | `bigint`; check >= 0 kecuali field selisih atau ledger yang signed. Nilai pada JSON/API dikirim sebagai integer JSON hanya dalam rentang aman; selain itu string desimal tanpa pemisah. |
| Kuantitas         | `numeric(18,4)`; pembulatan/precision dari unit produk; tidak menggunakan floating point untuk perhitungan bisnis.                                                                      |
| Rate/persentase   | `numeric(9,6)` atau precision spesifik yang ditetapkan migration; simpan definisi skala dan batas 0..100/rasio.                                                                         |
| Tanggal/waktu     | `date` untuk hari bisnis; `timestamptz` untuk kejadian; waktu booking disimpan dalam UTC dan menyimpan timezone cabang snapshot.                                                        |
| Status            | `text` + `CHECK`/lookup dan transition function. Jangan terima status baru dari request bebas.                                                                                          |
| Snapshot/rule     | `jsonb` hanya untuk payload terstruktur yang tervalidasi schema berversi; data yang dipakai join/filter tetap menjadi kolom relasional.                                                 |
| Identitas manusia | `code`, `number`, SKU, barcode memakai unique index dalam scope organisasi/cabang yang dinyatakan; bukan PK.                                                                            |

### 023.2 Kamus field master dan operasional

#### Organisasi, customer, pet

| Tabel                     | Kolom minimum dan constraint                                                                                                                                                                                                                                                                                |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organizations`           | `id`, `legal_name`, `display_name`, `business_type`, `country_code`, `default_timezone`, `default_locale`, `currency_code` (IDR), `tax_identifier_ciphertext NULL`, `is_active`, timestamps. Instalasi menolak organisasi kedua yang aktif.                                                                 |
| `branches`                | `id`, `organization_id`, `code`, `name`, `address_*`, `phone`, `email`, `timezone`, `operating_hours jsonb` tervalidasi, `is_active`; unique `(organization_id,code)`. `operating_hours` memakai hari ISO 1-7 dan interval lokal, mendukung tutup/libur khusus melalui tabel terpisah.                      |
| `profiles`                | `user_id uuid PK references auth.users(id)`, `organization_id`, `full_name`, `phone NULL`, `avatar_path NULL`, `status` (`invited`,`active`,`disabled`), `locale`, `last_seen_at NULL`. Auth email/password tidak diduplikasi di sini.                                                                      |
| `customers`               | `id`, `organization_id`, `customer_number`, `display_name`, `status` (`active`,`blocked`,`merged`), `merged_into_customer_id NULL`, `preferred_contact_channel`, `marketing_status`, `notes_internal NULL`, timestamps; unique nomor. Kontak berada di `customer_contacts`, consent di `customer_consents`. |
| `customer_contacts`       | `id`, `customer_id`, `type` (`phone`,`email`), `value_ciphertext`, `normalized_lookup_hash`, `masked_value`, `is_primary`, `verified_at NULL`, `valid_from/to`; lookup hash tidak dapat dipakai untuk mengirim pesan.                                                                                       |
| `customer_consents`       | `id`, `customer_id`, `channel`, `purpose`, `status` (`granted`,`withdrawn`,`denied`), `notice_version`, `source`, `recorded_at`, `recorded_by NULL`; withdrawal menambah event, tidak overwrite sejarah.                                                                                                    |
| `pets`                    | `id`, `organization_id`, `pet_number`, `name`, `species_id`, `breed_id NULL`, `sex`, `birth_date NULL`, `birth_date_is_estimated`, `color NULL`, `microchip_ciphertext NULL`, `sterilized_status`, `deceased_at NULL`, `notes_internal NULL`, timestamps; indeks nama dengan batas scope.                   |
| `pet_owners`              | `id`, `pet_id`, `customer_id`, `relationship`, `is_primary`, `valid_from`, `valid_to NULL`, `created_by`; constraint maksimal satu pemilik primary aktif per pet, perubahan ownership mempunyai event/audit.                                                                                                |
| `pet_weight_measurements` | `id`, `pet_id`, `weight_kg numeric(7,3)>0`, `measured_at`, `measured_by`, `source` (`clinic`,`boarding`,`manual`), `medical_record_id NULL`. Histori tidak disimpan overwrite di `pets`.                                                                                                                    |

#### Katalog, purchasing, dan inventory

| Tabel                                    | Kolom minimum dan constraint                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `products`                               | `id`, `organization_id`, `sku`, `barcode NULL`, `name`, `description NULL`, `category_id`, `brand_id NULL`, `base_unit_id`, `product_kind` (`stock`,`non_stock`,`bundle`,`consignment`), `tracks_batch`, `tracks_expiry`, `requires_prescription`, `tax_category_id`, `is_sellable`, `is_purchasable`, `archived_at`; SKU unik organisasi, barcode unik jika non-null. `minimum_price`/HPP bukan kolom editable POS. |
| `product_branch_settings`                | `product_id`, `branch_id`, `selling_price bigint>=0`, `minimum_price bigint>=0`, `reorder_point numeric(18,4)>=0`, `target_stock numeric(18,4)>=0`, `valid_from`, `valid_to NULL`, `approved_by`; tidak boleh ada rentang harga aktif overlap untuk product+branch.                                                                                                                                                  |
| `units` / `unit_conversions`             | Unit `code`, nama, precision; conversion `from_unit`, `to_base_factor numeric>0`, effective dates. Konversi yang dipakai transaksi disalin menjadi snapshot.                                                                                                                                                                                                                                                         |
| `purchase_orders`                        | `id`, org/branch, `po_number`, supplier, destination warehouse, `status`, order/expected dates, currency IDR, subtotal/discount/tax/shipping/grand total, approval actor/time, supplier terms snapshot, cancellation reason; jumlah header harus sama dengan line snapshot.                                                                                                                                          |
| `purchase_order_items`                   | `id`, PO, line number, product, description snapshot, ordered quantity, received quantity projection, unit snapshot, unit cost, discount/tax, line total, expected date; qty received tidak bisa melebihi ordered tanpa approved variance.                                                                                                                                                                           |
| `goods_receipts` / `goods_receipt_items` | Receipt references PO, receiving warehouse, received time/actor, status (`draft`,`posted`,`voided`); lines record accepted/rejected quantity, batch, expiry, cost, quality status. Hanya `posted` receipt menghasilkan movement.                                                                                                                                                                                     |
| `inventory_lots`                         | Kontrak sesuai `003.7`; tambahkan `source_receipt_item_id`, `quality_status` (`available`,`quarantine`,`recalled`,`expired`,`damaged`), `last_counted_at`. Lot quarantine/expired tidak tersedia untuk sale/dispense.                                                                                                                                                                                                |
| `stock_reservations`                     | `id`, `branch_id`, `warehouse_id`, `product_id`, `lot_id NULL`, `source_type/id`, `quantity>0`, `status` (`active`,`committed`,`released`,`expired`), `expires_at`; unique active source line/product; commit/release idempotent.                                                                                                                                                                                    |
| `stock_movements`                        | Kontrak sesuai `003.7`; `movement_type` allowlist: `receipt_in`, `sale_out`, `return_in`, `transfer_out`, `transfer_in`, `adjustment_in`, `adjustment_out`, `damage_out`, `expiry_out`, `dispense_out`, `reservation_commit`, `consignment_sale`; reservation saja tidak mencatat movement on-hand.                                                                                                                  |

#### Layanan, SDM, dan konfigurasi

| Tabel                    | Kolom minimum dan constraint                                                                                                                                                                                                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `services`               | `id`, org, `code`, `name`, `service_type`, `base_duration_minutes>0`, `buffer_minutes>=0`, `requires_pet`, `requires_staff`, `requires_room`, `tax_category_id`, `is_active`, version. Harga cabang dan species/size disimpan pada effective-dated service price.                      |
| `service_branch_prices`  | `service_id`, `branch_id`, `species_id NULL`, `size_code NULL`, `price bigint>=0`, `duration_override_minutes NULL`, `valid_from/to`; kombinasi aktif tidak overlap.                                                                                                                   |
| `booking_services`       | `booking_id`, service, price/name/duration/tax snapshot, qty, status; satu booking bisa berisi beberapa layanan, tiap layanan menyelesaikan statusnya sendiri.                                                                                                                         |
| `employees`              | `id`, `organization_id`, `employee_number`, `profile_id NULL`, `full_name`, `position_id`, `employment_type`, `join_date`, `end_date NULL`, `status`, `primary_branch_id`, `credential_status NULL`; unique employee number. Data bank/ID disimpan pada tabel terpisah berizin khusus. |
| `attendance_records`     | `id`, employee, branch, event (`clock_in`,`clock_out`,`break_start`,`break_end`), `occurred_at_server`, `source`, `device_ref NULL`, `correction_of NULL`, `approved_by NULL`, `reason NULL`; raw event immutable, koreksi berupa event tambahan.                                      |
| `payroll_runs`           | `id`, organization, period start/end, `version`, status, input cutoff, totals earning/deduction/net, formula version, created/reviewed/approved/paid actors and timestamps; unique employee+period pada `payroll_lines` final.                                                         |
| `payroll_lines`          | employee, salary snapshot, attendance snapshot, commission snapshot, allowance/deduction components, tax/benefit components, gross/net bigint, calculation detail schema version; restricted HR/finance access.                                                                        |
| `system_config_registry` | Kontrak sesuai `003.7`; setiap key mempunyai `owner_role`, `risk_level`, `requires_approval`, `validation_schema_version`, `default_value`, dan `deprecation_state`.                                                                                                                   |

### 023.3 State machine lintas modul

Semua transition dilakukan melalui use case API/RPC; client tidak mengubah kolom status langsung. Transition gagal dengan `409 INVALID_STATE_TRANSITION` bila status sumber tidak sesuai. Setiap transisi menulis `status_history`/audit dengan actor, waktu, alasan, dan request ID.

| Aggregate         | Status/transition legal                                                                                                                        | Efek saat transition final                                                                                                                                                                                                             |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sales transaction | `draft -> completed`; `draft -> voided`; `completed -> partially_refunded -> refunded`; `completed -> refunded` bila seluruh nilai direfund    | `completed` memposting payment, stock, loyalty dan jurnal. `voided` tidak menghasilkan sale. Refund membuat record baru dan reversal, tidak mengedit line sumber. Payment pending gateway berada di payment status, bukan status sale. |
| Payment           | `created -> pending -> succeeded`; `created/pending -> failed/expired/cancelled`; `succeeded -> partially_refunded -> refunded`                | Hanya callback/status provider tervalidasi yang menandai settlement eksternal. Uang internal tidak dianggap diterima sebelum `succeeded`.                                                                                              |
| Purchase order    | `draft -> pending_approval -> approved -> ordered -> partially_received -> received -> closed`; reject/cancel hanya pada status yang diizinkan | `received` menyatakan qty penuh diterima; PO dapat ditutup dengan short-close beralasan. Receipt terpisah menentukan stock posting.                                                                                                    |
| Goods receipt     | `draft -> posted`; `draft -> voided`                                                                                                           | `posted` immutable dan membuat movements; koreksi memakai return/reversal receipt.                                                                                                                                                     |
| Stock transfer    | `draft -> pending_approval -> approved -> dispatched -> partially_received -> received -> closed`; `draft/approved -> cancelled`               | Dispatch memindahkan saldo ke transit; receiving membukukan saldo tujuan; selisih memerlukan claim/adjustment terpisah.                                                                                                                |
| Booking           | `requested -> confirmed -> checked_in -> in_progress -> completed`; `requested/confirmed -> cancelled` atau `no_show`                          | `reschedule` bukan status final: simpan event reschedule dan versi slot baru; slot lama dilepas atomik. Deposit/payment status terpisah.                                                                                               |
| Grooming session  | `planned -> checked_in -> in_progress -> awaiting_pickup -> completed`; status non-final dapat `cancelled`                                     | Add-on yang mengubah harga memerlukan persetujuan customer dan snapshot sebelum dilakukan.                                                                                                                                             |
| Boarding stay     | `reserved -> checked_in -> active -> checkout_pending -> checked_out`; reservasi dapat `cancelled/no_show`                                     | Checkout mengunci perhitungan bill dan release kamar hanya setelah cleaning checklist.                                                                                                                                                 |
| Medical record    | `draft -> finalized`; final record tidak diedit. Amendment mempunyai lifecycle `draft -> signed -> superseded`                                 | `signed` amendment tidak menimpa isi lama; record asal menunjukkan ada amendment melalui relasi/version.                                                                                                                               |
| Prescription      | `active -> partially_dispensed -> dispensed`; `active/partial -> cancelled/expired`                                                            | Dispensing mengecek qty tersisa dan stok dalam satu transaksi. Setiap dispense adalah fakta terpisah.                                                                                                                                  |
| Journal entry     | `draft -> posted -> reversed`                                                                                                                  | `reversed` hanya bila reversal entry yang seimbang telah dipost; perubahan berikutnya dibuat entry baru. Periode closed menolak post.                                                                                                  |
| Payroll run       | `draft -> calculated -> review -> approved -> paid -> closed`; draft/calculated dapat `cancelled`; approved perlu reopen berizin               | Setiap calculate menghasilkan version; approval dan pembayaran dicatat terpisah; final line tidak di-overwrite.                                                                                                                        |
| Outbox job        | `pending -> processing -> succeeded`; gagal `retry_wait -> processing`, akhir `dead_letter`                                                    | Claim lease dan dedupe key mencegah worker paralel menggandakan side effect. Retry manual mencatat actor dan alasan.                                                                                                                   |

### 023.4 Kontrak request/response API per use case

Semua endpoint berada di `/functions/v1/{function-name}`. Field `branch_id` dari body tidak dipercaya sebelum dicek terhadap role assignment. Nominal pada request adalah Rupiah integer, kuantitas decimal string/number yang tervalidasi. Response transaksi mengembalikan canonical totals/status dan IDs, bukan hanya pesan sukses.

| Use case                    | Request minimum                                                                                                  | Respons canonical / aturan khusus                                                                                                                  |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `auth-invite-user`          | `email`, `full_name`, optional `employee_id`, `role_ids[]`, `branch_ids[]`, expiry                               | `invitation_id`, `expires_at`; admin tidak menentukan password. Idempotent per email+undangan aktif.                                               |
| `customer-create`           | nama, contacts, consent events, optional pet seed                                                                | `customer_id`, nomor customer; normalisasi kontak server-side dan duplicate warning/409 berdasarkan policy.                                        |
| `customer-merge`            | `source_customer_id`, `target_customer_id`, reason, expected versions                                            | merge summary; hanya permission khusus, lock kedua record, pindahkan relasi yang dapat dipindah dan audit seluruh efek.                            |
| `product-upsert`            | master fields, unit, tax category, tracking flags, branch price versions                                         | `product_id`, validation warnings; SKU/barcode conflicts 409; perubahan harga efektif tidak mengubah transaksi lama.                               |
| `inventory-adjust`          | `branch_id`, `warehouse_id`, product/lot, signed qty, reason code, note, evidence paths                          | movement ID, balance before/after, approval status; adjustment final immutable.                                                                    |
| `inventory-transfer`        | source/target warehouse, lines `{product_id,lot_id,quantity}`, reason                                            | transfer ID/number/status; source/target harus berbeda cabang/gudang dan scope actor valid.                                                        |
| `purchase-order-submit`     | supplier, branch, warehouse, expected date, lines, price/tax snapshot references                                 | PO ID, total, approval route; server hitung ulang total; submit tidak menaikkan stok.                                                              |
| `purchase-receive`          | PO ID, lines `{po_item_id,accepted_qty,rejected_qty,batch,expiry,quality_status}`                                | receipt ID, movements, outstanding qty; batch/expiry wajib sesuai master; duplicate key tidak menggandakan receipt.                                |
| `sales-preview`             | branch/register, cart lines, customer/pet, voucher, points                                                       | canonical item prices, discount/tax breakdown, warnings, `config_version`, `quote_expires_at`; tidak memesan stok/kuota kecuali disebut eksplisit. |
| `sales-create`              | `quote_id`, payment intents, shift, key, customer consent flags                                                  | transaction ID/number/status, totals, payment status, receipt ref; quote expired/price changed menghasilkan conflict dan preview ulang.            |
| `sales-refund`              | sale ID, lines `{sales_item_id,qty,amount}`, refund method, return disposition, reason, approver token/reference | refund ID/status, refund amount, stock return movement bila diterima; tidak melebihi sisa refundable quantity/amount.                              |
| `shift-open/close`          | register, opening float atau closing denomination count, cash movement refs                                      | shift ID, expected/counted/variance; hanya satu open shift sesuai unique constraint register/cashier policy.                                       |
| `booking-create`            | branch, customer/pet, service lines, requested slot, staff/resource preferences, source                          | booking ID/number, confirmed slot, price/deposit snapshot; conflict menyertakan slot alternatif yang tersedia tanpa membocorkan jadwal pribadi.    |
| `booking-transition`        | booking ID, action, expected version, reason, optional new slot                                                  | updated status/version, released/acquired reservation; compare-and-swap mencegah race.                                                             |
| `medical-record-save`       | encounter ID, expected version, clinical sections schema, attachments metadata                                   | draft version; hanya role klinis dapat membaca/menulis sesuai cabang.                                                                              |
| `medical-record-finalize`   | record ID, expected version, sign confirmation, diagnosis/treatment completeness                                 | finalized record number/version/hash; transaksi gagal jika prescription reference atau required field invalid.                                     |
| `pharmacy-dispense`         | prescription ID, lines `{prescription_item_id,lot_id,qty}`, pharmacist confirmation                              | dispense ID, remaining qty, stock movements, label ref; partial dispense didukung atomik.                                                          |
| `payroll-calculate`         | period ID, calculation version, include/exclude approved adjustments                                             | run ID/version, totals, exception list; tidak melakukan payment.                                                                                   |
| `payroll-approve/pay`       | run ID/version, approval decision/reason atau payment batch ref                                                  | approved/paid state dan journal/payment refs; maker-checker diberlakukan.                                                                          |
| `journal-post/reverse`      | journal ID/version atau source ID/reason/date                                                                    | posted/reversal IDs, debit-credit totals; period, mapping, balance dan duplicate source diperiksa DB-side.                                         |
| `config-update`             | key, scope ID, value, expected version, reason                                                                   | new version/effective value; optimistic concurrency `409 CONFIG_VERSION_CONFLICT`; sensitive config dapat memerlukan second approver.              |
| `integration-secret-rotate` | provider code, branch scope, secret input via TLS, sandbox flag                                                  | credential version, masked suffix, test result; secret tidak muncul dalam response/log; activation memerlukan test success.                        |
| `report-create`             | report code, period, branch scope, filters, output format                                                        | job ID, status, expiry; server validasi permission, maksimum rentang dan masking data.                                                             |

### 023.5 Kontrak layar dan interaksi

Setiap layar harus mendefinisikan permission route, branch context, filter tersimpan di URL bila aman, tabel/pagination, aksi utama, konfirmasi, loading/empty/error/forbidden/offline states, dan invalidation query key. Aksi mutasi menampilkan hasil server serta request ID untuk dukungan.

| Layar             | Data/filters utama                                  | Aksi dan validasi wajib                                                  | Tampilan/state khusus                                                      |
| ----------------- | --------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| Dashboard         | tanggal bisnis, cabang, perbandingan periode        | drill-down mempertahankan date/branch dan permission                     | skeleton, no data, stale timestamp, warning partial data                   |
| Customers/Pets    | nama/nomor/telepon, tag, status, cabang aktivitas   | create/edit, merge approval, ownership update, consent capture           | masking kontak, duplicate warning, timeline permission-filtered            |
| Product catalog   | SKU/barcode/name/category/active/stock branch       | create/archive, effective price, tax category, batch flags               | stock preview bukan editable balance; price change confirmation            |
| Inventory         | branch/warehouse/product/lot/expiry/status/date     | adjust, transfer, count, receiving                                       | ledger opening/closing balance; expired/quarantine warning; approval state |
| Purchase order    | supplier/status/date/branch/approver                | create, submit, approve/reject, receive, close short                     | line variance highlight, PO vs receipt reconciliation                      |
| POS               | barcode, category, customer, pet, discount, payment | add/remove, quote refresh, hold draft, pay, reprint                      | fixed cart/total panel, processing/unknown result, offline hard stop       |
| Refund            | transaction number/customer/date/status             | select residual item qty, disposition, reason, approval                  | original payment/return state separate; duplicate/refund cap visible       |
| Booking calendar  | branch/date/service/staff/room/status               | create, confirm, check-in, reschedule, cancel/no-show                    | timezone local, conflict feedback, capacity indicators                     |
| Grooming/boarding | queue, pet, assigned staff/room, service status     | intake, checklist, log care, incident, pickup/checkout                   | medical alerts shown minimum-necessary; photo upload progress              |
| Clinic encounter  | queue, pet, history, allergies, vitals, diagnosis   | draft autosave, prescription, procedure/lab, finalize/amend              | unsaved warning, signed lock, allergy acknowledgment, private attachment   |
| Pharmacy          | prescription status, product, lot, expiry, branch   | partial/full dispense, print label                                       | quantity remaining and FEFO suggestion; no expiry bypass normal path       |
| Finance           | period/branch/account/status/source                 | create/approve/post/reverse/reconcile/export                             | debit-credit balance, closed period lock, unposted queue                   |
| HR/payroll        | period, employee/branch/status                      | attendance correction, calculate, review, approve, pay                   | salary masked by default, calculation exceptions before approval           |
| Configuration     | key/category/scope/effective date/version           | edit, test, approve, rollback as new version                             | current/effective preview, dependency blocker, audit diff                  |
| Integration       | provider/scope/environment/status                   | configure, rotate, test, enable/disable, retry webhook                   | secret write-only, sandbox/live badge, redacted latest error               |
| Customer portal   | customer-linked pets, booking, invoice, consent     | profile/contact verify, booking, cancel per policy, download shared docs | accessible mobile UI, no internal notes, consent and loading states        |

### 023.6 Event catalog dan konsumen

Event ditulis ke outbox pada commit sumber. Payload memakai schema version, event ID UUID, aggregate ID/version, organization/branch, occurrence time, actor kind, correlation/request ID, serta data minimum. Consumer harus idempotent dan tidak memuat PII yang tidak diperlukan.

| Event                         | Producer                            | Consumer/side effect                                                            | Dedupe/retry                                                    |
| ----------------------------- | ----------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `sales.transaction.completed` | `sales_create_atomic`               | receipt job, loyalty/report projection, notification                            | unique event ID; payment state harus succeeded/internal settled |
| `sales.refund.completed`      | `sales_refund_atomic`               | payment refund, inventory return if accepted, accounting reversal, notification | refund ID sebagai dedupe; payment provider retry terpisah       |
| `inventory.stock.low`         | stock projection/job                | internal notification, reorder suggestion                                       | key product+branch+threshold-version; daily suppression         |
| `inventory.lot.expiring`      | scheduled query                     | inventory/pharmacy alert                                                        | key lot+alert offset; stop after lot depleted                   |
| `booking.confirmed`           | booking transition                  | customer reminder scheduling, staff queue                                       | booking version+template+offset                                 |
| `booking.cancelled`           | booking transition                  | release reservation, deposit policy, notify                                     | cancellation event ID; release idempotent                       |
| `service.completed`           | grooming/boarding/clinic completion | invoice draft, commission candidate, survey                                     | service completion version                                      |
| `medical_record.finalized`    | medical finalize RPC                | document/PDF, portal share workflow, follow-up queue                            | record ID+version; content stays private by default             |
| `pharmacy.dispensed`          | dispense RPC                        | label/receipt, stock alert, accounting                                          | dispense ID; never auto-send medical detail to external channel |
| `purchase.receipt.posted`     | receive RPC                         | stock projection, GRNI/AP, variance alert                                       | receipt ID unique                                               |
| `payroll.approved`            | payroll approval                    | journal/payable/slip generation                                                 | run ID+version                                                  |
| `integration.delivery.failed` | adapter worker                      | retry scheduler, admin health alert                                             | provider event/message ID; bounded retries then dead letter     |

### 023.7 Definisi laporan dan angka dashboard

Semua laporan menyatakan `as_of`, timezone, branch scope, status transaksi, basis tax/refund, dan rumus. Cabang tanpa akses tidak boleh muncul sebagai baris nol.

| Metric/report             | Definisi normatif                                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Gross sales               | Jumlah line gross sebelum diskon, tidak termasuk transaksi void; bukan angka penerimaan kas.                                   |
| Net sales                 | Gross dikurangi diskon dan refund sales; pajak ditampilkan terpisah; basis transaksi selesai sesuai tanggal bisnis cabang.     |
| Cash collected            | Payment berstatus succeeded yang diterima/settled pada rentang tanggal; payment pending tidak dihitung.                        |
| Average transaction value | Net sales dibagi jumlah transaksi completed yang tidak fully refunded; pembagi nol menghasilkan null, bukan nol palsu.         |
| COGS / gross margin       | COGS dari snapshot moving-average movement; margin = net revenue ex-tax - COGS. Tidak mencampur jasa tanpa biaya langsung.     |
| Inventory on-hand/value   | Saldo proyeksi per `as_of`; nilai memakai unit cost ledger, melaporkan lot quarantine/expired terpisah.                        |
| Booking utilization       | Menit/resource terbooking dibagi kapasitas menit operasional setelah jam tutup/blackout; canceled/no-show dilaporkan terpisah. |
| Customer active/retained  | Definisi aktivitas dan cohort period dikonfigurasi serta dilabeli; bukan sekadar customer login.                               |
| AR/AP aging               | Saldo open per due date pada cutoff; bucket 0-30, 31-60, 61-90, >90 hari, credit note/payment allocation diperhitungkan.       |
| Payroll total             | Total run approved berdasarkan snapshot run, bukan penjumlahan salary master saat ini.                                         |

### 023.8 Acceptance test yang harus ada

1. Dua kasir submit payload dan key sama bersamaan: tepat satu transaksi, satu set stock movements, satu jurnal, dan response identik.
2. Dua kasir membeli lot terakhir bersamaan: hanya satu berhasil; yang lain memperoleh `STOCK_INSUFFICIENT`; saldo dan ledger tetap cocok.
3. User cabang A membaca ID transaksi cabang B melalui list, direct ID, RPC, Realtime, Storage URL dan export: tidak ada data yang bocor.
4. Gateway mengirim webhook yang sama berulang, terlambat, salah signature dan out-of-order: hanya event valid memengaruhi payment state sekali.
5. Refund dua permintaan paralel untuk item yang sama: gabungan qty/amount tidak melewati saldo refundable; approval self-approval sesuai policy ditolak.
6. Dua booking memesan resource dan slot overlap: hanya satu reserve berhasil sesuai kapasitas dan timezone cabang.
7. Finalisasi rekam medis concurrent dari dua tab: versi kedua gagal conflict; record final tidak bisa diubah oleh direct REST/RPC selain amendment resmi.
8. Dispense bersamaan untuk lot terakhir/expiry/alergi: lock mencegah oversell; lot expired ditolak; warning alergi harus acknowledgment yang tercatat.
9. Posting jurnal tidak seimbang, periode tutup, account non-postable dan source duplicate: seluruhnya ditolak tanpa jurnal setengah jadi.
10. Config update memakai stale version atau schema salah: ditolak; effective value untuk branch dan audit before/after tetap benar.
11. Provider notification gagal berkali-kali: transaksi sumber tetap committed, outbox retry/dead-letter tercatat, operator dapat retry tanpa duplikasi pesan yang sudah terkirim.
12. Restore backup database dan storage ke project terisolasi: jumlah transaksi, file referensi, RLS dan fungsi kritis diverifikasi dengan checksum/report rekonsiliasi.

### 023.9 Aturan menjaga versi 1.0

Versi dokumen ini tetap `1.0.0` sebagai baseline yang diminta pemilik produk. Perbaikan editorial yang tidak mengubah kontrak tidak mengganti versi; perubahan perilaku, schema, API, status, permission, atau deployment memerlukan keputusan perubahan dengan ID baru serta pembaruan dokumen dan test terkait. File ini tetap merupakan satu-satunya blueprint PETORA.

---

## PETORA-FINAL-024 - Formula dan Kalkulasi Domain

Semua kalkulasi otoritatif berjalan server-side dengan decimal/integer arithmetic; UI hanya menampilkan preview dari formula dan versi yang sama. Tiap transaksi menyimpan input, rate, versi formula/config, hasil per line dan hasil agregat agar dapat direkonsiliasi setelah konfigurasi berubah.

### 024.1 Urutan hitung transaksi

Untuk setiap baris barang/jasa:

1. Konversi jumlah input ke unit dasar menggunakan faktor conversion snapshot; tolak jumlah <= 0 atau precision yang tidak diizinkan unit.
2. `gross_i = round_idr(quantity_base * unit_price_snapshot)`.
3. Terapkan diskon baris yang memenuhi rule dan permission; hasil `net_i = gross_i - item_discount_i` tidak boleh negatif atau di bawah minimum price tanpa override beralasan.
4. Bentuk basis eligible cart dari baris setelah diskon item; exclude baris yang ditandai non-discountable oleh rule/catalog.
5. Evaluasi membership, promosi, voucher, lalu manual cart discount dalam urutan prioritas config. Rule yang sama tidak dapat diaplikasikan dua kali; voucher kuota diklaim atomik.
6. Alokasikan diskon cart ke item eligible secara proporsional terhadap net baris sebelum cart discount. Pembulatan Rupiah menggunakan largest-remainder: floor setiap alokasi, sisa Rupiah diberikan menurut remainder terbesar, tie-break `sales_item_id` ascending. Total alokasi wajib sama tepat dengan cart discount.
7. Hitung taxable base per tax category setelah seluruh diskon teralokasi; lalu hitung tax sesuai inclusive/exclusive profile.
8. `grand_total = sum(line_net_after_discount + tax_line) + service_fee + shipping_fee + rounding_adjustment`. Nilai rounding default nol; pembulatan pembayaran hanya berlaku jika metode/aturan legal yang disetujui mensyaratkannya.
9. Bandingkan payment sukses/otorisasi internal terhadap grand total; selisih tunai di atas total menjadi `change_total`, bukan revenue. Kredit hanya masuk bila receivable policy aktif dan limit disetujui.

Tidak boleh memakai floating point IEEE-754 untuk uang atau costing. Gunakan PostgreSQL `numeric` pada perkalian/persentase lalu `round(...,0)` Rupiah; pembulatan setengah mengikuti PostgreSQL `round(numeric)` (half away from zero). Nilai total line dan header harus direkonsiliasi dalam RPC sebelum commit.

### 024.2 Pajak dan diskon

Tax profile menyimpan `calculation_mode` (`exclusive`, `inclusive`, `exempt`), rate effective-dated, tax category, rounding scope (`line`), dan mapping akun. Formula:

| Mode      | Basis/nominal per line                                                                                      |
| --------- | ----------------------------------------------------------------------------------------------------------- |
| Exclusive | `tax = round_idr(taxable_base * rate / 100)`; line payable = taxable base + tax.                            |
| Inclusive | `tax = round_idr(taxable_base_inclusive * rate / (100 + rate))`; line payable tetap taxable base inclusive. |
| Exempt    | Tax = 0; reason/category exempt disimpan pada snapshot jika diperlukan aturan pelaporan.                    |

Tax dihitung per line/tax category setelah alokasi diskon, lalu diakumulasikan; header tax adalah jumlah line tax dan tidak dihitung ulang dengan rate campuran. Tidak ada tax rate fallback diam-diam. Jika produk/service taxable tidak memiliki profile efektif pada tanggal transaksi, transaksi ditolak dengan `TAX_PROFILE_REQUIRED`.

Urutan stacking baku: harga efektif cabang -> harga grosir yang memenuhi qty -> membership benefit -> product promotion -> cart promotion -> satu voucher -> manual discount yang diizinkan role. Config dapat mengecualikan pasangan stacking, tetapi tidak dapat mengubah urutan prioritas atau melewati floor price tanpa permission khusus. Refund mengembalikan proporsi diskon/tax snapshot pada item yang direfund, bukan rate hari refund.

### 024.3 Kuantitas, costing, dan FEFO

Semua kuantitas dinormalisasi ke unit dasar sebelum stok berubah. Receiving mengonversi harga unit beli ke harga per unit dasar, termasuk alokasi biaya landed yang disetujui. Untuk moving average setelah penerimaan:

```text
new_qty = old_qty + received_qty
new_average_unit_cost = round_idr(
  (old_qty * old_average_unit_cost + received_qty * received_unit_cost) / new_qty
)
```

Jika saldo awal nol, average cost menjadi received unit cost. Return pembelian mengurangi qty dan memakai biaya lot sumber; stock adjustment masuk tanpa invoice memerlukan unit cost yang disetujui finance, sedangkan adjustment keluar memakai average cost saat posting. COGS outbound adalah `round_idr(quantity_base * average_unit_cost_at_posting)`; nilainya disalin ke stock movement dan jurnal.

Available quantity adalah `on_hand_qty - active_reserved_qty`. Reservation hanya mengurangi available, bukan on-hand/GL. Saat commit sale/dispense, lock lot dalam urutan `(expiry_date asc nulls last, received_at asc, id asc)`, pilih expiry terdekat yang belum expired, kurangi on-hand/reservation sesuai sumber, buat movement dan COGS dalam transaksi sama. Lot quarantine, recalled, expired, atau damaged tidak dapat dipakai. Transfer memindahkan nilai cost snapshot ke gudang transit lalu ke tujuan tanpa mengubah total nilai inventory organisasi.

### 024.4 Loyalty, membership, dan komisi

| Kalkulasi           | Formula normatif                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Eligible spend poin | Net merchandise/service amount setelah diskon yang eligible dan sebelum pajak; pengecualian kategori didefinisikan pada program/version. Shipping, gift card, deposit dan redemption bukan eligible secara default. |
| Poin earned         | `floor(eligible_spend_idr / spend_per_point_idr * tier_multiplier)`; hasil integer non-negatif, disimpan pada `loyalty_ledger`.                                                                                     |
| Nilai redeem        | `redeem_value = points * point_value_idr`; tidak melebihi saldo available atau `min(max_redemption_percent * eligible_total, eligible_total)`.                                                                      |
| Saldo poin          | Jumlah ledger signed yang tidak expired/reversed; cache saldo hanya proyeksi dan dapat direkonsiliasi. Poin expired memakai lot earn paling awal terlebih dahulu.                                                   |
| Komisi              | Basis default adalah net line revenue setelah discount, sebelum tax dan setelah refund/return proporsional; `commission = round_idr(basis * rate / 100)` atau fixed amount per unit sesuai rule snapshot.           |

Program harus menentukan spend threshold, earn rounding, point value, expiry clock, eligible categories, redemption cap dan refund behavior sebelum enable. Refund/reversal menambah ledger lawan yang merujuk earning/redemption asal; adjustment manual butuh reason, approval dan batas nominal. Komisi pada item yang direfund dibalik pada periode berjalan atau adjustment payroll terbuka; periode payroll closed menggunakan adjustment pada periode berikutnya, tidak mengedit slip final.

### 024.5 Booking, boarding, dan deposit

Slot layanan memakai waktu UTC dan tampil dalam zona cabang. Durasi slot adalah `service_duration + buffer_minutes`, dengan buffer nol sebagai default kecuali service menetapkan nilai. Resource reservation memeriksa jam buka, blackout, assignment staff, room capacity dan resource locks pada satu transaksi.

Boarding billing memakai rate snapshot dan konfigurasi cabang `boarding.billing_unit_hours` serta `boarding.checkout_grace_minutes`; initial default masing-masing 24 jam dan 120 menit, dapat diubah sebelum stay baru. Billable units:

```text
elapsed = max(0, checkout_at - check_in_at)
billable_units = max(1, ceil(max(0, elapsed - checkout_grace) / billing_unit))
boarding_charge = billable_units * rate_per_unit_snapshot
```

Perusahaan dapat memilih policy per-night berbasis tanggal lokal jika demikian praktik operasionalnya; policy tersebut menjadi `billing_method` berversi dan wajib dipilih sebelum menerima booking. Billing method tidak dapat diganti setelah check-in. Deposit adalah liabilitas sampai diaplikasikan ke invoice/refund; deposit tidak dihitung revenue saat diterima.

### 024.6 Shift kas, payment, dan rekonsiliasi

Expected cash dihitung `opening_float + cash_sales_collected - cash_change_given + cash_in - cash_out - cash_refunds`. Payment non-cash tidak masuk cash drawer. Selisih `counted_cash - expected_cash` disimpan apa adanya; threshold variance memerlukan approval dan cash adjustment journal terpisah. Tutup shift tidak mengubah payment status transaksi.

Untuk split payment, sum payment `succeeded` harus sama dengan grand total setelah change cash; payment `pending` tidak dianggap settle. Gateway intent mempunyai expiry dan provider reference unik. Refund gateway hanya dianggap settled setelah webhook/API provider terverifikasi; bila asynchronous, record refund tetap `pending` dan laporan membedakan requested dari settled.

### 024.7 Waktu bisnis dan cutoff laporan

`business_date` ditentukan dari event timestamp yang dikonversi ke timezone cabang pada saat kejadian dan disimpan sebagai snapshot. Perubahan timezone cabang berlaku untuk kejadian selanjutnya, tidak memindahkan transaksi historis. Rentang tanggal laporan bersifat inklusif untuk tanggal pilihan UI dan diterjemahkan menjadi interval UTC `[start_of_first_day, start_of_day_after_last_day)`. Dashboard yang belum selesai refresh menampilkan `as_of` agar tidak mencampur hari lokal dan UTC.

---

## PETORA-FINAL-025 - Model Otorisasi dan RLS

Bagian ini adalah kontrak otorisasi yang harus diterjemahkan menjadi policy PostgreSQL dan test negatif. RLS membatasi baris; permission menentukan operasi; field sensitif menggunakan RPC/view khusus karena RLS tidak menyamarkan kolom.

### 025.1 Sumber identitas dan helper otorisasi

- Identitas actor selalu `auth.uid()` dari JWT Supabase yang sudah diverifikasi. `user_role_assignments` menghubungkan actor ke role dan optional branch; role/permission aktif, profil aktif dan cabang aktif wajib diperiksa.
- Instalasi satu-organisasi menentukan `organization_id` melalui baris singleton aktif di `organizations`; client tidak memilih organisasi. Semua operasi menolak jika tidak ada tepat satu organisasi aktif.
- `has_permission(permission_code, branch_id)` mengembalikan boolean; `can_access_customer(customer_id, branch_id)` memeriksa hak profil customer dan aturan hubungan kerja; `can_access_medical_record(record_id)` mengecek permission klinis plus cabang. Helper harus `SECURITY DEFINER` dengan owner non-login, `search_path` terkunci (`pg_catalog` dan schema yang diperlukan saja), argumen eksplisit, tidak mengembalikan data, `EXECUTE` dicabut dari `public` lalu diberikan hanya ke role yang perlu.
- Helper tidak boleh menerima `user_id` dari client untuk memilih actor. Hanya `auth.uid()` yang dipakai. Bila RPC memerlukan service actor, endpoint internal membentuk konteks actor secara eksplisit, tervalidasi, dan diaudit.
- Hak admin database/service role tidak dipakai dalam sesi browser. Semua function `SECURITY DEFINER` mempunyai review privilege, test, dan daftar tabel yang boleh diakses.

### 025.2 Kelas data dan policy

| Kelas                    | Contoh tabel                                                                  | SELECT                                                                                                  | INSERT/UPDATE                                                                                     | DELETE                                                                     |
| ------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Organisasi/cabang master | `organizations`, `branches`                                                   | `org:organization:read` atau `org:branch:read`; hanya organisasi aktif dan cabang yang di-scope         | permission `create/update` + field allowlist dan scope; update sensitif perlu audit               | dilarang untuk baris yang pernah direferensikan; gunakan `is_active=false` |
| Master organisasi        | products, services, suppliers, COA                                            | permission domain read; company-wide sesuai role                                                        | `create/update/archive` eksplisit; branch price selalu cek assignment                             | hanya draft unused; archive, jangan hard delete                            |
| Transaksi cabang         | sales, booking, PO, stock movements                                           | permission read AND branch assignment (`branch_id is null` hanya jika scope perusahaan eksplisit)       | `create` melalui RPC; tabel final tidak menerima direct INSERT/UPDATE dari client                 | tidak pernah untuk fakta final                                             |
| Profil customer          | customers, pets, contacts                                                     | staff permission dan tujuan layanan; customer portal hanya baris yang terhubung via owner link verified | portal hanya field self-service allowlist; staff update permission; merge melalui function khusus | arsip/merge workflow, tidak cascade ke history                             |
| Klinis                   | records, diagnosis, prescription, lab                                         | `vet:medical_record:read`/permission khusus dan branch; pharmacist mendapat prescription minimum        | draft/final/amend/dispense hanya lewat RPC role khusus                                            | dilarang; retention/legal hold berlaku                                     |
| HR/payroll/PII           | employee private data, payroll lines                                          | `hr:employee:sensitive_read`/`hr:payroll:read` terpisah dari staff directory                            | dedicated HR/finance RPC, maker-checker                                                           | dilarang untuk payroll final/audit                                         |
| Finance                  | journal, bank, AP/AR, tax                                                     | permission finance per cabang/perusahaan                                                                | create/post/reverse terpisah; `post` melalui RPC                                                  | posted row tidak dapat dihapus                                             |
| Audit/config/jobs        | audit_logs, config values, outbox                                             | audit/config/job permission terpisah; data secret tidak tersedia                                        | audit/outbox internal saja; config via versioned function                                         | tidak oleh user aplikasi; retention purge hanya privileged job terkontrol  |
| Storage                  | bucket `customer-private`, `medical-private`, `hr-private`, `finance-private` | cek metadata file + parent record + purpose permission dan cabang                                       | upload signed policy; file ownership dan MIME/size tervalidasi server                             | delete hanya draft/unlinked; dokumen historis di-version/archive           |
| Public projection        | katalog online dan availability                                               | hanya RPC/view yang mengembalikan field allowlist tanpa PII/stock exact                                 | public `anon` tidak boleh mutate data transaksi                                                   | dilarang                                                                   |

### 025.3 Permission code registry kanonis

Permission berikut di-seed dalam tabel `permissions`; nama action tidak boleh dibuat bebas dari panel. `read_sensitive` dipisahkan dari `read` untuk PII/medis/HR. `approve`, `post`, `export`, dan `admin` adalah hak berbeda.

| Domain                       | Permission code minimum                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Organisasi/identitas         | `org:organization:read`, `org:organization:update`, `org:branch:read`, `org:branch:create`, `org:branch:update`, `identity:user:read`, `identity:user:invite`, `identity:user:disable`, `identity:user:role_assign`, `identity:session:revoke`, `identity:role:manage`                                                                                                                                                                                                                                                                                    |
| Config/security              | `settings:config:read`, `settings:config:update`, `settings:config:approve`, `settings:integration:read`, `settings:integration:update`, `settings:integration:test`, `settings:integration:rotate`, `audit:log:read`, `audit:log:export`, `system:health:read`, `system:job:retry`                                                                                                                                                                                                                                                                       |
| Customer/pet                 | `customer:customer:read`, `customer:customer:read_sensitive`, `customer:customer:create`, `customer:customer:update`, `customer:customer:merge`, `customer:customer:export`, `pet:pet:read`, `pet:pet:create`, `pet:pet:update`, `customer:consent:manage`                                                                                                                                                                                                                                                                                                |
| Catalog/inventory/purchase   | `catalog:product:read`, `catalog:product:manage`, `catalog:price:approve`, `inventory:stock:read`, `inventory:stock:adjust`, `inventory:count:create`, `inventory:count:approve`, `inventory:transfer:create`, `inventory:transfer:receive`, `purchase:order:create`, `purchase:order:approve`, `purchase:receipt:create`, `purchase:bill:read`, `purchase:payment:post`                                                                                                                                                                                  |
| POS/service                  | `pos:transaction:read`, `pos:transaction:create`, `pos:transaction:void`, `pos:refund:create`, `pos:refund:approve`, `pos:shift:open`, `pos:shift:close`, `pos:shift:reconcile`, `booking:booking:read`, `booking:booking:create`, `booking:booking:transition`, `service:session:manage`, `boarding:stay:manage`                                                                                                                                                                                                                                         |
| Klinik/farmasi               | `vet:queue:read`, `vet:medical_record:read`, `vet:medical_record:read_sensitive`, `vet:medical_record:create`, `vet:medical_record:finalize`, `vet:medical_record:amend`, `vet:medical_record:share`, `vet:lab:manage`, `vet:surgery:manage`, `pharmacy:prescription:read`, `pharmacy:dispense`, `pharmacy:controlled:manage`                                                                                                                                                                                                                             |
| SDM/finance/report           | `hr:employee:read`, `hr:employee:sensitive_read`, `hr:attendance:manage`, `hr:payroll:read`, `hr:payroll:calculate`, `hr:payroll:approve`, `hr:payroll:pay`, `finance:account:read`, `finance:journal:create`, `finance:journal:post`, `finance:journal:reverse`, `finance:cash:manage`, `finance:payable:manage`, `finance:receivable:manage`, `finance:tax:manage`, `report:operational:read`, `report:finance:read`, `report:clinical:read`, `report:export`                                                                                           |
| Membership/marketing         | `membership:tier:read`, `membership:tier:manage`, `loyalty:ledger:read`, `loyalty:redeem`, `loyalty:adjust`, `promotion:rule:read`, `promotion:rule:manage`, `promotion:rule:approve`, `promotion:voucher:issue`, `promotion:voucher:redeem`, `promotion:campaign:manage`, `promotion:campaign:send`                                                                                                                                                                                                                                                      |
| Operasional/dokumen/delivery | `operations:asset:read`, `operations:asset:manage`, `operations:checklist:template_manage`, `operations:checklist:execute`, `operations:expense:create`, `operations:expense:approve`, `budget:plan:read`, `budget:plan:manage`, `budget:plan:approve`, `document:document:read`, `document:document:upload`, `document:document:share`, `document:document:archive`, `delivery:order:read`, `delivery:order:manage`, `delivery:order:dispatch`, `delivery:order:complete`, `ecommerce:catalog:manage`, `ecommerce:order:read`, `ecommerce:order:fulfill` |

Role template hanya mengikat kumpulan kode di registry; permission baru memerlukan migration dan review. Maker-checker wajib untuk refund threshold, PO approval, adjustment material, payroll approval, jurnal manual, config high-risk, dan perubahan role/admin. Pemrakarsa tidak dapat menyetujui objeknya sendiri untuk tindakan tersebut.

### 025.4 Policy predicate wajib

Untuk setiap tabel API-accessible, migration menetapkan dan menguji `ENABLE ROW LEVEL SECURITY` serta `FORCE ROW LEVEL SECURITY` bila aman untuk fungsi dan owner setup. Policy SELECT mengharuskan identitas aktif + permission read + scope record; INSERT mengharuskan permission create + `WITH CHECK` organisasi/cabang valid; UPDATE memeriksa scope lama di `USING` dan scope baru di `WITH CHECK` agar record tidak dapat dipindah lintas cabang; DELETE dilarang untuk ledger/final records. Policy tak terdefinisi berarti akses ditolak.

Pola predicate konseptual:

```text
SELECT: has_permission('<domain>:<resource>:read', row.branch_id)
  AND row.organization_id = active_organization_id()
INSERT: has_permission('<domain>:<resource>:create', new.branch_id)
  AND new.organization_id = active_organization_id()
  AND branch_is_assigned(new.branch_id)
UPDATE: permission berlaku pada OLD dan NEW scope
  AND immutable_fields_unchanged
DELETE: false untuk transaksi, ledger, klinis final, payroll final, journal posted, audit
```

Data organisasi-wide yang dibutuhkan customer lintas cabang (mis. profil customer) tidak otomatis membuka riwayat transaksinya. History tetap branch-scoped dan memakai permission per domain. Customer portal mengakses parent/children hanya dengan `auth.uid()` yang terhubung ke customer verified dan relasi pet-owner aktif. Storage object path tidak menjadi bukti otorisasi; keputusan selalu dari metadata row dan parent record.

### 025.5 Audit/read/export dan RLS test matrix

Setiap test menguji Supabase REST, RPC direct call, Edge Function, Realtime publication, Storage download/upload, dan report/export. Minimal principal: anon, customer A, cashier branch A, cashier branch B, branch manager A, vet A, pharmacist A, finance A, HR, super admin, owner. Assertions meliputi: tak dapat mengganti `branch_id`, `organization_id`, customer/pet owner, actor, status final, journal account, amount, secret ref, atau `created_by` melalui request.

Ekspor customer, medical, HR, finance atau audit memerlukan permission `export` khusus, mencatat filter/row count/actor/request ID, membatasi rentang dan ukuran, menyamarkan field yang tidak diperlukan serta menyimpan file private dengan expiry. Audit viewer tidak dapat mengubah atau menghapus audit rows.

---

## PETORA-FINAL-026 - Runbook Deployment Produksi

Bagian ini menetapkan prosedur yang dapat dijalankan untuk deploy Vercel atau Cloudflare Pages dengan backend Supabase. Satu environment menggunakan satu provider frontend aktif. Perintah CI memakai tool/version yang dikunci di lockfile; contoh di bawah adalah urutan dan variabel, bukan kredensial.

### 026.1 Konfigurasi project dan domain

Setiap app dibuat sebagai project hosting terpisah (`staff`, `pos`, `customer`) agar release/rollback dan CSP dapat diatur mandiri. Build root tetap monorepo. Domain production/staging harus memetakan satu app ke satu origin; API/Auth/Storage tetap Supabase.

| App      | Build command                          | Output directory     | SPA fallback                          |
| -------- | -------------------------------------- | -------------------- | ------------------------------------- |
| Staff    | `pnpm --filter @petora/staff build`    | `apps/staff/dist`    | semua route non-file ke `/index.html` |
| POS      | `pnpm --filter @petora/pos build`      | `apps/pos/dist`      | semua route non-file ke `/index.html` |
| Customer | `pnpm --filter @petora/customer build` | `apps/customer/dist` | semua route non-file ke `/index.html` |

Project Supabase memiliki local/staging/production tersendiri. Untuk setiap environment tetapkan: project ref, API URL, public key, allowed Auth redirect origins, CORS origins, storage buckets, Edge secrets, provider sandbox/live credentials, Sentry environment, job schedule, backup destination, custom domain dan pemilik on-call. Tidak ada secret yang dibagikan antar environment kecuali root key platform yang dirotasi dan disetujui.

### 026.2 Hosting Vercel

Untuk tiap app, konfigurasi root directory repo, install `pnpm install --frozen-lockfile`, command app sesuai tabel, dan output directory. Set environment variables di scope preview/staging/production secara terpisah. SPA fallback mengarahkan route app ke index tanpa mengubah file/static asset; hindari rewrite API Supabase ke origin frontend. Header minimum: `Strict-Transport-Security` pada domain HTTPS, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `X-Frame-Options: DENY`/`frame-ancestors 'none'`, `Permissions-Policy` default deny, dan CSP allowlist origin app/Supabase/provider yang diperlukan. CSP tidak menggunakan `unsafe-eval`; inline style/script hanya jika build shadcn/Vite membuktikan perlu dan pengecualian diberi batas.

### 026.3 Hosting Cloudflare Pages

Buat project Pages per app dengan root repo, command/output yang sama, build variables yang sama-sama publik saja, dan branch mapping staging/production. Masukkan `_redirects` untuk SPA fallback dan `_headers` untuk security headers ke output masing-masing app; verifikasi bahwa direct URL seperti `/staff/inventory/stock` atau `/pos/sale` kembali ke app yang benar dan asset tidak tertimpa fallback. Jangan mengaktifkan Cloudflare Worker/proxy untuk API/database kecuali ada desain dan test terpisah. Pages Functions tidak menjadi tempat business logic; seluruh domain backend tetap di Supabase Edge Functions/Postgres.

### 026.4 Build variables dan secrets per environment

| Nama variable                                 | Lokasi                      | Ketentuan                                                |
| --------------------------------------------- | --------------------------- | -------------------------------------------------------- |
| `VITE_SUPABASE_URL`                           | build app                   | URL project environment tersebut, nilai publik           |
| `VITE_SUPABASE_ANON_KEY` atau publishable key | build app                   | nilai publik; RLS tetap wajib                            |
| `VITE_APP_ENV`                                | build app                   | `local`, `staging`, atau `production`; non-secret        |
| `VITE_SENTRY_DSN`                             | build app                   | DSN publik bila SDK frontend memerlukannya               |
| `SUPABASE_ACCESS_TOKEN`                       | CI only                     | deploy automation; masked, minimum scope, rotate         |
| `SUPABASE_PROJECT_REF`                        | CI variable                 | project ref environment, bukan password                  |
| provider/webhook/encryption secrets           | Supabase Edge secrets/Vault | server-only; rotasi dan no-log                           |
| hosting deploy token                          | CI secret                   | scope project saja; jangan expose pada preview dari fork |

CI harus gagal jika production build mendeteksi key yang berisi `service_role`, `secret`, `password`, private key, atau value yang tidak termasuk allowlist publik. Source map upload memakai token Sentry khusus CI dan map tidak dipublikasikan sebagai asset tanpa sengaja.

### 026.5 Urutan first deployment

1. Owner menetapkan domain, platform frontend pilihan, Supabase region/plan, owner akun, billing alert, support contacts dan recovery contacts.
2. Buat project Supabase staging/production terpisah; aktifkan MFA untuk pemilik platform; atur Auth email provider, redirect allowlist, MFA, rate limit dan email templates.
3. CI menjalankan migration dari database kosong; periksa extension yang didukung oleh tier/project, grants/RLS, storage bucket private, helper function grants dan seed reference. Tidak pernah mengisi password/service key ke migration.
4. Seed role/permission/config registry, satu organisasi, cabang, gudang, register, COA template dan document sequences. User pertama dibuat melalui invitation satu-kali; hapus/disable bootstrap path setelah selesai.
5. Simpan Edge secrets/Vault, deploy functions, jalankan health checks, test outbox/scheduler dan tes koneksi provider sandbox. Aktifkan provider live hanya setelah test dan owner approval.
6. Deploy frontend staging, set redirect/CORS/CSP origins, lakukan full UAT dan security/performance checks.
7. Buat restore point dan bukti backup; promote migration kompatibel ke production, deploy functions, deploy app, jalankan smoke test non-destruktif, lalu enable traffic/portal/payment secara terkendali.
8. Rekonsiliasi opening inventory, receivable/payable, cash/bank, tax profile, staff, produk dan harga dengan sumber yang disetujui; catat import batch dan checksum.

### 026.6 Urutan release normal

PR -> CI lengkap -> migration review -> deploy staging -> UAT domain owner -> backup/restore point -> manual production approval -> apply backward-compatible migration -> deploy Edge Functions -> deploy frontend immutable build -> health/smoke tests -> monitor canary period -> tutup release record. Feature configuration untuk perubahan berisiko dimulai disabled dan hanya diaktifkan setelah data/schema serta runbook siap.

Migration memakai expand/contract: tambah struktur kompatibel, deploy code yang membaca/menulis sesuai tahap, backfill terukur/idempotent, verifikasi, baru hapus struktur lama pada release terpisah setelah tidak ada consumer. Migration produksi tidak dijalankan manual dari laptop; CI menggunakan project ref/secrets environment dan merekam migration ID/log tanpa secret.

### 026.7 Rollback dan kegagalan deployment

- Frontend: kembalikan immutable deployment sebelumnya pada platform yang sama; jangan mengubah schema untuk rollback app.
- Edge Function: redeploy versi dari commit terakhir yang diketahui baik; kompatibilitas function/schema dijaga selama window rollback.
- Database: tidak ada `db reset` atau drop data di production. Utamakan forward-fix migration. Reversal migration hanya untuk perubahan additive yang dapat dibalik tanpa kehilangan data dan telah diuji. Data corruption ditangani restore plan oleh incident commander setelah evaluasi kehilangan data/RPO.
- Integrasi bermasalah: disable adapter via config, pertahankan transaksi internal, antrekan retry/dead-letter dan gunakan prosedur manual yang direkonsiliasi.
- Payment incident: hentikan metode provider terdampak, jangan tandai pending sebagai paid, simpan provider refs, rekonsiliasi sebelum retry/refund.
- Security incident: revoke/rotate token, tutup origin/provider yang terdampak, pertahankan audit/evidence, batasi akses dan jalankan incident response; jangan menghapus log.

Setiap rollback mencatat release ID, waktu, pelaksana/approver, alasan, status transaksi terdampak, migration/function/frontend version, hasil smoke test dan keputusan melanjutkan traffic.

### 026.8 Smoke test dan bukti operasional

Sesudah deploy, tes `/health` database/function readiness tanpa membocorkan detail, login/MFA, branch scope, list data kosong, create/void draft, sales sandbox/internal dengan produk uji yang direkonsiliasi, receipt preview, booking slot, private file access, config read, outbox worker, notification sandbox dan observability alert. Jangan membuat transaksi finansial fiktif di production; bila perlu tes nominal, lakukan transaksi terotorisasi dan reversal dengan audit.

Release dianggap selesai setelah: migration version cocok, semua app asset berasal dari release yang sama, Edge Functions health, Auth redirects, RLS smoke checks, provider webhooks, backup age, Sentry release, error budget dan job backlog diperiksa. Lampirkan bukti log/URL dashboard pada release record tanpa PII/secret.

### 026.9 Operasi harian dan checklist cabang

Pembukaan cabang: cek jadwal/holiday, kasir/register, opening cash, printer/barcode, konektivitas, stok minimum dan queue integrasi. Penutupan: close seluruh shift, rekonsiliasi cash variance, transaksi/payment pending, booking/stay aktif, stock receiving/transit, pending approval dan outbox. System health menampilkan cabang terdampak dan owner follow-up.

Operator tidak mengedit saldo, jurnal, status final, permission, atau config langsung di SQL/dashboard. Tindakan pemulihan dilakukan melalui UI/RPC administratif yang tercatat atau change procedure yang disetujui dan disertai reconciliation setelahnya.

---

## PETORA-FINAL-027 - Kontrak Modul, Fungsi, dan Workflow

Bagian ini menetapkan kontrak operasional seragam bagi seluruh modul. Setiap fungsi mutasi mengikuti API/RPC pada `PETORA-FINAL-005`; setiap rule angka mengikuti `PETORA-FINAL-024`; setiap akses mengikuti permission dan RLS `PETORA-FINAL-025`. UI tidak menulis status atau saldo langsung.

### 027.1 Template wajib untuk implementasi modul

Shorthand seperti `inventory:*` pada uraian modul berarti permission terdaftar dalam keluarga tersebut, bukan wildcard permission yang disimpan di database. Setiap module feature wajib mendefinisikan: owner, actor/permission, query dan command, input/output, state transition, precondition, transaction boundary, event/outbox, failure/retry behavior, audit fields, dan acceptance tests. “CRUD” tidak cukup sebagai spesifikasi fungsi: operasi yang final/approved/post harus menggunakan command bernama dan transisi legal. Semua list memakai branch/filter scope, pagination, sorting allowlist, empty/loading/error/forbidden states.

Konvensi command: `create` membuat draft/intent; `submit` mengunci draft dan memulai approval; `approve/reject` mencatat actor terpisah; `post/complete/finalize` membukukan efek immutable; `cancel/void` hanya pada status yang diizinkan; `reverse/amend` membuat fakta koreksi baru. Setiap command mengirim `expected_version`, `Idempotency-Key` untuk operasi retryable, dan reason saat menolak/mengoreksi.

### 027.2 `MOD-PET-ORG-001` - Organisasi, cabang, identitas, dashboard

**Actor/permission:** owner dan super admin untuk setup; `org:*`, `identity:user:*`, `identity:role:manage`; branch manager hanya cabangnya. MFA wajib untuk admin.

**Fungsi/layar:** setup organisasi; branch list/detail/jam operasional/hari libur; register dan warehouse setup; user invitation/resend/revoke; user status/session revoke; role/permission assignment; dashboard per cabang/periode. Form edit tidak menampilkan service-role atau provider secrets.

**Workflow:** create branch draft -> validasi code/timezone/address -> aktifkan branch -> buat default warehouse/register -> assign manager -> configure operating hours. User: invite -> invited -> email verified/credential setup -> active -> disabled/revoke sessions. Cabang tidak dapat deactivate jika memiliki shift, stay, booking, transfer atau approval aktif.

**Integrasi/aturan:** branch timezone menjadi sumber tampilan dan `business_date`; perubahan timezone berlaku untuk event baru. Role change berlaku setelah commit dan mencabut cache/session permission berikutnya. Dashboard hanya memakai read model; setiap KPI mempunyai definisi 023.7 dan timestamp `as_of`.

**Acceptance:** user disabled tidak dapat login atau memanggil function; user branch A gagal query/write branch B; cabang tidak bisa ditutup selama ada stay/transaksi aktif; perubahan role/config tercatat sebelum/after dan actor.

### 027.3 `MOD-PET-CUSTOMER-001` - Customer, pet, consent, CRM, keluhan

**Actor/permission:** customer service/front desk dan role customer sesuai `customer:*`, `pet:*`, `customer:consent:manage`; akses medical detail memerlukan permission klinis terpisah.

**Fungsi/layar:** cari berdasarkan nomor/nama/kontak normalized; create/edit profile; create pet; owner/caretaker link; vaksin/berat; tag/segment; history timeline; merge duplikat; consent center; campaign preview/send; complaint ticket, assignee, SLA, komunikasi dan resolution.

**Workflow:** create customer -> normalize contact -> exact duplicate check -> verify contact/identity -> create/link pet -> consent event -> aktivitas/transaksi. Merge: lock source+target -> preview relasi dan konflik -> approval -> reparent relasi yang legal -> mark source merged -> audit; tidak menggabungkan ledger ke saldo baru tanpa reversal/rebuild. Complaint: open -> triaged -> assigned -> investigating -> waiting_customer/resolved -> closed/reopened; setiap status memiliki owner dan timestamp.

**Validasi/efek:** contact tidak unik secara global bila keluarga berbagi nomor; duplicate handling menampilkan kandidat dan meminta operator memilih, bukan auto-merge. Owner aktif minimal satu; hanya primary satu pada satu waktu. Consent withdrawal langsung suppress campaign queue; queued message yang belum terkirim dibatalkan. Segmentasi tidak memasukkan data klinis/PII tanpa permission.

**Acceptance:** customer portal tidak dapat mengganti `customer_id`; merge tidak mengubah referensi penjualan/medical; consent withdrawn mencegah scheduled broadcast; pet dengan lebih dari satu owner tetap memiliki riwayat/izin yang benar.

### 027.4 `MOD-PET-INVENTORY-001` - Katalog, purchasing, inventory, supplier

**Actor/permission:** product admin, inventory staff, purchasing staff, branch manager/approver; permission `catalog:*`, `inventory:*`, `purchase:*` dari 025.3.

**Fungsi/layar:** product/category/unit/brand; harga efektif per branch; supplier; stock by warehouse/lot; reorder alerts; purchase request/PO; receipt/quality inspection; vendor bill/match; transfer; count/opname; adjustment request/approval; consignment settlement.

**Workflow pembelian:** draft PO -> submit -> approval berdasar nilai/variance -> ordered -> receive one/more shipments -> quality accepted/rejected -> post receipt -> match invoice -> approve bill -> allocate payment -> close. Draft receipt tidak memengaruhi stok. Short-close memerlukan reason dan approval. Supplier return merujuk receipt/lot awal dan membuat outbound movement.

**Workflow stok:** reservation create/expire/release -> sale/dispense commit -> append movement + update projection; transfer draft -> approved -> dispatch to transit -> receive target -> discrepancy resolution -> close. Count draft snapshot -> count lines -> freeze/capture intervening movements -> review variance -> approve -> post adjustment -> reconcile count.

**Validasi/efek:** harga dan unit conversion effective-dated; tracked batch/expiry wajib saat receipt; FEFO dan quality status diterapkan saat outbound. Ubah SKU/barcode yang dipakai historis harus mempertahankan alias. Transfer cabang beda memerlukan scope kedua cabang atau approval terpusat. Reorder suggestion tidak membuat PO otomatis tanpa setting/approval eksplisit.

**Acceptance:** receipt retry tidak menambah stok ganda; concurrent sales tidak oversell; expired/quarantine ditolak; transfer partial tetap menunjukkan transit; stock balance cocok dengan jumlah ledger pada rekonsiliasi.

### 027.5 `MOD-PET-SALES-001` - POS, shift, pembayaran, refund

**Actor/permission:** cashier/register scope; manager untuk override/approval; finance untuk settlement; permission `pos:transaction:*`, `pos:shift:*`, `pos:refund:*`.

**Fungsi/layar:** open/close shift; scan/search/add/remove item; customer/pet lookup; quote/price recalculation; promo/redeem; cash/split/gateway tender; hold draft; transaction history; receipt/reprint; void draft; refund/return; payment reconciliation.

**Workflow:** open shift -> draft cart -> get server quote/config version -> customer confirms -> create payment intent or cash tender -> atomic commit/internal settlement -> receipt/outbox -> close shift/count/reconcile. Gateway checkout selalu `pending` sampai signature-verified callback/update; timeout memakai idempotency key lama. Refund mencatat original sale/item, sisa refundable qty, kondisi barang, restock disposition, approval, payment state dan reversal journal.

**Validasi/efek:** server menghitung semua amount; kasir tidak dapat ubah harga, tax atau paid status. Split tender sukses harus menutup total; change hanya cash. Print failure tidak mengubah sale. Refund memecah amount/tax/discount dari snapshot asli; inventory return hanya setelah penerimaan fisik. Maker-checker pada threshold dan self-approval ditolak.

**Acceptance:** timeout setelah commit menampilkan transaksi yang sama; gateway webhook duplicate/out-of-order tidak menggandakan pembayaran; refund melebihi saldo refundable ditolak; tutup shift menyimpan variance tanpa mengubah transaksi.

### 027.6 `MOD-PET-SERVICE-001` - Service catalog, booking, grooming, boarding, transport

**Actor/permission:** front desk, customer portal, groomer, boarding staff, branch manager; `booking:*`, `service:session:manage`, `boarding:stay:manage`.

**Fungsi/layar:** availability/slot search; booking create/confirm/reschedule/cancel; deposit; waitlist; calendar/queue; check-in/intake; service session/checklists/photos; pet care log; room occupancy/cleaning; checkout bill preview; driver dispatch/POD.

**Workflow booking:** request -> quote -> reservation/resource lock -> confirm -> reminder -> check-in -> in-progress -> completed -> invoice/payment. Reschedule atomik melepas slot lama dan mengunci yang baru; jika gagal, slot lama tetap. Cancellation/no-show menghitung deposit/fee dari policy snapshot. Service completion menghasilkan billing draft, bukan revenue sampai sales payment.

**Workflow grooming:** intake notes/photos -> health/behavior warning -> service consent -> groomer assigned/start -> checklist/progress/add-on consent -> finish/photo -> supervisor handover -> invoice. Boarding: reserve room -> check-in condition/care plan -> time-stamped care logs/incident -> checkout bill -> payment/deposit reconcile -> cleaning -> release room. Transport: dispatch -> pickup -> in_transit -> delivered/POD atau failed_return.

**Validasi/efek:** ketersediaan dihitung pada timezone cabang, jam operasional, resource, durasi+buffer dan kapasitas. Konflik concurrent ditolak atomik. Care log obat harus merujuk instruction/authorization; insiden dibuat ticket dan alert owner sesuai consent/channel. Additional service memerlukan price approval sebelum pelaksanaan kecuali emergency note.

**Acceptance:** overlapping booking tidak oversubscribe resource; reschedule race mempertahankan booking lama; boarding checkout memakai billable rule snapshot; kamar tidak `available` sebelum cleaning pass; transport delivered harus punya actor/time/POD policy.

### 027.7 `MOD-PET-CLINIC-001` - Klinik, medical record, lab, surgery, pharmacy

**Actor/permission:** vet, triage/front desk terbatas, pharmacist, lab staff; `vet:*`, `pharmacy:*` dan branch clinical scope. Super admin tidak otomatis membaca rekam medis.

**Fungsi/layar:** triage/queue; encounter draft; history/allergies/vitals; diagnosis/procedure; consent; prescription; lab order/result/verification; surgery checklist; hospitalization/care chart; medical finalize/amend; pharmacy dispense/label; customer share authorization.

**Workflow:** arrival -> identity/pet link -> triage severity -> queue assignment -> encounter draft -> exam/diagnosis/treatment -> orders/consent -> finalization -> dispense/lab/surgery/hospitalization as applicable -> billing -> discharge/follow-up. Finalize membutuhkan veterinarian actor dan expected version. Amendment menjadi record terpisah bertanda tangan; tidak menghapus teks asli. Lab: ordered -> sample_collected -> processing -> result_draft -> verified -> released_to_record. Surgery: planned -> consented -> preop_checked -> in_progress -> recovery -> discharged/cancelled.

**Validasi/efek:** allergies/medication warnings ditampilkan sebelum resep dan dispense; acknowledgment berisi user/time. Rx qty tidak dapat melebihi prescribed minus dispensed/refill allowance. Lot FEFO/expiry dan permission controlled drug dicek atomik. File medical private; share-to-customer merupakan explicit action per document/result. Kasir hanya menerima item/service billing snapshot, bukan medical notes.

**Acceptance:** staff nonclinical tidak membaca medical payload; vet finalisasi stale version mendapat conflict; pharmacist tidak dapat mengubah diagnosis; dispense parsial menjaga remaining qty; patient portal hanya melihat bagian shared.

### 027.8 `MOD-PET-LOYALTY-001` - Membership, loyalty, promotion, campaign

**Actor/permission:** marketing/customer service untuk campaign/tier; cashier redeem sesuai batas; owner approve perubahan rule; `membership:*`, `loyalty:*`, `promotion:*`, `customer:consent:manage`.

**Fungsi/layar:** tier/benefit effective dates; customer enrollment/status; ledger/poin expiry; promotion rule builder berbasis schema; voucher issue/redeem; campaign audience preview/test-send/schedule/suppress; redemption/reversal history.

**Workflow:** configure draft rule -> validate/date/branch/stacking -> approve -> activate -> quote engine evaluates -> transaction commit reserves/redeems usage -> ledger earning/redeem -> scheduled expiry/reversal. Campaign: audience build -> exclude nonconsented/suppressed -> estimate recipients/cost -> test -> approve/schedule -> rate-limited dispatch -> delivery/opt-out tracking.

**Validasi/efek:** rule version immutable setelah dipakai; update membuat version baru. Kuota/voucher lock dalam transaksi. Earn/redeem/refund mengikuti formula 024.4. Campaign mengecek consent kembali saat send, tidak hanya saat audience dibuat; frequency cap per kontak/channel berlaku.

**Acceptance:** dua request bersamaan tidak melebihi kuota voucher; refund mencatat reversal poin tepat; consent yang dicabut setelah schedule menghentikan pesan; promo ditolak dengan reason code deterministik.

### 027.9 `MOD-PET-HR-001` - Karyawan, jadwal, absensi, komisi, payroll

**Actor/permission:** HR staff, employee self-service, manager approver, finance payroll approver; `hr:employee:*`, `hr:attendance:*`, `hr:payroll:*`.

**Fungsi/layar:** employee directory/private profile; branch assignment; contract/document expiry; schedules/shift swap/leave; raw attendance and correction; commission rule/entries; payroll period/run/exception review; approval/payment/slip; exports.

**Workflow:** employee invited/active/on_leave/terminated -> schedule published -> clock event by server -> correction request/review -> commission source finalized -> payroll period cutoff -> calculate immutable input version -> exception resolution -> maker review -> independent approve -> payment batch -> payslip -> close. Reopen requires elevated permission, reason, new version and downstream accounting adjustment.

**Validasi/efek:** employee cannot edit server time or approved attendance; one active assignment/period policy is enforced. Salary, bank, national IDs and payslip require sensitive permission and field masking. Commission uses line snapshot and refund adjustments. Statutory formula version must be explicit; if unconfigured, payroll cannot be marked compliant/paid as final.

**Acceptance:** duplicate payroll final for same employee/period rejected; preparer cannot self-approve; terminated user sessions revoked but history retained; closed payroll unaffected by later attendance edits.

### 027.10 `MOD-PET-FINANCE-001` - Kas, piutang, utang, accounting, tax, reports

**Actor/permission:** finance staff, branch manager for expense approval, owner for period close; `finance:*`, `report:finance:*`.

**Fungsi/layar:** COA and mapping; accounting periods; source journal queue; manual journal; post/reversal; cash/bank accounts; expense submit/approve/pay; AR/AP invoice, allocation, aging; bank statement import/match; tax configuration/reports; close period; financial statements.

**Workflow:** operational source commits -> event/source journal posted atomically or visible in exception queue -> finance reconciles clearing/AR/AP -> reviews exception -> approves/manual posts -> bank/payment match -> period checklist -> close. Period close checks unposted source events, out-of-balance journals, unresolved settlement, AP/AR reconciliation and report totals. Reopen requires owner/finance dual approval and audit.

**Validasi/efek:** journal double-entry and source uniqueness; no backdate to closed period; customer credit respects limit/due date; supplier payment allocation <= open payable; bank statement import dedupes external line ID. Tax reports use tax snapshot/effective date, not current product rate. Financial report endpoints are read-only and use consistent cutoff.

**Acceptance:** post fails without partial journal when mapping absent; debit/credit totals reconcile; reversal balances and references source; AR/AP aging agrees with open item ledger; a closed period rejects direct REST/RPC posting.

### 027.11 `MOD-PET-OPS-001` - Aset, SOP, dokumen, pengeluaran, budget, delivery, online order

**Actor/permission:** operations staff, asset custodian, approver, dispatcher, online customer; `operations:*`, `document:*`, `budget:*`, `delivery:*`.

**Fungsi/layar:** asset register/depreciation/maintenance; checklist template/version/execution; private document categories/version/expiry; expense and budget request; delivery dispatch/tracking/POD; online catalog/order/fulfillment/refund.

**Workflow:** asset acquire -> assign/location -> maintenance -> disposal/journal; SOP draft -> publish version -> scheduled run -> execute evidence -> reviewer close; document upload -> scan/metadata/permission -> version/expiry reminders -> retention/archive; expense/budget request -> approve -> pay/post -> actual; online order -> reserve stock TTL -> payment verified -> pick/pack -> dispatch -> POD -> close, or cancel/expire/release reservation.

**Validasi/efek:** checklist must use published template version; document path is never public by default; expired legal/asset document alerts owner. Online item stock uses same inventory balance/reservation as POS. Shipment cancellation releases reservation only if not dispatched; returns enter return/inspection flow before stock available. Budget actual derives from posted journals.

**Acceptance:** public user cannot access private document path; order expiration releases stock once; delivery status cannot skip required scan/POD; SOP edit does not alter completed checklist; asset disposal posts approved accounting entry when enabled.

### 027.12 `MOD-PET-ADMIN-001` - Super Admin, config, integrations, jobs, audit

**Actor/permission:** super admin with MFA; config approver/secret owner as separate duties; `settings:*`, `system:*`, `audit:*`.

**Fungsi/layar:** effective config by environment/org/branch; config schema/history/diff/rollback; provider adapter setup/test/rotate/disable; outbox/job health/retry; audit search/export; system health/capacity; feature availability and maintenance banner.

**Workflow:** read registry -> validate requested key/scope/risk -> check permission/expected version -> show effective-value impact -> approval if high-risk -> commit new config version/audit -> invalidate server cache -> workers pick version -> observe/test -> rollback by writing a new version. Integration secret path is write-only; successful test precedes enable. Retry dead-letter requires actor, reason and idempotency.

**Validasi/efek:** unknown key/provider/URL rejected; admin cannot submit SQL/JS/network destination; secret value never returned/logged; changes to tax, accounting, stock, clinical sharing or authentication have effective date and dual approval. Job retry cannot duplicate business mutation.

**Acceptance:** stale config version gets conflict; masked credential never appears in API/log; disabling provider preserves source transactions; audit before/after is redacted and immutable; non-super-admin cannot read health details or operate jobs.

### 027.13 Kontrak workflow lintas modul dan penyelesaian exception

1. Semua workflow dimulai dari source object ID dan `expected_version`; cross-module operation memegang satu source of truth dan tidak membuat transaksi paralel yang nilainya dapat berbeda.
2. Synchronous atomic effects yang diperlukan untuk kebenaran saldo (payment internal, stock, loyalty ledger, journal) harus commit bersama source di RPC; network effects melalui outbox dan tidak menahan sumber transaksi.
3. Jika asynchronous effect gagal, source tetap pada status bisnis yang benar dan exception terlihat di queue dengan `retry`, `resolve_manual`, atau `cancel` yang berpermission. Operator tidak menandai event sukses tanpa provider evidence/reconciliation.
4. Status parent tidak boleh `completed` bila child wajib belum final; child optional dapat `not_required` dengan reason. Contoh: purchase close menunggu receipt/bill policy; stay checkout menunggu care log dan invoice state; medical encounter final tidak menunggu external notification.
5. Semua workflow menyediakan resume dari checkpoint: load source/status/history -> identifikasi side effects yang sudah terjadi -> retry hanya effect yang belum berhasil menggunakan dedupe key -> rekonsiliasi ledger dan audit.
6. Halaman detail menampilkan timeline ordered by server timestamp, actor, status change, approval, payment, stock movement, journal, outbox delivery, correction/reversal dan request ID; field sensitif difilter permission.

### 027.14 Acceptance test kontrak modul

Setiap modul wajib memiliki test positif, negatif, concurrency dan recovery: aksi berpermission berhasil; aksi role/cabang lain ditolak; input invalid tidak meninggalkan side effect; retry idempotent; stale version mendapat conflict; audit dan outbox tepat; downstream failure dapat dilihat/dilanjutkan; reversal mempertahankan histori. UAT menandatangani fungsi utama per role, bukan hanya screenshot halaman.
