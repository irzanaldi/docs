# POS Phase 2A — Inventory Management Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add inventory management to POS backend + web dashboard — suppliers, warehouses, purchase orders (with approval flow), automatic stock deduction from transactions, stock opname, low-stock notifications.

**Architecture:** Extend existing NestJS monolith with 6 new modules (suppliers, warehouses, purchase-orders, stock-movements, stock-opnames, notifications). Modify PosService to auto-deduct stock on transaction. Update Prisma schema with 8 new models + 3 enums. Add 6 new web dashboard pages.

**Tech Stack:** NestJS, TypeScript, PostgreSQL, Prisma, Next.js, shadcn/ui

**Spec:** `docs/superpowers/specs/2026-05-01-pos-phase2-inventory-finance-design.md`

---

## File Structure

### Backend — New Files

```
backend/src/
├── suppliers/
│   ├── suppliers.module.ts
│   ├── suppliers.controller.ts
│   ├── suppliers.service.ts
│   ├── suppliers.service.spec.ts
│   └── dto/
│       ├── create-supplier.dto.ts
│       └── update-supplier.dto.ts
├── warehouses/
│   ├── warehouses.module.ts
│   ├── warehouses.controller.ts
│   ├── warehouses.service.ts
│   └── dto/
│       ├── create-warehouse.dto.ts
│       └── update-warehouse.dto.ts
├── purchase-orders/
│   ├── purchase-orders.module.ts
│   ├── purchase-orders.controller.ts
│   ├── purchase-orders.service.ts
│   ├── purchase-orders.service.spec.ts
│   └── dto/
│       ├── create-purchase-order.dto.ts
│       ├── update-purchase-order.dto.ts
│       └── receive-purchase-order.dto.ts
├── stock-movements/
│   ├── stock-movements.module.ts
│   ├── stock-movements.controller.ts
│   └── stock-movements.service.ts
├── stock-opnames/
│   ├── stock-opnames.module.ts
│   ├── stock-opnames.controller.ts
│   ├── stock-opnames.service.ts
│   ├── stock-opnames.service.spec.ts
│   └── dto/
│       ├── create-stock-opname.dto.ts
│       └── update-stock-opname.dto.ts
├── notifications/
│   ├── notifications.module.ts
│   ├── notifications.controller.ts
│   └── notifications.service.ts
```

### Backend — Modified Files

```
backend/prisma/schema.prisma          — add 8 new models, 3 enums, relations on User + Ingredient
backend/prisma/seed.ts                — add warehouse, suppliers, Phase 2 permissions
backend/src/app.module.ts             — register 6 new modules
backend/src/pos/pos.service.ts        — inject stock deduction + notification logic
backend/src/pos/pos.module.ts         — import new modules
```

### Web — New Files

```
web/src/app/(dashboard)/suppliers/page.tsx
web/src/app/(dashboard)/warehouses/page.tsx
web/src/app/(dashboard)/purchase-orders/page.tsx
web/src/app/(dashboard)/stock-movements/page.tsx
web/src/app/(dashboard)/stock-opnames/page.tsx
web/src/components/layout/notification-bell.tsx
web/src/services/supplier.service.ts
web/src/services/warehouse.service.ts
web/src/services/purchase-order.service.ts
web/src/services/stock-movement.service.ts
web/src/services/stock-opname.service.ts
web/src/services/notification.service.ts
web/src/types/supplier.ts
web/src/types/warehouse.ts
web/src/types/purchase-order.ts
web/src/types/stock-movement.ts
web/src/types/stock-opname.ts
web/src/types/notification.ts
```

### Web — Modified Files

```
web/src/lib/constants.ts              — add new permissions + sidebar items
web/src/components/layout/sidebar.tsx — add new icon imports
web/src/components/layout/header.tsx  — add notification bell
```

---

### Task 1: Prisma Schema — Add Inventory Models

**Files:** Modify: `backend/prisma/schema.prisma`

- [ ] **Step 1: Add Supplier, Warehouse, PurchaseOrder models + enums**

Add to end of `backend/prisma/schema.prisma`:

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
  id           String         @id @default(uuid())
  poNumber     String         @unique
  supplierId   String
  warehouseId  String
  status       POStatus       @default(DRAFT)
  totalAmount  Float          @default(0)
  notes        String?
  createdById  String
  approvedById String?
  approvedAt   DateTime?
  receivedAt   DateTime?
  createdAt    DateTime       @default(now())
  updatedAt    DateTime       @updatedAt
  supplier     Supplier       @relation(fields: [supplierId], references: [id])
  warehouse    Warehouse      @relation(fields: [warehouseId], references: [id])
  createdBy    User           @relation("po_created", fields: [createdById], references: [id])
  approvedBy   User?          @relation("po_approved", fields: [approvedById], references: [id])
  items        PurchaseOrderItem[]

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
  id            String            @id @default(uuid())
  ingredientId  String
  warehouseId   String
  type          StockMovementType
  qty           Float
  referenceType String
  referenceId   String?
  notes         String?
  createdById   String
  createdAt     DateTime          @default(now())
  ingredient    Ingredient        @relation(fields: [ingredientId], references: [id])
  warehouse     Warehouse         @relation(fields: [warehouseId], references: [id])
  createdBy     User              @relation(fields: [createdById], references: [id])

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
  type      String
  title     String
  message   String
  isRead    Boolean  @default(false)
  data      Json?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])

  @@map("notifications")
}
```

- [ ] **Step 2: Add relations to existing User model**

Add these fields to the existing `User` model in schema.prisma:

```prisma
  purchaseOrdersCreated  PurchaseOrder[] @relation("po_created")
  purchaseOrdersApproved PurchaseOrder[] @relation("po_approved")
  stockMovements         StockMovement[]
  stockOpnames           StockOpname[]
  notifications          Notification[]
```

- [ ] **Step 3: Add relations to existing Ingredient model**

Add these fields to the existing `Ingredient` model in schema.prisma:

```prisma
  purchaseOrderItems PurchaseOrderItem[]
  stockMovements     StockMovement[]
  stockOpnameItems   StockOpnameItem[]
```

- [ ] **Step 4: Validate schema**

```bash
cd ~/Downloads/PROJECT/pos/backend
npx prisma validate
```

Expected: "The schema at prisma/schema.prisma is valid"

- [ ] **Step 5: Run migration**

```bash
cd ~/Downloads/PROJECT/pos/backend
npx prisma migrate dev --name add_inventory_models
```

Expected: Migration applied successfully

- [ ] **Step 6: Regenerate Prisma client**

```bash
npx prisma generate
```

- [ ] **Step 7: Verify build**

```bash
npx nest build
```

Expected: Build clean

- [ ] **Step 8: Commit**

```bash
git add prisma/
git commit -m "feat: add inventory Prisma models (supplier, warehouse, PO, stock, opname, notification)"
```

---

### Task 2: Suppliers Module

**Files:** Create: `backend/src/suppliers/` (module, controller, service, DTOs)

- [ ] **Step 1: Create DTOs**

Create `backend/src/suppliers/dto/create-supplier.dto.ts`:
```typescript
import { IsString, IsOptional, IsEmail, IsBoolean } from 'class-validator';

