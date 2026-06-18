# Initial NestJS + Prisma + PostgreSQL Setup Notes

## Book My Venue Backend Foundation

---

# Goal

Set up:

```text
NestJS
   ↓
Config Module
   ↓
Prisma
   ↓
PostgreSQL
```

Architecture:

```text
AuthService
      ↓
PrismaService
      ↓
PrismaClient
      ↓
PostgreSQL
```

This becomes the foundation for:

* Authentication
* Profiles
* Venues
* Bookings
* Notifications
* Analytics

---

# Step 1: Create NestJS Project

```bash
nest new book-my-venue-backend
```

Creates:

```text
src/
├── app.module.ts
├── app.controller.ts
└── app.service.ts
```

---

# Step 2: Install Prisma

### Install Prisma CLI

```bash
npm install prisma --save-dev
```

Purpose:

```text
Prisma CLI
     ↓
Initialize Prisma
Generate Prisma Client
Run Migrations
Manage Database
```

---

### Install Prisma Client

```bash
npm install @prisma/client
```

Purpose:

Provides:

```ts
PrismaClient
```

Which gives methods like:

```ts
prisma.user.create()
prisma.user.findUnique()
prisma.user.findMany()
prisma.user.update()
prisma.user.delete()
```

---

# Step 3: Initialize Prisma

```bash
npx prisma init
```

Creates:

```text
prisma/
├── schema.prisma
└── migrations/

.env
```

---

# Project Structure

```text
project/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── src/
│
└── .env
```

---

# Purpose of Each File

---

## prisma/schema.prisma

Database blueprint.

Contains:

* Models
* Relations
* Enums
* Database configuration

Example:

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
}
```

Think:

```text
schema.prisma
      ↓
Blueprint of database
```

---

## prisma/migrations/

Contains database history.

Example:

```text
migrations/
├── 20260617_init/
├── 20260618_add_role/
└── 20260619_add_profile/
```

Purpose:

```text
Tracks database changes
```

---

# Step 4: Configure Environment Variables

Create:

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/bookmyvenue
```

Purpose:

Never hardcode:

```ts
const url =
'postgresql://postgres:password...'
```

because:

```text
❌ Not secure
❌ Difficult to change
❌ Different for production
```

---

# Step 5: Install Config Module

```bash
npm install @nestjs/config
```

---

# Why ConfigModule?

NestJS cannot automatically read:

```env
DATABASE_URL=
JWT_SECRET=
PORT=
```

ConfigModule:

```text
Reads .env
      ↓
Creates ConfigService
      ↓
Makes variables available everywhere
```

---

# App Module Setup

## app.module.ts

```ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

---

# What Happens?

```text
Application Starts
        ↓
ConfigModule.forRoot()
        ↓
Reads .env
        ↓
Creates ConfigService
        ↓
Stores variables
        ↓
Makes ConfigService available globally
```

---

# Why isGlobal?

Without:

```ts
imports: [ConfigModule]
```

would be required in:

```text
AuthModule
PrismaModule
VenueModule
BookingModule
```

With:

```ts
isGlobal: true
```

ConfigService is available everywhere.

---

# ConfigService Mental Model

Think:

```text
.env
     ↓
DATABASE_URL=...
JWT_SECRET=...
PORT=3000

ConfigService
     ↓
Dictionary of settings
```

Example:

```ts
configService.get('DATABASE_URL')
```

returns:

```text
postgresql://postgres:...
```

---

# Step 6: Configure Prisma Schema

Example:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
}
```

---

# Step 7: Create User Model

Example:

```prisma
enum Role {
  USER
  OWNER
  ADMIN
}

model User {
  id                 Int       @id @default(autoincrement())
  email              String    @unique
  passwordHash       String
  role               Role      @default(USER)
  isEmailVerified    Boolean   @default(false)
  refreshTokenHash   String?

  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
}
```

---

# Generate Prisma Client

```bash
npx prisma generate
```

Purpose:

