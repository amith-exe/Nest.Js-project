# Connection with database


### here we are using psql

* here we are using prisma ORM(object relation mapping)

install it 
```npm install @prisma/client
npm install prisma --save-dev
```
* intializng prisma
use this code
```npx prisma init```
it creates 
```prisma/
└── schema.prisma

.env
```
in env we have to set username and password of db

```
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/databasename"
```
* then set up schema.prisma file

example
```// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

// Get a free hosted Postgres database in seconds: `npx create-db`

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
}

enum Role {
  USER
  VENUE_OWNER
  ADMIN
}

model User {
  id                String   @id @default(uuid())
  email             String   @unique
  passwordHash      String?
  googleId          String?  @unique
  role              Role     @default(USER)
  isEmailVerified   Boolean  @default(false)
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
}
```
then use this code to create the db tables

```
# Prisma Migrations in NestJS + PostgreSQL

## What is a Migration?

A migration is a **version-controlled change to your database schema**.

Think of it like Git commits, but for your database.

```text
Code Changes
      ↓
Migration
      ↓
Database Changes
```

Every time you add, remove, or modify a table, Prisma creates a migration file so that the database can be updated automatically.

---

# Understanding This Command

```bash
npx prisma migrate dev --name init
```

This command is one of the most important Prisma commands.

Let's break it down piece by piece.

---

# 1. `npx`

```bash
npx prisma
```

## What is it?

`npx` runs packages directly from your project's `node_modules`.

## Why do we use it?

Prisma is installed inside your project:

```text
node_modules/
└── prisma/
```

Instead of installing Prisma globally:

```bash
npm install -g prisma
```

we can simply do:

```bash
npx prisma
```

and npm automatically finds the local Prisma installation.

---

## Think of it as

```text
npx
 ↓
"Find Prisma inside node_modules and execute it."
```

---

# 2. `prisma`

```bash
npx prisma
```

## What is it?

Prisma is the CLI (Command Line Interface) for managing:

* Database schema
* Migrations
* Prisma Client generation
* Database introspection
* Prisma Studio

Think of it like:

```text
git     → version control
npm     → package manager
prisma  → database management tool
```

---

# 3. `migrate`

```bash
npx prisma migrate
```

## What is it?

This tells Prisma:

> "I changed my schema. Please update my database."

Prisma compares:

```text
schema.prisma
       ↓
Current Database
```

and determines what SQL needs to run.

---

## Example

Current database:

```text
bookmyvenue
      ↓
(no tables)
```

Schema:

```prisma
model User {
  id       Int
  email    String
  password String
}
```

Prisma notices:

```text
Database has no User table.
```

So it generates SQL:

```sql
CREATE TABLE "User" (
  "id" SERIAL PRIMARY KEY,
  "email" TEXT NOT NULL,
  "password" TEXT NOT NULL
);
```

---

# 4. `dev`

```bash
npx prisma migrate dev
```

## What is it?

Runs migrations in development mode.

It performs several tasks automatically.

---

## Step 1

Reads:

```text
schema.prisma
```

---

## Step 2

Generates SQL migrations.

---

## Step 3

Runs SQL on PostgreSQL.

---

## Step 4

Stores migration history.

---

## Step 5

Regenerates Prisma Client.

---

# Why do we use `dev`?

Because during development:

* Models change frequently
* Tables are modified often
* New relationships are added

`migrate dev` automates everything.

---

# 5. `--name init`

```bash
npx prisma migrate dev --name init
```

## What is it?

Gives a name to the migration.

Here:

```text
Migration Name = init
```

Prisma creates:

```text
prisma/
└── migrations/
    └── 20260617123000_init/
        └── migration.sql
