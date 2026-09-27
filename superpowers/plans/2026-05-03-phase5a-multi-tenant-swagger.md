# Phase 5A: Multi-tenant + Swagger/API Versioning — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform single-tenant POS into multi-tenant SaaS with shared-DB tenant isolation, add Swagger API docs and URI-based versioning (`/v1/`).

**Architecture:** Add `tenant_id` to all business tables, create REQUEST-scoped Prisma extension that auto-filters by tenant. Auth JWT gains `tenantId` claim. Login flow adds tenant selection. Swagger auto-generated from NestJS decorators.

**Tech Stack:** NestJS 11, Prisma 7.8, @nestjs/swagger, PostgreSQL, Next.js 16, Flutter 3.9

**Spec:** `docs/superpowers/specs/2026-05-03-phase5a-multi-tenant-swagger-design.md`

---

## File Structure

### New Files

```
backend/src/tenants/
├── tenants.module.ts
├── tenants.controller.ts
├── tenants.service.ts
├── tenants.service.spec.ts
├── tenant.middleware.ts
├── tenant-prisma.provider.ts
├── tenant-models.const.ts
├── decorators/
│   └── current-tenant.decorator.ts
└── dto/
    ├── create-tenant.dto.ts
    └── update-tenant-settings.dto.ts
```

### Modified Files (Key)

```
backend/prisma/schema.prisma                    — new models + tenant_id on all tables
backend/src/main.ts                             — Swagger + versioning setup
backend/src/app.module.ts                       — TenantsModule + middleware
backend/src/auth/auth.service.ts                — login flow with tenant selection
backend/src/auth/strategies/jwt.strategy.ts     — validate tenantId in payload
backend/src/common/guards/permissions.guard.ts  — role from user_tenants
backend/src/customers/auth/customer-auth.service.ts — tenant from X-Tenant-Id
backend/src/customers/auth/customer-jwt.strategy.ts — tenant validation
backend/src/prisma/prisma.service.ts            — support tenant extension
backend/src/orders/orders.gateway.ts            — tenant-scoped rooms
backend/package.json                            — add @nestjs/swagger
All 32 controllers                              — @ApiTags, @ApiOperation, @Version('1')
web/src/lib/api.ts                              — baseURL /v1
web/src/lib/constants.ts                        — tenant nav items
customer-app/lib/config/app_config.dart         — baseURL /v1
customer-app/lib/core/api/api_client.dart       — X-Tenant-Id header
mobile/lib/config/app_config.dart               — baseURL /v1
```

---

## Task 1: Prisma Schema — New Tenant Models

**Files:**
- Modify: `pos/backend/prisma/schema.prisma`

- [ ] **Step 1: Add Tenant, TenantSettings, UserTenant, CustomerTenant models**

Add to schema.prisma:

```prisma
// ============================================
// Phase 5A: Multi-tenant
// ============================================

model Tenant {
  id          String   @id @default(uuid())
  name        String
  slug        String   @unique
  ownerUserId String   @map("owner_user_id")
  isActive    Boolean  @default(true) @map("is_active")
  createdAt   DateTime @default(now()) @map("created_at")
  updatedAt   DateTime @updatedAt @map("updated_at")

  owner    User           @relation("TenantOwner", fields: [ownerUserId], references: [id])
  settings TenantSettings?
  users    UserTenant[]
  customers CustomerTenant[]

  // Tenant-scoped relations
  menuItems       MenuItem[]
  ingredients     Ingredient[]
  retailItems     RetailItem[]
  orders          Order[]
  transactions    Transaction[]
  promos          Promo[]
  vouchers        Voucher[]
  deliveries      Delivery[]
  cmsBanners      CmsBanner[]
  cmsSections     CmsSection[]
  cmsStoreInfos   CmsStoreInfo[]
  warehouses      Warehouse[]
  suppliers       Supplier[]
  purchaseOrders  PurchaseOrder[]
  stockMovements  StockMovement[]
  stockOpnames    StockOpname[]
  dailyClosings   DailyClosing[]
  operationalCosts OperationalCost[]
  monthlyReports  MonthlyReport[]
  notifications   Notification[]
  customerOtps    CustomerOtp[]
  customerDevices CustomerDevice[]
  customerNotifications CustomerNotification[]

  @@map("tenants")
}

model TenantSettings {
  id             String   @id @default(uuid())
  tenantId       String   @unique @map("tenant_id")
  logoUrl        String?  @map("logo_url")
  primaryColor   String?  @map("primary_color")
  currencyFormat String   @default("IDR") @map("currency_format")
  taxRate        Decimal  @default(10) @map("tax_rate") @db.Decimal(5, 2)
  timezone       String   @default("Asia/Jakarta")
  updatedAt      DateTime @updatedAt @map("updated_at")

  tenant Tenant @relation(fields: [tenantId], references: [id])

  @@map("tenant_settings")
}

model UserTenant {
  id        String  @id @default(uuid())
  userId    String  @map("user_id")
  tenantId  String  @map("tenant_id")
  roleId    String  @map("role_id")
  isDefault Boolean @default(false) @map("is_default")

  user   User   @relation(fields: [userId], references: [id])
  tenant Tenant @relation(fields: [tenantId], references: [id])
  role   Role   @relation(fields: [roleId], references: [id])

  @@unique([userId, tenantId])
  @@map("user_tenants")
}

model CustomerTenant {
  id         String   @id @default(uuid())
  customerId String   @map("customer_id")
  tenantId   String   @map("tenant_id")
  createdAt  DateTime @default(now()) @map("created_at")

  customer Customer @relation(fields: [customerId], references: [id])
  tenant   Tenant   @relation(fields: [tenantId], references: [id])

  @@unique([customerId, tenantId])
  @@map("customer_tenants")
}
```

