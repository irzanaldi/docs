# Phase 4: Online Ordering & CMS — Design Spec

## Overview

Phase 4 extends the POS system with a customer-facing mobile app (Flutter) for online ordering, a voucher system, delivery management, and CMS for managing the app's content from the existing web dashboard. This phase reuses the existing monolithic NestJS backend — new modules are added alongside Phase 1-3 modules.

## Architecture Decision

**Approach: Monolithic Extension** — new modules added to existing NestJS backend. Customer app hits the same REST API with a separate auth context (CustomerAuthGuard vs StaffAuthGuard). Online orders reuse existing `orders` table and kitchen WebSocket pipeline.

Rationale:
- Shared DB, shared Prisma schema — no sync issues
- Online orders reuse existing order → kitchen flow, just different entry point
- Simpler deployment for a portfolio project
- Consistent with Phase 1-3 architecture

## Key Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Payment | Pay on-site (COD/kasir). Payment gateway is a future enhancement |
| 2 | Customer auth | Phone OTP login, email optional in profile |
| 3 | Order tracking | FCM push notifications (no WebSocket to customer app) |
| 4 | Order confirmation | Kasir confirms all online orders before they enter kitchen |
| 5 | Delivery fee | Configurable flat rate from CMS (cms_store_info) |
| 6 | Order type scope | All 3: dine-in + delivery + pickup |
| 7 | CMS preview | Static phone-frame preview in dashboard, no draft/publish — save = live |
| 8 | OTP provider | Abstracted interface, static OTP ("123456") for dev |

## New Backend Modules

```
backend/src/
├── ...existing (auth, menu, pos, orders, etc.)
├── customers/         # customer auth (OTP) + profile + addresses
├── online-orders/     # customer-facing order flow
├── vouchers/          # voucher CRUD + validation
├── deliveries/        # delivery lifecycle management
├── cms/               # banners, sections, store info
├── notifications/     # FCM push notifications
└── uploads/           # image upload (banners, store logo/cover)
```

---

## 1. Customer Auth & Profile

### Auth Flow

```
Customer opens app
    |
    +-- Input phone number
    |
    +-- Backend generates OTP -> send via OtpService (abstracted)
    |       +-- Dev mode: always "123456"
    |
    +-- Customer inputs OTP
    |
    +-- Backend verifies -> issues Customer JWT
    |       +-- JWT payload: { sub: customer_id, role: "CUSTOMER", phone }
    |
    +-- First time? -> auto-create customer record
        +-- Returning? -> login, load profile
```

### Auth Separation

Two separate guards in NestJS:

- `@UseGuards(StaffAuthGuard)` — existing, for kasir/kitchen/owner/etc
- `@UseGuards(CustomerAuthGuard)` — new, decodes customer JWT

Endpoint namespacing:
- `/api/customers/auth/*` — OTP request, verify, refresh
- `/api/customers/profile` — get/update profile
- `/api/customer-app/*` — all customer-facing endpoints

### Database Schema

```
customers
+-- id (UUID, PK)
+-- name (varchar)
+-- phone (varchar, unique)
+-- email (varchar, nullable)
+-- address_default (text, nullable)
+-- created_at (timestamp)
+-- updated_at (timestamp)

customer_addresses
+-- id (UUID, PK)
+-- customer_id (FK -> customers)
+-- label (varchar — "Rumah", "Kantor", etc.)
+-- address (text)
+-- latitude (decimal, nullable)
+-- longitude (decimal, nullable)
+-- is_default (boolean)

customer_otps
+-- id (UUID, PK)
+-- phone (varchar)
+-- code (varchar, hashed)
+-- expires_at (timestamp)
+-- verified (boolean, default false)
+-- created_at (timestamp)
```

### OTP Service Interface

```typescript
interface OtpService {
  send(phone: string, code: string): Promise<void>;
  // Dev: logs to console
  // Prod: implement SMS/WA provider
}
```

Rate limit: max 3 OTP requests per phone per 5 minutes. OTP valid 5 minutes.

### Endpoints

