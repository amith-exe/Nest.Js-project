# What is NestJS?

NestJS is a progressive Node.js framework that uses:

TypeScript by default
Dependency Injection (DI)
Modular architecture
Decorators
Built-in support for APIs, WebSockets, GraphQL, microservices,

It is built on top of Express (by default) and deeply utilizes TypeScript to provide strong typing and better code quality.

It adopts design patterns from Angular. This means it uses things like Decorators (@Controller(), @Injectable())

# Install Nest CLI
for using the nest in our project

```npm install -g @nestjs/cli```
 
## starting new project 
```nest new my-app
  cd my-app
```
->will create the following files
then go inside my-app dir to start working with our project/app

```my-app/
│
├── src/
│   ├── app.controller.ts
│   ├── app.service.ts
│   ├── app.module.ts
│   └── main.ts
│
├── test/
├── package.json
├── tsconfig.json
├── nest-cli.json
└── README.md
```
## Details

```main.ts          → Starts application
app.module.ts    → Organizes application
app.controller.ts→ Receives requests
app.service.ts   → Business logic

test/            → Testing
package.json     → Project blueprint
node_modules/    → Installed libraries
nest-cli.json    → Nest CLI settings
tsconfig.json    → TypeScript settings
eslint.config.mjs→ Code quality rules
.prettierrc      → Code formatting rules
.gitignore       → Files Git should ignore
README.md        → Documentation
```
* detalis of other files
``` .prettierrc

Configuration for Prettier.

Prettier automatically formats code.

Example:

Before:

const user={name:"John"}

After:

const user = { name: 'John' };

Keeps code clean and consistent.

6. eslint.config.mjs

Configuration for ESLint.

ESLint finds problems.

Example:

Bad:

const a = 10;
const b = 20;

If b is never used:

'b' is assigned a value but never used

It helps keep code quality high.

7. nest-cli.json

Configuration for Nest CLI.

Example:

{
  "sourceRoot": "src"
}

When you run:

nest generate module users

Nest CLI reads this file to know where to create files.

8. package.json

Probably the most important configuration file.

Contains:

Project information
{
  "name": "my-app",
  "version": "0.0.1"
}
Scripts
"scripts": {
  "start": "nest start",
  "start:dev": "nest start --watch",
  "build": "nest build",
  "test": "jest"
}

You execute them with:

npm run start
npm run start:dev
npm run build
npm run test
Dependencies
"dependencies": {
  "@nestjs/common": "...",
  "@nestjs/core": "...",
  "rxjs": "..."
}

Think of package.json as:

Project Blueprint
9. package-lock.json

Generated automatically by npm.

Stores exact package versions.

Example:

express -> 5.1.0
rxjs -> 7.8.1

Ensures everyone on the team installs the same versions.

Usually:

✅ Commit it to Git

❌ Don't edit manually.
```



* main.ts : this is the start the server at port 3000 
 ```import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```
* app.controller.ts:
  Controllers are responsible for request handling and response sending, acting as a gateway between the client and server. This file currently contains a single route that handles your GET request and returns "Hello World!".

  current thing in this file

``` import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';  // it is called from the app.service file that contain the hello word

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```
here basicly all the @get @ post @delete @ put thing are handelled  so it listen for the request from the user 

and then call the service according to the request and like this we can create multiple conroller for each modules and the here the app module is the main moduel of this whole app 

![alt text](image.png)

so this is an example of simple spotify like app module details 

* app.service.ts : Services encapsulate your business logic. If you look at this file, you'll see the function that actually returns the "Hello World!" string. The controller calls this service, promoting "Separation of Concerns".

means like the main logic od the service provided by app is wrriten in this file in intail it just shows hello word 

to saw this we need to start our app after intializing thing ```npm run start:dev```

after intailzng with ```npm new project_name``` we will get files like this 
![alt text](image-1.png)

* test folder is for testing our app dont need to look that now 

* in src/ the controller.spec.ts is for testing app moduel sperately

# Modules 

![alt text](image-2.png)


here in this each featuer is module like 
auth module , user moduel , login moduel like that eah thing has spearte controller ,service and modules 

-> everthing is conrolled  by app moduel the whole project 

### Creating other featuers 

creating user module use this command ```nest g co songs --no-spec```

* co stands for controller. Adding --no-spec tells the CLI not to create the test file, keeping our folder clean just like you wanted

then add some code in it 

* Open src/songs/songs.controller.ts. Let's add some routes for our API:

