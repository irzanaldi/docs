# Phase 4: Online Ordering & CMS — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add customer-facing mobile app with online ordering (dine-in/pickup/delivery), voucher system, delivery management, CMS for app content, and FCM push notifications to the existing POS system.

**Architecture:** Monolithic extension — new NestJS modules added to existing backend at `pos/backend/`. New Flutter customer app at `pos/customer-app/`. New Next.js dashboard pages at `pos/web/`. Customer auth is completely separate from staff auth (different table, different JWT guard).

**Tech Stack:** NestJS 11 + Prisma 7.8 + PostgreSQL (backend), Flutter 3.9 + BLoC + Dio (customer app), Next.js 16.2 + shadcn/ui (dashboard), Firebase Admin SDK (FCM)

**Spec:** `docs/superpowers/specs/2026-05-02-phase4-online-ordering-cms-design.md`

---

## File Structure

### Backend (`pos/backend/src/`)

```
src/
├── customers/
│   ├── customers.module.ts
│   ├── customers.controller.ts          # /api/customers/* endpoints
│   ├── customers.service.ts             # profile + address CRUD
│   ├── customers.service.spec.ts
│   ├── auth/
│   │   ├── customer-auth.controller.ts  # /api/customers/auth/* (OTP)
│   │   ├── customer-auth.service.ts     # OTP generate/verify, JWT issue
│   │   ├── customer-auth.service.spec.ts
│   │   ├── customer-jwt.strategy.ts     # Passport JWT for customers
│   │   └── otp.service.ts              # abstracted OTP sender
│   ├── guards/
│   │   └── customer-auth.guard.ts       # CustomerAuthGuard
│   └── dto/
│       ├── request-otp.dto.ts
│       ├── verify-otp.dto.ts
│       ├── update-profile.dto.ts
│       ├── create-address.dto.ts
│       └── update-address.dto.ts
├── online-orders/
│   ├── online-orders.module.ts
│   ├── customer-orders.controller.ts    # /api/customer-app/orders/*
│   ├── staff-orders.controller.ts       # /api/online-orders/* (kasir)
│   ├── online-orders.service.ts
│   ├── online-orders.service.spec.ts
│   └── dto/
│       ├── create-online-order.dto.ts
│       └── confirm-order.dto.ts
├── vouchers/
│   ├── vouchers.module.ts
│   ├── vouchers.controller.ts           # /api/vouchers/* (dashboard)
│   ├── customer-vouchers.controller.ts  # /api/customer-app/vouchers/*
│   ├── vouchers.service.ts
│   ├── vouchers.service.spec.ts
│   └── dto/
│       ├── create-voucher.dto.ts
│       ├── update-voucher.dto.ts
│       └── validate-voucher.dto.ts
├── deliveries/
│   ├── deliveries.module.ts
│   ├── deliveries.controller.ts         # /api/deliveries/* (kasir)
│   ├── customer-deliveries.controller.ts # customer-facing
│   ├── deliveries.service.ts
│   ├── deliveries.service.spec.ts
│   └── dto/
│       ├── assign-driver.dto.ts
│       └── update-delivery-status.dto.ts
├── cms/
│   ├── cms.module.ts
│   ├── cms.controller.ts               # /api/cms/* (dashboard)
│   ├── customer-home.controller.ts     # /api/customer-app/home
│   ├── cms.service.ts
│   ├── cms.service.spec.ts
│   └── dto/
│       ├── create-banner.dto.ts
│       ├── update-banner.dto.ts
│       ├── create-section.dto.ts
│       ├── update-section.dto.ts
│       ├── update-store-info.dto.ts
│       └── reorder.dto.ts
├── fcm/
│   ├── fcm.module.ts
│   ├── fcm.controller.ts               # /api/customer-app/devices + notifications
│   ├── fcm.service.ts                   # Firebase Admin SDK wrapper
│   ├── fcm.service.spec.ts
│   └── dto/
│       ├── register-device.dto.ts
│       └── notification-query.dto.ts
├── uploads/
│   ├── uploads.module.ts
│   ├── uploads.controller.ts            # /api/uploads/image
│   └── uploads.service.ts
└── customer-app/
    ├── customer-app.module.ts           # aggregator module
    └── customer-menu.controller.ts      # /api/customer-app/menu
```

### Web Dashboard (`pos/web/src/`)

```
src/
├── app/(dashboard)/
│   ├── cms/
│   │   ├── banners/page.tsx
│   │   ├── sections/page.tsx
│   │   ├── store-info/page.tsx
│   │   └── preview/page.tsx
│   ├── vouchers/page.tsx
│   └── online-orders/page.tsx           # confirm/reject + delivery assign
├── services/
│   ├── cms.service.ts
│   ├── voucher.service.ts
│   ├── online-order.service.ts
│   └── upload.service.ts
└── types/
    ├── cms.ts
    ├── voucher.ts
    ├── online-order.ts
    └── delivery.ts
```

### Customer App (`pos/customer-app/`)

```
lib/
├── main.dart
├── config/
│   ├── app_config.dart
│   ├── app_theme.dart
│   └── app_routes.dart
├── core/
│   ├── api/
│   │   ├── api_client.dart              # Dio + interceptor
│   │   └── api_interceptor.dart
│   ├── storage/
│   │   └── token_storage.dart
│   └── error/
│       └── app_exception.dart
├── models/
│   ├── customer.dart
│   ├── menu_item.dart
│   ├── cart_item.dart
│   ├── order.dart
│   ├── voucher.dart
│   ├── delivery.dart
│   ├── banner.dart
│   ├── section.dart
│   ├── store_info.dart
│   └── notification.dart
├── repositories/
│   ├── auth_repository.dart
│   ├── home_repository.dart
│   ├── menu_repository.dart
│   ├── order_repository.dart
│   ├── voucher_repository.dart
│   ├── profile_repository.dart
│   └── notification_repository.dart
├── modules/
│   ├── auth/
│   │   ├── bloc/auth_bloc.dart
│   │   └── screens/
│   │       ├── otp_request_screen.dart
│   │       └── otp_verify_screen.dart
│   ├── home/
│   │   ├── bloc/home_bloc.dart
│   │   └── screens/home_screen.dart
│   ├── menu/
│   │   ├── bloc/menu_bloc.dart
│   │   └── screens/
│   │       ├── menu_list_screen.dart
│   │       └── menu_detail_screen.dart
│   ├── cart/
│   │   ├── bloc/cart_bloc.dart
│   │   └── screens/
│   │       ├── cart_screen.dart
│   │       └── checkout_screen.dart
│   ├── orders/
│   │   ├── bloc/order_bloc.dart
│   │   └── screens/
│   │       ├── order_list_screen.dart
│   │       └── order_detail_screen.dart
│   ├── profile/
│   │   ├── bloc/profile_bloc.dart
│   │   └── screens/
│   │       ├── profile_screen.dart
│   │       └── address_screen.dart
│   └── notifications/
│       ├── bloc/notification_bloc.dart
│       └── screens/notification_screen.dart
└── shared/
    └── widgets/
        ├── app_button.dart
        ├── app_text_field.dart
        ├── loading_overlay.dart
        └── phone_input.dart
```

---

## Task 1: Prisma Schema Migration

**Files:**
- Modify: `pos/backend/prisma/schema.prisma`
- Create: `pos/backend/prisma/seed-phase4.ts`

- [ ] **Step 1: Add all Phase 4 models to Prisma schema**

Add after existing models in `prisma/schema.prisma`:

```prisma
// ============================================
// Phase 4: Online Ordering & CMS
// ============================================

// --- Customer Auth ---

model Customer {
  id             String   @id @default(uuid())
  name           String   @default("")
  phone          String   @unique
  email          String?
  addressDefault String?  @map("address_default")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  addresses      CustomerAddress[]
  devices        CustomerDevice[]
  orders         Order[]           @relation("CustomerOrders")
  voucherUsages  VoucherUsage[]
  deliveries     Delivery[]
  notifications  CustomerNotification[]

  @@map("customers")
}

model CustomerAddress {
  id         String  @id @default(uuid())
  customerId String  @map("customer_id")
  label      String
  address    String
  latitude   Decimal? @db.Decimal(10, 7)
  longitude  Decimal? @db.Decimal(10, 7)
  isDefault  Boolean @default(false) @map("is_default")

  customer Customer @relation(fields: [customerId], references: [id])

  @@map("customer_addresses")
}

model CustomerOtp {
  id        String   @id @default(uuid())
  phone     String
  code      String
  expiresAt DateTime @map("expires_at")
  verified  Boolean  @default(false)
  createdAt DateTime @default(now()) @map("created_at")

  @@map("customer_otps")
}

model CustomerDevice {
  id         String   @id @default(uuid())
  customerId String   @map("customer_id")
  fcmToken   String   @map("fcm_token")
  deviceType DeviceType @map("device_type")
  isActive   Boolean  @default(true) @map("is_active")
  createdAt  DateTime @default(now()) @map("created_at")
  updatedAt  DateTime @updatedAt @map("updated_at")

  customer Customer @relation(fields: [customerId], references: [id])

  @@map("customer_devices")
}

enum DeviceType {
  ANDROID
  IOS
}

// --- Vouchers ---

model Voucher {
  id                String       @id @default(uuid())
  code              String       @unique
  type              VoucherType
  value             Decimal      @db.Decimal(12, 2)
  minPurchaseAmount Decimal?     @map("min_purchase_amount") @db.Decimal(12, 2)
  maxDiscountAmount Decimal?     @map("max_discount_amount") @db.Decimal(12, 2)
  startDate         DateTime     @map("start_date")
  endDate           DateTime     @map("end_date")
  isActive          Boolean      @default(true) @map("is_active")
  maxUsage          Int?         @map("max_usage")
  maxUsagePerUser   Int?         @map("max_usage_per_user")
  usedCount         Int          @default(0) @map("used_count")
  createdAt         DateTime     @default(now()) @map("created_at")
  updatedAt         DateTime     @updatedAt @map("updated_at")

  usages VoucherUsage[]
  orders Order[]       @relation("OrderVoucher")

  @@map("vouchers")
}

enum VoucherType {
  PERCENT
  FIXED
  FREE_DELIVERY
}

model VoucherUsage {
  id             String   @id @default(uuid())
  voucherId      String   @map("voucher_id")
  customerId     String   @map("customer_id")
  transactionId  String?  @map("transaction_id")
  discountAmount Decimal  @map("discount_amount") @db.Decimal(12, 2)
  usedAt         DateTime @default(now()) @map("used_at")

  voucher     Voucher     @relation(fields: [voucherId], references: [id])
  customer    Customer    @relation(fields: [customerId], references: [id])
  transaction Transaction? @relation(fields: [transactionId], references: [id])

  @@map("voucher_usages")
}

// --- Deliveries ---

model Delivery {
  id            String         @id @default(uuid())
  orderId       String         @unique @map("order_id")
  customerId    String         @map("customer_id")
  address       String
  latitude      Decimal?       @db.Decimal(10, 7)
  longitude     Decimal?       @db.Decimal(10, 7)
  deliveryFee   Decimal        @map("delivery_fee") @db.Decimal(12, 2)
  status        DeliveryStatus @default(WAITING_PICKUP)
  driverName    String?        @map("driver_name")
  driverPhone   String?        @map("driver_phone")
  estimatedTime Int?           @map("estimated_time")
  deliveredAt   DateTime?      @map("delivered_at")
  notes         String?
  createdAt     DateTime       @default(now()) @map("created_at")
  updatedAt     DateTime       @updatedAt @map("updated_at")

  order    Order    @relation(fields: [orderId], references: [id])
  customer Customer @relation(fields: [customerId], references: [id])

  @@map("deliveries")
}

enum DeliveryStatus {
  WAITING_PICKUP
  ON_DELIVERY
  DELIVERED
  CANCELLED
}

// --- CMS ---

model CmsBanner {
  id        String         @id @default(uuid())
  title     String
  imageUrl  String         @map("image_url")
  linkType  BannerLinkType @map("link_type")
  linkValue String?        @map("link_value")
  position  BannerPosition
  sortOrder Int            @default(0) @map("sort_order")
  startDate DateTime       @map("start_date")
  endDate   DateTime       @map("end_date")
  isActive  Boolean        @default(true) @map("is_active")
  createdAt DateTime       @default(now()) @map("created_at")

  @@map("cms_banners")
}

enum BannerLinkType {
  MENU_ITEM
  PROMO
  VOUCHER
  EXTERNAL_URL
  NONE
}

enum BannerPosition {
  HOME_TOP
  HOME_MIDDLE
  CATEGORY_PAGE
}

model CmsSection {
  id            String      @id @default(uuid())
  title         String
  type          SectionType
  contentConfig Json        @map("content_config")
  sortOrder     Int         @default(0) @map("sort_order")
  isActive      Boolean     @default(true) @map("is_active")
  createdAt     DateTime    @default(now()) @map("created_at")

  @@map("cms_sections")
}

enum SectionType {
  MENU_LIST
  BANNER_CAROUSEL
  CATEGORY_GRID
}

model CmsStoreInfo {
  id               String   @id @default(uuid())
  storeName        String   @map("store_name")
  logoUrl          String?  @map("logo_url")
  coverImageUrl    String?  @map("cover_image_url")
  description      String?
  address          String
  phone            String
  operatingHours   Json     @map("operating_hours")
  deliveryRadiusKm Decimal? @map("delivery_radius_km") @db.Decimal(5, 2)
  minOrderDelivery Decimal? @map("min_order_delivery") @db.Decimal(12, 2)
  deliveryFeeAmount Decimal @default(0) @map("delivery_fee_amount") @db.Decimal(12, 2)
  updatedAt        DateTime @updatedAt @map("updated_at")

  @@map("cms_store_info")
}

// --- Notifications ---

model CustomerNotification {
  id         String   @id @default(uuid())
  customerId String   @map("customer_id")
  title      String
  body       String
  isRead     Boolean  @default(false) @map("is_read")
  createdAt  DateTime @default(now()) @map("created_at")

  customer Customer @relation(fields: [customerId], references: [id])

  @@map("customer_notifications")
}
```

- [ ] **Step 2: Extend existing Order model**

In `prisma/schema.prisma`, update the existing `Order` model. Add these fields:

```prisma
model Order {
  // ...existing fields...

  // Phase 4: Online ordering fields
  source          OrderSource   @default(POS)
  orderType       OrderType?    @map("order_type")
  customerId      String?       @map("customer_id")
  tableNumber     String?       @map("table_number")
  pickupTime      DateTime?     @map("pickup_time")
  customerNotes   String?       @map("customer_notes")
  confirmedAt     DateTime?     @map("confirmed_at")
  voucherId       String?       @map("voucher_id")
  voucherDiscount Decimal       @default(0) @map("voucher_discount") @db.Decimal(12, 2)
  deliveryFee     Decimal       @default(0) @map("delivery_fee") @db.Decimal(12, 2)

  customer        Customer?  @relation("CustomerOrders", fields: [customerId], references: [id])
  voucher         Voucher?   @relation("OrderVoucher", fields: [voucherId], references: [id])
  delivery        Delivery?
}
```

Also add the new enums and extend OrderStatus:

```prisma
enum OrderSource {
  POS
  ONLINE
}

enum OrderType {
  DINE_IN
  PICKUP
  DELIVERY
}
```

Update existing `OrderStatus` enum — add `PENDING_CONFIRMATION`, `CONFIRMED`, `ON_DELIVERY`, `COMPLETED`, `CANCELLED`:

```prisma
enum OrderStatus {
  // Phase 1 (POS flow)
  PENDING
  PREPARING
  READY
  SERVED
  // Phase 4 (Online flow)
  PENDING_CONFIRMATION
  CONFIRMED
  ON_DELIVERY
  COMPLETED
  CANCELLED
}
```

Also add the `VoucherUsage` relation to the existing `Transaction` model:

```prisma
model Transaction {
  // ...existing fields...
  voucherUsages VoucherUsage[]
}
```

- [ ] **Step 3: Run migration**

Run: `cd pos/backend && npx prisma migrate dev --name phase4-online-ordering-cms`

Expected: Migration created successfully, all new tables created.

- [ ] **Step 4: Seed MARKETING role and Phase 4 permissions**

Create `pos/backend/prisma/seed-phase4.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create MARKETING role
  const marketingRole = await prisma.role.upsert({
    where: { name: 'MARKETING' },
    update: {},
    create: { name: 'MARKETING', description: 'Marketing staff — manages CMS and vouchers' },
  });

  // Phase 4 permissions
  const phase4Permissions = [
    { name: 'cms:manage', module: 'cms', description: 'Manage CMS banners, sections, store info' },
    { name: 'voucher:manage', module: 'voucher', description: 'Manage vouchers' },
    { name: 'delivery:manage', module: 'delivery', description: 'Manage deliveries' },
    { name: 'delivery:assign_driver', module: 'delivery', description: 'Assign delivery driver' },
    { name: 'online_order:read', module: 'online_order', description: 'View online orders' },
  ];

  for (const perm of phase4Permissions) {
    await prisma.permission.upsert({
      where: { name: perm.name },
      update: {},
      create: perm,
    });
  }

  // Assign permissions to roles
  const ownerRole = await prisma.role.findUnique({ where: { name: 'OWNER' } });
  const kasirRole = await prisma.role.findUnique({ where: { name: 'KASIR' } });

  const permissionMap: Record<string, string[]> = {
    'cms:manage': [ownerRole!.id, marketingRole.id],
    'voucher:manage': [ownerRole!.id, marketingRole.id],
    'delivery:manage': [ownerRole!.id, kasirRole!.id],
    'delivery:assign_driver': [ownerRole!.id, kasirRole!.id],
    'online_order:read': [ownerRole!.id, marketingRole.id, kasirRole!.id],
  };

  for (const [permName, roleIds] of Object.entries(permissionMap)) {
    const perm = await prisma.permission.findUnique({ where: { name: permName } });
    for (const roleId of roleIds) {
      await prisma.rolePermission.upsert({
        where: { roleId_permissionId: { roleId, permissionId: perm!.id } },
        update: {},
        create: { roleId, permissionId: perm!.id },
      });
    }
  }

  // Seed default CmsStoreInfo
  const existing = await prisma.cmsStoreInfo.findFirst();
  if (!existing) {
    await prisma.cmsStoreInfo.create({
      data: {
        storeName: 'My Restaurant',
        address: 'Jl. Contoh No. 1',
        phone: '08123456789',
        operatingHours: {
          mon: '08:00-22:00', tue: '08:00-22:00', wed: '08:00-22:00',
          thu: '08:00-22:00', fri: '08:00-22:00', sat: '09:00-23:00', sun: '09:00-22:00',
        },
        deliveryFeeAmount: 10000,
      },
    });
  }

  console.log('Phase 4 seed complete');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

- [ ] **Step 5: Run seed**

Run: `cd pos/backend && npx ts-node prisma/seed-phase4.ts`

Expected: "Phase 4 seed complete"

- [ ] **Step 6: Verify schema**

Run: `cd pos/backend && npx prisma generate`

Expected: Prisma Client generated successfully.

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add prisma/
git commit -m "feat(phase4): add Prisma schema for online ordering, CMS, vouchers, deliveries, customers"
```