```
POST   /api/customers/auth/otp/request    # send OTP
POST   /api/customers/auth/otp/verify     # verify OTP, return JWT
POST   /api/customers/auth/refresh        # refresh token
GET    /api/customers/profile             # get profile
PATCH  /api/customers/profile             # update profile
GET    /api/customers/addresses           # list addresses
POST   /api/customers/addresses           # add address
PATCH  /api/customers/addresses/:id       # update address
DELETE /api/customers/addresses/:id       # delete address
```

---

## 2. Online Ordering Flow

### Order Types

| Type | Entry Point | Extra Data |
|------|------------|------------|
| DINE_IN | Scan QR code at table | table_number |
| PICKUP | Select pickup time | pickup_time (ASAP or scheduled) |
| DELIVERY | Select/add address | delivery address + fee |

### Order Status Flow

```
Customer submits order
    |
    +-- PENDING_CONFIRMATION     <- waiting for kasir to confirm
    |
    +-- CONFIRMED                <- kasir accepts, enters kitchen
    |       +-- FCM: "Pesanan dikonfirmasi!"
    |
    +-- PREPARING                <- kitchen starts cooking
    |       +-- FCM: "Pesanan sedang diproses"
    |
    +-- READY                    <- kitchen done
    |       +-- Dine-in:  FCM "Pesanan siap diantar ke meja"
    |       +-- Pickup:   FCM "Pesanan siap diambil"
    |       +-- Delivery: FCM "Pesanan siap, menunggu driver"
    |
    +-- ON_DELIVERY              <- delivery only, driver picked up
    |       +-- FCM: "Pesanan dalam pengiriman"
    |
    +-- COMPLETED                <- done
    |       +-- Dine-in:  kasir marks served
    |       +-- Pickup:   customer picks up, kasir confirms
    |       +-- Delivery: customer confirms received
    |
    +-- CANCELLED                <- kasir rejects / customer cancels
            +-- FCM: "Pesanan dibatalkan"
```

### Orders Table Extension

Existing `orders` table from Phase 1 is extended with new columns:

```
orders (new columns)
+-- source (ENUM: POS, ONLINE)                    # order origin
+-- order_type (ENUM: DINE_IN, PICKUP, DELIVERY)   # online order type
+-- customer_id (FK -> customers, nullable)         # null for POS orders
+-- table_number (varchar, nullable)                # dine-in QR scan
+-- pickup_time (timestamp, nullable)               # scheduled pickup
+-- customer_notes (text, nullable)                  # customer instructions
+-- confirmed_at (timestamp, nullable)              # when kasir confirmed
+-- voucher_id (FK -> vouchers, nullable)           # applied voucher
+-- voucher_discount (decimal, default 0)           # discount amount
+-- delivery_fee (decimal, default 0)               # delivery charge
```

### Integration with Existing Kitchen Flow

1. Customer submits order -> `orders` record created (status: PENDING_CONFIRMATION, source: ONLINE)
2. Kasir confirms -> status = CONFIRMED, emit `order:new` WebSocket event to kitchen (identical to Phase 1 flow)
3. Kitchen processes -> same WebSocket events as Phase 1 (order:status, order:item_done)
4. Complete -> transaction record created (pay on-site)

### Transaction Timing

Since payment is on-site:
- **Dine-in/Pickup:** `transaction` created when customer pays at kasir (after READY)
- **Delivery:** `transaction` created when kasir confirms order (pre-recorded, customer pays on delivery)

### Cancel Rules

- Customer can cancel while status is `PENDING_CONFIRMATION`
- Kasir can reject (= cancel) at `PENDING_CONFIRMATION`
- After `CONFIRMED`, only kasir can cancel

### QR Dine-in

QR code per table encodes: `{ table_number: "A5" }`. App scans -> auto-sets `table_number` + `order_type: DINE_IN`.

### Customer App Endpoints

```
GET    /api/customer-app/menu                # browse menu (active items only)
GET    /api/customer-app/menu/:id            # menu detail + image + price
POST   /api/customer-app/orders              # submit order
GET    /api/customer-app/orders              # order history (paginated)
GET    /api/customer-app/orders/:id          # order detail + current status
PATCH  /api/customer-app/orders/:id/cancel   # cancel order (pre-confirmation only)
```

---

## 3. Voucher System

### Scope

