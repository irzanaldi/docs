# POS System Phase 1 — Web Dashboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the Next.js web dashboard/CMS with RBAC-based navigation, profit analytics, and management pages for all Phase 1 modules.

**Architecture:** Next.js App Router with server and client components. Uses shadcn/ui for UI components, Recharts for charts, and axios for API calls. Auth via JWT stored in cookies. Middleware handles route protection and RBAC.

**Tech Stack:** Next.js 14+, TypeScript, Tailwind CSS, shadcn/ui, Recharts, axios, js-cookie

**Spec:** `docs/superpowers/specs/2026-04-18-pos-system-design.md`
**Backend API Plan:** `docs/superpowers/plans/2026-04-18-pos-backend-api.md`

---

## Backend API Reference (Quick Summary)

| Endpoint | Method | Description |
|---|---|---|
| `POST /auth/login` | POST | Login → `{ accessToken, user }` |
| `GET /auth/me` | GET | Get current user profile |
| `GET /users` | GET | List users (paginated) |
| `GET /users/roles` | GET | List roles |
| `POST /users` | POST | Create user |
| `PATCH /users/:id` | PATCH | Update user |
| `DELETE /users/:id` | DELETE | Delete user |
| `GET /menu` | GET | List menus with HPP |
| `GET /menu/:id` | GET | Single menu with HPP |
| `GET /menu/:id/hpp` | GET | HPP breakdown |
| `GET /menu/low-margin` | GET | Menus below margin threshold |
| `POST /menu` | POST | Create menu + recipes |
| `PATCH /menu/:id` | PATCH | Update menu + recipes |
| `DELETE /menu/:id` | DELETE | Delete menu |
| `GET /ingredients` | GET | List ingredients |
| `GET /ingredients/:id` | GET | Single ingredient + recent price logs |
| `GET /ingredients/:id/price-history` | GET | Full price history |
| `GET /ingredients/:id/affected-menus` | GET | Menus using this ingredient |
| `POST /ingredients` | POST | Create ingredient |
| `PATCH /ingredients/:id` | PATCH | Update ingredient (auto price log) |
| `DELETE /ingredients/:id` | DELETE | Delete ingredient |
| `GET /retail` | GET | List retail items |
| `POST /retail` | POST | Create retail item |
| `PATCH /retail/:id` | PATCH | Update retail item |
| `DELETE /retail/:id` | DELETE | Delete retail item |
| `GET /promos` | GET | List promos |
| `GET /promos/active` | GET | Active promos |
| `POST /promos` | POST | Create promo |
| `PATCH /promos/:id` | PATCH | Update promo |
| `DELETE /promos/:id` | DELETE | Delete promo |
| `GET /pos/transactions` | GET | List transactions |
| `GET /pos/transactions/:id` | GET | Transaction detail |
| `GET /dashboard/summary` | GET | Summary cards (revenue, HPP, profit) |
| `GET /dashboard/sales-trend` | GET | Daily sales trend |
| `GET /dashboard/top-profit` | GET | Top menus by profit |
| `GET /dashboard/top-qty` | GET | Top menus by quantity |
| `GET /dashboard/profit-by-category` | GET | Profit by category |

All endpoints accept `?startDate=YYYY-MM-DD&endDate=YYYY-MM-DD` for date filtering.
Paginated endpoints accept `?page=1&limit=20`.

---

## File Structure

```
web/
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── postcss.config.mjs
├── .env.local
├── components.json                    # shadcn/ui config
├── public/
│   └── logo.svg
├── src/
│   ├── app/
│   │   ├── layout.tsx                 # Root layout (providers)
│   │   ├── globals.css                # Tailwind + shadcn base
│   │   ├── login/
│   │   │   └── page.tsx               # Login page
│   │   └── (dashboard)/
│   │       ├── layout.tsx             # Dashboard layout (sidebar + header)
│   │       ├── page.tsx               # Dashboard home (redirect or summary)
│   │       ├── dashboard/
│   │       │   └── page.tsx           # Owner dashboard (charts + cards)
│   │       ├── menu/
│   │       │   └── page.tsx           # Menu management
│   │       ├── ingredients/
│   │       │   └── page.tsx           # Ingredient management
│   │       ├── retail/
│   │       │   └── page.tsx           # Retail management
│   │       ├── promos/
│   │       │   └── page.tsx           # Promo management
│   │       ├── users/
│   │       │   └── page.tsx           # User management
│   │       └── transactions/
│   │           └── page.tsx           # Transaction history
│   ├── components/
│   │   ├── ui/                        # shadcn/ui components (auto-generated)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── input.tsx
│   │   │   ├── label.tsx
│   │   │   ├── select.tsx
│   │   │   ├── table.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── separator.tsx
│   │   │   ├── sheet.tsx
│   │   │   ├── tabs.tsx
│   │   │   ├── popover.tsx
│   │   │   ├── calendar.tsx
│   │   │   ├── command.tsx
│   │   │   ├── toast.tsx
│   │   │   ├── toaster.tsx
│   │   │   ├── use-toast.ts
│   │   │   ├── textarea.tsx
│   │   │   ├── switch.tsx
│   │   │   ├── skeleton.tsx
│   │   │   └── scroll-area.tsx
│   │   ├── layout/
│   │   │   ├── sidebar.tsx            # RBAC sidebar navigation
│   │   │   ├── header.tsx             # Top header with user info
│   │   │   └── sidebar-items.ts       # Menu config with permissions
│   │   ├── dashboard/
│   │   │   ├── summary-cards.tsx
│   │   │   ├── sales-trend-chart.tsx
│   │   │   ├── profit-category-chart.tsx
│   │   │   ├── top-profit-chart.tsx
│   │   │   ├── top-qty-chart.tsx
│   │   │   ├── scatter-chart.tsx
│   │   │   ├── promo-analytics.tsx
│   │   │   ├── menu-detail-table.tsx
│   │   │   └── dashboard-filters.tsx
│   │   ├── menu/
│   │   │   ├── menu-table.tsx
│   │   │   ├── menu-form.tsx
│   │   │   ├── recipe-editor.tsx
│   │   │   └── hpp-breakdown.tsx
│   │   ├── ingredients/
│   │   │   ├── ingredient-table.tsx
│   │   │   ├── ingredient-form.tsx
│   │   │   ├── price-history.tsx
│   │   │   └── affected-menus.tsx
│   │   ├── retail/
│   │   │   ├── retail-table.tsx
│   │   │   └── retail-form.tsx
│   │   ├── promos/
│   │   │   ├── promo-table.tsx
│   │   │   └── promo-form.tsx
│   │   ├── users/
│   │   │   ├── user-table.tsx
│   │   │   └── user-form.tsx
│   │   └── transactions/
│   │       ├── transaction-table.tsx
│   │       └── transaction-detail.tsx
│   ├── lib/
│   │   ├── api.ts                     # Axios instance with interceptors
│   │   ├── auth.ts                    # Cookie helpers, token management
│   │   ├── utils.ts                   # cn() helper + formatters
│   │   └── constants.ts              # RBAC config, menu permissions
│   ├── hooks/
│   │   ├── use-auth.ts               # Auth context/hook
│   │   └── use-debounce.ts           # Search debounce
│   ├── types/
│   │   ├── auth.ts
│   │   ├── user.ts
│   │   ├── menu.ts
│   │   ├── ingredient.ts
│   │   ├── retail.ts
│   │   ├── promo.ts
│   │   ├── transaction.ts
│   │   ├── dashboard.ts
│   │   └── api.ts                    # Generic API response types
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── menu.service.ts
│   │   ├── ingredient.service.ts
│   │   ├── retail.service.ts
│   │   ├── promo.service.ts
│   │   ├── transaction.service.ts
│   │   └── dashboard.service.ts
│   ├── providers/
│   │   └── auth-provider.tsx
│   └── middleware.ts                  # Next.js middleware for auth + RBAC
├── __tests__/
│   ├── components/
│   │   ├── summary-cards.test.tsx
│   │   ├── sidebar.test.tsx
│   │   ├── menu-table.test.tsx
│   │   └── login.test.tsx
│   └── hooks/
│       └── use-auth.test.tsx
├── jest.config.ts
└── jest.setup.ts
```

---

### Task 1: Project Setup & Dependencies

**Files:**
- Create: `web/package.json` (via create-next-app)
- Create: `web/.env.local`
- Create: `web/components.json`
- Modify: `web/tailwind.config.ts`

- [ ] **Step 1: Scaffold Next.js project**

```bash
cd ~/Downloads/PROJECT
npx create-next-app@latest web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm --skip-install
```

- [ ] **Step 2: Install dependencies**

```bash
cd ~/Downloads/PROJECT/web
npm install
npm install axios js-cookie recharts date-fns lucide-react clsx tailwind-merge class-variance-authority
npm install -D @types/js-cookie @testing-library/react @testing-library/jest-dom @testing-library/user-event jest jest-environment-jsdom @types/jest ts-jest
```

- [ ] **Step 3: Initialize shadcn/ui**

```bash
cd ~/Downloads/PROJECT/web
npx shadcn@latest init -d
```

- [ ] **Step 4: Install shadcn/ui components**

```bash
cd ~/Downloads/PROJECT/web
npx shadcn@latest add button card dialog dropdown-menu input label select table badge separator sheet tabs popover calendar command toast textarea switch skeleton scroll-area
```

- [ ] **Step 5: Create .env.local**

Create `web/.env.local`:
```env
NEXT_PUBLIC_API_URL=http://localhost:3000
```

- [ ] **Step 6: Create jest.config.ts**

Create `web/jest.config.ts`:
```typescript
import type { Config } from 'jest';
import nextJest from 'next/jest';

const createJestConfig = nextJest({ dir: './' });

const config: Config = {
  setupFilesAfterSetup: ['<rootDir>/jest.setup.ts'],
  testEnvironment: 'jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};

export default createJestConfig(config);
```

- [ ] **Step 7: Create jest.setup.ts**

Create `web/jest.setup.ts`:
```typescript
import '@testing-library/jest-dom';
```

- [ ] **Step 8: Verify the app starts**

```bash
cd ~/Downloads/PROJECT/web
npm run dev
```

Expected: App running on http://localhost:3001

---

### Task 2: Core Layer — Types & Interfaces

**Files:**
- Create: `web/src/types/api.ts`
- Create: `web/src/types/auth.ts`
- Create: `web/src/types/user.ts`
- Create: `web/src/types/menu.ts`
- Create: `web/src/types/ingredient.ts`
- Create: `web/src/types/retail.ts`
- Create: `web/src/types/promo.ts`
- Create: `web/src/types/transaction.ts`
- Create: `web/src/types/dashboard.ts`

- [ ] **Step 1: Create generic API response types**

Create `web/src/types/api.ts`:
```typescript
export interface ApiResponse<T> {
  statusCode: number;
  data: T;
  message: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
}

export interface PaginationParams {
  page?: number;
  limit?: number;
}

export interface DateRangeParams {
  startDate?: string;
  endDate?: string;
}
```

- [ ] **Step 2: Create auth types**

Create `web/src/types/auth.ts`:
```typescript
export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  accessToken: string;
  user: AuthUser;
}

export interface AuthUser {
  id: string;
  name: string;
  email: string;
  role: string;
}

export type RoleName = 'OWNER' | 'KASIR' | 'KITCHEN' | 'INVENTORY' | 'FINANCE' | 'MARKETING';
```

- [ ] **Step 3: Create user types**

Create `web/src/types/user.ts`:
```typescript
export interface User {
  id: string;
  name: string;
  email: string;
  isActive: boolean;
  createdAt: string;
  role: Role;
}

export interface Role {
  id: string;
  name: string;
  description?: string;
  permissions?: RolePermission[];
}

export interface RolePermission {
  permission: Permission;
}

export interface Permission {
  id: string;
  name: string;
  module: string;
  description?: string;
}

export interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
  roleId: string;
}

export interface UpdateUserRequest {
  name?: string;
  email?: string;
  password?: string;
  roleId?: string;
  isActive?: boolean;
}
```

- [ ] **Step 4: Create menu types**

Create `web/src/types/menu.ts`:
```typescript
export type MenuCategory = 'FOOD' | 'BEVERAGE';

export interface MenuItem {
  id: string;
  name: string;
  category: MenuCategory;
  sellingPrice: number;
  imageUrl?: string;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
  recipes: Recipe[];
  hpp: number;
  profit: number;
  margin: number;
}

export interface Recipe {
  id: string;
  menuItemId: string;
  ingredientId: string;
  qtyNeeded: number;
  unit: string;
  ingredient: {
    id: string;
    name: string;
    pricePerUnit: number;
    unit: string;
  };
}

export interface RecipeItemInput {
  ingredientId: string;
  qtyNeeded: number;
  unit: string;
}

export interface CreateMenuRequest {
  name: string;
  category: MenuCategory;
  sellingPrice: number;
  imageUrl?: string;
  recipes: RecipeItemInput[];
}

export interface UpdateMenuRequest {
  name?: string;
  category?: MenuCategory;
  sellingPrice?: number;
  imageUrl?: string;
  isActive?: boolean;
  recipes?: RecipeItemInput[];
}

export interface HppBreakdown {
  menuItemId: string;
  menuItemName: string;
  sellingPrice: number;
  totalHpp: number;
  profit: number;
  margin: number;
  breakdown: HppBreakdownItem[];
}

export interface HppBreakdownItem {
  ingredientId: string;
  ingredientName: string;
  qtyNeeded: number;
  unit: string;
  pricePerUnit: number;
  cost: number;
}
```

- [ ] **Step 5: Create ingredient types**

Create `web/src/types/ingredient.ts`:
```typescript
export interface Ingredient {
  id: string;
  name: string;
  unit: string;
  pricePerUnit: number;
  stockQty: number;
  minStock: number;
  createdAt: string;
  updatedAt: string;
  priceLogs?: IngredientPriceLog[];
}

export interface IngredientPriceLog {
  id: string;
  ingredientId: string;
  oldPrice: number;
  newPrice: number;
  changedAt: string;
}

export interface AffectedMenu {
  menuItem: {
    id: string;
    name: string;
    sellingPrice: number;
  };
  qtyNeeded: number;
}

export interface CreateIngredientRequest {
  name: string;
  unit: string;
  pricePerUnit: number;
  stockQty?: number;
  minStock?: number;
}

export interface UpdateIngredientRequest {
  name?: string;
  unit?: string;
  pricePerUnit?: number;
  stockQty?: number;
  minStock?: number;
}
```

- [ ] **Step 6: Create retail types**

Create `web/src/types/retail.ts`:
```typescript
export interface RetailItem {
  id: string;
  name: string;
  category: string;
  buyPrice: number;
  sellingPrice: number;
  stockQty: number;
  minStock: number;
  imageUrl?: string;
  createdAt: string;
  updatedAt: string;
}

export interface CreateRetailRequest {
  name: string;
  category: string;
  buyPrice: number;
  sellingPrice: number;
  stockQty?: number;
  minStock?: number;
  imageUrl?: string;
}

export interface UpdateRetailRequest {
  name?: string;
  category?: string;
  buyPrice?: number;
  sellingPrice?: number;
  stockQty?: number;
  minStock?: number;
  imageUrl?: string;
}
```

- [ ] **Step 7: Create promo types**

Create `web/src/types/promo.ts`:
```typescript
export type PromoType = 'PERCENT' | 'FIXED' | 'BUY_X_GET_Y' | 'BUNDLE' | 'MIN_PURCHASE';
export type ItemType = 'MENU' | 'RETAIL';
export type PromoItemRole = 'TARGET' | 'FREE_ITEM' | 'BUNDLE_MEMBER';

export interface Promo {
  id: string;
  name: string;
  type: PromoType;
  value: number;
  minPurchaseAmount?: number;
  startDate: string;
  endDate: string;
  isActive: boolean;
  maxUsage?: number;
  usedCount: number;
  createdAt: string;
  updatedAt: string;
  items: PromoItem[];
}

export interface PromoItem {
  id: string;
  promoId: string;
  itemType: ItemType;
  itemId: string;
  role: PromoItemRole;
}

export interface PromoItemInput {
  itemType: ItemType;
  itemId: string;
  role: PromoItemRole;
}

export interface CreatePromoRequest {
  name: string;
  type: PromoType;
  value: number;
  minPurchaseAmount?: number;
  startDate: string;
  endDate: string;
  isActive?: boolean;
  maxUsage?: number;
  items?: PromoItemInput[];
}

export interface UpdatePromoRequest {
  name?: string;
  type?: PromoType;
  value?: number;
  minPurchaseAmount?: number;
  startDate?: string;
  endDate?: string;
  isActive?: boolean;
  maxUsage?: number;
  items?: PromoItemInput[];
}
```

- [ ] **Step 8: Create transaction types**

Create `web/src/types/transaction.ts`:
```typescript
export type PaymentMethod = 'CASH' | 'QRIS' | 'EDC';

export interface Transaction {
  id: string;
  transactionCode: string;
  cashierId: string;
  paymentMethod: PaymentMethod;
  totalAmount: number;
  totalHpp: number;
  totalProfit: number;
  promoDiscount: number;
  createdAt: string;
  updatedAt: string;
  cashier: { id: string; name: string };
  items: TransactionItem[];
  promos: TransactionPromo[];
  orders?: TransactionOrder[];
}

export interface TransactionItem {
  id: string;
  transactionId: string;
  itemType: 'MENU' | 'RETAIL';
  itemId: string;
  qty: number;
  unitPrice: number;
  unitHpp: number;
  subtotal: number;
  subHpp: number;
}

export interface TransactionPromo {
  id: string;
  transactionId: string;
  promoId: string;
  discountAmount: number;
  promo: {
    id: string;
    name: string;
    type: string;
    value: number;
  };
}

export interface TransactionOrder {
  id: string;
  orderNumber: number;
  status: string;
  items: {
    id: string;
    menuItemId: string;
    qty: number;
    notes?: string;
    status: string;
    menuItem: { id: string; name: string };
  }[];
}

export interface TransactionFilters {
  startDate?: string;
  endDate?: string;
  paymentMethod?: PaymentMethod | '';
  page?: number;
  limit?: number;
}
```

- [ ] **Step 9: Create dashboard types**

Create `web/src/types/dashboard.ts`:
```typescript
export interface DashboardSummary {
  totalRevenue: number;
  totalHpp: number;
  totalProfit: number;
  totalPromoDiscount: number;
  transactionCount: number;
  profitMargin: number;
}

export interface SalesTrendPoint {
  date: string;
  revenue: number;
  hpp: number;
  profit: number;
  count: number;
}

export interface TopMenuByProfit {
  menuItem: {
    id: string;
    name: string;
    category: string;
    sellingPrice: number;
  };
  totalQty: number;
  totalRevenue: number;
  totalHpp: number;
  totalProfit: number;
  margin: number;
}

export interface TopMenuByQty {
  menuItem: {
    id: string;
    name: string;
    category: string;
  };
  totalQty: number;
}

export interface ProfitByCategory {
  category: string;
  revenue: number;
  hpp: number;
  profit: number;
}

export interface ScatterDataPoint {
  name: string;
  qty: number;
  margin: number;
  profit: number;
}

export interface PromoAnalytics {
  totalDiscount: number;
  transactionsWithPromo: number;
  transactionsWithoutPromo: number;
  revenueWithPromo: number;
  revenueWithoutPromo: number;
  mostEffectivePromo: {
    name: string;
    usedCount: number;
    totalDiscount: number;
    totalRevenueGenerated: number;
  } | null;
}

export type PeriodFilter = 'today' | '7days' | '30days' | 'custom';

export interface DashboardFilters {
  period: PeriodFilter;
  startDate?: string;
  endDate?: string;
  category?: string;
  paymentMethod?: string;
}
```

---

### Task 3: Core Layer — API Client & Utilities

**Files:**
- Create: `web/src/lib/api.ts`
- Create: `web/src/lib/auth.ts`
- Create: `web/src/lib/utils.ts`
- Create: `web/src/lib/constants.ts`

- [ ] **Step 1: Create utility functions**