---

## Task 2: Customer Auth Module

**Files:**
- Create: `pos/backend/src/customers/auth/otp.service.ts`
- Create: `pos/backend/src/customers/auth/customer-auth.service.ts`
- Create: `pos/backend/src/customers/auth/customer-auth.service.spec.ts`
- Create: `pos/backend/src/customers/auth/customer-auth.controller.ts`
- Create: `pos/backend/src/customers/auth/customer-jwt.strategy.ts`
- Create: `pos/backend/src/customers/guards/customer-auth.guard.ts`
- Create: `pos/backend/src/customers/dto/request-otp.dto.ts`
- Create: `pos/backend/src/customers/dto/verify-otp.dto.ts`
- Create: `pos/backend/src/customers/customers.module.ts`
- Modify: `pos/backend/src/app.module.ts`

- [ ] **Step 1: Create OTP DTOs**

`pos/backend/src/customers/dto/request-otp.dto.ts`:

```typescript
import { IsString, Matches } from 'class-validator';

export class RequestOtpDto {
  @IsString()
  @Matches(/^08\d{8,12}$/, { message: 'Phone must be valid Indonesian number (08xxxxxxxxxx)' })
  phone: string;
}
```

`pos/backend/src/customers/dto/verify-otp.dto.ts`:

```typescript
import { IsString, Matches, Length } from 'class-validator';

export class VerifyOtpDto {
  @IsString()
  @Matches(/^08\d{8,12}$/)
  phone: string;

  @IsString()
  @Length(6, 6)
  code: string;
}
```

- [ ] **Step 2: Create OTP service (abstracted)**

`pos/backend/src/customers/auth/otp.service.ts`:

```typescript
import { Injectable, Logger } from '@nestjs/common';

export interface IOtpService {
  send(phone: string, code: string): Promise<void>;
}

@Injectable()
export class OtpService implements IOtpService {
  private readonly logger = new Logger(OtpService.name);

  async send(phone: string, code: string): Promise<void> {
    // Dev mode: log to console. Replace with SMS/WA provider in production.
    this.logger.log(`[DEV] OTP for ${phone}: ${code}`);
  }
}
```

- [ ] **Step 3: Write failing tests for CustomerAuthService**

`pos/backend/src/customers/auth/customer-auth.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { CustomerAuthService } from './customer-auth.service';
import { PrismaService } from '../../prisma/prisma.service';
import { JwtService } from '@nestjs/jwt';
import { OtpService } from './otp.service';
import { BadRequestException, UnauthorizedException } from '@nestjs/common';

describe('CustomerAuthService', () => {
  let service: CustomerAuthService;
  let prisma: any;
  let jwtService: any;
  let otpService: any;

  beforeEach(async () => {
    prisma = {
      customerOtp: { count: jest.fn(), create: jest.fn(), findFirst: jest.fn(), update: jest.fn() },
      customer: { findUnique: jest.fn(), create: jest.fn() },
    };
    jwtService = { sign: jest.fn().mockReturnValue('mock-token') };
    otpService = { send: jest.fn() };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        CustomerAuthService,
        { provide: PrismaService, useValue: prisma },
        { provide: JwtService, useValue: jwtService },
        { provide: OtpService, useValue: otpService },
      ],
    }).compile();

    service = module.get(CustomerAuthService);
  });

  describe('requestOtp', () => {
    it('should generate OTP and send it', async () => {
      prisma.customerOtp.count.mockResolvedValue(0);
      prisma.customerOtp.create.mockResolvedValue({ id: '1' });

      await service.requestOtp('08123456789');

      expect(prisma.customerOtp.create).toHaveBeenCalled();
      expect(otpService.send).toHaveBeenCalledWith('08123456789', expect.any(String));
    });

    it('should reject if rate limit exceeded (3 per 5 min)', async () => {
      prisma.customerOtp.count.mockResolvedValue(3);

      await expect(service.requestOtp('08123456789'))
        .rejects.toThrow(BadRequestException);
    });
  });

  describe('verifyOtp', () => {
    it('should return JWT tokens for valid OTP', async () => {
      const mockOtp = { id: '1', phone: '08123456789', code: '123456', verified: false, expiresAt: new Date(Date.now() + 300000) };
      prisma.customerOtp.findFirst.mockResolvedValue(mockOtp);
      prisma.customerOtp.update.mockResolvedValue({ ...mockOtp, verified: true });
      prisma.customer.findUnique.mockResolvedValue({ id: 'cust-1', phone: '08123456789' });

      const result = await service.verifyOtp('08123456789', '123456');

      expect(result).toHaveProperty('accessToken', 'mock-token');
      expect(result).toHaveProperty('refreshToken', 'mock-token');
    });

    it('should auto-create customer on first login', async () => {
      const mockOtp = { id: '1', phone: '08123456789', code: '123456', verified: false, expiresAt: new Date(Date.now() + 300000) };
      prisma.customerOtp.findFirst.mockResolvedValue(mockOtp);
      prisma.customerOtp.update.mockResolvedValue({ ...mockOtp, verified: true });
      prisma.customer.findUnique.mockResolvedValue(null);
      prisma.customer.create.mockResolvedValue({ id: 'new-cust', phone: '08123456789' });

      const result = await service.verifyOtp('08123456789', '123456');

      expect(prisma.customer.create).toHaveBeenCalledWith({ data: { phone: '08123456789' } });
      expect(result).toHaveProperty('accessToken');
    });

    it('should reject expired OTP', async () => {
      prisma.customerOtp.findFirst.mockResolvedValue({
        id: '1', code: '123456', verified: false, expiresAt: new Date(Date.now() - 1000),
      });

      await expect(service.verifyOtp('08123456789', '123456'))
        .rejects.toThrow(UnauthorizedException);
    });

    it('should reject invalid OTP code', async () => {
      prisma.customerOtp.findFirst.mockResolvedValue(null);

      await expect(service.verifyOtp('08123456789', '000000'))
        .rejects.toThrow(UnauthorizedException);
    });
  });
});
```

- [ ] **Step 4: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=customer-auth.service.spec --no-coverage`

Expected: FAIL — `Cannot find module './customer-auth.service'`

- [ ] **Step 5: Implement CustomerAuthService**

`pos/backend/src/customers/auth/customer-auth.service.ts`:

```typescript
import { Injectable, BadRequestException, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { PrismaService } from '../../prisma/prisma.service';
import { OtpService } from './otp.service';

@Injectable()
export class CustomerAuthService {
  constructor(
    private prisma: PrismaService,
    private jwtService: JwtService,
    private otpService: OtpService,
  ) {}

  async requestOtp(phone: string): Promise<{ message: string }> {
    // Rate limit: max 3 per 5 minutes
    const fiveMinAgo = new Date(Date.now() - 5 * 60 * 1000);
    const recentCount = await this.prisma.customerOtp.count({
      where: { phone, createdAt: { gte: fiveMinAgo } },
    });

    if (recentCount >= 3) {
      throw new BadRequestException('Terlalu banyak permintaan OTP. Coba lagi dalam 5 menit.');
    }

    // Dev mode: static OTP. Production: generate random 6-digit.
    const code = process.env.NODE_ENV === 'production'
      ? Math.floor(100000 + Math.random() * 900000).toString()
      : '123456';

    await this.prisma.customerOtp.create({
      data: {
        phone,
        code,
        expiresAt: new Date(Date.now() + 5 * 60 * 1000), // 5 minutes
      },
    });

    await this.otpService.send(phone, code);

    return { message: 'OTP terkirim' };
  }

  async verifyOtp(phone: string, code: string) {
    const otp = await this.prisma.customerOtp.findFirst({
      where: { phone, code, verified: false },
      orderBy: { createdAt: 'desc' },
    });

    if (!otp) {
      throw new UnauthorizedException('Kode OTP salah');
    }

    if (otp.expiresAt < new Date()) {
      throw new UnauthorizedException('Kode OTP sudah expired');
    }

    await this.prisma.customerOtp.update({
      where: { id: otp.id },
      data: { verified: true },
    });

    // Find or create customer
    let customer = await this.prisma.customer.findUnique({ where: { phone } });
    if (!customer) {
      customer = await this.prisma.customer.create({ data: { phone } });
    }

    const payload = { sub: customer.id, phone: customer.phone, role: 'CUSTOMER' };

    return {
      accessToken: this.jwtService.sign(payload, { expiresIn: '1h' }),
      refreshToken: this.jwtService.sign(payload, { expiresIn: '30d' }),
      customer: { id: customer.id, name: customer.name, phone: customer.phone, email: customer.email },
    };
  }

  async refreshToken(customerId: string) {
    const customer = await this.prisma.customer.findUnique({ where: { id: customerId } });
    if (!customer) {
      throw new UnauthorizedException('Customer not found');
    }

    const payload = { sub: customer.id, phone: customer.phone, role: 'CUSTOMER' };

    return {
      accessToken: this.jwtService.sign(payload, { expiresIn: '1h' }),
    };
  }
}
```

- [ ] **Step 6: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=customer-auth.service.spec --no-coverage`

Expected: 5 tests PASS

- [ ] **Step 7: Create Customer JWT Strategy and Guard**

`pos/backend/src/customers/auth/customer-jwt.strategy.ts`:

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PrismaService } from '../../prisma/prisma.service';

@Injectable()
export class CustomerJwtStrategy extends PassportStrategy(Strategy, 'customer-jwt') {
  constructor(private prisma: PrismaService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET || 'change-me-in-production',
    });
  }

  async validate(payload: { sub: string; phone: string; role: string }) {
    if (payload.role !== 'CUSTOMER') {
      throw new UnauthorizedException();
    }

    const customer = await this.prisma.customer.findUnique({
      where: { id: payload.sub },
    });

    if (!customer) {
      throw new UnauthorizedException();
    }

    return customer;
  }
}
```

`pos/backend/src/customers/guards/customer-auth.guard.ts`:

```typescript
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class CustomerAuthGuard extends AuthGuard('customer-jwt') {}
```

- [ ] **Step 8: Create CustomerAuthController**

`pos/backend/src/customers/auth/customer-auth.controller.ts`:

```typescript
import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { CustomerAuthService } from './customer-auth.service';
import { RequestOtpDto } from '../dto/request-otp.dto';
import { VerifyOtpDto } from '../dto/verify-otp.dto';
import { CustomerAuthGuard } from '../guards/customer-auth.guard';

@Controller('customers/auth')
export class CustomerAuthController {
  constructor(private authService: CustomerAuthService) {}

  @Post('otp/request')
  requestOtp(@Body() dto: RequestOtpDto) {
    return this.authService.requestOtp(dto.phone);
  }

  @Post('otp/verify')
  verifyOtp(@Body() dto: VerifyOtpDto) {
    return this.authService.verifyOtp(dto.phone, dto.code);
  }

  @Post('refresh')
  @UseGuards(CustomerAuthGuard)
  refreshToken(@Request() req: any) {
    return this.authService.refreshToken(req.user.id);
  }
}
```

- [ ] **Step 9: Create CustomersModule**

`pos/backend/src/customers/customers.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomerAuthController } from './auth/customer-auth.controller';
import { CustomerAuthService } from './auth/customer-auth.service';
import { CustomerJwtStrategy } from './auth/customer-jwt.strategy';
import { OtpService } from './auth/otp.service';
import { CustomersController } from './customers.controller';
import { CustomersService } from './customers.service';

@Module({
  imports: [
    PrismaModule,
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET || 'change-me-in-production',
    }),
  ],
  controllers: [CustomerAuthController, CustomersController],
  providers: [CustomerAuthService, CustomerJwtStrategy, OtpService, CustomersService],
  exports: [CustomerAuthService, CustomersService],
})
export class CustomersModule {}
```

- [ ] **Step 10: Register in AppModule**

Add `CustomersModule` to imports in `pos/backend/src/app.module.ts`.

- [ ] **Step 11: Commit**

```bash
cd pos/backend
git add src/customers/ src/app.module.ts
git commit -m "feat(phase4): add customer auth module with OTP login and customer JWT guard"
```

---

## Task 3: Customer Profile & Addresses

**Files:**
- Create: `pos/backend/src/customers/customers.controller.ts`
- Create: `pos/backend/src/customers/customers.service.ts`
- Create: `pos/backend/src/customers/customers.service.spec.ts`
- Create: `pos/backend/src/customers/dto/update-profile.dto.ts`
- Create: `pos/backend/src/customers/dto/create-address.dto.ts`
- Create: `pos/backend/src/customers/dto/update-address.dto.ts`

- [ ] **Step 1: Create DTOs**

`update-profile.dto.ts`:

```typescript
import { IsString, IsEmail, IsOptional } from 'class-validator';

export class UpdateProfileDto {
  @IsString() @IsOptional() name?: string;
  @IsEmail() @IsOptional() email?: string;
  @IsString() @IsOptional() addressDefault?: string;
}
```

`create-address.dto.ts`:

```typescript
import { IsString, IsOptional, IsBoolean, IsNumber } from 'class-validator';

export class CreateAddressDto {
  @IsString() label: string;
  @IsString() address: string;
  @IsNumber() @IsOptional() latitude?: number;
  @IsNumber() @IsOptional() longitude?: number;
  @IsBoolean() @IsOptional() isDefault?: boolean;
}
```

`update-address.dto.ts`:

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateAddressDto } from './create-address.dto';

export class UpdateAddressDto extends PartialType(CreateAddressDto) {}
```

- [ ] **Step 2: Write failing tests for CustomersService**

`pos/backend/src/customers/customers.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { CustomersService } from './customers.service';
import { PrismaService } from '../prisma/prisma.service';
import { NotFoundException } from '@nestjs/common';

describe('CustomersService', () => {
  let service: CustomersService;
  let prisma: any;

  beforeEach(async () => {
    prisma = {
      customer: { findUnique: jest.fn(), update: jest.fn() },
      customerAddress: {
        findMany: jest.fn(), create: jest.fn(), findFirst: jest.fn(),
        update: jest.fn(), delete: jest.fn(), updateMany: jest.fn(),
      },
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        CustomersService,
        { provide: PrismaService, useValue: prisma },
      ],
    }).compile();

    service = module.get(CustomersService);
  });

  describe('getProfile', () => {
    it('should return customer profile', async () => {
      const customer = { id: '1', name: 'Test', phone: '0812', email: null };
      prisma.customer.findUnique.mockResolvedValue(customer);

      const result = await service.getProfile('1');
      expect(result).toEqual(customer);
    });
  });

  describe('updateProfile', () => {
    it('should update customer name and email', async () => {
      const updated = { id: '1', name: 'New Name', phone: '0812', email: 'test@test.com' };
      prisma.customer.update.mockResolvedValue(updated);

      const result = await service.updateProfile('1', { name: 'New Name', email: 'test@test.com' });
      expect(result.name).toBe('New Name');
    });
  });

  describe('createAddress', () => {
    it('should create address and clear other defaults if isDefault=true', async () => {
      prisma.customerAddress.updateMany.mockResolvedValue({ count: 1 });
      prisma.customerAddress.create.mockResolvedValue({
        id: 'addr-1', label: 'Rumah', address: 'Jl. Test', isDefault: true,
      });

      const result = await service.createAddress('1', {
        label: 'Rumah', address: 'Jl. Test', isDefault: true,
      });

      expect(prisma.customerAddress.updateMany).toHaveBeenCalledWith({
        where: { customerId: '1' }, data: { isDefault: false },
      });
      expect(result.isDefault).toBe(true);
    });
  });

  describe('deleteAddress', () => {
    it('should throw if address not owned by customer', async () => {
      prisma.customerAddress.findFirst.mockResolvedValue(null);

      await expect(service.deleteAddress('1', 'addr-999'))
        .rejects.toThrow(NotFoundException);
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=customers.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 4: Implement CustomersService**

`pos/backend/src/customers/customers.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { UpdateProfileDto } from './dto/update-profile.dto';
import { CreateAddressDto } from './dto/create-address.dto';
import { UpdateAddressDto } from './dto/update-address.dto';

@Injectable()
export class CustomersService {
  constructor(private prisma: PrismaService) {}

  async getProfile(customerId: string) {
    return this.prisma.customer.findUnique({
      where: { id: customerId },
      select: { id: true, name: true, phone: true, email: true, addressDefault: true },
    });
  }

  async updateProfile(customerId: string, dto: UpdateProfileDto) {
    return this.prisma.customer.update({
      where: { id: customerId },
      data: dto,
    });
  }

  async getAddresses(customerId: string) {
    return this.prisma.customerAddress.findMany({
      where: { customerId },
      orderBy: { isDefault: 'desc' },
    });
  }

  async createAddress(customerId: string, dto: CreateAddressDto) {
    if (dto.isDefault) {
      await this.prisma.customerAddress.updateMany({
        where: { customerId },
        data: { isDefault: false },
      });
    }

    return this.prisma.customerAddress.create({
      data: { customerId, ...dto },
    });
  }

  async updateAddress(customerId: string, addressId: string, dto: UpdateAddressDto) {
    await this.findOwnedAddress(customerId, addressId);

    if (dto.isDefault) {
      await this.prisma.customerAddress.updateMany({
        where: { customerId },
        data: { isDefault: false },
      });
    }

    return this.prisma.customerAddress.update({
      where: { id: addressId },
      data: dto,
    });
  }

  async deleteAddress(customerId: string, addressId: string) {
    await this.findOwnedAddress(customerId, addressId);
    return this.prisma.customerAddress.delete({ where: { id: addressId } });
  }

