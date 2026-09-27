# POS System Phase 2 — Inventory Management & Finance Design Spec

## Overview

Phase 2 extends the POS system with two major modules:
1. **Inventory Management** — supplier management, purchase orders (with owner approval), automatic stock deduction from transactions, stock opname, low-stock notifications
2. **Finance & Reporting** — daily cash closing, optional operational costs, monthly P&L reports, PDF/Excel export

These build on top of Phase 1's existing transaction, ingredient, and menu infrastructure.

## Tech Stack (same as Phase 1)

- **Backend:** NestJS, PostgreSQL, Prisma ORM, JWT Auth
- **Web Dashboard:** Next.js, shadcn/ui, Tailwind CSS, Recharts
- **Flutter App:** Additional inventory screens for INVENTORY role (optional, can be web-only first)

## Implementation Approach

**Inventory first, Finance second.** Inventory is a prerequisite for accurate finance data — stock movements from transactions must work before reports can be trusted.

Sub-plan A: Inventory Management (backend + web)
Sub-plan B: Finance & Reporting (backend + web)

---

## Database Schema

### Inventory Tables

```prisma
model Supplier {
  id          String   @id @default(uuid())
  name        String
  contactName String?
  phone       String?
  email       String?
  address     String?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  purchaseOrders PurchaseOrder[]

  @@map("suppliers")
}

model Warehouse {
  id        String   @id @default(uuid())
  name      String
  location  String?
  isDefault Boolean  @default(false)
  createdAt DateTime @default(now())
  stockMovements StockMovement[]
  purchaseOrders PurchaseOrder[]
  stockOpnames   StockOpname[]

  @@map("warehouses")
}

model PurchaseOrder {
  id          String         @id @default(uuid())
  poNumber    String         @unique
  supplierId  String
  warehouseId String
  status      POStatus       @default(DRAFT)
  totalAmount Float          @default(0)
  notes       String?
  createdById String
  approvedById String?
  approvedAt  DateTime?
  receivedAt  DateTime?
  createdAt   DateTime       @default(now())
  updatedAt   DateTime       @updatedAt
  supplier    Supplier       @relation(fields: [supplierId], references: [id])
  warehouse   Warehouse      @relation(fields: [warehouseId], references: [id])
  createdBy   User           @relation("po_created", fields: [createdById], references: [id])
  approvedBy  User?          @relation("po_approved", fields: [approvedById], references: [id])
  items       PurchaseOrderItem[]

  @@map("purchase_orders")
}

enum POStatus {
  DRAFT
  SUBMITTED
  APPROVED
  RECEIVED
  CANCELLED
}

model PurchaseOrderItem {
  id           String        @id @default(uuid())
  poId         String
  ingredientId String
  qtyOrdered   Float
  qtyReceived  Float         @default(0)
  unit         String
  unitPrice    Float
  purchaseOrder PurchaseOrder @relation(fields: [poId], references: [id], onDelete: Cascade)
  ingredient   Ingredient    @relation(fields: [ingredientId], references: [id])

  @@map("purchase_order_items")
}

model StockMovement {
  id            String        @id @default(uuid())
  ingredientId  String
  warehouseId   String
  type          StockMovementType
  qty           Float
  referenceType String        // PO, TRANSACTION, OPNAME, MANUAL
  referenceId   String?
  notes         String?
  createdById   String
  createdAt     DateTime      @default(now())
  ingredient    Ingredient    @relation(fields: [ingredientId], references: [id])
  warehouse     Warehouse     @relation(fields: [warehouseId], references: [id])
  createdBy     User          @relation(fields: [createdById], references: [id])

  @@map("stock_movements")
}

enum StockMovementType {
  IN
  OUT
  ADJUSTMENT
}

model StockOpname {
  id          String            @id @default(uuid())
  warehouseId String
  status      OpnameStatus      @default(DRAFT)
  notes       String?
  createdById String
  completedAt DateTime?
  createdAt   DateTime          @default(now())
  warehouse   Warehouse         @relation(fields: [warehouseId], references: [id])
  createdBy   User              @relation(fields: [createdById], references: [id])
  items       StockOpnameItem[]

  @@map("stock_opnames")
}

enum OpnameStatus {
  DRAFT
  COMPLETED
}

model StockOpnameItem {
  id           String      @id @default(uuid())
  opnameId     String
  ingredientId String
  systemQty    Float
  actualQty    Float
  difference   Float
  opname       StockOpname @relation(fields: [opnameId], references: [id], onDelete: Cascade)
  ingredient   Ingredient  @relation(fields: [ingredientId], references: [id])

  @@map("stock_opname_items")
}

model Notification {
  id        String   @id @default(uuid())
  userId    String
  type      String   // LOW_STOCK, PO_APPROVED, PO_RECEIVED, etc.
  title     String
  message   String
  isRead    Boolean  @default(false)
  data      Json?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])

  @@map("notifications")
}
```

### Finance Tables