- [ ] **Step 2: Add `tenantId` field to ALL tenant-scoped models**

For every tenant-scoped model, add:

```prisma
tenantId String @map("tenant_id")
tenant   Tenant @relation(fields: [tenantId], references: [id])
```

Models to update (28 models): MenuItem, Recipe, Ingredient, IngredientPriceLog, RetailItem, Order, OrderItem, Transaction, TransactionItem, TransactionPromo, Promo, PromoItem, Voucher, VoucherUsage, Delivery, CmsBanner, CmsSection, CmsStoreInfo, Warehouse, Supplier, PurchaseOrder, PurchaseOrderItem, StockMovement, StockOpname, StockOpnameItem, DailyClosing, OperationalCost, MonthlyReport, Notification, CustomerOtp, CustomerDevice, CustomerNotification.

Note: Some child models (Recipe, OrderItem, TransactionItem, etc.) inherit tenant context from their parent. Add `tenantId` anyway for direct query filtering.

- [ ] **Step 3: Update User model — add tenant relations, keep roleId for migration compatibility**

```prisma
model User {
  // ... existing fields stay
  ownedTenants Tenant[]     @relation("TenantOwner")
  tenants      UserTenant[]
}
```

Add `UserTenant` relation to `Role` model too:
```prisma
model Role {
  // ... existing
  userTenants UserTenant[]
}
```

Add `CustomerTenant` relation to `Customer` model:
```prisma
model Customer {
  // ... existing
  tenants CustomerTenant[]
}
```

- [ ] **Step 4: Update voucher unique constraint**

Change voucher code from globally unique to unique per tenant:
```prisma
model Voucher {
  // ... remove @unique from code field
  code String  // was: @unique
  // ... add compound unique:
  @@unique([code, tenantId])
}
```

- [ ] **Step 5: Run migration**

Run: `cd pos/backend && npx prisma migrate dev --name phase5a-multi-tenant`

- [ ] **Step 6: Create data migration script**

Create `pos/backend/prisma/migrate-tenant.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // 1. Create default tenant
  const owner = await prisma.user.findFirst({ where: { role: { name: 'OWNER' } } });
  if (!owner) { console.log('No owner found, skip'); return; }

  const tenant = await prisma.tenant.upsert({
    where: { slug: 'default' },
    update: {},
    create: {
      name: 'Default Restaurant',
      slug: 'default',
      ownerUserId: owner.id,
    },
  });

  // 2. Create tenant settings
  await prisma.tenantSettings.upsert({
    where: { tenantId: tenant.id },
    update: {},
    create: { tenantId: tenant.id },
  });

  // 3. Backfill tenant_id on all tenant-scoped tables
  const tables = [
    'menu_items', 'recipes', 'ingredients', 'ingredient_price_logs',
    'retail_items', 'orders', 'order_items',
    'transactions', 'transaction_items', 'transaction_promos',
    'promos', 'promo_items', 'vouchers', 'voucher_usages',
    'deliveries', 'cms_banners', 'cms_sections', 'cms_store_info',
    'warehouses', 'suppliers', 'purchase_orders', 'purchase_order_items',
    'stock_movements', 'stock_opnames', 'stock_opname_items',
    'daily_closings', 'operational_costs', 'monthly_reports',
    'notifications', 'customer_otps', 'customer_devices', 'customer_notifications',
  ];

  for (const table of tables) {
    await prisma.$executeRawUnsafe(
      `UPDATE "${table}" SET tenant_id = $1 WHERE tenant_id IS NULL`, tenant.id
    );
    console.log(`Backfilled ${table}`);
  }

  // 4. Migrate user roles to user_tenants
  const users = await prisma.user.findMany({ where: { roleId: { not: null } } });
  for (const user of users) {
    await prisma.userTenant.upsert({
      where: { userId_tenantId: { userId: user.id, tenantId: tenant.id } },
      update: {},
      create: {
        userId: user.id,
        tenantId: tenant.id,
        roleId: user.roleId!,
        isDefault: true,
      },
    });
  }

  // 5. Migrate customers to customer_tenants
  const customers = await prisma.customer.findMany();
  for (const customer of customers) {
    await prisma.customerTenant.upsert({
      where: { customerId_tenantId: { customerId: customer.id, tenantId: tenant.id } },
      update: {},
      create: { customerId: customer.id, tenantId: tenant.id },
    });
  }

  console.log('Tenant migration complete');
}

main().catch(console.error).finally(() => prisma.$disconnect());
```

- [ ] **Step 7: Run data migration**

Run: `cd pos/backend && npx ts-node prisma/migrate-tenant.ts`

- [ ] **Step 8: Run prisma generate**

Run: `cd pos/backend && npx prisma generate`

- [ ] **Step 9: Commit**

```bash
cd pos/backend
git add prisma/
git commit -m "feat(phase5a): add multi-tenant schema — tenant models + tenant_id on all business tables"
```

---

## Task 2: Tenant-Aware Prisma Provider

**Files:**
- Create: `pos/backend/src/tenants/tenant-models.const.ts`
- Create: `pos/backend/src/tenants/tenant-prisma.provider.ts`
- Create: `pos/backend/src/tenants/decorators/current-tenant.decorator.ts`

- [ ] **Step 1: Create tenant models constant**

`pos/backend/src/tenants/tenant-models.const.ts`:

```typescript
export const TENANT_MODELS = [
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
] as const;
```

- [ ] **Step 2: Create @CurrentTenant() decorator**

