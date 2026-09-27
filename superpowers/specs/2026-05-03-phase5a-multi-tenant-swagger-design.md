# Phase 5A: Multi-tenant + Swagger/API Versioning — Design Spec

## Overview

Phase 5A transforms the single-tenant POS system into a multi-tenant SaaS platform and adds API documentation with versioning. This is a foundational change — all existing tables become tenant-scoped, auth flow gains tenant context, and a Prisma extension handles automatic tenant filtering.

Phase 5B (Loyalty Points + Table Management) will be built on top of this tenant-aware foundation.

## Goal

Production-ready multi-tenant POS that can be sold to multiple F&B businesses. Each tenant (outlet/restaurant) gets isolated data in a shared database, with configurable settings (logo, tax rate, timezone).

## Key Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Goal | Production-ready, sellable to F&B businesses |
| 2 | Target market | Single outlet → scale to multi-outlet chain |
| 3 | Tenant isolation | Shared DB, `tenant_id` column on all tenant-scoped tables |
| 4 | Swagger | Auto-generate from NestJS decorators |
| 5 | API versioning | URI-based (`/v1/`), default version 1 |

---

## 1. Multi-tenant Architecture

### New Database Models

```
tenants
+-- id (UUID, PK)
+-- name (varchar — "Warung Bahari")
+-- slug (varchar, unique — "warung-bahari")
+-- ownerUserId (FK -> users)
+-- isActive (boolean, default true)
+-- createdAt (timestamp)
+-- updatedAt (timestamp)

tenant_settings
+-- id (UUID, PK)
+-- tenantId (FK -> tenants, unique)
+-- logoUrl (varchar, nullable)
+-- primaryColor (varchar, nullable — hex)
+-- currencyFormat (varchar, default "IDR")
+-- taxRate (decimal, default 10 — PB1 percentage)
+-- timezone (varchar, default "Asia/Jakarta")
+-- updatedAt (timestamp)

user_tenants
+-- userId (FK -> users)
+-- tenantId (FK -> tenants)
+-- roleId (FK -> roles)
+-- isDefault (boolean)
+-- @@unique([userId, tenantId])

customer_tenants
+-- id (UUID, PK)
+-- customerId (FK -> customers)
+-- tenantId (FK -> tenants)
+-- createdAt (timestamp)
+-- @@unique([customerId, tenantId])
```

### Global vs Tenant-scoped Tables

**Global (no tenant_id):**
- `users` — global accounts, role per tenant via `user_tenants`
- `customers` — global accounts, association per tenant via `customer_tenants`
- `roles` — shared role definitions (OWNER, KASIR, KITCHEN, INVENTORY, FINANCE, MARKETING)
- `permissions` — shared permission definitions
- `role_permissions` — shared role-permission mappings
- `tenants` — tenant registry
- `tenant_settings` — per-tenant settings
- `user_tenants` — user-tenant junction
- `customer_tenants` — customer-tenant junction

**Tenant-scoped (gets tenant_id column):**
- `menu_items`, `recipes`, `ingredients`, `ingredient_price_logs`
- `retail_items`
- `orders`, `order_items`
- `transactions`, `transaction_items`, `transaction_promos`
- `promos`, `promo_items`
- `vouchers`, `voucher_usages`
- `deliveries`
- `cms_banners`, `cms_sections`, `cms_store_info`
- `warehouses`, `suppliers`
- `purchase_orders`, `purchase_order_items`
- `stock_movements`, `stock_opnames`, `stock_opname_items`
- `daily_closings`, `operational_costs`, `monthly_reports`
- `notifications` (staff notifications)
- `customer_otps`, `customer_devices`, `customer_notifications`

### User-Tenant Relationship

- 1 user can belong to multiple tenants (e.g., owner with 3 outlets)
- Each user has a role **per tenant** (could be OWNER in one, KASIR in another)
- Existing `users.role_id` is replaced by `user_tenants.role_id`
- On login, if user has >1 tenant, they select which tenant to use

### Customer-Tenant Relationship

- Customers are globally unique by phone number
- Each customer can be associated with multiple tenants (registers at different restaurants)
- Customer data (orders, voucher usages, notifications) is tenant-scoped

---

## 2. Migration Strategy

### Migration Steps

```
1. Create tenants, tenant_settings, user_tenants, customer_tenants tables
2. Create a "default" tenant record
3. Add tenant_id column (nullable) to ALL tenant-scoped tables
4. Backfill: UPDATE <table> SET tenant_id = <default_tenant_id> WHERE tenant_id IS NULL
5. ALTER tenant_id to NOT NULL + add FK constraint
6. Migrate users.role_id to user_tenants (move existing user-role to user_tenants with default tenant)
7. Migrate customers to customer_tenants
8. Update unique constraints to be tenant-scoped where needed
```

