Issue: Prisma 7 startup failure in `PrismaService`

Current reproducible error

When running `npm run start`, the app fails during Nest bootstrap with:

```text
PrismaClientInitializationError: `PrismaClient` needs to be constructed with a non-empty, valid `PrismaClientOptions`
```

The stack points to:

- `src/prisma/prisma.service.ts`
- the `super()` call inside `new PrismaService(...)`

What is actually wrong

- This project uses `@prisma/client` `7.8.0`.
- The generated Prisma client in v7 is using the `client` engine path, which requires either:
  - a driver adapter, or
  - Prisma Accelerate.
- The current code constructs Prisma with `super()` and no adapter.
- The generated schema for this client also has no datasource `url` in `schema.prisma`, so simply setting `process.env.DATABASE_URL` is not enough for this generated client shape.

Why the previous report was incomplete

- The earlier `datasources` / `super()` ordering problem may have been real in a previous revision.
- It is not the current blocker anymore.
- The present startup failure happens earlier and is fully reproducible with the current codebase.

Correct fix

Use Prisma's PostgreSQL driver adapter and pass it into `PrismaClient`:

```ts
const databaseUrl =
  configService.get<string>('DATABASE_URL') || process.env.DATABASE_URL;

super({
  adapter: new PrismaPg({ connectionString: databaseUrl }),
});
```

Dependencies required

- `@prisma/adapter-pg`
- `pg`

Related cleanup

- `AuthModule` was also declaring `PrismaService` directly even though `PrismaModule` already provides it.
- That would create duplicate Prisma service instances once startup succeeds, so it should import `PrismaModule` and stop re-providing `PrismaService`.

Status

- Root cause identified: missing Prisma 7 driver adapter configuration.
- Code updated locally to use the adapter-based pattern.
- Package installation is still required before the app can be re-run successfully.

  Issue: Prisma 7 startup failure in `PrismaService`

Current reproducible error

When running `npm run start`, the app fails during Nest bootstrap with:

```text
PrismaClientInitializationError: `PrismaClient` needs to be constructed with a non-empty, valid `PrismaClientOptions`
```

The stack points to:

- `src/prisma/prisma.service.ts`
- the `super()` call inside `new PrismaService(...)`

What is actually wrong

- This project uses `@prisma/client` `7.8.0`.
- The generated Prisma client in v7 is using the `client` engine path, which requires either:
  - a driver adapter, or
  - Prisma Accelerate.
- The current code constructs Prisma with `super()` and no adapter.
- The generated schema for this client also has no datasource `url` in `schema.prisma`, so simply setting `process.env.DATABASE_URL` is not enough for this generated client shape.

Why the previous report was incomplete

- The earlier `datasources` / `super()` ordering problem may have been real in a previous revision.
- It is not the current blocker anymore.
- The present startup failure happens earlier and is fully reproducible with the current codebase.

Correct fix

Use Prisma's PostgreSQL driver adapter and pass it into `PrismaClient`:

```ts
const databaseUrl =
  configService.get<string>('DATABASE_URL') || process.env.DATABASE_URL;

super({
  adapter: new PrismaPg({ connectionString: databaseUrl }),
});
```

Dependencies required

- `@prisma/adapter-pg`
- `pg`

Related cleanup

- `AuthModule` was also declaring `PrismaService` directly even though `PrismaModule` already provides it.
- That would create duplicate Prisma service instances once startup succeeds, so it should import `PrismaModule` and stop re-providing `PrismaService`.

Status

- Root cause identified: missing Prisma 7 driver adapter configuration.
- Code updated locally to use the adapter-based pattern.
- Package installation is still required before the app can be re-run successfully.