```import { Controller, Get, Post, Delete, Put } from '@nestjs/common';

@Controller('songs') // This means all routes inside this controller start with /songs
export class SongsController {

  @Post()
  createSong() {
    return 'This action adds a new song';
  }

  @Get()
  findAllSongs() {
    return 'This action returns all songs';
  }

  @Get(':id')
  findOneSong() {
    return 'This action returns a single song based on ID';
  }

  @Put(':id')
  updateSong() {
    return 'This action updates a song';
  }

  @Delete(':id')
  deleteSong() {
    return 'This action deletes a song';
  }
}
```
* then creating service ```nest g s songs --no-spec```
 s for service 

 ```import { Injectable } from '@nestjs/common';

@Injectable() // This decorator makes it a provider that can be injected into controllers
export class SongsService {
  // Temporary local array data
  private readonly songs: string[] = ['Bohemian Rhapsody', 'Hotel California', 'Blinding Lights'];

  create(song: string) {
    this.songs.push(song);
    return `Song "${song}" added successfully!`;
  }

  findAll() {
    return this.songs;
  }
}
```
---> connect the controller and the service by going to controller
* Go back to src/songs/songs.controller.ts and update it to look like this:

```import { Controller, Get, Post, Body } from '@nestjs/common';
import { SongsService } from './songs.service'; // 1. Import the service

@Controller('songs')
export class SongsController {
  // 2. Inject the service via the constructor
  constructor(private songsService: SongsService) {}

  @Post()
  createSong(@Body('title') title: string) { // @Body extracts the payload from the request
    return this.songsService.create(title);   // 3. Call service method
  }

  @Get()
  findAllSongs() {
    return this.songsService.findAll();       // 3. Call service method
  }
}
```
## Insted of this we  create Generate a complete resource (Module + Controller + Service + DTO)

 using --> ```nest g resource user``` or songs 

```It will ask:

? What transport layer do you use?
❯ REST API

Then:

? Would you like to generate CRUD entry points?
❯ Yes
```

then it will create file like this

```src/user/
├── dto/
│   ├── create-user.dto.ts
│   └── update-user.dto.ts
├── entities/
│   └── user.entity.ts
├── users.controller.ts
├── users.service.ts
├── users.module.ts
└── users.controller.spec.ts
└── users.service.spec.ts
```
# how we can test this API ?

here we can use rest client in vs code 

install it and crate a file with ```.http``` extension and type all the end points to test in vs code

#### like this 
```### Get all songs
GET http://localhost:3000/songs

### Add a new song
POST http://localhost:3000/songs
Content-Type: application/json

{
  "title": "Starboy"
} 

### Get a single song by ID (Testing the route)
GET http://localhost:3000/songs/1
```
# DTO (Data Transfer Object)

for this wee need to install ```npm install class-validator class-transformer```

after installing we should tell src/main.ts  applay it to all of the project like this

```import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common'; // 1. Import ValidationPipe

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(new ValidationPipe()); // 2. Turn on automatic validation!
  
  await app.listen(3000);
}
bootstrap();
```
* then we should create  file for creating dto validating class

src/songs/dto/create-song.dto.ts
```import { IsString, IsNotEmpty, IsArray, IsDateString } from 'class-validator';

export class CreateSongDto {
  @IsString()
  @IsNotEmpty()
  readonly title: string;

  @IsArray()
  @IsString({ each: true }) // Every item inside the array must be a string
  readonly artists: string[];

  @IsDateString()
  readonly releaseDate: string;
}
```

update the controller to use the class validator

```import { Controller, Post, Body } from '@nestjs/common';
import { CreateSongDto } from './dto/create-song.dto'; // Import the DTO
import { SongsService } from './songs.service';

@Controller('songs')
export class SongsController {
  constructor(private songsService: SongsService) {}

  @Post()
  createSong(@Body() createSongDto: CreateSongDto) { // Enforce the DTO validation rules
    return this.songsService.create(createSongDto.title);
  }
}

```import { Injectable } from '@nestjs/common';

@Injectable()
export class SongsService {
  private readonly songs: string[] = []; // Your temporary array

  create_song(title: string) { // 👈 Must match the name called in controller
    this.songs.push(title);
    return `Song "${title}" added successfully!`;
  }
}
```
* so there will be comfussion in the name of function here the names are same but they do diff thing becuse there in diff class one in controller and other in service

* @Boday()
used with POST
it will help as to extarct boady of the request from the user

-> example
request from user 
```POST /auth/login
Content-Type: application/json

