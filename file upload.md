# Profile Picture Upload - BookMyVenue

## Objective

Allow authenticated users to upload a profile picture, store the image on the server, and save the image URL inside the database.

---

# Endpoint

```http
POST /profile/picture
```

Authentication:

```http
Authorization: Bearer <JWT>
```

Content-Type:

```text
multipart/form-data
```

Request:

```text
file: profile.jpg
```

---

# Why POST?

Uploading a profile picture creates a new file resource.

Profile updates:

```http
PATCH /profile
```

are used for:

* Name
* Phone Number
* Billing Address

Profile picture upload:

```http
POST /profile/picture
```

is different because it uploads a file instead of updating JSON fields.

---

# Why Separate Endpoint?

Bad approach:

```text
PATCH /profile
{
  name,
  phoneNumber,
  billingAddress,
  file
}
```

Problems:

* Mixing JSON and files
* Different Content Types
* More complex validation
* More difficult frontend implementation

Better:

```http
PATCH /profile
```

Handles:

* Name
* Phone Number
* Billing Address

and

```http
POST /profile/picture
```

Handles:

* Profile Image Upload

This follows the Single Responsibility Principle (SRP).

---

# Required Packages

Install Multer:

```bash
npm install multer
npm install -D @types/multer
```

Multer is responsible for handling file uploads.

---

# Request Flow

```text
Flutter
      ↓
multipart/form-data
      ↓
JwtAuthGuard
      ↓
FileInterceptor
      ↓
Multer
      ↓
diskStorage
      ↓
uploads/profiles
      ↓
Controller
      ↓
ProfileService
      ↓
Prisma
      ↓
Database
```

---

# Step 1 - JwtAuthGuard

```ts
@UseGuards(JwtAuthGuard)
```

Responsibilities:

* Validate JWT
* Decode payload
* Populate req.user

Example:

JWT Payload:

```json
{
  "sub": "3f28941e",
  "email": "user@example.com",
  "role": "USER"
}
```

After validation:

```ts
req.user = {
  userId: "3f28941e",
  email: "user@example.com",
  role: "USER"
}
```

This allows us to know exactly which user is uploading the image.

---

# Step 2 - FileInterceptor

```ts
@UseInterceptors(
  FileInterceptor('file')
)
```

FileInterceptor is a NestJS interceptor.

Responsibilities:

* Intercept multipart requests
* Extract files
* Pass files to Multer
* Attach file object to controller

Without FileInterceptor:

```ts
@UploadedFile() file
```

would be:

```ts
undefined
```

because Nest does not understand multipart/form-data by default.

---

# Why is it called an Interceptor?

Nest request lifecycle:

```text
Request
    ↓
Middleware
    ↓
Guards
    ↓
Interceptors
    ↓
Controller
    ↓
Response
```

Interceptors sit between the request and controller and can:

* Modify requests
* Modify responses
* Log information
* Handle file uploads

---

# FileInterceptor('file')

```ts
FileInterceptor('file')
```

means:

Look for a multipart field named:

```text
file
```

Frontend:

```text
file: profile.jpg
```

must match:

```ts
FileInterceptor('file')
```

Otherwise:

```ts
@UploadedFile()
```

will be undefined.

---

# Step 3 - diskStorage

```ts
storage: diskStorage({
  destination: './uploads/profiles',
  filename: ...
})
```

Multer allows different storage engines.

We use:

```text
diskStorage
```

which saves files directly on the server.

---

# destination

```ts
destination:
  './uploads/profiles'
```

means:

Store uploaded files inside:

```text
uploads
└── profiles
```

Example:

```text
uploads/profiles/profile.jpg
```

---

# filename

```ts
filename:
  (_req, file, callback) => {
```

This function determines:

```text
What should this file be called?
```

---

# Why not keep original names?

Suppose:

User A uploads:

```text
profile.jpg
```

User B uploads:

```text
profile.jpg
```