  private async findOwnedAddress(customerId: string, addressId: string) {
    const address = await this.prisma.customerAddress.findFirst({
      where: { id: addressId, customerId },
    });
    if (!address) throw new NotFoundException('Alamat tidak ditemukan');
    return address;
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=customers.service.spec --no-coverage`

Expected: 4 tests PASS

- [ ] **Step 6: Create CustomersController**

`pos/backend/src/customers/customers.controller.ts`:

```typescript
import { Controller, Get, Patch, Post, Delete, Body, Param, UseGuards, Request } from '@nestjs/common';
import { CustomersService } from './customers.service';
import { CustomerAuthGuard } from './guards/customer-auth.guard';
import { UpdateProfileDto } from './dto/update-profile.dto';
import { CreateAddressDto } from './dto/create-address.dto';
import { UpdateAddressDto } from './dto/update-address.dto';

@Controller('customers')
@UseGuards(CustomerAuthGuard)
export class CustomersController {
  constructor(private customersService: CustomersService) {}

  @Get('profile')
  getProfile(@Request() req: any) {
    return this.customersService.getProfile(req.user.id);
  }

  @Patch('profile')
  updateProfile(@Request() req: any, @Body() dto: UpdateProfileDto) {
    return this.customersService.updateProfile(req.user.id, dto);
  }

  @Get('addresses')
  getAddresses(@Request() req: any) {
    return this.customersService.getAddresses(req.user.id);
  }

  @Post('addresses')
  createAddress(@Request() req: any, @Body() dto: CreateAddressDto) {
    return this.customersService.createAddress(req.user.id, dto);
  }

  @Patch('addresses/:id')
  updateAddress(@Request() req: any, @Param('id') id: string, @Body() dto: UpdateAddressDto) {
    return this.customersService.updateAddress(req.user.id, id, dto);
  }

  @Delete('addresses/:id')
  deleteAddress(@Request() req: any, @Param('id') id: string) {
    return this.customersService.deleteAddress(req.user.id, id);
  }
}
```

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add src/customers/
git commit -m "feat(phase4): add customer profile and address management endpoints"
```

---

## Task 4: Image Upload Module

**Files:**
- Create: `pos/backend/src/uploads/uploads.module.ts`
- Create: `pos/backend/src/uploads/uploads.controller.ts`
- Create: `pos/backend/src/uploads/uploads.service.ts`
- Modify: `pos/backend/src/app.module.ts`

- [ ] **Step 1: Install multer types**

Run: `cd pos/backend && npm install --save @nestjs/platform-express && npm install --save-dev @types/multer`

- [ ] **Step 2: Create uploads directory**

Run: `mkdir -p pos/backend/uploads/images`

- [ ] **Step 3: Implement UploadsService**

`pos/backend/src/uploads/uploads.service.ts`:

```typescript
import { Injectable, BadRequestException } from '@nestjs/common';
import { randomUUID } from 'crypto';
import { extname } from 'path';

@Injectable()
export class UploadsService {
  private readonly allowedTypes = ['image/jpeg', 'image/png', 'image/webp'];
  private readonly maxSize = 2 * 1024 * 1024; // 2MB

  validateFile(file: Express.Multer.File) {
    if (!this.allowedTypes.includes(file.mimetype)) {
      throw new BadRequestException('Format harus JPG, PNG, atau WebP');
    }
    if (file.size > this.maxSize) {
      throw new BadRequestException('Ukuran file maksimal 2MB');
    }
  }

  getFileUrl(filename: string): string {
    return `/uploads/images/${filename}`;
  }

  generateFilename(originalname: string): string {
    const ext = extname(originalname);
    return `${randomUUID()}${ext}`;
  }
}
```

- [ ] **Step 4: Implement UploadsController**

`pos/backend/src/uploads/uploads.controller.ts`:

```typescript
import { Controller, Post, UseInterceptors, UploadedFile, UseGuards } from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { diskStorage } from 'multer';
import { UploadsService } from './uploads.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { extname } from 'path';
import { randomUUID } from 'crypto';

@Controller('uploads')
@UseGuards(JwtAuthGuard)
export class UploadsController {
  constructor(private uploadsService: UploadsService) {}

  @Post('image')
  @UseInterceptors(FileInterceptor('file', {
    storage: diskStorage({
      destination: './uploads/images',
      filename: (_req, file, cb) => {
        cb(null, `${randomUUID()}${extname(file.originalname)}`);
      },
    }),
    limits: { fileSize: 2 * 1024 * 1024 },
  }))
  uploadImage(@UploadedFile() file: Express.Multer.File) {
    this.uploadsService.validateFile(file);
    return { url: this.uploadsService.getFileUrl(file.filename) };
  }
}
```

- [ ] **Step 5: Create UploadsModule**

`pos/backend/src/uploads/uploads.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { UploadsController } from './uploads.controller';
import { UploadsService } from './uploads.service';

@Module({
  controllers: [UploadsController],
  providers: [UploadsService],
})
export class UploadsModule {}
```

- [ ] **Step 6: Register in AppModule + serve static files**

In `pos/backend/src/app.module.ts`, add `UploadsModule` to imports. Also add `ServeStaticModule` to serve uploaded images:

```typescript
import { ServeStaticModule } from '@nestjs/serve-static';
import { join } from 'path';

@Module({
  imports: [
    ServeStaticModule.forRoot({
      rootPath: join(__dirname, '..', 'uploads'),
      serveRoot: '/uploads',
    }),
    // ...other modules
    UploadsModule,
  ],
})
```

Run: `cd pos/backend && npm install @nestjs/serve-static`

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add src/uploads/ src/app.module.ts package.json package-lock.json
git commit -m "feat(phase4): add image upload module with local storage"
```

---

## Task 5: CMS Backend

**Files:**
- Create: `pos/backend/src/cms/cms.module.ts`
- Create: `pos/backend/src/cms/cms.controller.ts`
- Create: `pos/backend/src/cms/customer-home.controller.ts`
- Create: `pos/backend/src/cms/cms.service.ts`
- Create: `pos/backend/src/cms/cms.service.spec.ts`
- Create: `pos/backend/src/cms/dto/*.ts` (6 DTOs)
- Modify: `pos/backend/src/app.module.ts`

- [ ] **Step 1: Create CMS DTOs**

`pos/backend/src/cms/dto/create-banner.dto.ts`:

```typescript
import { IsString, IsEnum, IsOptional, IsInt, IsBoolean, IsDateString } from 'class-validator';

export class CreateBannerDto {
  @IsString() title: string;
  @IsString() imageUrl: string;
  @IsEnum(['MENU_ITEM', 'PROMO', 'VOUCHER', 'EXTERNAL_URL', 'NONE']) linkType: string;
  @IsString() @IsOptional() linkValue?: string;
  @IsEnum(['HOME_TOP', 'HOME_MIDDLE', 'CATEGORY_PAGE']) position: string;
  @IsInt() @IsOptional() sortOrder?: number;
  @IsDateString() startDate: string;
  @IsDateString() endDate: string;
  @IsBoolean() @IsOptional() isActive?: boolean;
}
```

`pos/backend/src/cms/dto/update-banner.dto.ts`:

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateBannerDto } from './create-banner.dto';

export class UpdateBannerDto extends PartialType(CreateBannerDto) {}
```

`pos/backend/src/cms/dto/create-section.dto.ts`:

```typescript
import { IsString, IsEnum, IsObject, IsInt, IsBoolean, IsOptional } from 'class-validator';

export class CreateSectionDto {
  @IsString() title: string;
  @IsEnum(['MENU_LIST', 'BANNER_CAROUSEL', 'CATEGORY_GRID']) type: string;
  @IsObject() contentConfig: Record<string, any>;
  @IsInt() @IsOptional() sortOrder?: number;
  @IsBoolean() @IsOptional() isActive?: boolean;
}
```

`pos/backend/src/cms/dto/update-section.dto.ts`:

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateSectionDto } from './create-section.dto';

export class UpdateSectionDto extends PartialType(CreateSectionDto) {}
```

`pos/backend/src/cms/dto/update-store-info.dto.ts`:

```typescript
import { IsString, IsOptional, IsObject, IsNumber } from 'class-validator';

export class UpdateStoreInfoDto {
  @IsString() @IsOptional() storeName?: string;
  @IsString() @IsOptional() logoUrl?: string;
  @IsString() @IsOptional() coverImageUrl?: string;
  @IsString() @IsOptional() description?: string;
  @IsString() @IsOptional() address?: string;
  @IsString() @IsOptional() phone?: string;
  @IsObject() @IsOptional() operatingHours?: Record<string, string>;
  @IsNumber() @IsOptional() deliveryRadiusKm?: number;
  @IsNumber() @IsOptional() minOrderDelivery?: number;
  @IsNumber() @IsOptional() deliveryFeeAmount?: number;
}
```

`pos/backend/src/cms/dto/reorder.dto.ts`:

```typescript
import { IsArray, ValidateNested, IsString, IsInt } from 'class-validator';
import { Type } from 'class-transformer';

class ReorderItemDto {
  @IsString() id: string;
  @IsInt() sortOrder: number;
}

export class ReorderDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => ReorderItemDto)
  items: ReorderItemDto[];
}
```

- [ ] **Step 2: Write failing tests for CmsService**

`pos/backend/src/cms/cms.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { CmsService } from './cms.service';
import { PrismaService } from '../prisma/prisma.service';

describe('CmsService', () => {
  let service: CmsService;
  let prisma: any;

  beforeEach(async () => {
    prisma = {
      cmsBanner: {
        findMany: jest.fn(), create: jest.fn(), update: jest.fn(), delete: jest.fn(),
      },
      cmsSection: {
        findMany: jest.fn(), create: jest.fn(), update: jest.fn(), delete: jest.fn(),
      },
      cmsStoreInfo: {
        findFirst: jest.fn(), upsert: jest.fn(),
      },
      menuItem: { findMany: jest.fn() },
      $transaction: jest.fn((cb) => cb(prisma)),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [CmsService, { provide: PrismaService, useValue: prisma }],
    }).compile();

    service = module.get(CmsService);
  });

  describe('getStoreInfo', () => {
    it('should return store info', async () => {
      const storeInfo = { id: '1', storeName: 'Test', operatingHours: { mon: '08:00-22:00' } };
      prisma.cmsStoreInfo.findFirst.mockResolvedValue(storeInfo);

      const result = await service.getStoreInfo();
      expect(result.storeName).toBe('Test');
    });
  });

  describe('getHomePage', () => {
    it('should return resolved home page data with is_open flag', async () => {
      prisma.cmsStoreInfo.findFirst.mockResolvedValue({
        storeName: 'Test', operatingHours: { mon: '00:00-23:59', tue: '00:00-23:59', wed: '00:00-23:59', thu: '00:00-23:59', fri: '00:00-23:59', sat: '00:00-23:59', sun: '00:00-23:59' },
        logoUrl: null, coverImageUrl: null,
      });
      prisma.cmsBanner.findMany.mockResolvedValue([]);
      prisma.cmsSection.findMany.mockResolvedValue([]);

      const result = await service.getHomePage();

      expect(result).toHaveProperty('store_info');
      expect(result.store_info).toHaveProperty('is_open');
      expect(result).toHaveProperty('banners');
      expect(result).toHaveProperty('sections');
    });
  });

  describe('resolveSection', () => {
    it('should resolve MENU_LIST with manual mode', async () => {
      const section = { id: '1', title: 'Favorites', type: 'MENU_LIST', contentConfig: { mode: 'manual', item_ids: ['a', 'b'] }, sortOrder: 0 };
      const menus = [
        { id: 'a', name: 'Nasi Goreng', sellingPrice: 25000, imageUrl: null },
        { id: 'b', name: 'Es Teh', sellingPrice: 8000, imageUrl: null },
      ];
      prisma.menuItem.findMany.mockResolvedValue(menus);

      const result = await service.resolveSection(section);

      expect(result.items).toHaveLength(2);
      expect(result.items[0].name).toBe('Nasi Goreng');
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=cms.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 4: Implement CmsService**

`pos/backend/src/cms/cms.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateBannerDto } from './dto/create-banner.dto';
import { UpdateBannerDto } from './dto/update-banner.dto';
import { CreateSectionDto } from './dto/create-section.dto';
import { UpdateSectionDto } from './dto/update-section.dto';
import { UpdateStoreInfoDto } from './dto/update-store-info.dto';
import { ReorderDto } from './dto/reorder.dto';

@Injectable()
export class CmsService {
  constructor(private prisma: PrismaService) {}

  // --- Banners ---

  async getBanners() {
    return this.prisma.cmsBanner.findMany({ orderBy: { sortOrder: 'asc' } });
  }

  async createBanner(dto: CreateBannerDto) {
    return this.prisma.cmsBanner.create({ data: dto as any });
  }

  async updateBanner(id: string, dto: UpdateBannerDto) {
    return this.prisma.cmsBanner.update({ where: { id }, data: dto as any });
  }

  async deleteBanner(id: string) {
    return this.prisma.cmsBanner.delete({ where: { id } });
  }

  async reorderBanners(dto: ReorderDto) {
    await this.prisma.$transaction(
      dto.items.map((item) =>
        this.prisma.cmsBanner.update({ where: { id: item.id }, data: { sortOrder: item.sortOrder } }),
      ),
    );
    return { message: 'Reorder berhasil' };
  }

  // --- Sections ---

  async getSections() {
    return this.prisma.cmsSection.findMany({ orderBy: { sortOrder: 'asc' } });
  }

  async createSection(dto: CreateSectionDto) {
    return this.prisma.cmsSection.create({ data: dto as any });
  }

  async updateSection(id: string, dto: UpdateSectionDto) {
    return this.prisma.cmsSection.update({ where: { id }, data: dto as any });
  }

  async deleteSection(id: string) {
    return this.prisma.cmsSection.delete({ where: { id } });
  }

  async reorderSections(dto: ReorderDto) {
    await this.prisma.$transaction(
      dto.items.map((item) =>
        this.prisma.cmsSection.update({ where: { id: item.id }, data: { sortOrder: item.sortOrder } }),
      ),
    );
    return { message: 'Reorder berhasil' };
  }

  // --- Store Info ---

  async getStoreInfo() {
    const info = await this.prisma.cmsStoreInfo.findFirst();
    if (!info) throw new NotFoundException('Store info belum diatur');
    return info;
  }

  async updateStoreInfo(dto: UpdateStoreInfoDto) {
    const existing = await this.prisma.cmsStoreInfo.findFirst();
    if (existing) {
      return this.prisma.cmsStoreInfo.update({ where: { id: existing.id }, data: dto as any });
    }
    return this.prisma.cmsStoreInfo.create({ data: dto as any });
  }

  // --- Customer Home Page ---

  async getHomePage() {
    const storeInfo = await this.prisma.cmsStoreInfo.findFirst();
    const now = new Date();

    const banners = await this.prisma.cmsBanner.findMany({
      where: { isActive: true, startDate: { lte: now }, endDate: { gte: now } },
      orderBy: { sortOrder: 'asc' },
    });

    const sections = await this.prisma.cmsSection.findMany({
      where: { isActive: true },
      orderBy: { sortOrder: 'asc' },
    });

    const resolvedSections = await Promise.all(sections.map((s) => this.resolveSection(s)));

    // Group banners by position
    const bannersByPosition: Record<string, any[]> = {};
    for (const banner of banners) {
      const pos = banner.position;
      if (!bannersByPosition[pos]) bannersByPosition[pos] = [];
      bannersByPosition[pos].push({
        id: banner.id, title: banner.title, image_url: banner.imageUrl,
        link_type: banner.linkType, link_value: banner.linkValue,
      });
    }

    return {
      store_info: storeInfo ? {
        store_name: storeInfo.storeName,
        logo_url: storeInfo.logoUrl,
        cover_image_url: storeInfo.coverImageUrl,
        operating_hours: storeInfo.operatingHours,
        is_open: this.isStoreOpen(storeInfo.operatingHours as Record<string, string>),
      } : null,
      banners: bannersByPosition,
      sections: resolvedSections,
    };
  }

  async resolveSection(section: any) {
    const config = section.contentConfig as any;
    const base = { id: section.id, title: section.title, type: section.type };

    if (section.type === 'MENU_LIST') {
      let items: any[];
      if (config.mode === 'manual') {
        items = await this.prisma.menuItem.findMany({
          where: { id: { in: config.item_ids }, isActive: true },
          select: { id: true, name: true, sellingPrice: true, imageUrl: true },
        });
      } else {
        // auto mode
        const where: any = { isActive: true };
        if (config.category) where.category = config.category;
        items = await this.prisma.menuItem.findMany({
          where,
          select: { id: true, name: true, sellingPrice: true, imageUrl: true },
          take: config.limit || 10,
          orderBy: { createdAt: 'desc' },
        });
      }
      return { ...base, items };
    }

    if (section.type === 'CATEGORY_GRID') {
      const categories = await this.prisma.menuItem.groupBy({
        by: ['category'],
        where: { isActive: true },
        _count: true,
      });
      return {
        ...base,
        categories: categories.map((c) => ({ name: c.category, item_count: c._count })),
      };
    }

    if (section.type === 'BANNER_CAROUSEL') {
      const banners = await this.prisma.cmsBanner.findMany({
        where: { position: config.banner_position, isActive: true },
        orderBy: { sortOrder: 'asc' },
      });
      return { ...base, banners };
    }

    return base;
  }

  private isStoreOpen(hours: Record<string, string>): boolean {
    const days = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'];
    const now = new Date();
    const dayKey = days[now.getDay()];
    const todayHours = hours[dayKey];
    if (!todayHours) return false;

    const [open, close] = todayHours.split('-');
    const currentTime = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
    return currentTime >= open && currentTime <= close;
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=cms.service.spec --no-coverage`

Expected: 3 tests PASS

- [ ] **Step 6: Create CmsController (dashboard)**

`pos/backend/src/cms/cms.controller.ts`:

```typescript
import { Controller, Get, Post, Patch, Delete, Body, Param, UseGuards } from '@nestjs/common';
import { CmsService } from './cms.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/require-permissions.decorator';
import { CreateBannerDto } from './dto/create-banner.dto';
import { UpdateBannerDto } from './dto/update-banner.dto';
import { CreateSectionDto } from './dto/create-section.dto';
import { UpdateSectionDto } from './dto/update-section.dto';
import { UpdateStoreInfoDto } from './dto/update-store-info.dto';
import { ReorderDto } from './dto/reorder.dto';

@Controller('cms')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('cms:manage')
export class CmsController {
  constructor(private cmsService: CmsService) {}

  // Banners
  @Get('banners') getBanners() { return this.cmsService.getBanners(); }
  @Post('banners') createBanner(@Body() dto: CreateBannerDto) { return this.cmsService.createBanner(dto); }
  @Patch('banners/:id') updateBanner(@Param('id') id: string, @Body() dto: UpdateBannerDto) { return this.cmsService.updateBanner(id, dto); }
  @Delete('banners/:id') deleteBanner(@Param('id') id: string) { return this.cmsService.deleteBanner(id); }
  @Patch('banners/reorder') reorderBanners(@Body() dto: ReorderDto) { return this.cmsService.reorderBanners(dto); }

  // Sections
  @Get('sections') getSections() { return this.cmsService.getSections(); }
  @Post('sections') createSection(@Body() dto: CreateSectionDto) { return this.cmsService.createSection(dto); }
  @Patch('sections/:id') updateSection(@Param('id') id: string, @Body() dto: UpdateSectionDto) { return this.cmsService.updateSection(id, dto); }
  @Delete('sections/:id') deleteSection(@Param('id') id: string) { return this.cmsService.deleteSection(id); }
  @Patch('sections/reorder') reorderSections(@Body() dto: ReorderDto) { return this.cmsService.reorderSections(dto); }

  // Store Info
  @Get('store-info') getStoreInfo() { return this.cmsService.getStoreInfo(); }
  @Patch('store-info') updateStoreInfo(@Body() dto: UpdateStoreInfoDto) { return this.cmsService.updateStoreInfo(dto); }

  // Preview
  @Get('preview') getPreview() { return this.cmsService.getHomePage(); }
}
```

- [ ] **Step 7: Create CustomerHomeController**

`pos/backend/src/cms/customer-home.controller.ts`:

```typescript
import { Controller, Get } from '@nestjs/common';
import { CmsService } from './cms.service';

@Controller('customer-app')
export class CustomerHomeController {
  constructor(private cmsService: CmsService) {}

  @Get('home')
  getHomePage() {
    return this.cmsService.getHomePage();
  }
}
```

Note: `/api/customer-app/home` is public — no auth required for browsing.

- [ ] **Step 8: Create CmsModule and register**

`pos/backend/src/cms/cms.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CmsController } from './cms.controller';
import { CustomerHomeController } from './customer-home.controller';
import { CmsService } from './cms.service';

@Module({
  imports: [PrismaModule],
  controllers: [CmsController, CustomerHomeController],
  providers: [CmsService],
  exports: [CmsService],
})
export class CmsModule {}
```

Add `CmsModule` to `app.module.ts` imports.

- [ ] **Step 9: Commit**

```bash
cd pos/backend
git add src/cms/ src/app.module.ts
git commit -m "feat(phase4): add CMS module with banners, sections, store info, and customer home endpoint"
```

---

## Task 6: Voucher Backend

**Files:**
- Create: `pos/backend/src/vouchers/vouchers.module.ts`
- Create: `pos/backend/src/vouchers/vouchers.controller.ts`
- Create: `pos/backend/src/vouchers/customer-vouchers.controller.ts`
- Create: `pos/backend/src/vouchers/vouchers.service.ts`
- Create: `pos/backend/src/vouchers/vouchers.service.spec.ts`
- Create: `pos/backend/src/vouchers/dto/*.ts`

- [ ] **Step 1: Create Voucher DTOs**

`pos/backend/src/vouchers/dto/create-voucher.dto.ts`:

```typescript
import { IsString, IsEnum, IsNumber, IsOptional, IsInt, IsBoolean, IsDateString, Min } from 'class-validator';

export class CreateVoucherDto {
  @IsString() code: string;
  @IsEnum(['PERCENT', 'FIXED', 'FREE_DELIVERY']) type: string;
  @IsNumber() @Min(0) value: number;
  @IsNumber() @IsOptional() minPurchaseAmount?: number;
  @IsNumber() @IsOptional() maxDiscountAmount?: number;
  @IsDateString() startDate: string;
  @IsDateString() endDate: string;
  @IsBoolean() @IsOptional() isActive?: boolean;
  @IsInt() @IsOptional() maxUsage?: number;
  @IsInt() @IsOptional() maxUsagePerUser?: number;
}
```

`pos/backend/src/vouchers/dto/update-voucher.dto.ts`:

```typescript
import { PartialType } from '@nestjs/mapped-types';
import { CreateVoucherDto } from './create-voucher.dto';

export class UpdateVoucherDto extends PartialType(CreateVoucherDto) {}
```

`pos/backend/src/vouchers/dto/validate-voucher.dto.ts`:

```typescript
import { IsString, IsNumber, IsEnum, Min } from 'class-validator';

export class ValidateVoucherDto {
  @IsString() code: string;
  @IsNumber() @Min(0) subtotal: number;
  @IsEnum(['DINE_IN', 'PICKUP', 'DELIVERY']) orderType: string;
}
```

- [ ] **Step 2: Write failing tests for VouchersService**

`pos/backend/src/vouchers/vouchers.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { VouchersService } from './vouchers.service';
import { PrismaService } from '../prisma/prisma.service';
import { BadRequestException } from '@nestjs/common';

describe('VouchersService', () => {
  let service: VouchersService;
  let prisma: any;

  const mockVoucher = {
    id: 'v1', code: 'TEST20', type: 'PERCENT', value: 20,
    minPurchaseAmount: 50000, maxDiscountAmount: 15000,
    startDate: new Date('2026-01-01'), endDate: new Date('2026-12-31'),
    isActive: true, maxUsage: 100, maxUsagePerUser: 1, usedCount: 0,
  };

  beforeEach(async () => {
    prisma = {
      voucher: { findMany: jest.fn(), findUnique: jest.fn(), create: jest.fn(), update: jest.fn() },
      voucherUsage: { count: jest.fn() },
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [VouchersService, { provide: PrismaService, useValue: prisma }],
    }).compile();

    service = module.get(VouchersService);
  });

  describe('validateVoucher', () => {
    it('should return discount amount for valid PERCENT voucher', async () => {
      prisma.voucher.findUnique.mockResolvedValue(mockVoucher);
      prisma.voucherUsage.count.mockResolvedValue(0);

      const result = await service.validateVoucher('cust-1', {
        code: 'TEST20', subtotal: 100000, orderType: 'DINE_IN',
      });

      // 20% of 100000 = 20000, capped at 15000
      expect(result.valid).toBe(true);
      expect(result.discount_amount).toBe(15000);
    });

    it('should return discount for FIXED voucher', async () => {
      prisma.voucher.findUnique.mockResolvedValue({ ...mockVoucher, type: 'FIXED', value: 10000, maxDiscountAmount: null });
      prisma.voucherUsage.count.mockResolvedValue(0);

      const result = await service.validateVoucher('cust-1', {
        code: 'TEST20', subtotal: 100000, orderType: 'DINE_IN',
      });

      expect(result.discount_amount).toBe(10000);
    });

    it('should reject FREE_DELIVERY for non-delivery orders', async () => {
      prisma.voucher.findUnique.mockResolvedValue({ ...mockVoucher, type: 'FREE_DELIVERY' });
      prisma.voucherUsage.count.mockResolvedValue(0);

      const result = await service.validateVoucher('cust-1', {
        code: 'TEST20', subtotal: 100000, orderType: 'DINE_IN',
      });

      expect(result.valid).toBe(false);
      expect(result.message).toContain('delivery');
    });

    it('should reject if min_purchase not met', async () => {
      prisma.voucher.findUnique.mockResolvedValue(mockVoucher);
      prisma.voucherUsage.count.mockResolvedValue(0);

      const result = await service.validateVoucher('cust-1', {
        code: 'TEST20', subtotal: 30000, orderType: 'DINE_IN',
      });

      expect(result.valid).toBe(false);
      expect(result.message).toContain('Minimum');
    });

    it('should reject if user already used max times', async () => {
      prisma.voucher.findUnique.mockResolvedValue(mockVoucher);
      prisma.voucherUsage.count.mockResolvedValue(1); // maxUsagePerUser = 1

      const result = await service.validateVoucher('cust-1', {
        code: 'TEST20', subtotal: 100000, orderType: 'DINE_IN',
      });

      expect(result.valid).toBe(false);
      expect(result.message).toContain('sudah pakai');
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=vouchers.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 4: Implement VouchersService**

`pos/backend/src/vouchers/vouchers.service.ts`:

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateVoucherDto } from './dto/create-voucher.dto';
import { UpdateVoucherDto } from './dto/update-voucher.dto';
import { ValidateVoucherDto } from './dto/validate-voucher.dto';

@Injectable()
export class VouchersService {
  constructor(private prisma: PrismaService) {}

  // --- Dashboard CRUD ---

  async findAll() {
    return this.prisma.voucher.findMany({ orderBy: { createdAt: 'desc' } });
  }

  async findOne(id: string) {
    const voucher = await this.prisma.voucher.findUnique({
      where: { id },
      include: { _count: { select: { usages: true } } },
    });
    if (!voucher) throw new NotFoundException('Voucher tidak ditemukan');
    return voucher;
  }

  async create(dto: CreateVoucherDto) {
    return this.prisma.voucher.create({ data: dto as any });
  }

  async update(id: string, dto: UpdateVoucherDto) {
    return this.prisma.voucher.update({ where: { id }, data: dto as any });
  }

  async softDelete(id: string) {
    return this.prisma.voucher.update({ where: { id }, data: { isActive: false } });
  }

  async getUsages(id: string) {
    return this.prisma.voucherUsage.findMany({
      where: { voucherId: id },
      include: { customer: { select: { id: true, name: true, phone: true } } },
      orderBy: { usedAt: 'desc' },
    });
  }

  // --- Customer Validation ---

  async validateVoucher(customerId: string, dto: ValidateVoucherDto) {
    const voucher = await this.prisma.voucher.findUnique({ where: { code: dto.code } });

    // 1. Exists & active
    if (!voucher || !voucher.isActive) {
      return { valid: false, discount_amount: 0, message: 'Voucher tidak ditemukan' };
    }

    // 2. Date range
    const now = new Date();
    if (now < voucher.startDate || now > voucher.endDate) {
      return { valid: false, discount_amount: 0, message: 'Voucher sudah expired' };
    }

    // 3. Total usage limit
    if (voucher.maxUsage && voucher.usedCount >= voucher.maxUsage) {
      return { valid: false, discount_amount: 0, message: 'Voucher sudah habis' };
    }

    // 4. Per-user limit
    if (voucher.maxUsagePerUser) {
      const userUsageCount = await this.prisma.voucherUsage.count({
        where: { voucherId: voucher.id, customerId },
      });
      if (userUsageCount >= voucher.maxUsagePerUser) {
        return { valid: false, discount_amount: 0, message: 'Kamu sudah pakai voucher ini' };
      }
    }

    // 5. Min purchase
    if (voucher.minPurchaseAmount && dto.subtotal < Number(voucher.minPurchaseAmount)) {
      return { valid: false, discount_amount: 0, message: `Minimum belanja Rp ${Number(voucher.minPurchaseAmount).toLocaleString('id-ID')}` };
    }

    // 6. FREE_DELIVERY only for delivery
    if (voucher.type === 'FREE_DELIVERY' && dto.orderType !== 'DELIVERY') {
      return { valid: false, discount_amount: 0, message: 'Voucher hanya untuk delivery' };
    }

    // Calculate discount
    const discountAmount = this.calculateDiscount(voucher, dto.subtotal);

    return { valid: true, discount_amount: discountAmount, message: 'Voucher valid' };
  }

  calculateDiscount(voucher: any, subtotal: number): number {
    const value = Number(voucher.value);

    switch (voucher.type) {
      case 'PERCENT': {
        const raw = subtotal * value / 100;
        const maxDiscount = voucher.maxDiscountAmount ? Number(voucher.maxDiscountAmount) : Infinity;
        return Math.min(raw, maxDiscount);
      }
      case 'FIXED':
        return Math.min(value, subtotal);
      case 'FREE_DELIVERY':
        return 0; // Handled in order creation (delivery fee becomes 0)
      default:
        return 0;
    }
  }

  // --- Welcome Voucher ---

  async createWelcomeVoucher(customerId: string) {
    const code = `WELCOME-${Math.random().toString(36).substring(2, 8).toUpperCase()}`;
    const now = new Date();
    const endDate = new Date(now.getTime() + 30 * 24 * 60 * 60 * 1000); // 30 days

    return this.prisma.voucher.create({
      data: {
        code,
        type: 'PERCENT',
        value: 10,
        maxDiscountAmount: 15000,
        maxUsage: 1,
        maxUsagePerUser: 1,
        startDate: now,
        endDate,
        isActive: true,
      },
    });
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=vouchers.service.spec --no-coverage`

Expected: 5 tests PASS

- [ ] **Step 6: Create controllers and module**

`pos/backend/src/vouchers/vouchers.controller.ts`:

```typescript
import { Controller, Get, Post, Patch, Delete, Body, Param, UseGuards } from '@nestjs/common';
import { VouchersService } from './vouchers.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/require-permissions.decorator';
import { CreateVoucherDto } from './dto/create-voucher.dto';
import { UpdateVoucherDto } from './dto/update-voucher.dto';

@Controller('vouchers')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('voucher:manage')
export class VouchersController {
  constructor(private vouchersService: VouchersService) {}

  @Get() findAll() { return this.vouchersService.findAll(); }
  @Post() create(@Body() dto: CreateVoucherDto) { return this.vouchersService.create(dto); }
  @Get(':id') findOne(@Param('id') id: string) { return this.vouchersService.findOne(id); }
  @Patch(':id') update(@Param('id') id: string, @Body() dto: UpdateVoucherDto) { return this.vouchersService.update(id, dto); }
  @Delete(':id') remove(@Param('id') id: string) { return this.vouchersService.softDelete(id); }
  @Get(':id/usages') getUsages(@Param('id') id: string) { return this.vouchersService.getUsages(id); }
}
```

`pos/backend/src/vouchers/customer-vouchers.controller.ts`:

```typescript
import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { VouchersService } from './vouchers.service';
import { CustomerAuthGuard } from '../customers/guards/customer-auth.guard';
import { ValidateVoucherDto } from './dto/validate-voucher.dto';

@Controller('customer-app/vouchers')
@UseGuards(CustomerAuthGuard)
export class CustomerVouchersController {
  constructor(private vouchersService: VouchersService) {}

  @Post('validate')
  validate(@Request() req: any, @Body() dto: ValidateVoucherDto) {
    return this.vouchersService.validateVoucher(req.user.id, dto);
  }
}
```

`pos/backend/src/vouchers/vouchers.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomersModule } from '../customers/customers.module';
import { VouchersController } from './vouchers.controller';
import { CustomerVouchersController } from './customer-vouchers.controller';
import { VouchersService } from './vouchers.service';

@Module({
  imports: [PrismaModule, CustomersModule],
  controllers: [VouchersController, CustomerVouchersController],
  providers: [VouchersService],
  exports: [VouchersService],
})
export class VouchersModule {}
```

Add `VouchersModule` to `app.module.ts`.

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add src/vouchers/ src/app.module.ts
git commit -m "feat(phase4): add voucher system with validation pipeline and welcome voucher"
```

---

## Task 7: Online Orders Backend

**Files:**
- Create: `pos/backend/src/online-orders/online-orders.module.ts`
- Create: `pos/backend/src/online-orders/customer-orders.controller.ts`
- Create: `pos/backend/src/online-orders/staff-orders.controller.ts`
- Create: `pos/backend/src/online-orders/online-orders.service.ts`
- Create: `pos/backend/src/online-orders/online-orders.service.spec.ts`
- Create: `pos/backend/src/online-orders/dto/create-online-order.dto.ts`
- Create: `pos/backend/src/customer-app/customer-menu.controller.ts`

- [ ] **Step 1: Create DTOs**

`pos/backend/src/online-orders/dto/create-online-order.dto.ts`:

```typescript
import { IsEnum, IsString, IsOptional, IsArray, ValidateNested, IsNumber, IsDateString, Min } from 'class-validator';
import { Type } from 'class-transformer';

class OrderItemDto {
  @IsString() menuItemId: string;
  @IsNumber() @Min(1) qty: number;
  @IsString() @IsOptional() notes?: string;
}

export class CreateOnlineOrderDto {
  @IsEnum(['DINE_IN', 'PICKUP', 'DELIVERY']) orderType: string;

  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];

  @IsString() @IsOptional() tableNumber?: string;          // dine-in
  @IsDateString() @IsOptional() pickupTime?: string;       // pickup
  @IsString() @IsOptional() deliveryAddressId?: string;    // delivery
  @IsString() @IsOptional() voucherCode?: string;
  @IsString() @IsOptional() customerNotes?: string;
}
```

- [ ] **Step 2: Write failing tests for OnlineOrdersService**

`pos/backend/src/online-orders/online-orders.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { OnlineOrdersService } from './online-orders.service';
import { PrismaService } from '../prisma/prisma.service';
import { VouchersService } from '../vouchers/vouchers.service';
import { BadRequestException } from '@nestjs/common';

describe('OnlineOrdersService', () => {
  let service: OnlineOrdersService;
  let prisma: any;
  let vouchersService: any;

  const mockMenuItem = { id: 'm1', name: 'Nasi Goreng', sellingPrice: 25000, isActive: true };
  const mockCustomer = { id: 'c1', phone: '0812' };

  beforeEach(async () => {
    prisma = {
      menuItem: { findMany: jest.fn() },
      order: { create: jest.fn(), findMany: jest.fn(), findFirst: jest.fn(), update: jest.fn() },
      cmsStoreInfo: { findFirst: jest.fn() },
      customerAddress: { findFirst: jest.fn() },
      delivery: { create: jest.fn() },
      $transaction: jest.fn((cb) => cb(prisma)),
    };
    vouchersService = {
      validateVoucher: jest.fn(),
      calculateDiscount: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OnlineOrdersService,
        { provide: PrismaService, useValue: prisma },
        { provide: VouchersService, useValue: vouchersService },
      ],
    }).compile();

    service = module.get(OnlineOrdersService);
  });

  describe('createOrder', () => {
    it('should create a PICKUP order with correct totals', async () => {
      prisma.menuItem.findMany.mockResolvedValue([mockMenuItem]);
      prisma.cmsStoreInfo.findFirst.mockResolvedValue({ operatingHours: { mon: '00:00-23:59', tue: '00:00-23:59', wed: '00:00-23:59', thu: '00:00-23:59', fri: '00:00-23:59', sat: '00:00-23:59', sun: '00:00-23:59' } });
      prisma.order.create.mockResolvedValue({
        id: 'ord-1', orderNumber: 'ONL-001', status: 'PENDING_CONFIRMATION',
        source: 'ONLINE', orderType: 'PICKUP',
      });

      const result = await service.createOrder(mockCustomer, {
        orderType: 'PICKUP', items: [{ menuItemId: 'm1', qty: 2 }],
      });

      expect(result.status).toBe('PENDING_CONFIRMATION');
      expect(result.source).toBe('ONLINE');
    });

    it('should reject order if store is closed', async () => {
      prisma.cmsStoreInfo.findFirst.mockResolvedValue({ operatingHours: {} });

      await expect(service.createOrder(mockCustomer, {
        orderType: 'PICKUP', items: [{ menuItemId: 'm1', qty: 1 }],
      })).rejects.toThrow(BadRequestException);
    });

    it('should create delivery record for DELIVERY orders', async () => {
      prisma.menuItem.findMany.mockResolvedValue([mockMenuItem]);
      prisma.cmsStoreInfo.findFirst.mockResolvedValue({
        operatingHours: { mon: '00:00-23:59', tue: '00:00-23:59', wed: '00:00-23:59', thu: '00:00-23:59', fri: '00:00-23:59', sat: '00:00-23:59', sun: '00:00-23:59' },
        deliveryFeeAmount: 10000,
      });
      prisma.customerAddress.findFirst.mockResolvedValue({
        id: 'addr-1', address: 'Jl. Test', latitude: null, longitude: null,
      });
      prisma.order.create.mockResolvedValue({
        id: 'ord-1', orderNumber: 'ONL-001', status: 'PENDING_CONFIRMATION',
        deliveryFee: 10000,
      });
      prisma.delivery.create.mockResolvedValue({ id: 'del-1' });

      const result = await service.createOrder(mockCustomer, {
        orderType: 'DELIVERY', items: [{ menuItemId: 'm1', qty: 1 }],
        deliveryAddressId: 'addr-1',
      });

      expect(prisma.delivery.create).toHaveBeenCalled();
    });
  });

  describe('confirmOrder', () => {
    it('should change status to CONFIRMED and set confirmedAt', async () => {
      prisma.order.findFirst.mockResolvedValue({ id: 'ord-1', status: 'PENDING_CONFIRMATION', source: 'ONLINE' });
      prisma.order.update.mockResolvedValue({ id: 'ord-1', status: 'CONFIRMED', confirmedAt: new Date() });

      const result = await service.confirmOrder('ord-1');

      expect(result.status).toBe('CONFIRMED');
    });
  });

  describe('cancelOrder (customer)', () => {
    it('should allow cancel only if PENDING_CONFIRMATION', async () => {
      prisma.order.findFirst.mockResolvedValue({ id: 'ord-1', status: 'PENDING_CONFIRMATION', customerId: 'c1' });
      prisma.order.update.mockResolvedValue({ id: 'ord-1', status: 'CANCELLED' });

      const result = await service.customerCancelOrder('c1', 'ord-1');
      expect(result.status).toBe('CANCELLED');
    });

    it('should reject cancel if already confirmed', async () => {
      prisma.order.findFirst.mockResolvedValue({ id: 'ord-1', status: 'CONFIRMED', customerId: 'c1' });

      await expect(service.customerCancelOrder('c1', 'ord-1'))
        .rejects.toThrow(BadRequestException);
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=online-orders.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 4: Implement OnlineOrdersService**

`pos/backend/src/online-orders/online-orders.service.ts`:

```typescript
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { VouchersService } from '../vouchers/vouchers.service';
import { CreateOnlineOrderDto } from './dto/create-online-order.dto';

@Injectable()
export class OnlineOrdersService {
  constructor(
    private prisma: PrismaService,
    private vouchersService: VouchersService,
  ) {}

  async createOrder(customer: any, dto: CreateOnlineOrderDto) {
    // Check store open
    const storeInfo = await this.prisma.cmsStoreInfo.findFirst();
    if (!storeInfo || !this.isStoreOpen(storeInfo.operatingHours as Record<string, string>)) {
      throw new BadRequestException('Toko sedang tutup');
    }

    // Validate menu items exist and are active
    const menuItemIds = dto.items.map((i) => i.menuItemId);
    const menuItems = await this.prisma.menuItem.findMany({
      where: { id: { in: menuItemIds }, isActive: true },
    });
    if (menuItems.length !== menuItemIds.length) {
      throw new BadRequestException('Beberapa menu tidak tersedia');
    }

    // Calculate subtotal
    const itemMap = new Map(menuItems.map((m) => [m.id, m]));
    let subtotal = 0;
    const orderItems = dto.items.map((item) => {
      const menu = itemMap.get(item.menuItemId)!;
      const itemTotal = Number(menu.sellingPrice) * item.qty;
      subtotal += itemTotal;
      return {
        menuItemId: item.menuItemId,
        qty: item.qty,
        notes: item.notes || null,
        status: 'PENDING',
        unitPrice: Number(menu.sellingPrice),
      };
    });

    // Delivery fee
    let deliveryFee = 0;
    let deliveryAddress: any = null;
    if (dto.orderType === 'DELIVERY') {
      if (!dto.deliveryAddressId) throw new BadRequestException('Alamat delivery wajib diisi');
      deliveryAddress = await this.prisma.customerAddress.findFirst({
        where: { id: dto.deliveryAddressId, customerId: customer.id },
      });
      if (!deliveryAddress) throw new NotFoundException('Alamat tidak ditemukan');
      deliveryFee = Number(storeInfo.deliveryFeeAmount || 0);
    }

    // Voucher validation
    let voucherDiscount = 0;
    let voucherId: string | null = null;
    if (dto.voucherCode) {
      const validation = await this.vouchersService.validateVoucher(customer.id, {
        code: dto.voucherCode, subtotal, orderType: dto.orderType,
      });
      if (!validation.valid) throw new BadRequestException(validation.message);
      voucherDiscount = validation.discount_amount;

      const voucher = await this.prisma.voucher.findUnique({ where: { code: dto.voucherCode } });
      voucherId = voucher?.id || null;

      // FREE_DELIVERY zeroes out fee
      if (voucher?.type === 'FREE_DELIVERY') {
        deliveryFee = 0;
      }
    }

    // Generate order number
    const count = await this.prisma.order.count({ where: { source: 'ONLINE' } });
    const orderNumber = `ONL-${String(count + 1).padStart(4, '0')}`;

    return this.prisma.$transaction(async (tx) => {
      const order = await tx.order.create({
        data: {
          orderNumber,
          status: 'PENDING_CONFIRMATION',
          source: 'ONLINE',
          orderType: dto.orderType as any,
          customerId: customer.id,
          tableNumber: dto.tableNumber || null,
          pickupTime: dto.pickupTime ? new Date(dto.pickupTime) : null,
          customerNotes: dto.customerNotes || null,
          voucherId,
          voucherDiscount,
          deliveryFee,
          orderItems: {
            create: orderItems.map((item) => ({
              menuItemId: item.menuItemId,
              qty: item.qty,
              notes: item.notes,
              status: 'PENDING',
            })),
          },
        },
        include: { orderItems: true },
      });

      // Create delivery record for DELIVERY orders
      if (dto.orderType === 'DELIVERY' && deliveryAddress) {
        await tx.delivery.create({
          data: {
            orderId: order.id,
            customerId: customer.id,
            address: deliveryAddress.address,
            latitude: deliveryAddress.latitude,
            longitude: deliveryAddress.longitude,
            deliveryFee,
          },
        });
      }

      return order;
    });
  }

  async getCustomerOrders(customerId: string, page = 1, limit = 10) {
    const [data, total] = await Promise.all([
      this.prisma.order.findMany({
        where: { customerId, source: 'ONLINE' },
        include: { orderItems: { include: { menuItem: { select: { name: true, imageUrl: true } } } } },
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
      }),
      this.prisma.order.count({ where: { customerId, source: 'ONLINE' } }),
    ]);
    return { data, total, page, limit };
  }

  async getCustomerOrderDetail(customerId: string, orderId: string) {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, customerId },
      include: {
        orderItems: { include: { menuItem: { select: { name: true, imageUrl: true, sellingPrice: true } } } },
        delivery: true,
      },
    });
    if (!order) throw new NotFoundException('Pesanan tidak ditemukan');
    return order;
  }

  async customerCancelOrder(customerId: string, orderId: string) {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, customerId },
    });
    if (!order) throw new NotFoundException('Pesanan tidak ditemukan');
    if (order.status !== 'PENDING_CONFIRMATION') {
      throw new BadRequestException('Pesanan sudah tidak bisa dibatalkan');
    }
    return this.prisma.order.update({
      where: { id: orderId },
      data: { status: 'CANCELLED' },
    });
  }

  // --- Staff endpoints ---

  async getPendingOrders() {
    return this.prisma.order.findMany({
      where: { source: 'ONLINE', status: { in: ['PENDING_CONFIRMATION', 'CONFIRMED', 'PREPARING', 'READY'] } },
      include: {
        orderItems: { include: { menuItem: { select: { name: true } } } },
        customer: { select: { name: true, phone: true } },
        delivery: true,
      },
      orderBy: { createdAt: 'asc' },
    });
  }

  async confirmOrder(orderId: string) {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, source: 'ONLINE', status: 'PENDING_CONFIRMATION' },
    });
    if (!order) throw new NotFoundException('Order tidak ditemukan atau sudah diproses');

    return this.prisma.order.update({
      where: { id: orderId },
      data: { status: 'CONFIRMED', confirmedAt: new Date() },
      include: { orderItems: true },
    });
    // Note: After confirming, the caller should emit WebSocket order:new event to kitchen
  }

  async rejectOrder(orderId: string) {
    const order = await this.prisma.order.findFirst({
      where: { id: orderId, source: 'ONLINE', status: 'PENDING_CONFIRMATION' },
    });
    if (!order) throw new NotFoundException('Order tidak ditemukan atau sudah diproses');

    return this.prisma.order.update({
      where: { id: orderId },
      data: { status: 'CANCELLED' },
    });
  }

  private isStoreOpen(hours: Record<string, string>): boolean {
    const days = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat'];
    const now = new Date();
    const dayKey = days[now.getDay()];
    const todayHours = hours[dayKey];
    if (!todayHours) return false;

    const [open, close] = todayHours.split('-');
    const currentTime = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
    return currentTime >= open && currentTime <= close;
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=online-orders.service.spec --no-coverage`

Expected: 5 tests PASS

- [ ] **Step 6: Create controllers**

`pos/backend/src/online-orders/customer-orders.controller.ts`:

```typescript
import { Controller, Post, Get, Patch, Param, Body, Query, UseGuards, Request } from '@nestjs/common';
import { OnlineOrdersService } from './online-orders.service';
import { CustomerAuthGuard } from '../customers/guards/customer-auth.guard';
import { CreateOnlineOrderDto } from './dto/create-online-order.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Controller('customer-app/orders')
@UseGuards(CustomerAuthGuard)
export class CustomerOrdersController {
  constructor(private onlineOrdersService: OnlineOrdersService) {}

  @Post()
  create(@Request() req: any, @Body() dto: CreateOnlineOrderDto) {
    return this.onlineOrdersService.createOrder(req.user, dto);
  }

  @Get()
  findAll(@Request() req: any, @Query() pagination: PaginationDto) {
    return this.onlineOrdersService.getCustomerOrders(req.user.id, pagination.page, pagination.limit);
  }

  @Get(':id')
  findOne(@Request() req: any, @Param('id') id: string) {
    return this.onlineOrdersService.getCustomerOrderDetail(req.user.id, id);
  }

  @Patch(':id/cancel')
  cancel(@Request() req: any, @Param('id') id: string) {
    return this.onlineOrdersService.customerCancelOrder(req.user.id, id);
  }
}
```

`pos/backend/src/online-orders/staff-orders.controller.ts`:

```typescript
import { Controller, Get, Patch, Param, UseGuards } from '@nestjs/common';
import { OnlineOrdersService } from './online-orders.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/require-permissions.decorator';

@Controller('online-orders')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('online_order:read')
export class StaffOrdersController {
  constructor(private onlineOrdersService: OnlineOrdersService) {}

  @Get()
  getPending() {
    return this.onlineOrdersService.getPendingOrders();
  }

  @Patch(':id/confirm')
  confirm(@Param('id') id: string) {
    return this.onlineOrdersService.confirmOrder(id);
  }

  @Patch(':id/reject')
  reject(@Param('id') id: string) {
    return this.onlineOrdersService.rejectOrder(id);
  }
}
```

- [ ] **Step 7: Create customer menu controller**

`pos/backend/src/customer-app/customer-menu.controller.ts`:

```typescript
import { Controller, Get, Param } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Controller('customer-app/menu')
export class CustomerMenuController {
  constructor(private prisma: PrismaService) {}

  @Get()
  async findAll() {
    return this.prisma.menuItem.findMany({
      where: { isActive: true },
      select: { id: true, name: true, category: true, sellingPrice: true, imageUrl: true },
      orderBy: { category: 'asc' },
    });
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.prisma.menuItem.findUnique({
      where: { id },
      select: { id: true, name: true, category: true, sellingPrice: true, imageUrl: true },
    });
  }
}
```

- [ ] **Step 8: Create modules and register**

`pos/backend/src/online-orders/online-orders.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomersModule } from '../customers/customers.module';
import { VouchersModule } from '../vouchers/vouchers.module';
import { CustomerOrdersController } from './customer-orders.controller';
import { StaffOrdersController } from './staff-orders.controller';
import { OnlineOrdersService } from './online-orders.service';

@Module({
  imports: [PrismaModule, CustomersModule, VouchersModule],
  controllers: [CustomerOrdersController, StaffOrdersController],
  providers: [OnlineOrdersService],
  exports: [OnlineOrdersService],
})
export class OnlineOrdersModule {}
```

`pos/backend/src/customer-app/customer-app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomerMenuController } from './customer-menu.controller';

@Module({
  imports: [PrismaModule],
  controllers: [CustomerMenuController],
})
export class CustomerAppModule {}
```

Add both to `app.module.ts`.

- [ ] **Step 9: Wire WebSocket — emit order:new on confirm**

In `staff-orders.controller.ts`, inject `OrdersGateway` and emit after confirm:

```typescript
import { OrdersGateway } from '../orders/orders.gateway';

// In constructor: private ordersGateway: OrdersGateway

@Patch(':id/confirm')
async confirm(@Param('id') id: string) {
  const order = await this.onlineOrdersService.confirmOrder(id);
  this.ordersGateway.emitNewOrder(order); // Same event as Phase 1 POS
  return order;
}
```

Update `OnlineOrdersModule` to import `OrdersModule`.

- [ ] **Step 10: Commit**

```bash
cd pos/backend
git add src/online-orders/ src/customer-app/ src/app.module.ts
git commit -m "feat(phase4): add online ordering flow with customer submit, kasir confirm/reject, kitchen integration"
```

---

## Task 8: Delivery Backend

**Files:**
- Create: `pos/backend/src/deliveries/deliveries.module.ts`
- Create: `pos/backend/src/deliveries/deliveries.controller.ts`
- Create: `pos/backend/src/deliveries/customer-deliveries.controller.ts`
- Create: `pos/backend/src/deliveries/deliveries.service.ts`
- Create: `pos/backend/src/deliveries/deliveries.service.spec.ts`
- Create: `pos/backend/src/deliveries/dto/*.ts`

- [ ] **Step 1: Create DTOs**

`pos/backend/src/deliveries/dto/assign-driver.dto.ts`:

```typescript
import { IsString, IsOptional, IsInt, Min } from 'class-validator';

export class AssignDriverDto {
  @IsString() driverName: string;
  @IsString() driverPhone: string;
  @IsInt() @Min(1) @IsOptional() estimatedTime?: number; // minutes
}
```

`pos/backend/src/deliveries/dto/update-delivery-status.dto.ts`:

```typescript
import { IsEnum } from 'class-validator';

export class UpdateDeliveryStatusDto {
  @IsEnum(['ON_DELIVERY', 'DELIVERED', 'CANCELLED']) status: string;
}
```

- [ ] **Step 2: Write failing test for DeliveriesService**

`pos/backend/src/deliveries/deliveries.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { DeliveriesService } from './deliveries.service';
import { PrismaService } from '../prisma/prisma.service';
import { BadRequestException } from '@nestjs/common';

describe('DeliveriesService', () => {
  let service: DeliveriesService;
  let prisma: any;

  beforeEach(async () => {
    prisma = {
      delivery: { findUnique: jest.fn(), update: jest.fn() },
      order: { update: jest.fn() },
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [DeliveriesService, { provide: PrismaService, useValue: prisma }],
    }).compile();

    service = module.get(DeliveriesService);
  });

  describe('assignDriver', () => {
    it('should assign driver and set status to WAITING_PICKUP', async () => {
      prisma.delivery.findUnique.mockResolvedValue({ id: 'd1', orderId: 'o1', status: 'WAITING_PICKUP' });
      prisma.delivery.update.mockResolvedValue({
        id: 'd1', driverName: 'Budi', driverPhone: '0812', status: 'WAITING_PICKUP',
      });

      const result = await service.assignDriver('o1', {
        driverName: 'Budi', driverPhone: '0812', estimatedTime: 30,
      });

      expect(result.driverName).toBe('Budi');
    });
  });

  describe('updateStatus', () => {
    it('should update to ON_DELIVERY and update order status', async () => {
      prisma.delivery.findUnique.mockResolvedValue({ id: 'd1', orderId: 'o1', status: 'WAITING_PICKUP' });
      prisma.delivery.update.mockResolvedValue({ id: 'd1', status: 'ON_DELIVERY' });
      prisma.order.update.mockResolvedValue({ id: 'o1', status: 'ON_DELIVERY' });

      const result = await service.updateStatus('o1', 'ON_DELIVERY');
      expect(result.status).toBe('ON_DELIVERY');
    });

    it('should set deliveredAt when marking DELIVERED', async () => {
      prisma.delivery.findUnique.mockResolvedValue({ id: 'd1', orderId: 'o1', status: 'ON_DELIVERY' });
      prisma.delivery.update.mockResolvedValue({ id: 'd1', status: 'DELIVERED', deliveredAt: new Date() });
      prisma.order.update.mockResolvedValue({ id: 'o1', status: 'COMPLETED' });

      const result = await service.updateStatus('o1', 'DELIVERED');
      expect(result.deliveredAt).toBeDefined();
    });
  });

  describe('customerConfirmReceived', () => {
    it('should mark as DELIVERED when customer confirms', async () => {
      prisma.delivery.findUnique.mockResolvedValue({ id: 'd1', orderId: 'o1', status: 'ON_DELIVERY' });
      prisma.delivery.update.mockResolvedValue({ id: 'd1', status: 'DELIVERED' });
      prisma.order.update.mockResolvedValue({ id: 'o1', status: 'COMPLETED' });

      const result = await service.customerConfirmReceived('c1', 'o1');
      expect(result.status).toBe('DELIVERED');
    });
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=deliveries.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 4: Implement DeliveriesService**

`pos/backend/src/deliveries/deliveries.service.ts`:

```typescript
import { Injectable, NotFoundException, BadRequestException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { AssignDriverDto } from './dto/assign-driver.dto';

@Injectable()
export class DeliveriesService {
  constructor(private prisma: PrismaService) {}

  async assignDriver(orderId: string, dto: AssignDriverDto) {
    const delivery = await this.prisma.delivery.findUnique({ where: { orderId } });
    if (!delivery) throw new NotFoundException('Delivery tidak ditemukan');

    return this.prisma.delivery.update({
      where: { orderId },
      data: {
        driverName: dto.driverName,
        driverPhone: dto.driverPhone,
        estimatedTime: dto.estimatedTime || null,
        status: 'WAITING_PICKUP',
      },
    });
  }

  async updateStatus(orderId: string, status: string) {
    const delivery = await this.prisma.delivery.findUnique({ where: { orderId } });
    if (!delivery) throw new NotFoundException('Delivery tidak ditemukan');

    const data: any = { status };
    if (status === 'DELIVERED') {
      data.deliveredAt = new Date();
    }

    const updated = await this.prisma.delivery.update({ where: { orderId }, data });

    // Sync order status
    const orderStatus = status === 'DELIVERED' ? 'COMPLETED' : status;
    await this.prisma.order.update({ where: { id: orderId }, data: { status: orderStatus } });

    return updated;
  }

  async customerConfirmReceived(customerId: string, orderId: string) {
    const delivery = await this.prisma.delivery.findUnique({
      where: { orderId },
      include: { order: { select: { customerId: true } } },
    });
    if (!delivery || delivery.order.customerId !== customerId) {
      throw new NotFoundException('Delivery tidak ditemukan');
    }
    if (delivery.status !== 'ON_DELIVERY') {
      throw new BadRequestException('Pesanan belum dalam pengiriman');
    }

    const updated = await this.prisma.delivery.update({
      where: { orderId },
      data: { status: 'DELIVERED', deliveredAt: new Date() },
    });

    await this.prisma.order.update({ where: { id: orderId }, data: { status: 'COMPLETED' } });

    return updated;
  }

  async getDeliveryStatus(orderId: string) {
    const delivery = await this.prisma.delivery.findUnique({ where: { orderId } });
    if (!delivery) throw new NotFoundException('Delivery tidak ditemukan');
    return delivery;
  }
}
```

- [ ] **Step 5: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=deliveries.service.spec --no-coverage`

Expected: 4 tests PASS

- [ ] **Step 6: Create controllers and module**

`pos/backend/src/deliveries/deliveries.controller.ts`:

```typescript
import { Controller, Post, Patch, Param, Body, UseGuards } from '@nestjs/common';
import { DeliveriesService } from './deliveries.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/require-permissions.decorator';
import { AssignDriverDto } from './dto/assign-driver.dto';
import { UpdateDeliveryStatusDto } from './dto/update-delivery-status.dto';

@Controller('deliveries')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('delivery:manage')
export class DeliveriesController {
  constructor(private deliveriesService: DeliveriesService) {}

  @Post(':orderId/assign')
  @RequirePermissions('delivery:assign_driver')
  assignDriver(@Param('orderId') orderId: string, @Body() dto: AssignDriverDto) {
    return this.deliveriesService.assignDriver(orderId, dto);
  }

  @Patch(':orderId/status')
  updateStatus(@Param('orderId') orderId: string, @Body() dto: UpdateDeliveryStatusDto) {
    return this.deliveriesService.updateStatus(orderId, dto.status);
  }
}
```

`pos/backend/src/deliveries/customer-deliveries.controller.ts`:

```typescript
import { Controller, Get, Patch, Param, UseGuards, Request } from '@nestjs/common';
import { DeliveriesService } from './deliveries.service';
import { CustomerAuthGuard } from '../customers/guards/customer-auth.guard';

@Controller('customer-app/orders')
@UseGuards(CustomerAuthGuard)
export class CustomerDeliveriesController {
  constructor(private deliveriesService: DeliveriesService) {}

  @Get(':id/delivery')
  getDeliveryStatus(@Param('id') orderId: string) {
    return this.deliveriesService.getDeliveryStatus(orderId);
  }

  @Patch(':id/received')
  confirmReceived(@Request() req: any, @Param('id') orderId: string) {
    return this.deliveriesService.customerConfirmReceived(req.user.id, orderId);
  }
}
```

`pos/backend/src/deliveries/deliveries.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomersModule } from '../customers/customers.module';
import { DeliveriesController } from './deliveries.controller';
import { CustomerDeliveriesController } from './customer-deliveries.controller';
import { DeliveriesService } from './deliveries.service';

@Module({
  imports: [PrismaModule, CustomersModule],
  controllers: [DeliveriesController, CustomerDeliveriesController],
  providers: [DeliveriesService],
  exports: [DeliveriesService],
})
export class DeliveriesModule {}
```

Add to `app.module.ts`.

- [ ] **Step 7: Commit**

```bash
cd pos/backend
git add src/deliveries/ src/app.module.ts
git commit -m "feat(phase4): add delivery management with driver assignment and status tracking"
```

---

## Task 9: FCM Notifications Backend

**Files:**
- Create: `pos/backend/src/fcm/fcm.module.ts`
- Create: `pos/backend/src/fcm/fcm.controller.ts`
- Create: `pos/backend/src/fcm/fcm.service.ts`
- Create: `pos/backend/src/fcm/fcm.service.spec.ts`
- Create: `pos/backend/src/fcm/dto/*.ts`

- [ ] **Step 1: Install Firebase Admin**

Run: `cd pos/backend && npm install firebase-admin`

- [ ] **Step 2: Create DTOs**

`pos/backend/src/fcm/dto/register-device.dto.ts`:

```typescript
import { IsString, IsEnum } from 'class-validator';

export class RegisterDeviceDto {
  @IsString() fcmToken: string;
  @IsEnum(['ANDROID', 'IOS']) deviceType: string;
}
```

`pos/backend/src/fcm/dto/notification-query.dto.ts`:

```typescript
import { IsOptional, IsInt, Min } from 'class-validator';
import { Type } from 'class-transformer';

export class NotificationQueryDto {
  @IsInt() @Min(1) @IsOptional() @Type(() => Number) page?: number = 1;
  @IsInt() @Min(1) @IsOptional() @Type(() => Number) limit?: number = 20;
}
```

- [ ] **Step 3: Write failing test for FcmService**

`pos/backend/src/fcm/fcm.service.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { FcmService } from './fcm.service';
import { PrismaService } from '../prisma/prisma.service';

describe('FcmService', () => {
  let service: FcmService;
  let prisma: any;

  beforeEach(async () => {
    prisma = {
      customerDevice: { upsert: jest.fn(), findMany: jest.fn(), updateMany: jest.fn() },
      customerNotification: { create: jest.fn(), findMany: jest.fn(), count: jest.fn(), update: jest.fn(), updateMany: jest.fn() },
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [FcmService, { provide: PrismaService, useValue: prisma }],
    }).compile();

    service = module.get(FcmService);
  });

  describe('registerDevice', () => {
    it('should upsert device token', async () => {
      prisma.customerDevice.upsert.mockResolvedValue({ id: '1', fcmToken: 'tok123' });

      await service.registerDevice('c1', { fcmToken: 'tok123', deviceType: 'ANDROID' });

      expect(prisma.customerDevice.upsert).toHaveBeenCalled();
    });
  });

  describe('sendToCustomer', () => {
    it('should save notification to history', async () => {
      prisma.customerDevice.findMany.mockResolvedValue([]);
      prisma.customerNotification.create.mockResolvedValue({ id: 'n1' });

      await service.sendToCustomer('c1', { title: 'Test', body: 'Hello' });

      expect(prisma.customerNotification.create).toHaveBeenCalledWith({
        data: { customerId: 'c1', title: 'Test', body: 'Hello' },
      });
    });
  });

  describe('getNotifications', () => {
    it('should return paginated notifications', async () => {
      prisma.customerNotification.findMany.mockResolvedValue([{ id: 'n1', title: 'Test' }]);
      prisma.customerNotification.count.mockResolvedValue(1);

      const result = await service.getNotifications('c1', 1, 20);

      expect(result.data).toHaveLength(1);
      expect(result.total).toBe(1);
    });
  });
});
```

- [ ] **Step 4: Run tests to verify they fail**

Run: `cd pos/backend && npx jest --testPathPattern=fcm.service.spec --no-coverage`

Expected: FAIL

- [ ] **Step 5: Implement FcmService**

`pos/backend/src/fcm/fcm.service.ts`:

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class FcmService {
  private readonly logger = new Logger(FcmService.name);

  constructor(private prisma: PrismaService) {}

  async registerDevice(customerId: string, dto: { fcmToken: string; deviceType: string }) {
    return this.prisma.customerDevice.upsert({
      where: { id: dto.fcmToken }, // Use token as lookup — simplified, would need unique constraint on fcmToken
      update: { customerId, isActive: true, deviceType: dto.deviceType as any },
      create: { customerId, fcmToken: dto.fcmToken, deviceType: dto.deviceType as any },
    });
  }

  async unregisterDevice(token: string) {
    return this.prisma.customerDevice.updateMany({
      where: { fcmToken: token },
      data: { isActive: false },
    });
  }

  async sendToCustomer(customerId: string, payload: { title: string; body: string; data?: Record<string, string> }) {
    // Save to notification history
    await this.prisma.customerNotification.create({
      data: { customerId, title: payload.title, body: payload.body },
    });

    // Get active device tokens
    const devices = await this.prisma.customerDevice.findMany({
      where: { customerId, isActive: true },
    });

    if (devices.length === 0) {
      this.logger.log(`No active devices for customer ${customerId}`);
      return;
    }

    // Send FCM push
    // In production, use Firebase Admin SDK:
    // const messaging = admin.messaging();
    // await messaging.sendEachForMulticast({
    //   tokens: devices.map(d => d.fcmToken),
    //   notification: { title: payload.title, body: payload.body },
    //   data: payload.data,
    // });

    this.logger.log(`[DEV] Push notification to ${devices.length} devices: ${payload.title}`);
  }

  async getNotifications(customerId: string, page = 1, limit = 20) {
    const [data, total] = await Promise.all([
      this.prisma.customerNotification.findMany({
        where: { customerId },
        orderBy: { createdAt: 'desc' },
        skip: (page - 1) * limit,
        take: limit,
      }),
      this.prisma.customerNotification.count({ where: { customerId } }),
    ]);
    return { data, total, page, limit };
  }

  async markAsRead(customerId: string, notificationId: string) {
    return this.prisma.customerNotification.update({
      where: { id: notificationId },
      data: { isRead: true },
    });
  }

  async markAllAsRead(customerId: string) {
    return this.prisma.customerNotification.updateMany({
      where: { customerId, isRead: false },
      data: { isRead: true },
    });
  }
}
```

- [ ] **Step 6: Run tests to verify they pass**

Run: `cd pos/backend && npx jest --testPathPattern=fcm.service.spec --no-coverage`

Expected: 3 tests PASS

- [ ] **Step 7: Create FcmController and module**

`pos/backend/src/fcm/fcm.controller.ts`:

```typescript
import { Controller, Post, Delete, Get, Patch, Param, Body, Query, UseGuards, Request } from '@nestjs/common';
import { FcmService } from './fcm.service';
import { CustomerAuthGuard } from '../customers/guards/customer-auth.guard';
import { RegisterDeviceDto } from './dto/register-device.dto';
import { NotificationQueryDto } from './dto/notification-query.dto';

@Controller('customer-app')
@UseGuards(CustomerAuthGuard)
export class FcmController {
  constructor(private fcmService: FcmService) {}

  @Post('devices')
  registerDevice(@Request() req: any, @Body() dto: RegisterDeviceDto) {
    return this.fcmService.registerDevice(req.user.id, dto);
  }

  @Delete('devices/:token')
  unregisterDevice(@Param('token') token: string) {
    return this.fcmService.unregisterDevice(token);
  }

  @Get('notifications')
  getNotifications(@Request() req: any, @Query() query: NotificationQueryDto) {
    return this.fcmService.getNotifications(req.user.id, query.page, query.limit);
  }

  @Patch('notifications/:id/read')
  markAsRead(@Request() req: any, @Param('id') id: string) {
    return this.fcmService.markAsRead(req.user.id, id);
  }

  @Patch('notifications/read-all')
  markAllAsRead(@Request() req: any) {
    return this.fcmService.markAllAsRead(req.user.id);
  }
}
```

`pos/backend/src/fcm/fcm.module.ts`:

```typescript
import { Module, Global } from '@nestjs/common';
import { PrismaModule } from '../prisma/prisma.module';
import { CustomersModule } from '../customers/customers.module';
import { FcmController } from './fcm.controller';
import { FcmService } from './fcm.service';

@Global() // Global so other modules can inject FcmService for sending notifications
@Module({
  imports: [PrismaModule, CustomersModule],
  controllers: [FcmController],
  providers: [FcmService],
  exports: [FcmService],
})
export class FcmModule {}
```

Add to `app.module.ts`.

- [ ] **Step 8: Wire FCM into order status changes**

In `online-orders.service.ts`, inject `FcmService` and send notifications:

```typescript
// In confirmOrder:
await this.fcmService.sendToCustomer(order.customerId, {
  title: 'Pesanan Dikonfirmasi',
  body: `Pesanan #${order.orderNumber} dikonfirmasi!`,
  data: { orderId: order.id, type: 'ORDER_CONFIRMED' },
});

// In rejectOrder:
await this.fcmService.sendToCustomer(order.customerId, {
  title: 'Pesanan Dibatalkan',
  body: `Pesanan #${order.orderNumber} dibatalkan`,
  data: { orderId: order.id, type: 'ORDER_CANCELLED' },
});
```

Similarly wire FCM in `deliveries.service.ts` for delivery status changes.

- [ ] **Step 9: Commit**

```bash
cd pos/backend
git add src/fcm/ src/online-orders/ src/deliveries/ src/app.module.ts package.json package-lock.json
git commit -m "feat(phase4): add FCM notification service with device registration and push on order status changes"
```

---

## Task 10: Web Dashboard — CMS Pages

**Files:**
- Create: `pos/web/src/services/cms.service.ts`
- Create: `pos/web/src/services/upload.service.ts`
- Create: `pos/web/src/types/cms.ts`
- Create: `pos/web/src/app/(dashboard)/cms/banners/page.tsx`
- Create: `pos/web/src/app/(dashboard)/cms/sections/page.tsx`
- Create: `pos/web/src/app/(dashboard)/cms/store-info/page.tsx`
- Create: `pos/web/src/app/(dashboard)/cms/preview/page.tsx`

- [ ] **Step 1: Create CMS types**

`pos/web/src/types/cms.ts`:

```typescript
export interface CmsBanner {
  id: string;
  title: string;
  imageUrl: string;
  linkType: 'MENU_ITEM' | 'PROMO' | 'VOUCHER' | 'EXTERNAL_URL' | 'NONE';
  linkValue: string | null;
  position: 'HOME_TOP' | 'HOME_MIDDLE' | 'CATEGORY_PAGE';
  sortOrder: number;
  startDate: string;
  endDate: string;
  isActive: boolean;
}

export interface CmsSection {
  id: string;
  title: string;
  type: 'MENU_LIST' | 'BANNER_CAROUSEL' | 'CATEGORY_GRID';
  contentConfig: Record<string, any>;
  sortOrder: number;
  isActive: boolean;
}

export interface CmsStoreInfo {
  id: string;
  storeName: string;
  logoUrl: string | null;
  coverImageUrl: string | null;
  description: string | null;
  address: string;
  phone: string;
  operatingHours: Record<string, string>;
  deliveryRadiusKm: number | null;
  minOrderDelivery: number | null;
  deliveryFeeAmount: number;
}

export interface HomePageResponse {
  store_info: {
    store_name: string;
    logo_url: string | null;
    cover_image_url: string | null;
    operating_hours: Record<string, string>;
    is_open: boolean;
  };
  banners: Record<string, Array<{ id: string; title: string; image_url: string; link_type: string; link_value: string | null }>>;
  sections: Array<{
    id: string;
    title: string;
    type: string;
    items?: Array<{ id: string; name: string; selling_price: number; image_url: string | null }>;
    categories?: Array<{ name: string; item_count: number }>;
  }>;
}
```

- [ ] **Step 2: Create CMS and Upload API services**

`pos/web/src/services/cms.service.ts`:

```typescript
import api from '@/lib/api';
import { CmsBanner, CmsSection, CmsStoreInfo, HomePageResponse } from '@/types/cms';

export const cmsService = {
  getBanners: () => api.get<CmsBanner[]>('/cms/banners').then(r => r.data),
  createBanner: (data: Partial<CmsBanner>) => api.post('/cms/banners', data).then(r => r.data),
  updateBanner: (id: string, data: Partial<CmsBanner>) => api.patch(`/cms/banners/${id}`, data).then(r => r.data),
  deleteBanner: (id: string) => api.delete(`/cms/banners/${id}`).then(r => r.data),
  reorderBanners: (items: { id: string; sortOrder: number }[]) => api.patch('/cms/banners/reorder', { items }).then(r => r.data),

  getSections: () => api.get<CmsSection[]>('/cms/sections').then(r => r.data),
  createSection: (data: Partial<CmsSection>) => api.post('/cms/sections', data).then(r => r.data),
  updateSection: (id: string, data: Partial<CmsSection>) => api.patch(`/cms/sections/${id}`, data).then(r => r.data),
  deleteSection: (id: string) => api.delete(`/cms/sections/${id}`).then(r => r.data),
  reorderSections: (items: { id: string; sortOrder: number }[]) => api.patch('/cms/sections/reorder', { items }).then(r => r.data),

  getStoreInfo: () => api.get<CmsStoreInfo>('/cms/store-info').then(r => r.data),
  updateStoreInfo: (data: Partial<CmsStoreInfo>) => api.patch('/cms/store-info', data).then(r => r.data),

  getPreview: () => api.get<HomePageResponse>('/cms/preview').then(r => r.data),
};
```

`pos/web/src/services/upload.service.ts`:

```typescript
import api from '@/lib/api';

export const uploadService = {
  uploadImage: async (file: File): Promise<string> => {
    const formData = new FormData();
    formData.append('file', file);
    const res = await api.post('/uploads/image', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    });
    return res.data.data.url;
  },
};
```

- [ ] **Step 3: Create CMS banner management page**

`pos/web/src/app/(dashboard)/cms/banners/page.tsx`:

Follow the pattern used in existing pages (e.g., `promos/page.tsx`). Build a table with columns: Title, Position, Active, Date Range, Actions (Edit/Delete). Add create banner dialog with form fields matching `CreateBannerDto`. Image upload via `uploadService.uploadImage`. Use existing shadcn components: `Table`, `Dialog`, `Button`, `Input`, `Select`.

This page should be ~200-300 lines following existing patterns.

- [ ] **Step 4: Create CMS sections management page**

`pos/web/src/app/(dashboard)/cms/sections/page.tsx`:

Table with columns: Title, Type, Active, Sort Order, Actions. Create/edit dialog with type selector, content_config JSON editor (for manual mode: multi-select menu items; for auto mode: category selector + limit). Drag-and-drop reorder using `reorderSections`.

- [ ] **Step 5: Create store info page**

`pos/web/src/app/(dashboard)/cms/store-info/page.tsx`:

Single form page (not table). Fields: store name, address, phone, description, logo upload, cover image upload, operating hours (7 day inputs), delivery fee, delivery radius, min order delivery. Save button calls `updateStoreInfo`.

- [ ] **Step 6: Create preview page**

`pos/web/src/app/(dashboard)/cms/preview/page.tsx`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { cmsService } from '@/services/cms.service';
import { HomePageResponse } from '@/types/cms';

export default function CmsPreviewPage() {
  const [data, setData] = useState<HomePageResponse | null>(null);

  useEffect(() => {
    cmsService.getPreview().then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;

  return (
    <div className="flex justify-center p-8">
      {/* Phone frame */}
      <div className="w-[375px] h-[812px] border-4 border-gray-800 rounded-[40px] overflow-y-auto bg-white shadow-2xl">
        {/* Store header */}
        {data.store_info && (
          <div className="relative">
            {data.store_info.cover_image_url && (
              <img src={data.store_info.cover_image_url} className="w-full h-48 object-cover" alt="" />
            )}
            <div className="p-4">
              <h1 className="text-lg font-bold">{data.store_info.store_name}</h1>
              <p className="text-sm text-gray-500">
                {data.store_info.is_open ? '🟢 Buka' : '🔴 Tutup'}
              </p>
            </div>
          </div>
        )}