export class CreateSupplierDto {
  @IsString()
  name: string;

  @IsOptional()
  @IsString()
  contactName?: string;

  @IsOptional()
  @IsString()
  phone?: string;

  @IsOptional()
  @IsEmail()
  email?: string;

  @IsOptional()
  @IsString()
  address?: string;
}
```

Create `backend/src/suppliers/dto/update-supplier.dto.ts`:
```typescript
import { IsString, IsOptional, IsEmail, IsBoolean } from 'class-validator';

export class UpdateSupplierDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsString()
  contactName?: string;

  @IsOptional()
  @IsString()
  phone?: string;

  @IsOptional()
  @IsEmail()
  email?: string;

  @IsOptional()
  @IsString()
  address?: string;

  @IsOptional()
  @IsBoolean()
  isActive?: boolean;
}
```

- [ ] **Step 2: Create SuppliersService**

Create `backend/src/suppliers/suppliers.service.ts`:
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateSupplierDto } from './dto/create-supplier.dto';
import { UpdateSupplierDto } from './dto/update-supplier.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class SuppliersService {
  constructor(private prisma: PrismaService) {}

  async findAll(pagination: PaginationDto) {
    const page = pagination.page ?? 1;
    const limit = pagination.limit ?? 20;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      this.prisma.supplier.findMany({
        skip,
        take: limit,
        orderBy: { name: 'asc' },
      }),
      this.prisma.supplier.count(),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const supplier = await this.prisma.supplier.findUnique({ where: { id } });
    if (!supplier) throw new NotFoundException('Supplier not found');
    return supplier;
  }

  async create(dto: CreateSupplierDto) {
    return this.prisma.supplier.create({ data: dto });
  }

  async update(id: string, dto: UpdateSupplierDto) {
    await this.findOne(id);
    return this.prisma.supplier.update({ where: { id }, data: dto });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.supplier.delete({ where: { id } });
    return { message: 'Supplier deleted' };
  }
}
```

- [ ] **Step 3: Create SuppliersController**

Create `backend/src/suppliers/suppliers.controller.ts`:
```typescript
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { SuppliersService } from './suppliers.service';
import { CreateSupplierDto } from './dto/create-supplier.dto';
import { UpdateSupplierDto } from './dto/update-supplier.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('suppliers')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('supplier:manage')
export class SuppliersController {
  constructor(private suppliersService: SuppliersService) {}

  @Get()
  findAll(@Query() pagination: PaginationDto) {
    return this.suppliersService.findAll(pagination);
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.suppliersService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateSupplierDto) {
    return this.suppliersService.create(dto);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() dto: UpdateSupplierDto) {
    return this.suppliersService.update(id, dto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.suppliersService.remove(id);
  }
}
```

- [ ] **Step 4: Create SuppliersModule**

Create `backend/src/suppliers/suppliers.module.ts`:
```typescript
import { Module } from '@nestjs/common';
import { SuppliersController } from './suppliers.controller';
import { SuppliersService } from './suppliers.service';

@Module({
  controllers: [SuppliersController],
  providers: [SuppliersService],
  exports: [SuppliersService],
})
export class SuppliersModule {}
```

- [ ] **Step 5: Register in AppModule**

Add `SuppliersModule` to `backend/src/app.module.ts` imports.

- [ ] **Step 6: Build + verify**

```bash
npx nest build
```

- [ ] **Step 7: Commit**

```bash
git add src/suppliers/ src/app.module.ts
git commit -m "feat: add suppliers module with CRUD"
```

---

### Task 3: Warehouses Module

Same CRUD pattern as Suppliers. **Files:** Create: `backend/src/warehouses/` (module, controller, service, DTOs)

- [ ] **Step 1: Create DTOs**

Create `backend/src/warehouses/dto/create-warehouse.dto.ts`:
```typescript
import { IsString, IsOptional, IsBoolean } from 'class-validator';

export class CreateWarehouseDto {
  @IsString()
  name: string;

  @IsOptional()
  @IsString()
  location?: string;

  @IsOptional()
  @IsBoolean()
  isDefault?: boolean;
}
```

Create `backend/src/warehouses/dto/update-warehouse.dto.ts`:
```typescript
import { IsString, IsOptional, IsBoolean } from 'class-validator';

export class UpdateWarehouseDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsString()
  location?: string;

  @IsOptional()
  @IsBoolean()
  isDefault?: boolean;
}
```

- [ ] **Step 2: Create WarehousesService**

Create `backend/src/warehouses/warehouses.service.ts`:
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateWarehouseDto } from './dto/create-warehouse.dto';
import { UpdateWarehouseDto } from './dto/update-warehouse.dto';

@Injectable()
export class WarehousesService {
  constructor(private prisma: PrismaService) {}

  async findAll() {
    return this.prisma.warehouse.findMany({ orderBy: { name: 'asc' } });
  }

  async findOne(id: string) {
    const warehouse = await this.prisma.warehouse.findUnique({ where: { id } });
    if (!warehouse) throw new NotFoundException('Warehouse not found');
    return warehouse;
  }

  async getDefault() {
    return this.prisma.warehouse.findFirst({ where: { isDefault: true } });
  }

  async create(dto: CreateWarehouseDto) {
    return this.prisma.warehouse.create({ data: dto });
  }

  async update(id: string, dto: UpdateWarehouseDto) {
    await this.findOne(id);
    return this.prisma.warehouse.update({ where: { id }, data: dto });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.warehouse.delete({ where: { id } });
    return { message: 'Warehouse deleted' };
  }
}
```

- [ ] **Step 3: Create WarehousesController**

Create `backend/src/warehouses/warehouses.controller.ts`:
```typescript
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, UseGuards,
} from '@nestjs/common';
import { WarehousesService } from './warehouses.service';
import { CreateWarehouseDto } from './dto/create-warehouse.dto';
import { UpdateWarehouseDto } from './dto/update-warehouse.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('warehouses')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('warehouse:manage')
export class WarehousesController {
  constructor(private warehousesService: WarehousesService) {}