`pos/backend/src/tenants/decorators/current-tenant.decorator.ts`:

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentTenant = createParamDecorator(
  (_data: unknown, ctx: ExecutionContext): string => {
    const request = ctx.switchToHttp().getRequest();
    return request.tenantId;
  },
);
```

- [ ] **Step 3: Create tenant-aware Prisma provider**

`pos/backend/src/tenants/tenant-prisma.provider.ts`:

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import { PrismaService } from '../prisma/prisma.service';
import { TENANT_MODELS } from './tenant-models.const';

export const TENANT_PRISMA = 'TENANT_PRISMA';

@Injectable({ scope: Scope.REQUEST })
export class TenantPrismaProvider {
  public readonly client: any;

  constructor(
    private prisma: PrismaService,
    @Inject(REQUEST) private request: Request,
  ) {
    const tenantId = (request as any).tenantId;

    if (!tenantId) {
      // No tenant context — return raw Prisma (for global operations)
      this.client = prisma;
      return;
    }

    this.client = prisma.$extends({
      query: {
        $allModels: {
          async findMany({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.where = { ...args.where, tenantId };
            }
            return query(args);
          },
          async findFirst({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.where = { ...args.where, tenantId };
            }
            return query(args);
          },
          async findUnique({ model, args, query }) {
            // findUnique only works with unique fields, add tenantId check after
            return query(args);
          },
          async create({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.data = { ...args.data, tenantId };
            }
            return query(args);
          },
          async update({ model, args, query }) {
            return query(args);
          },
          async delete({ model, args, query }) {
            return query(args);
          },
          async count({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.where = { ...args.where, tenantId };
            }
            return query(args);
          },
          async aggregate({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.where = { ...args.where, tenantId };
            }
            return query(args);
          },
          async groupBy({ model, args, query }) {
            if (TENANT_MODELS.includes(model as any)) {
              args.where = { ...args.where, tenantId };
            }
            return query(args);
          },
        },
      },
    });
  }
}
```

- [ ] **Step 4: Commit**

```bash
cd pos/backend
git add src/tenants/
git commit -m "feat(phase5a): add tenant-aware Prisma provider with auto-filtering"
```

---

## Task 3: Tenant Middleware + Module Shell

**Files:**
- Create: `pos/backend/src/tenants/tenant.middleware.ts`
- Create: `pos/backend/src/tenants/tenants.module.ts`
- Modify: `pos/backend/src/app.module.ts`

- [ ] **Step 1: Create TenantMiddleware**

`pos/backend/src/tenants/tenant.middleware.ts`:

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class TenantMiddleware implements NestMiddleware {
  constructor(private jwtService: JwtService) {}

  use(req: Request, _res: Response, next: NextFunction) {
    // 1. Try JWT payload (staff requests)
    const authHeader = req.headers.authorization;
    if (authHeader?.startsWith('Bearer ')) {
      try {
        const token = authHeader.substring(7);
        const payload = this.jwtService.verify(token);
        if (payload.tenantId) {
          (req as any).tenantId = payload.tenantId;
          next();
          return;
        }
      } catch {
        // Invalid token — let auth guard handle it
      }
    }

    // 2. Try X-Tenant-Id header (customer app requests)
    const headerTenantId = req.headers['x-tenant-id'] as string;
    if (headerTenantId) {
      (req as any).tenantId = headerTenantId;
    }

    next();
  }
}
```

- [ ] **Step 2: Create TenantsModule shell**

`pos/backend/src/tenants/tenants.module.ts`:

```typescript
import { Module, Global, MiddlewareConsumer, NestModule } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PrismaModule } from '../prisma/prisma.module';
import { TenantMiddleware } from './tenant.middleware';
import { TenantPrismaProvider, TENANT_PRISMA } from './tenant-prisma.provider';