```

---

## Why give migrations names?

Imagine months later.

This:

```text
20260617123000_init
```

is much better than:

```text
20260617123000
```

You immediately know:

```text
init
```

means:

> Initial database setup.

---

# Good Migration Names

## Initial database

```bash
npx prisma migrate dev --name init
```

---

## Add role column

```bash
npx prisma migrate dev --name add_role
```

---

## Add booking table

```bash
npx prisma migrate dev --name add_booking
```

---

## Add venue relationships

```bash
npx prisma migrate dev --name venue_relations
```

---

# What Actually Happens Internally?

Suppose:

## schema.prisma

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  createdAt DateTime @default(now())
}
```

You execute:

```bash
npx prisma migrate dev --name init
```

---

# Step 1

Prisma reads:

```text
schema.prisma
```

and builds an internal representation:

```text
User
 ├── id
 ├── email
 ├── password
 └── createdAt
```

---

# Step 2

Prisma generates SQL.

```sql
CREATE TABLE "User" (
  "id" SERIAL PRIMARY KEY,
  "email" TEXT NOT NULL UNIQUE,
  "password" TEXT NOT NULL,
  "createdAt" TIMESTAMP NOT NULL DEFAULT NOW()
);
```

---

# Step 3

Prisma connects to PostgreSQL.

```text
localhost
      ↓
5432
      ↓
bookmyvenue database
```

and executes:

```sql
CREATE TABLE "User"
```

---

# Database Before Migration

```text
bookmyvenue
      ↓
(no tables)
```

---

# Database After Migration

```text
bookmyvenue
      ↓
User
_prisma_migrations
```

---

# Step 4

Prisma creates migration files.

```text
prisma/
├── schema.prisma
└── migrations/
    └── 20260617123000_init/
        └── migration.sql
```

---

# Why is `migration.sql` important?

Because your teammates can run:

```bash
npm install
npx prisma migrate dev
```

and get exactly the same database structure.

Database schema becomes version controlled.

---

# Step 5

Prisma updates:

```text
_prisma_migrations
```

table.

This table keeps track of:

```text
Migration Name
Execution Time
Status
Checksum
```

Example:

```text
_prisma_migrations
-------------------
init
add_role
add_booking
venue_relations
```

Prisma now knows:

```text
✓ init already executed
✓ add_role already executed
```

and won't execute them again.

---

# Step 6

Prisma automatically runs:

```bash
npx prisma generate
```

This creates:

```text
node_modules/
└── @prisma/client
```

Now TypeScript knows about your models.

You can write:

```ts
prisma.user.findMany();

prisma.user.create({
  data: {},
});

prisma.user.findUnique({
  where: {},
});
```

with full autocomplete and type safety.

---

# Complete Flow

```text
schema.prisma
      ↓
Prisma reads models
      ↓
Generates SQL
      ↓
Executes SQL on PostgreSQL
      ↓
Creates migration files
      ↓
Updates _prisma_migrations
      ↓
Generates Prisma Client
```

---

# Example: Adding a New Field

## Old Schema

```prisma
model User {
  id       Int
  email    String
  password String
}
```

---

## New Schema

```prisma
model User {
  id       Int
  email    String
  password String
  role     String
}
```

Run:

```bash
npx prisma migrate dev --name add_role
```

Prisma generates:

```sql
ALTER TABLE "User"
ADD COLUMN "role" TEXT;
```

Database is updated automatically.

---

# Most Common Prisma Commands

## Create a migration

```bash
npx prisma migrate dev --name init
```

**Use when:**

* Creating tables
* Adding columns
* Removing columns
* Changing relationships

---

## Generate Prisma Client

```bash
npx prisma generate
```

**Use when:**

* Schema changed
* Pulling latest code from Git
* Prisma Client missing

---

## Open Prisma Studio

```bash
npx prisma studio
```

**Use when:**

* Viewing data
* Editing records
* Debugging database entries

Think:

```text
Database GUI for Prisma
```

---

## Reset Database

```bash
npx prisma migrate reset
```

**Use when:**

* Development database is broken
* You want a clean database
* Testing migrations again

⚠️ Deletes all data.

---

# Mental Model

