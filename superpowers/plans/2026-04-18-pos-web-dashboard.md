# POS System Phase 1 — Web Dashboard Impl Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build Next.js web dashboard/CMS w/ RBAC nav, profit analytics, mgmt pages for all Phase 1 modules.

**Architecture:** Next.js App Router, server+client components. shadcn/ui for UI, Recharts for charts, axios for API. Auth via JWT in cookies. Middleware handles route protection + RBAC.

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

### Task 1: Project Setup & Deps

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

- [ ] **Step 2: Install deps**

```bash
cd ~/Downloads/PROJECT/web
npm install
npm install axios js-cookie recharts date-fns lucide-react clsx tailwind-merge class-variance-authority
npm install -D @types/js-cookie @testing-library/react @testing-library/jest-dom @testing-library/user-event jest jest-environment-jsdom @types/jest ts-jest
```

- [ ] **Step 3: Init shadcn/ui**

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

- [ ] **Step 8: Verify app starts**

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

- [ ] **Step 1: Generic API res types**

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

- [ ] **Step 2: Auth types**

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

- [ ] **Step 3: User types**

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

- [ ] **Step 4: Menu types**

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

- [ ] **Step 5: Ingredient types**

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

- [ ] **Step 6: Retail types**

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

- [ ] **Step 7: Promo types**

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

- [ ] **Step 8: Transaction types**

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

- [ ] **Step 9: Dashboard types**

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

### Task 3: Core Layer — API Client & Utils

**Files:**
- Create: `web/src/lib/api.ts`
- Create: `web/src/lib/auth.ts`
- Create: `web/src/lib/utils.ts`
- Create: `web/src/lib/constants.ts`

- [ ] **Step 1: Utility fns**

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

- [ ] **Step 2: Auth cookie helpers**

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

- [ ] **Step 3: Axios API client**

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

- [ ] **Step 4: RBAC constants**

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

- [ ] **Step 1: Auth service**

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

- [ ] **Step 2: User service**

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

- [ ] **Step 3: Menu service**

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

- [ ] **Step 4: Ingredient service**

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

- [ ] **Step 5: Retail service**

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

- [ ] **Step 6: Promo service**

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

- [ ] **Step 7: Transaction service**

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

- [ ] **Step 8: Dashboard service**

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

- [ ] **Step 1: useDebounce hook**

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

- [ ] **Step 2: useAuth hook**

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

- [ ] **Step 3: AuthProvider**

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

- [ ] **Step 4: Login page**

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

- [ ] **Step 5: Next.js middleware for auth route protection**

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

- [ ] **Step 6: Update root layout w/ AuthProvider**

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

- [ ] **Step 7: Login page test**

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

- [ ] **Step 1: Sidebar items config**

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

- [ ] **Step 2: Sidebar component**

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

- [ ] **Step 3: Header component**

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

- [ ] **Step 4: Dashboard layout**

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

- [ ] **Step 5: Dashboard root redirect page**

Create `web/src/app/(dashboard)/page.tsx`:
```tsx
import { redirect } from 'next/navigation';

export default function DashboardRootPage() {
  redirect('/dashboard');
}
```

- [ ] **Step 6: Sidebar RBAC test**

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
- Create: `web/src/app/(dashboard)/dashboard/page.tsx` (partial — shell + summary)

- [ ] **Step 1: DashboardFilters component**

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

- [ ] **Step 2: SummaryCards component**

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

- [ ] **Step 3: SummaryCards test**

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

### Task 8: Dashboard — Charts (Sales Trend, Profit Category, Top Menus)

**Files:**
- Create: `web/src/components/dashboard/sales-trend-chart.tsx`
- Create: `web/src/components/dashboard/profit-category-chart.tsx`
- Create: `web/src/components/dashboard/top-profit-chart.tsx`
- Create: `web/src/components/dashboard/top-qty-chart.tsx`
- Create: `web/src/components/dashboard/scatter-chart.tsx`

- [ ] **Step 1: SalesTrendChart**

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

- [ ] **Step 2: ProfitCategoryChart (pie)**

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

- [ ] **Step 3: TopProfitChart (bar)**

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

- [ ] **Step 4: TopQtyChart (bar)**

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

- [ ] **Step 5: ScatterChart (laris vs untung)**

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

### Task 9: Dashboard — Promo Analytics & Menu Detail Table

**Files:**
- Create: `web/src/components/dashboard/promo-analytics.tsx`
- Create: `web/src/components/dashboard/menu-detail-table.tsx`

- [ ] **Step 1: PromoAnalytics component**

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

- [ ] **Step 2: MenuDetailTable component**

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

- [ ] **Step 1: RecipeEditor component**

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

- [ ] **Step 2: HppBreakdown component**

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

- [ ] **Step 3: MenuForm component**

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

- [ ] **Step 4: MenuTable component**