Create `web/src/lib/utils.ts`:
```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(amount);
}

export function formatNumber(num: number): string {
  return new Intl.NumberFormat('id-ID').format(num);
}

export function formatPercent(value: number): string {
  return `${value.toFixed(1)}%`;
}

export function formatDate(dateString: string): string {
  return new Date(dateString).toLocaleDateString('id-ID', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
  });
}

export function formatDateTime(dateString: string): string {
  return new Date(dateString).toLocaleString('id-ID', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  });
}

export function getDateRange(period: string): { startDate: string; endDate: string } {
  const end = new Date();
  const start = new Date();

  switch (period) {
    case 'today':
      start.setHours(0, 0, 0, 0);
      break;
    case '7days':
      start.setDate(start.getDate() - 7);
      break;
    case '30days':
      start.setDate(start.getDate() - 30);
      break;
    default:
      start.setHours(0, 0, 0, 0);
  }

  return {
    startDate: start.toISOString().slice(0, 10),
    endDate: end.toISOString().slice(0, 10),
  };
}
```

- [ ] **Step 2: Create auth cookie helpers**

Create `web/src/lib/auth.ts`:
```typescript
import Cookies from 'js-cookie';

const TOKEN_KEY = 'pos_access_token';
const USER_KEY = 'pos_user';

export function getToken(): string | undefined {
  return Cookies.get(TOKEN_KEY);
}

export function setToken(token: string): void {
  Cookies.set(TOKEN_KEY, token, {
    expires: 1,
    sameSite: 'lax',
    secure: process.env.NODE_ENV === 'production',
  });
}

export function removeToken(): void {
  Cookies.remove(TOKEN_KEY);
}

export function getStoredUser(): { id: string; name: string; email: string; role: string } | null {
  const raw = Cookies.get(USER_KEY);
  if (!raw) return null;
  try {
    return JSON.parse(raw);
  } catch {
    return null;
  }
}

export function setStoredUser(user: { id: string; name: string; email: string; role: string }): void {
  Cookies.set(USER_KEY, JSON.stringify(user), {
    expires: 1,
    sameSite: 'lax',
    secure: process.env.NODE_ENV === 'production',
  });
}

export function removeStoredUser(): void {
  Cookies.remove(USER_KEY);
}

export function clearAuth(): void {
  removeToken();
  removeStoredUser();
}
```

- [ ] **Step 3: Create axios API client**

Create `web/src/lib/api.ts`:
```typescript
import axios from 'axios';
import { getToken, clearAuth } from './auth';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000',
  headers: {
    'Content-Type': 'application/json',
  },
});

api.interceptors.request.use(
  (config) => {
    const token = getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error),
);

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      clearAuth();
      if (typeof window !== 'undefined') {
        window.location.href = '/login';
      }
    }
    return Promise.reject(error);
  },
);

export default api;
```

- [ ] **Step 4: Create RBAC constants**

Create `web/src/lib/constants.ts`:
```typescript
import type { RoleName } from '@/types/auth';

export const ROLE_PERMISSIONS: Record<RoleName, string[]> = {
  OWNER: [
    'dashboard:view',
    'menu:create', 'menu:read', 'menu:update', 'menu:delete',
    'ingredient:manage', 'ingredient:read',
    'transaction:create', 'transaction:read',
    'order:read', 'order:update_status',
    'user:manage',
    'promo:manage', 'promo:apply',
  ],
  KASIR: [
    'menu:read',
    'transaction:create', 'transaction:read',
    'order:read',
    'promo:apply',
  ],
  KITCHEN: [
    'menu:create', 'menu:read', 'menu:update', 'menu:delete',
    'ingredient:manage', 'ingredient:read',
    'order:read', 'order:update_status',
  ],
  INVENTORY: [],
  FINANCE: [],
  MARKETING: [],
};

export interface SidebarMenuItem {
  title: string;
  href: string;
  icon: string;
  requiredPermissions: string[];
}

export const SIDEBAR_MENU: SidebarMenuItem[] = [
  {
    title: 'Dashboard',
    href: '/dashboard',
    icon: 'LayoutDashboard',
    requiredPermissions: ['dashboard:view'],
  },
  {
    title: 'Menu',
    href: '/menu',
    icon: 'UtensilsCrossed',
    requiredPermissions: ['menu:read'],
  },
  {
    title: 'Bahan Baku',
    href: '/ingredients',
    icon: 'Package',
    requiredPermissions: ['ingredient:read'],
  },
  {
    title: 'Retail',
    href: '/retail',
    icon: 'ShoppingBag',
    requiredPermissions: ['dashboard:view'],
  },
  {
    title: 'Promo',
    href: '/promos',
    icon: 'Percent',
    requiredPermissions: ['promo:manage'],
  },
  {
    title: 'User Management',
    href: '/users',
    icon: 'Users',
    requiredPermissions: ['user:manage'],
  },
  {
    title: 'Transaksi',
    href: '/transactions',
    icon: 'Receipt',
    requiredPermissions: ['transaction:read'],
  },
];

export function hasPermission(role: RoleName, requiredPermissions: string[]): boolean {
  const rolePerms = ROLE_PERMISSIONS[role] || [];
  return requiredPermissions.every((perm) => rolePerms.includes(perm));
}

export function getVisibleMenuItems(role: RoleName): SidebarMenuItem[] {
  return SIDEBAR_MENU.filter((item) => hasPermission(role, item.requiredPermissions));
}

export const MARGIN_ALERT_THRESHOLD = 30;

export const PROMO_TYPE_LABELS: Record<string, string> = {
  PERCENT: 'Diskon Persen (%)',
  FIXED: 'Potongan Tetap (Rp)',
  BUY_X_GET_Y: 'Beli X Gratis Y',
  BUNDLE: 'Paket Bundle',
  MIN_PURCHASE: 'Minimal Pembelian',
};

export const PAYMENT_METHOD_LABELS: Record<string, string> = {
  CASH: 'Tunai',
  QRIS: 'QRIS',
  EDC: 'EDC / Kartu',
};

export const CATEGORY_LABELS: Record<string, string> = {
  FOOD: 'Makanan',
  BEVERAGE: 'Minuman',
  RETAIL: 'Retail',
};
```

---

### Task 4: Core Layer — API Services

**Files:**
- Create: `web/src/services/auth.service.ts`
- Create: `web/src/services/user.service.ts`
- Create: `web/src/services/menu.service.ts`
- Create: `web/src/services/ingredient.service.ts`
- Create: `web/src/services/retail.service.ts`
- Create: `web/src/services/promo.service.ts`
- Create: `web/src/services/transaction.service.ts`
- Create: `web/src/services/dashboard.service.ts`

- [ ] **Step 1: Create auth service**

Create `web/src/services/auth.service.ts`:
```typescript
import api from '@/lib/api';
import type { LoginRequest, LoginResponse, AuthUser } from '@/types/auth';
import type { ApiResponse } from '@/types/api';

export const authService = {
  async login(data: LoginRequest): Promise<LoginResponse> {
    const response = await api.post<ApiResponse<LoginResponse>>('/auth/login', data);
    return response.data.data;
  },

  async getProfile(): Promise<AuthUser> {
    const response = await api.get<ApiResponse<AuthUser>>('/auth/me');
    return response.data.data;
  },
};
```

- [ ] **Step 2: Create user service**

Create `web/src/services/user.service.ts`:
```typescript
import api from '@/lib/api';
import type { User, Role, CreateUserRequest, UpdateUserRequest } from '@/types/user';
import type { ApiResponse, PaginatedResponse, PaginationParams } from '@/types/api';

export const userService = {
  async getAll(params?: PaginationParams): Promise<PaginatedResponse<User>> {
    const response = await api.get<ApiResponse<PaginatedResponse<User>>>('/users', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<User> {
    const response = await api.get<ApiResponse<User>>(`/users/${id}`);
    return response.data.data;
  },

  async create(data: CreateUserRequest): Promise<User> {
    const response = await api.post<ApiResponse<User>>('/users', data);
    return response.data.data;
  },

  async update(id: string, data: UpdateUserRequest): Promise<User> {
    const response = await api.patch<ApiResponse<User>>(`/users/${id}`, data);
    return response.data.data;
  },

  async delete(id: string): Promise<void> {
    await api.delete(`/users/${id}`);
  },

  async getRoles(): Promise<Role[]> {
    const response = await api.get<ApiResponse<Role[]>>('/users/roles');
    return response.data.data;
  },
};
```

- [ ] **Step 3: Create menu service**

Create `web/src/services/menu.service.ts`:
```typescript
import api from '@/lib/api';
import type { MenuItem, CreateMenuRequest, UpdateMenuRequest, HppBreakdown } from '@/types/menu';
import type { ApiResponse, PaginatedResponse, PaginationParams } from '@/types/api';

export const menuService = {
  async getAll(params?: PaginationParams): Promise<PaginatedResponse<MenuItem>> {
    const response = await api.get<ApiResponse<PaginatedResponse<MenuItem>>>('/menu', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<MenuItem> {
    const response = await api.get<ApiResponse<MenuItem>>(`/menu/${id}`);
    return response.data.data;
  },

  async getHppBreakdown(id: string): Promise<HppBreakdown> {
    const response = await api.get<ApiResponse<HppBreakdown>>(`/menu/${id}/hpp`);
    return response.data.data;
  },

  async getLowMargin(threshold?: number): Promise<MenuItem[]> {
    const response = await api.get<ApiResponse<MenuItem[]>>('/menu/low-margin', {
      params: threshold ? { threshold } : undefined,
    });
    return response.data.data;
  },

  async create(data: CreateMenuRequest): Promise<MenuItem> {
    const response = await api.post<ApiResponse<MenuItem>>('/menu', data);
    return response.data.data;
  },

  async update(id: string, data: UpdateMenuRequest): Promise<MenuItem> {
    const response = await api.patch<ApiResponse<MenuItem>>(`/menu/${id}`, data);
    return response.data.data;
  },

  async delete(id: string): Promise<void> {
    await api.delete(`/menu/${id}`);
  },
};
```

- [ ] **Step 4: Create ingredient service**

Create `web/src/services/ingredient.service.ts`:
```typescript
import api from '@/lib/api';
import type {
  Ingredient, IngredientPriceLog, AffectedMenu,
  CreateIngredientRequest, UpdateIngredientRequest,
} from '@/types/ingredient';
import type { ApiResponse, PaginatedResponse, PaginationParams } from '@/types/api';

export const ingredientService = {
  async getAll(params?: PaginationParams): Promise<PaginatedResponse<Ingredient>> {
    const response = await api.get<ApiResponse<PaginatedResponse<Ingredient>>>('/ingredients', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<Ingredient> {
    const response = await api.get<ApiResponse<Ingredient>>(`/ingredients/${id}`);
    return response.data.data;
  },

  async getPriceHistory(id: string): Promise<IngredientPriceLog[]> {
    const response = await api.get<ApiResponse<IngredientPriceLog[]>>(`/ingredients/${id}/price-history`);
    return response.data.data;
  },

  async getAffectedMenus(id: string): Promise<AffectedMenu[]> {
    const response = await api.get<ApiResponse<AffectedMenu[]>>(`/ingredients/${id}/affected-menus`);
    return response.data.data;
  },

  async create(data: CreateIngredientRequest): Promise<Ingredient> {
    const response = await api.post<ApiResponse<Ingredient>>('/ingredients', data);
    return response.data.data;
  },

  async update(id: string, data: UpdateIngredientRequest): Promise<Ingredient> {
    const response = await api.patch<ApiResponse<Ingredient>>(`/ingredients/${id}`, data);
    return response.data.data;
  },

  async delete(id: string): Promise<void> {
    await api.delete(`/ingredients/${id}`);
  },
};
```

- [ ] **Step 5: Create retail service**

Create `web/src/services/retail.service.ts`:
```typescript
import api from '@/lib/api';
import type { RetailItem, CreateRetailRequest, UpdateRetailRequest } from '@/types/retail';
import type { ApiResponse, PaginatedResponse, PaginationParams } from '@/types/api';

export const retailService = {
  async getAll(params?: PaginationParams): Promise<PaginatedResponse<RetailItem>> {
    const response = await api.get<ApiResponse<PaginatedResponse<RetailItem>>>('/retail', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<RetailItem> {
    const response = await api.get<ApiResponse<RetailItem>>(`/retail/${id}`);
    return response.data.data;
  },

  async create(data: CreateRetailRequest): Promise<RetailItem> {
    const response = await api.post<ApiResponse<RetailItem>>('/retail', data);
    return response.data.data;
  },

  async update(id: string, data: UpdateRetailRequest): Promise<RetailItem> {
    const response = await api.patch<ApiResponse<RetailItem>>(`/retail/${id}`, data);
    return response.data.data;
  },

  async delete(id: string): Promise<void> {
    await api.delete(`/retail/${id}`);
  },
};
```

- [ ] **Step 6: Create promo service**

Create `web/src/services/promo.service.ts`:
```typescript
import api from '@/lib/api';
import type { Promo, CreatePromoRequest, UpdatePromoRequest } from '@/types/promo';
import type { ApiResponse, PaginatedResponse, PaginationParams } from '@/types/api';

export const promoService = {
  async getAll(params?: PaginationParams): Promise<PaginatedResponse<Promo>> {
    const response = await api.get<ApiResponse<PaginatedResponse<Promo>>>('/promos', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<Promo> {
    const response = await api.get<ApiResponse<Promo>>(`/promos/${id}`);
    return response.data.data;
  },

  async getActive(): Promise<Promo[]> {
    const response = await api.get<ApiResponse<Promo[]>>('/promos/active');
    return response.data.data;
  },

  async create(data: CreatePromoRequest): Promise<Promo> {
    const response = await api.post<ApiResponse<Promo>>('/promos', data);
    return response.data.data;
  },

  async update(id: string, data: UpdatePromoRequest): Promise<Promo> {
    const response = await api.patch<ApiResponse<Promo>>(`/promos/${id}`, data);
    return response.data.data;
  },

  async delete(id: string): Promise<void> {
    await api.delete(`/promos/${id}`);
  },
};
```

- [ ] **Step 7: Create transaction service**

Create `web/src/services/transaction.service.ts`:
```typescript
import api from '@/lib/api';
import type { Transaction, TransactionFilters } from '@/types/transaction';
import type { ApiResponse, PaginatedResponse } from '@/types/api';

export const transactionService = {
  async getAll(filters?: TransactionFilters): Promise<PaginatedResponse<Transaction>> {
    const params: Record<string, string | number> = {};
    if (filters?.page) params.page = filters.page;
    if (filters?.limit) params.limit = filters.limit;
    if (filters?.startDate) params.startDate = filters.startDate;
    if (filters?.endDate) params.endDate = filters.endDate;
    if (filters?.paymentMethod) params.paymentMethod = filters.paymentMethod;

    const response = await api.get<ApiResponse<PaginatedResponse<Transaction>>>('/pos/transactions', { params });
    return response.data.data;
  },

  async getById(id: string): Promise<Transaction> {
    const response = await api.get<ApiResponse<Transaction>>(`/pos/transactions/${id}`);
    return response.data.data;
  },
};
```

- [ ] **Step 8: Create dashboard service**

Create `web/src/services/dashboard.service.ts`:
```typescript
import api from '@/lib/api';
import type {
  DashboardSummary, SalesTrendPoint, TopMenuByProfit,
  TopMenuByQty, ProfitByCategory, DashboardFilters, PromoAnalytics,
} from '@/types/dashboard';
import type { ApiResponse } from '@/types/api';
import type { MenuItem } from '@/types/menu';
import type { Transaction } from '@/types/transaction';

function buildParams(filters: DashboardFilters): Record<string, string> {
  const params: Record<string, string> = {};
  if (filters.startDate) params.startDate = filters.startDate;
  if (filters.endDate) params.endDate = filters.endDate;
  if (filters.category) params.category = filters.category;
  if (filters.paymentMethod) params.paymentMethod = filters.paymentMethod;
  return params;
}

export const dashboardService = {
  async getSummary(filters: DashboardFilters): Promise<DashboardSummary> {
    const response = await api.get<ApiResponse<DashboardSummary>>('/dashboard/summary', {
      params: buildParams(filters),
    });
    return response.data.data;
  },

  async getSalesTrend(filters: DashboardFilters): Promise<SalesTrendPoint[]> {
    const response = await api.get<ApiResponse<SalesTrendPoint[]>>('/dashboard/sales-trend', {
      params: buildParams(filters),
    });
    return response.data.data;
  },

  async getTopMenuByProfit(filters: DashboardFilters, limit = 10): Promise<TopMenuByProfit[]> {
    const response = await api.get<ApiResponse<TopMenuByProfit[]>>('/dashboard/top-profit', {
      params: { ...buildParams(filters), limit: String(limit) },
    });
    return response.data.data;
  },

  async getTopMenuByQty(filters: DashboardFilters, limit = 10): Promise<TopMenuByQty[]> {
    const response = await api.get<ApiResponse<TopMenuByQty[]>>('/dashboard/top-qty', {
      params: { ...buildParams(filters), limit: String(limit) },
    });
    return response.data.data;
  },

  async getProfitByCategory(filters: DashboardFilters): Promise<ProfitByCategory[]> {
    const response = await api.get<ApiResponse<ProfitByCategory[]>>('/dashboard/profit-by-category', {
      params: buildParams(filters),
    });
    return response.data.data;
  },

  async getPromoAnalytics(filters: DashboardFilters): Promise<PromoAnalytics> {
    const params = buildParams(filters);
    const response = await api.get<ApiResponse<Transaction[]>>('/pos/transactions', {
      params: { ...params, limit: '9999' },
    });
    const transactions = response.data.data as unknown as Transaction[];

    const withPromo = Array.isArray(transactions)
      ? transactions.filter((t) => t.promoDiscount > 0)
      : [];
    const withoutPromo = Array.isArray(transactions)
      ? transactions.filter((t) => t.promoDiscount === 0)
      : [];

    const totalDiscount = withPromo.reduce((sum, t) => sum + t.promoDiscount, 0);
    const revenueWithPromo = withPromo.reduce((sum, t) => sum + t.totalAmount, 0);
    const revenueWithoutPromo = withoutPromo.reduce((sum, t) => sum + t.totalAmount, 0);

    const promoCountMap = new Map<string, { name: string; count: number; discount: number; revenue: number }>();
    for (const t of withPromo) {
      for (const tp of t.promos) {
        const existing = promoCountMap.get(tp.promoId) ?? {
          name: tp.promo.name,
          count: 0,
          discount: 0,
          revenue: 0,
        };
        existing.count += 1;
        existing.discount += tp.discountAmount;
        existing.revenue += t.totalAmount;
        promoCountMap.set(tp.promoId, existing);
      }
    }

    let mostEffectivePromo: PromoAnalytics['mostEffectivePromo'] = null;
    let maxCount = 0;
    for (const [, data] of promoCountMap) {
      if (data.count > maxCount) {
        maxCount = data.count;
        mostEffectivePromo = {
          name: data.name,
          usedCount: data.count,
          totalDiscount: data.discount,
          totalRevenueGenerated: data.revenue,
        };
      }
    }

    return {
      totalDiscount,
      transactionsWithPromo: withPromo.length,
      transactionsWithoutPromo: withoutPromo.length,
      revenueWithPromo,
      revenueWithoutPromo,
      mostEffectivePromo,
    };
  },

  async getMenuDetailTable(filters: DashboardFilters): Promise<MenuItem[]> {
    const response = await api.get<ApiResponse<{ data: MenuItem[] }>>('/menu', {
      params: { limit: '100' },
    });
    const result = response.data.data;
    return Array.isArray(result) ? result : result.data;
  },

  async getLowMarginMenus(threshold = 30): Promise<MenuItem[]> {
    const response = await api.get<ApiResponse<MenuItem[]>>('/menu/low-margin', {
      params: { threshold: String(threshold) },
    });
    return response.data.data;
  },

  buildScatterData(profitData: TopMenuByProfit[]): { name: string; qty: number; margin: number; profit: number }[] {
    return profitData.map((item) => ({
      name: item.menuItem?.name ?? 'Unknown',
      qty: item.totalQty,
      margin: item.margin,
      profit: item.totalProfit,
    }));
  },
};
```