  @Get()
  findAll() {
    return this.warehousesService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.warehousesService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateWarehouseDto) {
    return this.warehousesService.create(dto);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() dto: UpdateWarehouseDto) {
    return this.warehousesService.update(id, dto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.warehousesService.remove(id);
  }
}
```

- [ ] **Step 4: Create WarehousesModule, register in AppModule**

Create `backend/src/warehouses/warehouses.module.ts`:
```typescript
import { Module } from '@nestjs/common';
import { WarehousesController } from './warehouses.controller';
import { WarehousesService } from './warehouses.service';

@Module({
  controllers: [WarehousesController],
  providers: [WarehousesService],
  exports: [WarehousesService],
})
export class WarehousesModule {}
```

Add `WarehousesModule` to `AppModule` imports.

- [ ] **Step 5: Build + commit**

```bash
npx nest build
git add src/warehouses/ src/app.module.ts
git commit -m "feat: add warehouses module with CRUD"
```

---

### Task 4: Notifications Module

**Files:** Create: `backend/src/notifications/` (module, controller, service)

- [ ] **Step 1: Create NotificationsService**

Create `backend/src/notifications/notifications.service.ts`:
```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class NotificationsService {
  constructor(private prisma: PrismaService) {}

  async findAll(userId: string, page = 1, limit = 20) {
    const skip = (page - 1) * limit;
    const [data, total] = await Promise.all([
      this.prisma.notification.findMany({
        where: { userId },
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.notification.count({ where: { userId } }),
    ]);
    return { data, total, page, limit };
  }

  async getUnreadCount(userId: string) {
    return this.prisma.notification.count({
      where: { userId, isRead: false },
    });
  }

  async markAsRead(id: string, userId: string) {
    return this.prisma.notification.updateMany({
      where: { id, userId },
      data: { isRead: true },
    });
  }

  async markAllAsRead(userId: string) {
    return this.prisma.notification.updateMany({
      where: { userId, isRead: false },
      data: { isRead: true },
    });
  }

  async create(data: {
    userId: string;
    type: string;
    title: string;
    message: string;
    data?: Record<string, unknown>;
  }) {
    return this.prisma.notification.create({ data });
  }

  async createForUsersWithPermission(
    permissionName: string,
    notification: { type: string; title: string; message: string; data?: Record<string, unknown> },
  ) {
    const rolePermissions = await this.prisma.rolePermission.findMany({
      where: { permission: { name: permissionName } },
      include: { role: { include: { users: { select: { id: true } } } } },
    });

    const userIds = new Set<string>();
    for (const rp of rolePermissions) {
      for (const user of rp.role.users) {
        userIds.add(user.id);
      }
    }

    const notifications = [...userIds].map((userId) => ({
      userId,
      ...notification,
    }));

    if (notifications.length > 0) {
      await this.prisma.notification.createMany({ data: notifications });
    }
  }
}
```

- [ ] **Step 2: Create NotificationsController**

Create `backend/src/notifications/notifications.controller.ts`:
```typescript
import { Controller, Get, Patch, Param, Query, UseGuards } from '@nestjs/common';
import { NotificationsService } from './notifications.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { CurrentUser } from '../common/decorators/current-user.decorator';

@Controller('notifications')
@UseGuards(JwtAuthGuard)
export class NotificationsController {
  constructor(private notificationsService: NotificationsService) {}

  @Get()
  findAll(
    @CurrentUser('id') userId: string,
    @Query('page') page?: number,
    @Query('limit') limit?: number,
  ) {
    return this.notificationsService.findAll(userId, page ?? 1, limit ?? 20);
  }

  @Get('unread-count')
  getUnreadCount(@CurrentUser('id') userId: string) {
    return this.notificationsService.getUnreadCount(userId);
  }

  @Patch(':id/read')
  markAsRead(@Param('id') id: string, @CurrentUser('id') userId: string) {
    return this.notificationsService.markAsRead(id, userId);
  }

  @Patch('read-all')
  markAllAsRead(@CurrentUser('id') userId: string) {
    return this.notificationsService.markAllAsRead(userId);
  }
}
```

- [ ] **Step 3: Create NotificationsModule, register in AppModule**

Create `backend/src/notifications/notifications.module.ts`:
```typescript
import { Global, Module } from '@nestjs/common';
import { NotificationsController } from './notifications.controller';
import { NotificationsService } from './notifications.service';

@Global()
@Module({
  controllers: [NotificationsController],
  providers: [NotificationsService],
  exports: [NotificationsService],
})
export class NotificationsModule {}
```

Add `NotificationsModule` to `AppModule` imports.

- [ ] **Step 4: Build + commit**

```bash
npx nest build
git add src/notifications/ src/app.module.ts
git commit -m "feat: add notifications module (global, permission-based broadcast)"
```

---

### Task 5: Stock Movements Module

**Files:** Create: `backend/src/stock-movements/` (module, controller, service)

- [ ] **Step 1: Create StockMovementsService**

Create `backend/src/stock-movements/stock-movements.service.ts`:
```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class StockMovementsService {
  constructor(private prisma: PrismaService) {}

  async findAll(filters: {
    ingredientId?: string;
    warehouseId?: string;
    type?: string;
    startDate?: string;
    endDate?: string;
    page?: number;
    limit?: number;
  }) {
    const page = filters.page ?? 1;
    const limit = filters.limit ?? 50;
    const skip = (page - 1) * limit;

    const where: Record<string, unknown> = {};
    if (filters.ingredientId) where.ingredientId = filters.ingredientId;
    if (filters.warehouseId) where.warehouseId = filters.warehouseId;
    if (filters.type) where.type = filters.type;
    if (filters.startDate || filters.endDate) {
      where.createdAt = {};
      if (filters.startDate) (where.createdAt as Record<string, unknown>).gte = new Date(filters.startDate);
      if (filters.endDate) (where.createdAt as Record<string, unknown>).lte = new Date(filters.endDate);
    }

    const [data, total] = await Promise.all([
      this.prisma.stockMovement.findMany({
        where: where as any,
        skip,
        take: limit,
        include: {
          ingredient: { select: { id: true, name: true, unit: true } },
          warehouse: { select: { id: true, name: true } },
          createdBy: { select: { id: true, name: true } },
        },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.stockMovement.count({ where: where as any }),
    ]);

    return { data, total, page, limit };
  }

  async createMovement(data: {
    ingredientId: string;
    warehouseId: string;
    type: 'IN' | 'OUT' | 'ADJUSTMENT';
    qty: number;
    referenceType: string;
    referenceId?: string;
    notes?: string;
    createdById: string;
  }) {
    return this.prisma.stockMovement.create({ data: data as any });
  }
}
```

- [ ] **Step 2: Create StockMovementsController**

Create `backend/src/stock-movements/stock-movements.controller.ts`:
```typescript
import { Controller, Get, Query, UseGuards } from '@nestjs/common';
import { StockMovementsService } from './stock-movements.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('stock-movements')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('stock:read')
export class StockMovementsController {
  constructor(private stockMovementsService: StockMovementsService) {}

  @Get()
  findAll(
    @Query('ingredientId') ingredientId?: string,
    @Query('warehouseId') warehouseId?: string,
    @Query('type') type?: string,
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
    @Query('page') page?: number,
    @Query('limit') limit?: number,
  ) {
    return this.stockMovementsService.findAll({
      ingredientId, warehouseId, type, startDate, endDate, page, limit,
    });
  }
}
```

- [ ] **Step 3: Create StockMovementsModule, register in AppModule**

Create `backend/src/stock-movements/stock-movements.module.ts`:
```typescript
import { Global, Module } from '@nestjs/common';
import { StockMovementsController } from './stock-movements.controller';
import { StockMovementsService } from './stock-movements.service';

@Global()
@Module({
  controllers: [StockMovementsController],
  providers: [StockMovementsService],
  exports: [StockMovementsService],
})
export class StockMovementsModule {}
```

Add `StockMovementsModule` to `AppModule` imports.

- [ ] **Step 4: Build + commit**

```bash
npx nest build
git add src/stock-movements/ src/app.module.ts
git commit -m "feat: add stock-movements module (read-only log + create helper)"
```

---

### Task 6: Purchase Orders Module

**Files:** Create: `backend/src/purchase-orders/` (module, controller, service, DTOs, test)

- [ ] **Step 1: Create DTOs**

Create `backend/src/purchase-orders/dto/create-purchase-order.dto.ts`:
```typescript
import { IsString, IsOptional, IsArray, ValidateNested, IsNumber, Min } from 'class-validator';
import { Type } from 'class-transformer';

class PurchaseOrderItemDto {
  @IsString()
  ingredientId: string;

  @IsNumber()
  @Min(0.01)
  qtyOrdered: number;

  @IsString()
  unit: string;

  @IsNumber()
  @Min(0)
  unitPrice: number;
}

export class CreatePurchaseOrderDto {
  @IsString()
  supplierId: string;

  @IsString()
  warehouseId: string;

  @IsOptional()
  @IsString()
  notes?: string;

  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => PurchaseOrderItemDto)
  items: PurchaseOrderItemDto[];
}
```

Create `backend/src/purchase-orders/dto/receive-purchase-order.dto.ts`:
```typescript
import { IsArray, ValidateNested, IsString, IsNumber, Min } from 'class-validator';
import { Type } from 'class-transformer';

class ReceiveItemDto {
  @IsString()
  id: string;

  @IsNumber()
  @Min(0)
  qtyReceived: number;
}

export class ReceivePurchaseOrderDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => ReceiveItemDto)
  items: ReceiveItemDto[];
}
```

- [ ] **Step 2: Write PurchaseOrdersService test**

Create `backend/src/purchase-orders/purchase-orders.service.spec.ts`:
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { PurchaseOrdersService } from './purchase-orders.service';
import { PrismaService } from '../prisma/prisma.service';
import { StockMovementsService } from '../stock-movements/stock-movements.service';
import { NotificationsService } from '../notifications/notifications.service';
import { BadRequestException } from '@nestjs/common';

describe('PurchaseOrdersService', () => {
  let service: PurchaseOrdersService;

  const mockPrisma = {
    purchaseOrder: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    purchaseOrderItem: { updateMany: jest.fn() },
    ingredient: { findUnique: jest.fn(), update: jest.fn() },
    ingredientPriceLog: { create: jest.fn() },
    $transaction: jest.fn((cb) => cb(mockPrisma)),
  };

  const mockStockMovements = { createMovement: jest.fn() };
  const mockNotifications = {
    createForUsersWithPermission: jest.fn(),
    create: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        PurchaseOrdersService,
        { provide: PrismaService, useValue: mockPrisma },
        { provide: StockMovementsService, useValue: mockStockMovements },
        { provide: NotificationsService, useValue: mockNotifications },
      ],
    }).compile();

    service = module.get<PurchaseOrdersService>(PurchaseOrdersService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('submit', () => {
    it('should change status from DRAFT to SUBMITTED', async () => {
      mockPrisma.purchaseOrder.findUnique.mockResolvedValue({
        id: 'po1', status: 'DRAFT', poNumber: 'PO-001',
      });
      mockPrisma.purchaseOrder.update.mockResolvedValue({
        id: 'po1', status: 'SUBMITTED',
      });

      const result = await service.submit('po1');
      expect(result.status).toBe('SUBMITTED');
    });

    it('should reject if not DRAFT', async () => {
      mockPrisma.purchaseOrder.findUnique.mockResolvedValue({
        id: 'po1', status: 'APPROVED',
      });

      await expect(service.submit('po1')).rejects.toThrow(BadRequestException);
    });
  });

  describe('approve', () => {
    it('should change status from SUBMITTED to APPROVED', async () => {
      mockPrisma.purchaseOrder.findUnique.mockResolvedValue({
        id: 'po1', status: 'SUBMITTED', createdById: 'user1',
      });
      mockPrisma.purchaseOrder.update.mockResolvedValue({
        id: 'po1', status: 'APPROVED',
      });

      const result = await service.approve('po1', 'owner1');
      expect(result.status).toBe('APPROVED');
    });
  });
});
```

- [ ] **Step 3: Run test → verify fails**

```bash
npx jest src/purchase-orders/purchase-orders.service.spec.ts --verbose
```

Expected: FAIL — PurchaseOrdersService not found

- [ ] **Step 4: Implement PurchaseOrdersService**

Create `backend/src/purchase-orders/purchase-orders.service.ts`:
```typescript
import {
  Injectable, NotFoundException, BadRequestException,
} from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { StockMovementsService } from '../stock-movements/stock-movements.service';
import { NotificationsService } from '../notifications/notifications.service';
import { CreatePurchaseOrderDto } from './dto/create-purchase-order.dto';
import { ReceivePurchaseOrderDto } from './dto/receive-purchase-order.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class PurchaseOrdersService {
  constructor(
    private prisma: PrismaService,
    private stockMovementsService: StockMovementsService,
    private notificationsService: NotificationsService,
  ) {}

  generatePoNumber(count: number): string {
    const today = new Date().toISOString().slice(0, 10).replace(/-/g, '');
    const seq = String(count + 1).padStart(4, '0');
    return `PO-${today}-${seq}`;
  }

  async findAll(pagination: PaginationDto, status?: string) {
    const page = pagination.page ?? 1;
    const limit = pagination.limit ?? 20;
    const skip = (page - 1) * limit;

    const where: Record<string, unknown> = {};
    if (status) where.status = status;

    const [data, total] = await Promise.all([
      this.prisma.purchaseOrder.findMany({
        where: where as any,
        skip,
        take: limit,
        include: {
          supplier: { select: { id: true, name: true } },
          warehouse: { select: { id: true, name: true } },
          createdBy: { select: { id: true, name: true } },
          approvedBy: { select: { id: true, name: true } },
          _count: { select: { items: true } },
        },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.purchaseOrder.count({ where: where as any }),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const po = await this.prisma.purchaseOrder.findUnique({
      where: { id },
      include: {
        supplier: true,
        warehouse: true,
        createdBy: { select: { id: true, name: true } },
        approvedBy: { select: { id: true, name: true } },
        items: { include: { ingredient: { select: { id: true, name: true, unit: true, pricePerUnit: true } } } },
      },
    });
    if (!po) throw new NotFoundException('Purchase order not found');
    return po;
  }

  async create(dto: CreatePurchaseOrderDto, createdById: string) {
    const todayCount = await this.prisma.purchaseOrder.count({
      where: {
        createdAt: { gte: new Date(new Date().toISOString().slice(0, 10)) },
      },
    });

    const totalAmount = dto.items.reduce(
      (sum, item) => sum + item.qtyOrdered * item.unitPrice, 0,
    );

    return this.prisma.purchaseOrder.create({
      data: {
        poNumber: this.generatePoNumber(todayCount),
        supplierId: dto.supplierId,
        warehouseId: dto.warehouseId,
        notes: dto.notes,
        createdById,
        totalAmount,
        items: {
          create: dto.items.map((item) => ({
            ingredientId: item.ingredientId,
            qtyOrdered: item.qtyOrdered,
            unit: item.unit,
            unitPrice: item.unitPrice,
          })),
        },
      },
      include: { items: true },
    });
  }

  async submit(id: string) {
    const po = await this.findOne(id);
    if (po.status !== 'DRAFT') {
      throw new BadRequestException('Only DRAFT POs can be submitted');
    }

    const updated = await this.prisma.purchaseOrder.update({
      where: { id },
      data: { status: 'SUBMITTED' as any },
      include: { items: true },
    });

    await this.notificationsService.createForUsersWithPermission('po:approve', {
      type: 'PO_SUBMITTED',
      title: 'PO Baru Perlu Approval',
      message: `PO ${po.poNumber} dari ${(po as any).supplier?.name} menunggu approval`,
      data: { poId: id, poNumber: po.poNumber },
    });

    return updated;
  }

  async approve(id: string, approvedById: string) {
    const po = await this.findOne(id);
    if (po.status !== 'SUBMITTED') {
      throw new BadRequestException('Only SUBMITTED POs can be approved');
    }

    const updated = await this.prisma.purchaseOrder.update({
      where: { id },
      data: {
        status: 'APPROVED' as any,
        approvedById,
        approvedAt: new Date(),
      },
      include: { items: true },
    });

    await this.notificationsService.create({
      userId: po.createdById,
      type: 'PO_APPROVED',
      title: 'PO Disetujui',
      message: `PO ${po.poNumber} telah disetujui`,
      data: { poId: id, poNumber: po.poNumber },
    });

    return updated;
  }

  async receive(id: string, dto: ReceivePurchaseOrderDto, receivedById: string) {
    const po = await this.findOne(id);
    if (po.status !== 'APPROVED') {
      throw new BadRequestException('Only APPROVED POs can be received');
    }

    return this.prisma.$transaction(async (tx) => {
      for (const receiveItem of dto.items) {
        const poItem = po.items.find((i) => i.id === receiveItem.id);
        if (!poItem) continue;

        await tx.purchaseOrderItem.update({
          where: { id: receiveItem.id },
          data: { qtyReceived: receiveItem.qtyReceived },
        });

        // Update ingredient stock
        await tx.ingredient.update({
          where: { id: poItem.ingredientId },
          data: { stockQty: { increment: receiveItem.qtyReceived } },
        });

        // Create stock movement
        await this.stockMovementsService.createMovement({
          ingredientId: poItem.ingredientId,
          warehouseId: po.warehouseId,
          type: 'IN',
          qty: receiveItem.qtyReceived,
          referenceType: 'PO',
          referenceId: id,
          createdById: receivedById,
        });

        // Check if price changed → update + log
        const ingredient = await tx.ingredient.findUnique({
          where: { id: poItem.ingredientId },
        });
        if (ingredient && poItem.unitPrice !== ingredient.pricePerUnit) {
          await tx.ingredientPriceLog.create({
            data: {
              ingredientId: poItem.ingredientId,
              oldPrice: ingredient.pricePerUnit,
              newPrice: poItem.unitPrice,
            },
          });
          await tx.ingredient.update({
            where: { id: poItem.ingredientId },
            data: { pricePerUnit: poItem.unitPrice },
          });
        }
      }

      return tx.purchaseOrder.update({
        where: { id },
        data: {
          status: 'RECEIVED' as any,
          receivedAt: new Date(),
        },
        include: { items: true },
      });
    });
  }

  async cancel(id: string) {
    const po = await this.findOne(id);
    if (!['SUBMITTED', 'APPROVED'].includes(po.status)) {
      throw new BadRequestException('Only SUBMITTED or APPROVED POs can be cancelled');
    }

    return this.prisma.purchaseOrder.update({
      where: { id },
      data: { status: 'CANCELLED' as any },
    });
  }

  async remove(id: string) {
    const po = await this.findOne(id);
    if (po.status !== 'DRAFT') {
      throw new BadRequestException('Only DRAFT POs can be deleted');
    }
    await this.prisma.purchaseOrder.delete({ where: { id } });
    return { message: 'Purchase order deleted' };
  }
}
```

- [ ] **Step 5: Run test → verify passes**

```bash
npx jest src/purchase-orders/purchase-orders.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create PurchaseOrdersController**

Create `backend/src/purchase-orders/purchase-orders.controller.ts`:
```typescript
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { PurchaseOrdersService } from './purchase-orders.service';
import { CreatePurchaseOrderDto } from './dto/create-purchase-order.dto';
import { ReceivePurchaseOrderDto } from './dto/receive-purchase-order.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';
import { CurrentUser } from '../common/decorators/current-user.decorator';

@Controller('purchase-orders')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class PurchaseOrdersController {
  constructor(private purchaseOrdersService: PurchaseOrdersService) {}

  @Get()
  @RequirePermissions('po:create')
  findAll(@Query() pagination: PaginationDto, @Query('status') status?: string) {
    return this.purchaseOrdersService.findAll(pagination, status);
  }

  @Get(':id')
  @RequirePermissions('po:create')
  findOne(@Param('id') id: string) {
    return this.purchaseOrdersService.findOne(id);
  }

  @Post()
  @RequirePermissions('po:create')
  create(@Body() dto: CreatePurchaseOrderDto, @CurrentUser('id') userId: string) {
    return this.purchaseOrdersService.create(dto, userId);
  }

  @Delete(':id')
  @RequirePermissions('po:create')
  remove(@Param('id') id: string) {
    return this.purchaseOrdersService.remove(id);
  }

  @Patch(':id/submit')
  @RequirePermissions('po:create')
  submit(@Param('id') id: string) {
    return this.purchaseOrdersService.submit(id);
  }

  @Patch(':id/approve')
  @RequirePermissions('po:approve')
  approve(@Param('id') id: string, @CurrentUser('id') userId: string) {
    return this.purchaseOrdersService.approve(id, userId);
  }

  @Patch(':id/receive')
  @RequirePermissions('po:receive')
  receive(
    @Param('id') id: string,
    @Body() dto: ReceivePurchaseOrderDto,
    @CurrentUser('id') userId: string,
  ) {
    return this.purchaseOrdersService.receive(id, dto, userId);
  }

  @Patch(':id/cancel')
  @RequirePermissions('po:create')
  cancel(@Param('id') id: string) {
    return this.purchaseOrdersService.cancel(id);
  }
}
```

- [ ] **Step 7: Create PurchaseOrdersModule, register in AppModule**

Create `backend/src/purchase-orders/purchase-orders.module.ts`:
```typescript
import { Module } from '@nestjs/common';
import { PurchaseOrdersController } from './purchase-orders.controller';
import { PurchaseOrdersService } from './purchase-orders.service';

@Module({
  controllers: [PurchaseOrdersController],
  providers: [PurchaseOrdersService],
  exports: [PurchaseOrdersService],
})
export class PurchaseOrdersModule {}
```

Add `PurchaseOrdersModule` to `AppModule` imports.

- [ ] **Step 8: Build + commit**

```bash
npx nest build
git add src/purchase-orders/ src/app.module.ts
git commit -m "feat: add purchase-orders module with approval flow and stock receiving"
```

---

### Task 7: Stock Opnames Module

**Files:** Create: `backend/src/stock-opnames/` (module, controller, service, DTOs, test)

- [ ] **Step 1: Create DTOs**

Create `backend/src/stock-opnames/dto/create-stock-opname.dto.ts`:
```typescript
import { IsString, IsOptional } from 'class-validator';

export class CreateStockOpnameDto {
  @IsString()
  warehouseId: string;

  @IsOptional()
  @IsString()
  notes?: string;
}
```

Create `backend/src/stock-opnames/dto/update-stock-opname.dto.ts`:
```typescript
import { IsArray, ValidateNested, IsString, IsNumber } from 'class-validator';
import { Type } from 'class-transformer';

class OpnameItemUpdateDto {
  @IsString()
  id: string;

  @IsNumber()
  actualQty: number;
}

export class UpdateStockOpnameDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OpnameItemUpdateDto)
  items: OpnameItemUpdateDto[];
}
```

- [ ] **Step 2: Write StockOpnamesService test**

Create `backend/src/stock-opnames/stock-opnames.service.spec.ts`:
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { StockOpnamesService } from './stock-opnames.service';
import { PrismaService } from '../prisma/prisma.service';
import { StockMovementsService } from '../stock-movements/stock-movements.service';

describe('StockOpnamesService', () => {
  let service: StockOpnamesService;

  const mockPrisma = {
    stockOpname: { findUnique: jest.fn(), create: jest.fn(), update: jest.fn(), findMany: jest.fn(), count: jest.fn() },
    stockOpnameItem: { update: jest.fn() },
    ingredient: { findMany: jest.fn(), update: jest.fn() },
    $transaction: jest.fn((cb) => cb(mockPrisma)),
  };

  const mockStockMovements = { createMovement: jest.fn() };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        StockOpnamesService,
        { provide: PrismaService, useValue: mockPrisma },
        { provide: StockMovementsService, useValue: mockStockMovements },
      ],
    }).compile();

    service = module.get<StockOpnamesService>(StockOpnamesService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('complete', () => {
    it('should adjust stock for items with difference', async () => {
      mockPrisma.stockOpname.findUnique.mockResolvedValue({
        id: 'op1', status: 'DRAFT', warehouseId: 'w1',
        items: [
          { id: 'oi1', ingredientId: 'i1', systemQty: 50, actualQty: 45, difference: -5 },
          { id: 'oi2', ingredientId: 'i2', systemQty: 100, actualQty: 100, difference: 0 },
        ],
      });
      mockPrisma.stockOpname.update.mockResolvedValue({ id: 'op1', status: 'COMPLETED' });
      mockPrisma.ingredient.update.mockResolvedValue({});

      const result = await service.complete('op1', 'user1');
      expect(result.status).toBe('COMPLETED');
      expect(mockPrisma.ingredient.update).toHaveBeenCalledTimes(1);
      expect(mockStockMovements.createMovement).toHaveBeenCalledTimes(1);
    });
  });
});
```