```text
schema.prisma
      ↓
(Blueprint)

migrate dev
      ↓
(Builder)

PostgreSQL Database
      ↓
(House)

Prisma Client
      ↓
(Remote Control for the House)
```

---

# Typical Workflow During Development

```text
1. Edit schema.prisma
          ↓
2. npx prisma migrate dev --name some_change
          ↓
3. Prisma updates PostgreSQL
          ↓
4. Prisma regenerates Prisma Client
          ↓
5. Use prisma.user, prisma.booking,
   prisma.venue in NestJS services
```

---

# Real Example for Book My Venue

```text
schema.prisma
      ↓
User
Venue
Booking
      ↓
npx prisma migrate dev --name init
      ↓
PostgreSQL Tables Created
      ↓
Prisma Client Generated
      ↓
NestJS Services Can Query Database
```
# NestJS + PostgreSQL + Prisma (After Initial Setup)

At this point, you should already know:

* Installing PostgreSQL
* Creating databases
* Connecting with `psql`
* Installing Prisma
* `npx prisma init`
* Configuring `.env`
* Understanding `schema.prisma`
* Creating models
* Running migrations
* Creating `PrismaService`
* Injecting `PrismaService`
* Basic CRUD operations

---

# What Should I Learn Next?

The database connection is working, but there are still several important Prisma concepts that you'll use in real applications.

---

# 1. Database Relationships

This is probably the most important topic after basic CRUD.

Real applications rarely have isolated tables.

Example:

```text
User
 ├── owns many Venues
 └── has many Bookings

Venue
 └── has many Bookings
```

---

## One-to-Many Relationship

### Example

One user can own many venues.

```prisma
model User {
  id      Int     @id @default(autoincrement())
  email   String  @unique

  venues  Venue[]
}

model Venue {
  id       Int    @id @default(autoincrement())
  name     String

  ownerId  Int
  owner    User   @relation(
    fields: [ownerId],
    references: [id]
  )
}
```

---

## Understanding This

### User

```text
User
 └── venues: Venue[]
```

means:

> A user can have multiple venues.

---

### Venue

```text
Venue
 ├── ownerId
 └── owner
```

means:

> Every venue belongs to one user.

---

## Database Representation

### User Table

| id | email                                     |
| -- | ----------------------------------------- |
| 1  | [owner@gmail.com](mailto:owner@gmail.com) |

### Venue Table

| id | name   | ownerId |
| -- | ------ | ------- |
| 1  | Hall A | 1       |
| 2  | Hall B | 1       |

User 1 owns:

```text
Hall A
Hall B
```

---

# Why Relationships Matter

For Book My Venue:

```text
User
 ├── owns many Venues
 ├── has many Bookings
 └── may have many Reviews

Venue
 ├── belongs to User
 ├── has many Bookings
 └── has many Reviews

Booking
 ├── belongs to User
 └── belongs to Venue
```

Almost every API endpoint will use relationships.

---

# 2. Prisma Queries

After creating models, you'll query data.

---

## Find All Records

```ts
await prisma.user.findMany();
```

SQL equivalent:

```sql
SELECT * FROM User;
```

---

## Find One Record

```ts
await prisma.user.findUnique({
  where: {
    email: 'amith@gmail.com',
  },
});
```

SQL:

```sql
SELECT *
FROM User
WHERE email='amith@gmail.com';
```

---

## Create Record

```ts
await prisma.user.create({
  data: {
    email: 'amith@gmail.com',
    password: '123',
  },
});
```

SQL:

```sql
INSERT INTO User;
```

---

## Update Record

```ts
await prisma.user.update({
  where: {
    id: 1,
  },
  data: {
    email: 'new@gmail.com',
  },
});
```

SQL:

```sql
UPDATE User
SET email='new@gmail.com'
WHERE id=1;
```

---

## Delete Record

```ts
await prisma.user.delete({
  where: {
    id: 1,
  },
});
```

SQL:

```sql
DELETE FROM User
WHERE id=1;
```

---