Without renaming:

```text
profile.jpg
```

would overwrite:

```text
profile.jpg
```

This is dangerous.

---

# Creating Unique Names

```ts
const uniqueSuffix =
`${Date.now()}-${Math.round(Math.random() * 1e9)}`;
```

Example:

```text
1718850000-293847234
```

Then:

```ts
callback(
  null,
  `${uniqueSuffix}${extname(file.originalname)}`
);
```

Result:

```text
1718850000-293847234.jpg
```

Every uploaded file receives a unique name.

---

# callback()

Multer asks:

```text
I have received a file.
What name should I save it with?
```

We answer:

```ts
callback(
  null,
  generatedFileName
);
```

The first argument:

```ts
null
```

means:

No error occurred.

The second argument:

```ts
generatedFileName
```

is the filename Multer should use.

---

# Saving the File

Multer automatically writes:

```text
uploads/profiles/1718850000-293847234.jpg
```

onto disk.

At this point the image is already stored.

---

# Controller

```ts
uploadPicture(
  @Req() req,
  @UploadedFile() file
)
```

After FileInterceptor finishes:

```ts
file
```

contains information about the uploaded file.

Example:

```ts
{
  filename:
    '1718850000-293847234.jpg',
  mimetype:
    'image/jpeg'
}
```

---

# Validation

```ts
if (!file?.filename)
```

Protects against:

```text
POST /profile/picture
```

without selecting a file.

Returns:

```json
{
  "message":
  "Profile picture file is required"
}
```

---

# Calling the Service

```ts
return this.profileService
  .uploadPicture(
    req.user.userId,
    file,
  );
```

Example:

```ts
uploadPicture(
  '3f28941e',
  {
    filename:
      '1718850000.jpg'
  }
)
```

---

# Service Layer

## Step 1

Find profile:

```ts
const profile =
await this.prisma.profile.findUnique({
  where: {
    userId,
  },
});
```

Reason:

```text
User may exist
Profile may not exist
```

---

## Step 2

Generate URL:

```ts
`/uploads/profiles/${file.filename}`
```

Example:

```text
/uploads/profiles/1718850000.jpg
```

---

# Why store URLs instead of images?

Bad:

```text
Database
      ↓
Entire image binary
```

Problems:

* Large database size
* Slow backups
* Poor scalability

Better:

```text
Database
      ↓
String URL
```

Example:

```text
/uploads/profiles/1718850000.jpg
```

Database stays lightweight.

---

# Updating Database

```ts
return this.prisma.profile.update({
  where: {
    userId,
  },
  data: {
    profilePicture:
      `/uploads/profiles/${file.filename}`,
  },
});
```

Profile table:

Before:

```text
profilePicture = null
```

After:

```text
profilePicture =
/uploads/profiles/1718850000.jpg
```

---

# ServeStaticModule

Previously configured:

```ts
ServeStaticModule.forRoot({
  rootPath:
    join(process.cwd(), 'uploads'),
  serveRoot:
    '/uploads',
})
```

This maps:

```text
/uploads
```

to:

```text
project/uploads
```

---

# Browser Access

Database:

```text
/uploads/profiles/1718850000.jpg
```

Browser:

```text
http://localhost:3000/uploads/profiles/1718850000.jpg
```

Nest automatically serves the image.

---

# Final Architecture

```text
Flutter
      ↓
Choose Image
      ↓
POST /profile/picture
      ↓
JwtAuthGuard
      ↓
FileInterceptor
      ↓
Multer
      ↓
diskStorage
      ↓
uploads/profiles
      ↓
Controller
      ↓
ProfileService
      ↓
Prisma
      ↓
Database stores image URL
      ↓
Browser can access image
```

---

# Current Status

✅ Local file storage working

✅ Profile image upload endpoint working

✅ JWT protection enabled

✅ Database stores image URLs

✅ Images accessible through browser

⏳ Future migration to AWS S3