### Unique Constraints Update

```
vouchers.code (unique)  →  vouchers.(code, tenant_id) unique
```

Users and customers remain globally unique (email / phone). Tenant association via junction tables.

### cms_store_info Change

Was singleton (1 row globally) → becomes 1 row per tenant. The `tenant_id` makes it naturally scoped.

---

## 3. Tenant Resolution & Middleware

### TenantMiddleware

Extracts `tenantId` from the request and attaches it:

```
Request
  |
  +-- Staff request: tenantId from JWT payload
  |
  +-- Customer request: X-Tenant-Id header
  |
  +-- Public endpoints (login, register): no tenant needed
  |
  +-- Attach to request -> available via @CurrentTenant() decorator
```

### JWT Payload

```typescript
// Before (Phase 1-4):
{ sub: userId, email, role }

// After (Phase 5A):
{ sub: userId, email, tenantId, role }
```

### Login Flow Change

```
User logs in (email + password)
  |
  +-- Fetch user_tenants for this user
  |
  +-- 1 tenant? Auto-select, issue JWT with tenantId
  |
  +-- >1 tenants? Return tenant list
  |       +-- User calls POST /v1/tenants/select/:id
  |       +-- Issue JWT with selected tenantId
  |
  +-- 0 tenants? Return error (or prompt to create tenant)
```

### Prisma Tenant Extension

Per-request scoped Prisma client that auto-injects tenant_id:

```typescript
const prismaWithTenant = prisma.$extends({
  query: {
    $allModels: {
      async findMany({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.where = { ...args.where, tenantId };
        }
        return query(args);
      },
      async create({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.data = { ...args.data, tenantId };
        }
        return query(args);
      },
      async findFirst({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.where = { ...args.where, tenantId };
        }
        return query(args);
      },
      async update({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.where = { ...args.where, tenantId };
        }
        return query(args);
      },
      async delete({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.where = { ...args.where, tenantId };
        }
        return query(args);
      },
      async count({ args, query, model }) {
        if (TENANT_MODELS.includes(model)) {
          args.where = { ...args.where, tenantId };
        }
        return query(args);
      },
    },
  },
});
```

Injected via `REQUEST`-scoped provider. Each request gets a Prisma client pre-filtered to the active tenant.

### TENANT_MODELS List

```typescript
const TENANT_MODELS = [
  'MenuItem', 'Recipe', 'Ingredient', 'IngredientPriceLog',
  'RetailItem',
  'Order', 'OrderItem',
  'Transaction', 'TransactionItem', 'TransactionPromo',
  'Promo', 'PromoItem',
  'Voucher', 'VoucherUsage',
  'Delivery',
  'CmsBanner', 'CmsSection', 'CmsStoreInfo',
  'Warehouse', 'Supplier',
  'PurchaseOrder', 'PurchaseOrderItem',
  'StockMovement', 'StockOpname', 'StockOpnameItem',
  'DailyClosing', 'OperationalCost', 'MonthlyReport',
  'Notification',
  'CustomerOtp', 'CustomerDevice', 'CustomerNotification',
];
```

---

## 4. Tenant Management Module

### Backend Structure

```
backend/src/tenants/
+-- tenants.module.ts
+-- tenants.controller.ts
+-- tenants.service.ts
+-- tenants.service.spec.ts
+-- tenant.middleware.ts
+-- tenant-prisma.provider.ts      # REQUEST-scoped Prisma with tenant filter
+-- decorators/
|   +-- current-tenant.decorator.ts
+-- dto/
    +-- create-tenant.dto.ts
    +-- update-tenant-settings.dto.ts
```

### Endpoints

```
# Authenticated user (no tenant context needed)
POST   /v1/tenants                         # create new tenant
GET    /v1/tenants/my                      # list user's tenants
POST   /v1/tenants/select/:id              # switch tenant -> new JWT

# Tenant Owner (requires tenant context)
GET    /v1/tenants/current                 # current tenant info + settings
PATCH  /v1/tenants/current                 # update tenant name/slug
PATCH  /v1/tenants/current/settings        # update logo, color, tax, timezone

# Staff management
GET    /v1/tenants/current/members         # list members + roles
POST   /v1/tenants/current/invite          # invite user by email
DELETE /v1/tenants/current/members/:userId # remove member
PATCH  /v1/tenants/current/members/:userId/role  # change role
```

