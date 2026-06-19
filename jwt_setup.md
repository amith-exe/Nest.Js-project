# JWT Setup in NestJS (Production Notes)

# What is JWT?

JWT stands for **JSON Web Token**.

JWT is a secure way to store a user's identity information and send it between the client and server.

Example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

JWT is commonly used for:

- Login Authentication
- Protected Routes
- User Sessions
- API Authentication
- Mobile App Authentication

---

# Why Do We Need JWT?

HTTP is stateless.

Example:

```text
Request 1
↓
Login
↓
Server responds
↓
Connection ends

Request 2
↓
Get Profile
↓
Server has forgotten who you are
```

The server needs a way to know:

```text
Who is making this request?
```

JWT solves this problem.

---

# Authentication Flow

```text
Login Request
      ↓
Verify Email & Password
      ↓
Generate JWT
      ↓
Send JWT to Frontend
      ↓
Frontend stores token
      ↓
Every request sends token
      ↓
Server verifies token
      ↓
User identified
```

---

# JWT Structure

JWT has 3 parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
eyJhbGciOiJIUzI1Ni...
```

---

# Header

Contains algorithm information.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

# Payload

Contains user information.

Example:

```json
{
  "sub": "user-id",
  "email": "user@example.com",
  "role": "USER"
}
```

---

# Signature

Generated using:

```text
Payload
+
Secret Key
↓
Digital Signature
```

The signature prevents people from modifying tokens.

---

# Why JWT_SECRET Exists

Suppose payload:

```json
{
  "sub": "123",
  "role": "USER"
}
```

Attacker changes it:

```json
{
  "sub": "123",
  "role": "ADMIN"
}
```

Without signature:

```text
Attacker becomes admin.
```

With JWT_SECRET:

```text
Payload changed
↓
Signature invalid
↓
401 Unauthorized
```

---

# Install JWT Packages

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install -D @types/passport-jwt
```

---

# Why Each Package?

## @nestjs/jwt

Provides:

```ts
JwtService;
```

Used for:

```ts
sign();
signAsync();
verify();
```

---

## passport

Authentication framework.

Used for:

```text
Strategies
Guards
Authentication flow
```

---

## passport-jwt

Passport strategy for JWT.

Used for:

```text
Read token
Verify token
Decode payload
```

---

## @nestjs/passport

NestJS integration for Passport.

Used for:

```ts
PassportStrategy;
AuthGuard("jwt");
```

---

# Environment Variables

## .env

```env
JWT_SECRET=bookmyvenue-secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

---

# JWT_SECRET

Used to sign and verify JWT tokens.

Example:

```text
Payload
+
JWT_SECRET
↓
Signature
↓
JWT Token
```

For development:

```env
JWT_SECRET=bookmyvenue-secret
```

For production:

Use a long random string.

---

# JWT_ACCESS_EXPIRES_IN

Controls how long access tokens live.

Example:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

Means:

```text
Token expires after 15 minutes.
```

---

# JWT_REFRESH_EXPIRES_IN

Controls how long refresh tokens live.

Example:

```env
JWT_REFRESH_EXPIRES_IN=7d
```

Means:

```text
Refresh token expires after 7 days.
```

---

# Configure JwtModule

## auth.module.ts

```ts
JwtModule.registerAsync({
  imports: [ConfigModule],
  inject: [ConfigService],

  useFactory: (configService: ConfigService) => ({
    secret: configService.get<string>("JWT_SECRET")!,

    signOptions: {
      expiresIn: configService.get<string>("JWT_ACCESS_EXPIRES_IN")!,
    },
  }),
});
```

---

# registerAsync()

Means:

```text
Configure JwtModule dynamically
when application starts.
```

Flow:

```text
Nest Starts
      ↓
Load ConfigModule
      ↓
Read .env
      ↓
Configure JwtModule
      ↓
Create JwtService
```

---

# imports

```ts
imports: [ConfigModule];
```

Means:

```text
JwtModule depends on ConfigModule.
```

Because:

```ts
ConfigService;
```

comes from:

```ts
ConfigModule;
```

---

# inject

```ts
inject: [ConfigService];
```

Means:

```text
Nest,
please give me
an instance of ConfigService.
```

Equivalent idea:

```ts
const configService = new ConfigService();
```

Nest does this automatically.

---

# useFactory()

```ts
useFactory:
(configService) => ({
  ...
})
```

Think:

```text
Receive ConfigService
↓
Read environment variables
↓
Return JWT configuration
```

Equivalent:

```ts
function createJwtConfig(
  configService,
) {
  return {
    ...
  };
}
```

---

# secret

```ts
secret: configService.get<string>("JWT_SECRET")!;
```

Suppose:

```env
JWT_SECRET=bookmyvenue-secret
```

Result:

```ts
secret: "bookmyvenue-secret";
```

Used for:

```text
Generate Signature
Verify Signature
```

---

# expiresIn

```ts
expiresIn: configService.get<string>("JWT_ACCESS_EXPIRES_IN")!;
```

Suppose:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

Result:

```ts
expiresIn: "15m";
```

Every generated token expires after:

```text
15 minutes
```

---

# Why ! Is Used

TypeScript thinks:

```ts
configService.get<string>("JWT_SECRET");
```

returns:

```ts
string | undefined;
```

because maybe:

```env
JWT_SECRET=
```

doesn't exist.

The operator:

```ts
!
```

means:

```text
Trust me.
This value exists.
```

It only removes TypeScript errors.

It does NOT affect security.

---

# Dependency Injection Flow

```text
ConfigModule
      ↓