```text
schema.prisma
      ↓
Generate TypeScript Database Client
      ↓
PrismaClient
```

Provides:

```ts
prisma.user.create()
prisma.user.findUnique()
prisma.user.update()
```

---

# Create Migration

```bash
npx prisma migrate dev --name init
```

Purpose:

```text
schema.prisma
      ↓
SQL Migration Generated
      ↓
Applied to PostgreSQL
      ↓
Database Tables Created
      ↓
Prisma Client Regenerated
```

---

# Step 8: Create Prisma Module

```bash
nest g module prisma
nest g service prisma
```

Structure:

```text
src/
└── prisma/
    ├── prisma.module.ts
    └── prisma.service.ts
```

---

# Why PrismaModule?

Without it:

```ts
const prisma =
new PrismaClient();
```

inside every service.

Example:

```text
AuthService
VenueService
BookingService
ProfileService
```

Each service creates database access logic.

This becomes messy.

---

# Better Architecture

```text
AuthService
VenueService
BookingService
ProfileService
        ↓
   PrismaService
        ↓
   PrismaClient
        ↓
   PostgreSQL
```

Centralized database access.

---

# Prisma Service

## prisma.service.ts

```ts
@Injectable()
export class PrismaService
  extends PrismaClient {

  constructor() {
    super();
  }
}
```

---

# Why extend PrismaClient?

PrismaClient already knows:

```ts
user.findUnique()
user.create()
venue.findMany()
booking.create()
```

By extending it:

```ts
class PrismaService
  extends PrismaClient
```

PrismaService automatically gets all database methods.

---

# Understanding super()

```ts
super();
```

means:

```text
Run PrismaClient constructor
```

Flow:

```text
PrismaService
      ↓
super()
      ↓
PrismaClient constructor
      ↓
Prisma initialized
      ↓
Database ready
```

---

# Mental Model

```text
extends
      ↓
Inherit parent class

super()
      ↓
Run parent constructor
```

---

# Step 9: Prisma Module

## prisma.module.ts

```ts
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

---

# providers

```ts
providers: [PrismaService]
```

means:

```text
Create PrismaService
Store inside DI Container
```

---

# exports

```ts
exports: [PrismaService]
```

means:

```text
Other modules can use PrismaService
```

---

# Step 10: Import PrismaModule

Example:

```ts
@Module({
  imports: [PrismaModule],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}
```

---

# Dependency Injection Flow

```text
AuthService
      ↓
needs PrismaService
      ↓
AuthModule imports PrismaModule
      ↓
PrismaModule exports PrismaService
      ↓
Nest injects PrismaService
```

---

# Using PrismaService

```ts
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma:
      PrismaService,
  ) {}
}
```

Now:

```ts
this.prisma.user.findUnique()
this.prisma.user.create()
this.prisma.user.update()
```

are available.

---

# Final Architecture

```text
Controller
      ↓
Service
      ↓
PrismaService
      ↓
PrismaClient
      ↓
PostgreSQL
```

---

# Project Structure After Initial Setup

```text
src/
├── app.module.ts
│
├── prisma/
│   ├── prisma.module.ts
│   └── prisma.service.ts
│
├── auth/
│   ├── auth.module.ts
│   ├── auth.service.ts
│   └── auth.controller.ts
│
└── main.ts

prisma/
├── schema.prisma
└── migrations/

.env
```

---

# Setup Completion Checklist

```text
✓ PostgreSQL installed
✓ Database created
✓ Prisma installed
✓ Prisma initialized
✓ Environment variables configured
✓ ConfigModule configured
✓ User model created
✓ Prisma client generated
✓ Migration created
✓ PrismaModule created
✓ PrismaService created
✓ Database connection established
✓ PrismaService injected into modules
```

At this point, the backend foundation is ready and you can begin implementing:

```text
Authentication
      ↓
Register
      ↓
Login
      ↓
JWT
      ↓
Guards
      ↓
Role Authorization
      ↓
Profiles
      ↓
Venues
      ↓
Bookings
```