Vouchers are online-only, separate from Phase 1 kasir promos. Customers apply voucher codes in the app before submitting orders.

### Database Schema

```
vouchers
+-- id (UUID, PK)
+-- code (varchar, unique — e.g. "WELCOME50")
+-- type (ENUM: PERCENT, FIXED, FREE_DELIVERY)
+-- value (decimal)
+-- min_purchase_amount (decimal, nullable)
+-- max_discount_amount (decimal, nullable)    # cap for percent discounts
+-- start_date (timestamp)
+-- end_date (timestamp)
+-- is_active (boolean)
+-- max_usage (integer, nullable)              # total usage limit
+-- max_usage_per_user (integer, nullable)
+-- used_count (integer, default 0)
+-- created_at (timestamp)
+-- updated_at (timestamp)

voucher_usages
+-- id (UUID, PK)
+-- voucher_id (FK -> vouchers)
+-- customer_id (FK -> customers)
+-- transaction_id (FK -> transactions)
+-- discount_amount (decimal)
+-- used_at (timestamp)
```

### Validation Pipeline

Sequential validation when customer applies a voucher:

1. Code exists & is_active? -> "Voucher tidak ditemukan"
2. Within date range? -> "Voucher sudah expired"
3. max_usage reached? -> "Voucher sudah habis"
4. max_usage_per_user reached? -> "Kamu sudah pakai voucher ini"
5. min_purchase_amount met? -> "Minimum belanja Rp X"
6. FREE_DELIVERY + order bukan delivery? -> "Voucher hanya untuk delivery"

### Discount Calculation

| Type | Formula |
|------|---------|
| PERCENT | `min(subtotal * value / 100, max_discount_amount)` |
| FIXED | `min(value, subtotal)` — discount cannot exceed subtotal |
| FREE_DELIVERY | `delivery_fee` becomes 0 |

### Order Total Calculation

```
total = subtotal - voucher_discount + delivery_fee
```

Where `subtotal` = sum of all item prices * quantities.

### Voucher in Order Flow

1. Customer adds items to cart
2. Customer applies voucher code -> `POST /api/customer-app/vouchers/validate`
3. Backend returns `{ valid, discount_amount, message }`
4. Customer submits order (with voucher_code) -> backend re-validates to prevent race conditions
5. Order completes -> `voucher_usages` record created, `vouchers.used_count` incremented

### Auto Welcome Voucher

On first customer registration, system auto-generates a personal voucher:
- Code: `WELCOME-{random6chars}`
- Type: PERCENT, value: 10, max_discount_amount: 15000
- max_usage: 1, max_usage_per_user: 1
- Valid 30 days from registration date

### Profit Impact

- `transaction.total_amount` = subtotal - voucher_discount
- `transaction.total_profit` = total_amount - total_hpp
- Voucher discounts appear in Phase 3 reports as revenue reduction

### Endpoints

```
# Customer App
POST   /api/customer-app/vouchers/validate    # validate code, preview discount

# Dashboard (Owner, Marketing)
GET    /api/vouchers                          # list all vouchers
POST   /api/vouchers                          # create voucher
GET    /api/vouchers/:id                      # detail + usage stats
PATCH  /api/vouchers/:id                      # update voucher
DELETE /api/vouchers/:id                      # soft delete (is_active=false)
GET    /api/vouchers/:id/usages               # usage history
```

---

## 4. Delivery Management

### Delivery Flow

```
Customer orders (type: DELIVERY)
    |
    +-- 1. Kasir confirms -> kitchen starts cooking
    |
    +-- 2. Kitchen done -> status READY
    |
    +-- 3. Kasir assigns driver (manual input: name + phone)
    |       +-- deliveries record created, status: WAITING_PICKUP
    |       +-- FCM: "Pesanan siap, menunggu driver"
    |
    +-- 4. Kasir updates -> ON_DELIVERY
    |       +-- FCM: "Pesanan dalam pengiriman"
    |
    +-- 5a. Customer confirms received in app -> DELIVERED
    |       OR
    +-- 5b. Kasir marks delivered (fallback)
    |
    +-- 6. CANCELLED — kasir can cancel any time before DELIVERED
```

### Database Schema

