# POS Phase 2B — Finance & Reporting Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add finance module to POS — daily cash closing, optional operational costs, monthly P&L reports with PDF/Excel export.

**Architecture:** Extend existing NestJS backend with 3 new modules (closings, operational-costs, reports). Install pdfkit + exceljs for export. Add 3 new web dashboard pages.

**Tech Stack:** NestJS, TypeScript, PostgreSQL, Prisma, pdfkit, exceljs, Next.js, shadcn/ui

**Spec:** `docs/superpowers/specs/2026-05-01-pos-phase2-inventory-finance-design.md`

**Prerequisite:** Phase 2A (Inventory) must be completed first.

---

## File Structure

### Backend — New Files

```
backend/src/
├── closings/
│   ├── closings.module.ts
│   ├── closings.controller.ts
│   ├── closings.service.ts
│   ├── closings.service.spec.ts
│   └── dto/
│       └── create-closing.dto.ts
├── operational-costs/
│   ├── operational-costs.module.ts
│   ├── operational-costs.controller.ts
│   ├── operational-costs.service.ts
│   └── dto/
│       ├── create-operational-cost.dto.ts
│       └── update-operational-cost.dto.ts
├── reports/
│   ├── reports.module.ts
│   ├── reports.controller.ts
│   ├── reports.service.ts
│   ├── reports.service.spec.ts
│   └── export/
│       ├── pdf-export.service.ts
│       └── excel-export.service.ts
```

### Backend — Modified Files

```
backend/prisma/schema.prisma    — add DailyClosing, OperationalCost, MonthlyReport + enums
backend/prisma/seed.ts          — add Finance permissions
backend/src/app.module.ts       — register 3 new modules
```

### Web — New Files

```
web/src/app/(dashboard)/closings/page.tsx
web/src/app/(dashboard)/operational-costs/page.tsx
web/src/app/(dashboard)/reports/page.tsx
web/src/services/closing.service.ts
web/src/services/operational-cost.service.ts
web/src/services/report.service.ts
web/src/types/closing.ts
web/src/types/operational-cost.ts
web/src/types/report.ts
```

---

### Task 1: Prisma Schema — Add Finance Models

**Files:** Modify: `backend/prisma/schema.prisma`

- [ ] **Step 1: Add DailyClosing, OperationalCost, MonthlyReport models + enums**

Add to end of `backend/prisma/schema.prisma`:

```prisma
model DailyClosing {
  id          String        @id @default(uuid())
  cashierId   String
  date        DateTime
  systemTotal Float
  actualCash  Float
  difference  Float
  notes       String?
  status      ClosingStatus @default(OPEN)
  createdAt   DateTime      @default(now())
  cashier     User          @relation(fields: [cashierId], references: [id])

  @@unique([cashierId, date])
  @@map("daily_closings")
}

enum ClosingStatus {
  OPEN
  CLOSED
}

model OperationalCost {
  id          String       @id @default(uuid())
  name        String
  category    OpexCategory
  amount      Float
  periodMonth Int
  periodYear  Int
  notes       String?
  createdById String
  createdAt   DateTime     @default(now())
  createdBy   User         @relation(fields: [createdById], references: [id])

  @@map("operational_costs")
}

enum OpexCategory {
  SALARY
  RENT
  UTILITY
  OTHER
}

model MonthlyReport {
  id                   String   @id @default(uuid())
  month                Int
  year                 Int
  totalRevenue         Float
  totalHpp             Float
  grossProfit          Float
  totalOperationalCost Float    @default(0)
  totalPromoDiscount   Float    @default(0)
  netProfit            Float
  generatedAt          DateTime @default(now())

  @@unique([month, year])
  @@map("monthly_reports")
}
```

- [ ] **Step 2: Add relations to User model**

```prisma
  dailyClosings    DailyClosing[]
  operationalCosts OperationalCost[]
```

- [ ] **Step 3: Validate + migrate**

```bash
npx prisma validate
npx prisma migrate dev --name add_finance_models
npx prisma generate
```

- [ ] **Step 4: Commit**

```bash
git add prisma/
git commit -m "feat: add finance Prisma models (closing, opex, monthly report)"
```

---

### Task 2: Install Export Dependencies

- [ ] **Step 1: Install pdfkit + exceljs**

