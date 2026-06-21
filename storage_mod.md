# Storage Module Setup - BookMyVenue

## Purpose

The Storage Module is responsible for handling all file-related operations in the application.

Examples:

* Profile pictures
* Venue images
* Future documents (licenses, invoices, certificates, etc.)

The goal is to keep file storage logic separate from business logic.

---

# Why Create a Separate Storage Module?

Without a Storage Module:

```text
ProfileService
    ↓
File Upload Logic

VenueService
    ↓
File Upload Logic
```

The same upload code would be duplicated across multiple modules.

With a Storage Module:

```text
ProfileService
        ↓
StorageService
        ↓
File System

VenueService
        ↓
StorageService
        ↓
File System
```

All upload functionality exists in one place.

This follows the Single Responsibility Principle (SRP).

---

# Current Storage Strategy

For the MVP, files are stored locally inside the project directory.

We are NOT using:

* AWS S3
* Cloudinary
* Google Drive

yet.

The application stores files directly on the server's file system.

---

# Project Structure

```text
bmv-main
├── src
│   ├── auth
│   ├── profile
│   ├── venue
│   └── storage
├── prisma
├── uploads
│   ├── profiles
│   ├── venues
│   └── documents
├── package.json
└── tsconfig.json
```

---

# Why is uploads outside src?

The `src` directory should contain only source code:

* Controllers
* Services
* Modules
* DTOs
* Entities

The uploads folder contains runtime files:

* Images
* PDFs
* Documents

These files are data, not source code.

Keeping uploads outside src also avoids issues after TypeScript compilation.

---

# Generated Storage Module

Commands:

```bash
nest g module storage
nest g service storage
```

Generated structure:

```text
src
└── storage
    ├── storage.module.ts
    └── storage.service.ts
```

No controller was created because uploads will be triggered from other modules.

---

# Why No Storage Controller?

Uploads naturally belong to their respective domains.

Examples:

```text
POST /profile/picture
POST /venues/:id/images
```

These endpoints are more meaningful than:

```text
POST /storage/upload
```

Therefore:

ProfileController
→ StorageService

VenueController
→ StorageService

---

# Upload Directories

Created folders:

```text
uploads
├── profiles
├── venues
└── documents
```

Purpose of each directory:

profiles/

* User profile pictures

venues/

* Venue images

documents/

* Future business documents

Examples:

```text
uploads/profiles/user123.jpg
uploads/venues/venue1/image1.jpg
uploads/documents/license.pdf
```

---

# Serving Static Files

Installed:

```bash
npm install @nestjs/serve-static
```

Configured inside AppModule:

```ts
ServeStaticModule.forRoot({
  rootPath: join(process.cwd(), 'uploads'),
  serveRoot: '/uploads',
})
```

---

# Why process.cwd()?

Initial attempt:

```ts
join(__dirname, '..', 'uploads')
```

Result:

```text
ENOENT:
dist/uploads/index.html
```

Nest was looking for the uploads folder inside the compiled dist directory.

Solution:

```ts
join(process.cwd(), 'uploads')
```

process.cwd() returns:

```text
C:\Users\amith\bmv\BookMyVenue\bmv-main
```

Therefore:

```text
C:\Users\amith\bmv\BookMyVenue\bmv-main\uploads
```

which is exactly where our uploads folder exists.

---

# How Static Serving Works

File:

```text
uploads/profiles/test.txt
```

becomes accessible through:

```text
http://localhost:3000/uploads/profiles/test.txt
```

Nest automatically serves files from the uploads directory.

---

# Current Storage Architecture

```text
Flutter
      ↓
NestJS API
      ↓
StorageService
      ↓
uploads/
      ├── profiles
      ├── venues
      └── documents
```

---

# Planned StorageService Responsibilities

Current skeleton:

```ts
@Injectable()
export class StorageService {
  async saveProfilePicture() {}

  async saveVenueImages() {}

  async deleteFile() {}
}
```

Responsibilities:

saveProfilePicture()

* Save profile images

saveVenueImages()

* Save venue images

deleteFile()

* Remove old files

---

# Future Profile Picture Flow

```text
Flutter
      ↓
Choose Image
      ↓
POST /profile/picture
      ↓
ProfileController
      ↓
ProfileService
      ↓
StorageService
      ↓
uploads/profiles
      ↓
Generated URL
      ↓
Update Profile.profilePicture
```

---

# Future Venue Image Flow

```text
Flutter
      ↓
Choose Images
      ↓
POST /venues/:id/images
      ↓
VenueController
      ↓
VenueService
      ↓
StorageService
      ↓
uploads/venues
      ↓
Store URLs in database
```

---

# Database Design

Profile:

```prisma
profilePicture String?
```

Examples:

```text
/uploads/profiles/user123.jpg
```

or

```text
http://localhost:3000/uploads/profiles/user123.jpg
```

Venue images will follow the same approach.

---

# Future Migration to AWS S3

One reason for storing only URLs in the database:

Current:

```text
/uploads/profiles/user123.jpg
```

Future:

```text
https://bucket.s3.amazonaws.com/profiles/user123.jpg
```

No database schema changes are required.

Only StorageService implementation changes.

---

# Final Architecture

```text
AuthModule
ProfileModule
VenueModule
StorageModule
BookingModule
```

StorageModule becomes a reusable infrastructure module used by every feature that needs file handling.

---

# Current Status

✅ StorageModule created

✅ StorageService created

✅ uploads folder created

✅ Static file serving configured

✅ Local file storage working

⏳ Profile picture upload endpoint pending

⏳ Venue image upload endpoint pending

⏳ AWS S3 integration postponed until later stages of development