- [ ] **Step 3: Implement StockOpnamesService**

Create `backend/src/stock-opnames/stock-opnames.service.ts`:
```typescript
import { Injectable, NotFoundException, BadRequestException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { StockMovementsService } from '../stock-movements/stock-movements.service';
import { CreateStockOpnameDto } from './dto/create-stock-opname.dto';
import { UpdateStockOpnameDto } from './dto/update-stock-opname.dto';

@Injectable()
export class StockOpnamesService {
  constructor(
    private prisma: PrismaService,
    private stockMovementsService: StockMovementsService,
  ) {}

  async findAll(page = 1, limit = 20) {
    const skip = (page - 1) * limit;
    const [data, total] = await Promise.all([
      this.prisma.stockOpname.findMany({
        skip, take: limit,
        include: {
          warehouse: { select: { id: true, name: true } },
          createdBy: { select: { id: true, name: true } },
          _count: { select: { items: true } },
        },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.stockOpname.count(),
    ]);
    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const opname = await this.prisma.stockOpname.findUnique({
      where: { id },
      include: {
        warehouse: true,
        createdBy: { select: { id: true, name: true } },
        items: { include: { ingredient: { select: { id: true, name: true, unit: true } } } },
      },
    });
    if (!opname) throw new NotFoundException('Stock opname not found');
    return opname;
  }

  async create(dto: CreateStockOpnameDto, createdById: string) {
    const ingredients = await this.prisma.ingredient.findMany({
      select: { id: true, stockQty: true },
    });

    return this.prisma.stockOpname.create({
      data: {
        warehouseId: dto.warehouseId,
        notes: dto.notes,
        createdById,
        items: {
          create: ingredients.map((ing) => ({
            ingredientId: ing.id,
            systemQty: ing.stockQty,
            actualQty: ing.stockQty,
            difference: 0,
          })),
        },
      },
      include: { items: { include: { ingredient: true } } },
    });
  }

  async updateItems(id: string, dto: UpdateStockOpnameDto) {
    const opname = await this.findOne(id);
    if (opname.status !== 'DRAFT') {
      throw new BadRequestException('Only DRAFT opnames can be updated');
    }

    for (const item of dto.items) {
      const opnameItem = opname.items.find((i) => i.id === item.id);
      if (!opnameItem) continue;

      await this.prisma.stockOpnameItem.update({
        where: { id: item.id },
        data: {
          actualQty: item.actualQty,
          difference: item.actualQty - opnameItem.systemQty,
        },
      });
    }

    return this.findOne(id);
  }

  async complete(id: string, userId: string) {
    const opname = await this.findOne(id);
    if (opname.status !== 'DRAFT') {
      throw new BadRequestException('Only DRAFT opnames can be completed');
    }

    return this.prisma.$transaction(async (tx) => {
      for (const item of opname.items) {
        if (item.difference === 0) continue;

        await tx.ingredient.update({
          where: { id: item.ingredientId },
          data: { stockQty: item.actualQty },
        });

        await this.stockMovementsService.createMovement({
          ingredientId: item.ingredientId,
          warehouseId: opname.warehouseId,
          type: 'ADJUSTMENT',
          qty: item.difference,
          referenceType: 'OPNAME',
          referenceId: id,
          notes: `Stock opname adjustment: ${item.difference > 0 ? '+' : ''}${item.difference}`,
          createdById: userId,
        });
      }

      return tx.stockOpname.update({
        where: { id },
        data: { status: 'COMPLETED' as any, completedAt: new Date() },
        include: { items: { include: { ingredient: true } } },
      });
    });
  }
}
```

