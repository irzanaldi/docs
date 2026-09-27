# POS System with HPP Engine — Design Spec

## Overview

Sistem Point of Sale hybrid (Restoran + Retail) dengan fitur utama **HPP Engine** yang menghitung cost dan profit per menu berdasarkan resep dan harga bahan baku. Dilengkapi kitchen display real-time, web dashboard sebagai CMS terpusat untuk semua role (Owner, Kitchen, Inventory, Finance, Marketing), dan customer mobile app untuk online ordering.

## Tech Stack

- **Backend:** NestJS (TypeScript), PostgreSQL, Prisma ORM, JWT Auth, Socket.IO (WebSocket)
- **Frontend Kasir & Kitchen:** Flutter (tablet, landscape), BLoC state management, Socket.IO client
- **Frontend Web CMS/Dashboard:** Next.js, Tailwind CSS + shadcn/ui, Recharts
- **Frontend Customer App (Phase 4):** Flutter (mobile), BLoC state management, Socket.IO client
- **API:** REST

## Project Structure

```
pos-system/
├── backend/          # NestJS
│   ├── src/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── menu/          # menu + resep + HPP
│   │   ├── ingredients/   # master bahan baku
│   │   ├── pos/           # transaksi penjualan
│   │   ├── orders/        # order flow + kitchen
│   │   ├── retail/        # item retail
│   │   ├── promos/        # promo management
│   │   ├── dashboard/     # analytics & reporting
│   │   └── cms/           # content management
│   └── prisma/
├── mobile/           # Flutter (kasir + kitchen display)
└── web/              # Next.js (dashboard owner)
```

## Phasing

- **Phase 1 (MVP):** POS + HPP Engine + Kitchen Display + Promo + Dashboard
- **Phase 2:** Inventory Management (PO, stok, gudang)
- **Phase 3:** Finance & Reporting (daily closing, laba rugi, laporan bulanan)
- **Phase 4:** Online Ordering & CMS (pesan online, voucher, delivery, CMS tampilan app)

---

## Phase 1: MVP

### Database Design

#### Users & RBAC

```
users
├── id (UUID)
├── name
├── email
├── password (hashed)
├── role_id (FK → roles)
├── is_active
├── created_at
└── updated_at

roles
├── id (UUID)
├── name (OWNER, KASIR, KITCHEN, INVENTORY, FINANCE, MARKETING)
└── description

permissions
├── id (UUID)
├── name (e.g. menu:create, transaction:read)
├── module
└── description

role_permissions
├── role_id (FK → roles)
└── permission_id (FK → permissions)
```

**Permission Matrix Phase 1:**

| Permission                | OWNER | KASIR | KITCHEN |
| ------------------------- | ----- | ----- | ------- |
| menu:create/update/delete | yes   | no    | yes     |
| menu:read                 | yes   | yes   | yes     |
| ingredient:manage         | yes   | no    | yes     |
| ingredient:read           | yes   | no    | yes     |
| transaction:create        | no    | yes   | no      |
| transaction:read          | yes   | yes   | no      |
| order:update_status       | no    | no    | yes     |
| order:read                | yes   | yes   | yes     |
| dashboard:view            | yes   | no    | no      |
| user:manage               | yes   | no    | no      |
| promo:manage              | yes   | no    | no      |
| promo:apply               | no    | yes   | no      |

#### Master Bahan Baku

```
ingredients
├── id (UUID)
├── name
├── unit (kg, liter, butir, pcs)
├── price_per_unit
├── stock_qty
├── min_stock
├── created_at
└── updated_at

ingredient_price_logs
├── id (UUID)
├── ingredient_id (FK → ingredients)
├── old_price
├── new_price
└── changed_at
```

#### Menu & Resep

```
menu_items
├── id (UUID)
├── name
├── category (FOOD, BEVERAGE)
├── selling_price
├── image_url
├── is_active
├── created_at
└── updated_at

recipes
├── id (UUID)
├── menu_item_id (FK → menu_items)
├── ingredient_id (FK → ingredients)
├── qty_needed
└── unit
```

HPP dihitung dinamis: `SUM(recipe.qty_needed * ingredient.price_per_unit)`

#### Retail Items

```
retail_items
├── id (UUID)
├── name
├── category
├── buy_price
├── selling_price
├── stock_qty
├── min_stock
├── image_url
├── created_at
└── updated_at
```

#### Transaksi (POS)