{
  "email": "john@gmail.com",
  "password": "123456"
}
```
-> the controller takes the request 

```@Post('login')
login(@Body() loginDto: LoginDto) {
  return loginDto;
}
```
->then we typescripte will convert that json to this formate

```const loginDto = {
  email: 'john@gmail.com',
  password: '123456',
};
```
->with out dto object we can use type any
we can also exctart only oe thing from the boady like this
```ogin(@Body(email) boday_email: type) {
  return lboday_email
}
```
here we will take only the email from the response


* Param()
 it is used in GET request to extract content from url or used with PUT
GET /users/15
 ```@Get(':id')
findOne(@Param('id') id: string) {
  return this.usersService.findOne(id);
}
```
this will take that 15 from the url by defult it is string we can also convert it to other type

# NestJS Request Decorators

## Rule of Thumb

| Decorator    | Gets Data From        | Example                        |
| ------------ | --------------------- | ------------------------------ |
| `@Body()`    | Request body (JSON)   | `email`, `password`, form data |
| `@Param()`   | URL path parameters   | `/users/15` → `15`             |
| `@Query()`   | Query string          | `/users?page=1` → `page=1`     |
| `@Headers()` | Request headers       | `Authorization` token          |
| `@Req()`     | Entire request object | Access everything              |

---

## Quick Memory Guide

```text
POST /users              -> @Body()
GET  /users/15           -> @Param('id')
GET  /users?page=2       -> @Query('page')
GET  /profile            -> @Headers('authorization')
GET  /anything           -> @Req()
```

---

## Examples

### `@Body()`

```ts
@Post()
create(@Body() dto: CreateUserDto) {
  return dto;
}
```

Request:

```json
{
  "name": "Amith",
  "email": "amith@gmail.com"
}
```

---

### `@Param()`

```ts
@Get(':id')
findOne(@Param('id') id: string) {
  return id;
}
```

Request:

```http
GET /users/15
```

Result:

```ts
id = "15";
```

---

### `@Query()`

```ts
@Get()
findAll(@Query('page') page: string) {
  return page;
}
```

Request:

```http
GET /users?page=2
```

Result:

```ts
page = "2";
```

---

### `@Headers()`

```ts
@Get()
getProfile(
  @Headers('authorization') token: string,
) {
  return token;
}
```

Request:

```http
Authorization: Bearer xyz123
```

Result:

```ts
token = "Bearer xyz123";
```

---

### `@Req()`

```ts
@Get()
getRequest(@Req() req: Request) {
  console.log(req.body);
  console.log(req.params);
  console.log(req.query);
  console.log(req.headers);
}
```

`@Req()` gives you access to the entire HTTP request object.

```
```
 # NestJS Providers and Dependency Injection (DI)

## What is a Provider?

A **Provider** is a class that NestJS can create and manage for you.

Example:

```ts
@Injectable()
export class AuthService {
  login() {
    return 'Logged in';
  }
}
```

`AuthService` is a provider.

---

## Why Do We Need Providers?

### Without Providers

```ts
@Controller('auth')
export class AuthController {
  private authService = new AuthService();

  @Post('login')
  login() {
    return this.authService.login();
  }
}
```

Problems:

* Controller creates the service itself.
* Tight coupling between classes.
* Difficult to test.
* Difficult to replace implementations.

---

### With Providers

```ts
@Controller('auth')
export class AuthController {
  constructor(
    private readonly authService: AuthService,
  ) {}

  @Post('login')
  login() {
    return this.authService.login();
  }
}
```

You never do this:

```ts
new AuthService();
```

NestJS creates and injects the service for you.

---

## How NestJS Creates a Provider

### Step 1: Mark the Class as Injectable

```ts
@Injectable()
export class AuthService {}
```

This tells Nest:

> "This class can participate in Dependency Injection."

---

### Step 2: Register the Provider

```ts
@Module({
  providers: [AuthService],
})
export class AuthModule {}
```

This tells Nest:

> "Manage an instance of AuthService."

---

### Step 3: Inject the Provider

```ts
constructor(
  private readonly authService: AuthService,
) {}
```

Internally, Nest does something similar to:

```ts
const service = new AuthService();
const controller = new AuthController(service);
```

You never write this code yourself.

---

## Provider Creation Flow

```text
AuthModule
    │
    └── providers: [AuthService]
                  │
                  ▼
          Nest creates AuthService
                  │
                  ▼
      Injects it into AuthController
                  │
                  ▼
        Controller can use it