- [ ] **Step 4: Run test → verify passes**

```bash
npx jest src/stock-opnames/stock-opnames.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 5: Create StockOpnamesController, Module, register in AppModule**

Create `backend/src/stock-opnames/stock-opnames.controller.ts`:
```typescript
import { Controller, Get, Post, Patch, Param, Body, Query, UseGuards } from '@nestjs/common';
import { StockOpnamesService } from './stock-opnames.service';
import { CreateStockOpnameDto } from './dto/create-stock-opname.dto';
import { UpdateStockOpnameDto } from './dto/update-stock-opname.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';
import { CurrentUser } from '../common/decorators/current-user.decorator';

@Controller('stock-opnames')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('stock:opname')
export class StockOpnamesController {
  constructor(private stockOpnamesService: StockOpnamesService) {}

  @Get()
  findAll(@Query('page') page?: number, @Query('limit') limit?: number) {
    return this.stockOpnamesService.findAll(page ?? 1, limit ?? 20);
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.stockOpnamesService.findOne(id);
  }

  @Post()
  create(@Body() dto: CreateStockOpnameDto, @CurrentUser('id') userId: string) {
    return this.stockOpnamesService.create(dto, userId);
  }

  @Patch(':id')
  updateItems(@Param('id') id: string, @Body() dto: UpdateStockOpnameDto) {
    return this.stockOpnamesService.updateItems(id, dto);
  }