ConfigService
      ↓
JwtModule
      ↓
JwtService
      ↓
AuthService
```

---

# Inject JwtService

```ts
constructor(
  private prisma: PrismaService,
  private jwtService: JwtService,
) {}
```

Now you can use:

```ts
this.jwtService.sign();
this.jwtService.signAsync();
this.jwtService.verify();
```

---

# Creating JWT Payload

Example:

```ts
const payload = {
  sub: user.id,
  email: user.email,
  role: user.role,
};
```

---

# Why sub?

JWT standard:

```text
sub
↓
Subject
↓
User ID
```

Most production applications use:

```ts
payload.sub;
```

instead of:

```ts
payload.id;
```

---

# Generate Token

```ts
const accessToken = await this.jwtService.signAsync(payload);
```

Flow:

```text
Payload
     ↓
JWT_SECRET
     ↓
Digital Signature
     ↓
JWT Token
```

Result:

```text
eyJhbGciOiJIUzI1Ni...
```

---

# Login Flow

```text
POST /auth/login
        ↓
Find User
        ↓
Compare Password
        ↓
Create Payload
        ↓
Generate JWT
        ↓
Return Token
```

Response:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1Ni..."
}
```

---

# Current Progress

```text
☑ Install JWT packages
☑ Setup .env variables
☑ Configure JwtModule
☑ Create JwtService
☑ Inject JwtService
☐ Generate access token
☐ Validate token
☐ Create JwtStrategy
☐ Create JwtGuard
☐ Protected routes
☐ Refresh tokens
```

---

# Next Step

After configuration:

```text
Login
      ↓
Generate Access Token
      ↓
Return Token
      ↓
Passport JWT Strategy
      ↓
JwtAuthGuard
      ↓
Protected Routes
      ↓
req.user
```

At this point your authentication system starts behaving like a production backend.# JWT Configuration in NestJS (Production Notes)

# Purpose

The goal of this configuration is:

```text
Read .env
    ↓
Create JWT Configuration Object
    ↓
Give Configuration to JwtModule
    ↓
JwtModule creates JwtService
    ↓
JwtService can generate and verify JWTs
```

---

# Environment Variables

## .env

```env
JWT_SECRET=bookmyvenue-secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

---

# What is JWT_SECRET?

The secret key used to:

1. Generate JWT signatures
2. Verify JWT signatures

Flow:

```text
Payload
   +
JWT_SECRET
   ↓
Digital Signature
   ↓
JWT Token
```

Example:

```json
{
  "sub": "123",
  "email": "user@example.com",
  "role": "USER"
}
```

Result:

```text
eyJhbGciOiJIUzI1Ni...
```

---

# Why do we need JWT_SECRET?

Without a signature:

```json
{
  "role": "USER"
}
```

could be modified to:

```json
{
  "role": "ADMIN"
}
```

With JWT_SECRET:

```text
Payload Modified
       ↓
Signature Invalid
       ↓
401 Unauthorized
```

---

# What is JWT_ACCESS_EXPIRES_IN?

Controls how long access tokens remain valid.

Example:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

Means:

```text
Access Token
      ↓
Valid for 15 minutes
      ↓
Expires
      ↓
401 Unauthorized
```

---

# JwtModule Configuration

```ts
JwtModule.registerAsync({
  imports: [ConfigModule],
  inject: [ConfigService],

  useFactory: (
    configService: ConfigService,
  ) => {
    const secret =
      configService.get<string>(
        'JWT_SECRET',
      );

    const expiresRaw =
      configService.get<string>(
        'JWT_ACCESS_EXPIRES_IN',
      );

    const expiresIn:
      | number
      | string
      | undefined =
      expiresRaw &&
      /^[0-9]+$/.test(
        expiresRaw,
      )
        ? parseInt(
            expiresRaw,
            10,
          )
        : expiresRaw;

    return {
      secret,
      signOptions: {
        expiresIn:
          expiresIn as any,
      },
    };
  },
});
```

---

# Understanding registerAsync()

```ts
JwtModule.registerAsync(...)
```

Means:

```text
Configure JwtModule
dynamically at runtime.
```

Flow:

```text
Nest Starts
      ↓
Load ConfigModule
      ↓
Create ConfigService
      ↓
Read .env
      ↓
Configure JwtModule
      ↓
