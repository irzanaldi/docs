# POS System Phase 1 — Backend API Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the complete NestJS backend API for the POS system with HPP Engine, RBAC, kitchen order flow (WebSocket), promo system, and dashboard analytics.

**Architecture:** Monolith NestJS with modular structure. Each domain (auth, menu, ingredients, pos, orders, promos, dashboard) is a separate NestJS module. PostgreSQL via Prisma ORM. JWT for auth with RBAC guard. Socket.IO for real-time kitchen display.

**Tech Stack:** NestJS, TypeScript, PostgreSQL, Prisma, JWT (passport-jwt), Socket.IO, class-validator, class-transformer, bcrypt, uuid

**Spec:** `docs/superpowers/specs/2026-04-18-pos-system-design.md`

---

## File Structure

```
backend/
├── package.json
├── tsconfig.json
├── tsconfig.build.json
├── nest-cli.json
├── .env
├── .env.example
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── common/
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   └── permissions.decorator.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   └── permissions.guard.ts
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts
│   │   ├── interceptors/
│   │   │   └── transform.interceptor.ts
│   │   └── dto/
│   │       └── pagination.dto.ts
│   ├── prisma/
│   │   ├── prisma.module.ts
│   │   └── prisma.service.ts
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.service.spec.ts
│   │   ├── strategies/
│   │   │   └── jwt.strategy.ts
│   │   └── dto/
│   │       ├── login.dto.ts
│   │       └── register.dto.ts
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.service.spec.ts
│   │   └── dto/
│   │       ├── create-user.dto.ts
│   │       └── update-user.dto.ts
│   ├── ingredients/
│   │   ├── ingredients.module.ts
│   │   ├── ingredients.controller.ts
│   │   ├── ingredients.service.ts
│   │   ├── ingredients.service.spec.ts
│   │   └── dto/
│   │       ├── create-ingredient.dto.ts
│   │       └── update-ingredient.dto.ts
│   ├── menu/
│   │   ├── menu.module.ts
│   │   ├── menu.controller.ts
│   │   ├── menu.service.ts
│   │   ├── menu.service.spec.ts
│   │   └── dto/
│   │       ├── create-menu.dto.ts
│   │       └── update-menu.dto.ts
│   ├── retail/
│   │   ├── retail.module.ts
│   │   ├── retail.controller.ts
│   │   ├── retail.service.ts
│   │   ├── retail.service.spec.ts
│   │   └── dto/
│   │       ├── create-retail.dto.ts
│   │       └── update-retail.dto.ts
│   ├── promos/
│   │   ├── promos.module.ts
│   │   ├── promos.controller.ts
│   │   ├── promos.service.ts
│   │   ├── promos.service.spec.ts
│   │   └── dto/
│   │       ├── create-promo.dto.ts
│   │       └── apply-promo.dto.ts
│   ├── pos/
│   │   ├── pos.module.ts
│   │   ├── pos.controller.ts
│   │   ├── pos.service.ts
│   │   ├── pos.service.spec.ts
│   │   └── dto/
│   │       └── create-transaction.dto.ts
│   ├── orders/
│   │   ├── orders.module.ts
│   │   ├── orders.controller.ts
│   │   ├── orders.service.ts
│   │   ├── orders.service.spec.ts
│   │   ├── orders.gateway.ts
│   │   └── dto/
│   │       └── update-order-status.dto.ts
│   └── dashboard/
│       ├── dashboard.module.ts
│       ├── dashboard.controller.ts
│       ├── dashboard.service.ts
│       └── dashboard.service.spec.ts
└── test/
    ├── jest-e2e.json
    └── app.e2e-spec.ts
```

---

### Task 1: Project Setup & Configuration

**Files:**
- Create: `backend/package.json`
- Create: `backend/tsconfig.json`
- Create: `backend/tsconfig.build.json`
- Create: `backend/nest-cli.json`
- Create: `backend/.env`
- Create: `backend/.env.example`
- Create: `backend/src/main.ts`
- Create: `backend/src/app.module.ts`

- [ ] **Step 1: Scaffold NestJS project**

```bash
cd ~/Downloads/PROJECT
npx @nestjs/cli new backend --package-manager npm --skip-git
```

- [ ] **Step 2: Install dependencies**

```bash
cd ~/Downloads/PROJECT/backend
npm install @nestjs/config @nestjs/passport @nestjs/jwt passport passport-jwt @nestjs/websockets @nestjs/platform-socket.io socket.io class-validator class-transformer bcrypt uuid prisma @prisma/client
npm install -D @types/passport-jwt @types/bcrypt @types/uuid
```

- [ ] **Step 3: Create .env and .env.example**

Create `backend/.env`:
```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/pos_system?schema=public"
JWT_SECRET="your-super-secret-jwt-key-change-in-production"
JWT_EXPIRES_IN="15m"
JWT_REFRESH_EXPIRES_IN="7d"
PORT=3000
```

Create `backend/.env.example` with same keys but placeholder values.

- [ ] **Step 4: Update main.ts with global config**

```typescript
// backend/src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  app.enableCors({
    origin: ['http://localhost:3001', 'http://localhost:8080'],
    credentials: true,
  });

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

- [ ] **Step 5: Update app.module.ts**

```typescript
// backend/src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { PrismaModule } from './prisma/prisma.module';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    PrismaModule,
  ],
})
export class AppModule {}
```

- [ ] **Step 6: Verify the app starts**

```bash
cd ~/Downloads/PROJECT/backend
npm run start:dev
```

Expected: Application listening on port 3000

- [ ] **Step 7: Commit**

```bash
cd ~/Downloads/PROJECT/backend
git init
git add .
git commit -m "chore: scaffold NestJS project with config"
```

---

### Task 2: Prisma Schema & Database Setup

**Files:**
- Create: `backend/prisma/schema.prisma`
- Create: `backend/src/prisma/prisma.module.ts`
- Create: `backend/src/prisma/prisma.service.ts`

- [ ] **Step 1: Initialize Prisma**

```bash
cd ~/Downloads/PROJECT/backend
npx prisma init
```

- [ ] **Step 2: Write the full Prisma schema**

```prisma
// backend/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Role {
  id          String           @id @default(uuid())
  name        String           @unique
  description String?
  users       User[]
  permissions RolePermission[]

  @@map("roles")
}

model Permission {
  id          String           @id @default(uuid())
  name        String           @unique
  module      String
  description String?
  roles       RolePermission[]

  @@map("permissions")
}

model RolePermission {
  roleId       String
  permissionId String
  role         Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission   Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)

  @@id([roleId, permissionId])
  @@map("role_permissions")
}

model User {
  id           String        @id @default(uuid())
  name         String
  email        String        @unique
  password     String
  roleId       String
  isActive     Boolean       @default(true)
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
  role         Role          @relation(fields: [roleId], references: [id])
  transactions Transaction[]

  @@map("users")
}

model Ingredient {
  id           String                @id @default(uuid())
  name         String
  unit         String
  pricePerUnit Float
  stockQty     Float                 @default(0)
  minStock     Float                 @default(0)
  createdAt    DateTime              @default(now())
  updatedAt    DateTime              @updatedAt
  recipes      Recipe[]
  priceLogs    IngredientPriceLog[]

  @@map("ingredients")
}

model IngredientPriceLog {
  id           String     @id @default(uuid())
  ingredientId String
  oldPrice     Float
  newPrice     Float
  changedAt    DateTime   @default(now())
  ingredient   Ingredient @relation(fields: [ingredientId], references: [id], onDelete: Cascade)

  @@map("ingredient_price_logs")
}

model MenuItem {
  id           String      @id @default(uuid())
  name         String
  category     MenuCategory
  sellingPrice Float
  imageUrl     String?
  isActive     Boolean     @default(true)
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
  recipes      Recipe[]
  orderItems   OrderItem[]
  promoItems   PromoItem[]

  @@map("menu_items")
}

enum MenuCategory {
  FOOD
  BEVERAGE
}

model Recipe {
  id           String     @id @default(uuid())
  menuItemId   String
  ingredientId String
  qtyNeeded    Float
  unit         String
  menuItem     MenuItem   @relation(fields: [menuItemId], references: [id], onDelete: Cascade)
  ingredient   Ingredient @relation(fields: [ingredientId], references: [id], onDelete: Cascade)

  @@unique([menuItemId, ingredientId])
  @@map("recipes")
}

model RetailItem {
  id           String      @id @default(uuid())
  name         String
  category     String
  buyPrice     Float
  sellingPrice Float
  stockQty     Float       @default(0)
  minStock     Float       @default(0)
  imageUrl     String?
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
  promoItems   PromoItem[]

  @@map("retail_items")
}

model Transaction {
  id              String             @id @default(uuid())
  transactionCode String             @unique
  cashierId       String
  paymentMethod   PaymentMethod
  totalAmount     Float
  totalHpp        Float
  totalProfit     Float
  promoDiscount   Float              @default(0)
  createdAt       DateTime           @default(now())
  updatedAt       DateTime           @updatedAt
  cashier         User               @relation(fields: [cashierId], references: [id])
  items           TransactionItem[]
  orders          Order[]
  promos          TransactionPromo[]

  @@map("transactions")
}

enum PaymentMethod {
  CASH
  QRIS
  EDC
}

model TransactionItem {
  id            String      @id @default(uuid())
  transactionId String
  itemType      ItemType
  itemId        String
  qty           Int
  unitPrice     Float
  unitHpp       Float
  subtotal      Float
  subHpp        Float
  transaction   Transaction @relation(fields: [transactionId], references: [id], onDelete: Cascade)

  @@map("transaction_items")
}

enum ItemType {
  MENU
  RETAIL
}

model Order {
  id            String      @id @default(uuid())
  transactionId String
  orderNumber   Int
  status        OrderStatus @default(PENDING)
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
  transaction   Transaction @relation(fields: [transactionId], references: [id], onDelete: Cascade)
  items         OrderItem[]

  @@map("orders")
}

enum OrderStatus {
  PENDING
  PREPARING
  READY
  SERVED
}

model OrderItem {
  id         String          @id @default(uuid())
  orderId    String
  menuItemId String
  qty        Int
  notes      String?
  status     OrderItemStatus @default(PENDING)
  order      Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  menuItem   MenuItem        @relation(fields: [menuItemId], references: [id])

  @@map("order_items")
}

enum OrderItemStatus {
  PENDING
  PREPARING
  DONE
}

model Promo {
  id                String             @id @default(uuid())
  name              String
  type              PromoType
  value             Float
  minPurchaseAmount Float?
  startDate         DateTime
  endDate           DateTime
  isActive          Boolean            @default(true)
  maxUsage          Int?
  usedCount         Int                @default(0)
  createdAt         DateTime           @default(now())
  updatedAt         DateTime           @updatedAt
  items             PromoItem[]
  transactions      TransactionPromo[]

  @@map("promos")
}

enum PromoType {
  PERCENT
  FIXED
  BUY_X_GET_Y
  BUNDLE
  MIN_PURCHASE
}

model PromoItem {
  id         String        @id @default(uuid())
  promoId    String
  itemType   ItemType
  itemId     String
  role       PromoItemRole
  promo      Promo         @relation(fields: [promoId], references: [id], onDelete: Cascade)
  menuItem   MenuItem?     @relation(fields: [itemId], references: [id])
  retailItem RetailItem?

  @@map("promo_items")
}

enum PromoItemRole {
  TARGET
  FREE_ITEM
  BUNDLE_MEMBER
}

model TransactionPromo {
  id              String      @id @default(uuid())
  transactionId   String
  promoId         String
  discountAmount  Float
  transaction     Transaction @relation(fields: [transactionId], references: [id], onDelete: Cascade)
  promo           Promo       @relation(fields: [promoId], references: [id])

  @@map("transaction_promos")
}
```

- [ ] **Step 3: Run migration**

```bash
cd ~/Downloads/PROJECT/backend
npx prisma migrate dev --name init
```

Expected: Migration applied successfully

- [ ] **Step 4: Create PrismaService**

```typescript
// backend/src/prisma/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

- [ ] **Step 5: Create PrismaModule**

