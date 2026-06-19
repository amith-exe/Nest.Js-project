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
# Setting Up DTOs in NestJS Authentication

## Login DTO - Book My Venue Backend

---

# What is a DTO?

DTO stands for:

```text
Data Transfer Object
```

A DTO is a TypeScript class that defines:

1. The shape of incoming data
2. Validation rules
3. Swagger documentation

Think of it as a contract.

```text
Client
   ↓
JSON Request
   ↓
DTO
   ↓
Validation
   ↓
Controller
   ↓
Service
```

---

# Why Do We Need DTOs?

Without DTOs:

```json
{
  "email": 123,
  "password": null
}
```

could reach your service and cause errors.

DTOs ensure:

```json
{
  "email": "amith@gmail.com",
  "password": "Password@123"
}
```

is the only acceptable format.

---

# Install Required Packages

```bash
npm install class-validator class-transformer
```

---

# Why class-validator?

Provides decorators like:

```ts
@IsEmail()
@IsString()
@MinLength()
@Length()
@IsUUID()
```

These validate incoming data.

---

# Why class-transformer?

Converts:

```json
{
  "email": "amith@gmail.com",
  "password": "Password@123"
}
```

into:

```ts
new LoginDto()
```

Without it, Nest cannot properly transform request bodies into DTO instances.

---

# Project Structure

```text
src/
└── auth/
    ├── dto/
    │   ├── login.dto.ts
    │   └── register.dto.ts
    │
    ├── auth.controller.ts
    ├── auth.service.ts
    └── auth.module.ts
```

---

# Login Requirements

For login, we need:

```text
Email
Password
```

Request:

```json
{
  "email": "amith@gmail.com",
  "password": "Password@123"
}
```

---

# Creating Login DTO

## login.dto.ts

```ts
import { ApiProperty } from '@nestjs/swagger';
import {
  IsEmail,
  IsString,
} from 'class-validator';

export class LoginDto {
  @ApiProperty({
    example: 'amith@gmail.com',
    description: 'User email address',
  })
  @IsEmail()
  email!: string;

  @ApiProperty({
    example: 'Password@123',
    description: 'User password',
  })
  @IsString()
  password!: string;
}
```

---

# Understanding Each Line

## ApiProperty

```ts
@ApiProperty({
  example: 'amith@gmail.com',
})
```

Purpose:

```text
Swagger Documentation
```

Displays:

```text
email
string
Example:
amith@gmail.com
```

in:

```text
http://localhost:3000/api
```

---

# IsEmail

```ts
@IsEmail()
```

Checks:

```json
{
  "email": "amith@gmail.com"
}
```

✅ Valid

---

```json
{
  "email": "hello"
}
```

❌ Invalid

Response:

```json
{
  "statusCode": 400,
  "message": [
    "email must be an email"
  ]
}
```

---

# IsString

```ts
@IsString()
```

Checks:

```json
{
  "password": "Password123"
}
```

✅ Valid

---

```json
{
  "password": 123
}
```

❌ Invalid

Response:

```json
{
  "statusCode": 400,
  "message": [
    "password must be a string"
  ]
}
```

---

# Why use `!`?

```ts
email!: string;
password!: string;
```

The `!` is called the:

```text
Definite Assignment Assertion Operator
```

It means:

```text
I promise this property
will be assigned later.
```

---

# Why is this needed?

TypeScript strict mode expects:

```ts
class LoginDto {
  email: string;
}
```

to be initialized:

```ts
email = '';
```

or:

```ts
constructor() {
  this.email = '';
}
```

Otherwise:

```text
Property 'email'
has no initializer
and is not definitely assigned.
```

---

# Why does `!` work in DTOs?

Because NestJS automatically assigns values.

Request:

```json
{
  "email": "amith@gmail.com",
  "password": "Password123"
}
```

Flow:

```text
Request Body
      ↓
ValidationPipe
      ↓
class-transformer
      ↓
new LoginDto()
      ↓
email assigned
password assigned
```

So:

```ts
email!: string;
```

is perfectly safe.

---

# Why not use `?`

Example:

```ts
email?: string;
```

Type becomes:

```ts
string | undefined
```

Meaning:

```text
email may be undefined
```

Then everywhere:

```ts
dto.email
```

TypeScript assumes:

```ts
string | undefined
```

which is unnecessary for login.

---

# Why not use default values?