```
transactions
├── id (UUID)
├── transaction_code
├── cashier_id (FK → users)
├── payment_method (CASH, QRIS, EDC)
├── total_amount
├── total_hpp
├── total_profit
├── promo_discount
├── created_at
└── updated_at

transaction_items
├── id (UUID)
├── transaction_id (FK → transactions)
├── item_type (MENU, RETAIL)
├── item_id
├── qty
├── unit_price
├── unit_hpp (snapshot)
├── subtotal
└── sub_hpp
```

#### Order Flow (Kitchen)

```
orders
├── id (UUID)
├── transaction_id (FK → transactions)
├── order_number
├── status (PENDING, PREPARING, READY, SERVED)
├── created_at
└── updated_at

order_items
├── id (UUID)
├── order_id (FK → orders)
├── menu_item_id (FK → menu_items)
├── qty
├── notes
└── status (PENDING, PREPARING, DONE)
```

#### Promo

```
promos
├── id (UUID)
├── name
├── type (PERCENT, FIXED, BUY_X_GET_Y, BUNDLE, MIN_PURCHASE)
├── value
├── min_purchase_amount (nullable)
├── start_date
├── end_date
├── is_active
├── max_usage (nullable)
├── used_count
├── created_at
└── updated_at

promo_items
├── id (UUID)
├── promo_id (FK → promos)
├── item_type (MENU, RETAIL)
├── item_id
└── role (TARGET, FREE_ITEM, BUNDLE_MEMBER)

transaction_promos
├── id (UUID)
├── transaction_id (FK → transactions)
├── promo_id (FK → promos)
└── discount_amount (snapshot)
```

**Tipe promo:**

| Tipe         | Contoh                            | Dampak ke Profit                                |
| ------------ | --------------------------------- | ----------------------------------------------- |
| PERCENT      | Nasi Goreng diskon 20%            | Harga jual turun, HPP tetap, margin menyusut    |
| FIXED        | Potongan Rp 5.000 untuk Es Teh    | Harga jual turun                                |
| BUY_X_GET_Y  | Beli 2 kopi, gratis 1             | Item gratis tetap ada HPP, profit berkurang     |
| BUNDLE       | Paket Nasi + Minum Rp 30.000      | HPP gabungan, profit = harga bundle - total HPP |
| MIN_PURCHASE | Belanja min Rp 100.000 diskon 10% | Diskon di level transaksi                       |

**Aturan:**

- 1 transaksi hanya bisa menggunakan 1 promo (tidak bisa stack/gabung beberapa promo)
- Promo punya periode aktif (start_date - end_date)
- Opsional: max usage
- Owner bisa buat promo, Kasir hanya apply

---

### HPP Engine

Fitur utama yang membedakan portofolio ini.

**Cara kerja:**

```
Menu: Nasi Goreng (Harga Jual: Rp 25.000)
│
├── Resep:
│   ├── Nasi        200gr × Rp 12.000/kg  = Rp 2.400
│   ├── Telur       1 butir × Rp 2.500    = Rp 2.500
│   ├── Minyak      30ml × Rp 18.000/ltr  = Rp 540
│   ├── Bumbu       50gr × Rp 40.000/kg   = Rp 2.000
│   └── Packaging   1 pcs × Rp 1.000      = Rp 1.000
│
├── HPP Total: Rp 8.440
├── Profit: Rp 16.560
└── Margin: 66.2%
```

**Fitur:**

1. **Auto-calculate HPP** — HPP = SUM(qty_needed \* price_per_unit), dihitung dinamis dari data terkini
2. **Cascading price update** — harga bahan berubah → HPP semua menu terkait otomatis berubah
3. **Price change alert** — warning jika margin menu < 30%
4. **Profit snapshot per transaksi** — unit_hpp disimpan saat transaksi untuk akurasi laporan historis
5. **Retail item** — profit = selling_price - buy_price (tanpa resep)

**Endpoint utama:**

- `POST /menu` — buat menu + resep
- `GET /menu/:id/hpp` — breakdown HPP per menu
- `PATCH /ingredients/:id/price` — update harga bahan (auto log)
- `GET /dashboard/profit-analysis` — profit per menu, ranking, margin alert

---

### Order Flow & Kitchen Display

**Flow:**

```
Kasir (Flutter)                    Kitchen Display (Flutter)
     │                                      │
     ├── 1. Input order ──── WebSocket ────► │ Order muncul otomatis
     │                                      │
     │                                      ├── 2. Kitchen tap "Preparing"
     │   ◄──── WebSocket ──── status update │
     │                                      │
     │                                      ├── 3. Kitchen tap "Ready"
     │   ◄──── WebSocket ──── status update │
     │                                      │
     ├── 4. Kasir tap "Served" ────────────► │ Order hilang dari display
     │                                      │
     └── 5. Proses pembayaran & cetak struk
```