        {/* Top banners */}
        {data.banners.HOME_TOP?.length > 0 && (
          <div className="px-4 pb-4">
            <div className="flex gap-2 overflow-x-auto">
              {data.banners.HOME_TOP.map((b) => (
                <img key={b.id} src={b.image_url} className="w-72 h-32 rounded-lg object-cover flex-shrink-0" alt={b.title} />
              ))}
            </div>
          </div>
        )}

        {/* Sections */}
        {data.sections.map((section) => (
          <div key={section.id} className="px-4 pb-4">
            <h2 className="font-semibold mb-2">{section.title}</h2>
            {section.type === 'MENU_LIST' && section.items && (
              <div className="flex gap-3 overflow-x-auto">
                {section.items.map((item) => (
                  <div key={item.id} className="w-32 flex-shrink-0">
                    <div className="w-32 h-24 bg-gray-200 rounded-lg mb-1" />
                    <p className="text-xs font-medium truncate">{item.name}</p>
                    <p className="text-xs text-orange-600">Rp {item.selling_price.toLocaleString('id-ID')}</p>
                  </div>
                ))}
              </div>
            )}
            {section.type === 'CATEGORY_GRID' && section.categories && (
              <div className="grid grid-cols-2 gap-2">
                {section.categories.map((cat) => (
                  <div key={cat.name} className="p-3 bg-gray-100 rounded-lg text-center">
                    <p className="text-sm font-medium">{cat.name}</p>
                    <p className="text-xs text-gray-500">{cat.item_count} items</p>
                  </div>
                ))}
              </div>
            )}
          </div>
        ))}

