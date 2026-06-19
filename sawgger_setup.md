# Swagger Setup Guide for NestJS

## Book My Venue Backend Documentation

---

# What is Swagger?

Swagger is an API documentation tool that automatically generates interactive documentation for your NestJS application.

Instead of manually documenting APIs:

```text
Frontend Developer
        ↓
Ask Backend Developer
        ↓
What is the endpoint?
What is the request body?
What does it return?
```

Swagger provides everything automatically:

```text
Browser
      ↓
http://localhost:3000/api
      ↓
Interactive API Documentation
      ↓
Test Endpoints Directly
```

---

# Why Use Swagger?

### Without Swagger

```text
❌ Manual documentation
❌ Constantly using Postman
❌ Frontend team asks for endpoint details
❌ Hard to understand request bodies
❌ Difficult to test APIs
```

### With Swagger

```text
✅ Auto-generated documentation
✅ Interactive UI
✅ Test APIs directly from browser
✅ Shows request and response schemas
✅ Supports JWT authentication
✅ Great for team collaboration
```

---

# Installation

Install Swagger packages:

```bash
npm install @nestjs/swagger swagger-ui-express
```

---

# Architecture

```text
DTO
 ↓
Swagger Decorators
 ↓
Controller
 ↓
SwaggerModule
 ↓
OpenAPI Document
 ↓
Browser UI
```

---

# Configure Swagger

## main.ts

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

import { SwaggerModule, DocumentBuilder } from "@nestjs/swagger";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle("Book My Venue API")
    .setDescription("Backend API Documentation for Book My Venue")
    .setVersion("1.0")
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);

  SwaggerModule.setup("api", app, document);

  await app.listen(3000);
}

bootstrap();
```

---

# What Happens?

```text
Application Starts
        ↓
Swagger Config Created
        ↓
OpenAPI Document Generated
        ↓
Swagger UI Created
        ↓
Visit:
http://localhost:3000/api
```

---

# Understanding DocumentBuilder

## Set Title

```ts
.setTitle('Book My Venue API')
```

Displays:

```text
Book My Venue API
```

---

## Set Description

```ts
.setDescription(
  'Backend API Documentation'
)
```

Displays:

```text
Backend API Documentation
```

---

## Set Version

```ts
.setVersion('1.0')
```

Displays:

```text
Version 1.0
```

Useful for:

```text
v1
v2
v3
```

API versioning.

---

## Add Bearer Authentication

```ts
.addBearerAuth()
```

Adds:

```text
Authorize 🔒
```

button in Swagger UI.

Used for:

```text
JWT Authentication
Protected Endpoints
```

---

# SwaggerModule.createDocument()

```ts
const document = SwaggerModule.createDocument(app, config);
```

Purpose:

```text
Read Controllers
      ↓
Read DTOs
      ↓
Read Decorators
      ↓
Generate OpenAPI Specification
```

---

# SwaggerModule.setup()

```ts
SwaggerModule.setup("api", app, document);
```

Creates:

```text
http://localhost:3000/api
```

---

# Request Flow

```text
Browser
      ↓
http://localhost:3000/api
      ↓
Swagger UI
      ↓
OpenAPI Document
      ↓
Controllers
      ↓
Endpoints
```

---

# Organizing APIs

## Controller Tags

```ts
import { ApiTags } from "@nestjs/swagger";

@ApiTags("Authentication")
@Controller("auth")
export class AuthController {}
```

Swagger:

```text
Authentication
├── POST /auth/register
├── POST /auth/login
└── POST /auth/logout
```

---

# Documenting Endpoints

## Register Endpoint

```ts
@Post('register')
@ApiOperation({
  summary: 'Register User',
})
register() {}
```

Displays:

```text
POST /auth/register

Summary:
Register User
```

---

## Login Endpoint

```ts
@Post('login')
@ApiOperation({
  summary: 'Login User',
})
login() {}
```

Displays:

```text
POST /auth/login

