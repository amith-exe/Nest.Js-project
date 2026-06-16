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
main.ts : this is the start the server at port 3000 
 ```import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```