```bash
cd ~/Downloads/PROJECT/pos/backend
npm install pdfkit exceljs
npm install -D @types/pdfkit
```

- [ ] **Step 2: Commit**

```bash
git add package.json package-lock.json
git commit -m "chore: install pdfkit and exceljs for report export"
```

---

### Task 3: Daily Closings Module

**Files:** Create: `backend/src/closings/`

- [ ] **Step 1: Create DTO**

Create `backend/src/closings/dto/create-closing.dto.ts`:
```typescript
import { IsNumber, IsOptional, IsString } from 'class-validator';

export class CreateClosingDto {
  @IsNumber()
  actualCash: number;

  @IsOptional()
  @IsString()
  notes?: string;
}
```

- [ ] **Step 2: Write ClosingsService test**

Create `backend/src/closings/closings.service.spec.ts`:
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { ClosingsService } from './closings.service';
import { PrismaService } from '../prisma/prisma.service';

describe('ClosingsService', () => {
  let service: ClosingsService;

  const mockPrisma = {
    transaction: { aggregate: jest.fn() },
    dailyClosing: { create: jest.fn(), findMany: jest.fn(), findUnique: jest.fn(), count: jest.fn() },
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ClosingsService,
        { provide: PrismaService, useValue: mockPrisma },
      ],
    }).compile();

    service = module.get<ClosingsService>(ClosingsService);
  });

  afterEach(() => jest.clearAllMocks());

  describe('create', () => {
    it('should calculate system total and difference', async () => {
      mockPrisma.transaction.aggregate.mockResolvedValue({
        _sum: { totalAmount: 500000 },
      });
      mockPrisma.dailyClosing.create.mockResolvedValue({
        id: 'c1', systemTotal: 500000, actualCash: 490000, difference: -10000, status: 'CLOSED',
      });

      const result = await service.create('cashier1', { actualCash: 490000 });
      expect(result.systemTotal).toBe(500000);
      expect(result.difference).toBe(-10000);
    });
  });
});
```

- [ ] **Step 3: Implement ClosingsService**

Create `backend/src/closings/closings.service.ts`:
```typescript
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateClosingDto } from './dto/create-closing.dto';

@Injectable()
export class ClosingsService {
  constructor(private prisma: PrismaService) {}

  async findAll(filters: { cashierId?: string; startDate?: string; endDate?: string; page?: number; limit?: number }) {
    const page = filters.page ?? 1;
    const limit = filters.limit ?? 20;
    const skip = (page - 1) * limit;

    const where: Record<string, unknown> = {};
    if (filters.cashierId) where.cashierId = filters.cashierId;
    if (filters.startDate || filters.endDate) {
      where.date = {};
      if (filters.startDate) (where.date as Record<string, unknown>).gte = new Date(filters.startDate);
      if (filters.endDate) (where.date as Record<string, unknown>).lte = new Date(filters.endDate);
    }

    const [data, total] = await Promise.all([
      this.prisma.dailyClosing.findMany({
        where: where as any,
        skip, take: limit,
        include: { cashier: { select: { id: true, name: true } } },
        orderBy: { date: 'desc' },
      }),
      this.prisma.dailyClosing.count({ where: where as any }),
    ]);

    return { data, total, page, limit };
  }

  async findOne(id: string) {
    const closing = await this.prisma.dailyClosing.findUnique({
      where: { id },
      include: { cashier: { select: { id: true, name: true } } },
    });
    if (!closing) throw new NotFoundException('Closing not found');
    return closing;
  }