Summary:
Login User
```

---

# Documenting Responses

## Success Response

```ts
@ApiResponse({
  status: 201,
  description:
    'User registered successfully',
})
```

---

## Error Response

```ts
@ApiResponse({
  status: 401,
  description:
    'Invalid credentials',
})
```

---

# Example

```ts
@Post('login')
@ApiOperation({
  summary: 'Login User',
})
@ApiResponse({
  status: 200,
  description:
    'Login Successful',
})
@ApiResponse({
  status: 401,
  description:
    'Invalid Credentials',
})
login() {}
```

---

# DTO Documentation

Without decorators:

```ts
export class RegisterDto {
  email: string;
  password: string;
}
```

Swagger knows very little.

---

# Using ApiProperty

```ts
import { ApiProperty } from "@nestjs/swagger";

export class RegisterDto {
  @ApiProperty({
    example: "amith@gmail.com",
  })
  email: string;

  @ApiProperty({
    example: "Password@123",
  })
  password: string;
}
```

---

# Swagger Output

```text
RegisterDto

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

# Login DTO

```ts
export class LoginDto {
  @ApiProperty({
    example: "amith@gmail.com",
  })
  email: string;

  @ApiProperty({
    example: "Password@123",
  })
  password: string;
}
```

---

# JWT Authentication

## Protected Routes

```ts
@ApiBearerAuth()
@UseGuards(JwtAuthGuard)
@Get('me')
getProfile() {}
```

Swagger displays:

```text
🔒 GET /auth/me
```

---

# Using Authorization Button

Click:

```text
Authorize 🔒
```

Enter:

```text
Bearer eyJhbGciOi...
```

Swagger automatically sends:

```http
Authorization: Bearer eyJhbGciOi...
```

with every protected request.

---

# Book My Venue Authentication APIs

```text
Authentication
├── POST /auth/register
├── POST /auth/login
├── POST /auth/refresh
├── POST /auth/logout
├── POST /auth/verify-email
├── POST /auth/forgot-password
├── POST /auth/reset-password
├── GET  /auth/google
└── GET  /auth/google/callback
```

---

# Suggested Decorators

## Controller

```ts
@ApiTags("Authentication")
@Controller("auth")
export class AuthController {}
```

---

## Register

```ts
@Post('register')
@ApiOperation({
  summary:
    'Register a new user',
})
@ApiResponse({
  status: 201,
  description:
    'User registered successfully',
})
```

---

## Login

```ts
@Post('login')
@ApiOperation({
  summary:
    'Login user',
})
@ApiResponse({
  status: 200,
  description:
    'Login successful',
})
@ApiResponse({
  status: 401,
  description:
    'Invalid credentials',
})
```

---

## Profile

```ts
@ApiBearerAuth()
@UseGuards(JwtAuthGuard)
@Get('me')
@ApiOperation({
  summary:
    'Get current user profile',
})
```

---

# Production Benefits

```text
Backend Developer
      ↓
Build Endpoint
      ↓
Swagger Updates Automatically
      ↓
Frontend Opens Browser
      ↓
Understands API Immediately
```

---

# Mental Model

```text
DTO
 ↓
@ApiProperty()

Controller
 ↓
@ApiTags()
@ApiOperation()
@ApiResponse()
@ApiBearerAuth()

SwaggerModule
 ↓
OpenAPI Document
 ↓
Interactive Browser UI
```

---

# Recommended Setup Order for Book My Venue

```text
Prisma
      ↓
Config Module
      ↓
User Entity
      ↓
Swagger
      ↓
DTO Validation
      ↓
Register
      ↓
Login
      ↓
JWT
      ↓
RBAC
      ↓
Email Verification
      ↓
Password Reset
      ↓
Google OAuth
```

---

# Checklist

```text
✓ Install @nestjs/swagger
✓ Install swagger-ui-express
✓ Configure Swagger in main.ts
✓ Create API documentation endpoint
✓ Add Controller tags
✓ Add Endpoint descriptions
✓ Document DTOs
✓ Add JWT Bearer authentication
✓ Test APIs directly from browser
```

Swagger Endpoint:

```text
http://localhost:3000/api
```

This should remain enabled during development and can optionally be disabled in production environments.