```
deliveries
+-- id (UUID, PK)
+-- order_id (FK -> orders)
+-- customer_id (FK -> customers)
+-- address (text)                           # snapshot from customer address
+-- latitude (decimal, nullable)
+-- longitude (decimal, nullable)
+-- delivery_fee (decimal)                   # from cms_store_info flat rate
+-- status (ENUM: WAITING_PICKUP, ON_DELIVERY, DELIVERED, CANCELLED)
+-- driver_name (varchar, nullable)
+-- driver_phone (varchar, nullable)
+-- estimated_time (integer, nullable)       # minutes, manual input by kasir
+-- delivered_at (timestamp, nullable)
+-- notes (text, nullable)
+-- created_at (timestamp)
+-- updated_at (timestamp)
```

### Address Snapshot

When customer submits a delivery order, the address is snapshotted into `deliveries.address`. Subsequent edits to customer addresses do not affect existing delivery records.

### Delivery Fee

- Flat rate configured in `cms_store_info.delivery_fee_amount`
- Customer app fetches and displays fee in cart
- FREE_DELIVERY voucher sets fee to 0

### Delivery Radius Check

`cms_store_info.delivery_radius_km`:
- If set and customer address has lat/lng: calculate distance, reject if beyond radius
- If null: no restriction (all addresses allowed)
- If address has no lat/lng: skip check, allow order

### Endpoints

```
# Kasir/Staff
POST   /api/deliveries/:orderId/assign     # assign driver (name, phone, est. time)
PATCH  /api/deliveries/:orderId/status      # update status

# Customer App
GET    /api/customer-app/orders/:id/delivery    # delivery status + driver info
PATCH  /api/customer-app/orders/:id/received    # customer confirms received
```

---

## 5. CMS (Content Management for Customer App)

### Scope

Owner and Marketing manage customer app appearance from the web dashboard. No draft/publish workflow — saving changes makes them live immediately.

### Database Schema

```
cms_banners
+-- id (UUID, PK)
+-- title (varchar)
+-- image_url (varchar)
+-- link_type (ENUM: MENU_ITEM, PROMO, VOUCHER, EXTERNAL_URL, NONE)
+-- link_value (varchar, nullable — item_id or URL)
+-- position (ENUM: HOME_TOP, HOME_MIDDLE, CATEGORY_PAGE)
+-- sort_order (integer)
+-- start_date (timestamp)
+-- end_date (timestamp)
+-- is_active (boolean)
+-- created_at (timestamp)

cms_sections
+-- id (UUID, PK)
+-- title (varchar — e.g. "Menu Favorit", "Promo Spesial")
+-- type (ENUM: MENU_LIST, BANNER_CAROUSEL, CATEGORY_GRID)
+-- content_config (JSON)
+-- sort_order (integer)
+-- is_active (boolean)
+-- created_at (timestamp)

cms_store_info
+-- id (UUID, PK)                          # singleton — 1 row per store
+-- store_name (varchar)
+-- logo_url (varchar, nullable)
+-- cover_image_url (varchar, nullable)
+-- description (text, nullable)
+-- address (text)
+-- phone (varchar)
+-- operating_hours (JSON — { "mon": "08:00-22:00", ... })
+-- delivery_radius_km (decimal, nullable)
+-- min_order_delivery (decimal, nullable)
+-- delivery_fee_amount (decimal)           # flat rate delivery fee
+-- updated_at (timestamp)
```

### content_config Formats

```jsonc
// MENU_LIST — manual pick
{ "mode": "manual", "item_ids": ["uuid1", "uuid2", "uuid3"] }

// MENU_LIST — auto by category
{ "mode": "auto", "category": "FOOD", "limit": 6 }

// MENU_LIST — auto by tag
{ "mode": "auto", "tag": "new", "limit": 4 }

// CATEGORY_GRID — show all active categories
{ "mode": "auto" }

// BANNER_CAROUSEL — references banners by position
{ "banner_position": "HOME_TOP" }
```

### Customer App Home Endpoint

Single endpoint returns fully resolved home page data:

