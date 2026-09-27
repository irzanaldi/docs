# POS → ERP — Program Roadmap (Design)

- **Tanggal:** 2026-06-24
- **Status:** Disetujui (brainstorm) → siap dipecah jadi plan per fase
- **Lokasi project:** `/Users/irzan-aldi/Downloads/PROJECT/pos/` (backend, web, mobile, customer-app)
- **Tipe dokumen:** Program roadmap (bukan satu spec implementasi). Tiap fase/modul punya siklus spec→plan→TDD sendiri.

---

## 1. Konteks

`pos/` sudah matang, **bukan greenfield**. Sudah terbangun (verified via eksplorasi + grep, Jun 2026):

- **backend** (NestJS + Prisma, PostgreSQL `pos_system`): 27 modul. Multi-tenant + RBAC, HPP/recipe costing, promo/voucher engine, POS transaksi, kitchen order (WebSocket), online order, customer OTP, delivery, supplier/warehouse/PO→stock-movement→opname (inventory), daily-closing, opex, P&L bulanan (PDF/Excel), CMS, FCM.
- **web** (Next.js 16, App Router): ~24 halaman admin, RBAC sidebar, analytics dashboard, CRUD semua modul.
- **mobile** (Flutter BLoC): app kasir — menu grid, cart, checkout, promo, order tracking, KDS.
- **customer-app** (Flutter BLoC): OTP auth, browse, cart, checkout, voucher, order tracking, profile, address, notif.

Riwayat build terekam di netza (`01_Projects/POS *.md`): Phase 1–4 selesai, plus spec `2026-05-03-phase5a-multi-tenant-swagger` (multi-tenant) dan `2026-05-22-presensi-phase1-core-hr` (HR/absensi — project terpisah, relevan sbg referensi modul HR nanti).

### Profil tujuan (hasil brainstorm)

| Dimensi | Keputusan |
|---|---|
| Endgame | **SaaS multi-tenant + ERP**. POS = front operasional; ERP = backbone data. |
| Stage | **Pre-customer**. Prioritas: production-grade dulu, jualan belakangan. |
| Resourcing | User mengarahkan; agent mengeksekusi koding via plan + subagent, **TDD, sekuensial per item**. |
| Strategi urutan | **C — Foundation first** (base bersih dulu, baru fitur). Diperkuat oleh ambisi ERP. |

---

## 2. Temuan current-state (verified)

Gap kritis dikonfirmasi via grep, bukan asumsi:

| Temuan | Bukti |
|---|---|
| **Tidak ada payment gateway** | `package.json` backend: nihil midtrans/xendit/stripe. `PaymentMethod` (CASH/QRIS/EDC) cuma label enum; tak ada pemrosesan. customer-app: tak ada flow bayar sama sekali. |
| **Pajak tidak dihitung** | `taxRate` hanya muncul di 1 tempat (DTO tenant settings), tak pernah dipakai. `pos.service.ts`: `totalAmount = Σsubtotal − promoDiscount`. Tak ada tax/service-charge/rounding. |
| **Tidak ada refund/void** | 0 hit di backend. Tak ada model/endpoint pembalik transaksi. |
| **FCM setengah jadi** | `firebase-admin@^13.8.0` ter-install, tapi pengiriman di-comment: `// const messaging = admin.messaging();`. |
| **Isolasi tenant: ADA (perlu audit)** | `tenant-prisma.provider.ts` pakai `prisma.$extends({...})` + `tenant-models.const.ts` + `tenant.middleware.ts`. Service query pakai `where:{id}` polos → nyandar auto-inject tenantId. Perlu dipastikan **lengkap & dipakai konsisten** (bukan dibangun dari nol). |
| **Audit log generic: tidak ada** | Tak ada model `AuditLog`. Cuma 8 occurrence `createdBy/updatedBy`. |
| **Error filter: ada, belum standar** | `common/filters/http-exception.filter.ts` ada; envelope belum konsisten. |
| **Test baseline** | 20 spec unit + 1 e2e; script `test:cov` ada. Unit lumayan, e2e tipis. |

---

## 3. Keputusan arsitektur kunci

1. **"Data digabung satu project" = satu codebase + satu DB + GL sebagai tulang-punggung finansial.** BUKAN refactor master-data big-bang. Model existing (MenuItem/RetailItem/Ingredient, Customer/Supplier) **tetap**; modul ERP ditambah *di-samping*, diikat lewat referensi + **layer posting ke General Ledger**. (User eksplisit menolak unified master-data.)
2. **Monolith + PostgreSQL tunggal dipertahankan.** Cocok untuk ERP; tak perlu microservices.
3. **GL = integrasi finansial.** Semua modul posting ke buku besar: POS txn → jurnal penjualan; PO/terima-barang → inventory + AP; payroll → beban gaji. Inilah mekanisme "penyatuan data" di lapis finansial tanpa menyatukan tiap master table.
4. **Payment gateway = model BYO** (tiap merchant colok akun Midtrans/Xendit sendiri), bukan aggregator/escrow — menghindari beban lisensi PJP. Fork final diputus di spec Fase 1.1.
5. **SaaS commercial layer (billing/onboarding/superadmin) DI-DEFER.** Di-*design-for*, tidak dibangun sekarang (belum ada customer).

---

## 4. Roadmap (5 fase, ter-dekomposisi, urut dependency)

Tiap item bernomor = satu siklus **spec → writing-plans → TDD implement** (via subagent), dengan checkpoint review.

### Fase 0 — Foundation hardening
*Diperkuat oleh ERP: nilai ERP = data core bersih & konsisten.*