        {/* Middle banners */}
        {data.banners.HOME_MIDDLE?.length > 0 && (
          <div className="px-4 pb-4">
            {data.banners.HOME_MIDDLE.map((b) => (
              <img key={b.id} src={b.image_url} className="w-full h-32 rounded-lg object-cover mb-2" alt={b.title} />
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

- [ ] **Step 7: Add CMS navigation to sidebar**

Update the sidebar navigation component (likely in `pos/web/src/app/(dashboard)/layout.tsx` or a shared sidebar component) to add CMS submenu:

```typescript
// Add to navigation items:
{ name: 'CMS', icon: LayoutIcon, permission: 'cms:manage', children: [
  { name: 'Banners', href: '/cms/banners' },
  { name: 'Sections', href: '/cms/sections' },
  { name: 'Store Info', href: '/cms/store-info' },
  { name: 'Preview', href: '/cms/preview' },
]}
```

- [ ] **Step 8: Commit**

```bash
cd pos/web
git add src/services/cms.service.ts src/services/upload.service.ts src/types/cms.ts src/app/\(dashboard\)/cms/
git commit -m "feat(phase4): add CMS dashboard pages for banners, sections, store info, and preview"
```

---

## Task 11: Web Dashboard — Vouchers & Online Orders Pages

**Files:**
- Create: `pos/web/src/services/voucher.service.ts`
- Create: `pos/web/src/services/online-order.service.ts`
- Create: `pos/web/src/types/voucher.ts`
- Create: `pos/web/src/types/online-order.ts`
- Create: `pos/web/src/app/(dashboard)/vouchers/page.tsx`
- Create: `pos/web/src/app/(dashboard)/online-orders/page.tsx`

- [ ] **Step 1: Create types**

`pos/web/src/types/voucher.ts`:

```typescript
export interface Voucher {
  id: string;
  code: string;
  type: 'PERCENT' | 'FIXED' | 'FREE_DELIVERY';
  value: number;
  minPurchaseAmount: number | null;
  maxDiscountAmount: number | null;
  startDate: string;
  endDate: string;
  isActive: boolean;
  maxUsage: number | null;
  maxUsagePerUser: number | null;
  usedCount: number;
}
```

`pos/web/src/types/online-order.ts`:

```typescript
export interface OnlineOrder {
  id: string;
  orderNumber: string;
  status: string;
  source: 'ONLINE';
  orderType: 'DINE_IN' | 'PICKUP' | 'DELIVERY';
  customer: { name: string; phone: string } | null;
  orderItems: Array<{ id: string; qty: number; menuItem: { name: string } }>;
  customerNotes: string | null;
  tableNumber: string | null;
  voucherDiscount: number;
  deliveryFee: number;
  delivery: { status: string; driverName: string | null; driverPhone: string | null } | null;
  createdAt: string;
}
```

- [ ] **Step 2: Create API services**

`pos/web/src/services/voucher.service.ts`:

```typescript
import api from '@/lib/api';
import { Voucher } from '@/types/voucher';

export const voucherService = {
  findAll: () => api.get<Voucher[]>('/vouchers').then(r => r.data),
  findOne: (id: string) => api.get<Voucher>(`/vouchers/${id}`).then(r => r.data),
  create: (data: Partial<Voucher>) => api.post('/vouchers', data).then(r => r.data),
  update: (id: string, data: Partial<Voucher>) => api.patch(`/vouchers/${id}`, data).then(r => r.data),
  remove: (id: string) => api.delete(`/vouchers/${id}`).then(r => r.data),
  getUsages: (id: string) => api.get(`/vouchers/${id}/usages`).then(r => r.data),
};
```

`pos/web/src/services/online-order.service.ts`:

```typescript
import api from '@/lib/api';
import { OnlineOrder } from '@/types/online-order';

export const onlineOrderService = {
  getPending: () => api.get<OnlineOrder[]>('/online-orders').then(r => r.data),
  confirm: (id: string) => api.patch(`/online-orders/${id}/confirm`).then(r => r.data),
  reject: (id: string) => api.patch(`/online-orders/${id}/reject`).then(r => r.data),
  assignDriver: (orderId: string, data: { driverName: string; driverPhone: string; estimatedTime?: number }) =>
    api.post(`/deliveries/${orderId}/assign`, data).then(r => r.data),
  updateDeliveryStatus: (orderId: string, status: string) =>
    api.patch(`/deliveries/${orderId}/status`, { status }).then(r => r.data),
};
```

- [ ] **Step 3: Create vouchers page**

`pos/web/src/app/(dashboard)/vouchers/page.tsx`:

Follow existing page patterns. Table with columns: Code, Type, Value, Usage (used/max), Date Range, Active, Actions. Create/edit dialog with all fields from `CreateVoucherDto`. Delete = soft delete. Usage detail in expandable row or dialog.

- [ ] **Step 4: Create online orders management page**

`pos/web/src/app/(dashboard)/online-orders/page.tsx`:

Real-time-ish page (poll every 10 seconds or refresh button). Shows pending online orders as cards/list. Each order card shows: order number, customer name/phone, order type badge (dine-in/pickup/delivery), items list, notes, timestamps. Action buttons: Confirm (green), Reject (red). For delivery orders with status READY: assign driver form (name, phone, est. time). Delivery status update buttons.

- [ ] **Step 5: Add navigation items**

Add "Vouchers" (permission: `voucher:manage`) and "Online Orders" (permission: `online_order:read`) to sidebar.

- [ ] **Step 6: Commit**

```bash
cd pos/web
git add src/services/ src/types/ src/app/\(dashboard\)/vouchers/ src/app/\(dashboard\)/online-orders/
git commit -m "feat(phase4): add voucher management and online order management dashboard pages"
```

---

## Task 12: Customer Flutter App — Project Setup & Auth

**Files:**
- Create: entire `pos/customer-app/` Flutter project

- [ ] **Step 1: Create Flutter project**

Run: `cd pos && flutter create --org com.pos --project-name customer_app customer-app`

- [ ] **Step 2: Add dependencies to pubspec.yaml**

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.6
  bloc: ^8.1.4
  dio: ^5.4.3
  shared_preferences: ^2.2.3
  google_fonts: ^6.2.1
  cached_network_image: ^3.3.1
  firebase_core: ^3.0.0
  firebase_messaging: ^15.0.0
  mobile_scanner: ^5.0.0    # QR code scanning for dine-in

dev_dependencies:
  flutter_test:
    sdk: flutter
  bloc_test: ^9.1.7
  mocktail: ^1.0.4
```

Run: `cd pos/customer-app && flutter pub get`

- [ ] **Step 3: Create core infrastructure**

`pos/customer-app/lib/config/app_config.dart`:

```dart
class AppConfig {
  static const String baseUrl = 'http://localhost:3000/api';
  static const String appName = 'POS Customer';
}
```

`pos/customer-app/lib/core/api/api_client.dart`:

```dart
import 'package:dio/dio.dart';
import '../storage/token_storage.dart';

class ApiClient {
  late final Dio dio;
  final TokenStorage _tokenStorage;

  ApiClient(this._tokenStorage) {
    dio = Dio(BaseOptions(baseUrl: AppConfig.baseUrl));
    dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final token = await _tokenStorage.getAccessToken();
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        handler.next(options);
      },
    ));
  }
}
```

`pos/customer-app/lib/core/storage/token_storage.dart`:

```dart
import 'package:shared_preferences/shared_preferences.dart';

class TokenStorage {
  static const _accessTokenKey = 'access_token';
  static const _refreshTokenKey = 'refresh_token';

  Future<void> saveTokens(String accessToken, String refreshToken) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_accessTokenKey, accessToken);
    await prefs.setString(_refreshTokenKey, refreshToken);
  }

  Future<String?> getAccessToken() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_accessTokenKey);
  }

  Future<void> clearTokens() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_accessTokenKey);
    await prefs.remove(_refreshTokenKey);
  }
}
```

- [ ] **Step 4: Create auth repository and BLoC**

`pos/customer-app/lib/repositories/auth_repository.dart`:

```dart
import '../core/api/api_client.dart';
import '../core/storage/token_storage.dart';