  @Patch(':id/complete')
  complete(@Param('id') id: string, @CurrentUser('id') userId: string) {
    return this.stockOpnamesService.complete(id, userId);
  }
}
```

Create `backend/src/stock-opnames/stock-opnames.module.ts`:
```typescript
import { Module } from '@nestjs/common';
import { StockOpnamesController } from './stock-opnames.controller';
import { StockOpnamesService } from './stock-opnames.service';

@Module({
  controllers: [StockOpnamesController],
  providers: [StockOpnamesService],
  exports: [StockOpnamesService],
})
export class StockOpnamesModule {}
```

Add `StockOpnamesModule` to `AppModule` imports.

- [ ] **Step 6: Build + commit**

```bash
npx nest build
git add src/stock-opnames/ src/app.module.ts
git commit -m "feat: add stock-opnames module with auto-fill and stock adjustment"
```

---

### Task 8: Wire Stock Deduction into PosService

**Files:** Modify: `backend/src/pos/pos.service.ts`, `backend/src/pos/pos.module.ts`

- [ ] **Step 1: Update PosModule imports**

Add `StockMovementsModule` and `NotificationsModule` imports to `backend/src/pos/pos.module.ts` (NotificationsModule is global so no import needed, but StockMovementsModule is global too — just inject services).

- [ ] **Step 2: Inject services into PosService**

Add to PosService constructor:
```typescript
import { StockMovementsService } from '../stock-movements/stock-movements.service';
import { NotificationsService } from '../notifications/notifications.service';