```typescript
// backend/src/prisma/prisma.module.ts
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

- [ ] **Step 6: Verify Prisma connects**

```bash
cd ~/Downloads/PROJECT/backend
npx prisma db push
npx prisma studio
```

Expected: Prisma Studio opens showing all tables

- [ ] **Step 7: Commit**

```bash
git add prisma/ src/prisma/
git commit -m "feat: add Prisma schema with all Phase 1 tables"
```

---

### Task 3: Common Guards, Decorators & Interceptors

**Files:**
- Create: `backend/src/common/decorators/current-user.decorator.ts`
- Create: `backend/src/common/decorators/permissions.decorator.ts`
- Create: `backend/src/common/guards/jwt-auth.guard.ts`
- Create: `backend/src/common/guards/permissions.guard.ts`
- Create: `backend/src/common/filters/http-exception.filter.ts`
- Create: `backend/src/common/interceptors/transform.interceptor.ts`
- Create: `backend/src/common/dto/pagination.dto.ts`

- [ ] **Step 1: Create CurrentUser decorator**

```typescript
// backend/src/common/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);
```

- [ ] **Step 2: Create Permissions decorator**

```typescript
// backend/src/common/decorators/permissions.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const PERMISSIONS_KEY = 'permissions';
export const RequirePermissions = (...permissions: string[]) =>
  SetMetadata(PERMISSIONS_KEY, permissions);
```

- [ ] **Step 3: Create JwtAuthGuard**

```typescript
// backend/src/common/guards/jwt-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

- [ ] **Step 4: Create PermissionsGuard**

```typescript
// backend/src/common/guards/permissions.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { PERMISSIONS_KEY } from '../decorators/permissions.decorator';
import { PrismaService } from '../../prisma/prisma.service';

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private prisma: PrismaService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const requiredPermissions = this.reflector.getAllAndOverride<string[]>(
      PERMISSIONS_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!requiredPermissions || requiredPermissions.length === 0) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    if (!user) return false;

    const rolePermissions = await this.prisma.rolePermission.findMany({
      where: { roleId: user.roleId },
      include: { permission: true },
    });

    const userPermissions = rolePermissions.map((rp) => rp.permission.name);

    return requiredPermissions.every((perm) => userPermissions.includes(perm));
  }
}
```

- [ ] **Step 5: Create HttpExceptionFilter**

```typescript
// backend/src/common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Response } from 'express';

@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.getResponse()
        : 'Internal server error';

    response.status(status).json({
      statusCode: status,
      message: typeof message === 'string' ? message : (message as any).message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

- [ ] **Step 6: Create TransformInterceptor**

```typescript
// backend/src/common/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface ApiResponse<T> {
  statusCode: number;
  data: T;
  message: string;
}

@Injectable()
export class TransformInterceptor<T>
  implements NestInterceptor<T, ApiResponse<T>>
{
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<ApiResponse<T>> {
    return next.handle().pipe(
      map((data) => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        data,
        message: 'Success',
      })),
    );
  }
}
```

- [ ] **Step 7: Create PaginationDto**

```typescript
// backend/src/common/dto/pagination.dto.ts
import { IsOptional, IsInt, Min, Max } from 'class-validator';
import { Type } from 'class-transformer';

export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}
```

- [ ] **Step 8: Register global filter and interceptor in main.ts**

Update `backend/src/main.ts` — add after `app.useGlobalPipes(...)`:

```typescript
import { HttpExceptionFilter } from './common/filters/http-exception.filter';
import { TransformInterceptor } from './common/interceptors/transform.interceptor';

// add inside bootstrap():
app.useGlobalFilters(new HttpExceptionFilter());
app.useGlobalInterceptors(new TransformInterceptor());
```

- [ ] **Step 9: Commit**

```bash
git add src/common/ src/main.ts
git commit -m "feat: add common guards, decorators, filters, interceptors"
```

---

### Task 4: Auth Module (JWT + Login/Register)

**Files:**
- Create: `backend/src/auth/auth.module.ts`
- Create: `backend/src/auth/auth.controller.ts`
- Create: `backend/src/auth/auth.service.ts`
- Create: `backend/src/auth/auth.service.spec.ts`
- Create: `backend/src/auth/strategies/jwt.strategy.ts`
- Create: `backend/src/auth/dto/login.dto.ts`
- Create: `backend/src/auth/dto/register.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/auth/dto/login.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}
```

```typescript
// backend/src/auth/dto/register.dto.ts
import { IsEmail, IsString, MinLength, IsUUID } from 'class-validator';

export class RegisterDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;

  @IsUUID()
  roleId: string;
}
```

- [ ] **Step 2: Create JwtStrategy**

```typescript
// backend/src/auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { PrismaService } from '../../prisma/prisma.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    configService: ConfigService,
    private prisma: PrismaService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  async validate(payload: { sub: string; email: string }) {
    const user = await this.prisma.user.findUnique({
      where: { id: payload.sub },
      include: { role: true },
    });

    if (!user || !user.isActive) {
      throw new UnauthorizedException();
    }

    return {
      id: user.id,
      email: user.email,
      name: user.name,
      roleId: user.roleId,
      roleName: user.role.name,
    };
  }
}
```

- [ ] **Step 3: Write failing test for AuthService**

```typescript
// backend/src/auth/auth.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { AuthService } from './auth.service';
import { PrismaService } from '../prisma/prisma.service';
import { JwtService } from '@nestjs/jwt';
import { UnauthorizedException } from '@nestjs/common';
import * as bcrypt from 'bcrypt';

describe('AuthService', () => {
  let service: AuthService;
  let prisma: PrismaService;
  let jwtService: JwtService;

  const mockPrisma = {
    user: {
      findUnique: jest.fn(),
      create: jest.fn(),
    },
    role: {
      findUnique: jest.fn(),
    },
  };

  const mockJwtService = {
    sign: jest.fn().mockReturnValue('mock-token'),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        AuthService,
        { provide: PrismaService, useValue: mockPrisma },
        { provide: JwtService, useValue: mockJwtService },
      ],
    }).compile();

    service = module.get<AuthService>(AuthService);
    prisma = module.get<PrismaService>(PrismaService);
    jwtService = module.get<JwtService>(JwtService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('login', () => {
    it('should return access token for valid credentials', async () => {
      const hashedPassword = await bcrypt.hash('password123', 10);
      mockPrisma.user.findUnique.mockResolvedValue({
        id: 'user-1',
        email: 'test@test.com',
        password: hashedPassword,
        isActive: true,
        role: { name: 'OWNER' },
        roleId: 'role-1',
      });

      const result = await service.login({
        email: 'test@test.com',
        password: 'password123',
      });

      expect(result).toHaveProperty('accessToken');
      expect(mockJwtService.sign).toHaveBeenCalled();
    });

    it('should throw UnauthorizedException for invalid password', async () => {
      const hashedPassword = await bcrypt.hash('password123', 10);
      mockPrisma.user.findUnique.mockResolvedValue({
        id: 'user-1',
        email: 'test@test.com',
        password: hashedPassword,
        isActive: true,
      });

      await expect(
        service.login({ email: 'test@test.com', password: 'wrong' }),
      ).rejects.toThrow(UnauthorizedException);
    });

    it('should throw UnauthorizedException for non-existent user', async () => {
      mockPrisma.user.findUnique.mockResolvedValue(null);

      await expect(
        service.login({ email: 'no@test.com', password: 'password123' }),
      ).rejects.toThrow(UnauthorizedException);
    });
  });

  describe('register', () => {
    it('should create a new user and return access token', async () => {
      mockPrisma.role.findUnique.mockResolvedValue({ id: 'role-1', name: 'KASIR' });
      mockPrisma.user.create.mockResolvedValue({
        id: 'user-2',
        email: 'new@test.com',
        name: 'New User',
        roleId: 'role-1',
        role: { name: 'KASIR' },
      });

      const result = await service.register({
        name: 'New User',
        email: 'new@test.com',
        password: 'password123',
        roleId: 'role-1',
      });

      expect(result).toHaveProperty('accessToken');
      expect(mockPrisma.user.create).toHaveBeenCalled();
    });
  });
});
```

- [ ] **Step 4: Run test to verify it fails**

```bash
cd ~/Downloads/PROJECT/backend
npx jest src/auth/auth.service.spec.ts --verbose
```

Expected: FAIL — AuthService not found

- [ ] **Step 5: Implement AuthService**

```typescript
// backend/src/auth/auth.service.ts
import { Injectable, UnauthorizedException, ConflictException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { PrismaService } from '../prisma/prisma.service';
import { LoginDto } from './dto/login.dto';
import { RegisterDto } from './dto/register.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private prisma: PrismaService,
    private jwtService: JwtService,
  ) {}

  async login(dto: LoginDto) {
    const user = await this.prisma.user.findUnique({
      where: { email: dto.email },
      include: { role: true },
    });

    if (!user || !user.isActive) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(dto.password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const payload = { sub: user.id, email: user.email, role: user.role.name };
    return {
      accessToken: this.jwtService.sign(payload),
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role.name,
      },
    };
  }

  async register(dto: RegisterDto) {
    const existing = await this.prisma.user.findUnique({
      where: { email: dto.email },
    });
    if (existing) {
      throw new ConflictException('Email already exists');
    }

    const hashedPassword = await bcrypt.hash(dto.password, 10);
    const user = await this.prisma.user.create({
      data: {
        name: dto.name,
        email: dto.email,
        password: hashedPassword,
        roleId: dto.roleId,
      },
      include: { role: true },
    });

    const payload = { sub: user.id, email: user.email, role: user.role.name };
    return {
      accessToken: this.jwtService.sign(payload),
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role.name,
      },
    };
  }
}
```

- [ ] **Step 6: Run test to verify it passes**

```bash
npx jest src/auth/auth.service.spec.ts --verbose
```

Expected: All 3 tests PASS

- [ ] **Step 7: Create AuthController**

```typescript
// backend/src/auth/auth.controller.ts
import { Controller, Post, Body, Get, UseGuards } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LoginDto } from './dto/login.dto';
import { RegisterDto } from './dto/register.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { CurrentUser } from '../common/decorators/current-user.decorator';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }

  @Post('register')
  register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @UseGuards(JwtAuthGuard)
  @Get('me')
  getProfile(@CurrentUser() user: any) {
    return user;
  }
}
```

- [ ] **Step 8: Create AuthModule**

```typescript
// backend/src/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigService } from '@nestjs/config';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    PassportModule,
    JwtModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get<string>('JWT_SECRET'),
        signOptions: { expiresIn: config.get<string>('JWT_EXPIRES_IN') },
      }),
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

- [ ] **Step 9: Register AuthModule in AppModule**

Add to `backend/src/app.module.ts` imports array:

```typescript
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    PrismaModule,
    AuthModule,
  ],
})
export class AppModule {}
```

- [ ] **Step 10: Commit**

```bash
git add src/auth/ src/app.module.ts
git commit -m "feat: add auth module with JWT login/register and RBAC guard"
```

---

### Task 5: Users Module (CRUD + RBAC)

**Files:**
- Create: `backend/src/users/users.module.ts`
- Create: `backend/src/users/users.controller.ts`
- Create: `backend/src/users/users.service.ts`
- Create: `backend/src/users/users.service.spec.ts`
- Create: `backend/src/users/dto/create-user.dto.ts`
- Create: `backend/src/users/dto/update-user.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, IsUUID } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;

  @IsUUID()
  roleId: string;
}
```

```typescript
// backend/src/users/dto/update-user.dto.ts
import { IsEmail, IsString, MinLength, IsUUID, IsBoolean, IsOptional } from 'class-validator';

export class UpdateUserDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  name?: string;

  @IsOptional()
  @IsEmail()
  email?: string;

  @IsOptional()
  @IsString()
  @MinLength(6)
  password?: string;

  @IsOptional()
  @IsUUID()
  roleId?: string;

  @IsOptional()
  @IsBoolean()
  isActive?: boolean;
}
```

- [ ] **Step 2: Write failing test for UsersService**

```typescript
// backend/src/users/users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { PrismaService } from '../prisma/prisma.service';
import { NotFoundException } from '@nestjs/common';

describe('UsersService', () => {
  let service: UsersService;

  const mockPrisma = {
    user: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    role: {
      findMany: jest.fn(),
    },
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('findAll', () => {
    it('should return paginated users', async () => {
      const users = [
        { id: '1', name: 'User 1', email: 'u1@test.com', role: { name: 'OWNER' } },
      ];
      mockPrisma.user.findMany.mockResolvedValue(users);
      mockPrisma.user.count.mockResolvedValue(1);

      const result = await service.findAll({ page: 1, limit: 20 });

      expect(result.data).toEqual(users);
      expect(result.total).toBe(1);
    });
  });

  describe('findOne', () => {
    it('should return a user by id', async () => {
      const user = { id: '1', name: 'User 1', role: { name: 'OWNER' } };
      mockPrisma.user.findUnique.mockResolvedValue(user);

      const result = await service.findOne('1');
      expect(result).toEqual(user);
    });

    it('should throw NotFoundException if user not found', async () => {
      mockPrisma.user.findUnique.mockResolvedValue(null);
      await expect(service.findOne('999')).rejects.toThrow(NotFoundException);
    });
  });

  describe('getRoles', () => {
    it('should return all roles', async () => {
      const roles = [{ id: '1', name: 'OWNER' }, { id: '2', name: 'KASIR' }];
      mockPrisma.role.findMany.mockResolvedValue(roles);

      const result = await service.getRoles();
      expect(result).toEqual(roles);
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/users/users.service.spec.ts --verbose
```