```
GET /api/customer-app/home

Response:
{
  "store_info": {
    "store_name": "...",
    "logo_url": "...",
    "cover_image_url": "...",
    "operating_hours": { "mon": "08:00-22:00", ... },
    "is_open": true
  },
  "banners": {
    "HOME_TOP": [{ "id", "title", "image_url", "link_type", "link_value" }],
    "HOME_MIDDLE": [...]
  },
  "sections": [
    {
      "id": "...",
      "title": "Menu Favorit",
      "type": "MENU_LIST",
      "items": [{ "id", "name", "selling_price", "image_url" }]
    },
    {
      "id": "...",
      "title": "Kategori",
      "type": "CATEGORY_GRID",
      "categories": [{ "name", "item_count" }]
    }
  ]
}
```

Backend resolves `content_config` into populated data. App just renders.

### Static Preview in Dashboard

- Owner/Marketing edits sections -> clicks "Preview"
- Backend returns same response shape as `/api/customer-app/home`
- Dashboard renders in a fixed 375x812 phone-frame container
- Read-only visual, no interaction

### Operating Hours & Store Status

App checks `operating_hours` against current time:
- Outside operating hours: menu remains browsable, order button disabled, displays "Toko sedang tutup"

### Image Upload

Banners and store images are uploaded via a shared upload endpoint:

```
POST /api/uploads/image
Content-Type: multipart/form-data
Response: { "url": "/uploads/images/abc123.jpg" }
```

Constraints: max 2MB, formats: JPG, PNG, WebP. Storage: local filesystem (MVP), S3-compatible for production.

### Endpoints

```
# Banners
GET    /api/cms/banners
POST   /api/cms/banners
PATCH  /api/cms/banners/:id
DELETE /api/cms/banners/:id
PATCH  /api/cms/banners/reorder              # batch update sort_order

# Sections
GET    /api/cms/sections
POST   /api/cms/sections
PATCH  /api/cms/sections/:id
DELETE /api/cms/sections/:id
PATCH  /api/cms/sections/reorder              # batch update sort_order

# Store Info
GET    /api/cms/store-info
PATCH  /api/cms/store-info                    # upsert singleton

# Preview
GET    /api/cms/preview                       # same shape as customer-app/home

# Upload
POST   /api/uploads/image
```

---

## 6. FCM Notifications

### Customer Device Registration

```
customer_devices
+-- id (UUID, PK)
+-- customer_id (FK -> customers)
+-- fcm_token (varchar)
+-- device_type (ENUM: ANDROID, IOS)
+-- is_active (boolean)
+-- created_at (timestamp)
+-- updated_at (timestamp)
```

Customer login -> app registers FCM token with backend. Multiple devices supported per customer.

### Notification Events

| Trigger | Message |
|---------|---------|
| Kasir confirms order | "Pesanan #{order_number} dikonfirmasi!" |
| Kasir rejects order | "Pesanan #{order_number} dibatalkan" |
| Kitchen starts preparing | "Pesanan #{order_number} sedang diproses" |
| Kitchen done (READY) | "Pesanan #{order_number} siap!" |
| Driver assigned | "Driver {name} sedang menuju ke toko" |
| Status ON_DELIVERY | "Pesanan #{order_number} dalam pengiriman" |
| Status DELIVERED | "Pesanan selesai! Terima kasih" |
| Welcome voucher created | "Selamat datang! Pakai voucher {code}" |

### Notification Service Interface

```typescript
interface NotificationService {
  sendToCustomer(customerId: string, payload: {
    title: string;
    body: string;
    data?: Record<string, string>;  // deep link info
  }): Promise<void>;
}
```

Implementation: Firebase Admin SDK. Lookups all active `fcm_token` entries for customer and sends to each.

### Notification History

```
notifications
+-- id (UUID, PK)
+-- customer_id (FK -> customers)
+-- title (varchar)
+-- body (text)
+-- is_read (boolean, default false)
+-- created_at (timestamp)
```

### Notification Endpoints

```
# Customer App
POST   /api/customer-app/devices              # register FCM token
DELETE /api/customer-app/devices/:token        # unregister token
GET    /api/customer-app/notifications         # list notifications (paginated)
PATCH  /api/customer-app/notifications/:id/read     # mark as read
PATCH  /api/customer-app/notifications/read-all     # mark all as read
```

---

## 7. RBAC Extension