Create JwtService
```

---

# imports

```ts
imports: [ConfigModule]
```

Means:

```text
JwtModule depends on ConfigModule.
```

Because:

```ts
ConfigService
```

comes from:

```ts
ConfigModule
```

---

# inject

```ts
inject: [ConfigService]
```

Means:

```text
Nest,
please create ConfigService
and give it to my factory function.
```

Equivalent idea:

```ts
const configService =
  new ConfigService();
```

Nest performs this automatically using Dependency Injection.

---

# useFactory()

```ts
useFactory:
(configService) => {
  ...
}
```

This is simply a function.

Equivalent:

```ts
function createJwtConfig(
  configService,
) {
  ...
}
```

Nest executes this function and expects a configuration object to be returned.

---

# Reading JWT_SECRET

```ts
const secret =
  configService.get<string>(
    'JWT_SECRET',
  );
```

Suppose:

```env
JWT_SECRET=bookmyvenue-secret
```

Result:

```ts
secret =
'bookmyvenue-secret'
```

---

# Return Type

```ts
string | undefined
```

Why?

Because TypeScript cannot read your `.env` file.

It thinks:

```text
Maybe JWT_SECRET exists.
Maybe it doesn't.
```

---

# Reading JWT_ACCESS_EXPIRES_IN

```ts
const expiresRaw =
  configService.get<string>(
    'JWT_ACCESS_EXPIRES_IN',
  );
```

Suppose:

```env
JWT_ACCESS_EXPIRES_IN=15m
```

Result:

```ts
expiresRaw = '15m'
```

---

# Why the Extra Logic?

```ts
const expiresIn =
  expiresRaw &&
  /^[0-9]+$/.test(
    expiresRaw,
  )
    ? parseInt(
        expiresRaw,
        10,
      )
    : expiresRaw;
```

JWT accepts expiration values in two formats.

---

# Format 1: String

```ts
expiresIn: '15m'
expiresIn: '1h'
expiresIn: '7d'
```

Examples:

```text
15m = 15 minutes
1h = 1 hour
7d = 7 days
```

---

# Format 2: Number

```ts
expiresIn: 3600
```

Means:

```text
3600 seconds
```

---

# Understanding the Logic

## Step 1

```ts
expiresRaw &&
```

Checks:

```text
Does a value exist?
```

---

## Step 2

```ts
/^[0-9]+$/.test(
  expiresRaw,
)
```

Checks:

```text
Does the value contain only digits?
```

Examples:

```ts
'3600'
```

↓

```text
true
```

---

```ts
'15m'
```

↓

```text
false
```

---

# Step 3

If only digits:

```ts
parseInt(
  expiresRaw,
  10,
)
```

Example:

```ts
'3600'
```

↓

```ts
3600
```

(number)

Otherwise:

```ts
'15m'
```

stays:

```ts
'15m'
```

(string)

---

# What Does useFactory Return?

Suppose:

```env
JWT_SECRET=bookmyvenue-secret
JWT_ACCESS_EXPIRES_IN=15m
```

The factory returns:

```ts
{
  secret:
    'bookmyvenue-secret',

  signOptions: {
    expiresIn:
      '15m',
  },
}
```

---

# Why Return This Object?

JwtModule itself does not know:

```text
What secret should I use?
How long should tokens live?
```

Your factory function answers those questions.

---

# What Happens Next?

Nest internally does something similar to:

```ts
new JwtService({
  secret:
    'bookmyvenue-secret',

  signOptions: {
    expiresIn:
      '15m',
  },
});
```

Then:

```ts
this.jwtService.sign(...)
this.jwtService.signAsync(...)
this.jwtService.verify(...)
```

become available.

---

# Complete Startup Flow

```text
npm run start
      ↓
Create ConfigService
      ↓
Execute useFactory()
      ↓
Read JWT_SECRET
      ↓
Read JWT_ACCESS_EXPIRES_IN
      ↓
Return:
{
  secret,
  signOptions
}
      ↓
JwtModule receives configuration
      ↓
Creates JwtService
      ↓
JwtService ready
```

---

# Dependency Flow

```text
.env
 │
 ├── JWT_SECRET
 └── JWT_ACCESS_EXPIRES_IN
          ↓
ConfigModule
          ↓
ConfigService
          ↓
useFactory()
          ↓
{
  secret,
  signOptions
}
          ↓
JwtModule
          ↓
JwtService
          ↓
AuthService
          ↓
sign()
signAsync()
verify()
```

---

# What Have We Built?

At this stage:

```text
✓ Read JWT settings from .env
✓ Configure JwtModule
✓ Create JwtService
✓ Set JWT secret
✓ Set JWT expiration
✓ Make JwtService injectable
```

Still Remaining:

```text
☐ Generate Access Token
☐ Validate Token
☐ Passport JWT Strategy
☐ JwtAuthGuard
☐ Protected Routes
☐ Refresh Tokens
```

---

# Mental Model

```text
Configuration Stage
       ↓
Read Environment Variables
       ↓
Build JWT Configuration Object
       ↓
Create JwtService
       ↓
Ready to Generate and Verify JWT Tokens
```