Expected: FAIL — UsersService not found

- [ ] **Step 4: Implement UsersService**

```typescript
// backend/src/users/users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(private prisma: PrismaService) {}

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      this.prisma.user.findMany({
        skip,
        take: limit,
        select: {
          id: true,
          name: true,
          email: true,
          isActive: true,
          createdAt: true,
          role: { select: { id: true, name: true } },
        },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.user.count(),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const user = await this.prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        name: true,
        email: true,
        isActive: true,
        createdAt: true,
        role: { select: { id: true, name: true } },
      },
    });

    if (!user) throw new NotFoundException('User not found');
    return user;
  }

  async create(dto: CreateUserDto) {
    const existing = await this.prisma.user.findUnique({
      where: { email: dto.email },
    });
    if (existing) throw new ConflictException('Email already exists');

    const hashedPassword = await bcrypt.hash(dto.password, 10);
    return this.prisma.user.create({
      data: { ...dto, password: hashedPassword },
      select: {
        id: true,
        name: true,
        email: true,
        isActive: true,
        role: { select: { id: true, name: true } },
      },
    });
  }

  async update(id: string, dto: UpdateUserDto) {
    await this.findOne(id);

    const data: any = { ...dto };
    if (dto.password) {
      data.password = await bcrypt.hash(dto.password, 10);
    }

    return this.prisma.user.update({
      where: { id },
      data,
      select: {
        id: true,
        name: true,
        email: true,
        isActive: true,
        role: { select: { id: true, name: true } },
      },
    });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.user.delete({ where: { id } });
    return { message: 'User deleted' };
  }

  async getRoles() {
    return this.prisma.role.findMany({
      include: {
        permissions: { include: { permission: true } },
      },
    });
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/users/users.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create UsersController**

```typescript
// backend/src/users/users.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('users')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get()
  @RequirePermissions('user:manage')
  findAll(@Query() pagination: PaginationDto) {
    return this.usersService.findAll(pagination);
  }

  @Get('roles')
  @RequirePermissions('user:manage')
  getRoles() {
    return this.usersService.getRoles();
  }

  @Get(':id')
  @RequirePermissions('user:manage')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  @RequirePermissions('user:manage')
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Patch(':id')
  @RequirePermissions('user:manage')
  update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return this.usersService.update(id, dto);
  }

  @Delete(':id')
  @RequirePermissions('user:manage')
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

- [ ] **Step 7: Create UsersModule and register in AppModule**

```typescript
// backend/src/users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

Add `UsersModule` to `AppModule` imports.

- [ ] **Step 8: Commit**

```bash
git add src/users/ src/app.module.ts
git commit -m "feat: add users module with CRUD and role management"
```

---

### Task 6: Ingredients Module

**Files:**
- Create: `backend/src/ingredients/ingredients.module.ts`
- Create: `backend/src/ingredients/ingredients.controller.ts`
- Create: `backend/src/ingredients/ingredients.service.ts`
- Create: `backend/src/ingredients/ingredients.service.spec.ts`
- Create: `backend/src/ingredients/dto/create-ingredient.dto.ts`
- Create: `backend/src/ingredients/dto/update-ingredient.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/ingredients/dto/create-ingredient.dto.ts
import { IsString, IsNumber, Min, IsOptional } from 'class-validator';

export class CreateIngredientDto {
  @IsString()
  name: string;

  @IsString()
  unit: string;

  @IsNumber()
  @Min(0)
  pricePerUnit: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  stockQty?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  minStock?: number;
}
```

```typescript
// backend/src/ingredients/dto/update-ingredient.dto.ts
import { IsString, IsNumber, Min, IsOptional } from 'class-validator';

export class UpdateIngredientDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsString()
  unit?: string;

  @IsOptional()
  @IsNumber()
  @Min(0)
  pricePerUnit?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  stockQty?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  minStock?: number;
}
```

- [ ] **Step 2: Write failing test for IngredientsService**

```typescript
// backend/src/ingredients/ingredients.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { IngredientsService } from './ingredients.service';
import { PrismaService } from '../prisma/prisma.service';

