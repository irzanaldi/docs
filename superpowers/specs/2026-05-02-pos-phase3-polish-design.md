# POS System Phase 3 — Polish & Production Ready Design Spec

## Overview

Phase 3 enhances the existing POS web dashboard with real analytics, CRUD modals for all management pages, and search/filter/sort on all tables. No new backend modules — uses existing API services.

## Implementation Approach

Split into 2 sub-plans:
- **Phase 3A:** Dashboard analytics (charts, cards, filters, low-stock alerts)
- **Phase 3B:** CRUD modals + Search/Filter/Sort for all 11 management pages

## Tech Stack (existing)

- **Web:** Next.js, shadcn/ui, Tailwind CSS, Recharts, Lucide icons
- **Backend:** No changes needed — all APIs already exist from Phase 1-2

---

## Phase 3A: Dashboard Analytics

### Dashboard Page Layout

Top section: Date picker (calendar range picker, shadcn Popover + Calendar)

Row 1 — **4 Summary Cards:**
- Total Revenue (blue) — `formatCurrency(totalRevenue)`
- Total HPP (red) — `formatCurrency(totalHpp)`
- Total Profit (green) — `formatCurrency(totalProfit)` + margin %
- Transactions (purple) — count

Row 2 — **2 Charts side by side:**
- Left (60%): **Revenue Trend** — Recharts LineChart, daily data points, 2 lines (revenue + profit), tooltip with formatted values
- Right (40%): **Profit by Category** — Recharts PieChart/DonutChart (MENU vs RETAIL), legend + percentages

Row 3 — **2 Charts side by side:**
- Left (50%): **Top Menu by Profit** — Recharts horizontal BarChart, top 10, bar = totalProfit, label = menu name
- Right (50%): **Top Menu by Quantity** — Recharts horizontal BarChart, top 10, bar = totalQty

Row 4 — **2 Cards side by side:**
- Left (50%): **Low Stock Alerts** — Card with table: ingredient name, current stock, min stock, unit. Red/yellow badges. Links to /ingredients.
- Right (50%): **Recent Transactions** — Card with table: last 10 transactions (code, cashier, total, time)

### Date Filter

- shadcn Popover + Calendar (date range picker)
- Default: last 30 days
- On change: re-fetch all dashboard data
- Display selected range as text: "1 Apr 2026 — 30 Apr 2026"

### Data Sources (existing services)

All already implemented in `web/src/services/dashboard.service.ts`:
- `getSummary(filters)` → summary cards
- `getSalesTrend(filters)` → revenue trend chart
- `getTopMenuByProfit(filters)` → top profit chart
- `getTopMenuByQty(filters)` → top qty chart
- `getProfitByCategory(filters)` → category donut

Additional calls:
- `ingredientService.getAll()` → filter where `stockQty <= minStock` for low-stock card
- `transactionService.getAll({ limit: 10 })` → recent transactions

### New Components

```
web/src/components/dashboard/
├── summary-cards.tsx          — 4 stat cards
├── revenue-trend-chart.tsx    — LineChart
├── profit-category-chart.tsx  — PieChart
├── top-profit-chart.tsx       — horizontal BarChart
├── top-qty-chart.tsx          — horizontal BarChart
├── low-stock-card.tsx         — alert table
├── recent-transactions-card.tsx — recent tx table
└── date-range-picker.tsx      — calendar range picker
```

---

## Phase 3B: CRUD Modals + Search/Filter/Sort

### CRUD Modals Pattern

Every management page gets:
- **Create dialog** — shadcn Dialog with form, validation, submit → service.create() → refresh table
- **Edit dialog** — same Dialog, pre-filled with existing data, submit → service.update()
- **Delete confirmation** — shadcn AlertDialog with confirm/cancel, submit → service.delete()

Common pattern per page:
```
PageComponent
├── Header (title + "Tambah" button → opens create dialog)
├── SearchBar + Filters
├── DataTable (sortable columns)
│   └── Row actions: Edit, Delete
├── CreateDialog (form)
├── EditDialog (form, pre-filled)
└── DeleteDialog (confirmation)
```

### Pages & Their Forms

**1. Menu** (`/menu`)
- Create/Edit: name, category (select FOOD/BEVERAGE), sellingPrice, imageUrl, isActive
- Recipe editor: dynamic list of ingredients (select ingredient + qtyNeeded + unit), add/remove rows
- Filters: category (ALL/FOOD/BEVERAGE), active status
- Sort: name, sellingPrice, hpp, margin

**2. Ingredients** (`/ingredients`)
- Create/Edit: name, unit, pricePerUnit, stockQty, minStock
- Low-stock badge: red badge if stockQty <= minStock
- Filters: none (search only)
- Sort: name, pricePerUnit, stockQty

**3. Retail** (`/retail`)
- Create/Edit: name, category, buyPrice, sellingPrice, stockQty, minStock, imageUrl
- Filters: category
- Sort: name, sellingPrice, stockQty

**4. Promos** (`/promos`)
- Create/Edit: name, type (select), value, minPurchaseAmount, startDate, endDate, isActive, maxUsage
- Filters: type, active status
- Sort: name, startDate, usedCount

**5. Users** (`/users`)
- Create: name, email, password, roleId (select from roles)
- Edit: name, email, roleId, isActive (no password edit unless explicit)
- Filters: role, active status
- Sort: name, email, createdAt

**6. Suppliers** (`/suppliers`)
- Create/Edit: name, contactName, phone, email, address, isActive
- Filters: active status
- Sort: name

**7. Warehouses** (`/warehouses`)
- Create/Edit: name, location, isDefault
- No filters (small list)
- Sort: name

**8. Purchase Orders** (`/purchase-orders`)
- Create: supplierId (select), warehouseId (select), notes, items (dynamic: ingredientId + qtyOrdered + unit + unitPrice)
- Status actions (not modal): Submit, Approve, Receive, Cancel buttons in row/detail
- Receive modal: list PO items with qtyReceived input per item
- Filters: status (tabs: ALL/DRAFT/SUBMITTED/APPROVED/RECEIVED/CANCELLED)
- Sort: poNumber, totalAmount, createdAt

**9. Stock Opnames** (`/stock-opnames`)
- Create: warehouseId (select), notes → auto-creates with all ingredients
- Edit modal: table of items, input actualQty per item, shows difference
- Complete action button
- Filters: status (DRAFT/COMPLETED)
- Sort: createdAt

**10. Operational Costs** (`/operational-costs`)
- Create/Edit: name, category (select), amount, periodMonth, periodYear, notes
- Filters: category, periodMonth, periodYear
- Sort: name, amount, periodMonth

**11. Daily Closing** (`/closings`)
- Create modal: actualCash input, notes (system shows systemTotal after creation)
- Read-only after creation (CLOSED)
- Filters: date range
- Sort: date, difference

### Search/Filter/Sort Shared Components

```
web/src/components/shared/
├── data-table.tsx            — generic sortable table wrapper
├── search-input.tsx          — debounced search input
├── column-filter.tsx         — dropdown filter for specific column
└── delete-dialog.tsx         — reusable delete confirmation
```

**Search:** Client-side filter on loaded data (most tables < 100 rows). Debounce 300ms.

**Column Filters:** Dropdown selects. Client-side filtering on loaded data.

**Sort:** Click column header → toggle asc/desc. Client-side sort. Arrow indicator on active sort column.

---

## Out of Scope

- Pagination (tables already paginated server-side, but client-side is sufficient for Phase 3 since most data < 100 rows)
- Backend changes (all APIs exist)
- Flutter app changes
- Image upload (imageUrl stays manual text input)
- Inline editing (all editing via modals)