### Onboarding Flow

```
1. Owner registers or logs in
2. POST /v1/tenants { name: "My Restaurant", slug: "my-restaurant" }
3. Backend creates: tenant, tenant_settings (defaults), cms_store_info (defaults)
4. Owner added to user_tenants with OWNER role
5. JWT re-issued with tenantId
6. Ready to use — add menu, staff, etc.
```

---

## 5. Swagger & API Versioning

### Swagger Setup

Auto-generated from NestJS decorators at `/api/docs`.

```typescript
// main.ts
const config = new DocumentBuilder()
  .setTitle('POS System API')
  .setDescription('Multi-tenant POS with HPP Engine, Online Ordering, CMS')
  .setVersion('1.0')
  .addBearerAuth()
  .addApiKey({ type: 'apiKey', name: 'X-Tenant-Id', in: 'header' }, 'tenant')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

### Decorators

All controllers get:
- `@ApiTags('domain')` — groups endpoints
- `@ApiBearerAuth()` — marks as authenticated

All endpoints get:
- `@ApiOperation({ summary: '...' })` — description
- `@ApiResponse({ status, description })` — response docs

All DTOs get:
- `@ApiProperty({ description, example, required })` — field docs

### API Tags

| Tag | Description |
|-----|-------------|
| Auth | Staff login, customer OTP |
| Tenants | Tenant management, settings, members |
| Menu | Menu items CRUD |
| POS | Transactions |
| Orders | Kitchen order flow |
| Online Orders | Customer ordering |
| Vouchers | CRUD + validation |
| CMS | Banners, sections, store info |
| Deliveries | Driver management |
| Customers | Profile, addresses |
| Notifications | FCM, notification history |
| Uploads | Image upload |
| Inventory | Warehouses, suppliers, PO, stock |
| Finance | Closings, operational costs, reports |

### API Versioning

URI-based versioning with default version 1:

```typescript
app.enableVersioning({
  type: VersioningType.URI,
  defaultVersion: '1',
  prefix: 'v',
});
```

All existing routes move under `/v1/`:
- `/menu` -> `/v1/menu`
- `/customer-app/home` -> `/v1/customer-app/home`
- etc.

### Client Updates

Base URL changes in all clients:
- Flutter apps: `AppConfig.baseUrl` adds `/v1`
- Web dashboard: `api.ts` base URL adds `/v1`

---

## 6. Impact on Existing Code

### Services

All services that use `PrismaService` directly will switch to the REQUEST-scoped tenant-aware Prisma provider. The service code itself doesn't change — tenant filtering is handled by the Prisma extension layer.

### Auth Module

- `JwtStrategy.validate()` — extract `tenantId` from payload, verify user belongs to tenant
- Login response — include tenant list if >1
- JWT signing — include `tenantId` in payload

### Guards

- `JwtAuthGuard` — unchanged, JWT validation still works
- `PermissionsGuard` — role now comes from `user_tenants` instead of `users.role_id`
- `CustomerAuthGuard` — `tenantId` from `X-Tenant-Id` header, verify customer belongs to tenant

### WebSocket (Kitchen Display)

Socket.IO namespace becomes tenant-aware:
- Join room: `tenant:{tenantId}`
- Events scoped to tenant rooms — kitchen display only sees orders from its tenant

### Dashboard

- Add tenant selector (if user has >1 tenant)
- Add Tenant Settings page
- Sidebar shows current tenant name

### Flutter Apps

- Add `X-Tenant-Id` header to customer app requests
- Kasir app: tenant from JWT (selected at login)
- Customer app: tenant configured per deployment or selected on first launch

---

## 7. Complete New API Endpoints

```
# Tenant Management
POST   /v1/tenants
GET    /v1/tenants/my
POST   /v1/tenants/select/:id
GET    /v1/tenants/current
PATCH  /v1/tenants/current
PATCH  /v1/tenants/current/settings
GET    /v1/tenants/current/members
POST   /v1/tenants/current/invite
DELETE /v1/tenants/current/members/:userId
PATCH  /v1/tenants/current/members/:userId/role

# All existing endpoints prefixed with /v1/
# (40+ endpoints unchanged except URL prefix)
```

---

## 8. New Database Tables Summary

```
tenants
tenant_settings
user_tenants
customer_tenants
```

Modified: ALL tenant-scoped tables get `tenant_id` column + FK.

Removed: `users.role_id` column (replaced by `user_tenants.role_id`).