```

---

## Real Login Example

### Provider

```ts
@Injectable()
export class AuthService {
  login(email: string) {
    return `Welcome ${email}`;
  }
}
```

### Controller

```ts
@Controller('auth')
export class AuthController {
  constructor(
    private readonly authService: AuthService,
  ) {}

  @Post('login')
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto.email);
  }
}
```

---

## Request Flow

```text
POST /auth/login
       │
       ▼
AuthController.login()
       │
       ▼
AuthService.login()
       │
       ▼
Response
```

---

## Providers Are Singleton by Default

Nest creates:

```ts
new AuthService();
```

only once.

Every request uses the same instance.

Example:

```ts
@Injectable()
export class CounterService {
  count = 0;

  increment() {
    this.count++;
    return this.count;
  }
}
```

Request 1:

```text
count = 1
```

Request 2:

```text
count = 2
```

Same object instance is reused.

---

## Dependency Injection (DI)

Example:

```ts
@Injectable()
export class UsersService {}

@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService,
  ) {}
}
```

Nest sees:

```text
AuthService
      │
      ▼
needs UsersService
```

So Nest creates:

```text
UsersService
      │
      ▼
AuthService
      │
      ▼
AuthController
```

This process is called **Dependency Injection (DI)**.

---

## Dependency Graph

```text
UsersService
      │
      ▼
AuthService
      │
      ▼
AuthController
```

Nest automatically builds this graph.

---

## Providers Are Not Only Services

Any injectable class can be a provider.

```ts
@Injectable()
export class EmailService {}

@Injectable()
export class JwtService {}

@Injectable()
export class CacheService {}
```

Register them:

```ts
@Module({
  providers: [
    AuthService,
    EmailService,
    JwtService,
    CacheService,
  ],
})
export class AuthModule {}
```

---

## Production Example

```text
AuthController
       │
       ▼
AuthService
       │
       ▼
UsersService
       │
       ▼
PrismaService
       │
       ▼
PostgreSQL
```

Nest creates and injects every dependency automatically.

---

## Mental Model

Think of NestJS as a company manager.

```text
NestJS (Manager)
      │
      ├── AuthService (Employee)
      ├── UsersService (Employee)
      └── EmailService (Employee)
```

Controller says:

```text
"I need AuthService."
```

Nest replies:

```text
"Here is the AuthService instance."
```

You never hire employees (`new`) yourself.

Nest hires them and gives them to whoever needs them.

---

## Rule of Thumb

```text
@Injectable()
      ↓
Marks a class as injectable.

providers: []
      ↓
Registers the class with Nest.

constructor(...)
      ↓
Asks Nest to inject dependencies.

Singleton
      ↓
One instance shared across the application.
```
# NestJS Custom Providers

## What is a Custom Provider?

Normally, Nest creates providers like this:

```ts
@Injectable()
export class AuthService {}
```

```ts
@Module({
  providers: [AuthService],
})
export class AuthModule {}
```

Nest internally does:

```ts
const authService = new AuthService();
```

---

## Why Custom Providers?

Sometimes you want to:

* Inject configuration values
* Inject API keys
* Use different implementations
* Create objects manually
* Create objects asynchronously
* Use interfaces or tokens

For these cases, Nest provides **Custom Providers**.

---

# 1. `useValue`

Inject a fixed value or object.

### Example

```ts
@Module({
  providers: [
    {
      provide: 'APP_NAME',
      useValue: 'Book My Venue',
    },
  ],
})
export class AppModule {}
```

Inject it:

```ts
@Injectable()
export class AppService {
  constructor(
    @Inject('APP_NAME')
    private readonly appName: string,
  ) {}

  getName() {
    return this.appName;
  }
}
```

Result:

```text
Book My Venue
```

---

## Another Example

```ts
{
  provide: 'CONFIG',
  useValue: {
    database: 'postgres',
    port: 5432,
  },
}
```

Inject:

```ts
constructor(
  @Inject('CONFIG')
  private readonly config: any,
) {}
```

---

# Flow

```text
Token: 'CONFIG'
       │
       ▼
{ database: 'postgres', port: 5432 }
       │
       ▼