constructor(
  private prisma: PrismaService,
  private menuService: MenuService,
  private promosService: PromosService,
  private ordersService: OrdersService,
  private ordersGateway: OrdersGateway,
  private stockMovementsService: StockMovementsService,
  private notificationsService: NotificationsService,
) {}
```

- [ ] **Step 3: Add stock deduction method**

Add to PosService:
```typescript
private async deductStock(
  transactionItems: Array<{ itemType: string; itemId: string; qty: number }>,
  transactionId: string,
  cashierId: string,
  tx: any,
) {
  // Get default warehouse
  const warehouse = await tx.warehouse.findFirst({ where: { isDefault: true } });
  const warehouseId = warehouse?.id;

  for (const item of transactionItems) {
    if (item.itemType === 'MENU') {
      const menuItem = await tx.menuItem.findUnique({
        where: { id: item.itemId },
        include: { recipes: true },
      });
      if (!menuItem) continue;

      for (const recipe of menuItem.recipes) {
        const deductQty = recipe.qtyNeeded * item.qty;
        await tx.ingredient.update({
          where: { id: recipe.ingredientId },
          data: { stockQty: { decrement: deductQty } },
        });

        if (warehouseId) {
          await this.stockMovementsService.createMovement({
            ingredientId: recipe.ingredientId,
            warehouseId,
            type: 'OUT',
            qty: deductQty,
            referenceType: 'TRANSACTION',
            referenceId: transactionId,
            createdById: cashierId,
          });
        }

        // Check low stock
        const ingredient = await tx.ingredient.findUnique({
          where: { id: recipe.ingredientId },
        });
        if (ingredient && ingredient.stockQty <= ingredient.minStock) {
          await this.notificationsService.createForUsersWithPermission('stock:read', {
            type: 'LOW_STOCK',
            title: 'Stok Rendah',
            message: `${ingredient.name} tersisa ${ingredient.stockQty} ${ingredient.unit} (minimum: ${ingredient.minStock})`,
            data: { ingredientId: ingredient.id, currentStock: ingredient.stockQty, minStock: ingredient.minStock },
          });
        }
      }
    } else if (item.itemType === 'RETAIL') {
      await tx.retailItem.update({
        where: { id: item.itemId },
        data: { stockQty: { decrement: item.qty } },
      });
    }
  }
}
```

- [ ] **Step 4: Call deductStock in create() after transaction creation**

In PosService.create(), add after `this.ordersGateway.emitNewOrder(order)` block and before `return transaction`:

```typescript
// Deduct stock
await this.deductStock(
  dto.items.map((i) => ({ itemType: i.itemType, itemId: i.itemId, qty: i.qty })),
  transaction.id,
  cashierId,
  tx,
);
```

- [ ] **Step 5: Update POS test to include new mocks**

Update `backend/src/pos/pos.service.spec.ts` — add mock providers for `StockMovementsService` and `NotificationsService`.

- [ ] **Step 6: Build + run all tests**

```bash
npx nest build
npx jest --verbose
```

Expected: All tests pass

- [ ] **Step 7: Commit**

```bash
git add src/pos/
git commit -m "feat: auto-deduct stock from transactions + low-stock notifications"
```

---

### Task 9: Update Seed Data

**Files:** Modify: `backend/prisma/seed.ts`

- [ ] **Step 1: Add Phase 2 permissions, warehouse, suppliers to seed**

Add after existing seed data:

```typescript
// Phase 2 Permissions
const phase2Permissions = [
  { name: 'supplier:manage', module: 'suppliers' },
  { name: 'warehouse:manage', module: 'warehouses' },
  { name: 'po:create', module: 'purchase-orders' },
  { name: 'po:approve', module: 'purchase-orders' },
  { name: 'po:receive', module: 'purchase-orders' },
  { name: 'stock:read', module: 'stock' },
  { name: 'stock:opname', module: 'stock' },
  { name: 'notification:read', module: 'notifications' },
];