describe('IngredientsService', () => {
  let service: IngredientsService;

  const mockPrisma = {
    ingredient: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    ingredientPriceLog: {
      create: jest.fn(),
      findMany: jest.fn(),
    },
    recipe: {
      findMany: jest.fn(),
    },
    $transaction: jest.fn((cb) => cb(mockPrisma)),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        IngredientsService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<IngredientsService>(IngredientsService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('updatePrice', () => {
    it('should update price and create price log', async () => {
      const ingredient = { id: '1', name: 'Telur', pricePerUnit: 2500, unit: 'butir' };
      mockPrisma.ingredient.findUnique.mockResolvedValue(ingredient);
      mockPrisma.ingredient.update.mockResolvedValue({ ...ingredient, pricePerUnit: 3000 });
      mockPrisma.ingredientPriceLog.create.mockResolvedValue({});

      const result = await service.updatePrice('1', 3000);

      expect(result.pricePerUnit).toBe(3000);
      expect(mockPrisma.ingredientPriceLog.create).toHaveBeenCalledWith({
        data: expect.objectContaining({
          ingredientId: '1',
          oldPrice: 2500,
          newPrice: 3000,
        }),
      });
    });
  });

  describe('getAffectedMenus', () => {
    it('should return menus that use a given ingredient', async () => {
      mockPrisma.recipe.findMany.mockResolvedValue([
        {
          menuItem: { id: 'm1', name: 'Nasi Goreng', sellingPrice: 25000 },
          qtyNeeded: 1,
        },
      ]);

      const result = await service.getAffectedMenus('1');
      expect(result).toHaveLength(1);
      expect(result[0].menuItem.name).toBe('Nasi Goreng');
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/ingredients/ingredients.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement IngredientsService**

```typescript
// backend/src/ingredients/ingredients.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateIngredientDto } from './dto/create-ingredient.dto';
import { UpdateIngredientDto } from './dto/update-ingredient.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class IngredientsService {
  constructor(private prisma: PrismaService) {}

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      this.prisma.ingredient.findMany({
        skip,
        take: limit,
        orderBy: { name: 'asc' },
      }),
      this.prisma.ingredient.count(),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const ingredient = await this.prisma.ingredient.findUnique({
      where: { id },
      include: {
        priceLogs: { orderBy: { changedAt: 'desc' }, take: 10 },
      },
    });
    if (!ingredient) throw new NotFoundException('Ingredient not found');
    return ingredient;
  }

  async create(dto: CreateIngredientDto) {
    return this.prisma.ingredient.create({ data: dto });
  }

  async update(id: string, dto: UpdateIngredientDto) {
    await this.findOne(id);

    if (dto.pricePerUnit !== undefined) {
      return this.updatePrice(id, dto.pricePerUnit);
    }

    return this.prisma.ingredient.update({
      where: { id },
      data: dto,
    });
  }

  async updatePrice(id: string, newPrice: number) {
    const ingredient = await this.prisma.ingredient.findUnique({ where: { id } });
    if (!ingredient) throw new NotFoundException('Ingredient not found');

    const oldPrice = ingredient.pricePerUnit;

    return this.prisma.$transaction(async (tx) => {
      await tx.ingredientPriceLog.create({
        data: {
          ingredientId: id,
          oldPrice,
          newPrice,
        },
      });

      return tx.ingredient.update({
        where: { id },
        data: { pricePerUnit: newPrice },
      });
    });
  }

  async getAffectedMenus(ingredientId: string) {
    return this.prisma.recipe.findMany({
      where: { ingredientId },
      include: {
        menuItem: { select: { id: true, name: true, sellingPrice: true } },
      },
    });
  }

  async getPriceHistory(id: string) {
    return this.prisma.ingredientPriceLog.findMany({
      where: { ingredientId: id },
      orderBy: { changedAt: 'desc' },
    });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.ingredient.delete({ where: { id } });
    return { message: 'Ingredient deleted' };
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/ingredients/ingredients.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create IngredientsController**

```typescript
// backend/src/ingredients/ingredients.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { IngredientsService } from './ingredients.service';
import { CreateIngredientDto } from './dto/create-ingredient.dto';
import { UpdateIngredientDto } from './dto/update-ingredient.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('ingredients')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class IngredientsController {
  constructor(private ingredientsService: IngredientsService) {}

  @Get()
  @RequirePermissions('ingredient:read')
  findAll(@Query() pagination: PaginationDto) {
    return this.ingredientsService.findAll(pagination);
  }

  @Get(':id')
  @RequirePermissions('ingredient:read')
  findOne(@Param('id') id: string) {
    return this.ingredientsService.findOne(id);
  }

  @Get(':id/price-history')
  @RequirePermissions('ingredient:read')
  getPriceHistory(@Param('id') id: string) {
    return this.ingredientsService.getPriceHistory(id);
  }

  @Get(':id/affected-menus')
  @RequirePermissions('ingredient:read')
  getAffectedMenus(@Param('id') id: string) {
    return this.ingredientsService.getAffectedMenus(id);
  }

  @Post()
  @RequirePermissions('ingredient:manage')
  create(@Body() dto: CreateIngredientDto) {
    return this.ingredientsService.create(dto);
  }

  @Patch(':id')
  @RequirePermissions('ingredient:manage')
  update(@Param('id') id: string, @Body() dto: UpdateIngredientDto) {
    return this.ingredientsService.update(id, dto);
  }

  @Delete(':id')
  @RequirePermissions('ingredient:manage')
  remove(@Param('id') id: string) {
    return this.ingredientsService.remove(id);
  }
}
```

- [ ] **Step 7: Create IngredientsModule and register in AppModule**

```typescript
// backend/src/ingredients/ingredients.module.ts
import { Module } from '@nestjs/common';
import { IngredientsController } from './ingredients.controller';
import { IngredientsService } from './ingredients.service';

@Module({
  controllers: [IngredientsController],
  providers: [IngredientsService],
  exports: [IngredientsService],
})
export class IngredientsModule {}
```

Add `IngredientsModule` to `AppModule` imports.

- [ ] **Step 8: Commit**

```bash
git add src/ingredients/ src/app.module.ts
git commit -m "feat: add ingredients module with price tracking and history"
```

---

### Task 7: Menu Module + HPP Engine

**Files:**
- Create: `backend/src/menu/menu.module.ts`
- Create: `backend/src/menu/menu.controller.ts`
- Create: `backend/src/menu/menu.service.ts`
- Create: `backend/src/menu/menu.service.spec.ts`
- Create: `backend/src/menu/dto/create-menu.dto.ts`
- Create: `backend/src/menu/dto/update-menu.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/menu/dto/create-menu.dto.ts
import {
  IsString, IsNumber, IsEnum, IsOptional, IsArray,
  ValidateNested, Min,
} from 'class-validator';
import { Type } from 'class-transformer';

class RecipeItemDto {
  @IsString()
  ingredientId: string;

  @IsNumber()
  @Min(0)
  qtyNeeded: number;

  @IsString()
  unit: string;
}

export class CreateMenuDto {
  @IsString()
  name: string;

  @IsEnum(['FOOD', 'BEVERAGE'])
  category: 'FOOD' | 'BEVERAGE';

  @IsNumber()
  @Min(0)
  sellingPrice: number;

  @IsOptional()
  @IsString()
  imageUrl?: string;

  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => RecipeItemDto)
  recipes: RecipeItemDto[];
}
```

```typescript
// backend/src/menu/dto/update-menu.dto.ts
import {
  IsString, IsNumber, IsEnum, IsOptional, IsArray, IsBoolean,
  ValidateNested, Min,
} from 'class-validator';
import { Type } from 'class-transformer';

class RecipeItemDto {
  @IsString()
  ingredientId: string;

  @IsNumber()
  @Min(0)
  qtyNeeded: number;

  @IsString()
  unit: string;
}

export class UpdateMenuDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsEnum(['FOOD', 'BEVERAGE'])
  category?: 'FOOD' | 'BEVERAGE';

  @IsOptional()
  @IsNumber()
  @Min(0)
  sellingPrice?: number;

  @IsOptional()
  @IsString()
  imageUrl?: string;

  @IsOptional()
  @IsBoolean()
  isActive?: boolean;

  @IsOptional()
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => RecipeItemDto)
  recipes?: RecipeItemDto[];
}
```

- [ ] **Step 2: Write failing test for MenuService (HPP calculation)**

```typescript
// backend/src/menu/menu.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { MenuService } from './menu.service';
import { PrismaService } from '../prisma/prisma.service';

describe('MenuService', () => {
  let service: MenuService;

  const mockPrisma = {
    menuItem: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
    recipe: {
      deleteMany: jest.fn(),
      createMany: jest.fn(),
    },
    $transaction: jest.fn((cb) => cb(mockPrisma)),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        MenuService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<MenuService>(MenuService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('calculateHpp', () => {
    it('should calculate HPP from recipe ingredients', () => {
      const recipes = [
        { qtyNeeded: 0.2, ingredient: { pricePerUnit: 12000, unit: 'kg' } },  // Nasi 200gr = 2400
        { qtyNeeded: 1, ingredient: { pricePerUnit: 2500, unit: 'butir' } },   // Telur = 2500
        { qtyNeeded: 0.03, ingredient: { pricePerUnit: 18000, unit: 'liter' } }, // Minyak 30ml = 540
      ];

      const hpp = service.calculateHpp(recipes as any);
      expect(hpp).toBe(5440);
    });

    it('should return 0 for empty recipe', () => {
      const hpp = service.calculateHpp([]);
      expect(hpp).toBe(0);
    });
  });

  describe('getHppBreakdown', () => {
    it('should return full HPP breakdown for a menu item', async () => {
      mockPrisma.menuItem.findUnique.mockResolvedValue({
        id: 'm1',
        name: 'Nasi Goreng',
        sellingPrice: 25000,
        recipes: [
          {
            id: 'r1',
            qtyNeeded: 0.2,
            unit: 'kg',
            ingredient: { id: 'i1', name: 'Nasi', pricePerUnit: 12000, unit: 'kg' },
          },
          {
            id: 'r2',
            qtyNeeded: 1,
            unit: 'butir',
            ingredient: { id: 'i2', name: 'Telur', pricePerUnit: 2500, unit: 'butir' },
          },
        ],
      });

      const result = await service.getHppBreakdown('m1');

      expect(result.totalHpp).toBe(4900);
      expect(result.profit).toBe(20100);
      expect(result.margin).toBeCloseTo(80.4, 1);
      expect(result.breakdown).toHaveLength(2);
      expect(result.breakdown[0]).toMatchObject({
        ingredientName: 'Nasi',
        cost: 2400,
      });
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/menu/menu.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement MenuService**

```typescript
// backend/src/menu/menu.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateMenuDto } from './dto/create-menu.dto';
import { UpdateMenuDto } from './dto/update-menu.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class MenuService {
  constructor(private prisma: PrismaService) {}

  calculateHpp(recipes: { qtyNeeded: number; ingredient: { pricePerUnit: number } }[]): number {
    return recipes.reduce((sum, r) => sum + r.qtyNeeded * r.ingredient.pricePerUnit, 0);
  }

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [items, total] = await Promise.all([
      this.prisma.menuItem.findMany({
        skip,
        take: limit,
        include: {
          recipes: { include: { ingredient: true } },
        },
        orderBy: { name: 'asc' },
      }),
      this.prisma.menuItem.count(),
    ]);

    const data = items.map((item) => {
      const hpp = this.calculateHpp(item.recipes);
      return {
        ...item,
        hpp,
        profit: item.sellingPrice - hpp,
        margin: item.sellingPrice > 0 ? ((item.sellingPrice - hpp) / item.sellingPrice) * 100 : 0,
      };
    });

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const item = await this.prisma.menuItem.findUnique({
      where: { id },
      include: {
        recipes: { include: { ingredient: true } },
      },
    });
    if (!item) throw new NotFoundException('Menu item not found');

    const hpp = this.calculateHpp(item.recipes);
    return {
      ...item,
      hpp,
      profit: item.sellingPrice - hpp,
      margin: item.sellingPrice > 0 ? ((item.sellingPrice - hpp) / item.sellingPrice) * 100 : 0,
    };
  }

  async getHppBreakdown(id: string) {
    const item = await this.prisma.menuItem.findUnique({
      where: { id },
      include: {
        recipes: { include: { ingredient: true } },
      },
    });
    if (!item) throw new NotFoundException('Menu item not found');

    const breakdown = item.recipes.map((r) => ({
      ingredientId: r.ingredient.id,
      ingredientName: r.ingredient.name,
      qtyNeeded: r.qtyNeeded,
      unit: r.unit,
      pricePerUnit: r.ingredient.pricePerUnit,
      cost: r.qtyNeeded * r.ingredient.pricePerUnit,
    }));

    const totalHpp = breakdown.reduce((sum, b) => sum + b.cost, 0);

    return {
      menuItemId: item.id,
      menuItemName: item.name,
      sellingPrice: item.sellingPrice,
      totalHpp,
      profit: item.sellingPrice - totalHpp,
      margin: item.sellingPrice > 0 ? ((item.sellingPrice - totalHpp) / item.sellingPrice) * 100 : 0,
      breakdown,
    };
  }

  async create(dto: CreateMenuDto) {
    const { recipes, ...menuData } = dto;

    return this.prisma.$transaction(async (tx) => {
      const menuItem = await tx.menuItem.create({
        data: {
          ...menuData,
          recipes: {
            create: recipes.map((r) => ({
              ingredientId: r.ingredientId,
              qtyNeeded: r.qtyNeeded,
              unit: r.unit,
            })),
          },
        },
        include: {
          recipes: { include: { ingredient: true } },
        },
      });

      return menuItem;
    });
  }

  async update(id: string, dto: UpdateMenuDto) {
    await this.findOne(id);
    const { recipes, ...menuData } = dto;

    return this.prisma.$transaction(async (tx) => {
      if (recipes) {
        await tx.recipe.deleteMany({ where: { menuItemId: id } });
        await tx.recipe.createMany({
          data: recipes.map((r) => ({
            menuItemId: id,
            ingredientId: r.ingredientId,
            qtyNeeded: r.qtyNeeded,
            unit: r.unit,
          })),
        });
      }

      return tx.menuItem.update({
        where: { id },
        data: menuData,
        include: {
          recipes: { include: { ingredient: true } },
        },
      });
    });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.menuItem.delete({ where: { id } });
    return { message: 'Menu item deleted' };
  }

  async getMenusWithLowMargin(threshold: number = 30) {
    const items = await this.prisma.menuItem.findMany({
      where: { isActive: true },
      include: { recipes: { include: { ingredient: true } } },
    });

    return items
      .map((item) => {
        const hpp = this.calculateHpp(item.recipes);
        const margin = item.sellingPrice > 0
          ? ((item.sellingPrice - hpp) / item.sellingPrice) * 100
          : 0;
        return { ...item, hpp, margin };
      })
      .filter((item) => item.margin < threshold);
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/menu/menu.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create MenuController**

```typescript
// backend/src/menu/menu.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { MenuService } from './menu.service';
import { CreateMenuDto } from './dto/create-menu.dto';
import { UpdateMenuDto } from './dto/update-menu.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('menu')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class MenuController {
  constructor(private menuService: MenuService) {}

  @Get()
  @RequirePermissions('menu:read')
  findAll(@Query() pagination: PaginationDto) {
    return this.menuService.findAll(pagination);
  }

  @Get('low-margin')
  @RequirePermissions('dashboard:view')
  getLowMargin(@Query('threshold') threshold?: number) {
    return this.menuService.getMenusWithLowMargin(threshold);
  }

  @Get(':id')
  @RequirePermissions('menu:read')
  findOne(@Param('id') id: string) {
    return this.menuService.findOne(id);
  }

  @Get(':id/hpp')
  @RequirePermissions('menu:read')
  getHpp(@Param('id') id: string) {
    return this.menuService.getHppBreakdown(id);
  }

  @Post()
  @RequirePermissions('menu:create')
  create(@Body() dto: CreateMenuDto) {
    return this.menuService.create(dto);
  }

  @Patch(':id')
  @RequirePermissions('menu:update')
  update(@Param('id') id: string, @Body() dto: UpdateMenuDto) {
    return this.menuService.update(id, dto);
  }

  @Delete(':id')
  @RequirePermissions('menu:delete')
  remove(@Param('id') id: string) {
    return this.menuService.remove(id);
  }
}
```

- [ ] **Step 7: Create MenuModule and register in AppModule**

```typescript
// backend/src/menu/menu.module.ts
import { Module } from '@nestjs/common';
import { MenuController } from './menu.controller';
import { MenuService } from './menu.service';

@Module({
  controllers: [MenuController],
  providers: [MenuService],
  exports: [MenuService],
})
export class MenuModule {}
```

Add `MenuModule` to `AppModule` imports.

- [ ] **Step 8: Commit**

```bash
git add src/menu/ src/app.module.ts
git commit -m "feat: add menu module with HPP engine and recipe management"
```

---

### Task 8: Retail Module

**Files:**
- Create: `backend/src/retail/retail.module.ts`
- Create: `backend/src/retail/retail.controller.ts`
- Create: `backend/src/retail/retail.service.ts`
- Create: `backend/src/retail/retail.service.spec.ts`
- Create: `backend/src/retail/dto/create-retail.dto.ts`
- Create: `backend/src/retail/dto/update-retail.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/retail/dto/create-retail.dto.ts
import { IsString, IsNumber, IsOptional, Min } from 'class-validator';

export class CreateRetailDto {
  @IsString()
  name: string;

  @IsString()
  category: string;

  @IsNumber()
  @Min(0)
  buyPrice: number;

  @IsNumber()
  @Min(0)
  sellingPrice: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  stockQty?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  minStock?: number;

  @IsOptional()
  @IsString()
  imageUrl?: string;
}
```

```typescript
// backend/src/retail/dto/update-retail.dto.ts
import { IsString, IsNumber, IsOptional, Min } from 'class-validator';

export class UpdateRetailDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsString()
  category?: string;

  @IsOptional()
  @IsNumber()
  @Min(0)
  buyPrice?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  sellingPrice?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  stockQty?: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  minStock?: number;

  @IsOptional()
  @IsString()
  imageUrl?: string;
}
```

- [ ] **Step 2: Write failing test**

```typescript
// backend/src/retail/retail.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { RetailService } from './retail.service';
import { PrismaService } from '../prisma/prisma.service';

describe('RetailService', () => {
  let service: RetailService;

  const mockPrisma = {
    retailItem: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        RetailService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<RetailService>(RetailService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('findAll', () => {
    it('should return retail items with profit calculation', async () => {
      mockPrisma.retailItem.findMany.mockResolvedValue([
        { id: '1', name: 'Saus Sambal', buyPrice: 15000, sellingPrice: 25000 },
      ]);
      mockPrisma.retailItem.count.mockResolvedValue(1);

      const result = await service.findAll({ page: 1, limit: 20 });

      expect(result.data[0].profit).toBe(10000);
      expect(result.data[0].margin).toBeCloseTo(40, 1);
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/retail/retail.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement RetailService**

```typescript
// backend/src/retail/retail.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateRetailDto } from './dto/create-retail.dto';
import { UpdateRetailDto } from './dto/update-retail.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class RetailService {
  constructor(private prisma: PrismaService) {}

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [items, total] = await Promise.all([
      this.prisma.retailItem.findMany({ skip, take: limit, orderBy: { name: 'asc' } }),
      this.prisma.retailItem.count(),
    ]);

    const data = items.map((item) => ({
      ...item,
      profit: item.sellingPrice - item.buyPrice,
      margin: item.sellingPrice > 0
        ? ((item.sellingPrice - item.buyPrice) / item.sellingPrice) * 100
        : 0,
    }));

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const item = await this.prisma.retailItem.findUnique({ where: { id } });
    if (!item) throw new NotFoundException('Retail item not found');
    return {
      ...item,
      profit: item.sellingPrice - item.buyPrice,
      margin: item.sellingPrice > 0
        ? ((item.sellingPrice - item.buyPrice) / item.sellingPrice) * 100
        : 0,
    };
  }

  async create(dto: CreateRetailDto) {
    return this.prisma.retailItem.create({ data: dto });
  }

  async update(id: string, dto: UpdateRetailDto) {
    await this.findOne(id);
    return this.prisma.retailItem.update({ where: { id }, data: dto });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.retailItem.delete({ where: { id } });
    return { message: 'Retail item deleted' };
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/retail/retail.service.spec.ts --verbose
```

Expected: PASS

- [ ] **Step 6: Create RetailController, RetailModule, register in AppModule**

```typescript
// backend/src/retail/retail.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { RetailService } from './retail.service';
import { CreateRetailDto } from './dto/create-retail.dto';
import { UpdateRetailDto } from './dto/update-retail.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('retail')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class RetailController {
  constructor(private retailService: RetailService) {}

  @Get()
  @RequirePermissions('menu:read')
  findAll(@Query() pagination: PaginationDto) {
    return this.retailService.findAll(pagination);
  }

  @Get(':id')
  @RequirePermissions('menu:read')
  findOne(@Param('id') id: string) {
    return this.retailService.findOne(id);
  }

  @Post()
  @RequirePermissions('menu:create')
  create(@Body() dto: CreateRetailDto) {
    return this.retailService.create(dto);
  }

  @Patch(':id')
  @RequirePermissions('menu:update')
  update(@Param('id') id: string, @Body() dto: UpdateRetailDto) {
    return this.retailService.update(id, dto);
  }

  @Delete(':id')
  @RequirePermissions('menu:delete')
  remove(@Param('id') id: string) {
    return this.retailService.remove(id);
  }
}
```

```typescript
// backend/src/retail/retail.module.ts
import { Module } from '@nestjs/common';
import { RetailController } from './retail.controller';
import { RetailService } from './retail.service';

@Module({
  controllers: [RetailController],
  providers: [RetailService],
  exports: [RetailService],
})
export class RetailModule {}
```

Add `RetailModule` to `AppModule` imports.

- [ ] **Step 7: Commit**

```bash
git add src/retail/ src/app.module.ts
git commit -m "feat: add retail module with profit calculation"
```

---

### Task 9: Promos Module

**Files:**
- Create: `backend/src/promos/promos.module.ts`
- Create: `backend/src/promos/promos.controller.ts`
- Create: `backend/src/promos/promos.service.ts`
- Create: `backend/src/promos/promos.service.spec.ts`
- Create: `backend/src/promos/dto/create-promo.dto.ts`
- Create: `backend/src/promos/dto/apply-promo.dto.ts`

- [ ] **Step 1: Create DTOs**

```typescript
// backend/src/promos/dto/create-promo.dto.ts
import {
  IsString, IsNumber, IsEnum, IsOptional, IsArray, IsDateString,
  IsBoolean, IsInt, ValidateNested, Min,
} from 'class-validator';
import { Type } from 'class-transformer';

class PromoItemDto {
  @IsEnum(['MENU', 'RETAIL'])
  itemType: 'MENU' | 'RETAIL';

  @IsString()
  itemId: string;

  @IsEnum(['TARGET', 'FREE_ITEM', 'BUNDLE_MEMBER'])
  role: 'TARGET' | 'FREE_ITEM' | 'BUNDLE_MEMBER';
}

export class CreatePromoDto {
  @IsString()
  name: string;

  @IsEnum(['PERCENT', 'FIXED', 'BUY_X_GET_Y', 'BUNDLE', 'MIN_PURCHASE'])
  type: 'PERCENT' | 'FIXED' | 'BUY_X_GET_Y' | 'BUNDLE' | 'MIN_PURCHASE';

  @IsNumber()
  @Min(0)
  value: number;

  @IsOptional()
  @IsNumber()
  @Min(0)
  minPurchaseAmount?: number;

  @IsDateString()
  startDate: string;

  @IsDateString()
  endDate: string;

  @IsOptional()
  @IsBoolean()
  isActive?: boolean;

  @IsOptional()
  @IsInt()
  @Min(1)
  maxUsage?: number;

  @IsOptional()
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => PromoItemDto)
  items?: PromoItemDto[];
}
```

```typescript
// backend/src/promos/dto/apply-promo.dto.ts
import { IsString } from 'class-validator';

export class ApplyPromoDto {
  @IsString()
  promoId: string;
}
```

- [ ] **Step 2: Write failing test for PromosService**

```typescript
// backend/src/promos/promos.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { PromosService } from './promos.service';
import { PrismaService } from '../prisma/prisma.service';
import { BadRequestException } from '@nestjs/common';

describe('PromosService', () => {
  let service: PromosService;

  const mockPrisma = {
    promo: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        PromosService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<PromosService>(PromosService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('calculateDiscount', () => {
    it('should calculate PERCENT discount correctly', () => {
      const promo = { type: 'PERCENT', value: 20 } as any;
      const discount = service.calculateDiscount(promo, 100000);
      expect(discount).toBe(20000);
    });

    it('should calculate FIXED discount correctly', () => {
      const promo = { type: 'FIXED', value: 5000 } as any;
      const discount = service.calculateDiscount(promo, 100000);
      expect(discount).toBe(5000);
    });

    it('should not exceed total amount for FIXED discount', () => {
      const promo = { type: 'FIXED', value: 50000 } as any;
      const discount = service.calculateDiscount(promo, 30000);
      expect(discount).toBe(30000);
    });

    it('should calculate MIN_PURCHASE discount only if threshold met', () => {
      const promo = { type: 'MIN_PURCHASE', value: 10, minPurchaseAmount: 100000 } as any;
      expect(service.calculateDiscount(promo, 150000)).toBe(15000);
      expect(service.calculateDiscount(promo, 50000)).toBe(0);
    });
  });

  describe('validatePromo', () => {
    it('should reject expired promo', () => {
      const promo = {
        isActive: true,
        startDate: new Date('2020-01-01'),
        endDate: new Date('2020-12-31'),
        maxUsage: null,
        usedCount: 0,
      } as any;

      expect(() => service.validatePromo(promo)).toThrow(BadRequestException);
    });

    it('should reject promo that exceeded max usage', () => {
      const promo = {
        isActive: true,
        startDate: new Date('2020-01-01'),
        endDate: new Date('2030-12-31'),
        maxUsage: 10,
        usedCount: 10,
      } as any;

      expect(() => service.validatePromo(promo)).toThrow(BadRequestException);
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/promos/promos.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement PromosService**

```typescript
// backend/src/promos/promos.service.ts
import { Injectable, NotFoundException, BadRequestException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreatePromoDto } from './dto/create-promo.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class PromosService {
  constructor(private prisma: PrismaService) {}

  calculateDiscount(
    promo: { type: string; value: number; minPurchaseAmount?: number },
    totalAmount: number,
  ): number {
    switch (promo.type) {
      case 'PERCENT':
        return (promo.value / 100) * totalAmount;
      case 'FIXED':
        return Math.min(promo.value, totalAmount);
      case 'MIN_PURCHASE':
        if (promo.minPurchaseAmount && totalAmount >= promo.minPurchaseAmount) {
          return (promo.value / 100) * totalAmount;
        }
        return 0;
      default:
        return 0;
    }
  }

  validatePromo(promo: {
    isActive: boolean;
    startDate: Date;
    endDate: Date;
    maxUsage: number | null;
    usedCount: number;
  }) {
    if (!promo.isActive) {
      throw new BadRequestException('Promo is not active');
    }

    const now = new Date();
    if (now < promo.startDate || now > promo.endDate) {
      throw new BadRequestException('Promo is not within valid period');
    }

    if (promo.maxUsage !== null && promo.usedCount >= promo.maxUsage) {
      throw new BadRequestException('Promo has reached maximum usage');
    }
  }

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      this.prisma.promo.findMany({
        skip,
        take: limit,
        include: { items: true },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.promo.count(),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const promo = await this.prisma.promo.findUnique({
      where: { id },
      include: { items: true },
    });
    if (!promo) throw new NotFoundException('Promo not found');
    return promo;
  }

  async getActivePromos() {
    const now = new Date();
    return this.prisma.promo.findMany({
      where: {
        isActive: true,
        startDate: { lte: now },
        endDate: { gte: now },
      },
      include: { items: true },
    });
  }

  async create(dto: CreatePromoDto) {
    const { items, ...promoData } = dto;

    return this.prisma.promo.create({
      data: {
        ...promoData,
        startDate: new Date(promoData.startDate),
        endDate: new Date(promoData.endDate),
        items: items
          ? { create: items.map((i) => ({ itemType: i.itemType, itemId: i.itemId, role: i.role })) }
          : undefined,
      },
      include: { items: true },
    });
  }

  async update(id: string, dto: Partial<CreatePromoDto>) {
    await this.findOne(id);
    const { items, ...promoData } = dto;

    const data: any = { ...promoData };
    if (promoData.startDate) data.startDate = new Date(promoData.startDate);
    if (promoData.endDate) data.endDate = new Date(promoData.endDate);

    return this.prisma.promo.update({
      where: { id },
      data,
      include: { items: true },
    });
  }

  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.promo.delete({ where: { id } });
    return { message: 'Promo deleted' };
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/promos/promos.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create PromosController, PromosModule, register in AppModule**

```typescript
// backend/src/promos/promos.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { PromosService } from './promos.service';
import { CreatePromoDto } from './dto/create-promo.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('promos')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class PromosController {
  constructor(private promosService: PromosService) {}

  @Get()
  @RequirePermissions('promo:manage')
  findAll(@Query() pagination: PaginationDto) {
    return this.promosService.findAll(pagination);
  }

  @Get('active')
  @RequirePermissions('promo:apply')
  getActive() {
    return this.promosService.getActivePromos();
  }

  @Get(':id')
  @RequirePermissions('promo:manage')
  findOne(@Param('id') id: string) {
    return this.promosService.findOne(id);
  }

  @Post()
  @RequirePermissions('promo:manage')
  create(@Body() dto: CreatePromoDto) {
    return this.promosService.create(dto);
  }

  @Patch(':id')
  @RequirePermissions('promo:manage')
  update(@Param('id') id: string, @Body() dto: Partial<CreatePromoDto>) {
    return this.promosService.update(id, dto);
  }

  @Delete(':id')
  @RequirePermissions('promo:manage')
  remove(@Param('id') id: string) {
    return this.promosService.remove(id);
  }
}
```

```typescript
// backend/src/promos/promos.module.ts
import { Module } from '@nestjs/common';
import { PromosController } from './promos.controller';
import { PromosService } from './promos.service';

@Module({
  controllers: [PromosController],
  providers: [PromosService],
  exports: [PromosService],
})
export class PromosModule {}
```

Add `PromosModule` to `AppModule` imports.

- [ ] **Step 7: Commit**

```bash
git add src/promos/ src/app.module.ts
git commit -m "feat: add promos module with discount calculation and validation"
```

---

### Task 10: POS / Transaction Module

**Files:**
- Create: `backend/src/pos/pos.module.ts`
- Create: `backend/src/pos/pos.controller.ts`
- Create: `backend/src/pos/pos.service.ts`
- Create: `backend/src/pos/pos.service.spec.ts`
- Create: `backend/src/pos/dto/create-transaction.dto.ts`

- [ ] **Step 1: Create DTO**

```typescript
// backend/src/pos/dto/create-transaction.dto.ts
import {
  IsEnum, IsArray, IsOptional, IsString, IsInt,
  ValidateNested, Min,
} from 'class-validator';
import { Type } from 'class-transformer';

class TransactionItemDto {
  @IsEnum(['MENU', 'RETAIL'])
  itemType: 'MENU' | 'RETAIL';

  @IsString()
  itemId: string;

  @IsInt()
  @Min(1)
  qty: number;
}

export class CreateTransactionDto {
  @IsEnum(['CASH', 'QRIS', 'EDC'])
  paymentMethod: 'CASH' | 'QRIS' | 'EDC';

  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => TransactionItemDto)
  items: TransactionItemDto[];

  @IsOptional()
  @IsString()
  promoId?: string;
}
```

- [ ] **Step 2: Write failing test for PosService**

```typescript
// backend/src/pos/pos.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { PosService } from './pos.service';
import { PrismaService } from '../prisma/prisma.service';
import { MenuService } from '../menu/menu.service';
import { RetailService } from '../retail/retail.service';
import { PromosService } from '../promos/promos.service';

describe('PosService', () => {
  let service: PosService;

  const mockPrisma = {
    transaction: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      count: jest.fn(),
    },
    menuItem: { findUnique: jest.fn() },
    retailItem: { findUnique: jest.fn() },
    promo: { findUnique: jest.fn(), update: jest.fn() },
    order: { count: jest.fn() },
    $transaction: jest.fn((cb) => cb(mockPrisma)),
  };

  const mockMenuService = {
    calculateHpp: jest.fn(),
  };

  const mockRetailService = {};

  const mockPromosService = {
    validatePromo: jest.fn(),
    calculateDiscount: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        PosService,
        { provide: PrismaService, useValue: mockPrisma },
        { provide: MenuService, useValue: mockMenuService },
        { provide: RetailService, useValue: mockRetailService },
        { provide: PromosService, useValue: mockPromosService },
      ],
    }).compile();

    service = module.get<PosService>(PosService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('generateTransactionCode', () => {
    it('should generate code in format TRX-YYYYMMDD-XXXX', () => {
      const code = service.generateTransactionCode(5);
      const today = new Date().toISOString().slice(0, 10).replace(/-/g, '');
      expect(code).toMatch(new RegExp(`^TRX-${today}-\\d{4}$`));
    });
  });

  describe('buildTransactionItems', () => {
    it('should build items with HPP snapshot for menu items', async () => {
      mockPrisma.menuItem.findUnique.mockResolvedValue({
        id: 'm1',
        sellingPrice: 25000,
        recipes: [
          { qtyNeeded: 0.2, ingredient: { pricePerUnit: 12000 } },
          { qtyNeeded: 1, ingredient: { pricePerUnit: 2500 } },
        ],
      });
      mockMenuService.calculateHpp.mockReturnValue(4900);

      const items = await service.buildTransactionItems(
        [{ itemType: 'MENU' as const, itemId: 'm1', qty: 2 }],
        mockPrisma as any,
      );

      expect(items).toHaveLength(1);
      expect(items[0].unitPrice).toBe(25000);
      expect(items[0].unitHpp).toBe(4900);
      expect(items[0].subtotal).toBe(50000);
      expect(items[0].subHpp).toBe(9800);
    });

    it('should build items with buy price as HPP for retail items', async () => {
      mockPrisma.retailItem.findUnique.mockResolvedValue({
        id: 'r1',
        sellingPrice: 25000,
        buyPrice: 15000,
      });

      const items = await service.buildTransactionItems(
        [{ itemType: 'RETAIL' as const, itemId: 'r1', qty: 1 }],
        mockPrisma as any,
      );

      expect(items[0].unitHpp).toBe(15000);
      expect(items[0].subtotal).toBe(25000);
      expect(items[0].subHpp).toBe(15000);
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/pos/pos.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement PosService**

```typescript
// backend/src/pos/pos.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { MenuService } from '../menu/menu.service';
import { PromosService } from '../promos/promos.service';
import { CreateTransactionDto } from './dto/create-transaction.dto';
import { PaginationDto } from '../common/dto/pagination.dto';

@Injectable()
export class PosService {
  constructor(
    private prisma: PrismaService,
    private menuService: MenuService,
    private promosService: PromosService,
  ) {}

  generateTransactionCode(count: number): string {
    const today = new Date().toISOString().slice(0, 10).replace(/-/g, '');
    const seq = String(count + 1).padStart(4, '0');
    return `TRX-${today}-${seq}`;
  }

  async buildTransactionItems(
    items: { itemType: 'MENU' | 'RETAIL'; itemId: string; qty: number }[],
    tx: any,
  ) {
    const result = [];

    for (const item of items) {
      if (item.itemType === 'MENU') {
        const menuItem = await tx.menuItem.findUnique({
          where: { id: item.itemId },
          include: { recipes: { include: { ingredient: true } } },
        });
        if (!menuItem) throw new NotFoundException(`Menu item ${item.itemId} not found`);

        const unitHpp = this.menuService.calculateHpp(menuItem.recipes);
        result.push({
          itemType: 'MENU' as const,
          itemId: item.itemId,
          qty: item.qty,
          unitPrice: menuItem.sellingPrice,
          unitHpp,
          subtotal: menuItem.sellingPrice * item.qty,
          subHpp: unitHpp * item.qty,
        });
      } else {
        const retailItem = await tx.retailItem.findUnique({
          where: { id: item.itemId },
        });
        if (!retailItem) throw new NotFoundException(`Retail item ${item.itemId} not found`);

        result.push({
          itemType: 'RETAIL' as const,
          itemId: item.itemId,
          qty: item.qty,
          unitPrice: retailItem.sellingPrice,
          unitHpp: retailItem.buyPrice,
          subtotal: retailItem.sellingPrice * item.qty,
          subHpp: retailItem.buyPrice * item.qty,
        });
      }
    }

    return result;
  }

  async create(dto: CreateTransactionDto, cashierId: string) {
    return this.prisma.$transaction(async (tx) => {
      const transactionItems = await this.buildTransactionItems(dto.items, tx);

      const totalAmount = transactionItems.reduce((sum, i) => sum + i.subtotal, 0);
      const totalHpp = transactionItems.reduce((sum, i) => sum + i.subHpp, 0);

      let promoDiscount = 0;
      if (dto.promoId) {
        const promo = await tx.promo.findUnique({
          where: { id: dto.promoId },
          include: { items: true },
        });
        if (!promo) throw new NotFoundException('Promo not found');
        this.promosService.validatePromo(promo);
        promoDiscount = this.promosService.calculateDiscount(promo, totalAmount);
      }

      const todayCount = await tx.order.count({
        where: {
          createdAt: {
            gte: new Date(new Date().toISOString().slice(0, 10)),
          },
        },
      });
      const transactionCode = this.generateTransactionCode(todayCount);

      const transaction = await tx.transaction.create({
        data: {
          transactionCode,
          cashierId,
          paymentMethod: dto.paymentMethod,
          totalAmount: totalAmount - promoDiscount,
          totalHpp,
          totalProfit: totalAmount - promoDiscount - totalHpp,
          promoDiscount,
          items: { create: transactionItems },
          promos: dto.promoId
            ? { create: { promoId: dto.promoId, discountAmount: promoDiscount } }
            : undefined,
        },
        include: {
          items: true,
          promos: { include: { promo: true } },
          cashier: { select: { id: true, name: true } },
        },
      });

      if (dto.promoId) {
        await tx.promo.update({
          where: { id: dto.promoId },
          data: { usedCount: { increment: 1 } },
        });
      }

      return transaction;
    });
  }

  async findAll(pagination: PaginationDto) {
    const { page, limit } = pagination;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      this.prisma.transaction.findMany({
        skip,
        take: limit,
        include: {
          items: true,
          cashier: { select: { id: true, name: true } },
          promos: { include: { promo: true } },
        },
        orderBy: { createdAt: 'desc' },
      }),
      this.prisma.transaction.count(),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const transaction = await this.prisma.transaction.findUnique({
      where: { id },
      include: {
        items: true,
        cashier: { select: { id: true, name: true } },
        promos: { include: { promo: true } },
        orders: { include: { items: { include: { menuItem: true } } } },
      },
    });
    if (!transaction) throw new NotFoundException('Transaction not found');
    return transaction;
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/pos/pos.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create PosController, PosModule, register in AppModule**

```typescript
// backend/src/pos/pos.controller.ts
import {
  Controller, Get, Post, Body, Param, Query, UseGuards,
} from '@nestjs/common';
import { PosService } from './pos.service';
import { CreateTransactionDto } from './dto/create-transaction.dto';
import { PaginationDto } from '../common/dto/pagination.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';
import { CurrentUser } from '../common/decorators/current-user.decorator';

@Controller('pos')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class PosController {
  constructor(private posService: PosService) {}

  @Post('transactions')
  @RequirePermissions('transaction:create')
  create(
    @Body() dto: CreateTransactionDto,
    @CurrentUser('id') cashierId: string,
  ) {
    return this.posService.create(dto, cashierId);
  }

  @Get('transactions')
  @RequirePermissions('transaction:read')
  findAll(@Query() pagination: PaginationDto) {
    return this.posService.findAll(pagination);
  }

  @Get('transactions/:id')
  @RequirePermissions('transaction:read')
  findOne(@Param('id') id: string) {
    return this.posService.findOne(id);
  }
}
```

```typescript
// backend/src/pos/pos.module.ts
import { Module } from '@nestjs/common';
import { PosController } from './pos.controller';
import { PosService } from './pos.service';
import { MenuModule } from '../menu/menu.module';
import { PromosModule } from '../promos/promos.module';

@Module({
  imports: [MenuModule, PromosModule],
  controllers: [PosController],
  providers: [PosService],
  exports: [PosService],
})
export class PosModule {}
```

Add `PosModule` to `AppModule` imports.

- [ ] **Step 7: Commit**

```bash
git add src/pos/ src/app.module.ts
git commit -m "feat: add POS module with transaction creation and HPP snapshot"
```

---

### Task 11: Orders Module + WebSocket Gateway

**Files:**
- Create: `backend/src/orders/orders.module.ts`
- Create: `backend/src/orders/orders.controller.ts`
- Create: `backend/src/orders/orders.service.ts`
- Create: `backend/src/orders/orders.service.spec.ts`
- Create: `backend/src/orders/orders.gateway.ts`
- Create: `backend/src/orders/dto/update-order-status.dto.ts`

- [ ] **Step 1: Create DTO**

```typescript
// backend/src/orders/dto/update-order-status.dto.ts
import { IsEnum, IsOptional } from 'class-validator';

export class UpdateOrderStatusDto {
  @IsEnum(['PENDING', 'PREPARING', 'READY', 'SERVED'])
  status: 'PENDING' | 'PREPARING' | 'READY' | 'SERVED';
}

export class UpdateOrderItemStatusDto {
  @IsEnum(['PENDING', 'PREPARING', 'DONE'])
  status: 'PENDING' | 'PREPARING' | 'DONE';
}
```

- [ ] **Step 2: Write failing test for OrdersService**

```typescript
// backend/src/orders/orders.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { OrdersService } from './orders.service';
import { PrismaService } from '../prisma/prisma.service';

describe('OrdersService', () => {
  let service: OrdersService;

  const mockPrisma = {
    order: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      count: jest.fn(),
    },
    orderItem: {
      update: jest.fn(),
      findMany: jest.fn(),
    },
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<OrdersService>(OrdersService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('createFromTransaction', () => {
    it('should create an order with menu items from transaction', async () => {
      const menuItems = [
        { itemType: 'MENU', itemId: 'm1', qty: 2 },
        { itemType: 'RETAIL', itemId: 'r1', qty: 1 },
      ];

      mockPrisma.order.count.mockResolvedValue(5);
      mockPrisma.order.create.mockResolvedValue({
        id: 'o1',
        transactionId: 't1',
        orderNumber: 6,
        status: 'PENDING',
        items: [{ menuItemId: 'm1', qty: 2, status: 'PENDING' }],
      });

      const result = await service.createFromTransaction('t1', menuItems as any);

      expect(result.orderNumber).toBe(6);
      expect(mockPrisma.order.create).toHaveBeenCalledWith(
        expect.objectContaining({
          data: expect.objectContaining({
            transactionId: 't1',
            items: {
              create: [{ menuItemId: 'm1', qty: 2 }],
            },
          }),
        }),
      );
    });

    it('should skip retail items (not sent to kitchen)', async () => {
      const menuItems = [
        { itemType: 'RETAIL', itemId: 'r1', qty: 1 },
      ];

      mockPrisma.order.count.mockResolvedValue(0);

      const result = await service.createFromTransaction('t1', menuItems as any);

      expect(result).toBeNull();
    });
  });

  describe('updateStatus', () => {
    it('should update order status', async () => {
      mockPrisma.order.findUnique.mockResolvedValue({
        id: 'o1',
        status: 'PENDING',
      });
      mockPrisma.order.update.mockResolvedValue({
        id: 'o1',
        status: 'PREPARING',
      });

      const result = await service.updateStatus('o1', 'PREPARING');
      expect(result.status).toBe('PREPARING');
    });
  });
});
```

- [ ] **Step 3: Run test to verify it fails**

```bash
npx jest src/orders/orders.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 4: Implement OrdersService**

```typescript
// backend/src/orders/orders.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class OrdersService {
  constructor(private prisma: PrismaService) {}

  async createFromTransaction(
    transactionId: string,
    items: { itemType: string; itemId: string; qty: number }[],
  ) {
    const menuItems = items.filter((i) => i.itemType === 'MENU');
    if (menuItems.length === 0) return null;

    const todayStart = new Date(new Date().toISOString().slice(0, 10));
    const count = await this.prisma.order.count({
      where: { createdAt: { gte: todayStart } },
    });

    return this.prisma.order.create({
      data: {
        transactionId,
        orderNumber: count + 1,
        items: {
          create: menuItems.map((i) => ({
            menuItemId: i.itemId,
            qty: i.qty,
          })),
        },
      },
      include: {
        items: { include: { menuItem: true } },
      },
    });
  }

  async getActiveOrders() {
    return this.prisma.order.findMany({
      where: {
        status: { in: ['PENDING', 'PREPARING', 'READY'] },
      },
      include: {
        items: { include: { menuItem: true } },
      },
      orderBy: { createdAt: 'asc' },
    });
  }

  async findOne(id: string) {
    const order = await this.prisma.order.findUnique({
      where: { id },
      include: {
        items: { include: { menuItem: true } },
        transaction: { include: { cashier: { select: { name: true } } } },
      },
    });
    if (!order) throw new NotFoundException('Order not found');
    return order;
  }

  async updateStatus(id: string, status: string) {
    const order = await this.prisma.order.findUnique({ where: { id } });
    if (!order) throw new NotFoundException('Order not found');

    return this.prisma.order.update({
      where: { id },
      data: { status: status as any },
      include: { items: { include: { menuItem: true } } },
    });
  }

  async updateItemStatus(itemId: string, status: string) {
    return this.prisma.orderItem.update({
      where: { id: itemId },
      data: { status: status as any },
      include: { menuItem: true },
    });
  }
}
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/orders/orders.service.spec.ts --verbose
```

Expected: All tests PASS

- [ ] **Step 6: Create OrdersGateway (WebSocket)**

```typescript
// backend/src/orders/orders.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { OrdersService } from './orders.service';

@WebSocketGateway({
  cors: { origin: '*' },
  namespace: '/orders',
})
export class OrdersGateway {
  @WebSocketServer()
  server: Server;

  constructor(private ordersService: OrdersService) {}

  emitNewOrder(order: any) {
    this.server.emit('order:new', order);
  }

  @SubscribeMessage('order:update_status')
  async handleUpdateStatus(
    @MessageBody() data: { orderId: string; status: string },
    @ConnectedSocket() client: Socket,
  ) {
    const order = await this.ordersService.updateStatus(data.orderId, data.status);
    this.server.emit('order:status', order);
    return order;
  }

  @SubscribeMessage('order:update_item_status')
  async handleUpdateItemStatus(
    @MessageBody() data: { itemId: string; status: string },
    @ConnectedSocket() client: Socket,
  ) {
    const item = await this.ordersService.updateItemStatus(data.itemId, data.status);
    this.server.emit('order:item_done', item);
    return item;
  }
}
```

- [ ] **Step 7: Create OrdersController**

```typescript
// backend/src/orders/orders.controller.ts
import {
  Controller, Get, Patch, Param, Body, UseGuards,
} from '@nestjs/common';
import { OrdersService } from './orders.service';
import { UpdateOrderStatusDto, UpdateOrderItemStatusDto } from './dto/update-order-status.dto';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';
import { OrdersGateway } from './orders.gateway';

@Controller('orders')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class OrdersController {
  constructor(
    private ordersService: OrdersService,
    private ordersGateway: OrdersGateway,
  ) {}

  @Get('active')
  @RequirePermissions('order:read')
  getActive() {
    return this.ordersService.getActiveOrders();
  }

  @Get(':id')
  @RequirePermissions('order:read')
  findOne(@Param('id') id: string) {
    return this.ordersService.findOne(id);
  }

  @Patch(':id/status')
  @RequirePermissions('order:update_status')
  async updateStatus(
    @Param('id') id: string,
    @Body() dto: UpdateOrderStatusDto,
  ) {
    const order = await this.ordersService.updateStatus(id, dto.status);
    this.ordersGateway.server.emit('order:status', order);
    return order;
  }

  @Patch('items/:itemId/status')
  @RequirePermissions('order:update_status')
  async updateItemStatus(
    @Param('itemId') itemId: string,
    @Body() dto: UpdateOrderItemStatusDto,
  ) {
    const item = await this.ordersService.updateItemStatus(itemId, dto.status);
    this.ordersGateway.server.emit('order:item_done', item);
    return item;
  }
}
```

- [ ] **Step 8: Create OrdersModule and register in AppModule**

```typescript
// backend/src/orders/orders.module.ts
import { Module } from '@nestjs/common';
import { OrdersController } from './orders.controller';
import { OrdersService } from './orders.service';
import { OrdersGateway } from './orders.gateway';

@Module({
  controllers: [OrdersController],
  providers: [OrdersService, OrdersGateway],
  exports: [OrdersService, OrdersGateway],
})
export class OrdersModule {}
```

Add `OrdersModule` to `AppModule` imports.

- [ ] **Step 9: Wire order creation into PosService**

Update `backend/src/pos/pos.service.ts` — inject `OrdersService` and `OrdersGateway`, then after transaction creation:

```typescript
// Add to PosService constructor:
import { OrdersService } from '../orders/orders.service';
import { OrdersGateway } from '../orders/orders.gateway';

constructor(
  private prisma: PrismaService,
  private menuService: MenuService,
  private promosService: PromosService,
  private ordersService: OrdersService,
  private ordersGateway: OrdersGateway,
) {}

// Add after transaction creation in create() method, before return:
const order = await this.ordersService.createFromTransaction(
  transaction.id,
  dto.items,
);

if (order) {
  this.ordersGateway.emitNewOrder(order);
}
```

Update `backend/src/pos/pos.module.ts` to import `OrdersModule`.

- [ ] **Step 10: Commit**

```bash
git add src/orders/ src/pos/ src/app.module.ts
git commit -m "feat: add orders module with WebSocket gateway for kitchen display"
```

---

### Task 12: Dashboard / Analytics Module

**Files:**
- Create: `backend/src/dashboard/dashboard.module.ts`
- Create: `backend/src/dashboard/dashboard.controller.ts`
- Create: `backend/src/dashboard/dashboard.service.ts`
- Create: `backend/src/dashboard/dashboard.service.spec.ts`

- [ ] **Step 1: Write failing test for DashboardService**

```typescript
// backend/src/dashboard/dashboard.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { DashboardService } from './dashboard.service';
import { PrismaService } from '../prisma/prisma.service';
import { MenuService } from '../menu/menu.service';

describe('DashboardService', () => {
  let service: DashboardService;

  const mockPrisma = {
    transaction: { aggregate: jest.fn(), findMany: jest.fn(), count: jest.fn() },
    transactionItem: { groupBy: jest.fn(), findMany: jest.fn() },
    transactionPromo: { aggregate: jest.fn() },
    menuItem: { findMany: jest.fn() },
  };

  const mockMenuService = {
    calculateHpp: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        DashboardService,
        { provide: PrismaService, useValue: mockPrisma },
        { provide: MenuService, useValue: mockMenuService },
      ],
    }).compile();

    service = module.get<DashboardService>(DashboardService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('getSummary', () => {
    it('should return today summary with revenue, hpp, profit', async () => {
      mockPrisma.transaction.aggregate.mockResolvedValue({
        _sum: { totalAmount: 500000, totalHpp: 200000, totalProfit: 280000, promoDiscount: 20000 },
      });
      mockPrisma.transaction.count.mockResolvedValue(15);

      const result = await service.getSummary(new Date(), new Date());

      expect(result.totalRevenue).toBe(500000);
      expect(result.totalHpp).toBe(200000);
      expect(result.totalProfit).toBe(280000);
      expect(result.totalPromoDiscount).toBe(20000);
      expect(result.transactionCount).toBe(15);
      expect(result.profitMargin).toBeCloseTo(56, 0);
    });
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/dashboard/dashboard.service.spec.ts --verbose
```

Expected: FAIL

- [ ] **Step 3: Implement DashboardService**

```typescript
// backend/src/dashboard/dashboard.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { MenuService } from '../menu/menu.service';

@Injectable()
export class DashboardService {
  constructor(
    private prisma: PrismaService,
    private menuService: MenuService,
  ) {}

  async getSummary(startDate: Date, endDate: Date) {
    const [aggregation, transactionCount] = await Promise.all([
      this.prisma.transaction.aggregate({
        where: { createdAt: { gte: startDate, lte: endDate } },
        _sum: {
          totalAmount: true,
          totalHpp: true,
          totalProfit: true,
          promoDiscount: true,
        },
      }),
      this.prisma.transaction.count({
        where: { createdAt: { gte: startDate, lte: endDate } },
      }),
    ]);

    const totalRevenue = aggregation._sum.totalAmount ?? 0;
    const totalHpp = aggregation._sum.totalHpp ?? 0;
    const totalProfit = aggregation._sum.totalProfit ?? 0;
    const totalPromoDiscount = aggregation._sum.promoDiscount ?? 0;

    return {
      totalRevenue,
      totalHpp,
      totalProfit,
      totalPromoDiscount,
      transactionCount,
      profitMargin: totalRevenue > 0 ? (totalProfit / totalRevenue) * 100 : 0,
    };
  }

  async getTopMenuByProfit(startDate: Date, endDate: Date, limit: number = 10) {
    const items = await this.prisma.transactionItem.findMany({
      where: {
        itemType: 'MENU',
        transaction: { createdAt: { gte: startDate, lte: endDate } },
      },
      select: {
        itemId: true,
        qty: true,
        subtotal: true,
        subHpp: true,
      },
    });

    const grouped = new Map<string, { totalQty: number; totalRevenue: number; totalHpp: number }>();
    for (const item of items) {
      const existing = grouped.get(item.itemId) ?? { totalQty: 0, totalRevenue: 0, totalHpp: 0 };
      existing.totalQty += item.qty;
      existing.totalRevenue += item.subtotal;
      existing.totalHpp += item.subHpp;
      grouped.set(item.itemId, existing);
    }

    const menuIds = [...grouped.keys()];
    const menus = await this.prisma.menuItem.findMany({
      where: { id: { in: menuIds } },
      select: { id: true, name: true, category: true, sellingPrice: true },
    });

    const menuMap = new Map(menus.map((m) => [m.id, m]));

    const result = [...grouped.entries()]
      .map(([id, data]) => ({
        menuItem: menuMap.get(id),
        ...data,
        totalProfit: data.totalRevenue - data.totalHpp,
        margin: data.totalRevenue > 0
          ? ((data.totalRevenue - data.totalHpp) / data.totalRevenue) * 100
          : 0,
      }))
      .sort((a, b) => b.totalProfit - a.totalProfit)
      .slice(0, limit);

    return result;
  }

  async getTopMenuByQty(startDate: Date, endDate: Date, limit: number = 10) {
    const items = await this.prisma.transactionItem.findMany({
      where: {
        itemType: 'MENU',
        transaction: { createdAt: { gte: startDate, lte: endDate } },
      },
      select: { itemId: true, qty: true },
    });

    const grouped = new Map<string, number>();
    for (const item of items) {
      grouped.set(item.itemId, (grouped.get(item.itemId) ?? 0) + item.qty);
    }

    const menuIds = [...grouped.keys()];
    const menus = await this.prisma.menuItem.findMany({
      where: { id: { in: menuIds } },
      select: { id: true, name: true, category: true },
    });

    const menuMap = new Map(menus.map((m) => [m.id, m]));

    return [...grouped.entries()]
      .map(([id, qty]) => ({ menuItem: menuMap.get(id), totalQty: qty }))
      .sort((a, b) => b.totalQty - a.totalQty)
      .slice(0, limit);
  }

  async getProfitByCategory(startDate: Date, endDate: Date) {
    const items = await this.prisma.transactionItem.findMany({
      where: {
        transaction: { createdAt: { gte: startDate, lte: endDate } },
      },
      select: { itemType: true, subtotal: true, subHpp: true },
    });

    const categories = new Map<string, { revenue: number; hpp: number }>();
    for (const item of items) {
      const key = item.itemType;
      const existing = categories.get(key) ?? { revenue: 0, hpp: 0 };
      existing.revenue += item.subtotal;
      existing.hpp += item.subHpp;
      categories.set(key, existing);
    }

    return [...categories.entries()].map(([category, data]) => ({
      category,
      revenue: data.revenue,
      hpp: data.hpp,
      profit: data.revenue - data.hpp,
    }));
  }

  async getSalesTrend(startDate: Date, endDate: Date) {
    const transactions = await this.prisma.transaction.findMany({
      where: { createdAt: { gte: startDate, lte: endDate } },
      select: {
        totalAmount: true,
        totalHpp: true,
        totalProfit: true,
        createdAt: true,
      },
      orderBy: { createdAt: 'asc' },
    });

    const daily = new Map<string, { revenue: number; hpp: number; profit: number; count: number }>();
    for (const t of transactions) {
      const date = t.createdAt.toISOString().slice(0, 10);
      const existing = daily.get(date) ?? { revenue: 0, hpp: 0, profit: 0, count: 0 };
      existing.revenue += t.totalAmount;
      existing.hpp += t.totalHpp;
      existing.profit += t.totalProfit;
      existing.count += 1;
      daily.set(date, existing);
    }

    return [...daily.entries()].map(([date, data]) => ({ date, ...data }));
  }
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/dashboard/dashboard.service.spec.ts --verbose
```

Expected: PASS

- [ ] **Step 5: Create DashboardController**

```typescript
// backend/src/dashboard/dashboard.controller.ts
import { Controller, Get, Query, UseGuards } from '@nestjs/common';
import { DashboardService } from './dashboard.service';
import { JwtAuthGuard } from '../common/guards/jwt-auth.guard';
import { PermissionsGuard } from '../common/guards/permissions.guard';
import { RequirePermissions } from '../common/decorators/permissions.decorator';

@Controller('dashboard')
@UseGuards(JwtAuthGuard, PermissionsGuard)
@RequirePermissions('dashboard:view')
export class DashboardController {
  constructor(private dashboardService: DashboardService) {}

  private parseDateRange(startDate?: string, endDate?: string) {
    const start = startDate ? new Date(startDate) : new Date(new Date().toISOString().slice(0, 10));
    const end = endDate ? new Date(endDate) : new Date();
    end.setHours(23, 59, 59, 999);
    return { start, end };
  }

  @Get('summary')
  getSummary(
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
  ) {
    const { start, end } = this.parseDateRange(startDate, endDate);
    return this.dashboardService.getSummary(start, end);
  }

  @Get('top-profit')
  getTopProfit(
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
    @Query('limit') limit?: number,
  ) {
    const { start, end } = this.parseDateRange(startDate, endDate);
    return this.dashboardService.getTopMenuByProfit(start, end, limit);
  }

  @Get('top-qty')
  getTopQty(
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
    @Query('limit') limit?: number,
  ) {
    const { start, end } = this.parseDateRange(startDate, endDate);
    return this.dashboardService.getTopMenuByQty(start, end, limit);
  }

  @Get('profit-by-category')
  getProfitByCategory(
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
  ) {
    const { start, end } = this.parseDateRange(startDate, endDate);
    return this.dashboardService.getProfitByCategory(start, end);
  }

  @Get('sales-trend')
  getSalesTrend(
    @Query('startDate') startDate?: string,
    @Query('endDate') endDate?: string,
  ) {
    const { start, end } = this.parseDateRange(startDate, endDate);
    return this.dashboardService.getSalesTrend(start, end);
  }
}
```

- [ ] **Step 6: Create DashboardModule and register in AppModule**

```typescript
// backend/src/dashboard/dashboard.module.ts
import { Module } from '@nestjs/common';
import { DashboardController } from './dashboard.controller';
import { DashboardService } from './dashboard.service';
import { MenuModule } from '../menu/menu.module';

@Module({
  imports: [MenuModule],
  controllers: [DashboardController],
  providers: [DashboardService],
})
export class DashboardModule {}
```

Add `DashboardModule` to `AppModule` imports.

- [ ] **Step 7: Commit**

```bash
git add src/dashboard/ src/app.module.ts
git commit -m "feat: add dashboard module with profit analytics and sales trend"
```

---

### Task 13: Seed Data

**Files:**
- Create: `backend/prisma/seed.ts`
- Modify: `backend/package.json`

- [ ] **Step 1: Create seed script**

```typescript
// backend/prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import * as bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // Roles
  const ownerRole = await prisma.role.create({ data: { name: 'OWNER', description: 'Restaurant owner' } });
  const kasirRole = await prisma.role.create({ data: { name: 'KASIR', description: 'Cashier' } });
  const kitchenRole = await prisma.role.create({ data: { name: 'KITCHEN', description: 'Kitchen staff' } });
  const inventoryRole = await prisma.role.create({ data: { name: 'INVENTORY', description: 'Inventory staff' } });
  const financeRole = await prisma.role.create({ data: { name: 'FINANCE', description: 'Finance staff' } });
  const marketingRole = await prisma.role.create({ data: { name: 'MARKETING', description: 'Marketing staff' } });

  // Permissions
  const permissionNames = [
    { name: 'menu:create', module: 'menu' },
    { name: 'menu:read', module: 'menu' },
    { name: 'menu:update', module: 'menu' },
    { name: 'menu:delete', module: 'menu' },
    { name: 'ingredient:manage', module: 'ingredients' },
    { name: 'ingredient:read', module: 'ingredients' },
    { name: 'transaction:create', module: 'pos' },
    { name: 'transaction:read', module: 'pos' },
    { name: 'order:read', module: 'orders' },
    { name: 'order:update_status', module: 'orders' },
    { name: 'dashboard:view', module: 'dashboard' },
    { name: 'user:manage', module: 'users' },
    { name: 'promo:manage', module: 'promos' },
    { name: 'promo:apply', module: 'promos' },
  ];

  const permissions: Record<string, string> = {};
  for (const p of permissionNames) {
    const created = await prisma.permission.create({ data: { ...p, description: p.name } });
    permissions[p.name] = created.id;
  }

  // Role-Permission mapping
  const rolePermMap: Record<string, string[]> = {
    [ownerRole.id]: Object.values(permissions),
    [kasirRole.id]: [
      permissions['menu:read'],
      permissions['transaction:create'],
      permissions['transaction:read'],
      permissions['order:read'],
      permissions['promo:apply'],
    ],
    [kitchenRole.id]: [
      permissions['menu:create'],
      permissions['menu:read'],
      permissions['menu:update'],
      permissions['menu:delete'],
      permissions['ingredient:manage'],
      permissions['ingredient:read'],
      permissions['order:read'],
      permissions['order:update_status'],
    ],
  };

  for (const [roleId, permIds] of Object.entries(rolePermMap)) {
    for (const permissionId of permIds) {
      await prisma.rolePermission.create({ data: { roleId, permissionId } });
    }
  }

  // Users
  const hashedPassword = await bcrypt.hash('password123', 10);
  await prisma.user.create({ data: { name: 'Owner Demo', email: 'owner@pos.com', password: hashedPassword, roleId: ownerRole.id } });
  await prisma.user.create({ data: { name: 'Kasir Demo', email: 'kasir@pos.com', password: hashedPassword, roleId: kasirRole.id } });
  await prisma.user.create({ data: { name: 'Kitchen Demo', email: 'kitchen@pos.com', password: hashedPassword, roleId: kitchenRole.id } });

  // Ingredients
  const nasi = await prisma.ingredient.create({ data: { name: 'Nasi', unit: 'kg', pricePerUnit: 12000, stockQty: 50, minStock: 10 } });
  const telur = await prisma.ingredient.create({ data: { name: 'Telur', unit: 'butir', pricePerUnit: 2500, stockQty: 100, minStock: 20 } });
  const minyak = await prisma.ingredient.create({ data: { name: 'Minyak Goreng', unit: 'liter', pricePerUnit: 18000, stockQty: 20, minStock: 5 } });
  const bumbuNasgor = await prisma.ingredient.create({ data: { name: 'Bumbu Nasi Goreng', unit: 'kg', pricePerUnit: 40000, stockQty: 10, minStock: 2 } });
  const ayam = await prisma.ingredient.create({ data: { name: 'Ayam', unit: 'kg', pricePerUnit: 35000, stockQty: 30, minStock: 5 } });
  const kopi = await prisma.ingredient.create({ data: { name: 'Kopi Bubuk', unit: 'kg', pricePerUnit: 80000, stockQty: 5, minStock: 1 } });
  const susu = await prisma.ingredient.create({ data: { name: 'Susu', unit: 'liter', pricePerUnit: 15000, stockQty: 10, minStock: 3 } });
  const gula = await prisma.ingredient.create({ data: { name: 'Gula', unit: 'kg', pricePerUnit: 14000, stockQty: 10, minStock: 2 } });
  const packaging = await prisma.ingredient.create({ data: { name: 'Packaging Box', unit: 'pcs', pricePerUnit: 1000, stockQty: 200, minStock: 50 } });

  // Menu Items with Recipes
  const nasiGoreng = await prisma.menuItem.create({
    data: {
      name: 'Nasi Goreng Spesial',
      category: 'FOOD',
      sellingPrice: 25000,
      recipes: {
        create: [
          { ingredientId: nasi.id, qtyNeeded: 0.2, unit: 'kg' },
          { ingredientId: telur.id, qtyNeeded: 1, unit: 'butir' },
          { ingredientId: minyak.id, qtyNeeded: 0.03, unit: 'liter' },
          { ingredientId: bumbuNasgor.id, qtyNeeded: 0.05, unit: 'kg' },
          { ingredientId: packaging.id, qtyNeeded: 1, unit: 'pcs' },
        ],
      },
    },
  });

  const ayamGoreng = await prisma.menuItem.create({
    data: {
      name: 'Ayam Goreng',
      category: 'FOOD',
      sellingPrice: 20000,
      recipes: {
        create: [
          { ingredientId: ayam.id, qtyNeeded: 0.25, unit: 'kg' },
          { ingredientId: minyak.id, qtyNeeded: 0.05, unit: 'liter' },
          { ingredientId: packaging.id, qtyNeeded: 1, unit: 'pcs' },
        ],
      },
    },
  });

  const kopiSusu = await prisma.menuItem.create({
    data: {
      name: 'Kopi Susu',
      category: 'BEVERAGE',
      sellingPrice: 18000,
      recipes: {
        create: [
          { ingredientId: kopi.id, qtyNeeded: 0.02, unit: 'kg' },
          { ingredientId: susu.id, qtyNeeded: 0.1, unit: 'liter' },
          { ingredientId: gula.id, qtyNeeded: 0.02, unit: 'kg' },
        ],
      },
    },
  });

  // Retail Items
  await prisma.retailItem.create({
    data: { name: 'Saus Sambal Botol', category: 'Condiment', buyPrice: 15000, sellingPrice: 25000, stockQty: 20, minStock: 5 },
  });
  await prisma.retailItem.create({
    data: { name: 'Kopi Bubuk 250gr', category: 'Merchandise', buyPrice: 25000, sellingPrice: 45000, stockQty: 15, minStock: 5 },
  });

  // Promo
  await prisma.promo.create({
    data: {
      name: 'Diskon Nasi Goreng 20%',
      type: 'PERCENT',
      value: 20,
      startDate: new Date('2026-01-01'),
      endDate: new Date('2026-12-31'),
      items: {
        create: [{ itemType: 'MENU', itemId: nasiGoreng.id, role: 'TARGET' }],
      },
    },
  });

  console.log('Seed completed successfully!');
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

- [ ] **Step 2: Add seed config to package.json**

Add to `backend/package.json`:

```json
"prisma": {
  "seed": "ts-node prisma/seed.ts"
}
```

- [ ] **Step 3: Install ts-node if needed and run seed**

```bash
cd ~/Downloads/PROJECT/backend
npm install -D ts-node
npx prisma db seed
```

Expected: "Seed completed successfully!"

- [ ] **Step 4: Verify seed data**

```bash
npx prisma studio
```

Expected: Tables populated with demo data — 3 users, 3 menu items with recipes, 2 retail items, 1 promo

- [ ] **Step 5: Commit**

```bash
git add prisma/seed.ts package.json
git commit -m "feat: add seed data with demo users, menu, ingredients, and promo"
```

---

### Task 14: Run All Tests & Final Verification

- [ ] **Step 1: Run all tests**

```bash
cd ~/Downloads/PROJECT/backend
npx jest --verbose
```

Expected: All tests pass (auth, users, ingredients, menu, retail, promos, pos, orders, dashboard)

- [ ] **Step 2: Start the app and verify**

```bash
npm run start:dev
```

Expected: Application starts without errors on port 3000

- [ ] **Step 3: Quick smoke test with curl**

```bash
# Login
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"owner@pos.com","password":"password123"}'

# Use the token from response to test authenticated endpoints
TOKEN="<paste-token-here>"

# Get menu with HPP
curl http://localhost:3000/menu -H "Authorization: Bearer $TOKEN"

# Get dashboard summary
curl http://localhost:3000/dashboard/summary -H "Authorization: Bearer $TOKEN"
```

- [ ] **Step 4: Final commit**

```bash
git add -A
git commit -m "chore: final backend Phase 1 verification"
```