**Order status:** `PENDING → PREPARING → READY → SERVED`

**WebSocket events:**

| Event           | Dari    | Ke      | Payload              |
| --------------- | ------- | ------- | -------------------- |
| order:new       | Kasir   | Kitchen | order detail + items |
| order:status    | Kitchen | Kasir   | order_id, new_status |
| order:item_done | Kitchen | Kasir   | order_id, item_id    |

**Kitchen Display:**

- Order sebagai card, urut waktu masuk
- Tiap item bisa ditandai done satu per satu
- Warna card: kuning (PENDING) → biru (PREPARING) → hijau (READY)
- Timer per order

**Kasir flow:**

- Buat order → item menu ke kitchen, item retail langsung masuk struk
- Lihat status pesanan real-time
- Semua READY + bayar → cetak struk
- Satu struk bisa campur menu + retail

**Promo di kasir:**

- Sistem auto-detect promo yang berlaku
- Kasir bisa apply / skip promo
- Struk menampilkan harga normal, potongan promo, total setelah promo

---

### Web CMS / Dashboard (Next.js)

Satu platform web terpusat untuk semua role. Setiap role login ke web yang sama, menu/halaman yang muncul sesuai permission RBAC.

**Akses per role:**

| Role | Halaman yang tersedia |
|---|---|
| Owner | Semua halaman |
| Kitchen / Divisi Menu | Menu Management, Ingredient Management |
| Inventory Staff | (Phase 2) PO, Stok, Gudang, Supplier |
| Finance | (Phase 3) Daily Closing, Laporan, Operational Cost |
| Marketing | (Phase 4) CMS Tampilan App, Promo, Voucher |

#### Dashboard (Owner)

**Summary Cards (hari ini):**

- Total Penjualan (revenue)
- Total HPP
- Gross Profit
- Profit Margin (%)
- Jumlah Transaksi

**Charts:**

- Trend penjualan vs profit — line chart (harian/mingguan/bulanan)
- Profit per kategori — pie chart (Food vs Beverage vs Retail)
- Top 10 menu paling untung — bar chart (by profit)
- Top 10 menu paling laris — bar chart (by qty)
- Scatter plot: laris vs untung — sumbu X = qty terjual, Y = profit margin

**Promo Analytics:**

- Total diskon yang diberikan per periode
- Perbandingan penjualan dengan promo vs tanpa promo
- Promo mana yang paling efektif (naikkan volume paling banyak)
- Profit setelah promo

**Tabel detail:**

- List menu + HPP + margin (sortable, searchable)
- Alert margin di bawah threshold (highlight merah)
- Histori perubahan harga bahan + dampak ke HPP

**Filter:**

- Periode: hari ini, 7 hari, 30 hari, custom range
- Kategori: Food, Beverage, Retail, All
- Payment method: Cash, QRIS, EDC, All

#### Menu Management (Owner, Kitchen)

- CRUD menu + resep + HPP breakdown
- Ingredient Management — CRUD bahan baku + update harga + price history

#### Retail Management (Owner)

- CRUD retail items

#### Promo Management (Owner)

- CRUD promo + analytics

#### User Management (Owner)

- CRUD users + assign role (RBAC)

#### Transaction History (Owner, Kasir)

- List transaksi + detail per struk

#### Inventory Management (Owner, Inventory Staff) — Phase 2

- PO management (buat, approve, terima)
- Master supplier + gudang
- Stok masuk/keluar, stock opname
- Min stock alert

#### Finance & Reporting (Owner, Finance) — Phase 3

- Daily closing — rekap kasir harian
- Laporan laba rugi
- Laporan bulanan — export PDF/Excel
- Input operational cost
- Perhitungan pajak

#### CMS Tampilan App (Owner, Marketing) — Phase 4

- Banner management — upload, set link, atur urutan
- Section management — drag & drop, pilih menu per section
- Store info — jam buka, alamat, logo, cover image
- Preview tampilan sebelum publish

---

## Phase 2: Inventory Management

### Fitur Utama