  async create(cashierId: string, dto: CreateClosingDto) {
    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const todayEnd = new Date(today);
    todayEnd.setHours(23, 59, 59, 999);

    // Calculate system total from CASH transactions today by this cashier
    const aggregation = await this.prisma.transaction.aggregate({
      where: {
        cashierId,
        paymentMethod: 'CASH' as any,
        createdAt: { gte: today, lte: todayEnd },
      },
      _sum: { totalAmount: true },
    });

    const systemTotal = aggregation._sum.totalAmount ?? 0;
    const difference = dto.actualCash - systemTotal;

    return this.prisma.dailyClosing.create({
      data: {
        cashierId,
        date: today,
        systemTotal,
        actualCash: dto.actualCash,
        difference,
        notes: dto.notes,
        status: 'CLOSED' as any,
      },
      include: { cashier: { select: { id: true, name: true } } },
    });
  }
}
```

- [ ] **Step 4: Run test → verify passes**

```bash
npx jest src/closings/closings.service.spec.ts --verbose
```

- [ ] **Step 5: Create Controller, Module, register in AppModule**

Create controller (POST /closings, GET /closings, GET /closings/:id), module, register.

- [ ] **Step 6: Build + commit**

```bash
npx nest build
git add src/closings/ src/app.module.ts
git commit -m "feat: add daily closings module with auto system total calculation"
```

---

### Task 4: Operational Costs Module

**Files:** Create: `backend/src/operational-costs/` — standard CRUD, similar to suppliers

- [ ] **Step 1: Create DTOs** (create + update with name, category, amount, periodMonth, periodYear, notes)
- [ ] **Step 2: Create Service** (findAll with month/year filter, create, update, delete)
- [ ] **Step 3: Create Controller** (guarded with `opex:manage`)
- [ ] **Step 4: Create Module, register in AppModule**
- [ ] **Step 5: Build + commit**

```bash
npx nest build
git add src/operational-costs/ src/app.module.ts
git commit -m "feat: add operational-costs module with CRUD"
```

---

### Task 5: Reports Module + Export

**Files:** Create: `backend/src/reports/`

- [ ] **Step 1: Write ReportsService test**

Test: `getSummary` aggregates revenue/HPP/profit/opex/promo correctly.

- [ ] **Step 2: Implement ReportsService**

```typescript
// Key methods:
// getOrGenerateMonthly(year, month) — check if exists, if not aggregate + save
// getProfitLoss(startDate, endDate) — custom date range
// exportPdf(year, month) — returns Buffer
// exportExcel(year, month) — returns Buffer
```

- [ ] **Step 3: Implement PdfExportService**

Create `backend/src/reports/export/pdf-export.service.ts`:
Uses `pdfkit` to generate P&L PDF with header, date, revenue/HPP/profit table.

- [ ] **Step 4: Implement ExcelExportService**

Create `backend/src/reports/export/excel-export.service.ts`:
Uses `exceljs` to generate workbook with Summary sheet + Transaction detail sheet.

- [ ] **Step 5: Run test → verify passes**

- [ ] **Step 6: Create ReportsController**

Endpoints: GET /reports/monthly, GET /reports/monthly/:year/:month, GET /reports/monthly/:year/:month/export?format=pdf|excel, GET /reports/profit-loss

- [ ] **Step 7: Create Module, register in AppModule**

- [ ] **Step 8: Build + commit**

```bash
npx nest build
git add src/reports/ src/app.module.ts
git commit -m "feat: add reports module with monthly P&L + PDF/Excel export"
```

---

### Task 6: Update Seed Data + Permissions

- [ ] **Step 1: Add finance permissions to seed**

Add: `closing:create`, `closing:read`, `report:read`, `report:export`, `opex:manage`

Map to OWNER (all), KASIR (closing:create), FINANCE (closing:read, report:read, report:export, opex:manage).

- [ ] **Step 2: Reset + seed**

```bash
npx prisma migrate reset --force
npx prisma db seed
```

- [ ] **Step 3: Commit**

---

### Task 7: Web — Types + Services + Pages for Finance

- [ ] **Step 1: Create type files** (closing, operational-cost, report)
- [ ] **Step 2: Create service files** (closing, operational-cost, report — including export download)
- [ ] **Step 3: Update constants.ts** with finance permissions + sidebar items
- [ ] **Step 4: Create /closings page** — list + create modal
- [ ] **Step 5: Create /operational-costs page** — CRUD table with month filter
- [ ] **Step 6: Create /reports page** — monthly list, P&L detail, export buttons
- [ ] **Step 7: Build + commit**

```bash
npx next build
git add src/
git commit -m "feat: add finance web pages (closings, opex, reports)"
```

---

### Task 8: Run All Tests & Final Verification

- [ ] **Step 1: Run all backend tests**

```bash
npx jest --verbose
```

- [ ] **Step 2: Build backend + web**

```bash
npx nest build
cd ../web && npx next build
```

- [ ] **Step 3: Final commit**

```bash
git add -A
git commit -m "chore: Phase 2B finance & reporting complete"
```
