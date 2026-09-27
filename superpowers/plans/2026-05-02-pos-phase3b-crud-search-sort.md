# POS Phase 3B — CRUD Modals + Search/Filter/Sort Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add create/edit/delete dialogs, search bar, column filters, and sortable columns to all 11 management pages.

**Architecture:** Build 4 reusable shared components (DataTable, SearchInput, ColumnFilter, DeleteDialog), then apply to each page with page-specific form dialogs. Client-side search/filter/sort on loaded data.

**Tech Stack:** Next.js, shadcn/ui (Dialog, AlertDialog, Select, Input, Table), Tailwind CSS

**Spec:** `docs/superpowers/specs/2026-05-02-pos-phase3-polish-design.md`

---

## File Structure

### New Shared Components
```
web/src/components/shared/
├── data-table.tsx        — Generic sortable table (column defs, sort state, click headers)
├── search-input.tsx      — Debounced search input (300ms)
├── column-filter.tsx     — Dropdown filter (label + options + onChange)
└── delete-dialog.tsx     — Reusable AlertDialog (title, message, onConfirm, loading)
```

### Modified Pages (each gets form dialogs + search/filter/sort)
```
web/src/app/(dashboard)/menu/page.tsx
web/src/app/(dashboard)/ingredients/page.tsx
web/src/app/(dashboard)/retail/page.tsx
web/src/app/(dashboard)/promos/page.tsx
web/src/app/(dashboard)/users/page.tsx
web/src/app/(dashboard)/suppliers/page.tsx
web/src/app/(dashboard)/warehouses/page.tsx
web/src/app/(dashboard)/purchase-orders/page.tsx
web/src/app/(dashboard)/stock-opnames/page.tsx
web/src/app/(dashboard)/operational-costs/page.tsx
web/src/app/(dashboard)/closings/page.tsx
web/src/app/(dashboard)/transactions/page.tsx  — search/filter/sort only (read-only)
```

---

### Task 1: Shared Components

**Files:** Create 4 files in `web/src/components/shared/`

- [ ] **Step 1: Create SearchInput**

Debounced text input (300ms). Props: `value`, `onChange`, `placeholder`. Uses `useDebounce` hook. Lucide Search icon prefix.

- [ ] **Step 2: Create ColumnFilter**

Select dropdown. Props: `label`, `options: {value, label}[]`, `value`, `onChange`. "Semua" as default/empty option.

- [ ] **Step 3: Create DeleteDialog**

shadcn AlertDialog. Props: `open`, `onOpenChange`, `title`, `description`, `onConfirm`, `loading`. Red "Hapus" button + "Batal" cancel.

- [ ] **Step 4: Create DataTable**

Generic wrapper around shadcn Table. Props:
- `columns: { key, label, sortable?, render? }[]`
- `data: T[]`
- `sortKey`, `sortDirection`, `onSort(key)`
- Renders TableHeader with clickable sort indicators (ArrowUpDown icon), TableBody with rows.

- [ ] **Step 5: Build + commit**

```bash
npx next build
git add src/components/shared/
git commit -m "feat: add shared data-table, search, filter, delete-dialog components"
```

---

### Task 2: Menu Page — CRUD + Search/Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/menu/page.tsx`

- [ ] **Step 1: Add state management**

States: `items`, `loading`, `search`, `categoryFilter`, `statusFilter`, `sortKey`, `sortDir`, `editItem`, `deleteItem`, `showCreate`. Client-side filter/sort pipeline: items → search filter → category filter → status filter → sort.

- [ ] **Step 2: Create MenuFormDialog**

shadcn Dialog. Fields: name (Input), category (Select FOOD/BEVERAGE), sellingPrice (Input number), imageUrl (Input), isActive (Switch). Recipe editor: dynamic list with ingredient select + qtyNeeded + unit inputs. Add/remove recipe rows. On submit: `menuService.create()` or `menuService.update()`.

- [ ] **Step 3: Wire SearchInput + ColumnFilter + DataTable + DeleteDialog**

SearchInput filters by name. ColumnFilter for category (ALL/FOOD/BEVERAGE) + status (ALL/Active/Inactive). DataTable with sortable columns (name, sellingPrice, hpp, margin). Row actions: Edit → opens MenuFormDialog with data, Delete → opens DeleteDialog.

- [ ] **Step 4: Build + commit**

---

### Task 3: Ingredients Page — CRUD + Search/Sort + Low-Stock Badge

**Files:** Modify: `web/src/app/(dashboard)/ingredients/page.tsx`

- [ ] **Step 1: Add IngredientFormDialog**

Fields: name, unit, pricePerUnit, stockQty, minStock. On submit: create/update.

- [ ] **Step 2: Wire search + sort + delete + low-stock badge**

SearchInput by name. Sortable: name, pricePerUnit, stockQty. Red badge if `stockQty <= minStock`. Row actions: Edit, Delete.

- [ ] **Step 3: Build + commit**

---

### Task 4: Retail Page — CRUD + Search/Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/retail/page.tsx`

- [ ] **Step 1: Add RetailFormDialog**

Fields: name, category (Input), buyPrice, sellingPrice, stockQty, minStock, imageUrl.

