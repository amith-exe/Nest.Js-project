# Google OAuth Setup in NestJS (Book My Venue)

## Goal

Allow users to sign in using their Google account.

Flow:

```text
Client
   ↓
GET /auth/google
   ↓
Google Login Screen
   ↓
Google redirects to callback
   ↓
NestJS receives Google profile
   ↓
Find/Create User
   ↓
Generate JWT
   ↓
Return JWT
```

---

# Architecture

```text
GoogleAuthGuard
        ↓
GoogleStrategy
        ↓
validate()
        ↓
req.user
        ↓
Controller
        ↓
AuthService
        ↓
JWT
```

---

# Step 1: Install Packages

Install Google Passport strategy.

```bash
npm install passport-google-oauth20
npm install -D @types/passport-google-oauth20
```

Required packages:

| Package                        | Purpose               |
| ------------------------------ | --------------------- |
| passport-google-oauth20        | Google OAuth Strategy |
| @types/passport-google-oauth20 | TypeScript typings    |

---

# Step 2: Create Google Cloud Project

Go to:

https://console.cloud.google.com

### Create Project

Example:

```text
Book My Venue
```

---

# Step 3: Configure OAuth Consent Screen

Navigate to:

```text
APIs & Services
        ↓
OAuth Consent Screen
```

Choose:

```text
External
```

Fill:

```text
App Name:
Book My Venue

User Support Email:
your email

Developer Contact:
your email
```

Save.

---

# Step 4: Create OAuth Credentials

Navigate:

```text
APIs & Services
       ↓
Credentials
       ↓
Create Credentials
       ↓
OAuth Client ID
```

Application Type:

```text
Web Application
```

Name:

```text
Book My Venue Local
```

---

# Step 5: Configure URLs

## Authorized JavaScript Origins

```text
http://localhost:3000
```

Do NOT put:

```text
http://localhost:3000/auth/google/callback
```

Origins cannot contain paths.

---

## Authorized Redirect URIs

```text
http://localhost:3000/auth/google/callback
```

This is where Google redirects after login.

---

# Step 6: Obtain Credentials

Google provides:

```text
Client ID
Client Secret
```

---

# Step 7: Environment Variables

.env

```env
GOOGLE_CLIENT_ID=xxxxxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxxxxxxx
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

Never commit these values to GitHub.

---

# Project Structure

```text
src
├── auth
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.module.ts
│
├── strategy
│   └── google.strategy.ts
│
├── guard
│   └── google.guard.ts
```

---

# Step 8: Create Google Strategy

File:

```text
src/strategy/google.strategy.ts
```

```ts
@Injectable()
export class GoogleStrategy
  extends PassportStrategy(
    Strategy,
    'google',
  ) {
  constructor(
    private readonly configService:
      ConfigService,
  ) {
    super({
      clientID:
        configService.get<string>(
          'GOOGLE_CLIENT_ID',
        )!,
      clientSecret:
        configService.get<string>(
          'GOOGLE_CLIENT_SECRET',
        )!,
      callbackURL:
        configService.get<string>(
          'GOOGLE_CALLBACK_URL',
        )!,
      scope: ['email', 'profile'],
    });
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: Profile,
  ) {
    return profile;
  }
}
```

---

# Understanding super()

```ts
super({
  clientID,
  clientSecret,
  callbackURL,
  scope,
});
```

Configures the Google strategy.

---

## clientID

Identifies your application.

---

## clientSecret

Proves to Google that your backend application is genuine.

---

## callbackURL

Where Google redirects after successful login.

Example:

```text
http://localhost:3000/auth/google/callback
```

---

## scope

```ts
scope: ['email', 'profile']
```

Requests:

* Email
* Name
* Profile Picture

---

# Why PassportStrategy(Strategy, 'google') ?

```ts
PassportStrategy(
  Strategy,
  'google',
)
```

Registers a strategy named:

```text
google
```

This connects to:

```ts
AuthGuard('google')
```

The names must match.

---

# Step 9: validate()

Passport automatically calls:

```ts
validate(
  accessToken,
  refreshToken,
  profile,
)
```

Google provides:

```text
profile.id
profile.displayName
profile.emails
profile.photos
profile.name
```

Example:

```text
profile.id
↓
114138330886936974286

profile.emails[0].value
↓
user@gmail.com

profile.displayName
↓
Amith Biju

profile.photos[0].value
↓
https://....
```

Anything returned from validate():

```ts
return profile;
```

becomes:

```ts
req.user
```

inside the controller.

---

# Step 10: Create Guard

File:

```text
src/guard/google.guard.ts
```

```ts
@Injectable()
export class GoogleAuthGuard
  extends AuthGuard('google') {}
```

---

# Why Do We Need a Guard?

The guard says:

```text
Use the strategy named "google"
```

Flow:

```text
GoogleAuthGuard
        ↓
AuthGuard('google')
        ↓
GoogleStrategy
```

---

# Step 11: Register Strategy

auth.module.ts

```ts
providers: [
  AuthService,
  JwtStrategy,
  GoogleStrategy,
]
```

---

# Step 12: Create Login Route

```ts
@Get('google')
@UseGuards(GoogleAuthGuard)
googleLogin() {}
```

No method body is required.

Flow:

```text
Request
      ↓
GoogleAuthGuard
      ↓
Passport
      ↓
Redirect to Google
```

---

# Step 13: Create Callback Route

```ts
@Get('google/callback')
@UseGuards(GoogleAuthGuard)
googleCallback(
  @Req() req,
) {
  return req.user;
}
```

Flow:

```text
Google
     ↓
/auth/google/callback
     ↓
validate()
     ↓
return profile
     ↓
req.user
     ↓
Controller
```

---

# Complete Flow

```text
GET /auth/google
        ↓
Google Login Screen
        ↓
User logs in
        ↓
Google redirects
        ↓
GET /auth/google/callback
        ↓
GoogleStrategy.validate()
        ↓
return profile
        ↓
req.user
        ↓
Controller
```

---

# Current Response

Example:

```json
{
  "userId": "114138330886936974286",
  "email": "user@gmail.com",
  "name": "Amith Biju"
}
```

---

# Production Flow

Instead of returning req.user:

```text
Google Login
      ↓
Find User by Email
      ↓
User Exists?
      ↓
No
      ↓
Create User
      ↓
Generate JWT
      ↓
Return JWT
```

---

# Future Integration with Profile Module

Google already provides:

```text
Email
Name
Profile Picture
```

Later:

```text
Google
   ↓
User
   ↓
Profile
```

Profile can automatically be populated with:

```text
firstName
lastName
profilePicture
```

---

# Mental Model

```text
GoogleAuthGuard
      ↓
Asks Passport to use Google strategy

GoogleStrategy
      ↓
Communicates with Google

validate()
      ↓
Processes Google profile

return value
      ↓
req.user

Controller
      ↓
Creates user/JWT response
```