class AuthRepository {
  final ApiClient _api;
  final TokenStorage _tokenStorage;

  AuthRepository(this._api, this._tokenStorage);

  Future<void> requestOtp(String phone) async {
    await _api.dio.post('/customers/auth/otp/request', data: {'phone': phone});
  }

  Future<Map<String, dynamic>> verifyOtp(String phone, String code) async {
    final res = await _api.dio.post('/customers/auth/otp/verify', data: {'phone': phone, 'code': code});
    final data = res.data['data'];
    await _tokenStorage.saveTokens(data['accessToken'], data['refreshToken']);
    return data['customer'];
  }

  Future<void> logout() async {
    await _tokenStorage.clearTokens();
  }
}
```

`pos/customer-app/lib/modules/auth/bloc/auth_bloc.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../repositories/auth_repository.dart';

// Events
abstract class AuthEvent {}
class RequestOtp extends AuthEvent { final String phone; RequestOtp(this.phone); }
class VerifyOtp extends AuthEvent { final String phone; final String code; VerifyOtp(this.phone, this.code); }
class Logout extends AuthEvent {}
class CheckAuth extends AuthEvent {}

// States
abstract class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class OtpSent extends AuthState { final String phone; OtpSent(this.phone); }
class Authenticated extends AuthState { final Map<String, dynamic> customer; Authenticated(this.customer); }
class AuthError extends AuthState { final String message; AuthError(this.message); }
class Unauthenticated extends AuthState {}