---

### Task 5: Auth — Login Page, Context, Middleware

**Files:**
- Create: `web/src/hooks/use-auth.ts`
- Create: `web/src/hooks/use-debounce.ts`
- Create: `web/src/providers/auth-provider.tsx`
- Create: `web/src/app/login/page.tsx`
- Create: `web/src/middleware.ts`
- Modify: `web/src/app/layout.tsx`

- [ ] **Step 1: Create useDebounce hook**

Create `web/src/hooks/use-debounce.ts`:
```typescript
'use client';

import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}
```

- [ ] **Step 2: Create useAuth hook**

Create `web/src/hooks/use-auth.ts`:
```typescript
'use client';

import { createContext, useContext } from 'react';
import type { AuthUser, RoleName } from '@/types/auth';

export interface AuthContextType {
  user: AuthUser | null;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  hasPermission: (permissions: string[]) => boolean;
}

export const AuthContext = createContext<AuthContextType>({
  user: null,
  isLoading: true,
  login: async () => {},
  logout: () => {},
  hasPermission: () => false,
});

export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

- [ ] **Step 3: Create AuthProvider**

Create `web/src/providers/auth-provider.tsx`:
```typescript
'use client';

import { useState, useEffect, useCallback, type ReactNode } from 'react';
import { useRouter } from 'next/navigation';
import { AuthContext } from '@/hooks/use-auth';
import { authService } from '@/services/auth.service';
import { setToken, setStoredUser, clearAuth, getToken, getStoredUser } from '@/lib/auth';
import { hasPermission } from '@/lib/constants';
import type { AuthUser, RoleName } from '@/types/auth';

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<AuthUser | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const router = useRouter();

  useEffect(() => {
    const token = getToken();
    if (token) {
      const storedUser = getStoredUser();
      if (storedUser) {
        setUser(storedUser);
        setIsLoading(false);
      } else {
        authService
          .getProfile()
          .then((profile) => {
            setUser(profile);
            setStoredUser(profile);
          })
          .catch(() => {
            clearAuth();
          })
          .finally(() => setIsLoading(false));
      }
    } else {
      setIsLoading(false);
    }
  }, []);

  const login = useCallback(
    async (email: string, password: string) => {
      const response = await authService.login({ email, password });
      setToken(response.accessToken);
      setStoredUser(response.user);
      setUser(response.user);
      router.push('/dashboard');
    },
    [router],
  );

  const logout = useCallback(() => {
    clearAuth();
    setUser(null);
    router.push('/login');
  }, [router]);

  const checkPermission = useCallback(
    (permissions: string[]) => {
      if (!user) return false;
      return hasPermission(user.role as RoleName, permissions);
    },
    [user],
  );

  return (
    <AuthContext.Provider
      value={{
        user,
        isLoading,
        login,
        logout,
        hasPermission: checkPermission,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}
```

- [ ] **Step 4: Create Login page**

Create `web/src/app/login/page.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { useAuth } from '@/hooks/use-auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';

export default function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const { login } = useAuth();
  const router = useRouter();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError('');
    setIsSubmitting(true);

    try {
      await login(email, password);
    } catch (err: unknown) {
      const message =
        err instanceof Error
          ? err.message
          : 'Email atau password salah';
      setError(message);
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-50 px-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <CardTitle className="text-2xl font-bold">POS System</CardTitle>
          <CardDescription>Masuk ke dashboard</CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            {error && (
              <div className="rounded-md bg-red-50 p-3 text-sm text-red-600">
                {error}
              </div>
            )}
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="owner@pos.com"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
                autoFocus
              />
            </div>
            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                placeholder="Masukkan password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                required
              />
            </div>
            <Button type="submit" className="w-full" disabled={isSubmitting}>
              {isSubmitting ? 'Memproses...' : 'Masuk'}
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

- [ ] **Step 5: Create Next.js middleware for auth route protection**

Create `web/src/middleware.ts`:
```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const publicPaths = ['/login'];

export function middleware(request: NextRequest) {
  const token = request.cookies.get('pos_access_token')?.value;
  const { pathname } = request.nextUrl;

  if (publicPaths.some((path) => pathname.startsWith(path))) {
    if (token) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
    return NextResponse.next();
  }

  if (!token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('redirect', pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|logo.svg).*)'],
};
```

- [ ] **Step 6: Update root layout with AuthProvider**

Replace `web/src/app/layout.tsx`:
```tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';
import { AuthProvider } from '@/providers/auth-provider';
import { Toaster } from '@/components/ui/toaster';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'POS Dashboard',
  description: 'POS System Dashboard & CMS',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="id" suppressHydrationWarning>
      <body className={inter.className}>
        <AuthProvider>
          {children}
          <Toaster />
        </AuthProvider>
      </body>
    </html>
  );
}
```

- [ ] **Step 7: Write test for Login page**

Create `web/__tests__/components/login.test.tsx`:
```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginPage from '@/app/login/page';
import { AuthContext } from '@/hooks/use-auth';
import type { AuthContextType } from '@/hooks/use-auth';

const mockLogin = jest.fn();
const mockPush = jest.fn();

jest.mock('next/navigation', () => ({
  useRouter: () => ({ push: mockPush }),
}));

function renderWithAuth(ui: React.ReactElement, overrides?: Partial<AuthContextType>) {
  const defaults: AuthContextType = {
    user: null,
    isLoading: false,
    login: mockLogin,
    logout: jest.fn(),
    hasPermission: () => false,
  };
  return render(
    <AuthContext.Provider value={{ ...defaults, ...overrides }}>{ui}</AuthContext.Provider>,
  );
}

describe('LoginPage', () => {
  beforeEach(() => jest.clearAllMocks());

  it('renders login form', () => {
    renderWithAuth(<LoginPage />);
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /masuk/i })).toBeInTheDocument();
  });

  it('calls login with email and password on submit', async () => {
    renderWithAuth(<LoginPage />);
    const user = userEvent.setup();

    await user.type(screen.getByLabelText(/email/i), 'owner@pos.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /masuk/i }));

    await waitFor(() => {
      expect(mockLogin).toHaveBeenCalledWith('owner@pos.com', 'password123');
    });
  });

  it('shows error message on failed login', async () => {
    mockLogin.mockRejectedValueOnce(new Error('Invalid credentials'));
    renderWithAuth(<LoginPage />);
    const user = userEvent.setup();

    await user.type(screen.getByLabelText(/email/i), 'bad@test.com');
    await user.type(screen.getByLabelText(/password/i), 'wrong');
    await user.click(screen.getByRole('button', { name: /masuk/i }));

    await waitFor(() => {
      expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
    });
  });
});
```

---

### Task 6: Layout — Sidebar, Header, Dashboard Shell

**Files:**
- Create: `web/src/components/layout/sidebar-items.ts`
- Create: `web/src/components/layout/sidebar.tsx`
- Create: `web/src/components/layout/header.tsx`
- Create: `web/src/app/(dashboard)/layout.tsx`
- Create: `web/src/app/(dashboard)/page.tsx`

- [ ] **Step 1: Create sidebar items config**

Create `web/src/components/layout/sidebar-items.ts`:
```typescript
import {
  LayoutDashboard, UtensilsCrossed, Package, ShoppingBag,
  Percent, Users, Receipt,
} from 'lucide-react';
import type { LucideIcon } from 'lucide-react';

export interface NavItem {
  title: string;
  href: string;
  icon: LucideIcon;
  requiredPermissions: string[];
}

export const navItems: NavItem[] = [
  {
    title: 'Dashboard',
    href: '/dashboard',
    icon: LayoutDashboard,
    requiredPermissions: ['dashboard:view'],
  },
  {
    title: 'Menu',
    href: '/menu',
    icon: UtensilsCrossed,
    requiredPermissions: ['menu:read'],
  },
  {
    title: 'Bahan Baku',
    href: '/ingredients',
    icon: Package,
    requiredPermissions: ['ingredient:read'],
  },
  {
    title: 'Retail',
    href: '/retail',
    icon: ShoppingBag,
    requiredPermissions: ['dashboard:view'],
  },
  {
    title: 'Promo',
    href: '/promos',
    icon: Percent,
    requiredPermissions: ['promo:manage'],
  },
  {
    title: 'Users',
    href: '/users',
    icon: Users,
    requiredPermissions: ['user:manage'],
  },
  {
    title: 'Transaksi',
    href: '/transactions',
    icon: Receipt,
    requiredPermissions: ['transaction:read'],
  },
];
```

- [ ] **Step 2: Create Sidebar component**

Create `web/src/components/layout/sidebar.tsx`:
```tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { cn } from '@/lib/utils';
import { useAuth } from '@/hooks/use-auth';
import { navItems } from './sidebar-items';
import { LogOut } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { ScrollArea } from '@/components/ui/scroll-area';
import { Separator } from '@/components/ui/separator';

export function Sidebar() {
  const pathname = usePathname();
  const { user, logout, hasPermission } = useAuth();

  const visibleItems = navItems.filter((item) =>
    hasPermission(item.requiredPermissions),
  );

  return (
    <div className="flex h-full w-64 flex-col border-r bg-white">
      <div className="flex h-16 items-center gap-2 px-6">
        <div className="flex h-8 w-8 items-center justify-center rounded-lg bg-primary text-white text-sm font-bold">
          P
        </div>
        <span className="text-lg font-semibold">POS System</span>
      </div>
      <Separator />
      <ScrollArea className="flex-1 px-3 py-4">
        <nav className="flex flex-col gap-1">
          {visibleItems.map((item) => {
            const isActive =
              pathname === item.href || pathname.startsWith(item.href + '/');
            const Icon = item.icon;
            return (
              <Link
                key={item.href}
                href={item.href}
                className={cn(
                  'flex items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium transition-colors',
                  isActive
                    ? 'bg-primary/10 text-primary'
                    : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900',
                )}
              >
                <Icon className="h-5 w-5" />
                {item.title}
              </Link>
            );
          })}
        </nav>
      </ScrollArea>
      <Separator />
      <div className="p-3">
        <div className="mb-2 px-3">
          <p className="text-sm font-medium text-gray-900 truncate">{user?.name}</p>
          <p className="text-xs text-gray-500 truncate">{user?.role}</p>
        </div>
        <Button
          variant="ghost"
          className="w-full justify-start gap-3 text-red-600 hover:bg-red-50 hover:text-red-700"
          onClick={logout}
        >
          <LogOut className="h-5 w-5" />
          Keluar
        </Button>
      </div>
    </div>
  );
}
```

- [ ] **Step 3: Create Header component**

Create `web/src/components/layout/header.tsx`:
```tsx
'use client';

import { useAuth } from '@/hooks/use-auth';
import { Menu } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Sheet, SheetContent, SheetTrigger } from '@/components/ui/sheet';
import { Sidebar } from './sidebar';

export function Header() {
  const { user, logout } = useAuth();

  return (
    <header className="flex h-16 items-center gap-4 border-b bg-white px-6">
      <Sheet>
        <SheetTrigger asChild>
          <Button variant="ghost" size="icon" className="lg:hidden">
            <Menu className="h-5 w-5" />
          </Button>
        </SheetTrigger>
        <SheetContent side="left" className="w-64 p-0">
          <Sidebar />
        </SheetContent>
      </Sheet>

      <div className="flex-1" />

      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button variant="ghost" className="gap-2">
            <div className="flex h-8 w-8 items-center justify-center rounded-full bg-primary/10 text-sm font-semibold text-primary">
              {user?.name?.charAt(0)?.toUpperCase() ?? 'U'}
            </div>
            <span className="hidden sm:inline-block text-sm">{user?.name}</span>
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent align="end" className="w-56">
          <DropdownMenuLabel>
            <p className="text-sm font-medium">{user?.name}</p>
            <p className="text-xs text-gray-500">{user?.email}</p>
            <p className="text-xs text-gray-400">{user?.role}</p>
          </DropdownMenuLabel>
          <DropdownMenuSeparator />
          <DropdownMenuItem onClick={logout} className="text-red-600">
            Keluar
          </DropdownMenuItem>
        </DropdownMenuContent>
      </DropdownMenu>
    </header>
  );
}
```

- [ ] **Step 4: Create dashboard layout**

Create `web/src/app/(dashboard)/layout.tsx`:
```tsx
'use client';

import { useAuth } from '@/hooks/use-auth';
import { Sidebar } from '@/components/layout/sidebar';
import { Header } from '@/components/layout/header';
import { Skeleton } from '@/components/ui/skeleton';

export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  const { isLoading } = useAuth();

  if (isLoading) {
    return (
      <div className="flex min-h-screen items-center justify-center">
        <div className="space-y-4 w-64">
          <Skeleton className="h-8 w-full" />
          <Skeleton className="h-4 w-3/4" />
          <Skeleton className="h-4 w-1/2" />
        </div>
      </div>
    );
  }

  return (
    <div className="flex h-screen overflow-hidden">
      <div className="hidden lg:block">
        <Sidebar />
      </div>
      <div className="flex flex-1 flex-col overflow-hidden">
        <Header />
        <main className="flex-1 overflow-y-auto bg-gray-50 p-6">{children}</main>
      </div>
    </div>
  );
}
```

- [ ] **Step 5: Create dashboard root redirect page**

Create `web/src/app/(dashboard)/page.tsx`:
```tsx
import { redirect } from 'next/navigation';

export default function DashboardRootPage() {
  redirect('/dashboard');
}
```

- [ ] **Step 6: Write test for Sidebar RBAC filtering**

Create `web/__tests__/components/sidebar.test.tsx`:
```tsx
import { render, screen } from '@testing-library/react';
import { Sidebar } from '@/components/layout/sidebar';
import { AuthContext } from '@/hooks/use-auth';
import type { AuthContextType } from '@/hooks/use-auth';

jest.mock('next/navigation', () => ({
  usePathname: () => '/dashboard',
}));

function renderSidebar(role: string, permissions: string[]) {
  const auth: AuthContextType = {
    user: { id: '1', name: 'Test', email: 'test@test.com', role },
    isLoading: false,
    login: jest.fn(),
    logout: jest.fn(),
    hasPermission: (required: string[]) => required.every((p) => permissions.includes(p)),
  };
  return render(
    <AuthContext.Provider value={auth}>
      <Sidebar />
    </AuthContext.Provider>,
  );
}

describe('Sidebar', () => {
  it('shows all menu items for OWNER', () => {
    renderSidebar('OWNER', [
      'dashboard:view', 'menu:read', 'ingredient:read',
      'promo:manage', 'user:manage', 'transaction:read',
    ]);
    expect(screen.getByText('Dashboard')).toBeInTheDocument();
    expect(screen.getByText('Menu')).toBeInTheDocument();
    expect(screen.getByText('Bahan Baku')).toBeInTheDocument();
    expect(screen.getByText('Retail')).toBeInTheDocument();
    expect(screen.getByText('Promo')).toBeInTheDocument();
    expect(screen.getByText('Users')).toBeInTheDocument();
    expect(screen.getByText('Transaksi')).toBeInTheDocument();
  });

  it('shows only Menu and Bahan Baku for KITCHEN', () => {
    renderSidebar('KITCHEN', ['menu:read', 'ingredient:read']);
    expect(screen.getByText('Menu')).toBeInTheDocument();
    expect(screen.getByText('Bahan Baku')).toBeInTheDocument();
    expect(screen.queryByText('Dashboard')).not.toBeInTheDocument();
    expect(screen.queryByText('Users')).not.toBeInTheDocument();
    expect(screen.queryByText('Promo')).not.toBeInTheDocument();
  });
});
```

---

### Task 7: Dashboard Page — Summary Cards

**Files:**
- Create: `web/src/components/dashboard/summary-cards.tsx`
- Create: `web/src/components/dashboard/dashboard-filters.tsx`
- Create: `web/src/app/(dashboard)/dashboard/page.tsx` (partial — this task creates the shell + summary)

- [ ] **Step 1: Create DashboardFilters component**

Create `web/src/components/dashboard/dashboard-filters.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Calendar } from '@/components/ui/calendar';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { CalendarIcon } from 'lucide-react';
import { format } from 'date-fns';
import { id as idLocale } from 'date-fns/locale';
import type { DashboardFilters as DashboardFiltersType, PeriodFilter } from '@/types/dashboard';

interface DashboardFiltersProps {
  filters: DashboardFiltersType;
  onFiltersChange: (filters: DashboardFiltersType) => void;
}

export function DashboardFilters({ filters, onFiltersChange }: DashboardFiltersProps) {
  const [startDate, setStartDate] = useState<Date | undefined>(
    filters.startDate ? new Date(filters.startDate) : undefined,
  );
  const [endDate, setEndDate] = useState<Date | undefined>(
    filters.endDate ? new Date(filters.endDate) : undefined,
  );

  function handlePeriodChange(period: PeriodFilter) {
    const now = new Date();
    let start: Date;
    const end = now;

    switch (period) {
      case 'today':
        start = new Date(now.getFullYear(), now.getMonth(), now.getDate());
        break;
      case '7days':
        start = new Date(now);
        start.setDate(start.getDate() - 7);
        break;
      case '30days':
        start = new Date(now);
        start.setDate(start.getDate() - 30);
        break;
      case 'custom':
        onFiltersChange({ ...filters, period: 'custom' });
        return;
      default:
        start = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    }

    setStartDate(start);
    setEndDate(end);
    onFiltersChange({
      ...filters,
      period,
      startDate: start.toISOString().slice(0, 10),
      endDate: end.toISOString().slice(0, 10),
    });
  }

  function handleCustomDateApply() {
    if (startDate && endDate) {
      onFiltersChange({
        ...filters,
        period: 'custom',
        startDate: startDate.toISOString().slice(0, 10),
        endDate: endDate.toISOString().slice(0, 10),
      });
    }
  }

  return (
    <div className="flex flex-wrap items-center gap-3">
      <div className="flex rounded-lg border bg-white">
        {(['today', '7days', '30days', 'custom'] as PeriodFilter[]).map((period) => (
          <Button
            key={period}
            variant={filters.period === period ? 'default' : 'ghost'}
            size="sm"
            className="rounded-none first:rounded-l-lg last:rounded-r-lg"
            onClick={() => handlePeriodChange(period)}
          >
            {period === 'today' && 'Hari Ini'}
            {period === '7days' && '7 Hari'}
            {period === '30days' && '30 Hari'}
            {period === 'custom' && 'Custom'}
          </Button>
        ))}
      </div>

      {filters.period === 'custom' && (
        <div className="flex items-center gap-2">
          <Popover>
            <PopoverTrigger asChild>
              <Button variant="outline" size="sm" className="gap-2">
                <CalendarIcon className="h-4 w-4" />
                {startDate ? format(startDate, 'dd MMM yyyy', { locale: idLocale }) : 'Dari'}
              </Button>
            </PopoverTrigger>
            <PopoverContent className="w-auto p-0" align="start">
              <Calendar mode="single" selected={startDate} onSelect={setStartDate} />
            </PopoverContent>
          </Popover>
          <span className="text-sm text-gray-500">-</span>
          <Popover>
            <PopoverTrigger asChild>
              <Button variant="outline" size="sm" className="gap-2">
                <CalendarIcon className="h-4 w-4" />
                {endDate ? format(endDate, 'dd MMM yyyy', { locale: idLocale }) : 'Sampai'}
              </Button>
            </PopoverTrigger>
            <PopoverContent className="w-auto p-0" align="start">
              <Calendar mode="single" selected={endDate} onSelect={setEndDate} />
            </PopoverContent>
          </Popover>
          <Button size="sm" onClick={handleCustomDateApply}>
            Terapkan
          </Button>
        </div>
      )}

      <Select
        value={filters.category || 'all'}
        onValueChange={(val) => onFiltersChange({ ...filters, category: val === 'all' ? '' : val })}
      >
        <SelectTrigger className="w-36">
          <SelectValue placeholder="Kategori" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="all">Semua Kategori</SelectItem>
          <SelectItem value="FOOD">Makanan</SelectItem>
          <SelectItem value="BEVERAGE">Minuman</SelectItem>
          <SelectItem value="RETAIL">Retail</SelectItem>
        </SelectContent>
      </Select>

      <Select
        value={filters.paymentMethod || 'all'}
        onValueChange={(val) => onFiltersChange({ ...filters, paymentMethod: val === 'all' ? '' : val })}
      >
        <SelectTrigger className="w-36">
          <SelectValue placeholder="Pembayaran" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="all">Semua Metode</SelectItem>
          <SelectItem value="CASH">Tunai</SelectItem>
          <SelectItem value="QRIS">QRIS</SelectItem>
          <SelectItem value="EDC">EDC / Kartu</SelectItem>
        </SelectContent>
      </Select>
    </div>
  );
}
```

- [ ] **Step 2: Create SummaryCards component**

Create `web/src/components/dashboard/summary-cards.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { DollarSign, TrendingUp, TrendingDown, Percent, ShoppingCart } from 'lucide-react';
import { formatCurrency, formatNumber, formatPercent } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import type { DashboardSummary, DashboardFilters } from '@/types/dashboard';

interface SummaryCardsProps {
  filters: DashboardFilters;
}

export function SummaryCards({ filters }: SummaryCardsProps) {
  const [data, setData] = useState<DashboardSummary | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getSummary(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  if (isLoading) {
    return (
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-5">
        {Array.from({ length: 5 }).map((_, i) => (
          <Card key={i}>
            <CardHeader className="flex flex-row items-center justify-between pb-2">
              <Skeleton className="h-4 w-24" />
              <Skeleton className="h-4 w-4" />
            </CardHeader>
            <CardContent>
              <Skeleton className="h-7 w-32" />
            </CardContent>
          </Card>
        ))}
      </div>
    );
  }

  if (!data) return null;

  const cards = [
    {
      title: 'Total Penjualan',
      value: formatCurrency(data.totalRevenue),
      icon: DollarSign,
      color: 'text-blue-600',
      bg: 'bg-blue-100',
    },
    {
      title: 'Total HPP',
      value: formatCurrency(data.totalHpp),
      icon: TrendingDown,
      color: 'text-red-600',
      bg: 'bg-red-100',
    },
    {
      title: 'Gross Profit',
      value: formatCurrency(data.totalProfit),
      icon: TrendingUp,
      color: 'text-green-600',
      bg: 'bg-green-100',
    },
    {
      title: 'Profit Margin',
      value: formatPercent(data.profitMargin),
      icon: Percent,
      color: 'text-purple-600',
      bg: 'bg-purple-100',
    },
    {
      title: 'Jumlah Transaksi',
      value: formatNumber(data.transactionCount),
      icon: ShoppingCart,
      color: 'text-orange-600',
      bg: 'bg-orange-100',
    },
  ];

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-5">
      {cards.map((card) => {
        const Icon = card.icon;
        return (
          <Card key={card.title}>
            <CardHeader className="flex flex-row items-center justify-between pb-2">
              <CardTitle className="text-sm font-medium text-gray-500">{card.title}</CardTitle>
              <div className={`rounded-lg p-2 ${card.bg}`}>
                <Icon className={`h-4 w-4 ${card.color}`} />
              </div>
            </CardHeader>
            <CardContent>
              <p className="text-2xl font-bold">{card.value}</p>
            </CardContent>
          </Card>
        );
      })}
    </div>
  );
}
```

- [ ] **Step 3: Write test for SummaryCards**

Create `web/__tests__/components/summary-cards.test.tsx`:
```tsx
import { render, screen, waitFor } from '@testing-library/react';
import { SummaryCards } from '@/components/dashboard/summary-cards';
import { dashboardService } from '@/services/dashboard.service';

jest.mock('@/services/dashboard.service');

const mockFilters = { period: 'today' as const, startDate: '2026-04-18', endDate: '2026-04-18' };

describe('SummaryCards', () => {
  it('renders all summary cards with data', async () => {
    (dashboardService.getSummary as jest.Mock).mockResolvedValue({
      totalRevenue: 5000000,
      totalHpp: 2000000,
      totalProfit: 2800000,
      totalPromoDiscount: 200000,
      transactionCount: 50,
      profitMargin: 56,
    });

    render(<SummaryCards filters={mockFilters} />);

    await waitFor(() => {
      expect(screen.getByText('Total Penjualan')).toBeInTheDocument();
      expect(screen.getByText('Total HPP')).toBeInTheDocument();
      expect(screen.getByText('Gross Profit')).toBeInTheDocument();
      expect(screen.getByText('Profit Margin')).toBeInTheDocument();
      expect(screen.getByText('Jumlah Transaksi')).toBeInTheDocument();
    });
  });
});
```

---

### Task 8: Dashboard Page — Charts (Sales Trend, Profit Category, Top Menus)

**Files:**
- Create: `web/src/components/dashboard/sales-trend-chart.tsx`
- Create: `web/src/components/dashboard/profit-category-chart.tsx`
- Create: `web/src/components/dashboard/top-profit-chart.tsx`
- Create: `web/src/components/dashboard/top-qty-chart.tsx`
- Create: `web/src/components/dashboard/scatter-chart.tsx`

- [ ] **Step 1: Create SalesTrendChart**

Create `web/src/components/dashboard/sales-trend-chart.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer,
} from 'recharts';
import { formatCurrency } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import type { SalesTrendPoint, DashboardFilters } from '@/types/dashboard';

interface SalesTrendChartProps {
  filters: DashboardFilters;
}

function CustomTooltip({ active, payload, label }: { active?: boolean; payload?: { name: string; value: number; color: string }[]; label?: string }) {
  if (!active || !payload) return null;
  return (
    <div className="rounded-lg border bg-white p-3 shadow-md">
      <p className="text-sm font-medium text-gray-600 mb-1">{label}</p>
      {payload.map((entry) => (
        <p key={entry.name} className="text-sm" style={{ color: entry.color }}>
          {entry.name}: {formatCurrency(entry.value)}
        </p>
      ))}
    </div>
  );
}

export function SalesTrendChart({ filters }: SalesTrendChartProps) {
  const [data, setData] = useState<SalesTrendPoint[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getSalesTrend(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Trend Penjualan vs Profit</CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <Skeleton className="h-80 w-full" />
        ) : data.length === 0 ? (
          <div className="flex h-80 items-center justify-center text-gray-500">
            Belum ada data transaksi
          </div>
        ) : (
          <ResponsiveContainer width="100%" height={320}>
            <LineChart data={data}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
              <XAxis
                dataKey="date"
                tick={{ fontSize: 12 }}
                tickFormatter={(val) =>
                  new Date(val).toLocaleDateString('id-ID', { day: '2-digit', month: 'short' })
                }
              />
              <YAxis
                tick={{ fontSize: 12 }}
                tickFormatter={(val) =>
                  val >= 1000000 ? `${(val / 1000000).toFixed(1)}jt` : `${(val / 1000).toFixed(0)}rb`
                }
              />
              <Tooltip content={<CustomTooltip />} />
              <Legend />
              <Line
                type="monotone"
                dataKey="revenue"
                name="Penjualan"
                stroke="#3b82f6"
                strokeWidth={2}
                dot={{ r: 3 }}
                activeDot={{ r: 5 }}
              />
              <Line
                type="monotone"
                dataKey="profit"
                name="Profit"
                stroke="#22c55e"
                strokeWidth={2}
                dot={{ r: 3 }}
                activeDot={{ r: 5 }}
              />
              <Line
                type="monotone"
                dataKey="hpp"
                name="HPP"
                stroke="#ef4444"
                strokeWidth={1}
                strokeDasharray="5 5"
                dot={false}
              />
            </LineChart>
          </ResponsiveContainer>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 2: Create ProfitCategoryChart (pie chart)**

Create `web/src/components/dashboard/profit-category-chart.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { PieChart, Pie, Cell, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import { formatCurrency } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import { CATEGORY_LABELS } from '@/lib/constants';
import type { ProfitByCategory, DashboardFilters } from '@/types/dashboard';

interface ProfitCategoryChartProps {
  filters: DashboardFilters;
}

const COLORS = ['#3b82f6', '#22c55e', '#f59e0b', '#ef4444', '#8b5cf6'];

function CustomTooltip({ active, payload }: { active?: boolean; payload?: { name: string; value: number }[] }) {
  if (!active || !payload || !payload[0]) return null;
  return (
    <div className="rounded-lg border bg-white p-3 shadow-md">
      <p className="text-sm font-medium">{payload[0].name}</p>
      <p className="text-sm text-gray-600">{formatCurrency(payload[0].value)}</p>
    </div>
  );
}

export function ProfitCategoryChart({ filters }: ProfitCategoryChartProps) {
  const [data, setData] = useState<ProfitByCategory[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getProfitByCategory(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  const chartData = data.map((item) => ({
    name: CATEGORY_LABELS[item.category] || item.category,
    value: item.profit,
  }));

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Profit per Kategori</CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <Skeleton className="h-80 w-full" />
        ) : chartData.length === 0 ? (
          <div className="flex h-80 items-center justify-center text-gray-500">
            Belum ada data
          </div>
        ) : (
          <ResponsiveContainer width="100%" height={320}>
            <PieChart>
              <Pie
                data={chartData}
                cx="50%"
                cy="50%"
                innerRadius={60}
                outerRadius={100}
                paddingAngle={5}
                dataKey="value"
                label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
              >
                {chartData.map((_, index) => (
                  <Cell key={index} fill={COLORS[index % COLORS.length]} />
                ))}
              </Pie>
              <Tooltip content={<CustomTooltip />} />
              <Legend />
            </PieChart>
          </ResponsiveContainer>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 3: Create TopProfitChart (bar chart)**

Create `web/src/components/dashboard/top-profit-chart.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import { formatCurrency } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import type { TopMenuByProfit, DashboardFilters } from '@/types/dashboard';

interface TopProfitChartProps {
  filters: DashboardFilters;
}

function CustomTooltip({ active, payload }: { active?: boolean; payload?: { payload: { name: string; profit: number; margin: number; qty: number } }[] }) {
  if (!active || !payload || !payload[0]) return null;
  const data = payload[0].payload;
  return (
    <div className="rounded-lg border bg-white p-3 shadow-md">
      <p className="text-sm font-semibold">{data.name}</p>
      <p className="text-sm text-green-600">Profit: {formatCurrency(data.profit)}</p>
      <p className="text-sm text-gray-500">Margin: {data.margin.toFixed(1)}%</p>
      <p className="text-sm text-gray-500">Qty: {data.qty}</p>
    </div>
  );
}

export function TopProfitChart({ filters }: TopProfitChartProps) {
  const [data, setData] = useState<TopMenuByProfit[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getTopMenuByProfit(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  const chartData = data.map((item) => ({
    name: item.menuItem?.name ?? 'Unknown',
    profit: item.totalProfit,
    margin: item.margin,
    qty: item.totalQty,
  }));

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Top 10 Menu Paling Untung</CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <Skeleton className="h-80 w-full" />
        ) : chartData.length === 0 ? (
          <div className="flex h-80 items-center justify-center text-gray-500">
            Belum ada data
          </div>
        ) : (
          <ResponsiveContainer width="100%" height={320}>
            <BarChart data={chartData} layout="vertical" margin={{ left: 80 }}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
              <XAxis
                type="number"
                tick={{ fontSize: 12 }}
                tickFormatter={(val) => `${(val / 1000).toFixed(0)}rb`}
              />
              <YAxis
                type="category"
                dataKey="name"
                tick={{ fontSize: 11 }}
                width={80}
              />
              <Tooltip content={<CustomTooltip />} />
              <Bar dataKey="profit" fill="#22c55e" radius={[0, 4, 4, 0]} />
            </BarChart>
          </ResponsiveContainer>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 4: Create TopQtyChart (bar chart)**

Create `web/src/components/dashboard/top-qty-chart.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import { dashboardService } from '@/services/dashboard.service';
import type { TopMenuByQty, DashboardFilters } from '@/types/dashboard';

interface TopQtyChartProps {
  filters: DashboardFilters;
}

function CustomTooltip({ active, payload }: { active?: boolean; payload?: { payload: { name: string; qty: number } }[] }) {
  if (!active || !payload || !payload[0]) return null;
  const data = payload[0].payload;
  return (
    <div className="rounded-lg border bg-white p-3 shadow-md">
      <p className="text-sm font-semibold">{data.name}</p>
      <p className="text-sm text-blue-600">Terjual: {data.qty} porsi</p>
    </div>
  );
}

export function TopQtyChart({ filters }: TopQtyChartProps) {
  const [data, setData] = useState<TopMenuByQty[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getTopMenuByQty(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  const chartData = data.map((item) => ({
    name: item.menuItem?.name ?? 'Unknown',
    qty: item.totalQty,
  }));

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Top 10 Menu Paling Laris</CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <Skeleton className="h-80 w-full" />
        ) : chartData.length === 0 ? (
          <div className="flex h-80 items-center justify-center text-gray-500">
            Belum ada data
          </div>
        ) : (
          <ResponsiveContainer width="100%" height={320}>
            <BarChart data={chartData} layout="vertical" margin={{ left: 80 }}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
              <XAxis type="number" tick={{ fontSize: 12 }} />
              <YAxis
                type="category"
                dataKey="name"
                tick={{ fontSize: 11 }}
                width={80}
              />
              <Tooltip content={<CustomTooltip />} />
              <Bar dataKey="qty" fill="#3b82f6" radius={[0, 4, 4, 0]} />
            </BarChart>
          </ResponsiveContainer>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 5: Create ScatterChart (laris vs untung)**

Create `web/src/components/dashboard/scatter-chart.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import {
  ScatterChart as RechartsScatter, Scatter, XAxis, YAxis, CartesianGrid,
  Tooltip, ResponsiveContainer, ZAxis,
} from 'recharts';
import { formatCurrency } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import type { DashboardFilters, ScatterDataPoint } from '@/types/dashboard';

interface ScatterChartProps {
  filters: DashboardFilters;
}

function CustomTooltip({ active, payload }: { active?: boolean; payload?: { payload: ScatterDataPoint }[] }) {
  if (!active || !payload || !payload[0]) return null;
  const data = payload[0].payload;
  return (
    <div className="rounded-lg border bg-white p-3 shadow-md">
      <p className="text-sm font-semibold">{data.name}</p>
      <p className="text-sm text-blue-600">Terjual: {data.qty} porsi</p>
      <p className="text-sm text-green-600">Margin: {data.margin.toFixed(1)}%</p>
      <p className="text-sm text-gray-500">Profit: {formatCurrency(data.profit)}</p>
    </div>
  );
}

export function MenuScatterChart({ filters }: ScatterChartProps) {
  const [data, setData] = useState<ScatterDataPoint[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getTopMenuByProfit(filters, 50)
      .then((profitData) => {
        setData(dashboardService.buildScatterData(profitData));
      })
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Laris vs Untung</CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <Skeleton className="h-80 w-full" />
        ) : data.length === 0 ? (
          <div className="flex h-80 items-center justify-center text-gray-500">
            Belum ada data
          </div>
        ) : (
          <ResponsiveContainer width="100%" height={320}>
            <RechartsScatter margin={{ top: 20, right: 20, bottom: 20, left: 20 }}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
              <XAxis
                type="number"
                dataKey="qty"
                name="Qty Terjual"
                tick={{ fontSize: 12 }}
                label={{ value: 'Qty Terjual', position: 'insideBottom', offset: -10, fontSize: 12 }}
              />
              <YAxis
                type="number"
                dataKey="margin"
                name="Profit Margin"
                unit="%"
                tick={{ fontSize: 12 }}
                label={{ value: 'Margin (%)', angle: -90, position: 'insideLeft', fontSize: 12 }}
              />
              <ZAxis type="number" dataKey="profit" range={[50, 400]} />
              <Tooltip content={<CustomTooltip />} />
              <Scatter data={data} fill="#8b5cf6" />
            </RechartsScatter>
          </ResponsiveContainer>
        )}
      </CardContent>
    </Card>
  );
}
```

---

### Task 9: Dashboard Page — Promo Analytics & Menu Detail Table

**Files:**
- Create: `web/src/components/dashboard/promo-analytics.tsx`
- Create: `web/src/components/dashboard/menu-detail-table.tsx`

- [ ] **Step 1: Create PromoAnalytics component**

Create `web/src/components/dashboard/promo-analytics.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { Badge } from '@/components/ui/badge';
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import { formatCurrency, formatNumber } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import type { PromoAnalytics as PromoAnalyticsType, DashboardFilters } from '@/types/dashboard';
import { Tag, TrendingUp, Receipt } from 'lucide-react';

interface PromoAnalyticsProps {
  filters: DashboardFilters;
}

export function PromoAnalytics({ filters }: PromoAnalyticsProps) {
  const [data, setData] = useState<PromoAnalyticsType | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getPromoAnalytics(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  if (isLoading) {
    return (
      <Card>
        <CardHeader>
          <CardTitle className="text-base">Promo Analytics</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <Skeleton className="h-20 w-full" />
          <Skeleton className="h-40 w-full" />
        </CardContent>
      </Card>
    );
  }

  if (!data) return null;

  const comparisonData = [
    {
      label: 'Dengan Promo',
      transaksi: data.transactionsWithPromo,
      revenue: data.revenueWithPromo,
    },
    {
      label: 'Tanpa Promo',
      transaksi: data.transactionsWithoutPromo,
      revenue: data.revenueWithoutPromo,
    },
  ];

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Promo Analytics</CardTitle>
      </CardHeader>
      <CardContent className="space-y-6">
        <div className="grid grid-cols-3 gap-4">
          <div className="rounded-lg border p-4 text-center">
            <Tag className="mx-auto mb-2 h-5 w-5 text-orange-500" />
            <p className="text-xs text-gray-500">Total Diskon</p>
            <p className="text-lg font-bold text-orange-600">{formatCurrency(data.totalDiscount)}</p>
          </div>
          <div className="rounded-lg border p-4 text-center">
            <Receipt className="mx-auto mb-2 h-5 w-5 text-blue-500" />
            <p className="text-xs text-gray-500">Transaksi + Promo</p>
            <p className="text-lg font-bold text-blue-600">{formatNumber(data.transactionsWithPromo)}</p>
          </div>
          <div className="rounded-lg border p-4 text-center">
            <TrendingUp className="mx-auto mb-2 h-5 w-5 text-green-500" />
            <p className="text-xs text-gray-500">Promo Terbaik</p>
            <p className="text-sm font-bold text-green-600 truncate">
              {data.mostEffectivePromo?.name ?? '-'}
            </p>
            {data.mostEffectivePromo && (
              <p className="text-xs text-gray-400">
                {data.mostEffectivePromo.usedCount}x digunakan
              </p>
            )}
          </div>
        </div>

        <div>
          <p className="mb-3 text-sm font-medium text-gray-700">
            Perbandingan Dengan vs Tanpa Promo
          </p>
          <ResponsiveContainer width="100%" height={200}>
            <BarChart data={comparisonData}>
              <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
              <XAxis dataKey="label" tick={{ fontSize: 12 }} />
              <YAxis
                tick={{ fontSize: 12 }}
                tickFormatter={(val) =>
                  val >= 1000000 ? `${(val / 1000000).toFixed(1)}jt` : `${(val / 1000).toFixed(0)}rb`
                }
              />
              <Tooltip
                formatter={(value: number, name: string) => [
                  name === 'revenue' ? formatCurrency(value) : formatNumber(value),
                  name === 'revenue' ? 'Revenue' : 'Transaksi',
                ]}
              />
              <Legend />
              <Bar dataKey="revenue" name="Revenue" fill="#3b82f6" radius={[4, 4, 0, 0]} />
              <Bar dataKey="transaksi" name="Transaksi" fill="#22c55e" radius={[4, 4, 0, 0]} />
            </BarChart>
          </ResponsiveContainer>
        </div>

        {data.mostEffectivePromo && (
          <div className="rounded-lg border bg-green-50 p-4">
            <p className="text-sm font-medium text-green-800">Promo Paling Efektif</p>
            <p className="text-lg font-bold text-green-700">{data.mostEffectivePromo.name}</p>
            <div className="mt-2 flex gap-4 text-sm text-green-600">
              <span>Digunakan: {data.mostEffectivePromo.usedCount}x</span>
              <span>Total Diskon: {formatCurrency(data.mostEffectivePromo.totalDiscount)}</span>
              <span>Revenue: {formatCurrency(data.mostEffectivePromo.totalRevenueGenerated)}</span>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 2: Create MenuDetailTable component**

Create `web/src/components/dashboard/menu-detail-table.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { Badge } from '@/components/ui/badge';
import { Input } from '@/components/ui/input';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import { formatCurrency, formatPercent } from '@/lib/utils';
import { dashboardService } from '@/services/dashboard.service';
import { MARGIN_ALERT_THRESHOLD, CATEGORY_LABELS } from '@/lib/constants';
import { useDebounce } from '@/hooks/use-debounce';
import { AlertTriangle, Search, ArrowUpDown } from 'lucide-react';
import type { MenuItem } from '@/types/menu';
import type { DashboardFilters } from '@/types/dashboard';

interface MenuDetailTableProps {
  filters: DashboardFilters;
}

type SortField = 'name' | 'hpp' | 'margin' | 'sellingPrice' | 'profit';
type SortDir = 'asc' | 'desc';

export function MenuDetailTable({ filters }: MenuDetailTableProps) {
  const [data, setData] = useState<MenuItem[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [sortField, setSortField] = useState<SortField>('margin');
  const [sortDir, setSortDir] = useState<SortDir>('asc');
  const debouncedSearch = useDebounce(search, 300);

  useEffect(() => {
    setIsLoading(true);
    dashboardService
      .getMenuDetailTable(filters)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [filters]);

  function handleSort(field: SortField) {
    if (sortField === field) {
      setSortDir(sortDir === 'asc' ? 'desc' : 'asc');
    } else {
      setSortField(field);
      setSortDir('asc');
    }
  }

  const filtered = data
    .filter((item) =>
      item.name.toLowerCase().includes(debouncedSearch.toLowerCase()),
    )
    .sort((a, b) => {
      const aVal = a[sortField] ?? 0;
      const bVal = b[sortField] ?? 0;
      if (typeof aVal === 'string' && typeof bVal === 'string') {
        return sortDir === 'asc' ? aVal.localeCompare(bVal) : bVal.localeCompare(aVal);
      }
      return sortDir === 'asc'
        ? (aVal as number) - (bVal as number)
        : (bVal as number) - (aVal as number);
    });

  const SortHeader = ({ field, children }: { field: SortField; children: React.ReactNode }) => (
    <TableHead
      className="cursor-pointer select-none hover:bg-gray-100"
      onClick={() => handleSort(field)}
    >
      <div className="flex items-center gap-1">
        {children}
        <ArrowUpDown className="h-3 w-3" />
      </div>
    </TableHead>
  );

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle className="text-base">Detail Menu + HPP + Margin</CardTitle>
          <div className="relative w-64">
            <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
            <Input
              placeholder="Cari menu..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              className="pl-9"
            />
          </div>
        </div>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <div className="space-y-2">
            {Array.from({ length: 5 }).map((_, i) => (
              <Skeleton key={i} className="h-10 w-full" />
            ))}
          </div>
        ) : (
          <div className="rounded-md border">
            <Table>
              <TableHeader>
                <TableRow>
                  <SortHeader field="name">Nama Menu</SortHeader>
                  <TableHead>Kategori</TableHead>
                  <SortHeader field="sellingPrice">Harga Jual</SortHeader>
                  <SortHeader field="hpp">HPP</SortHeader>
                  <SortHeader field="profit">Profit</SortHeader>
                  <SortHeader field="margin">Margin</SortHeader>
                  <TableHead>Status</TableHead>
                </TableRow>
              </TableHeader>
              <TableBody>
                {filtered.length === 0 ? (
                  <TableRow>
                    <TableCell colSpan={7} className="text-center text-gray-500">
                      Tidak ada data
                    </TableCell>
                  </TableRow>
                ) : (
                  filtered.map((item) => (
                    <TableRow
                      key={item.id}
                      className={item.margin < MARGIN_ALERT_THRESHOLD ? 'bg-red-50' : ''}
                    >
                      <TableCell className="font-medium">{item.name}</TableCell>
                      <TableCell>
                        <Badge variant="secondary">
                          {CATEGORY_LABELS[item.category] || item.category}
                        </Badge>
                      </TableCell>
                      <TableCell>{formatCurrency(item.sellingPrice)}</TableCell>
                      <TableCell>{formatCurrency(item.hpp)}</TableCell>
                      <TableCell className="text-green-600 font-medium">
                        {formatCurrency(item.profit)}
                      </TableCell>
                      <TableCell>
                        <span
                          className={
                            item.margin < MARGIN_ALERT_THRESHOLD
                              ? 'font-bold text-red-600'
                              : 'text-green-600'
                          }
                        >
                          {formatPercent(item.margin)}
                        </span>
                      </TableCell>
                      <TableCell>
                        {item.margin < MARGIN_ALERT_THRESHOLD && (
                          <div className="flex items-center gap-1 text-red-600">
                            <AlertTriangle className="h-4 w-4" />
                            <span className="text-xs">Low Margin</span>
                          </div>
                        )}
                      </TableCell>
                    </TableRow>
                  ))
                )}
              </TableBody>
            </Table>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 3: Assemble Dashboard page**

Create `web/src/app/(dashboard)/dashboard/page.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { DashboardFilters } from '@/components/dashboard/dashboard-filters';
import { SummaryCards } from '@/components/dashboard/summary-cards';
import { SalesTrendChart } from '@/components/dashboard/sales-trend-chart';
import { ProfitCategoryChart } from '@/components/dashboard/profit-category-chart';
import { TopProfitChart } from '@/components/dashboard/top-profit-chart';
import { TopQtyChart } from '@/components/dashboard/top-qty-chart';
import { MenuScatterChart } from '@/components/dashboard/scatter-chart';
import { PromoAnalytics } from '@/components/dashboard/promo-analytics';
import { MenuDetailTable } from '@/components/dashboard/menu-detail-table';
import { getDateRange } from '@/lib/utils';
import type { DashboardFilters as DashboardFiltersType } from '@/types/dashboard';

export default function DashboardPage() {
  const todayRange = getDateRange('today');
  const [filters, setFilters] = useState<DashboardFiltersType>({
    period: 'today',
    startDate: todayRange.startDate,
    endDate: todayRange.endDate,
    category: '',
    paymentMethod: '',
  });

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Dashboard</h1>
        <p className="text-sm text-gray-500">Ringkasan performa bisnis</p>
      </div>

      <DashboardFilters filters={filters} onFiltersChange={setFilters} />

      <SummaryCards filters={filters} />

      <div className="grid gap-6 lg:grid-cols-2">
        <SalesTrendChart filters={filters} />
        <ProfitCategoryChart filters={filters} />
      </div>

      <div className="grid gap-6 lg:grid-cols-2">
        <TopProfitChart filters={filters} />
        <TopQtyChart filters={filters} />
      </div>

      <MenuScatterChart filters={filters} />

      <PromoAnalytics filters={filters} />

      <MenuDetailTable filters={filters} />
    </div>
  );
}
```

---

### Task 10: Menu Management Page

**Files:**
- Create: `web/src/components/menu/menu-table.tsx`
- Create: `web/src/components/menu/menu-form.tsx`
- Create: `web/src/components/menu/recipe-editor.tsx`
- Create: `web/src/components/menu/hpp-breakdown.tsx`
- Create: `web/src/app/(dashboard)/menu/page.tsx`

- [ ] **Step 1: Create RecipeEditor component**

Create `web/src/components/menu/recipe-editor.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Plus, Trash2 } from 'lucide-react';
import { ingredientService } from '@/services/ingredient.service';
import { formatCurrency } from '@/lib/utils';
import type { Ingredient } from '@/types/ingredient';
import type { RecipeItemInput } from '@/types/menu';

interface RecipeEditorProps {
  value: RecipeItemInput[];
  onChange: (recipes: RecipeItemInput[]) => void;
}

export function RecipeEditor({ value, onChange }: RecipeEditorProps) {
  const [ingredients, setIngredients] = useState<Ingredient[]>([]);

  useEffect(() => {
    ingredientService
      .getAll({ limit: 100 })
      .then((res) => setIngredients(res.data))
      .catch(console.error);
  }, []);

  function addItem() {
    onChange([...value, { ingredientId: '', qtyNeeded: 0, unit: '' }]);
  }

  function removeItem(index: number) {
    onChange(value.filter((_, i) => i !== index));
  }

  function updateItem(index: number, field: keyof RecipeItemInput, val: string | number) {
    const updated = [...value];
    updated[index] = { ...updated[index], [field]: val };

    if (field === 'ingredientId') {
      const ingredient = ingredients.find((ing) => ing.id === val);
      if (ingredient) {
        updated[index].unit = ingredient.unit;
      }
    }

    onChange(updated);
  }

  function getIngredient(id: string): Ingredient | undefined {
    return ingredients.find((ing) => ing.id === id);
  }

  function getItemCost(item: RecipeItemInput): number {
    const ingredient = getIngredient(item.ingredientId);
    if (!ingredient) return 0;
    return item.qtyNeeded * ingredient.pricePerUnit;
  }

  const totalHpp = value.reduce((sum, item) => sum + getItemCost(item), 0);

  return (
    <div className="space-y-3">
      <div className="flex items-center justify-between">
        <p className="text-sm font-medium">Resep / Bahan Baku</p>
        <Button type="button" variant="outline" size="sm" onClick={addItem} className="gap-1">
          <Plus className="h-4 w-4" />
          Tambah Bahan
        </Button>
      </div>

      {value.length === 0 && (
        <p className="text-sm text-gray-400 text-center py-4">
          Belum ada bahan. Klik "Tambah Bahan" untuk menambahkan.
        </p>
      )}

      {value.map((item, index) => {
        const ingredient = getIngredient(item.ingredientId);
        const cost = getItemCost(item);
        return (
          <div key={index} className="flex items-end gap-2 rounded-lg border p-3">
            <div className="flex-1 space-y-1">
              <label className="text-xs text-gray-500">Bahan</label>
              <Select
                value={item.ingredientId}
                onValueChange={(val) => updateItem(index, 'ingredientId', val)}
              >
                <SelectTrigger>
                  <SelectValue placeholder="Pilih bahan" />
                </SelectTrigger>
                <SelectContent>
                  {ingredients.map((ing) => (
                    <SelectItem key={ing.id} value={ing.id}>
                      {ing.name} ({formatCurrency(ing.pricePerUnit)}/{ing.unit})
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
            <div className="w-24 space-y-1">
              <label className="text-xs text-gray-500">Qty</label>
              <Input
                type="number"
                step="0.01"
                min="0"
                value={item.qtyNeeded || ''}
                onChange={(e) => updateItem(index, 'qtyNeeded', parseFloat(e.target.value) || 0)}
              />
            </div>
            <div className="w-20 space-y-1">
              <label className="text-xs text-gray-500">Unit</label>
              <Input
                value={item.unit}
                onChange={(e) => updateItem(index, 'unit', e.target.value)}
                readOnly={!!ingredient}
              />
            </div>
            <div className="w-28 space-y-1">
              <label className="text-xs text-gray-500">Cost</label>
              <p className="h-10 flex items-center text-sm font-medium text-green-600">
                {formatCurrency(cost)}
              </p>
            </div>
            <Button
              type="button"
              variant="ghost"
              size="icon"
              onClick={() => removeItem(index)}
              className="text-red-500 hover:text-red-700"
            >
              <Trash2 className="h-4 w-4" />
            </Button>
          </div>
        );
      })}

      {value.length > 0 && (
        <div className="flex justify-end border-t pt-3">
          <p className="text-sm font-semibold">
            Total HPP: <span className="text-green-600">{formatCurrency(totalHpp)}</span>
          </p>
        </div>
      )}
    </div>
  );
}
```

- [ ] **Step 2: Create HppBreakdown component**

Create `web/src/components/menu/hpp-breakdown.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import { formatCurrency, formatPercent } from '@/lib/utils';
import { menuService } from '@/services/menu.service';
import type { HppBreakdown as HppBreakdownType } from '@/types/menu';

interface HppBreakdownProps {
  menuId: string;
}

export function HppBreakdown({ menuId }: HppBreakdownProps) {
  const [data, setData] = useState<HppBreakdownType | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    menuService
      .getHppBreakdown(menuId)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [menuId]);

  if (isLoading) return <Skeleton className="h-40 w-full" />;
  if (!data) return null;

  return (
    <Card>
      <CardHeader className="pb-3">
        <CardTitle className="text-sm">HPP Breakdown — {data.menuItemName}</CardTitle>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Bahan</TableHead>
              <TableHead>Qty</TableHead>
              <TableHead>Harga/Unit</TableHead>
              <TableHead className="text-right">Cost</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {data.breakdown.map((item) => (
              <TableRow key={item.ingredientId}>
                <TableCell className="font-medium">{item.ingredientName}</TableCell>
                <TableCell>
                  {item.qtyNeeded} {item.unit}
                </TableCell>
                <TableCell>{formatCurrency(item.pricePerUnit)}</TableCell>
                <TableCell className="text-right">{formatCurrency(item.cost)}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
        <div className="mt-4 space-y-1 border-t pt-3 text-sm">
          <div className="flex justify-between">
            <span>Harga Jual</span>
            <span className="font-medium">{formatCurrency(data.sellingPrice)}</span>
          </div>
          <div className="flex justify-between">
            <span>Total HPP</span>
            <span className="font-medium text-red-600">{formatCurrency(data.totalHpp)}</span>
          </div>
          <div className="flex justify-between">
            <span>Profit</span>
            <span className="font-bold text-green-600">{formatCurrency(data.profit)}</span>
          </div>
          <div className="flex justify-between">
            <span>Margin</span>
            <span className={`font-bold ${data.margin < 30 ? 'text-red-600' : 'text-green-600'}`}>
              {formatPercent(data.margin)}
            </span>
          </div>
        </div>
      </CardContent>
    </Card>
  );
}
```

- [ ] **Step 3: Create MenuForm component**

Create `web/src/components/menu/menu-form.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Switch } from '@/components/ui/switch';
import {
  Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter,
} from '@/components/ui/dialog';
import { RecipeEditor } from './recipe-editor';
import { menuService } from '@/services/menu.service';
import { useToast } from '@/components/ui/use-toast';
import type { MenuItem, CreateMenuRequest, RecipeItemInput, MenuCategory } from '@/types/menu';

interface MenuFormProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  editItem?: MenuItem | null;
  onSuccess: () => void;
}

export function MenuForm({ open, onOpenChange, editItem, onSuccess }: MenuFormProps) {
  const isEdit = !!editItem;
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [name, setName] = useState(editItem?.name ?? '');
  const [category, setCategory] = useState<MenuCategory>(editItem?.category ?? 'FOOD');
  const [sellingPrice, setSellingPrice] = useState<number>(editItem?.sellingPrice ?? 0);
  const [imageUrl, setImageUrl] = useState(editItem?.imageUrl ?? '');
  const [isActive, setIsActive] = useState(editItem?.isActive ?? true);
  const [recipes, setRecipes] = useState<RecipeItemInput[]>(
    editItem?.recipes?.map((r) => ({
      ingredientId: r.ingredientId,
      qtyNeeded: r.qtyNeeded,
      unit: r.unit,
    })) ?? [],
  );

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      if (isEdit && editItem) {
        await menuService.update(editItem.id, {
          name,
          category,
          sellingPrice,
          imageUrl: imageUrl || undefined,
          isActive,
          recipes,
        });
        toast({ title: 'Menu berhasil diupdate' });
      } else {
        await menuService.create({
          name,
          category,
          sellingPrice,
          imageUrl: imageUrl || undefined,
          recipes,
        });
        toast({ title: 'Menu berhasil ditambahkan' });
      }
      onSuccess();
      onOpenChange(false);
    } catch (err) {
      toast({ title: 'Gagal menyimpan menu', variant: 'destructive' });
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-2xl max-h-[90vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle>{isEdit ? 'Edit Menu' : 'Tambah Menu Baru'}</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label htmlFor="name">Nama Menu</Label>
              <Input
                id="name"
                value={name}
                onChange={(e) => setName(e.target.value)}
                required
              />
            </div>
            <div className="space-y-2">
              <Label>Kategori</Label>
              <Select value={category} onValueChange={(val) => setCategory(val as MenuCategory)}>
                <SelectTrigger>
                  <SelectValue />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="FOOD">Makanan</SelectItem>
                  <SelectItem value="BEVERAGE">Minuman</SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label htmlFor="sellingPrice">Harga Jual (Rp)</Label>
              <Input
                id="sellingPrice"
                type="number"
                min="0"
                value={sellingPrice || ''}
                onChange={(e) => setSellingPrice(parseInt(e.target.value) || 0)}
                required
              />
            </div>
            <div className="space-y-2">
              <Label htmlFor="imageUrl">URL Gambar (opsional)</Label>
              <Input
                id="imageUrl"
                value={imageUrl}
                onChange={(e) => setImageUrl(e.target.value)}
              />
            </div>
          </div>
          {isEdit && (
            <div className="flex items-center gap-2">
              <Switch checked={isActive} onCheckedChange={setIsActive} />
              <Label>Aktif</Label>
            </div>
          )}
          <RecipeEditor value={recipes} onChange={setRecipes} />
          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>
              Batal
            </Button>
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Menyimpan...' : isEdit ? 'Update' : 'Simpan'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

- [ ] **Step 4: Create MenuTable component**

Create `web/src/components/menu/menu-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import {
  Dialog, DialogContent, DialogHeader, DialogTitle,
} from '@/components/ui/dialog';
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Plus, Search, MoreHorizontal, Pencil, Trash2, Eye, AlertTriangle } from 'lucide-react';
import { formatCurrency, formatPercent } from '@/lib/utils';
import { menuService } from '@/services/menu.service';
import { useDebounce } from '@/hooks/use-debounce';
import { useToast } from '@/components/ui/use-toast';
import { CATEGORY_LABELS, MARGIN_ALERT_THRESHOLD } from '@/lib/constants';
import { MenuForm } from './menu-form';
import { HppBreakdown } from './hpp-breakdown';
import type { MenuItem } from '@/types/menu';

export function MenuTable() {
  const [items, setItems] = useState<MenuItem[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [showForm, setShowForm] = useState(false);
  const [editItem, setEditItem] = useState<MenuItem | null>(null);
  const [showHpp, setShowHpp] = useState<string | null>(null);
  const debouncedSearch = useDebounce(search, 300);
  const { toast } = useToast();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const res = await menuService.getAll({ page, limit: 20 });
      setItems(res.data);
      setTotal(res.total);
    } catch (err) {
      console.error(err);
    } finally {
      setIsLoading(false);
    }
  }, [page]);

  useEffect(() => {
    loadData();
  }, [loadData]);

  const filtered = items.filter((item) =>
    item.name.toLowerCase().includes(debouncedSearch.toLowerCase()),
  );

  async function handleDelete(id: string) {
    if (!confirm('Yakin ingin menghapus menu ini?')) return;
    try {
      await menuService.delete(id);
      toast({ title: 'Menu berhasil dihapus' });
      loadData();
    } catch {
      toast({ title: 'Gagal menghapus menu', variant: 'destructive' });
    }
  }

  function handleEdit(item: MenuItem) {
    setEditItem(item);
    setShowForm(true);
  }

  function handleAdd() {
    setEditItem(null);
    setShowForm(true);
  }

  const totalPages = Math.ceil(total / 20);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="relative w-72">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
          <Input
            placeholder="Cari menu..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-9"
          />
        </div>
        <Button onClick={handleAdd} className="gap-2">
          <Plus className="h-4 w-4" />
          Tambah Menu
        </Button>
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nama</TableHead>
              <TableHead>Kategori</TableHead>
              <TableHead>Harga Jual</TableHead>
              <TableHead>HPP</TableHead>
              <TableHead>Profit</TableHead>
              <TableHead>Margin</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow>
                <TableCell colSpan={8} className="text-center py-8 text-gray-500">
                  Memuat data...
                </TableCell>
              </TableRow>
            ) : filtered.length === 0 ? (
              <TableRow>
                <TableCell colSpan={8} className="text-center py-8 text-gray-500">
                  Tidak ada menu ditemukan
                </TableCell>
              </TableRow>
            ) : (
              filtered.map((item) => (
                <TableRow key={item.id}>
                  <TableCell className="font-medium">{item.name}</TableCell>
                  <TableCell>
                    <Badge variant="secondary">
                      {CATEGORY_LABELS[item.category] || item.category}
                    </Badge>
                  </TableCell>
                  <TableCell>{formatCurrency(item.sellingPrice)}</TableCell>
                  <TableCell>{formatCurrency(item.hpp)}</TableCell>
                  <TableCell className="text-green-600">{formatCurrency(item.profit)}</TableCell>
                  <TableCell>
                    <span className={item.margin < MARGIN_ALERT_THRESHOLD ? 'text-red-600 font-bold' : 'text-green-600'}>
                      {formatPercent(item.margin)}
                    </span>
                    {item.margin < MARGIN_ALERT_THRESHOLD && (
                      <AlertTriangle className="ml-1 inline h-3 w-3 text-red-500" />
                    )}
                  </TableCell>
                  <TableCell>
                    <Badge variant={item.isActive ? 'default' : 'secondary'}>
                      {item.isActive ? 'Aktif' : 'Nonaktif'}
                    </Badge>
                  </TableCell>
                  <TableCell>
                    <DropdownMenu>
                      <DropdownMenuTrigger asChild>
                        <Button variant="ghost" size="icon">
                          <MoreHorizontal className="h-4 w-4" />
                        </Button>
                      </DropdownMenuTrigger>
                      <DropdownMenuContent align="end">
                        <DropdownMenuItem onClick={() => setShowHpp(item.id)}>
                          <Eye className="mr-2 h-4 w-4" /> Lihat HPP
                        </DropdownMenuItem>
                        <DropdownMenuItem onClick={() => handleEdit(item)}>
                          <Pencil className="mr-2 h-4 w-4" /> Edit
                        </DropdownMenuItem>
                        <DropdownMenuItem onClick={() => handleDelete(item.id)} className="text-red-600">
                          <Trash2 className="mr-2 h-4 w-4" /> Hapus
                        </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </TableCell>
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button
            variant="outline"
            size="sm"
            disabled={page <= 1}
            onClick={() => setPage(page - 1)}
          >
            Sebelumnya
          </Button>
          <span className="text-sm text-gray-500">
            Halaman {page} dari {totalPages}
          </span>
          <Button
            variant="outline"
            size="sm"
            disabled={page >= totalPages}
            onClick={() => setPage(page + 1)}
          >
            Selanjutnya
          </Button>
        </div>
      )}

      <MenuForm
        open={showForm}
        onOpenChange={setShowForm}
        editItem={editItem}
        onSuccess={loadData}
      />

      <Dialog open={!!showHpp} onOpenChange={() => setShowHpp(null)}>
        <DialogContent className="max-w-lg">
          <DialogHeader>
            <DialogTitle>HPP Breakdown</DialogTitle>
          </DialogHeader>
          {showHpp && <HppBreakdown menuId={showHpp} />}
        </DialogContent>
      </Dialog>
    </div>
  );
}
```

- [ ] **Step 5: Create Menu page**

Create `web/src/app/(dashboard)/menu/page.tsx`:
```tsx
import { MenuTable } from '@/components/menu/menu-table';

export default function MenuPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Menu Management</h1>
        <p className="text-sm text-gray-500">Kelola menu, resep, dan lihat HPP breakdown</p>
      </div>
      <MenuTable />
    </div>
  );
}
```

---

### Task 11: Ingredient Management Page

**Files:**
- Create: `web/src/components/ingredients/ingredient-table.tsx`
- Create: `web/src/components/ingredients/ingredient-form.tsx`
- Create: `web/src/components/ingredients/price-history.tsx`
- Create: `web/src/components/ingredients/affected-menus.tsx`
- Create: `web/src/app/(dashboard)/ingredients/page.tsx`

- [ ] **Step 1: Create PriceHistory component**

Create `web/src/components/ingredients/price-history.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Skeleton } from '@/components/ui/skeleton';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import { formatCurrency, formatDateTime } from '@/lib/utils';
import { ingredientService } from '@/services/ingredient.service';
import { ArrowRight } from 'lucide-react';
import type { IngredientPriceLog } from '@/types/ingredient';

interface PriceHistoryProps {
  ingredientId: string;
}

export function PriceHistory({ ingredientId }: PriceHistoryProps) {
  const [logs, setLogs] = useState<IngredientPriceLog[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    ingredientService
      .getPriceHistory(ingredientId)
      .then(setLogs)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [ingredientId]);

  if (isLoading) return <Skeleton className="h-40 w-full" />;

  if (logs.length === 0) {
    return <p className="text-sm text-gray-500 text-center py-4">Belum ada perubahan harga</p>;
  }

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Tanggal</TableHead>
          <TableHead>Harga Lama</TableHead>
          <TableHead></TableHead>
          <TableHead>Harga Baru</TableHead>
          <TableHead>Perubahan</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {logs.map((log) => {
          const diff = log.newPrice - log.oldPrice;
          const pct = log.oldPrice > 0 ? ((diff / log.oldPrice) * 100).toFixed(1) : '0';
          return (
            <TableRow key={log.id}>
              <TableCell className="text-sm">{formatDateTime(log.changedAt)}</TableCell>
              <TableCell>{formatCurrency(log.oldPrice)}</TableCell>
              <TableCell><ArrowRight className="h-4 w-4 text-gray-400" /></TableCell>
              <TableCell>{formatCurrency(log.newPrice)}</TableCell>
              <TableCell>
                <span className={diff > 0 ? 'text-red-600' : 'text-green-600'}>
                  {diff > 0 ? '+' : ''}{formatCurrency(diff)} ({pct}%)
                </span>
              </TableCell>
            </TableRow>
          );
        })}
      </TableBody>
    </Table>
  );
}
```

- [ ] **Step 2: Create AffectedMenus component**

Create `web/src/components/ingredients/affected-menus.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Skeleton } from '@/components/ui/skeleton';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import { formatCurrency } from '@/lib/utils';
import { ingredientService } from '@/services/ingredient.service';
import type { AffectedMenu } from '@/types/ingredient';

interface AffectedMenusProps {
  ingredientId: string;
}

export function AffectedMenus({ ingredientId }: AffectedMenusProps) {
  const [menus, setMenus] = useState<AffectedMenu[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    ingredientService
      .getAffectedMenus(ingredientId)
      .then(setMenus)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [ingredientId]);

  if (isLoading) return <Skeleton className="h-20 w-full" />;

  if (menus.length === 0) {
    return <p className="text-sm text-gray-500 text-center py-4">Bahan ini belum digunakan di menu manapun</p>;
  }

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Menu</TableHead>
          <TableHead>Harga Jual</TableHead>
          <TableHead>Qty Digunakan</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {menus.map((item, i) => (
          <TableRow key={i}>
            <TableCell className="font-medium">{item.menuItem.name}</TableCell>
            <TableCell>{formatCurrency(item.menuItem.sellingPrice)}</TableCell>
            <TableCell>{item.qtyNeeded}</TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}
```

- [ ] **Step 3: Create IngredientForm component**

Create `web/src/components/ingredients/ingredient-form.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter,
} from '@/components/ui/dialog';
import { ingredientService } from '@/services/ingredient.service';
import { useToast } from '@/components/ui/use-toast';
import type { Ingredient } from '@/types/ingredient';

interface IngredientFormProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  editItem?: Ingredient | null;
  onSuccess: () => void;
}

export function IngredientForm({ open, onOpenChange, editItem, onSuccess }: IngredientFormProps) {
  const isEdit = !!editItem;
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [name, setName] = useState(editItem?.name ?? '');
  const [unit, setUnit] = useState(editItem?.unit ?? '');
  const [pricePerUnit, setPricePerUnit] = useState<number>(editItem?.pricePerUnit ?? 0);
  const [stockQty, setStockQty] = useState<number>(editItem?.stockQty ?? 0);
  const [minStock, setMinStock] = useState<number>(editItem?.minStock ?? 0);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      if (isEdit && editItem) {
        await ingredientService.update(editItem.id, { name, unit, pricePerUnit, stockQty, minStock });
        toast({ title: 'Bahan baku berhasil diupdate' });
      } else {
        await ingredientService.create({ name, unit, pricePerUnit, stockQty, minStock });
        toast({ title: 'Bahan baku berhasil ditambahkan' });
      }
      onSuccess();
      onOpenChange(false);
    } catch {
      toast({ title: 'Gagal menyimpan bahan baku', variant: 'destructive' });
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>{isEdit ? 'Edit Bahan Baku' : 'Tambah Bahan Baku'}</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Label htmlFor="ing-name">Nama Bahan</Label>
            <Input id="ing-name" value={name} onChange={(e) => setName(e.target.value)} required />
          </div>
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label htmlFor="ing-unit">Satuan (kg, liter, butir, pcs)</Label>
              <Input id="ing-unit" value={unit} onChange={(e) => setUnit(e.target.value)} required />
            </div>
            <div className="space-y-2">
              <Label htmlFor="ing-price">Harga per Satuan (Rp)</Label>
              <Input
                id="ing-price"
                type="number"
                min="0"
                value={pricePerUnit || ''}
                onChange={(e) => setPricePerUnit(parseFloat(e.target.value) || 0)}
                required
              />
            </div>
          </div>
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label htmlFor="ing-stock">Stok</Label>
              <Input
                id="ing-stock"
                type="number"
                min="0"
                step="0.01"
                value={stockQty || ''}
                onChange={(e) => setStockQty(parseFloat(e.target.value) || 0)}
              />
            </div>
            <div className="space-y-2">
              <Label htmlFor="ing-minstock">Stok Minimum</Label>
              <Input
                id="ing-minstock"
                type="number"
                min="0"
                step="0.01"
                value={minStock || ''}
                onChange={(e) => setMinStock(parseFloat(e.target.value) || 0)}
              />
            </div>
          </div>
          {isEdit && editItem && pricePerUnit !== editItem.pricePerUnit && (
            <div className="rounded-md bg-yellow-50 border border-yellow-200 p-3 text-sm text-yellow-700">
              Perubahan harga akan otomatis tercatat di price history dan mempengaruhi HPP semua menu yang menggunakan bahan ini.
            </div>
          )}
          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>Batal</Button>
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Menyimpan...' : isEdit ? 'Update' : 'Simpan'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

- [ ] **Step 4: Create IngredientTable component**

Create `web/src/components/ingredients/ingredient-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table';
import {
  Dialog, DialogContent, DialogHeader, DialogTitle,
} from '@/components/ui/dialog';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import {
  DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Plus, Search, MoreHorizontal, Pencil, Trash2, History, LinkIcon, AlertTriangle } from 'lucide-react';
import { formatCurrency, formatNumber } from '@/lib/utils';
import { ingredientService } from '@/services/ingredient.service';
import { useDebounce } from '@/hooks/use-debounce';
import { useToast } from '@/components/ui/use-toast';
import { IngredientForm } from './ingredient-form';
import { PriceHistory } from './price-history';
import { AffectedMenus } from './affected-menus';
import type { Ingredient } from '@/types/ingredient';

export function IngredientTable() {
  const [items, setItems] = useState<Ingredient[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [showForm, setShowForm] = useState(false);
  const [editItem, setEditItem] = useState<Ingredient | null>(null);
  const [detailItem, setDetailItem] = useState<Ingredient | null>(null);
  const debouncedSearch = useDebounce(search, 300);
  const { toast } = useToast();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const res = await ingredientService.getAll({ page, limit: 20 });
      setItems(res.data);
      setTotal(res.total);
    } catch (err) {
      console.error(err);
    } finally {
      setIsLoading(false);
    }
  }, [page]);

  useEffect(() => {
    loadData();
  }, [loadData]);

  const filtered = items.filter((item) =>
    item.name.toLowerCase().includes(debouncedSearch.toLowerCase()),
  );

  async function handleDelete(id: string) {
    if (!confirm('Yakin ingin menghapus bahan ini?')) return;
    try {
      await ingredientService.delete(id);
      toast({ title: 'Bahan baku berhasil dihapus' });
      loadData();
    } catch {
      toast({ title: 'Gagal menghapus bahan baku', variant: 'destructive' });
    }
  }

  const totalPages = Math.ceil(total / 20);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="relative w-72">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
          <Input
            placeholder="Cari bahan baku..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-9"
          />
        </div>
        <Button onClick={() => { setEditItem(null); setShowForm(true); }} className="gap-2">
          <Plus className="h-4 w-4" />
          Tambah Bahan
        </Button>
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nama</TableHead>
              <TableHead>Satuan</TableHead>
              <TableHead>Harga/Unit</TableHead>
              <TableHead>Stok</TableHead>
              <TableHead>Min Stok</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow>
                <TableCell colSpan={7} className="text-center py-8 text-gray-500">Memuat data...</TableCell>
              </TableRow>
            ) : filtered.length === 0 ? (
              <TableRow>
                <TableCell colSpan={7} className="text-center py-8 text-gray-500">Tidak ada data</TableCell>
              </TableRow>
            ) : (
              filtered.map((item) => (
                <TableRow key={item.id}>
                  <TableCell className="font-medium">{item.name}</TableCell>
                  <TableCell>{item.unit}</TableCell>
                  <TableCell>{formatCurrency(item.pricePerUnit)}</TableCell>
                  <TableCell>
                    {formatNumber(item.stockQty)}
                    {item.stockQty <= item.minStock && (
                      <AlertTriangle className="ml-1 inline h-3 w-3 text-orange-500" />
                    )}
                  </TableCell>
                  <TableCell>{formatNumber(item.minStock)}</TableCell>
                  <TableCell>
                    {item.stockQty <= item.minStock ? (
                      <Badge variant="destructive">Low Stock</Badge>
                    ) : (
                      <Badge variant="secondary">OK</Badge>
                    )}
                  </TableCell>
                  <TableCell>
                    <DropdownMenu>
                      <DropdownMenuTrigger asChild>
                        <Button variant="ghost" size="icon"><MoreHorizontal className="h-4 w-4" /></Button>
                      </DropdownMenuTrigger>
                      <DropdownMenuContent align="end">
                        <DropdownMenuItem onClick={() => setDetailItem(item)}>
                          <History className="mr-2 h-4 w-4" /> Detail & Histori
                        </DropdownMenuItem>
                        <DropdownMenuItem onClick={() => { setEditItem(item); setShowForm(true); }}>
                          <Pencil className="mr-2 h-4 w-4" /> Edit
                        </DropdownMenuItem>
                        <DropdownMenuItem onClick={() => handleDelete(item.id)} className="text-red-600">
                          <Trash2 className="mr-2 h-4 w-4" /> Hapus
                        </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </TableCell>
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button variant="outline" size="sm" disabled={page <= 1} onClick={() => setPage(page - 1)}>Sebelumnya</Button>
          <span className="text-sm text-gray-500">Halaman {page} dari {totalPages}</span>
          <Button variant="outline" size="sm" disabled={page >= totalPages} onClick={() => setPage(page + 1)}>Selanjutnya</Button>
        </div>
      )}

      <IngredientForm open={showForm} onOpenChange={setShowForm} editItem={editItem} onSuccess={loadData} />

      <Dialog open={!!detailItem} onOpenChange={() => setDetailItem(null)}>
        <DialogContent className="max-w-2xl max-h-[80vh] overflow-y-auto">
          <DialogHeader>
            <DialogTitle>{detailItem?.name} — Detail</DialogTitle>
          </DialogHeader>
          {detailItem && (
            <Tabs defaultValue="price-history">
              <TabsList className="mb-4">
                <TabsTrigger value="price-history">Histori Harga</TabsTrigger>
                <TabsTrigger value="affected-menus">Menu Terkait</TabsTrigger>
              </TabsList>
              <TabsContent value="price-history">
                <PriceHistory ingredientId={detailItem.id} />
              </TabsContent>
              <TabsContent value="affected-menus">
                <AffectedMenus ingredientId={detailItem.id} />
              </TabsContent>
            </Tabs>
          )}
        </DialogContent>
      </Dialog>
    </div>
  );
}
```

- [ ] **Step 5: Create Ingredients page**

Create `web/src/app/(dashboard)/ingredients/page.tsx`:
```tsx
import { IngredientTable } from '@/components/ingredients/ingredient-table';

export default function IngredientsPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Bahan Baku</h1>
        <p className="text-sm text-gray-500">Kelola bahan baku, harga, dan lihat dampak ke HPP menu</p>
      </div>
      <IngredientTable />
    </div>
  );
}
```

---

### Task 12: Retail Management Page

**Files:**
- Create: `web/src/components/retail/retail-table.tsx`
- Create: `web/src/components/retail/retail-form.tsx`
- Create: `web/src/app/(dashboard)/retail/page.tsx`

- [ ] **Step 1: Create RetailForm component**

Create `web/src/components/retail/retail-form.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { retailService } from '@/services/retail.service';
import { useToast } from '@/components/ui/use-toast';
import type { RetailItem } from '@/types/retail';

interface RetailFormProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  editItem?: RetailItem | null;
  onSuccess: () => void;
}

export function RetailForm({ open, onOpenChange, editItem, onSuccess }: RetailFormProps) {
  const isEdit = !!editItem;
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [name, setName] = useState(editItem?.name ?? '');
  const [category, setCategory] = useState(editItem?.category ?? '');
  const [buyPrice, setBuyPrice] = useState<number>(editItem?.buyPrice ?? 0);
  const [sellingPrice, setSellingPrice] = useState<number>(editItem?.sellingPrice ?? 0);
  const [stockQty, setStockQty] = useState<number>(editItem?.stockQty ?? 0);
  const [minStock, setMinStock] = useState<number>(editItem?.minStock ?? 0);
  const [imageUrl, setImageUrl] = useState(editItem?.imageUrl ?? '');

  const profit = sellingPrice - buyPrice;
  const margin = sellingPrice > 0 ? ((profit / sellingPrice) * 100) : 0;

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      const data = { name, category, buyPrice, sellingPrice, stockQty, minStock, imageUrl: imageUrl || undefined };
      if (isEdit && editItem) {
        await retailService.update(editItem.id, data);
        toast({ title: 'Retail item berhasil diupdate' });
      } else {
        await retailService.create(data);
        toast({ title: 'Retail item berhasil ditambahkan' });
      }
      onSuccess();
      onOpenChange(false);
    } catch {
      toast({ title: 'Gagal menyimpan', variant: 'destructive' });
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>{isEdit ? 'Edit Item Retail' : 'Tambah Item Retail'}</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Nama</Label>
              <Input value={name} onChange={(e) => setName(e.target.value)} required />
            </div>
            <div className="space-y-2">
              <Label>Kategori</Label>
              <Input value={category} onChange={(e) => setCategory(e.target.value)} placeholder="Condiment, Snack, dll" required />
            </div>
          </div>
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Harga Beli (Rp)</Label>
              <Input type="number" min="0" value={buyPrice || ''} onChange={(e) => setBuyPrice(parseInt(e.target.value) || 0)} required />
            </div>
            <div className="space-y-2">
              <Label>Harga Jual (Rp)</Label>
              <Input type="number" min="0" value={sellingPrice || ''} onChange={(e) => setSellingPrice(parseInt(e.target.value) || 0)} required />
            </div>
          </div>
          {sellingPrice > 0 && buyPrice > 0 && (
            <div className="rounded-md bg-gray-50 p-3 text-sm">
              <span>Profit: <strong className="text-green-600">Rp {profit.toLocaleString('id-ID')}</strong></span>
              <span className="ml-4">Margin: <strong className={margin < 30 ? 'text-red-600' : 'text-green-600'}>{margin.toFixed(1)}%</strong></span>
            </div>
          )}
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Stok</Label>
              <Input type="number" min="0" value={stockQty || ''} onChange={(e) => setStockQty(parseFloat(e.target.value) || 0)} />
            </div>
            <div className="space-y-2">
              <Label>Min Stok</Label>
              <Input type="number" min="0" value={minStock || ''} onChange={(e) => setMinStock(parseFloat(e.target.value) || 0)} />
            </div>
          </div>
          <div className="space-y-2">
            <Label>URL Gambar (opsional)</Label>
            <Input value={imageUrl} onChange={(e) => setImageUrl(e.target.value)} />
          </div>
          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>Batal</Button>
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Menyimpan...' : isEdit ? 'Update' : 'Simpan'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

- [ ] **Step 2: Create RetailTable component**

Create `web/src/components/retail/retail-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Plus, Search, MoreHorizontal, Pencil, Trash2 } from 'lucide-react';
import { formatCurrency, formatPercent, formatNumber } from '@/lib/utils';
import { retailService } from '@/services/retail.service';
import { useDebounce } from '@/hooks/use-debounce';
import { useToast } from '@/components/ui/use-toast';
import { RetailForm } from './retail-form';
import type { RetailItem } from '@/types/retail';

export function RetailTable() {
  const [items, setItems] = useState<RetailItem[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [showForm, setShowForm] = useState(false);
  const [editItem, setEditItem] = useState<RetailItem | null>(null);
  const debouncedSearch = useDebounce(search, 300);
  const { toast } = useToast();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const res = await retailService.getAll({ page, limit: 20 });
      setItems(res.data);
      setTotal(res.total);
    } catch (err) { console.error(err); }
    finally { setIsLoading(false); }
  }, [page]);

  useEffect(() => { loadData(); }, [loadData]);

  const filtered = items.filter((i) => i.name.toLowerCase().includes(debouncedSearch.toLowerCase()));

  async function handleDelete(id: string) {
    if (!confirm('Yakin ingin menghapus item ini?')) return;
    try {
      await retailService.delete(id);
      toast({ title: 'Item berhasil dihapus' });
      loadData();
    } catch { toast({ title: 'Gagal menghapus', variant: 'destructive' }); }
  }

  const totalPages = Math.ceil(total / 20);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="relative w-72">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
          <Input placeholder="Cari item..." value={search} onChange={(e) => setSearch(e.target.value)} className="pl-9" />
        </div>
        <Button onClick={() => { setEditItem(null); setShowForm(true); }} className="gap-2">
          <Plus className="h-4 w-4" /> Tambah Item
        </Button>
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nama</TableHead>
              <TableHead>Kategori</TableHead>
              <TableHead>Harga Beli</TableHead>
              <TableHead>Harga Jual</TableHead>
              <TableHead>Profit</TableHead>
              <TableHead>Margin</TableHead>
              <TableHead>Stok</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow><TableCell colSpan={8} className="text-center py-8 text-gray-500">Memuat data...</TableCell></TableRow>
            ) : filtered.length === 0 ? (
              <TableRow><TableCell colSpan={8} className="text-center py-8 text-gray-500">Tidak ada data</TableCell></TableRow>
            ) : filtered.map((item) => {
              const profit = item.sellingPrice - item.buyPrice;
              const margin = item.sellingPrice > 0 ? (profit / item.sellingPrice) * 100 : 0;
              return (
                <TableRow key={item.id}>
                  <TableCell className="font-medium">{item.name}</TableCell>
                  <TableCell><Badge variant="secondary">{item.category}</Badge></TableCell>
                  <TableCell>{formatCurrency(item.buyPrice)}</TableCell>
                  <TableCell>{formatCurrency(item.sellingPrice)}</TableCell>
                  <TableCell className="text-green-600">{formatCurrency(profit)}</TableCell>
                  <TableCell>{formatPercent(margin)}</TableCell>
                  <TableCell>{formatNumber(item.stockQty)}</TableCell>
                  <TableCell>
                    <DropdownMenu>
                      <DropdownMenuTrigger asChild><Button variant="ghost" size="icon"><MoreHorizontal className="h-4 w-4" /></Button></DropdownMenuTrigger>
                      <DropdownMenuContent align="end">
                        <DropdownMenuItem onClick={() => { setEditItem(item); setShowForm(true); }}><Pencil className="mr-2 h-4 w-4" /> Edit</DropdownMenuItem>
                        <DropdownMenuItem onClick={() => handleDelete(item.id)} className="text-red-600"><Trash2 className="mr-2 h-4 w-4" /> Hapus</DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </TableCell>
                </TableRow>
              );
            })}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button variant="outline" size="sm" disabled={page <= 1} onClick={() => setPage(page - 1)}>Sebelumnya</Button>
          <span className="text-sm text-gray-500">Halaman {page} dari {totalPages}</span>
          <Button variant="outline" size="sm" disabled={page >= totalPages} onClick={() => setPage(page + 1)}>Selanjutnya</Button>
        </div>
      )}

      <RetailForm open={showForm} onOpenChange={setShowForm} editItem={editItem} onSuccess={loadData} />
    </div>
  );
}
```

- [ ] **Step 3: Create Retail page**

Create `web/src/app/(dashboard)/retail/page.tsx`:
```tsx
import { RetailTable } from '@/components/retail/retail-table';