## Upsert

Update if exists.

Create if not.

```ts
await prisma.user.upsert({
  where: {
    email: 'admin@gmail.com',
  },
  update: {},
  create: {
    email: 'admin@gmail.com',
    password: '123',
  },
});
```

Useful for:

* Seed data
* Default admin accounts
* Initial application setup

---

# 3. Filtering

Example:

```ts
await prisma.user.findMany({
  where: {
    role: 'ADMIN',
  },
});
```

SQL:

```sql
SELECT *
FROM User
WHERE role='ADMIN';
```

---

## Multiple Filters

```ts
await prisma.user.findMany({
  where: {
    role: 'OWNER',
    isActive: true,
  },
});
```

SQL:

```sql
SELECT *
FROM User
WHERE role='OWNER'
AND isActive=true;
```

---

# Why Filtering Matters

Examples:

```text
All active venues
All bookings for user
All venue owners
Search venues by city
Search bookings by date
```

You'll use filtering everywhere.

---

# 4. Selecting Fields

Sometimes you don't need every column.

Example:

```ts
await prisma.user.findMany({
  select: {
    id: true,
    email: true,
  },
});
```

Result:

```json
[
  {
    "id": 1,
    "email": "amith@gmail.com"
  }
]
```

---

## Why Use Select?

Avoid:

```json
{
  "id": 1,
  "email": "amith@gmail.com",
  "password": "hashed_password"
}
```

Never send passwords to clients.

---

# 5. Include Relationships

Example:

```ts
await prisma.user.findUnique({
  where: {
    id: 1,
  },
  include: {
    venues: true,
  },
});
```

Result:

```json
{
  "id": 1,
  "email": "owner@gmail.com",
  "venues": [
    {
      "id": 1,
      "name": "Hall A"
    }
  ]
}
```

---

# Why Include Matters

Examples:

```text
Get venue with owner details
Get user with bookings
Get booking with venue details
Get venue with reviews
```

You'll use `include` constantly.

---

# 6. Pagination

Imagine:

```text
5000 venues
```

Returning all:

```ts
findMany()
```

would be inefficient.

---

## Pagination

```ts
await prisma.venue.findMany({
  skip: 0,
  take: 10,
});
```

Result:

```text
Records 1-10
```

Next page:

```ts
await prisma.venue.findMany({
  skip: 10,
  take: 10,
});
```

Result:

```text
Records 11-20
```

---

# Why Pagination Matters

Examples:

```text
Browse venues
Booking history
Admin dashboard
Search results
```

Nearly every production API uses pagination.

---

# 7. Transactions

Sometimes multiple operations must succeed together.

Example:

Booking process:

```text
Create Booking
Create Payment
Update Venue Availability
```

If one fails:

```text
Everything should rollback.
```

---

## Prisma Transaction

```ts
await prisma.$transaction([
  prisma.booking.create(...),
  prisma.payment.create(...),
  prisma.venue.update(...),
]);
```

---

# Why Transactions Matter

Examples:

```text
Booking systems
Payments
Inventory updates
Banking operations
```

Book My Venue will definitely need transactions.

---

# 8. Prisma Error Handling

Database operations can fail.

Example:

```ts
try {
  await prisma.user.create(...);
} catch (error) {
  console.log(error);
}
```

---

## Common Errors

### P2002

```text
Unique constraint failed
```

Example:

```text
Email already exists.
```

---

### P2025

```text
Record not found
```

Example:

```text
User with id=5 does not exist.
```

---

# Why Error Handling Matters

Never trust:

```text
Database always succeeds
```

Failures happen.

Your APIs must handle them gracefully.

---

# 9. Database Seeding

Sometimes you need initial data.

Examples:

```text
Admin account
Default venue categories
Test users
Demo bookings
```

---

## Seed Command

```bash
npx prisma db seed
```

This automatically inserts starter data.

---

# 10. Prisma Studio

Prisma provides a database GUI.

Run:

```bash
npx prisma studio
```