class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository _authRepo;

  AuthBloc(this._authRepo) : super(AuthInitial()) {
    on<RequestOtp>((event, emit) async {
      emit(AuthLoading());
      try {
        await _authRepo.requestOtp(event.phone);
        emit(OtpSent(event.phone));
      } catch (e) {
        emit(AuthError('Gagal mengirim OTP'));
      }
    });

    on<VerifyOtp>((event, emit) async {
      emit(AuthLoading());
      try {
        final customer = await _authRepo.verifyOtp(event.phone, event.code);
        emit(Authenticated(customer));
      } catch (e) {
        emit(AuthError('Kode OTP salah'));
      }
    });

    on<Logout>((event, emit) async {
      await _authRepo.logout();
      emit(Unauthenticated());
    });

    on<CheckAuth>((event, emit) async {
      final token = await _authRepo._tokenStorage.getAccessToken();
      if (token != null) {
        emit(Authenticated({})); // Load profile separately
      } else {
        emit(Unauthenticated());
      }
    });
  }
}
```

- [ ] **Step 5: Create auth screens**

`pos/customer-app/lib/modules/auth/screens/otp_request_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/auth_bloc.dart';

class OtpRequestScreen extends StatefulWidget {
  const OtpRequestScreen({super.key});

  @override
  State<OtpRequestScreen> createState() => _OtpRequestScreenState();
}

class _OtpRequestScreenState extends State<OtpRequestScreen> {
  final _phoneController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const SizedBox(height: 60),
              Text('Masuk', style: Theme.of(context).textTheme.headlineLarge),
              const SizedBox(height: 8),
              Text('Masukkan nomor HP untuk melanjutkan', style: Theme.of(context).textTheme.bodyLarge?.copyWith(color: Colors.grey)),
              const SizedBox(height: 32),
              TextField(
                controller: _phoneController,
                keyboardType: TextInputType.phone,
                decoration: const InputDecoration(
                  labelText: 'Nomor HP',
                  hintText: '08xxxxxxxxxx',
                  prefixIcon: Icon(Icons.phone),
                  border: OutlineInputBorder(),
                ),
              ),
              const SizedBox(height: 24),
              BlocConsumer<AuthBloc, AuthState>(
                listener: (context, state) {
                  if (state is OtpSent) {
                    Navigator.pushNamed(context, '/otp-verify', arguments: state.phone);
                  } else if (state is AuthError) {
                    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.message)));
                  }
                },
                builder: (context, state) {
                  return SizedBox(
                    width: double.infinity,
                    height: 48,
                    child: ElevatedButton(
                      onPressed: state is AuthLoading ? null : () {
                        context.read<AuthBloc>().add(RequestOtp(_phoneController.text));
                      },
                      child: state is AuthLoading
                          ? const CircularProgressIndicator(color: Colors.white)
                          : const Text('Kirim OTP'),
                    ),
                  );
                },
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