export default function RetailPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Retail Management</h1>
        <p className="text-sm text-gray-500">Kelola item retail (non-menu) beserta profit</p>
      </div>
      <RetailTable />
    </div>
  );
}
```

---

### Task 13: Promo Management Page

**Files:**
- Create: `web/src/components/promos/promo-table.tsx`
- Create: `web/src/components/promos/promo-form.tsx`
- Create: `web/src/app/(dashboard)/promos/page.tsx`

- [ ] **Step 1: Create PromoForm component**

Create `web/src/components/promos/promo-form.tsx`:
```tsx
'use client';

import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Switch } from '@/components/ui/switch';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Calendar } from '@/components/ui/calendar';
import { CalendarIcon, Plus, Trash2 } from 'lucide-react';
import { format } from 'date-fns';
import { promoService } from '@/services/promo.service';
import { menuService } from '@/services/menu.service';
import { retailService } from '@/services/retail.service';
import { useToast } from '@/components/ui/use-toast';
import { PROMO_TYPE_LABELS } from '@/lib/constants';
import type { Promo, PromoType, PromoItemInput, PromoItemRole, ItemType } from '@/types/promo';
import type { MenuItem } from '@/types/menu';
import type { RetailItem } from '@/types/retail';

interface PromoFormProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  editItem?: Promo | null;
  onSuccess: () => void;
}