Create `web/src/components/menu/menu-table.tsx` — Full CRUD table w/ search, pagination, HPP dialog, edit/delete dropdown. Uses `MenuForm` + `HppBreakdown`. Shows margin alerts when < 30%. See backup for full code.

- [ ] **Step 5: Menu page**

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

- [ ] **Step 1: PriceHistory component**

Create `web/src/components/ingredients/price-history.tsx` — Table showing old→new price, date, % change. Uses `ingredientService.getPriceHistory()`. See backup for full code.

- [ ] **Step 2: AffectedMenus component**

Create `web/src/components/ingredients/affected-menus.tsx` — Table showing menus using this ingredient w/ qty. Uses `ingredientService.getAffectedMenus()`. See backup for full code.

- [ ] **Step 3: IngredientForm component**

Create `web/src/components/ingredients/ingredient-form.tsx` — Dialog form: name, unit, pricePerUnit, stockQty, minStock. Shows price change warning on edit. Uses `ingredientService`. See backup for full code.

- [ ] **Step 4: IngredientTable component**

Create `web/src/components/ingredients/ingredient-table.tsx` — Full CRUD table w/ search, pagination, detail dialog (tabs: price history + affected menus), low stock alerts. See backup for full code.

- [ ] **Step 5: Ingredients page**

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

- [ ] **Step 1: RetailForm component**

Create `web/src/components/retail/retail-form.tsx` — Dialog form: name, category, buyPrice, sellingPrice, stockQty, minStock, imageUrl. Shows live profit/margin calc. See backup for full code.

- [ ] **Step 2: RetailTable component**

Create `web/src/components/retail/retail-table.tsx` — Full CRUD table w/ search, pagination, profit/margin columns. See backup for full code.

- [ ] **Step 3: Retail page**

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

- [ ] **Step 1: PromoForm component**

Create `web/src/components/promos/promo-form.tsx` — Dialog form: name, type (PERCENT/FIXED/BUY_X_GET_Y/BUNDLE/MIN_PURCHASE), value, date range, maxUsage, isActive, promoItems (itemType/itemId/role). Loads menu+retail items for selection. See backup for full code.

- [ ] **Step 2: PromoTable component**

Create `web/src/components/promos/promo-table.tsx` — Full CRUD table w/ search, pagination, status badges (Aktif/Nonaktif/Belum Mulai/Berakhir/Max Usage). See backup for full code.

- [ ] **Step 3: Promos page**

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

- [ ] **Step 1: UserForm component**

Create `web/src/components/users/user-form.tsx` — Dialog form: name, email, password (optional on edit), role select, isActive switch. Loads roles via `userService.getRoles()`. See backup for full code.

- [ ] **Step 2: UserTable component**

Create `web/src/components/users/user-table.tsx` — Full CRUD table w/ search by name/email, pagination, role badge, active status. See backup for full code.

- [ ] **Step 3: Users page**

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

- [ ] **Step 1: TransactionDetail component**

Create `web/src/components/transactions/transaction-detail.tsx` — Shows txn code, date, cashier, payment method, items table (type/qty/price/HPP/subtotal), promo discount, totals (subtotal/discount/total/HPP/profit). See backup for full code.

- [ ] **Step 2: TransactionTable component**

Create `web/src/components/transactions/transaction-table.tsx` — Read-only table w/ date range + payment method filters, pagination, detail dialog. Columns: code, date, cashier, payment, total, profit, promo. See backup for full code.

- [ ] **Step 3: Transactions page**

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

- [ ] **Step 1: Verify all pages render**

```bash
cd ~/Downloads/PROJECT/web
npm run build
```

Expected: Build succeeds, no TS/import errors.

- [ ] **Step 2: Run tests**

```bash
cd ~/Downloads/PROJECT/web
npx jest --verbose
```

Expected: All component tests pass.

- [ ] **Step 3: Verify RBAC — Owner sees all**

Login as `owner@pos.com` / `password123` → all sidebar items visible: Dashboard, Menu, Bahan Baku, Retail, Promo, Users, Transaksi.

- [ ] **Step 4: Verify RBAC — Kitchen sees limited**

Login as `kitchen@pos.com` / `password123` → only Menu + Bahan Baku visible. Dashboard, Retail, Promo, Users, Transaksi NOT visible.

- [ ] **Step 5: Verify dashboard charts load**

Login as owner → Dashboard. Verify:
- 5 summary cards render (Total Penjualan, Total HPP, Gross Profit, Profit Margin, Jumlah Transaksi)
- Sales trend line chart renders
- Profit per category pie chart renders
- Top 10 profit bar chart renders
- Top 10 qty bar chart renders
- Scatter plot (laris vs untung) renders
- Promo analytics section renders
- Menu detail table renders w/ margin alerts

- [ ] **Step 6: Commit**

```bash
cd ~/Downloads/PROJECT/web
git init
git add .
git commit -m "feat: complete Phase 1 web dashboard with RBAC, analytics, and management pages"
```