- **0.1 Audit & kunci isolasi tenant.** Review `$extends` di `tenant-prisma.provider.ts`: cek nutup semua model di `tenant-models.const.ts` + semua operasi (findUnique/update/delete/deleteMany/upsert/aggregate/count). Grep tiap service — pastikan konsumsi scoped-client, bukan raw `PrismaService`. Tulis e2e *serangan lintas-tenant* (tenant A coba akses data tenant B → harus ditolak). **Pusat Fase 0.**
- **0.2 Standarin error envelope.** Keraskan `http-exception.filter.ts` → bentuk konsisten `{code, message, details}`, map error Prisma (P2002 dst) + validasi. Apps konsumsi seragam.
- **0.3 Audit log generic.** Model `AuditLog` (who/what/when/tenantId/before-after) + interceptor penangkap mutasi + isi `createdBy/updatedBy` di model kunci. Sekalian fondasi jejak-audit akunting.
- **0.4 Jaring test.** e2e jalur-uang (pos transaction) + auth + isolasi. Naikkan coverage service kritis. (Test isolasi 0.1 = bagian jaring ini.)
- **0.5 Design spike — keputusan arsitektur ERP (bukan koding).** Kunci: monolith + single-DB, modul-di-samping, **kontrak posting GL** + kerangka chart-of-accounts. Keystone semua modul Fase 3.

Urutan dalam fase: 0.1+0.4 (test-first serangan isolasi) → 0.2 → 0.3 → 0.5.

### Fase 1 — Financial core (kebenaran POS)

- **1.1 Payment gateway.** Backend modul payment + webhook + rekonsiliasi; customer-app checkout bayar beneran; mobile QRIS/EDC real. Fork BYO-vs-aggregator diputus di spec ini (condong BYO).
- **1.2 Tax + service charge.** Terapkan `tenant.taxRate` + serviceCharge di total `pos.service.ts`; betulkan `totalProfit`; update struk 2 app. **PPN 11%** — dobel-penting untuk akunting (Fase 3.1).
- **1.3 Refund/void.** Model `Refund`, balik stock-movement, balik bayar via gateway (numpang 1.1), dampak ke daily-closing; UI web + mobile.

### Fase 2 — Fitur produksi app-level

- **2.1 Offline kasir.** Antrian lokal (Hive/sqflite) + sync engine + submit transaksi idempoten. (ROI tertinggi sisi mobile — outage = tak bisa jualan.)
- **2.2 Hardware/ops kasir.** Thermal printer (esc/pos), barcode scan (`mobile_scanner` sudah di deps customer-app), hitung kembalian, hold/park order.
- **2.3 Customer-app finish.** Wire FCM push (firebase-admin sudah ada), status order real-time (socket), QR table ordering.
- **2.4 Admin polish.** Halaman customer CRUD, auto-alert low-stock, wiring pagination, KDS view di web.

### Fase 3 — Modul ERP
*Tiap modul = spec sendiri saat nyampe (requirement bakal geser; jangan over-spec sekarang). Urut by dependency GL.*

- **3.1 Accounting/GL** *(spine — duluan)*. Chart-of-accounts, jurnal double-entry, auto-posting dari POS + PO/AP + payroll, AP/AR aging, neraca + P&L real (accrual), lapor PPN. Membangun kontrak posting dari 0.5.
- **3.2 Advanced inventory.** Transfer antar-gudang, batch/lot + expiry tracking, valuasi FIFO/average *(posting ke GL via 3.1)*, reorder-point otomatis. Lanjutan inventory existing.
- **3.3 HR/Payroll.** Karyawan, absensi, shift, payroll run *(beban gaji posting ke GL via 3.1)*. Naik dari `OperationalCost.SALARY`. Referensi: spec `presensi-phase1-core-hr`.

### Fase 4 — SaaS commercial *(DEFERRED — design-for)*

Onboarding self-serve + setup wizard · subscription billing (merchant→platform) · plan tier + feature gating · superadmin panel + usage metering · loyalty/rewards.

---

## 5. Non-goals (YAGNI — sengaja dibuang)

Master-data big-bang refactor · manufacturing/production-order · fixed-asset accounting · budgeting/forecasting · multi-company consolidation · payment aggregator+escrow+lisensi · GPS delivery routing · demand forecasting · B2B wholesale pricing · konversi multi-currency.

---

## 6. Model eksekusi

- Dokumen ini = **roadmap program**. Sub-project pertama yang dikerjakan = **Fase 0** (punya 5 item, layak spec sendiri).
- Langkah berikut: brainstorm/spec **Fase 0** → writing-plans → eksekusi TDD per item, review checkpoint.
- Fase jauh (akunting, HR, SaaS) di-brainstorm + spec **saat nyampe**, bukan sekarang.
- Catatan git: dokumen ini ditaruh di `PROJECT/docs/superpowers/specs/` (konvensi spec POS existing). Root **bukan** git repo; `pos/` git repo sendiri. Belum di-commit.

---

## 7. Keputusan tertunda (diputus di spec per-fase)

- **1.1:** Provider payment (Midtrans Snap vs Xendit) + konfirmasi final model BYO.
- **0.5 / 3.1:** Detail kontrak posting GL + struktur chart-of-accounts (kas/bank, persediaan, AP, AR, pendapatan, HPP, PPN keluaran, beban).
- **3.2:** Metode valuasi (FIFO vs weighted-average) — pengaruh ke HPP & neraca.
- **2.1:** Strategi konflik sync offline (last-write-wins vs idempotent server reconcile).