Example:

```ts
email: string = '';
password: string = '';
```

Works, but:

```text
Unnecessary
More code
Not common in NestJS DTOs
```

Most NestJS projects use:

```ts
email!: string;
password!: string;
```

---

# Login Request Flow

```text
Client
   ↓
POST /auth/login
   ↓
Request Body
   ↓
ValidationPipe
   ↓
LoginDto
   ↓
class-validator
   ↓
AuthController
   ↓
AuthService
   ↓
PrismaService
   ↓
Database
```

---

# Invalid Example

Request:

```json
{
  "email": "hello",
  "password": 123
}
```

Validation:

```text
❌ email must be an email
❌ password must be a string
```

Controller never executes.

---

# Valid Example

Request:

```json
{
  "email": "amith@gmail.com",
  "password": "Password@123"
}
```

Validation:

```text
✓ email is valid
✓ password is string
```

Controller executes.

---

# Swagger Output

Because of:

```ts
@ApiProperty()
```

Swagger automatically generates:

```text
Authentication
└── POST /auth/login

Request Body

email
string
Example:
amith@gmail.com

password
string
Example:
Password@123
```

---

# Using Login DTO in Controller

```ts
@Post('login')
login(
  @Body() dto: LoginDto,
) {
  return this.authService.login(dto);
}
```

Flow:

```text
JSON Request
      ↓
LoginDto
      ↓
Validation
      ↓
Controller
      ↓
Service
```

---

# Why DTOs Are Important

DTOs become the single source of truth for:

```text
Request Structure
      +
Validation Rules
      +
Swagger Documentation
```

Everything about the request lives in one place.

---

# Final Login DTO

```ts
import { ApiProperty } from '@nestjs/swagger';
import {
  IsEmail,
  IsString,
} from 'class-validator';

export class LoginDto {
  @ApiProperty({
    example: 'amith@gmail.com',
    description: 'User email address',
  })
  @IsEmail()
  email!: string;

  @ApiProperty({
    example: 'Password@123',
    description: 'User password',
  })
  @IsString()
  password!: string;
}
```

---

# Next Steps

```text
✓ Install class-validator
✓ Install class-transformer
✓ Create dto folder
✓ Create LoginDto
⬜ Create RegisterDto
⬜ Enable ValidationPipe
⬜ Build Register Endpoint
⬜ Build Login Endpoint
⬜ JWT Authentication
⬜ Refresh Tokens
⬜ Email Verification
⬜ Password Reset
⬜ Google OAuth
```

# Async & Await in NestJS (Quick Notes)

## What is `async`?

`async` marks a function that performs operations that take time.

Examples:

* Database Queries
* Password Hashing (`bcrypt`)
* API Calls
* Email Sending
* File Operations

---

## What is `await`?

`await` pauses the current function until a Promise finishes.

```ts
const user = await this.prisma.user.findUnique();
```

Meaning:

> Wait for the database query to finish, then continue.

---

## Rule of Thumb

If you use:

```ts
await something();
```

the function must be:

```ts
async functionName() {}
```

Example:

```ts
async login() {
  const user = await this.prisma.user.findUnique();
  return user;
}
```

---

## Why Do We Need It?

Operations like database queries are not instant.

```text
Application
     ↓
Database Query
     ↓
Wait for Response
     ↓
Continue Execution
```

Node.js does not block the entire server while waiting.

---

## Common Examples in Authentication

### Register

```ts
async register() {
  await this.prisma.user.findUnique();
  await bcrypt.hash();
  await this.prisma.user.create();
}
```

### Login

```ts
async login() {
  await this.prisma.user.findUnique();
  await bcrypt.compare();
}
```

### Email Verification

```ts
async verifyEmail() {
  await this.emailService.sendOtp();
}
```

---

## Mental Model

```text
await
  ↓
"Pause this function only until the task finishes."
```

Not:

```text
❌ Pause the entire server
```

---

## Easy Way to Remember

Ask yourself:

> Does this operation talk to something outside my function?

Examples:

* Database ✅
* Network/API ✅
* Email Service ✅
* bcrypt Hashing ✅
* File System ✅

Usually:

```ts
async functionName() {
  await something();
}
```

---

## Summary

```text
async
  ↓
Function contains asynchronous operations

await
  ↓
Wait for a Promise to finish

Rule:
If you write await, the function must be async.
```