Browser opens:

```text
http://localhost:5555
```

---

# What Can You Do?

* View tables
* Create rows
* Edit rows
* Delete rows
* Debug data

Think:

```text
Prisma Studio
      ↓
Database Admin Panel
```

---

# Production Topics (Learn Later)

---

# Connection Lifecycle

```ts
onModuleInit()

enableShutdownHooks()
```

Purpose:

* Proper database startup
* Graceful shutdown
* Prevent connection leaks

---

# Soft Deletes

Instead of:

```text
DELETE
```

Use:

```text
deletedAt
```

Example:

```prisma
deletedAt DateTime?
```

Record remains in database.

Benefits:

* Recover deleted records
* Audit history
* Prevent accidental data loss

---

# Indexes

Example:

```prisma
@@index([email])
```

Purpose:

```text
Faster database searches
```

Useful for:

* email
* city
* bookingDate
* ownerId

---

# Enums

Example:

```prisma
enum Role {
  USER
  OWNER
  ADMIN
}
```

Then:

```prisma
role Role
```

Benefits:

```text
No invalid roles
Type safety
Cleaner code
```

Book My Venue will definitely use enums.

---

# Suggested Learning Order

```text
1. Relationships
2. Queries
3. Filtering
4. Select
5. Include
6. Pagination
7. Transactions
8. Error Handling
9. Seeding
10. Prisma Studio
11. Enums
12. Indexes
13. Soft Deletes
14. Connection Lifecycle
```

---

# Architecture So Far

```text
DTO
 ↓
Controller
 ↓
Service
 ↓
PrismaService
 ↓
Prisma Client
 ↓
PostgreSQL
 ↓
Database
 ↓
Response
```

---

# Next Major Topic

After learning these concepts:

```text
Authentication
      ↓
Register
      ↓
Hash Password (Argon2)
      ↓
Login
      ↓
JWT
      ↓
Guards
      ↓
Role Authorization
```

# Understanding Prisma Relationships in Depth

Prisma relationships are built using three things:

1. Relation Fields
2. Foreign Keys
3. `@relation()` attribute

Unlike TypeORM, Prisma does **not** use decorators like:

```ts
@OneToMany()
@ManyToOne()
@OneToOne()
@ManyToMany()
```

Instead, relationships are defined directly inside `schema.prisma`.

---

# Mental Model

```text
[]        → MANY
No []     → ONE

xxxId     → Foreign Key

owner User
      ↓
Related Object

@relation()
      ↓
Connects the foreign key
to another table's primary key
```

---

# Parent and Child Sides

Usually:

```text
Parent
   ↓
Array ([])

Child
   ↓
Foreign Key (xxxId)
+
Relation Field
+
@relation()
```

---

# Example: One User Owns Many Venues

## Relationship Diagram

```text
User
 ├── Hall A
 ├── Hall B
 └── Hall C
```

This is:

```text
User perspective
      ↓
One-to-Many

Venue perspective
      ↓
Many-to-One
```

Both are the same relationship viewed from opposite sides.

---

# Prisma Schema

```prisma
model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique

  venues  Venue[]
}

model Venue {
  id       Int      @id @default(autoincrement())
  name     String

  ownerId  Int
  owner    User     @relation(
    fields: [ownerId],
    references: [id]
  )
}
```

---

# Understanding Each Line

## Parent Side

```prisma
venues Venue[]
```

Read it as:

```text
One User
      ↓
Many Venues
```

The array:

```prisma
[]
```

means:

```text
MANY
```

---

## Child Side

```prisma
ownerId Int
```

This is the foreign key.

Think:

```text
Venue.ownerId
      ↓
1
      ↓
User.id
```

Database:

### User

| id | email                                     |
| -- | ----------------------------------------- |
| 1  | [owner@gmail.com](mailto:owner@gmail.com) |

### Venue

| id | name   | ownerId |
| -- | ------ | ------- |
| 1  | Hall A | 1       |
| 2  | Hall B | 1       |