- **Master Gudang** — lokasi penyimpanan (multi-gudang: dapur, bar, gudang utama)
- **Purchase Order (PO)** — request bahan ke supplier, approval flow
- **Stok Masuk** — terima barang dari supplier, update stok
- **Stok Keluar** — pengurangan stok otomatis saat transaksi (berdasarkan resep)
- **Stock Opname** — penyesuaian stok aktual vs sistem
- **Min Stock Alert** — notifikasi kalau bahan mendekati stok minimum

### Database Tambahan

```
warehouses
├── id, name, location

suppliers
├── id, name, contact, address

purchase_orders
├── id, supplier_id, warehouse_id
├── status (DRAFT, SUBMITTED, APPROVED, RECEIVED, CANCELLED)
├── created_by, approved_by, total_amount

purchase_order_items
├── id, po_id, ingredient_id, qty, unit_price

stock_movements
├── id, ingredient_id, warehouse_id
├── type (IN, OUT, ADJUSTMENT)
├── qty, reference_type (PO, TRANSACTION, OPNAME), reference_id
└── created_at
```

### Koneksi ke Phase 1

- Transaksi → stok bahan otomatis berkurang sesuai resep
- PO diterima → harga bahan bisa di-update → HPP ikut berubah
- Retail items stok berkurang saat terjual

---

## Phase 3: Finance & Reporting

### Fitur Utama

- **Daily Closing** — tutup kasir harian, rekap cash vs sistem
- **Laporan Laba Rugi** — revenue - HPP - operational cost = net profit
- **Laporan Bulanan** — summary per bulan, export PDF/Excel
- **Operational Cost** — input biaya operasional (gaji, sewa, listrik)
- **Tax** — perhitungan pajak (PB1 restoran 10%)

### Database Tambahan

```
daily_closings
├── id, cashier_id, date
├── system_total, actual_cash, difference, notes

operational_costs
├── id, name, category (SALARY, RENT, UTILITY, OTHER)
├── amount, period_month, period_year

monthly_reports
├── id, month, year
├── total_revenue, total_hpp, gross_profit
├── total_operational_cost, total_promo_discount
├── tax_amount, net_profit
```

### Koneksi ke Phase 1 & 2

- Data transaksi & HPP dari Phase 1
- Data PO & cost bahan dari Phase 2
- Revenue - HPP - operational cost - promo discount - tax = **net profit sebenarnya**

---

## Phase 4: Online Ordering & CMS

### Fitur Utama

#### 1. Online Ordering (Customer Mobile App — Flutter)

- **Browse menu** — pelanggan bisa lihat menu + harga + foto dari app
- **Pilih tipe order:**
  - **Dine-in** — pilih meja / scan QR meja → order masuk ke kitchen display
  - **Delivery** — input alamat → order masuk ke sistem + tracking status
  - **Pickup** — pesan dari app, ambil di tempat
- **Keranjang** — tambah item, atur qty, tambah notes per item
- **Apply voucher** — input kode voucher sebelum checkout
- **Order tracking** — real-time status (Dikonfirmasi → Diproses → Siap / Dalam Pengiriman → Selesai)
- **Order history** — riwayat pesanan pelanggan

#### 2. Voucher System (Online-only)

Berbeda dari promo di kasir (Phase 1). Voucher khusus untuk pelanggan online.

```
vouchers
├── id (UUID)
├── code (unique, e.g. "WELCOME50")
├── type (PERCENT, FIXED, FREE_DELIVERY)
├── value
├── min_purchase_amount (nullable)
├── max_discount_amount (nullable, cap untuk diskon persen)
├── start_date
├── end_date
├── is_active
├── max_usage (nullable, total penggunaan)
├── max_usage_per_user (nullable)
├── used_count
├── created_at
└── updated_at

voucher_usages
├── id (UUID)
├── voucher_id (FK → vouchers)
├── customer_id (FK → customers)
├── transaction_id (FK → transactions)
├── discount_amount
└── used_at
```

**Tipe voucher:**

| Tipe          | Contoh                    |
| ------------- | ------------------------- |
| PERCENT       | Diskon 20%, max Rp 15.000 |
| FIXED         | Potongan Rp 10.000        |
| FREE_DELIVERY | Gratis ongkir             |

**Distribusi voucher:**

- Kode manual (share via sosmed, cetak di struk)
- Auto-generate untuk customer baru (welcome voucher)
- Reward dari loyalitas / jumlah order

#### 3. Delivery Management