for (const p of phase2Permissions) {
  const created = await prisma.permission.create({
    data: { ...p, description: p.name },
  });
  permissions[p.name] = created.id;
}

// Add Phase 2 permissions to OWNER (all)
for (const permId of Object.values(permissions).filter((_, i) => i >= 14)) {
  await prisma.rolePermission.create({
    data: { roleId: ownerRole.id, permissionId: permId as string },
  });
}

// Add inventory permissions to INVENTORY role
const inventoryPerms = ['supplier:manage', 'warehouse:manage', 'po:create', 'po:receive', 'stock:read', 'stock:opname', 'notification:read'];
for (const perm of inventoryPerms) {
  await prisma.rolePermission.create({
    data: { roleId: inventoryRole.id, permissionId: permissions[perm] },
  });
}

// Add stock:read to KITCHEN role
await prisma.rolePermission.create({
  data: { roleId: kitchenRole.id, permissionId: permissions['stock:read'] },
});

// Default warehouse
await prisma.warehouse.create({
  data: { name: 'Gudang Utama', location: 'Lantai 1', isDefault: true },
});

// Suppliers
await prisma.supplier.create({
  data: { name: 'PT Bahan Segar', contactName: 'Budi', phone: '08123456789', email: 'budi@bahansegar.com' },
});
await prisma.supplier.create({
  data: { name: 'UD Maju Jaya', contactName: 'Siti', phone: '08198765432', email: 'siti@majujaya.com' },
});
```

- [ ] **Step 2: Reset DB + run seed**

```bash
cd ~/Downloads/PROJECT/pos/backend
npx prisma migrate reset --force
npx prisma db seed
```

Expected: "Seed completed successfully!"

- [ ] **Step 3: Commit**

```bash
git add prisma/seed.ts
git commit -m "feat: add Phase 2 seed data (permissions, warehouse, suppliers)"
```

---

### Task 10: Web — Types + Services for Inventory

**Files:** Create: `web/src/types/` and `web/src/services/` for inventory modules

- [ ] **Step 1: Create type files**

Create types for: supplier, warehouse, purchase-order, stock-movement, stock-opname, notification (each file with interfaces matching backend API responses).

- [ ] **Step 2: Create service files**

Create services for: supplier, warehouse, purchase-order, stock-movement, stock-opname, notification (each with CRUD methods calling backend API).

- [ ] **Step 3: Update constants.ts**

Add new permissions to `ROLE_PERMISSIONS` and new items to `SIDEBAR_MENU` in `web/src/lib/constants.ts`.

- [ ] **Step 4: Build check**

```bash
cd ~/Downloads/PROJECT/pos/web
npx next build
```

- [ ] **Step 5: Commit**

```bash
git add src/types/ src/services/ src/lib/constants.ts
git commit -m "feat: add web types + services for inventory modules"
```

---

### Task 11: Web — Inventory Pages

**Files:** Create 6 new page files under `web/src/app/(dashboard)/`

- [ ] **Step 1: Create /suppliers page** — CRUD table
- [ ] **Step 2: Create /warehouses page** — simple CRUD list
- [ ] **Step 3: Create /purchase-orders page** — table with status tabs, detail actions
- [ ] **Step 4: Create /stock-movements page** — read-only log with filters
- [ ] **Step 5: Create /stock-opnames page** — list + create/complete
- [ ] **Step 6: Create notification-bell component** — header dropdown

- [ ] **Step 7: Update sidebar + header**

Add new icon imports to sidebar. Add notification bell to header.

- [ ] **Step 8: Build + verify**

```bash
npx next build
```

- [ ] **Step 9: Commit**

```bash
git add src/app/ src/components/
git commit -m "feat: add inventory web pages (suppliers, warehouses, PO, stock, opname, notifications)"
```

---

### Task 12: Run All Tests & Final Verification

- [ ] **Step 1: Run all backend tests**

```bash
cd ~/Downloads/PROJECT/pos/backend
npx jest --verbose
```

Expected: All tests pass

- [ ] **Step 2: Build backend**

```bash
npx nest build
```

- [ ] **Step 3: Build web**

```bash
cd ~/Downloads/PROJECT/pos/web
npx next build
```

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "chore: Phase 2A inventory management complete"
```