---

## Relation Field

```prisma
owner User
```

This means:

```text
Every Venue
      ↓
has one User object
```

Because of this field:

```ts
venue.owner.email
```

becomes possible.

---

## `@relation()`

```prisma
@relation(
  fields: [ownerId],
  references: [id]
)
```

means:

```text
Use:
ownerId

to reference:

User.id
```

Visual:

```text
Venue.ownerId
       ↓
       1
       ↓
User.id
```

---

# Database Representation

### User Table

| id | email                                     |
| -- | ----------------------------------------- |
| 1  | [owner@gmail.com](mailto:owner@gmail.com) |

### Venue Table

| id | name   | ownerId |
| -- | ------ | ------- |
| 1  | Hall A | 1       |
| 2  | Hall B | 1       |

---

# Query: User → Venues

```ts
const user =
  await prisma.user.findUnique({
    where: {
      id: 1,
    },
    include: {
      venues: true,
    },
  });
```

Result:

```json
{
  "id": 1,
  "email": "owner@gmail.com",
  "venues": [
    {
      "id": 1,
      "name": "Hall A"
    },
    {
      "id": 2,
      "name": "Hall B"
    }
  ]
}
```

---

# Query: Venue → Owner

```ts
const venue =
  await prisma.venue.findUnique({
    where: {
      id: 1,
    },
    include: {
      owner: true,
    },
  });
```

Result:

```json
{
  "id": 1,
  "name": "Hall A",
  "owner": {
    "id": 1,
    "email": "owner@gmail.com"
  }
}
```

---

# One-to-One Relationship

Example:

```text
User
   ↓
Profile
```

One user has exactly one profile.

---

## Prisma Schema

```prisma
model User {
  id      Int      @id @default(autoincrement())

  profile Profile?
}

model Profile {
  id      Int      @id @default(autoincrement())

  userId  Int      @unique
  user    User     @relation(
    fields: [userId],
    references: [id]
  )
}
```

---

# Why `@unique`?

```prisma
userId Int @unique
```

means:

```text
User 1
   ↓
Profile 1

User 1
   ↓
❌ Cannot have Profile 2
```

Only one profile can exist for one user.

---

# Many-to-Many Relationship

Example:

```text
Student
      ↕
Course
```

One student:

```text
Math
Physics
Chemistry
```

One course:

```text
Amith
John
Alice
```

Both sides can have many records.

---

## Prisma Schema

```prisma
model Student {
  id       Int       @id @default(autoincrement())
  name     String

  courses  Course[]
}

model Course {
  id        Int       @id @default(autoincrement())
  title     String

  students  Student[]
}
```

Prisma automatically creates a junction table:

```text
StudentCourse
```

behind the scenes.

---

# Mapping TypeORM to Prisma

| TypeORM         | Prisma                                  |
| --------------- | --------------------------------------- |
| `@OneToMany()`  | `Venue[]`                               |
| `@ManyToOne()`  | `ownerId` + `owner User @relation(...)` |
| `@OneToOne()`   | `Profile?` + `userId @unique`           |
| `@ManyToMany()` | `Course[]` and `Student[]`              |

---

# Quick Rule of Thumb

```text
[]                 → MANY

No []              → ONE

xxxId              → Foreign Key

owner User         → Related Object

@relation()        → Connects tables

@unique            → Forces one-to-one

include            → Loads related data
```

---

# Book My Venue Database Design

```text
User
 ├── venues: Venue[]
 └── bookings: Booking[]

Venue
 ├── ownerId
 ├── owner: User
 └── bookings: Booking[]

Booking
 ├── userId
 ├── user: User
 ├── venueId
 └── venue: Venue
```

Visual:

```text
User
 ├─────────────┐
 │             │
 ▼             ▼
Venue       Booking
  │             ▲
  └─────────────┘
```

These relationships form the foundation of your entire backend and will be used in almost every API you build in Book My Venue.
