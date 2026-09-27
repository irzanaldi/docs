# POS Phase 3A — Dashboard Analytics Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build full analytics dashboard with summary cards, 4 charts (Recharts), low-stock alerts, recent transactions, and date range picker.

**Architecture:** Client-side Next.js page, calls existing backend dashboard/ingredient/transaction services. All components in `web/src/components/dashboard/`. Recharts for charts, shadcn Calendar+Popover for date picker.

**Tech Stack:** Next.js, Recharts, shadcn/ui (Card, Popover, Calendar, Table, Badge), Tailwind CSS

**Spec:** `docs/superpowers/specs/2026-05-02-pos-phase3-polish-design.md`

---

## File Structure

### New Files
```
web/src/components/dashboard/
├── date-range-picker.tsx        — Calendar range picker (Popover + Calendar)
├── summary-cards.tsx            — 4 stat cards (Revenue, HPP, Profit, Transactions)
├── revenue-trend-chart.tsx      — LineChart (revenue + profit daily)
├── profit-category-chart.tsx    — PieChart (MENU vs RETAIL)
├── top-profit-chart.tsx         — Horizontal BarChart (top 10 by profit)
├── top-qty-chart.tsx            — Horizontal BarChart (top 10 by qty)
├── low-stock-card.tsx           — Alert table (ingredients below minStock)
└── recent-transactions-card.tsx — Last 10 transactions table
```

### Modified Files
```
web/src/app/(dashboard)/dashboard/page.tsx — Replace placeholder with full dashboard
```

---

### Task 1: Date Range Picker Component

**Files:** Create: `web/src/components/dashboard/date-range-picker.tsx`

- [ ] **Step 1: Create DateRangePicker**

A controlled component using shadcn Popover + Calendar. Props: `startDate`, `endDate`, `onChange(start, end)`. Display formatted range as button text. Calendar in `mode="range"`.

- [ ] **Step 2: Verify import works from dashboard page**

---

### Task 2: Summary Cards Component

**Files:** Create: `web/src/components/dashboard/summary-cards.tsx`

- [ ] **Step 1: Create SummaryCards**

Props: `DashboardSummary` data. 4 cards in grid:
- Revenue (blue icon DollarSign) — `formatCurrency(totalRevenue)`
- HPP (red icon TrendingDown) — `formatCurrency(totalHpp)`
- Profit (green icon TrendingUp) — `formatCurrency(totalProfit)` + `formatPercent(profitMargin)`
- Transactions (purple icon ShoppingCart) — `formatNumber(transactionCount)`

Each card: shadcn Card, icon + label top, big number bottom.

---

### Task 3: Revenue Trend Chart

**Files:** Create: `web/src/components/dashboard/revenue-trend-chart.tsx`

- [ ] **Step 1: Create RevenueTrendChart**

Props: `data: SalesTrendPoint[]`. Recharts `ResponsiveContainer` + `LineChart` + `CartesianGrid` + `XAxis` (date) + `YAxis` (formatted currency) + 2 `Line`s (revenue=blue, profit=green) + `Tooltip` + `Legend`. Wrapped in shadcn Card with title "Tren Pendapatan".

---

### Task 4: Profit by Category Chart

**Files:** Create: `web/src/components/dashboard/profit-category-chart.tsx`

- [ ] **Step 1: Create ProfitCategoryChart**

Props: `data: ProfitByCategory[]`. Recharts `PieChart` with `Pie` (donut via innerRadius/outerRadius), custom colors (MENU=blue, RETAIL=orange), `Cell` per entry, `Legend`, `Tooltip`. Wrapped in Card with title "Profit per Kategori".

---

### Task 5: Top Profit + Top Qty Charts

**Files:** Create: `web/src/components/dashboard/top-profit-chart.tsx`, `web/src/components/dashboard/top-qty-chart.tsx`

- [ ] **Step 1: Create TopProfitChart**

Props: `data: TopMenuByProfit[]`. Recharts horizontal `BarChart` (`layout="vertical"`), `XAxis` type number, `YAxis` dataKey menu name, `Bar` dataKey totalProfit (green). Card title "Top Menu by Profit".

- [ ] **Step 2: Create TopQtyChart**

Same pattern. Props: `data: TopMenuByQty[]`. Bar dataKey totalQty (blue). Card title "Top Menu by Quantity".

---

### Task 6: Low Stock Card

**Files:** Create: `web/src/components/dashboard/low-stock-card.tsx`

- [ ] **Step 1: Create LowStockCard**

Props: `ingredients: Ingredient[]` (pre-filtered to low stock). Card with title "Stok Rendah" + warning icon. Table: name, currentStock, minStock, unit. Badge: red if stockQty == 0, yellow if stockQty <= minStock. "Lihat Semua →" link to /ingredients.

---

### Task 7: Recent Transactions Card

**Files:** Create: `web/src/components/dashboard/recent-transactions-card.tsx`

- [ ] **Step 1: Create RecentTransactionsCard**

Props: `transactions: Transaction[]`. Card with title "Transaksi Terbaru". Table: code, cashier, paymentMethod badge, total (formatCurrency), time (formatDateTime). "Lihat Semua →" link to /transactions.

---

### Task 8: Wire Dashboard Page

**Files:** Modify: `web/src/app/(dashboard)/dashboard/page.tsx`

- [ ] **Step 1: Replace placeholder with full dashboard**

State: `startDate`, `endDate` (default last 30 days), loading states. `useEffect` fetches all data on date change:
- `dashboardService.getSummary(filters)` → SummaryCards
- `dashboardService.getSalesTrend(filters)` → RevenueTrendChart
- `dashboardService.getProfitByCategory(filters)` → ProfitCategoryChart
- `dashboardService.getTopMenuByProfit(filters)` → TopProfitChart
- `dashboardService.getTopMenuByQty(filters)` → TopQtyChart
- `ingredientService.getAll({ limit: 100 })` → filter low stock → LowStockCard
- `transactionService.getAll({ limit: 10 })` → RecentTransactionsCard

Layout: DateRangePicker top → SummaryCards → 2-col (RevenueTrend + ProfitCategory) → 2-col (TopProfit + TopQty) → 2-col (LowStock + RecentTransactions)

- [ ] **Step 2: Build + verify**

```bash
cd ~/Downloads/PROJECT/pos/web && npx next build
```

- [ ] **Step 3: Commit**

```bash
git add src/components/dashboard/ src/app/\(dashboard\)/dashboard/
git commit -m "feat: add full dashboard with charts, cards, low-stock alerts"
```