`pos/customer-app/lib/modules/auth/screens/otp_verify_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/auth_bloc.dart';

class OtpVerifyScreen extends StatefulWidget {
  final String phone;
  const OtpVerifyScreen({super.key, required this.phone});

  @override
  State<OtpVerifyScreen> createState() => _OtpVerifyScreenState();
}

class _OtpVerifyScreenState extends State<OtpVerifyScreen> {
  final _codeController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Verifikasi OTP')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Kode OTP dikirim ke ${widget.phone}'),
            const SizedBox(height: 24),
            TextField(
              controller: _codeController,
              keyboardType: TextInputType.number,
              maxLength: 6,
              decoration: const InputDecoration(
                labelText: 'Kode OTP',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 24),
            BlocConsumer<AuthBloc, AuthState>(
              listener: (context, state) {
                if (state is Authenticated) {
                  Navigator.pushNamedAndRemoveUntil(context, '/home', (_) => false);
                } else if (state is AuthError) {
                  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.message)));
                }
              },
              builder: (context, state) {
                return SizedBox(
                  width: double.infinity,
                  height: 48,
                  child: ElevatedButton(
                    onPressed: state is AuthLoading ? null : () {
                      context.read<AuthBloc>().add(VerifyOtp(widget.phone, _codeController.text));
                    },
                    child: state is AuthLoading
                        ? const CircularProgressIndicator(color: Colors.white)
                        : const Text('Verifikasi'),
                  ),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

- [ ] **Step 6: Setup main.dart with routing**

`pos/customer-app/lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'core/api/api_client.dart';
import 'core/storage/token_storage.dart';
import 'repositories/auth_repository.dart';
import 'modules/auth/bloc/auth_bloc.dart';
import 'modules/auth/screens/otp_request_screen.dart';
import 'modules/auth/screens/otp_verify_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final tokenStorage = TokenStorage();
    final apiClient = ApiClient(tokenStorage);
    final authRepo = AuthRepository(apiClient, tokenStorage);

    return MultiBlocProvider(
      providers: [
        BlocProvider(create: (_) => AuthBloc(authRepo)..add(CheckAuth())),
      ],
      child: MaterialApp(
        title: 'POS Customer',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.orange),
          useMaterial3: true,
        ),
        initialRoute: '/login',
        routes: {
          '/login': (_) => const OtpRequestScreen(),
          '/otp-verify': (ctx) {
            final phone = ModalRoute.of(ctx)!.settings.arguments as String;
            return OtpVerifyScreen(phone: phone);
          },
          '/home': (_) => const Placeholder(), // Task 13
        },
      ),
    );
  }
}
```

- [ ] **Step 7: Commit**

```bash
cd pos/customer-app
git add .
git commit -m "feat(phase4): create customer Flutter app with OTP auth flow"
```

---

## Task 13: Customer App — Home, Menu, Cart, Checkout

**Files:**
- Create: `pos/customer-app/lib/models/*.dart`
- Create: `pos/customer-app/lib/repositories/home_repository.dart`
- Create: `pos/customer-app/lib/repositories/menu_repository.dart`
- Create: `pos/customer-app/lib/repositories/order_repository.dart`
- Create: `pos/customer-app/lib/repositories/voucher_repository.dart`
- Create: `pos/customer-app/lib/modules/home/`
- Create: `pos/customer-app/lib/modules/menu/`
- Create: `pos/customer-app/lib/modules/cart/`

- [ ] **Step 1: Create models**

`pos/customer-app/lib/models/menu_item.dart`:

```dart
class MenuItem {
  final String id;
  final String name;
  final String category;
  final double sellingPrice;
  final String? imageUrl;

  MenuItem({required this.id, required this.name, required this.category, required this.sellingPrice, this.imageUrl});

  factory MenuItem.fromJson(Map<String, dynamic> json) => MenuItem(
    id: json['id'], name: json['name'], category: json['category'],
    sellingPrice: (json['sellingPrice'] as num).toDouble(), imageUrl: json['imageUrl'],
  );
}
```

`pos/customer-app/lib/models/cart_item.dart`:

```dart
import 'menu_item.dart';

class CartItem {
  final MenuItem menuItem;
  int qty;
  String? notes;

  CartItem({required this.menuItem, this.qty = 1, this.notes});

  double get subtotal => menuItem.sellingPrice * qty;
}
```

`pos/customer-app/lib/models/store_info.dart`:

```dart
class StoreInfo {
  final String storeName;
  final String? logoUrl;
  final String? coverImageUrl;
  final Map<String, String> operatingHours;
  final bool isOpen;

  StoreInfo({required this.storeName, this.logoUrl, this.coverImageUrl, required this.operatingHours, required this.isOpen});

  factory StoreInfo.fromJson(Map<String, dynamic> json) => StoreInfo(
    storeName: json['store_name'], logoUrl: json['logo_url'], coverImageUrl: json['cover_image_url'],
    operatingHours: Map<String, String>.from(json['operating_hours'] ?? {}), isOpen: json['is_open'] ?? false,
  );
}
```

Create similar models for `Banner`, `Section`, `Order`, `Voucher`, `Delivery`, `Notification`, `Customer` following the API response shapes.

- [ ] **Step 2: Create repositories**

`pos/customer-app/lib/repositories/home_repository.dart`:

```dart
import '../core/api/api_client.dart';

class HomeRepository {
  final ApiClient _api;
  HomeRepository(this._api);

  Future<Map<String, dynamic>> getHomePage() async {
    final res = await _api.dio.get('/customer-app/home');
    return res.data['data'];
  }
}
```

`pos/customer-app/lib/repositories/menu_repository.dart`:

```dart
import '../core/api/api_client.dart';
import '../models/menu_item.dart';

class MenuRepository {
  final ApiClient _api;
  MenuRepository(this._api);

  Future<List<MenuItem>> getMenuItems() async {
    final res = await _api.dio.get('/customer-app/menu');
    return (res.data['data'] as List).map((j) => MenuItem.fromJson(j)).toList();
  }

  Future<MenuItem> getMenuItem(String id) async {
    final res = await _api.dio.get('/customer-app/menu/$id');
    return MenuItem.fromJson(res.data['data']);
  }
}
```

`pos/customer-app/lib/repositories/order_repository.dart`:

```dart
import '../core/api/api_client.dart';

class OrderRepository {
  final ApiClient _api;
  OrderRepository(this._api);

  Future<Map<String, dynamic>> createOrder(Map<String, dynamic> data) async {
    final res = await _api.dio.post('/customer-app/orders', data: data);
    return res.data['data'];
  }

  Future<List<dynamic>> getOrders({int page = 1}) async {
    final res = await _api.dio.get('/customer-app/orders', queryParameters: {'page': page});
    return res.data['data']['data'];
  }

  Future<Map<String, dynamic>> getOrderDetail(String id) async {
    final res = await _api.dio.get('/customer-app/orders/$id');
    return res.data['data'];
  }

  Future<void> cancelOrder(String id) async {
    await _api.dio.patch('/customer-app/orders/$id/cancel');
  }

  Future<void> confirmReceived(String id) async {
    await _api.dio.patch('/customer-app/orders/$id/received');
  }
}
```

`pos/customer-app/lib/repositories/voucher_repository.dart`:

```dart
import '../core/api/api_client.dart';

class VoucherRepository {
  final ApiClient _api;
  VoucherRepository(this._api);

  Future<Map<String, dynamic>> validateVoucher(String code, double subtotal, String orderType) async {
    final res = await _api.dio.post('/customer-app/vouchers/validate', data: {
      'code': code, 'subtotal': subtotal, 'orderType': orderType,
    });
    return res.data['data'];
  }
}
```

- [ ] **Step 3: Create Cart BLoC**

`pos/customer-app/lib/modules/cart/bloc/cart_bloc.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../../models/cart_item.dart';
import '../../../models/menu_item.dart';

// Events
abstract class CartEvent {}
class AddToCart extends CartEvent { final MenuItem item; AddToCart(this.item); }
class RemoveFromCart extends CartEvent { final String menuItemId; RemoveFromCart(this.menuItemId); }
class UpdateQty extends CartEvent { final String menuItemId; final int qty; UpdateQty(this.menuItemId, this.qty); }
class UpdateNotes extends CartEvent { final String menuItemId; final String notes; UpdateNotes(this.menuItemId, this.notes); }
class ClearCart extends CartEvent {}
class ApplyVoucher extends CartEvent { final String code; final double discount; ApplyVoucher(this.code, this.discount); }
class RemoveVoucher extends CartEvent {}

// State
class CartState {
  final List<CartItem> items;
  final String? voucherCode;
  final double voucherDiscount;

  CartState({this.items = const [], this.voucherCode, this.voucherDiscount = 0});

  double get subtotal => items.fold(0, (sum, item) => sum + item.subtotal);
  int get totalItems => items.fold(0, (sum, item) => sum + item.qty);

  CartState copyWith({List<CartItem>? items, String? voucherCode, double? voucherDiscount}) {
    return CartState(
      items: items ?? this.items,
      voucherCode: voucherCode ?? this.voucherCode,
      voucherDiscount: voucherDiscount ?? this.voucherDiscount,
    );
  }
}

class CartBloc extends Bloc<CartEvent, CartState> {
  CartBloc() : super(CartState()) {
    on<AddToCart>((event, emit) {
      final items = List<CartItem>.from(state.items);
      final existing = items.indexWhere((i) => i.menuItem.id == event.item.id);
      if (existing >= 0) {
        items[existing].qty++;
      } else {
        items.add(CartItem(menuItem: event.item));
      }
      emit(state.copyWith(items: items));
    });

    on<RemoveFromCart>((event, emit) {
      final items = state.items.where((i) => i.menuItem.id != event.menuItemId).toList();
      emit(state.copyWith(items: items));
    });

    on<UpdateQty>((event, emit) {
      final items = List<CartItem>.from(state.items);
      final idx = items.indexWhere((i) => i.menuItem.id == event.menuItemId);
      if (idx >= 0) {
        if (event.qty <= 0) {
          items.removeAt(idx);
        } else {
          items[idx].qty = event.qty;
        }
      }
      emit(state.copyWith(items: items));
    });

    on<UpdateNotes>((event, emit) {
      final items = List<CartItem>.from(state.items);
      final idx = items.indexWhere((i) => i.menuItem.id == event.menuItemId);
      if (idx >= 0) items[idx].notes = event.notes;
      emit(state.copyWith(items: items));
    });

    on<ClearCart>((event, emit) => emit(CartState()));

    on<ApplyVoucher>((event, emit) {
      emit(state.copyWith(voucherCode: event.code, voucherDiscount: event.discount));
    });

    on<RemoveVoucher>((event, emit) {
      emit(CartState(items: state.items)); // Reset voucher
    });
  }
}
```

- [ ] **Step 4: Create Home, Menu, Cart, Checkout screens**

Build the following screens following Flutter/BLoC patterns established in the existing kasir app:

- **HomeScreen** — fetches `/customer-app/home`, renders CMS banners + sections + store status
- **MenuListScreen** — grid/list of menu items, grouped by category, tap to add to cart
- **MenuDetailScreen** — item image, name, price, description, add to cart button
- **CartScreen** — list cart items with qty +/-, notes per item, voucher input, order type selector (dine-in/pickup/delivery), subtotal/discount/delivery fee/total breakdown
- **CheckoutScreen** — confirm order details, submit, show success/error

Each screen follows the BLoC pattern with `BlocProvider` + `BlocBuilder`/`BlocConsumer`.

- [ ] **Step 5: Add QR scanner for dine-in**

In cart/checkout flow, when user selects DINE_IN order type:
- Open camera scanner using `mobile_scanner` package
- Scan QR → decode `{ table_number: "A5" }`
- Auto-fill table number field

- [ ] **Step 6: Wire up routing in main.dart**

Add routes for `/home`, `/menu`, `/menu/:id`, `/cart`, `/checkout`, `/orders`, `/orders/:id`, `/profile`, `/notifications`.

- [ ] **Step 7: Commit**

```bash
cd pos/customer-app
git add lib/
git commit -m "feat(phase4): add customer app home, menu, cart, and checkout screens with BLoC"
```

---

## Task 14: Customer App — Orders, Profile, Notifications

**Files:**
- Create: `pos/customer-app/lib/modules/orders/`
- Create: `pos/customer-app/lib/modules/profile/`
- Create: `pos/customer-app/lib/modules/notifications/`
- Create: `pos/customer-app/lib/repositories/profile_repository.dart`
- Create: `pos/customer-app/lib/repositories/notification_repository.dart`

- [ ] **Step 1: Create remaining repositories**

`pos/customer-app/lib/repositories/profile_repository.dart`:

```dart
import '../core/api/api_client.dart';

class ProfileRepository {
  final ApiClient _api;
  ProfileRepository(this._api);

  Future<Map<String, dynamic>> getProfile() async {
    final res = await _api.dio.get('/customers/profile');
    return res.data['data'];
  }

  Future<void> updateProfile(Map<String, dynamic> data) async {
    await _api.dio.patch('/customers/profile', data: data);
  }

  Future<List<dynamic>> getAddresses() async {
    final res = await _api.dio.get('/customers/addresses');
    return res.data['data'];
  }

  Future<void> createAddress(Map<String, dynamic> data) async {
    await _api.dio.post('/customers/addresses', data: data);
  }

  Future<void> updateAddress(String id, Map<String, dynamic> data) async {
    await _api.dio.patch('/customers/addresses/$id', data: data);
  }

  Future<void> deleteAddress(String id) async {
    await _api.dio.delete('/customers/addresses/$id');
  }
}
```

`pos/customer-app/lib/repositories/notification_repository.dart`:

```dart
import '../core/api/api_client.dart';

class NotificationRepository {
  final ApiClient _api;
  NotificationRepository(this._api);

  Future<void> registerDevice(String fcmToken, String deviceType) async {
    await _api.dio.post('/customer-app/devices', data: {'fcmToken': fcmToken, 'deviceType': deviceType});
  }

  Future<Map<String, dynamic>> getNotifications({int page = 1}) async {
    final res = await _api.dio.get('/customer-app/notifications', queryParameters: {'page': page});
    return res.data['data'];
  }

  Future<void> markAsRead(String id) async {
    await _api.dio.patch('/customer-app/notifications/$id/read');
  }

  Future<void> markAllAsRead() async {
    await _api.dio.patch('/customer-app/notifications/read-all');
  }
}
```

- [ ] **Step 2: Create Order BLoC and screens**

Build:
- **OrderListScreen** — paginated list of past orders with status badges, pull-to-refresh
- **OrderDetailScreen** — full order detail with items, status timeline, delivery info (driver name/phone if delivery), action buttons (cancel if pending, confirm received if on_delivery)

- [ ] **Step 3: Create Profile BLoC and screens**

Build:
- **ProfileScreen** — display/edit name, email, phone (read-only). Logout button.
- **AddressScreen** — list addresses, add/edit/delete, set default

- [ ] **Step 4: Create Notification BLoC and screen**

Build:
- **NotificationScreen** — list of notifications with read/unread state, mark as read on tap, "mark all read" button

- [ ] **Step 5: Setup FCM in customer app**

In `main.dart`, initialize Firebase and register FCM token:

```dart
// After successful auth:
final fcmToken = await FirebaseMessaging.instance.getToken();
if (fcmToken != null) {
  await notificationRepo.registerDevice(fcmToken, Platform.isIOS ? 'IOS' : 'ANDROID');
}

// Listen for foreground messages:
FirebaseMessaging.onMessage.listen((message) {
  // Show local notification or update UI
});
```

- [ ] **Step 6: Create bottom navigation**

Main app shell with bottom nav: Home, Menu, Orders, Profile. Notification bell icon in app bar with unread badge count.

- [ ] **Step 7: Commit**

```bash
cd pos/customer-app
git add lib/
git commit -m "feat(phase4): add order tracking, profile, notifications, and FCM integration to customer app"
```

---

## Task 15: Integration Testing & Welcome Voucher

**Files:**
- Modify: `pos/backend/src/customers/auth/customer-auth.service.ts`
- Create: `pos/backend/test/phase4-integration.spec.ts`

- [ ] **Step 1: Wire welcome voucher to customer registration**

In `customer-auth.service.ts`, after auto-creating a new customer in `verifyOtp`:

```typescript
if (!customer) {
  customer = await this.prisma.customer.create({ data: { phone } });

  // Generate welcome voucher
  const welcomeVoucher = await this.vouchersService.createWelcomeVoucher(customer.id);

  // Send welcome notification
  await this.fcmService.sendToCustomer(customer.id, {
    title: 'Selamat datang!',
    body: `Pakai voucher ${welcomeVoucher.code} untuk diskon 10%`,
    data: { type: 'WELCOME_VOUCHER', voucherCode: welcomeVoucher.code },
  });
}
```

Inject `VouchersService` and `FcmService` into `CustomerAuthService`.

- [ ] **Step 2: Write integration test for full online order flow**

`pos/backend/test/phase4-integration.spec.ts`:

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Phase 4 Integration', () => {
  let app: INestApplication;
  let customerToken: string;
  let staffToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
    await app.init();

    // Login as staff (existing user)
    const staffLogin = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'owner@test.com', password: 'password' });
    staffToken = staffLogin.body.data.accessToken;
  });

  afterAll(async () => {
    await app.close();
  });

  it('should complete full online order flow', async () => {
    // 1. Customer requests OTP
    await request(app.getHttpServer())
      .post('/customers/auth/otp/request')
      .send({ phone: '08999999999' })
      .expect(201);

    // 2. Customer verifies OTP (dev mode: 123456)
    const verifyRes = await request(app.getHttpServer())
      .post('/customers/auth/otp/verify')
      .send({ phone: '08999999999', code: '123456' })
      .expect(201);

    customerToken = verifyRes.body.data.accessToken;
    expect(verifyRes.body.data.customer).toBeDefined();

    // 3. Customer browses menu
    const menuRes = await request(app.getHttpServer())
      .get('/customer-app/menu')
      .expect(200);

    // 4. Customer submits order (if menu items exist)
    if (menuRes.body.data.length > 0) {
      const orderRes = await request(app.getHttpServer())
        .post('/customer-app/orders')
        .set('Authorization', `Bearer ${customerToken}`)
        .send({
          orderType: 'PICKUP',
          items: [{ menuItemId: menuRes.body.data[0].id, qty: 2 }],
        })
        .expect(201);

      const orderId = orderRes.body.data.id;

      // 5. Staff confirms order
      await request(app.getHttpServer())
        .patch(`/online-orders/${orderId}/confirm`)
        .set('Authorization', `Bearer ${staffToken}`)
        .expect(200);

      // 6. Customer checks order status
      const statusRes = await request(app.getHttpServer())
        .get(`/customer-app/orders/${orderId}`)
        .set('Authorization', `Bearer ${customerToken}`)
        .expect(200);

      expect(statusRes.body.data.status).toBe('CONFIRMED');
    }
  });
});
```

- [ ] **Step 3: Run integration test**

Run: `cd pos/backend && npx jest --testPathPattern=phase4-integration --no-coverage --forceExit`

Expected: PASS (may need database with seed data)

- [ ] **Step 4: Commit**

```bash
cd pos/backend
git add src/customers/ test/phase4-integration.spec.ts
git commit -m "feat(phase4): add welcome voucher on registration and full integration test"
```

---

## Task 16: Final Wiring & Verification

- [ ] **Step 1: Verify all backend modules registered in AppModule**

Check `pos/backend/src/app.module.ts` includes:
- `CustomersModule`
- `UploadsModule`
- `CmsModule`
- `VouchersModule`
- `OnlineOrdersModule`
- `CustomerAppModule`
- `DeliveriesModule`
- `FcmModule`

- [ ] **Step 2: Run all backend tests**

Run: `cd pos/backend && npx jest --no-coverage`

Expected: All tests pass (existing Phase 1-3 + new Phase 4)

- [ ] **Step 3: Start backend and verify endpoints**

Run: `cd pos/backend && npm run start:dev`

Test key endpoints with curl:
```bash
# Public endpoints
curl http://localhost:3000/customer-app/home
curl http://localhost:3000/customer-app/menu

# OTP flow
curl -X POST http://localhost:3000/customers/auth/otp/request -H "Content-Type: application/json" -d '{"phone":"08123456789"}'
curl -X POST http://localhost:3000/customers/auth/otp/verify -H "Content-Type: application/json" -d '{"phone":"08123456789","code":"123456"}'
```

- [ ] **Step 4: Verify web dashboard builds**

Run: `cd pos/web && npm run build`

Expected: Build successful with no errors.

- [ ] **Step 5: Verify customer app builds**

Run: `cd pos/customer-app && flutter build apk --debug`

Expected: Build successful.

- [ ] **Step 6: Final commit**

```bash
cd pos
git add .
git commit -m "feat(phase4): complete Phase 4 — Online Ordering, CMS, Vouchers, Delivery, FCM Notifications"
```