- [ ] **Step 2: Wire search + category filter + sort + delete**

Filter by category. Sort: name, sellingPrice, stockQty.

- [ ] **Step 3: Build + commit**

---

### Task 5: Promos Page — CRUD + Search/Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/promos/page.tsx`

- [ ] **Step 1: Add PromoFormDialog**

Fields: name, type (Select 5 types), value, minPurchaseAmount (conditional on type), startDate (date input), endDate, isActive (Switch), maxUsage.

- [ ] **Step 2: Wire search + type filter + status filter + sort + delete**

Filter: type, active. Sort: name, startDate, usedCount.

- [ ] **Step 3: Build + commit**

---

### Task 6: Users Page — CRUD + Search/Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/users/page.tsx`

- [ ] **Step 1: Add UserFormDialog**

Create: name, email, password, roleId (Select from roles via `userService.getRoles()`). Edit: name, email, roleId, isActive (no password field).

- [ ] **Step 2: Wire search + role filter + status filter + sort + delete**

Filter: role, active. Sort: name, email, createdAt.

- [ ] **Step 3: Build + commit**

---

### Task 7: Suppliers Page — CRUD + Search/Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/suppliers/page.tsx`

- [ ] **Step 1: Add SupplierFormDialog**

Fields: name, contactName, phone, email, address, isActive.

- [ ] **Step 2: Wire search + status filter + sort + delete**

Filter: active. Sort: name.

- [ ] **Step 3: Build + commit**

---

### Task 8: Warehouses Page — CRUD + Sort

**Files:** Modify: `web/src/app/(dashboard)/warehouses/page.tsx`

- [ ] **Step 1: Add WarehouseFormDialog**

Fields: name, location, isDefault (Switch).

- [ ] **Step 2: Wire sort + delete**

Sort: name. No search needed (small list).

- [ ] **Step 3: Build + commit**

---

### Task 9: Purchase Orders Page — Create/Receive + Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/purchase-orders/page.tsx`

- [ ] **Step 1: Add CreatePODialog**

Fields: supplierId (Select suppliers), warehouseId (Select warehouses), notes. Dynamic items list: ingredientId (Select) + qtyOrdered + unit + unitPrice. Add/remove rows. Total auto-calculated.

- [ ] **Step 2: Add ReceivePODialog**

List PO items with qtyReceived input per item. Submit → `purchaseOrderService.receive()`.

- [ ] **Step 3: Add status action buttons**

Row actions based on status: DRAFT → Submit/Delete, SUBMITTED → Approve/Cancel, APPROVED → Receive/Cancel. Wire to service methods.

- [ ] **Step 4: Wire status tabs/filter + sort**

Filter: status (tabs ALL/DRAFT/SUBMITTED/APPROVED/RECEIVED/CANCELLED). Sort: poNumber, totalAmount, createdAt.

- [ ] **Step 5: Build + commit**

---

### Task 10: Stock Opnames Page — Create/Edit/Complete + Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/stock-opnames/page.tsx`

- [ ] **Step 1: Add CreateOpnameDialog**

Fields: warehouseId (Select), notes. Submit → `stockOpnameService.create()`.

- [ ] **Step 2: Add EditOpnameDialog**

Table of opname items. Each row: ingredient name, systemQty (readonly), actualQty (editable input), difference (auto-calc). Submit → `stockOpnameService.updateItems()`.

- [ ] **Step 3: Add Complete action + status filter**

Complete button for DRAFT opnames. Filter: status. Sort: createdAt.

- [ ] **Step 4: Build + commit**

---

### Task 11: Operational Costs Page — CRUD + Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/operational-costs/page.tsx`

- [ ] **Step 1: Add OpCostFormDialog**

Fields: name, category (Select SALARY/RENT/UTILITY/OTHER), amount, periodMonth (Select 1-12), periodYear (Input), notes.

- [ ] **Step 2: Wire search + category filter + period filter + sort + delete**

Filter: category, month/year. Sort: name, amount.

- [ ] **Step 3: Build + commit**

---

### Task 12: Closings Page — Create + Filter/Sort

**Files:** Modify: `web/src/app/(dashboard)/closings/page.tsx`

- [ ] **Step 1: Add CreateClosingDialog**

Fields: actualCash (Input number), notes (Textarea). On success: show result with systemTotal + difference.

- [ ] **Step 2: Wire date filter + sort**

Filter: date range. Sort: date, difference.

- [ ] **Step 3: Build + commit**

---

### Task 13: Transactions Page — Search/Filter/Sort (read-only)

**Files:** Modify: `web/src/app/(dashboard)/transactions/page.tsx`

- [ ] **Step 1: Add search + filters + sort**

SearchInput by transactionCode. Filter: paymentMethod (CASH/QRIS/EDC), date range. Sort: transactionCode, totalAmount, totalProfit, createdAt.

- [ ] **Step 2: Build + commit**

---

### Task 14: Final Build + Verification

- [ ] **Step 1: Build web**

```bash
cd ~/Downloads/PROJECT/pos/web && npx next build
```

Expected: All routes compile clean.

- [ ] **Step 2: Commit**

```bash
git add -A
git commit -m "feat: Phase 3B complete — CRUD modals + search/filter/sort all pages"
```
