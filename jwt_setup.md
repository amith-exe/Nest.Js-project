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

At this point your authentication system starts behaving like a production backend.
