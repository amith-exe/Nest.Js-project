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

```npm install -g @nestjs/cli```

## starting new project 
```nest new my-app```
->will create the following files

````
my-app/
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