@Global()
@Module({
  imports: [
    PrismaModule,
    JwtModule.register({ secret: process.env.JWT_SECRET || 'change-me-in-production' }),
  ],
  providers: [
    { provide: TENANT_PRISMA, useClass: TenantPrismaProvider },
  ],
  exports: [TENANT_PRISMA],
})
export class TenantsModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(TenantMiddleware).forRoutes('*');
  }
}
```

- [ ] **Step 3: Register TenantsModule in AppModule**

Add `TenantsModule` to imports in `pos/backend/src/app.module.ts`. Place it early in the imports array (before feature modules).

- [ ] **Step 4: Commit**

```bash
cd pos/backend
git add src/tenants/ src/app.module.ts
git commit -m "feat(phase5a): add tenant middleware and global tenant module"
```

---

## Task 4: Update Auth — JWT Payload + Login Flow

**Files:**
- Modify: `pos/backend/src/auth/auth.service.ts`
- Modify: `pos/backend/src/auth/auth.service.spec.ts`
- Modify: `pos/backend/src/auth/strategies/jwt.strategy.ts`
- Modify: `pos/backend/src/common/guards/permissions.guard.ts`

- [ ] **Step 1: Write failing tests for new login flow**

Update `auth.service.spec.ts` to test:
1. Login with single tenant → JWT includes tenantId
2. Login with multiple tenants → returns tenant list, no JWT yet
3. Login with 0 tenants → returns error or empty tenant list

```typescript
describe('login (multi-tenant)', () => {
  it('should auto-select tenant and include tenantId in JWT when user has 1 tenant', async () => {
    prisma.user.findUnique.mockResolvedValue({ id: '1', email: 'test@test.com', password: hashedPw, isActive: true, role: { name: 'OWNER' } });
    prisma.userTenant.findMany.mockResolvedValue([
      { tenantId: 't1', roleId: 'r1', role: { name: 'OWNER' }, tenant: { id: 't1', name: 'My Shop', isActive: true } },
    ]);

    const result = await service.login({ email: 'test@test.com', password: 'password' });

    expect(jwtService.sign).toHaveBeenCalledWith(
      expect.objectContaining({ tenantId: 't1', role: 'OWNER' })
    );
    expect(result.accessToken).toBeDefined();
  });

  it('should return tenant list when user has multiple tenants', async () => {
    prisma.user.findUnique.mockResolvedValue({ id: '1', email: 'test@test.com', password: hashedPw, isActive: true, role: { name: 'OWNER' } });
    prisma.userTenant.findMany.mockResolvedValue([
      { tenantId: 't1', role: { name: 'OWNER' }, tenant: { id: 't1', name: 'Shop A', isActive: true } },
      { tenantId: 't2', role: { name: 'KASIR' }, tenant: { id: 't2', name: 'Shop B', isActive: true } },
    ]);

    const result = await service.login({ email: 'test@test.com', password: 'password' });

    expect(result.tenants).toHaveLength(2);
    expect(result.accessToken).toBeUndefined();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd pos/backend && npx jest --testPathPattern=auth.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 3: Update AuthService login flow**

In `pos/backend/src/auth/auth.service.ts`, update `login()`:

```typescript
async login(dto: LoginDto) {
  const user = await this.prisma.user.findUnique({
    where: { email: dto.email },
    include: { role: true },
  });

  if (!user || !user.isActive) throw new UnauthorizedException('Invalid credentials');

  const passwordValid = await bcrypt.compare(dto.password, user.password);
  if (!passwordValid) throw new UnauthorizedException('Invalid credentials');

  // Fetch user's tenants
  const userTenants = await this.prisma.userTenant.findMany({
    where: { userId: user.id },
    include: { role: true, tenant: { select: { id: true, name: true, slug: true, isActive: true } } },
  });

  const activeTenants = userTenants.filter(ut => ut.tenant.isActive);

  if (activeTenants.length === 0) {
    // No tenants — return user info without token (needs to create/join tenant)
    return {
      user: { id: user.id, name: user.name, email: user.email },
      tenants: [],
      message: 'No active tenants. Create or join a tenant.',
    };
  }

  if (activeTenants.length === 1) {
    // Auto-select single tenant
    const ut = activeTenants[0];
    const payload = { sub: user.id, email: user.email, tenantId: ut.tenant.id, role: ut.role.name };

    return {
      accessToken: this.jwtService.sign(payload),
      user: { id: user.id, name: user.name, email: user.email },
      tenant: ut.tenant,
      role: ut.role.name,
    };
  }

  // Multiple tenants — return list, user must select
  return {
    user: { id: user.id, name: user.name, email: user.email },
    tenants: activeTenants.map(ut => ({
      id: ut.tenant.id, name: ut.tenant.name, slug: ut.tenant.slug, role: ut.role.name,
    })),
    message: 'Select a tenant to continue.',
  };
}

async selectTenant(userId: string, tenantId: string) {
  const userTenant = await this.prisma.userTenant.findUnique({
    where: { userId_tenantId: { userId, tenantId } },
    include: { role: true, tenant: true },
  });

  if (!userTenant || !userTenant.tenant.isActive) {
    throw new UnauthorizedException('Tenant not found');
  }

  const user = await this.prisma.user.findUnique({ where: { id: userId } });
  const payload = { sub: userId, email: user!.email, tenantId, role: userTenant.role.name };

  return {
    accessToken: this.jwtService.sign(payload),
    tenant: userTenant.tenant,
    role: userTenant.role.name,
  };
}
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=auth.service.spec --no-coverage`

Expected: PASS

- [ ] **Step 5: Update JwtStrategy to include tenantId**

In `pos/backend/src/auth/strategies/jwt.strategy.ts`, update `validate()`:

```typescript
async validate(payload: { sub: string; email: string; tenantId?: string; role: string }) {
  const user = await this.prisma.user.findUnique({
    where: { id: payload.sub },
  });

  if (!user || !user.isActive) throw new UnauthorizedException();

  // If tenantId present, verify user belongs to tenant
  let roleName = payload.role;
  if (payload.tenantId) {
    const userTenant = await this.prisma.userTenant.findUnique({
      where: { userId_tenantId: { userId: user.id, tenantId: payload.tenantId } },
      include: { role: true },
    });
    if (!userTenant) throw new UnauthorizedException();
    roleName = userTenant.role.name;
  }

  return {
    id: user.id,
    email: user.email,
    name: user.name,
    tenantId: payload.tenantId,
    roleName,
  };
}
```

- [ ] **Step 6: Update PermissionsGuard — role from user_tenants**

In `pos/backend/src/common/guards/permissions.guard.ts`, update to get role from the validated user object (which now includes `roleName` from `user_tenants`):

```typescript
async canActivate(context: ExecutionContext): Promise<boolean> {
  const requiredPermissions = this.reflector.getAllAndOverride<string[]>('permissions', [
    context.getHandler(), context.getClass(),
  ]);

  if (!requiredPermissions) return true;

  const request = context.switchToHttp().getRequest();
  const user = request.user;
  if (!user) return false;

  // Get role permissions — use roleName from JWT validation
  const role = await this.prisma.role.findUnique({
    where: { name: user.roleName },
    include: { permissions: { include: { permission: true } } },
  });

  if (!role) return false;

  const userPermissions = role.permissions.map(rp => rp.permission.name);
  return requiredPermissions.every(perm => userPermissions.includes(perm));
}
```

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add src/auth/ src/common/guards/
git commit -m "feat(phase5a): update auth flow with tenant selection and tenant-aware JWT"
```

---

## Task 5: Update Customer Auth — Tenant from Header

**Files:**
- Modify: `pos/backend/src/customers/auth/customer-auth.service.ts`
- Modify: `pos/backend/src/customers/auth/customer-jwt.strategy.ts`

- [ ] **Step 1: Update CustomerAuthService to require tenantId**

In `verifyOtp()`, include `tenantId` from the request context:

```typescript
async verifyOtp(phone: string, code: string, tenantId: string) {
  // ... existing OTP verification logic ...

  let customer = await this.prisma.customer.findUnique({ where: { phone } });
  if (!customer) {
    customer = await this.prisma.customer.create({ data: { phone } });
  }

  // Ensure customer-tenant association
  await this.prisma.customerTenant.upsert({
    where: { customerId_tenantId: { customerId: customer.id, tenantId } },
    update: {},
    create: { customerId: customer.id, tenantId },
  });

  const payload = { sub: customer.id, phone: customer.phone, role: 'CUSTOMER', tenantId };

  return {
    accessToken: this.jwtService.sign(payload, { expiresIn: '1h' }),
    refreshToken: this.jwtService.sign(payload, { expiresIn: '30d' }),
    customer: { id: customer.id, name: customer.name, phone: customer.phone, email: customer.email },
  };
}
```

Update controller to pass tenantId from request:
```typescript
@Post('otp/verify')
verifyOtp(@Body() dto: VerifyOtpDto, @Request() req: any) {
  return this.authService.verifyOtp(dto.phone, dto.code, req.tenantId);
}
```

- [ ] **Step 2: Update CustomerJwtStrategy**

```typescript
async validate(payload: { sub: string; phone: string; role: string; tenantId?: string }) {
  if (payload.role !== 'CUSTOMER') throw new UnauthorizedException();
  const customer = await this.prisma.customer.findUnique({ where: { id: payload.sub } });
  if (!customer) throw new UnauthorizedException();

  return { ...customer, tenantId: payload.tenantId };
}
```

- [ ] **Step 3: Update tests and commit**

Update customer-auth tests to mock tenantId. Run tests.

```bash
cd pos/backend
git add src/customers/
git commit -m "feat(phase5a): update customer auth with tenant context from X-Tenant-Id header"
```

---

## Task 6: Tenant Management Service + Controller

**Files:**
- Create: `pos/backend/src/tenants/tenants.service.ts`
- Create: `pos/backend/src/tenants/tenants.service.spec.ts`
- Create: `pos/backend/src/tenants/tenants.controller.ts`
- Create: `pos/backend/src/tenants/dto/create-tenant.dto.ts`
- Create: `pos/backend/src/tenants/dto/update-tenant-settings.dto.ts`
- Modify: `pos/backend/src/tenants/tenants.module.ts`

- [ ] **Step 1: Create DTOs**

`create-tenant.dto.ts`:
```typescript
import { IsString, Matches, MinLength } from 'class-validator';

export class CreateTenantDto {
  @IsString() @MinLength(2) name: string;
  @IsString() @Matches(/^[a-z0-9-]+$/, { message: 'Slug must be lowercase alphanumeric with hyphens' })
  slug: string;
}
```

`update-tenant-settings.dto.ts`:
```typescript
import { IsString, IsOptional, IsNumber, Min, Max } from 'class-validator';

export class UpdateTenantSettingsDto {
  @IsString() @IsOptional() logoUrl?: string;
  @IsString() @IsOptional() primaryColor?: string;
  @IsString() @IsOptional() currencyFormat?: string;
  @IsNumber() @Min(0) @Max(100) @IsOptional() taxRate?: number;
  @IsString() @IsOptional() timezone?: string;
}
```

- [ ] **Step 2: Write failing tests for TenantsService**

`pos/backend/src/tenants/tenants.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { TenantsService } from './tenants.service';
import { PrismaService } from '../prisma/prisma.service';
import { JwtService } from '@nestjs/jwt';
import { ConflictException } from '@nestjs/common';

describe('TenantsService', () => {
  let service: TenantsService;
  let prisma: any;
  let jwtService: any;

  beforeEach(async () => {
    prisma = {
      tenant: { create: jest.fn(), findUnique: jest.fn(), findMany: jest.fn(), update: jest.fn() },
      tenantSettings: { create: jest.fn(), findUnique: jest.fn(), update: jest.fn() },
      userTenant: { create: jest.fn(), findMany: jest.fn(), findUnique: jest.fn(), delete: jest.fn(), update: jest.fn() },
      cmsStoreInfo: { create: jest.fn() },
      role: { findUnique: jest.fn() },
      user: { findUnique: jest.fn() },
      $transaction: jest.fn((cb) => cb(prisma)),
    };
    jwtService = { sign: jest.fn().mockReturnValue('mock-token') };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        TenantsService,
        { provide: PrismaService, useValue: prisma },
        { provide: JwtService, useValue: jwtService },
      ],
    }).compile();

    service = module.get(TenantsService);
  });

  describe('createTenant', () => {
    it('should create tenant with settings and owner user_tenant', async () => {
      prisma.tenant.findUnique.mockResolvedValue(null);
      prisma.role.findUnique.mockResolvedValue({ id: 'role-owner' });
      prisma.tenant.create.mockResolvedValue({ id: 't1', name: 'Test', slug: 'test' });
      prisma.tenantSettings.create.mockResolvedValue({});
      prisma.cmsStoreInfo.create.mockResolvedValue({});
      prisma.userTenant.create.mockResolvedValue({});
      prisma.user.findUnique.mockResolvedValue({ id: 'u1', email: 'test@test.com' });

      const result = await service.createTenant('u1', { name: 'Test', slug: 'test' });

      expect(prisma.tenant.create).toHaveBeenCalled();
      expect(prisma.tenantSettings.create).toHaveBeenCalled();
      expect(prisma.userTenant.create).toHaveBeenCalled();
      expect(result).toHaveProperty('accessToken');
    });

    it('should reject duplicate slug', async () => {
      prisma.tenant.findUnique.mockResolvedValue({ id: 'existing' });

      await expect(service.createTenant('u1', { name: 'Test', slug: 'test' }))
        .rejects.toThrow(ConflictException);
    });
  });

  describe('getMyTenants', () => {
    it('should return tenants for user', async () => {
      prisma.userTenant.findMany.mockResolvedValue([
        { tenant: { id: 't1', name: 'Shop A' }, role: { name: 'OWNER' } },
      ]);

      const result = await service.getMyTenants('u1');
      expect(result).toHaveLength(1);
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=tenants.service.spec --no-coverage`

- [ ] **Step 4: Implement TenantsService**

`pos/backend/src/tenants/tenants.service.ts`:

```typescript
import { Injectable, ConflictException, NotFoundException, ForbiddenException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { JwtService } from '@nestjs/jwt';
import { CreateTenantDto } from './dto/create-tenant.dto';
import { UpdateTenantSettingsDto } from './dto/update-tenant-settings.dto';

@Injectable()
export class TenantsService {
  constructor(private prisma: PrismaService, private jwtService: JwtService) {}

  async createTenant(userId: string, dto: CreateTenantDto) {
    const existing = await this.prisma.tenant.findUnique({ where: { slug: dto.slug } });
    if (existing) throw new ConflictException('Slug sudah digunakan');

    const ownerRole = await this.prisma.role.findUnique({ where: { name: 'OWNER' } });

    return this.prisma.$transaction(async (tx) => {
      const tenant = await tx.tenant.create({
        data: { name: dto.name, slug: dto.slug, ownerUserId: userId },
      });

      await tx.tenantSettings.create({ data: { tenantId: tenant.id } });

      await tx.cmsStoreInfo.create({
        data: {
          tenantId: tenant.id,
          storeName: dto.name,
          address: '',
          phone: '',
          operatingHours: { mon: '08:00-22:00', tue: '08:00-22:00', wed: '08:00-22:00', thu: '08:00-22:00', fri: '08:00-22:00', sat: '09:00-23:00', sun: '09:00-22:00' },
        },
      });

      await tx.userTenant.create({
        data: { userId, tenantId: tenant.id, roleId: ownerRole!.id, isDefault: true },
      });

      const user = await tx.user.findUnique({ where: { id: userId } });
      const payload = { sub: userId, email: user!.email, tenantId: tenant.id, role: 'OWNER' };

      return { tenant, accessToken: this.jwtService.sign(payload) };
    });
  }

  async getMyTenants(userId: string) {
    const userTenants = await this.prisma.userTenant.findMany({
      where: { userId },
      include: { tenant: { select: { id: true, name: true, slug: true, isActive: true } }, role: { select: { name: true } } },
    });
    return userTenants.map(ut => ({ ...ut.tenant, role: ut.role.name }));
  }

  async getCurrentTenant(tenantId: string) {
    const tenant = await this.prisma.tenant.findUnique({
      where: { id: tenantId },
      include: { settings: true },
    });
    if (!tenant) throw new NotFoundException('Tenant not found');
    return tenant;
  }

  async updateTenant(tenantId: string, data: { name?: string; slug?: string }) {
    if (data.slug) {
      const existing = await this.prisma.tenant.findUnique({ where: { slug: data.slug } });
      if (existing && existing.id !== tenantId) throw new ConflictException('Slug sudah digunakan');
    }
    return this.prisma.tenant.update({ where: { id: tenantId }, data });
  }

  async updateSettings(tenantId: string, dto: UpdateTenantSettingsDto) {
    return this.prisma.tenantSettings.update({ where: { tenantId }, data: dto as any });
  }

  async getMembers(tenantId: string) {
    return this.prisma.userTenant.findMany({
      where: { tenantId },
      include: { user: { select: { id: true, name: true, email: true } }, role: { select: { id: true, name: true } } },
    });
  }

  async inviteMember(tenantId: string, email: string, roleId: string) {
    const user = await this.prisma.user.findUnique({ where: { email } });
    if (!user) throw new NotFoundException('User tidak ditemukan');

    const existing = await this.prisma.userTenant.findUnique({
      where: { userId_tenantId: { userId: user.id, tenantId } },
    });
    if (existing) throw new ConflictException('User sudah menjadi member');

    return this.prisma.userTenant.create({ data: { userId: user.id, tenantId, roleId } });
  }

  async removeMember(tenantId: string, userId: string, requesterId: string) {
    if (userId === requesterId) throw new ForbiddenException('Tidak bisa menghapus diri sendiri');
    return this.prisma.userTenant.delete({ where: { userId_tenantId: { userId, tenantId } } });
  }

  async changeMemberRole(tenantId: string, userId: string, roleId: string) {
    return this.prisma.userTenant.update({
      where: { userId_tenantId: { userId, tenantId } },
      data: { roleId },
    });
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=tenants.service.spec --no-coverage`

- [ ] **Step 6: Create TenantsController**

`pos/backend/src/tenants/tenants.controller.ts`:

```typescript
import { Controller, Get, Post, Patch, Delete, Body, Param, UseGuards, Request } from '@nestjs/common';
import { TenantsService } from './tenants.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';
import { CreateTenantDto } from './dto/create-tenant.dto';
import { UpdateTenantSettingsDto } from './dto/update-tenant-settings.dto';
import { AuthService } from '../auth/auth.service';

@Controller({ path: 'tenants', version: '1' })
export class TenantsController {
  constructor(
    private tenantsService: TenantsService,
    private authService: AuthService,
  ) {}

  @Post()
  @UseGuards(JwtAuthGuard)
  create(@Request() req: any, @Body() dto: CreateTenantDto) {
    return this.tenantsService.createTenant(req.user.id, dto);
  }

  @Get('my')
  @UseGuards(JwtAuthGuard)
  getMyTenants(@Request() req: any) {
    return this.tenantsService.getMyTenants(req.user.id);
  }

  @Post('select/:id')
  @UseGuards(JwtAuthGuard)
  selectTenant(@Request() req: any, @Param('id') tenantId: string) {
    return this.authService.selectTenant(req.user.id, tenantId);
  }

  @Get('current')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  getCurrent(@Request() req: any) {
    return this.tenantsService.getCurrentTenant(req.user.tenantId);
  }

  @Patch('current')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  updateCurrent(@Request() req: any, @Body() dto: { name?: string; slug?: string }) {
    return this.tenantsService.updateTenant(req.user.tenantId, dto);
  }

  @Patch('current/settings')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  updateSettings(@Request() req: any, @Body() dto: UpdateTenantSettingsDto) {
    return this.tenantsService.updateSettings(req.user.tenantId, dto);
  }

  @Get('current/members')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  getMembers(@Request() req: any) {
    return this.tenantsService.getMembers(req.user.tenantId);
  }

  @Post('current/invite')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  invite(@Request() req: any, @Body() dto: { email: string; roleId: string }) {
    return this.tenantsService.inviteMember(req.user.tenantId, dto.email, dto.roleId);
  }

  @Delete('current/members/:userId')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  removeMember(@Request() req: any, @Param('userId') userId: string) {
    return this.tenantsService.removeMember(req.user.tenantId, userId, req.user.id);
  }

  @Patch('current/members/:userId/role')
  @UseGuards(JwtAuthGuard, PermissionsGuard)
  @RequirePermissions('user:manage')
  changeRole(@Request() req: any, @Param('userId') userId: string, @Body() dto: { roleId: string }) {
    return this.tenantsService.changeMemberRole(req.user.tenantId, userId, dto.roleId);
  }
}
```

- [ ] **Step 7: Update TenantsModule with service + controller**

Add `TenantsService`, `TenantsController` to the module. Import `AuthModule` for `AuthService`.

- [ ] **Step 8: Commit**

```bash
cd pos/backend
git add src/tenants/
git commit -m "feat(phase5a): add tenant management — create, select, members, settings"
```

---

## Task 7: Swagger + API Versioning Setup

**Files:**
- Modify: `pos/backend/src/main.ts`
- Modify: `pos/backend/package.json`
- Modify: ALL 32 controllers (add `@ApiTags`, `version: '1'`)

- [ ] **Step 1: Install @nestjs/swagger**

Run: `cd pos/backend && npm install @nestjs/swagger`

- [ ] **Step 2: Update main.ts — Swagger + Versioning**

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe, VersioningType } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // API Versioning
  app.enableVersioning({
    type: VersioningType.URI,
    defaultVersion: '1',
  });

  // Global pipes, filters, interceptors (existing)
  app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
  // ... existing filters and interceptors

  // Swagger
  const config = new DocumentBuilder()
    .setTitle('POS System API')
    .setDescription('Multi-tenant POS with HPP Engine, Online Ordering, CMS')
    .setVersion('1.0')
    .addBearerAuth()
    .addApiKey({ type: 'apiKey', name: 'X-Tenant-Id', in: 'header' }, 'tenant')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document);

  // CORS (existing)
  app.enableCors({ origin: ['http://localhost:3001', 'http://localhost:8080'], credentials: true });

  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```

- [ ] **Step 3: Add version to ALL controllers**

Every controller changes from:
```typescript
@Controller('menu')
```
to:
```typescript
@Controller({ path: 'menu', version: '1' })
@ApiTags('Menu')
```

Apply to all 32 controllers. Add `@ApiTags` with appropriate tag name per controller.

- [ ] **Step 4: Add @ApiProperty to key DTOs**

Add Swagger decorators to the most important DTOs (auth, tenant, order creation). Example:

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class LoginDto {
  @ApiProperty({ example: 'owner@test.com' }) @IsEmail() email: string;
  @ApiProperty({ example: 'password123' }) @IsString() password: string;
}
```

- [ ] **Step 5: Verify Swagger UI loads**

Run: `cd pos/backend && npm run start:dev`

Open: `http://localhost:3000/api/docs`

Expected: Swagger UI with all endpoints grouped by tags.

- [ ] **Step 6: Commit**

```bash
cd pos/backend
git add src/ package.json package-lock.json
git commit -m "feat(phase5a): add Swagger API docs and URI-based versioning (/v1/)"
```

---

## Task 8: Update WebSocket — Tenant Rooms

**Files:**
- Modify: `pos/backend/src/orders/orders.gateway.ts`

- [ ] **Step 1: Update OrdersGateway for tenant-scoped rooms**

```typescript
@WebSocketGateway({ cors: { origin: '*' }, namespace: '/orders' })
export class OrdersGateway {
  @WebSocketServer() server: Server;

  handleConnection(client: Socket) {
    const tenantId = client.handshake.query.tenantId as string;
    if (tenantId) {
      client.join(`tenant:${tenantId}`);
    }
  }

  emitNewOrder(order: any) {
    const tenantId = order.tenantId;
    if (tenantId) {
      this.server.to(`tenant:${tenantId}`).emit('order:new', order);
    } else {
      this.server.emit('order:new', order);
    }
  }

  // Update other emit methods similarly
}
```

- [ ] **Step 2: Commit**

```bash
cd pos/backend
git add src/orders/orders.gateway.ts
git commit -m "feat(phase5a): scope WebSocket events to tenant rooms"
```

---

## Task 9: Update Client Apps — Base URL + Tenant Header

**Files:**
- Modify: `pos/web/src/lib/api.ts`
- Modify: `pos/customer-app/lib/config/app_config.dart`
- Modify: `pos/customer-app/lib/core/api/api_client.dart`
- Modify: `pos/mobile/lib/config/app_config.dart`

- [ ] **Step 1: Update web dashboard base URL**

In `pos/web/src/lib/api.ts`, change base URL:

```typescript
const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL + '/v1' || 'http://localhost:3000/v1',
});
```

- [ ] **Step 2: Update customer app base URL + tenant header**

`pos/customer-app/lib/config/app_config.dart`:
```dart
class AppConfig {
  static const String baseUrl = 'http://localhost:3000/v1';
}
```

`pos/customer-app/lib/core/api/api_client.dart` — add X-Tenant-Id header:
```dart
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) async {
    final token = await _tokenStorage.getAccessToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    // Add tenant header
    final tenantId = await _tokenStorage.getTenantId();
    if (tenantId != null) {
      options.headers['X-Tenant-Id'] = tenantId;
    }
    handler.next(options);
  },
));
```

Add `getTenantId()`/`saveTenantId()` to TokenStorage.

- [ ] **Step 3: Update kasir app base URL**

`pos/mobile/lib/config/app_config.dart`:
```dart
static const String baseUrl = 'http://localhost:3000/v1';
```

- [ ] **Step 4: Commit**

```bash
cd pos
git add web/src/lib/api.ts customer-app/lib/ mobile/lib/config/
git commit -m "feat(phase5a): update all client apps with /v1 base URL and tenant headers"
```

---

## Task 10: Web Dashboard — Tenant UI

**Files:**
- Create: `pos/web/src/services/tenant.service.ts`
- Create: `pos/web/src/types/tenant.ts`
- Create: `pos/web/src/app/(dashboard)/tenant-settings/page.tsx`
- Modify: `pos/web/src/providers/auth-provider.tsx` — tenant selection flow
- Modify: `pos/web/src/lib/constants.ts` — tenant nav items

- [ ] **Step 1: Create tenant types and service**

`pos/web/src/types/tenant.ts`:
```typescript
export interface Tenant {
  id: string;
  name: string;
  slug: string;
  isActive: boolean;
}

export interface TenantSettings {
  id: string;
  logoUrl: string | null;
  primaryColor: string | null;
  currencyFormat: string;
  taxRate: number;
  timezone: string;
}
```

`pos/web/src/services/tenant.service.ts`:
```typescript
import api from '@/lib/api';

export const tenantService = {
  getMyTenants: () => api.get('/tenants/my').then(r => r.data.data),
  selectTenant: (id: string) => api.post(`/tenants/select/${id}`).then(r => r.data.data),
  getCurrent: () => api.get('/tenants/current').then(r => r.data.data),
  updateCurrent: (data: any) => api.patch('/tenants/current', data).then(r => r.data.data),
  updateSettings: (data: any) => api.patch('/tenants/current/settings', data).then(r => r.data.data),
  getMembers: () => api.get('/tenants/current/members').then(r => r.data.data),
  invite: (data: { email: string; roleId: string }) => api.post('/tenants/current/invite', data).then(r => r.data.data),
  removeMember: (userId: string) => api.delete(`/tenants/current/members/${userId}`).then(r => r.data.data),
  changeRole: (userId: string, roleId: string) => api.patch(`/tenants/current/members/${userId}/role`, { roleId }).then(r => r.data.data),
};
```

- [ ] **Step 2: Update auth provider for tenant selection**

In `pos/web/src/providers/auth-provider.tsx`, after login:
- If response has `tenants` array (multi-tenant) → show tenant selector modal
- If response has `accessToken` (single tenant) → proceed as normal
- Store selected tenant in cookie/localStorage

- [ ] **Step 3: Create Tenant Settings page**

`pos/web/src/app/(dashboard)/tenant-settings/page.tsx`:
Form page with: tenant name, slug, logo upload, primary color picker, tax rate, timezone select, currency format. Member management section: list members, invite by email, change role, remove.

- [ ] **Step 4: Add nav items**

In `pos/web/src/lib/constants.ts`, add:
- "Tenant Settings" with `user:manage` permission
- Show current tenant name in sidebar header

- [ ] **Step 5: Commit**

```bash
cd pos/web
git add src/
git commit -m "feat(phase5a): add tenant selection flow and settings page to dashboard"
```

---

## Task 11: Run All Tests + Final Verification

- [ ] **Step 1: Run all backend tests**

Run: `cd pos/backend && npx jest --no-coverage`

Expected: All tests pass. Fix any failures from tenant_id additions.

- [ ] **Step 2: Run Flutter analyze**

Run: `cd pos/customer-app && flutter analyze`

Expected: 0 issues.

- [ ] **Step 3: Verify Swagger**

Start backend, open `http://localhost:3000/api/docs`. Verify all endpoints visible and grouped.

- [ ] **Step 4: Test tenant flow manually**

```bash
# Register user
curl -X POST http://localhost:3000/v1/auth/register -H "Content-Type: application/json" -d '{"name":"Test","email":"test@test.com","password":"password"}'

# Create tenant
curl -X POST http://localhost:3000/v1/tenants -H "Authorization: Bearer <token>" -H "Content-Type: application/json" -d '{"name":"My Restaurant","slug":"my-restaurant"}'

# Verify tenant-scoped data
curl http://localhost:3000/v1/menu -H "Authorization: Bearer <tenant-token>"
```

- [ ] **Step 5: Final commit**

```bash
cd pos
git add .
git commit -m "feat(phase5a): complete multi-tenant + Swagger/API versioning"
```