```
deliveries
├── id (UUID)
├── order_id (FK → orders)
├── customer_id (FK → customers)
├── address
├── delivery_fee
├── status (WAITING_PICKUP, ON_DELIVERY, DELIVERED, CANCELLED)
├── driver_name (nullable, manual input)
├── driver_phone (nullable)
├── estimated_time
├── delivered_at
└── notes

customers
├── id (UUID)
├── name
├── phone
├── email (nullable)
├── address_default
├── created_at
└── updated_at

customer_addresses
├── id (UUID)
├── customer_id (FK → customers)
├── label (Rumah, Kantor, dll)
├── address
├── latitude (nullable)
├── longitude (nullable)
└── is_default
```

**Delivery flow:**

```
Customer order (delivery)
     │
     ├── 1. Order masuk → Kitchen Display (sama seperti dine-in)
     │
     ├── 2. Kitchen selesai → status READY
     │
     ├── 3. Staff assign driver (manual input nama + no HP)
     │       └── Status: WAITING_PICKUP
     │
     ├── 4. Driver pickup → ON_DELIVERY
     │       └── Customer bisa lihat status di app
     │
     └── 5. Delivered → DELIVERED
             └── Customer konfirmasi terima
```

> **Note:** Untuk MVP Phase 4, driver management manual (input nama + HP). Integrasi ke platform ojol (GoSend, GrabExpress) bisa jadi enhancement di kemudian hari.

#### 4. CMS Tampilan Customer App (tambahan di Web Dashboard)

Fitur tambahan di web dashboard (yang sudah ada sejak Phase 1) untuk mengatur tampilan customer mobile app tanpa perlu update app. Diakses oleh role Owner dan Marketing.

**Yang bisa diatur via CMS:**

```
cms_banners
├── id (UUID)
├── title
├── image_url
├── link_type (MENU_ITEM, PROMO, VOUCHER, EXTERNAL_URL, NONE)
├── link_value (item_id atau URL)
├── position (HOME_TOP, HOME_MIDDLE, CATEGORY_PAGE)
├── sort_order
├── start_date
├── end_date
├── is_active
└── created_at

cms_sections
├── id (UUID)
├── title (e.g. "Menu Favorit", "Promo Spesial", "Menu Baru")
├── type (MENU_LIST, BANNER_CAROUSEL, CATEGORY_GRID)
├── content_config (JSON — filter: category, tag, manual item IDs)
├── sort_order
├── is_active
└── created_at

cms_store_info
├── id (UUID)
├── store_name
├── logo_url
├── cover_image_url
├── description
├── address
├── phone
├── operating_hours (JSON — { mon: "08:00-22:00", ... })
├── delivery_radius_km (nullable)
├── min_order_delivery
└── updated_at
```

**Tampilan customer app yang diatur CMS:**

```
┌─────────────────────────┐
│  [Banner Carousel]      │  ← cms_banners (HOME_TOP)
│  Promo / Highlight      │
├─────────────────────────┤
│  Menu Favorit ──────►   │  ← cms_sections (type: MENU_LIST)
│  [Item] [Item] [Item]   │
├─────────────────────────┤
│  [Banner]               │  ← cms_banners (HOME_MIDDLE)
├─────────────────────────┤
│  Kategori               │  ← cms_sections (type: CATEGORY_GRID)
│  [Food] [Beverage]      │
│  [Retail] [Paket]       │
├─────────────────────────┤
│  Menu Baru ──────►      │  ← cms_sections (type: MENU_LIST)
│  [Item] [Item]          │
└─────────────────────────┘
```

**CMS di web dashboard:**

- Drag & drop urutan section
- Upload banner + set link tujuan
- Pilih menu yang muncul di tiap section (manual pick atau auto by category)
- Edit info toko (jam buka, alamat, logo)
- Preview tampilan sebelum publish

#### 5. RBAC Tambahan Phase 4

| Permission             | OWNER | MARKETING | KASIR |
| ---------------------- | ----- | --------- | ----- |
| cms:manage             | yes   | yes       | no    |
| voucher:manage         | yes   | yes       | no    |
| delivery:manage        | yes   | no        | yes   |
| delivery:assign_driver | yes   | no        | yes   |
| online_order:read      | yes   | yes       | yes   |

### Koneksi ke Phase 1, 2 & 3

- Menu & harga dari Phase 1 → ditampilkan di customer app via CMS
- Order online masuk ke kitchen display yang sama (Phase 1)
- HPP & profit tracking tetap berlaku untuk order online
- Stok berkurang otomatis (Phase 2)
- Revenue online masuk ke laporan finance (Phase 3)
- Voucher discount masuk ke perhitungan profit