```prisma
model DailyClosing {
  id          String       @id @default(uuid())
  cashierId   String
  date        DateTime
  systemTotal Float
  actualCash  Float
  difference  Float
  notes       String?
  status      ClosingStatus @default(OPEN)
  createdAt   DateTime     @default(now())
  cashier     User         @relation(fields: [cashierId], references: [id])

  @@unique([cashierId, date])
  @@map("daily_closings")
}

enum ClosingStatus {
  OPEN
  CLOSED
}

model OperationalCost {
  id          String   @id @default(uuid())
  name        String
  category    OpexCategory
  amount      Float
  periodMonth Int
  periodYear  Int
  notes       String?
  createdById String
  createdAt   DateTime @default(now())
  createdBy   User     @relation(fields: [createdById], references: [id])

  @@map("operational_costs")
}

enum OpexCategory {
  SALARY
  RENT
  UTILITY
  OTHER
}

model MonthlyReport {
  id                   String   @id @default(uuid())
  month                Int
  year                 Int
  totalRevenue         Float
  totalHpp             Float
  grossProfit          Float
  totalOperationalCost Float    @default(0)
  totalPromoDiscount   Float    @default(0)
  netProfit            Float
  generatedAt          DateTime @default(now())

  @@unique([month, year])
  @@map("monthly_reports")
}
```

### Relations to Add on Existing Models

```prisma
// Add to User model:
purchaseOrdersCreated  PurchaseOrder[] @relation("po_created")
purchaseOrdersApproved PurchaseOrder[] @relation("po_approved")
stockMovements         StockMovement[]
stockOpnames           StockOpname[]
notifications          Notification[]
dailyClosings          DailyClosing[]
operationalCosts       OperationalCost[]

// Add to Ingredient model:
purchaseOrderItems PurchaseOrderItem[]
stockMovements     StockMovement[]
stockOpnameItems   StockOpnameItem[]
```

---

## Core Business Logic

### 1. Automatic Stock Deduction from Transactions

When `PosService.create()` processes a transaction:

**Menu items:**
- For each menu item in transaction → loop through its recipes
- For each recipe: deduct `qtyNeeded * transactionQty` from ingredient's `stockQty`
- Create `StockMovement` (type: OUT, referenceType: TRANSACTION, referenceId: transaction.id)

**Retail items:**
- Deduct `qty` from `RetailItem.stockQty`
- No stock_movement needed (retail tracks its own stockQty)

**Low stock check:**
- After deduction, if `ingredient.stockQty <= ingredient.minStock` → create `Notification` (type: LOW_STOCK) for all users with `stock:read` permission

### 2. Purchase Order Flow

```
DRAFT → SUBMITTED → APPROVED → RECEIVED
                  ↘ CANCELLED (from SUBMITTED or APPROVED)
```

- **DRAFT:** Create PO with supplier, warehouse, items (ingredient + qty + unit price). `totalAmount` auto-calculated from items.
- **SUBMITTED:** Inventory staff submits. Owner gets notification (PO_SUBMITTED).
- **APPROVED:** Owner approves. `approvedBy` + `approvedAt` set. Notification to creator (PO_APPROVED).
- **RECEIVED:** Input `qtyReceived` per item. For each item:
  - Add to `ingredient.stockQty`
  - Create `StockMovement` (type: IN, referenceType: PO)
  - If `unitPrice` differs from current `ingredient.pricePerUnit` → update price + create `IngredientPriceLog`
  - Set `receivedAt`
- **CANCELLED:** From SUBMITTED or APPROVED only. Notification to relevant parties.

### 3. Stock Opname

- Create opname session for a warehouse
- System auto-fills `systemQty` from current `ingredient.stockQty` for all ingredients
- User inputs `actualQty` for each ingredient
- `difference = actualQty - systemQty` (positive = surplus, negative = shortage)
- On complete:
  - For each item with `difference != 0`:
    - Update `ingredient.stockQty = actualQty`
    - Create `StockMovement` (type: ADJUSTMENT, referenceType: OPNAME)

### 4. Daily Closing

- Kasir initiates closing for today
- System calculates `systemTotal`: sum of all CASH transactions by this cashier on this date
- Kasir inputs `actualCash` (physical cash count)
- `difference = actualCash - systemTotal`
- Status set to CLOSED (immutable after)

### 5. Monthly Report Generation

- Triggered manually or auto on first access of a month's report
- Aggregates from transactions:
  - `totalRevenue = SUM(totalAmount)`
  - `totalHpp = SUM(totalHpp)`
  - `grossProfit = totalRevenue - totalHpp`
  - `totalPromoDiscount = SUM(promoDiscount)`
- Optional: `totalOperationalCost = SUM(operational_costs for that month)`
- `netProfit = grossProfit - totalOperationalCost - totalPromoDiscount`

### 6. Export (PDF & Excel)

- Monthly report export to PDF: formatted P&L statement
- Monthly report export to Excel: detailed breakdown with sheets (summary, transactions, costs)
- Use server-side generation: `pdfkit` or `@react-pdf/renderer` for PDF, `exceljs` for Excel

---

## Permissions