Injected into service
```

---

# 2. `useClass`

Tell Nest which class to instantiate.

### Example

```ts
@Injectable()
export class DevelopmentLogger {
  log(message: string) {
    console.log('[DEV]', message);
  }
}

@Injectable()
export class ProductionLogger {
  log(message: string) {
    console.log('[PROD]', message);
  }
}
```

Provider:

```ts
{
  provide: 'LOGGER',
  useClass: DevelopmentLogger,
}
```

Inject:

```ts
constructor(
  @Inject('LOGGER')
  private readonly logger: DevelopmentLogger,
) {}
```

Nest does:

```ts
const logger = new DevelopmentLogger();
```

---

## Real Use Case

Development:

```ts
{
  provide: 'LOGGER',
  useClass: DevelopmentLogger,
}
```

Production:

```ts
{
  provide: 'LOGGER',
  useClass: ProductionLogger,
}
```

Your code never changes:

```ts
constructor(
  @Inject('LOGGER')
  private readonly logger: any,
) {}
```

---

# Flow

```text
Token: 'LOGGER'
       │
       ▼
DevelopmentLogger
       │
       ▼
Nest creates instance
       │
       ▼
Injected into service
```

---

# 3. `useFactory`

Create the provider using a function.

### Example

```ts
{
  provide: 'DATABASE_URL',
  useFactory: () => {
    return process.env.DATABASE_URL;
  },
}
```

Inject:

```ts
constructor(
  @Inject('DATABASE_URL')
  private readonly dbUrl: string,
) {}
```

---

## Factory Can Use Dependencies

```ts
{
  provide: 'APP_MESSAGE',
  useFactory: (
    configService: ConfigService,
  ) => {
    return `App: ${configService.get('APP_NAME')}`;
  },
  inject: [ConfigService],
}
```

Nest does:

```text
Create ConfigService
       │
       ▼
Run factory function
       │
       ▼
Return value
       │
       ▼
Inject result
```

---

## Real Example

```ts
{
  provide: 'DB_CONNECTION',
  useFactory: () => {
    return new PrismaClient();
  },
}
```

Nest creates:

```ts
const db = new PrismaClient();
```

and injects it wherever needed.

---

# 4. `useExisting`

Create an alias for another provider.

```ts
@Injectable()
export class LoggerService {}
```

Provider:

```ts
providers: [
  LoggerService,
  {
    provide: 'LOGGER',
    useExisting: LoggerService,
  },
]
```

Inject:

```ts
constructor(
  @Inject('LOGGER')
  private readonly logger: LoggerService,
) {}
```

Nest does:

```ts
const logger = new LoggerService();
```

Both tokens point to the same object.

---

# Flow

```text
LoggerService Instance
      ▲
      │
'LOGGER' Token
      │
      ▼
Same Object
```

---

# Provider Tokens

Notice:

```ts
provide: 'LOGGER'
provide: 'CONFIG'
provide: 'DATABASE_URL'
```

These are called **Injection Tokens**.

Think:

```text
Token → Provider
```

Examples:

```text
'LOGGER'        → DevelopmentLogger
'CONFIG'        → Configuration Object
'DATABASE_URL'  → Connection String
```

---

# Real Production Example

```text
AuthController
       │
       ▼
AuthService
       │
       ▼
@Inject('LOGGER')
       │
       ▼
ProductionLogger
```

---

# Which One Should I Use?

| Provider Type | Use Case                                           |
| ------------- | -------------------------------------------------- |
| `useValue`    | Constants, config objects, API keys                |
| `useClass`    | Switch implementations                             |
| `useFactory`  | Complex initialization, environment-based creation |
| `useExisting` | Alias another provider                             |

---

# Rule of Thumb

```text
useValue
    ↓
Inject a fixed value

useClass
    ↓
Inject a class instance

useFactory
    ↓
Inject something created by a function

useExisting
    ↓
Reuse another provider's instance
```

---

# Mental Model

```text
Nest Container
      │
      ├── 'CONFIG'
      │      ↓
      │   useValue
      │
      ├── 'LOGGER'
      │      ↓
      │   useClass
      │
      ├── 'DB_CONNECTION'
      │      ↓
      │   useFactory
      │
      └── 'LOGGER_ALIAS'
             ↓
         useExisting
```

**Most common in real projects:**

1. `useFactory` → Database connections, JWT configuration, environment variables.
2. `useValue` → Constants and configuration objects.
3. `useClass` → Different implementations for development and production.
4. `useExisting` → Aliasing and reusing existing providers.