export function PromoForm({ open, onOpenChange, editItem, onSuccess }: PromoFormProps) {
  const isEdit = !!editItem;
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [name, setName] = useState(editItem?.name ?? '');
  const [type, setType] = useState<PromoType>(editItem?.type ?? 'PERCENT');
  const [value, setValue] = useState<number>(editItem?.value ?? 0);
  const [minPurchaseAmount, setMinPurchaseAmount] = useState<number>(editItem?.minPurchaseAmount ?? 0);
  const [startDate, setStartDate] = useState<Date | undefined>(editItem?.startDate ? new Date(editItem.startDate) : new Date());
  const [endDate, setEndDate] = useState<Date | undefined>(editItem?.endDate ? new Date(editItem.endDate) : undefined);
  const [isActive, setIsActive] = useState(editItem?.isActive ?? true);
  const [maxUsage, setMaxUsage] = useState<number>(editItem?.maxUsage ?? 0);
  const [promoItems, setPromoItems] = useState<PromoItemInput[]>(
    editItem?.items?.map((i) => ({ itemType: i.itemType, itemId: i.itemId, role: i.role })) ?? [],
  );

  const [menuItems, setMenuItems] = useState<MenuItem[]>([]);
  const [retailItems, setRetailItems] = useState<RetailItem[]>([]);

  useEffect(() => {
    menuService.getAll({ limit: 100 }).then((res) => setMenuItems(res.data)).catch(console.error);
    retailService.getAll({ limit: 100 }).then((res) => setRetailItems(res.data)).catch(console.error);
  }, []);

  function addPromoItem() {
    setPromoItems([...promoItems, { itemType: 'MENU', itemId: '', role: 'TARGET' }]);
  }

  function removePromoItem(index: number) {
    setPromoItems(promoItems.filter((_, i) => i !== index));
  }

  function updatePromoItem(index: number, field: keyof PromoItemInput, val: string) {
    const updated = [...promoItems];
    updated[index] = { ...updated[index], [field]: val };
    setPromoItems(updated);
  }

  const needsItems = ['PERCENT', 'FIXED', 'BUY_X_GET_Y', 'BUNDLE'].includes(type);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!startDate || !endDate) return;
    setIsSubmitting(true);

    try {
      const data = {
        name,
        type,
        value,
        minPurchaseAmount: minPurchaseAmount || undefined,
        startDate: startDate.toISOString(),
        endDate: endDate.toISOString(),
        isActive,
        maxUsage: maxUsage || undefined,
        items: promoItems.filter((i) => i.itemId),
      };

      if (isEdit && editItem) {
        await promoService.update(editItem.id, data);
        toast({ title: 'Promo berhasil diupdate' });
      } else {
        await promoService.create(data);
        toast({ title: 'Promo berhasil ditambahkan' });
      }
      onSuccess();
      onOpenChange(false);
    } catch {
      toast({ title: 'Gagal menyimpan promo', variant: 'destructive' });
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-2xl max-h-[90vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle>{isEdit ? 'Edit Promo' : 'Tambah Promo Baru'}</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Nama Promo</Label>
              <Input value={name} onChange={(e) => setName(e.target.value)} required />
            </div>
            <div className="space-y-2">
              <Label>Tipe Promo</Label>
              <Select value={type} onValueChange={(val) => setType(val as PromoType)}>
                <SelectTrigger><SelectValue /></SelectTrigger>
                <SelectContent>
                  {Object.entries(PROMO_TYPE_LABELS).map(([key, label]) => (
                    <SelectItem key={key} value={key}>{label}</SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Nilai ({type === 'PERCENT' || type === 'MIN_PURCHASE' ? '%' : 'Rp'})</Label>
              <Input type="number" min="0" value={value || ''} onChange={(e) => setValue(parseFloat(e.target.value) || 0)} required />
            </div>
            {type === 'MIN_PURCHASE' && (
              <div className="space-y-2">
                <Label>Min. Pembelian (Rp)</Label>
                <Input type="number" min="0" value={minPurchaseAmount || ''} onChange={(e) => setMinPurchaseAmount(parseInt(e.target.value) || 0)} />
              </div>
            )}
            <div className="space-y-2">
              <Label>Max Penggunaan (0 = unlimited)</Label>
              <Input type="number" min="0" value={maxUsage || ''} onChange={(e) => setMaxUsage(parseInt(e.target.value) || 0)} />
            </div>
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div className="space-y-2">
              <Label>Tanggal Mulai</Label>
              <Popover>
                <PopoverTrigger asChild>
                  <Button variant="outline" className="w-full justify-start gap-2">
                    <CalendarIcon className="h-4 w-4" />
                    {startDate ? format(startDate, 'dd/MM/yyyy') : 'Pilih tanggal'}
                  </Button>
                </PopoverTrigger>
                <PopoverContent className="w-auto p-0"><Calendar mode="single" selected={startDate} onSelect={setStartDate} /></PopoverContent>
              </Popover>
            </div>
            <div className="space-y-2">
              <Label>Tanggal Berakhir</Label>
              <Popover>
                <PopoverTrigger asChild>
                  <Button variant="outline" className="w-full justify-start gap-2">
                    <CalendarIcon className="h-4 w-4" />
                    {endDate ? format(endDate, 'dd/MM/yyyy') : 'Pilih tanggal'}
                  </Button>
                </PopoverTrigger>
                <PopoverContent className="w-auto p-0"><Calendar mode="single" selected={endDate} onSelect={setEndDate} /></PopoverContent>
              </Popover>
            </div>
          </div>

          <div className="flex items-center gap-2">
            <Switch checked={isActive} onCheckedChange={setIsActive} />
            <Label>Aktif</Label>
          </div>

          {needsItems && (
            <div className="space-y-3">
              <div className="flex items-center justify-between">
                <Label>Item Promo</Label>
                <Button type="button" variant="outline" size="sm" onClick={addPromoItem} className="gap-1">
                  <Plus className="h-4 w-4" /> Tambah Item
                </Button>
              </div>
              {promoItems.map((pi, idx) => (
                <div key={idx} className="flex items-end gap-2 rounded-lg border p-3">
                  <div className="w-28 space-y-1">
                    <label className="text-xs text-gray-500">Tipe</label>
                    <Select value={pi.itemType} onValueChange={(val) => updatePromoItem(idx, 'itemType', val)}>
                      <SelectTrigger><SelectValue /></SelectTrigger>
                      <SelectContent>
                        <SelectItem value="MENU">Menu</SelectItem>
                        <SelectItem value="RETAIL">Retail</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                  <div className="flex-1 space-y-1">
                    <label className="text-xs text-gray-500">Item</label>
                    <Select value={pi.itemId} onValueChange={(val) => updatePromoItem(idx, 'itemId', val)}>
                      <SelectTrigger><SelectValue placeholder="Pilih item" /></SelectTrigger>
                      <SelectContent>
                        {(pi.itemType === 'MENU' ? menuItems : retailItems).map((item) => (
                          <SelectItem key={item.id} value={item.id}>{item.name}</SelectItem>
                        ))}
                      </SelectContent>
                    </Select>
                  </div>
                  <div className="w-36 space-y-1">
                    <label className="text-xs text-gray-500">Role</label>
                    <Select value={pi.role} onValueChange={(val) => updatePromoItem(idx, 'role', val)}>
                      <SelectTrigger><SelectValue /></SelectTrigger>
                      <SelectContent>
                        <SelectItem value="TARGET">Target</SelectItem>
                        <SelectItem value="FREE_ITEM">Free Item</SelectItem>
                        <SelectItem value="BUNDLE_MEMBER">Bundle</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                  <Button type="button" variant="ghost" size="icon" onClick={() => removePromoItem(idx)} className="text-red-500">
                    <Trash2 className="h-4 w-4" />
                  </Button>
                </div>
              ))}
            </div>
          )}

          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>Batal</Button>
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Menyimpan...' : isEdit ? 'Update' : 'Simpan'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

- [ ] **Step 2: Create PromoTable component**

Create `web/src/components/promos/promo-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Plus, Search, MoreHorizontal, Pencil, Trash2 } from 'lucide-react';
import { formatCurrency, formatDate } from '@/lib/utils';
import { promoService } from '@/services/promo.service';
import { useDebounce } from '@/hooks/use-debounce';
import { useToast } from '@/components/ui/use-toast';
import { PROMO_TYPE_LABELS } from '@/lib/constants';
import { PromoForm } from './promo-form';
import type { Promo } from '@/types/promo';

export function PromoTable() {
  const [items, setItems] = useState<Promo[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [showForm, setShowForm] = useState(false);
  const [editItem, setEditItem] = useState<Promo | null>(null);
  const debouncedSearch = useDebounce(search, 300);
  const { toast } = useToast();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const res = await promoService.getAll({ page, limit: 20 });
      setItems(res.data);
      setTotal(res.total);
    } catch (err) { console.error(err); }
    finally { setIsLoading(false); }
  }, [page]);

  useEffect(() => { loadData(); }, [loadData]);

  const filtered = items.filter((i) => i.name.toLowerCase().includes(debouncedSearch.toLowerCase()));

  async function handleDelete(id: string) {
    if (!confirm('Yakin ingin menghapus promo ini?')) return;
    try {
      await promoService.delete(id);
      toast({ title: 'Promo berhasil dihapus' });
      loadData();
    } catch { toast({ title: 'Gagal menghapus', variant: 'destructive' }); }
  }

  function getStatusBadge(promo: Promo) {
    const now = new Date();
    const start = new Date(promo.startDate);
    const end = new Date(promo.endDate);
    if (!promo.isActive) return <Badge variant="secondary">Nonaktif</Badge>;
    if (now < start) return <Badge className="bg-yellow-100 text-yellow-800">Belum Mulai</Badge>;
    if (now > end) return <Badge variant="destructive">Berakhir</Badge>;
    if (promo.maxUsage && promo.usedCount >= promo.maxUsage) return <Badge variant="destructive">Max Usage</Badge>;
    return <Badge className="bg-green-100 text-green-800">Aktif</Badge>;
  }

  const totalPages = Math.ceil(total / 20);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="relative w-72">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
          <Input placeholder="Cari promo..." value={search} onChange={(e) => setSearch(e.target.value)} className="pl-9" />
        </div>
        <Button onClick={() => { setEditItem(null); setShowForm(true); }} className="gap-2">
          <Plus className="h-4 w-4" /> Tambah Promo
        </Button>
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nama</TableHead>
              <TableHead>Tipe</TableHead>
              <TableHead>Nilai</TableHead>
              <TableHead>Periode</TableHead>
              <TableHead>Penggunaan</TableHead>
              <TableHead>Status</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow><TableCell colSpan={7} className="text-center py-8 text-gray-500">Memuat data...</TableCell></TableRow>
            ) : filtered.length === 0 ? (
              <TableRow><TableCell colSpan={7} className="text-center py-8 text-gray-500">Tidak ada data</TableCell></TableRow>
            ) : filtered.map((item) => (
              <TableRow key={item.id}>
                <TableCell className="font-medium">{item.name}</TableCell>
                <TableCell><Badge variant="outline">{PROMO_TYPE_LABELS[item.type] || item.type}</Badge></TableCell>
                <TableCell>
                  {item.type === 'PERCENT' || item.type === 'MIN_PURCHASE' ? `${item.value}%` : formatCurrency(item.value)}
                </TableCell>
                <TableCell className="text-sm">
                  {formatDate(item.startDate)} - {formatDate(item.endDate)}
                </TableCell>
                <TableCell>
                  {item.usedCount}{item.maxUsage ? `/${item.maxUsage}` : ''}
                </TableCell>
                <TableCell>{getStatusBadge(item)}</TableCell>
                <TableCell>
                  <DropdownMenu>
                    <DropdownMenuTrigger asChild><Button variant="ghost" size="icon"><MoreHorizontal className="h-4 w-4" /></Button></DropdownMenuTrigger>
                    <DropdownMenuContent align="end">
                      <DropdownMenuItem onClick={() => { setEditItem(item); setShowForm(true); }}><Pencil className="mr-2 h-4 w-4" /> Edit</DropdownMenuItem>
                      <DropdownMenuItem onClick={() => handleDelete(item.id)} className="text-red-600"><Trash2 className="mr-2 h-4 w-4" /> Hapus</DropdownMenuItem>
                    </DropdownMenuContent>
                  </DropdownMenu>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button variant="outline" size="sm" disabled={page <= 1} onClick={() => setPage(page - 1)}>Sebelumnya</Button>
          <span className="text-sm text-gray-500">Halaman {page} dari {totalPages}</span>
          <Button variant="outline" size="sm" disabled={page >= totalPages} onClick={() => setPage(page + 1)}>Selanjutnya</Button>
        </div>
      )}

      <PromoForm open={showForm} onOpenChange={setShowForm} editItem={editItem} onSuccess={loadData} />
    </div>
  );
}
```

- [ ] **Step 3: Create Promos page**

Create `web/src/app/(dashboard)/promos/page.tsx`:
```tsx
import { PromoTable } from '@/components/promos/promo-table';

export default function PromosPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Promo Management</h1>
        <p className="text-sm text-gray-500">Kelola promo diskon, bundle, dan buy-x-get-y</p>
      </div>
      <PromoTable />
    </div>
  );
}
```

---

### Task 14: User Management Page

**Files:**
- Create: `web/src/components/users/user-table.tsx`
- Create: `web/src/components/users/user-form.tsx`
- Create: `web/src/app/(dashboard)/users/page.tsx`

- [ ] **Step 1: Create UserForm component**

Create `web/src/components/users/user-form.tsx`:
```tsx
'use client';

import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Switch } from '@/components/ui/switch';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter } from '@/components/ui/dialog';
import { userService } from '@/services/user.service';
import { useToast } from '@/components/ui/use-toast';
import type { User, Role } from '@/types/user';

interface UserFormProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  editItem?: User | null;
  onSuccess: () => void;
}

export function UserForm({ open, onOpenChange, editItem, onSuccess }: UserFormProps) {
  const isEdit = !!editItem;
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [roles, setRoles] = useState<Role[]>([]);
  const [name, setName] = useState(editItem?.name ?? '');
  const [email, setEmail] = useState(editItem?.email ?? '');
  const [password, setPassword] = useState('');
  const [roleId, setRoleId] = useState(editItem?.role?.id ?? '');
  const [isActive, setIsActive] = useState(editItem?.isActive ?? true);

  useEffect(() => {
    userService.getRoles().then(setRoles).catch(console.error);
  }, []);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      if (isEdit && editItem) {
        const data: Record<string, unknown> = { name, email, roleId, isActive };
        if (password) data.password = password;
        await userService.update(editItem.id, data);
        toast({ title: 'User berhasil diupdate' });
      } else {
        await userService.create({ name, email, password, roleId });
        toast({ title: 'User berhasil ditambahkan' });
      }
      onSuccess();
      onOpenChange(false);
    } catch {
      toast({ title: 'Gagal menyimpan user', variant: 'destructive' });
    } finally {
      setIsSubmitting(false);
    }
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>{isEdit ? 'Edit User' : 'Tambah User Baru'}</DialogTitle>
        </DialogHeader>
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="space-y-2">
            <Label>Nama</Label>
            <Input value={name} onChange={(e) => setName(e.target.value)} required />
          </div>
          <div className="space-y-2">
            <Label>Email</Label>
            <Input type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
          </div>
          <div className="space-y-2">
            <Label>{isEdit ? 'Password Baru (kosongkan jika tidak diubah)' : 'Password'}</Label>
            <Input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              required={!isEdit}
              minLength={6}
              placeholder={isEdit ? 'Kosongkan jika tidak diubah' : 'Min. 6 karakter'}
            />
          </div>
          <div className="space-y-2">
            <Label>Role</Label>
            <Select value={roleId} onValueChange={setRoleId}>
              <SelectTrigger><SelectValue placeholder="Pilih role" /></SelectTrigger>
              <SelectContent>
                {roles.map((role) => (
                  <SelectItem key={role.id} value={role.id}>{role.name}</SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>
          {isEdit && (
            <div className="flex items-center gap-2">
              <Switch checked={isActive} onCheckedChange={setIsActive} />
              <Label>Aktif</Label>
            </div>
          )}
          <DialogFooter>
            <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>Batal</Button>
            <Button type="submit" disabled={isSubmitting}>
              {isSubmitting ? 'Menyimpan...' : isEdit ? 'Update' : 'Simpan'}
            </Button>
          </DialogFooter>
        </form>
      </DialogContent>
    </Dialog>
  );
}
```

- [ ] **Step 2: Create UserTable component**

Create `web/src/components/users/user-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Badge } from '@/components/ui/badge';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu';
import { Plus, Search, MoreHorizontal, Pencil, Trash2 } from 'lucide-react';
import { formatDate } from '@/lib/utils';
import { userService } from '@/services/user.service';
import { useDebounce } from '@/hooks/use-debounce';
import { useToast } from '@/components/ui/use-toast';
import { UserForm } from './user-form';
import type { User } from '@/types/user';

export function UserTable() {
  const [items, setItems] = useState<User[]>([]);
  const [total, setTotal] = useState(0);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [showForm, setShowForm] = useState(false);
  const [editItem, setEditItem] = useState<User | null>(null);
  const debouncedSearch = useDebounce(search, 300);
  const { toast } = useToast();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const res = await userService.getAll({ page, limit: 20 });
      setItems(res.data);
      setTotal(res.total);
    } catch (err) { console.error(err); }
    finally { setIsLoading(false); }
  }, [page]);

  useEffect(() => { loadData(); }, [loadData]);

  const filtered = items.filter((u) =>
    u.name.toLowerCase().includes(debouncedSearch.toLowerCase()) ||
    u.email.toLowerCase().includes(debouncedSearch.toLowerCase()),
  );

  async function handleDelete(id: string) {
    if (!confirm('Yakin ingin menghapus user ini?')) return;
    try {
      await userService.delete(id);
      toast({ title: 'User berhasil dihapus' });
      loadData();
    } catch { toast({ title: 'Gagal menghapus', variant: 'destructive' }); }
  }

  const totalPages = Math.ceil(total / 20);

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="relative w-72">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400" />
          <Input placeholder="Cari user..." value={search} onChange={(e) => setSearch(e.target.value)} className="pl-9" />
        </div>
        <Button onClick={() => { setEditItem(null); setShowForm(true); }} className="gap-2">
          <Plus className="h-4 w-4" /> Tambah User
        </Button>
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Nama</TableHead>
              <TableHead>Email</TableHead>
              <TableHead>Role</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Dibuat</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow><TableCell colSpan={6} className="text-center py-8 text-gray-500">Memuat data...</TableCell></TableRow>
            ) : filtered.length === 0 ? (
              <TableRow><TableCell colSpan={6} className="text-center py-8 text-gray-500">Tidak ada data</TableCell></TableRow>
            ) : filtered.map((user) => (
              <TableRow key={user.id}>
                <TableCell className="font-medium">{user.name}</TableCell>
                <TableCell>{user.email}</TableCell>
                <TableCell><Badge variant="outline">{user.role.name}</Badge></TableCell>
                <TableCell>
                  <Badge variant={user.isActive ? 'default' : 'secondary'}>
                    {user.isActive ? 'Aktif' : 'Nonaktif'}
                  </Badge>
                </TableCell>
                <TableCell className="text-sm">{formatDate(user.createdAt)}</TableCell>
                <TableCell>
                  <DropdownMenu>
                    <DropdownMenuTrigger asChild><Button variant="ghost" size="icon"><MoreHorizontal className="h-4 w-4" /></Button></DropdownMenuTrigger>
                    <DropdownMenuContent align="end">
                      <DropdownMenuItem onClick={() => { setEditItem(user); setShowForm(true); }}><Pencil className="mr-2 h-4 w-4" /> Edit</DropdownMenuItem>
                      <DropdownMenuItem onClick={() => handleDelete(user.id)} className="text-red-600"><Trash2 className="mr-2 h-4 w-4" /> Hapus</DropdownMenuItem>
                    </DropdownMenuContent>
                  </DropdownMenu>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button variant="outline" size="sm" disabled={page <= 1} onClick={() => setPage(page - 1)}>Sebelumnya</Button>
          <span className="text-sm text-gray-500">Halaman {page} dari {totalPages}</span>
          <Button variant="outline" size="sm" disabled={page >= totalPages} onClick={() => setPage(page + 1)}>Selanjutnya</Button>
        </div>
      )}

      <UserForm open={showForm} onOpenChange={setShowForm} editItem={editItem} onSuccess={loadData} />
    </div>
  );
}
```

- [ ] **Step 3: Create Users page**

Create `web/src/app/(dashboard)/users/page.tsx`:
```tsx
import { UserTable } from '@/components/users/user-table';

export default function UsersPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">User Management</h1>
        <p className="text-sm text-gray-500">Kelola user dan assign role (RBAC)</p>
      </div>
      <UserTable />
    </div>
  );
}
```

---

### Task 15: Transaction History Page

**Files:**
- Create: `web/src/components/transactions/transaction-table.tsx`
- Create: `web/src/components/transactions/transaction-detail.tsx`
- Create: `web/src/app/(dashboard)/transactions/page.tsx`

- [ ] **Step 1: Create TransactionDetail component**

Create `web/src/components/transactions/transaction-detail.tsx`:
```tsx
'use client';

import { useEffect, useState } from 'react';
import { Skeleton } from '@/components/ui/skeleton';
import { Badge } from '@/components/ui/badge';
import { Separator } from '@/components/ui/separator';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { formatCurrency, formatDateTime } from '@/lib/utils';
import { PAYMENT_METHOD_LABELS } from '@/lib/constants';
import { transactionService } from '@/services/transaction.service';
import type { Transaction } from '@/types/transaction';

interface TransactionDetailProps {
  transactionId: string;
}

export function TransactionDetail({ transactionId }: TransactionDetailProps) {
  const [data, setData] = useState<Transaction | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    setIsLoading(true);
    transactionService
      .getById(transactionId)
      .then(setData)
      .catch(console.error)
      .finally(() => setIsLoading(false));
  }, [transactionId]);

  if (isLoading) return <Skeleton className="h-60 w-full" />;
  if (!data) return <p className="text-gray-500">Data tidak ditemukan</p>;

  return (
    <div className="space-y-4">
      <div className="grid grid-cols-2 gap-4 text-sm">
        <div>
          <p className="text-gray-500">Kode Transaksi</p>
          <p className="font-mono font-bold">{data.transactionCode}</p>
        </div>
        <div>
          <p className="text-gray-500">Tanggal</p>
          <p>{formatDateTime(data.createdAt)}</p>
        </div>
        <div>
          <p className="text-gray-500">Kasir</p>
          <p>{data.cashier.name}</p>
        </div>
        <div>
          <p className="text-gray-500">Metode Pembayaran</p>
          <Badge variant="outline">{PAYMENT_METHOD_LABELS[data.paymentMethod] || data.paymentMethod}</Badge>
        </div>
      </div>

      <Separator />

      <div>
        <p className="mb-2 text-sm font-medium">Item</p>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Tipe</TableHead>
              <TableHead>Item ID</TableHead>
              <TableHead>Qty</TableHead>
              <TableHead>Harga</TableHead>
              <TableHead>HPP</TableHead>
              <TableHead className="text-right">Subtotal</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {data.items.map((item) => (
              <TableRow key={item.id}>
                <TableCell><Badge variant="secondary">{item.itemType}</Badge></TableCell>
                <TableCell className="font-mono text-xs">{item.itemId.slice(0, 8)}...</TableCell>
                <TableCell>{item.qty}</TableCell>
                <TableCell>{formatCurrency(item.unitPrice)}</TableCell>
                <TableCell className="text-red-600">{formatCurrency(item.unitHpp)}</TableCell>
                <TableCell className="text-right">{formatCurrency(item.subtotal)}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      <Separator />

      <div className="space-y-1 text-sm">
        <div className="flex justify-between">
          <span>Subtotal</span>
          <span>{formatCurrency(data.totalAmount + data.promoDiscount)}</span>
        </div>
        {data.promoDiscount > 0 && (
          <div className="flex justify-between text-orange-600">
            <span>
              Diskon Promo
              {data.promos?.[0]?.promo?.name && ` (${data.promos[0].promo.name})`}
            </span>
            <span>-{formatCurrency(data.promoDiscount)}</span>
          </div>
        )}
        <div className="flex justify-between font-bold text-base">
          <span>Total</span>
          <span>{formatCurrency(data.totalAmount)}</span>
        </div>
        <Separator className="my-2" />
        <div className="flex justify-between text-red-600">
          <span>Total HPP</span>
          <span>{formatCurrency(data.totalHpp)}</span>
        </div>
        <div className="flex justify-between text-green-600 font-bold">
          <span>Profit</span>
          <span>{formatCurrency(data.totalProfit)}</span>
        </div>
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Create TransactionTable component**

Create `web/src/components/transactions/transaction-table.tsx`:
```tsx
'use client';

import { useEffect, useState, useCallback } from 'react';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover';
import { Calendar } from '@/components/ui/calendar';
import { CalendarIcon, Eye } from 'lucide-react';
import { format } from 'date-fns';
import { id as idLocale } from 'date-fns/locale';
import { formatCurrency, formatDateTime } from '@/lib/utils';
import { PAYMENT_METHOD_LABELS } from '@/lib/constants';
import { transactionService } from '@/services/transaction.service';
import { TransactionDetail } from './transaction-detail';
import type { Transaction, TransactionFilters, PaymentMethod } from '@/types/transaction';

export function TransactionTable() {
  const [items, setItems] = useState<Transaction[]>([]);
  const [total, setTotal] = useState(0);
  const [isLoading, setIsLoading] = useState(true);
  const [selectedId, setSelectedId] = useState<string | null>(null);
  const [filters, setFilters] = useState<TransactionFilters>({
    page: 1,
    limit: 20,
    paymentMethod: '',
  });
  const [startDate, setStartDate] = useState<Date | undefined>();
  const [endDate, setEndDate] = useState<Date | undefined>();

  const loadData = useCallback(async () => {
    setIsLoading(true);
    try {
      const params: TransactionFilters = {
        ...filters,
        startDate: startDate ? startDate.toISOString().slice(0, 10) : undefined,
        endDate: endDate ? endDate.toISOString().slice(0, 10) : undefined,
      };
      const res = await transactionService.getAll(params);
      setItems(res.data);
      setTotal(res.total);
    } catch (err) { console.error(err); }
    finally { setIsLoading(false); }
  }, [filters, startDate, endDate]);

  useEffect(() => { loadData(); }, [loadData]);

  const totalPages = Math.ceil(total / (filters.limit || 20));

  return (
    <div className="space-y-4">
      <div className="flex flex-wrap items-center gap-3">
        <Popover>
          <PopoverTrigger asChild>
            <Button variant="outline" className="gap-2">
              <CalendarIcon className="h-4 w-4" />
              {startDate ? format(startDate, 'dd MMM yyyy', { locale: idLocale }) : 'Dari tanggal'}
            </Button>
          </PopoverTrigger>
          <PopoverContent className="w-auto p-0"><Calendar mode="single" selected={startDate} onSelect={setStartDate} /></PopoverContent>
        </Popover>
        <span className="text-gray-400">-</span>
        <Popover>
          <PopoverTrigger asChild>
            <Button variant="outline" className="gap-2">
              <CalendarIcon className="h-4 w-4" />
              {endDate ? format(endDate, 'dd MMM yyyy', { locale: idLocale }) : 'Sampai tanggal'}
            </Button>
          </PopoverTrigger>
          <PopoverContent className="w-auto p-0"><Calendar mode="single" selected={endDate} onSelect={setEndDate} /></PopoverContent>
        </Popover>

        <Select
          value={filters.paymentMethod || 'all'}
          onValueChange={(val) => setFilters({ ...filters, paymentMethod: val === 'all' ? '' : val as PaymentMethod, page: 1 })}
        >
          <SelectTrigger className="w-40"><SelectValue placeholder="Pembayaran" /></SelectTrigger>
          <SelectContent>
            <SelectItem value="all">Semua Metode</SelectItem>
            <SelectItem value="CASH">Tunai</SelectItem>
            <SelectItem value="QRIS">QRIS</SelectItem>
            <SelectItem value="EDC">EDC / Kartu</SelectItem>
          </SelectContent>
        </Select>

        {(startDate || endDate) && (
          <Button variant="ghost" size="sm" onClick={() => { setStartDate(undefined); setEndDate(undefined); }}>
            Reset Filter
          </Button>
        )}
      </div>

      <div className="rounded-md border bg-white">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Kode</TableHead>
              <TableHead>Tanggal</TableHead>
              <TableHead>Kasir</TableHead>
              <TableHead>Pembayaran</TableHead>
              <TableHead>Total</TableHead>
              <TableHead>Profit</TableHead>
              <TableHead>Promo</TableHead>
              <TableHead className="w-16" />
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow><TableCell colSpan={8} className="text-center py-8 text-gray-500">Memuat data...</TableCell></TableRow>
            ) : items.length === 0 ? (
              <TableRow><TableCell colSpan={8} className="text-center py-8 text-gray-500">Tidak ada transaksi</TableCell></TableRow>
            ) : items.map((txn) => (
              <TableRow key={txn.id}>
                <TableCell className="font-mono text-sm">{txn.transactionCode}</TableCell>
                <TableCell className="text-sm">{formatDateTime(txn.createdAt)}</TableCell>
                <TableCell>{txn.cashier.name}</TableCell>
                <TableCell>
                  <Badge variant="outline">{PAYMENT_METHOD_LABELS[txn.paymentMethod] || txn.paymentMethod}</Badge>
                </TableCell>
                <TableCell className="font-medium">{formatCurrency(txn.totalAmount)}</TableCell>
                <TableCell className="text-green-600">{formatCurrency(txn.totalProfit)}</TableCell>
                <TableCell>
                  {txn.promoDiscount > 0 ? (
                    <Badge className="bg-orange-100 text-orange-800">-{formatCurrency(txn.promoDiscount)}</Badge>
                  ) : (
                    <span className="text-gray-400">-</span>
                  )}
                </TableCell>
                <TableCell>
                  <Button variant="ghost" size="icon" onClick={() => setSelectedId(txn.id)}>
                    <Eye className="h-4 w-4" />
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>

      {totalPages > 1 && (
        <div className="flex items-center justify-end gap-2">
          <Button variant="outline" size="sm" disabled={(filters.page || 1) <= 1} onClick={() => setFilters({ ...filters, page: (filters.page || 1) - 1 })}>Sebelumnya</Button>
          <span className="text-sm text-gray-500">Halaman {filters.page || 1} dari {totalPages}</span>
          <Button variant="outline" size="sm" disabled={(filters.page || 1) >= totalPages} onClick={() => setFilters({ ...filters, page: (filters.page || 1) + 1 })}>Selanjutnya</Button>
        </div>
      )}

      <Dialog open={!!selectedId} onOpenChange={() => setSelectedId(null)}>
        <DialogContent className="max-w-2xl max-h-[80vh] overflow-y-auto">
          <DialogHeader>
            <DialogTitle>Detail Transaksi</DialogTitle>
          </DialogHeader>
          {selectedId && <TransactionDetail transactionId={selectedId} />}
        </DialogContent>
      </Dialog>
    </div>
  );
}
```

- [ ] **Step 3: Create Transactions page**

Create `web/src/app/(dashboard)/transactions/page.tsx`:
```tsx
import { TransactionTable } from '@/components/transactions/transaction-table';

export default function TransactionsPage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold">Riwayat Transaksi</h1>
        <p className="text-sm text-gray-500">Lihat semua transaksi penjualan dan detail struk</p>
      </div>
      <TransactionTable />
    </div>
  );
}
```

---

### Task 16: Final — Verify & Smoke Test

- [ ] **Step 1: Verify all pages render without errors**

```bash
cd ~/Downloads/PROJECT/web
npm run build
```

Expected: Build succeeds without TypeScript or import errors.

- [ ] **Step 2: Run all tests**

```bash
cd ~/Downloads/PROJECT/web
npx jest --verbose
```

Expected: All component tests pass.

- [ ] **Step 3: Verify RBAC — Owner sees all menu items**

Start the dev server and login as `owner@pos.com` / `password123`. Confirm all sidebar items are visible: Dashboard, Menu, Bahan Baku, Retail, Promo, Users, Transaksi.

- [ ] **Step 4: Verify RBAC — Kitchen sees limited menu items**

Login as `kitchen@pos.com` / `password123`. Confirm only Menu and Bahan Baku appear in the sidebar. Dashboard, Retail, Promo, Users, Transaksi should NOT be visible.

- [ ] **Step 5: Verify dashboard charts load with data**

Login as owner, navigate to Dashboard. Verify:
- 5 summary cards render (Total Penjualan, Total HPP, Gross Profit, Profit Margin, Jumlah Transaksi)
- Sales trend line chart renders
- Profit per category pie chart renders
- Top 10 profit bar chart renders
- Top 10 qty bar chart renders
- Scatter plot (laris vs untung) renders
- Promo analytics section renders
- Menu detail table renders with margin alerts

- [ ] **Step 6: Commit**

```bash
cd ~/Downloads/PROJECT/web
git init
git add .
git commit -m "feat: complete Phase 1 web dashboard with RBAC, analytics, and management pages"
```