| Permission | OWNER | KASIR | KITCHEN | INVENTORY | FINANCE |
|---|---|---|---|---|---|
| `supplier:manage` | yes | | | yes | |
| `warehouse:manage` | yes | | | yes | |
| `po:create` | yes | | | yes | |
| `po:approve` | yes | | | | |
| `po:receive` | yes | | | yes | |
| `stock:read` | yes | | yes | yes | |
| `stock:opname` | yes | | | yes | |
| `closing:create` | yes | yes | | | |
| `closing:read` | yes | | | | yes |
| `report:read` | yes | | | | yes |
| `report:export` | yes | | | | yes |
| `opex:manage` | yes | | | | yes |
| `notification:read` | all (own) | all (own) | all (own) | all (own) | all (own) |

---

## API Endpoints

### Inventory

```
# Suppliers
GET    /suppliers              — list (paginated)
GET    /suppliers/:id          — detail
POST   /suppliers              — create
PATCH  /suppliers/:id          — update
DELETE /suppliers/:id          — delete

# Warehouses
GET    /warehouses             — list
POST   /warehouses             — create
PATCH  /warehouses/:id         — update
DELETE /warehouses/:id         — delete

# Purchase Orders
GET    /purchase-orders        — list (paginated, filter by status/supplier/date)
GET    /purchase-orders/:id    — detail with items
POST   /purchase-orders        — create (DRAFT)
PATCH  /purchase-orders/:id    — update (DRAFT only)
DELETE /purchase-orders/:id    — delete (DRAFT only)
PATCH  /purchase-orders/:id/submit   — DRAFT → SUBMITTED
PATCH  /purchase-orders/:id/approve  — SUBMITTED → APPROVED
PATCH  /purchase-orders/:id/receive  — APPROVED → RECEIVED (body: items with qtyReceived)
PATCH  /purchase-orders/:id/cancel   — SUBMITTED|APPROVED → CANCELLED

# Stock Movements
GET    /stock-movements        — list (filter by ingredient, warehouse, type, date range)

# Stock Opname
GET    /stock-opnames          — list
GET    /stock-opnames/:id      — detail with items
POST   /stock-opnames          — create (auto-fill systemQty)
PATCH  /stock-opnames/:id      — update items (actualQty)
PATCH  /stock-opnames/:id/complete — DRAFT → COMPLETED (adjust stock)

# Notifications
GET    /notifications          — list (own, paginated)
GET    /notifications/unread-count — count unread
PATCH  /notifications/:id/read — mark as read
PATCH  /notifications/read-all — mark all as read
```

### Finance

```
# Daily Closing
GET    /closings               — list (filter by cashier, date range)
GET    /closings/:id           — detail
POST   /closings               — create + close (auto-calc systemTotal)

# Operational Costs (optional feature)
GET    /operational-costs      — list (filter by month/year/category)
POST   /operational-costs      — create
PATCH  /operational-costs/:id  — update
DELETE /operational-costs/:id  — delete

# Monthly Reports
GET    /reports/monthly                    — list all generated reports
GET    /reports/monthly/:year/:month       — get or generate report
GET    /reports/monthly/:year/:month/export?format=pdf|excel — export
GET    /reports/profit-loss?startDate=&endDate= — custom date range P&L
```

---

## Web Dashboard Pages (New)

### Inventory Pages

- **/suppliers** — CRUD table, search, active/inactive filter
- **/warehouses** — simple CRUD list
- **/purchase-orders** — table with status tabs (DRAFT/SUBMITTED/APPROVED/RECEIVED), detail page with approval/receive actions
- **/stock-movements** — read-only log table with filters
- **/stock-opnames** — list + create/edit opname sessions
- **Notification bell** — header component, dropdown with unread notifications

### Finance Pages

- **/closings** — list of daily closings, create new closing modal
- **/operational-costs** — CRUD table with month/year filter
- **/reports** — monthly report list, detail view with P&L breakdown, export buttons (PDF/Excel)

### Updated Existing Pages

- **/ingredients** — add stock movement history, low-stock badge
- **/dashboard** — add low-stock alerts card, recent PO card

---

## Sidebar Menu Updates

Add to sidebar (RBAC filtered):

| Menu Item | Route | Permission |
|---|---|---|
| Supplier | /suppliers | supplier:manage |
| Gudang | /warehouses | warehouse:manage |
| Purchase Order | /purchase-orders | po:create |
| Stok Movements | /stock-movements | stock:read |
| Stock Opname | /stock-opnames | stock:opname |
| Tutup Kasir | /closings | closing:create |
| Biaya Operasional | /operational-costs | opex:manage |
| Laporan | /reports | report:read |

---

## NPM Dependencies (New)

### Backend
- `pdfkit` or `@react-pdf/renderer` — PDF generation
- `exceljs` — Excel export

### Web
- No new deps needed (existing shadcn/ui + recharts sufficient)

---

## Seed Data Updates

Add to existing seed:
- 1 default warehouse ("Gudang Utama", isDefault: true)
- 2 suppliers (e.g., "PT Bahan Segar", "UD Maju Jaya")
- New permissions for Phase 2 roles
- INVENTORY and FINANCE role permissions mapped

---

## Out of Scope

- Tax calculation (manual outside system)
- Multi-outlet / branch management
- Supplier payment tracking
- Delivery/shipping from suppliers
- Barcode/QR scanning for stock