### New Role

**MARKETING** role added to existing roles table.

### New Permissions

| Permission | OWNER | MARKETING | KASIR | KITCHEN |
|---|---|---|---|---|
| cms:manage | yes | yes | no | no |
| voucher:manage | yes | yes | no | no |
| delivery:manage | yes | no | yes | no |
| delivery:assign_driver | yes | no | yes | no |
| online_order:read | yes | yes | yes | no |

Kasir handles operations (confirm orders, assign drivers). Marketing handles content (CMS, vouchers).

### Customer Auth Separation

Customer authentication is completely separate from staff RBAC:
- Staff: `users` table + `roles` + `permissions` + StaffAuthGuard
- Customers: `customers` table + CustomerAuthGuard
- No overlap. Customers have no roles/permissions — all `/api/customer-app/*` endpoints are accessible by any authenticated customer.

---

## 8. Complete API Summary

### Customer App (`/api/customer-app/*`) — CustomerAuthGuard

```
# Auth
POST   /api/customers/auth/otp/request
POST   /api/customers/auth/otp/verify
POST   /api/customers/auth/refresh

# Profile
GET    /api/customers/profile
PATCH  /api/customers/profile
GET    /api/customers/addresses
POST   /api/customers/addresses
PATCH  /api/customers/addresses/:id
DELETE /api/customers/addresses/:id

# Home & Menu
GET    /api/customer-app/home
GET    /api/customer-app/menu
GET    /api/customer-app/menu/:id

# Orders
POST   /api/customer-app/orders
GET    /api/customer-app/orders
GET    /api/customer-app/orders/:id
PATCH  /api/customer-app/orders/:id/cancel
GET    /api/customer-app/orders/:id/delivery
PATCH  /api/customer-app/orders/:id/received

# Vouchers
POST   /api/customer-app/vouchers/validate

# Notifications
POST   /api/customer-app/devices
DELETE /api/customer-app/devices/:token
GET    /api/customer-app/notifications
PATCH  /api/customer-app/notifications/:id/read
PATCH  /api/customer-app/notifications/read-all
```

### Dashboard/Staff (`/api/*`) — StaffAuthGuard + RBAC

```
# Online Order Management (Kasir, Owner)
GET    /api/online-orders                     # list pending/active online orders
PATCH  /api/online-orders/:id/confirm         # confirm order -> kitchen
PATCH  /api/online-orders/:id/reject          # reject order

# Delivery Management (Kasir, Owner)
POST   /api/deliveries/:orderId/assign
PATCH  /api/deliveries/:orderId/status

# Voucher Management (Owner, Marketing)
GET    /api/vouchers
POST   /api/vouchers
GET    /api/vouchers/:id
PATCH  /api/vouchers/:id
DELETE /api/vouchers/:id
GET    /api/vouchers/:id/usages

# CMS (Owner, Marketing)
GET    /api/cms/banners
POST   /api/cms/banners
PATCH  /api/cms/banners/:id
DELETE /api/cms/banners/:id
PATCH  /api/cms/banners/reorder
GET    /api/cms/sections
POST   /api/cms/sections
PATCH  /api/cms/sections/:id
DELETE /api/cms/sections/:id
PATCH  /api/cms/sections/reorder
GET    /api/cms/store-info
PATCH  /api/cms/store-info
GET    /api/cms/preview

# Upload
POST   /api/uploads/image
```

---

## 9. Database Schema Summary (All New Tables)

```
customers
customer_addresses
customer_otps
customer_devices
vouchers
voucher_usages
deliveries
cms_banners
cms_sections
cms_store_info
notifications
```

Extended existing table: `orders` (10 new columns: source, order_type, customer_id, table_number, pickup_time, customer_notes, confirmed_at, voucher_id, voucher_discount, delivery_fee)

---

## 10. Connection to Phase 1, 2 & 3

- Menu & prices from Phase 1 -> displayed in customer app via CMS
- Online orders enter the same kitchen display pipeline (Phase 1 WebSocket)
- HPP & profit tracking applies to online orders
- Stock decreases automatically on order (Phase 2)
- Online revenue enters finance reports (Phase 3)
- Voucher discounts are factored into profit calculations
