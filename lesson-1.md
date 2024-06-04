# 1. Set up Prisma, PostgreSQL and NestJS

## Goal

The goal of this lesson is to set up and integrate NestJS and Prisma - Setting up NestJS, Prisma and PostgreSQL, generating a Nest Module and Service for `PrismaClient` and using it via [Dependency Injection (DI)](https://docs.nestjs.com/providers#dependency-injection) for a REST API.

## Setup

The first thing you will do is set up a NestJS project using the NestJS CLI. To start, run the following command in the location where you want the project to reside: 

```bash
npx @nestjs/cli new median
```

The CLI will prompt you to choose a *package manager* for your project — choose **npm**. Afterward, you should have a new NestJS project in the current directory.

- **Alternative:** Clone the starter project from Github. **
    
    ```bash
    git clone -b lesson-1-begin git@github.com:prisma/nestjs-workshop-prisma-day-22.git
    ```
    

Open the project in your preferred code editor (we recommend VSCode). You should see the following files:

```
median
  ├── node_modules
  ├── src
  │   ├── app.controller.spec.ts
  │   ├── app.controller.ts
  │   ├── app.module.ts
  │   ├── app.service.ts
  │   └── main.ts
  ├── test
  │   ├── app.e2e-spec.ts
  │   └── jest-e2e.json
  ├── README.md
  ├── nest-cli.json
  ├── package-lock.json
  ├── package.json
  ├── tsconfig.build.json
  └── tsconfig.json
```

Most of the code you work on will reside in the `src` directory. The NestJS CLI has already created a few files for you. Some of the notable ones are:

- `src/app.module.ts`: The root module of the application.
- `src/app.controller.ts`: A basic controller with a single route: `/`. This route will return a simple `'Hello World!'` message.
- `src/main.ts`: The entry point of the application. It will start the NestJS application

You can start your project by using the following command:

```bash
npm run start:dev
```

This command will watch your files, automatically recompiling and reloading the server whenever you make a change. To verify the server is running, go to the URL [`http://localhost:3000/`](http://localhost:3000/). You should see an empty page with the message `'Hello World!'`.

<aside>
💡 Note: You should keep the server running in the background as you go through this workshop.

</aside>

## Tasks

After you cloned the repo and installed the dependencies, you are ready to get started with the tasks for the first lesson.

### Task 1: Create a PostgreSQL instance

You will be using PostgreSQL as the database for your NestJS application. You will install and run PostgreSQL on your machine through a Docker container.

<aside>
💡 Note: If you don't want to use Docker, you can set up a [PostgreSQL instance natively](https://www.prisma.io/dataguide/postgresql/setting-up-a-local-postgresql-database) or get a [hosted PostgreSQL database on Heroku.](https://dev.to/prisma/how-to-setup-a-free-postgresql-database-on-heroku-1dc1) **This will not be covered during the workshop.**

</aside>

<aside>
❗ If there is no way you can get a PostgreSQL instance at the moment, you can also use [SQLite](https://www.sqlite.org/index.html) for this workshop. Alternative code snippets for SQLite will be provided when appropriate.

</aside>

First, create a `docker-compose.yml` file in the main folder of your project: 

```bash
touch docker-compose.yml
```

Inside the file create the following configuration:

```docker
# docker-compose.yml

version: '3.8'
services:

  # Docker connection string: postgres://postgres:postgres@localhost:5432/

  postgres:
    image: postgres:13.5
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - '5432:5432'

volumes:
  postgres:
```

A few things to understand about this configuration:

- The `image` option defines what Docker image to use. Here, you are using the [`postgres` image](https://hub.docker.com/_/postgres) version 13.5.
- The `environment` option specifies the environment variables passed to the container during initialization. You can define the configuration options and secrets – such as the username and password – the container will use here.
- The `volumes` option is used for persisting data in the host file system.
- The `ports` option maps ports from the host machine to the container. The format follows a `'host_port:container_port'` convention. In this case, you are mapping the port `5432` of the host machine to port `5432` of the `postgres` container. `5432` is conventionally the port used by PostgreSQL.

To start the `postgres` container, open a new terminal window and run the following command in the main folder of your project:

```bash
docker-compose up
```

If everything worked correctly, the new terminal window should show logs that the database system is ready to accept connections. You should see logs similar to the following inside the terminal window:

```
...
postgres_1  | 2022-03-05 12:47:02.410 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres_1  | 2022-03-05 12:47:02.410 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres_1  | 2022-03-05 12:47:02.411 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres_1  | 2022-03-05 12:47:02.419 UTC [1] LOG:  database system is ready to accept connections
```

Congratulations 🎉. You now have your own PostgreSQL database to play around with! 

### Task 2: Set up Prisma

Start by adding Prisma to the Nest project. First install the `prisma` CLI as a development dependency, initialize the Prisma schema and install the `@prisma/client` package.

- **Terminal command**
    
    ```bash
    npm install prisma --save-dev
    npx prisma init
    ```
    

This will create a new `prisma` directory with a `schema.prisma` file. This is the main configuration file that contains your database schema. This command also creates a `.env` file inside your project.

Inside the `.env` file, you should see a `DATABASE_URL` environment variable with a dummy connection string. Replace this connection string with the one for your PostgreSQL instance.

```bash
// .env
DATABASE_URL="postgres://postgres:postgres@localhost:5432/median-db"
```

- **Alternative code for SQLite**
    
    ```bash
    // .env
    DATABASE_URL="file:./dev.db"
    ```
    

<aside>
💡 **Note**: If you didn't use docker (as shown in the previous section) to create your PostgreSQL database, your connection string will be different from the one shown above. The connection string format for PostgreSQL is available in the [Prisma Docs](https://www.prisma.io/docs/concepts/database-connectors/postgresql#connection-url).

</aside>

If you open `prisma/schema.prisma`, you should see the following default schema:

```jsx
// prisma/schema.prisma

// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

This file is written in the *Prisma Schema Language*, which is a language that Prisma uses to define your database schema. The `schema.prisma` file has three main components:

- **Data source**: Specifies your database connection. The above configuration means that your database *provider* is PostgreSQL and the database connection string is available in the `DATABASE_URL` environment variable.
- **Generator**: Indicates that you want to generate Prisma Client, a type-safe query builder for your database. It is used to send queries to your database.
- **Data model**: Defines your database *models*. Each model will be mapped to a table in the underlying database. Right now there are no models in your schema, you will explore this part in the next section.

<aside>
ℹ️ The [Prisma VSCode extension](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma) adds some really nice IntelliSense and syntax highlighting for Prisma.

</aside>

### Task 2: Add Prisma model and run your first migration

Prisma schema also contains all your [*Prisma models*](https://www.prisma.io/docs/concepts/components/prisma-schema/data-model) representing your database tables. For this workshop, you will only need an `Article` model to represent each article on the blog.

Inside the `prisma/prisma.schema` file, add a new model to your schema named `Article`:

- **Solution**
    
    ```jsx
    // prisma/schema.prisma
    
    generator client {
      provider = "prisma-client-js"
    }
    
    datasource db {
      provider = "postgresql"
      url      = env("DATABASE_URL")
    }
    
    model Article {
      id          Int      @id @default(autoincrement())
      title       String   @unique
      description String?
      body        String
      published   Boolean  @default(false)
      createdAt   DateTime @default(now())
      updatedAt   DateTime @updatedAt
    }
    ```
    

Here, you have created an `Article` model with several fields. Each field has a name (`id`, `title`, etc.), a type (`Int`, `String`, etc.), and other optional attributes (`@id`, `@unique`, etc.). Fields can be made optional by adding a `?` after the field type.

The `id` field has a special attribute called `@id`. This attribute indicates that this field is the primary key of the model. The `@default(autoincrement())` attribute indicates that this field should be automatically incremented and assigned to any newly created record.

The `published` field is a flag to indicate whether an article is published or in draft mode. The `@default(false)` attribute indicates that this field should be set to `false` by default.

The two `DateTime` fields, `createdAt` and `updatedAt`, will track when an article is created and when it was last updated. The `@updatedAt` attribute will automatically update the field with the current timestamp whenever an article is modified.

- **Alternative code for SQLite**
    
    ```jsx
    // prisma/schema.prisma
    
    generator client {
      provider = "prisma-client-js"
    }
    
    datasource db {
      provider = "sqlite"
      url      = env("DATABASE_URL")
    }
    
    model Article {
      id          Int      @id @default(autoincrement())
      title       String   @unique
      description String?
      body        String
      published   Boolean  @default(false)
      createdAt   DateTime @default(now())
      updatedAt   DateTime @updatedAt
    }
    ```
    

With the Prisma schema defined, you will run migrations to create the actual tables in the database. To generate and execute your first migration, run the following command in the terminal:

- **Terminal command**
    
    ```bash
    npx prisma migrate dev --name "init"
    ```
    

This command will do three things:

1. **Save the migration**: Prisma Migrate will take a snapshot of your schema and figure out the SQL commands necessary to carry out the migration. Prisma will save the migration file containing the SQL commands to the newly created `prisma/migrations` folder.
2. **Execute the migration**: Prisma Migrate will execute the SQL in the migration file to create the underlying tables in your database.
3. **Generate Prisma Client**: Prisma will generate Prisma Client based on your latest schema. Since you did not have the Client library installed, the CLI will install it for you as well. You should see the `@prisma/client` package inside `dependencies` in your `package.json` file. Prisma Client is a TypeScript query builder auto-generated from your Prisma schema. It is *tailored* to your Prisma schema and will be used to send queries to the database.

<aside>
💡 **Note**: You can learn more about Prisma Migrate in the [Prisma docs](https://www.prisma.io/docs/concepts/components/prisma-migrate).

</aside>

If completed successfully, you should see a message like this: 

```bash
The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20220528101323_init/
    └─ migration.sql

Your database is now in sync with your schema.
...
✔ Generated Prisma Client (3.14.0 | library) to ./node_modules/@prisma/client in 31ms
```

### Task 3: Seed database with articles

Currently, the database is empty. So you will create a *seed script* that will populate the database with some dummy data. 

Start by creating a new `seed.ts` file in the `prisma` directory and implement the following steps:

- Import and instantiate `PrismaClient`
- Import `articles` seed data
- Create an `async` function called e.g. `main`
- Insert the articles (using `create` or `upsert`) to the database using `PrismaClient`
- Call the `main` function, catch and log any errors and finally disconnect the database connection

Check out the solution for the `seed.ts` file:

- **Solution**
    
    ```bash
    touch prisma/seed.ts
    ```
    
    ```tsx
    // prisma/seed.ts
    
    import { PrismaClient } from '@prisma/client';
    
    // initialize Prisma Client
    const prisma = new PrismaClient();
    
    async function main() {
      // create two dummy articles
      const post1 = await prisma.article.upsert({
        where: { title: 'Prisma Adds Support for MongoDB' },
        update: {},
        create: {
          title: 'Prisma Adds Support for MongoDB',
          body: 'Support for MongoDB has been one of the most requested features since the initial release of...',
          description:
            "We are excited to share that today's Prisma ORM release adds stable support for MongoDB!",
          published: false,
        },
      });
    
      const post2 = await prisma.article.upsert({
        where: { title: "What's new in Prisma? (Q1/22)" },
        update: {},
        create: {
          title: "What's new in Prisma? (Q1/22)",
          body: 'Our engineers have been working hard, issuing new releases with many improvements...',
          description:
            'Learn about everything in the Prisma ecosystem and community from January to March 2022.',
          published: true,
        },
      });
    
      console.log({ post1, post2 });
    }
    
    // execute the main function
    main()
      .catch((e) => {
        console.error(e);
        process.exit(1);
      })
      .finally(async () => {
        // close Prisma Client at the end
        await prisma.$disconnect();
      });
    ```
    

You need to tell Prisma what script to execute when running the seeding command. You can do this by adding the `prisma.seed` key to the end of your `package.json` file. This is needed to execute the `prisma/seed.ts` script. 

- **Solution**
    
    ```json
    // package.json
    
    // ...
      "scripts": {
        // ...
      },
      "dependencies": {
        // ...
      },
      "devDependencies": {
        // ...
      },
      "jest": {
        // ...
      },
      "prisma": {
        "seed": "ts-node prisma/seed.ts"
      }
    ```
    

Execute seeding with the following command:

```json
npx prisma db seed
```

You should see the following output: 

```
Running seed command `ts-node prisma/seed.ts` ...
{
  post1: {
    id: 1,
    title: 'Prisma Adds Support for MongoDB',
    description: "We are excited to share that today's Prisma ORM release adds stable support for MongoDB!",
    body: 'Support for MongoDB has been one of the most requested features since the initial release of...',
    published: false,
    createdAt: 2022-04-24T14:20:27.674Z,
    updatedAt: 2022-04-24T14:20:27.674Z
  },
  post2: {
    id: 2,
    title: "What's new in Prisma? (Q1/22)",
    description: 'Learn about everything in the Prisma ecosystem and community from January to March 2022.',
    body: 'Our engineers have been working hard, issuing new releases with many improvements...',
    published: true,
    createdAt: 2022-04-24T14:20:27.705Z,
    updatedAt: 2022-04-24T14:20:27.705Z
  }
}

🌱  The seed command has been executed.
```

You can verify with Prisma Studio if the data is in the database

```bash
npx prisma studio
```

<aside>
💡 **Note**: You can learn more about seeding in the [Prisma Docs](https://www.prisma.io/docs/guides/database/seed-database).

</aside>

### Task 4: Create a Nest module and service for `PrismaClient`

Prisma is set up and you are now ready to access the database via `PrismaClient`. To integrate `PrismaClient` into Nest architecture, generate a ***module*** and ***service*** both called `prisma` using the [Nest CLI](https://docs.nestjs.com/cli/usages#nest-generate). 

```bash
npx nest generate module prisma
npx nest generate service prisma

# or shorthand
npx nest g mo prisma
npx nest g s prisma
```

This should generate a new subdirectory `./src/prisma` with a `prisma.module.ts` and `prisma.service.ts` file. Update the service file to 

1. Extend the `PrismaClient` 
2. Implement `enableShutDownHooks` 
- **Solution**
    
    ```tsx
    // src/prisma/prisma.service.ts
    
    import { INestApplication, Injectable } from '@nestjs/common';
    import { PrismaClient } from '@prisma/client';
    
    @Injectable()
    export class PrismaService extends PrismaClient {
      async enableShutdownHooks(app: INestApplication) {
        this.$on('beforeExit', async () => {
          await app.close();
        });
      }
    }
    ```
    

<aside>
ℹ️ The `enableShutdownHooks` definition is needed to ensure your application shuts down gracefully. More information is available in the [NestJS docs](https://docs.nestjs.com/recipes/prisma#issues-with-enableshutdownhooks).

</aside>

The Prisma module will be responsible for creating a [singleton](https://docs.nestjs.com/modules#shared-modules) instance of the `PrismaService` and will allow sharing of the service throughout your application. To do this, add the `PrismaService` to the exports array in the `prisma.module.ts` file.

- **Solution**
    
    ```jsx
    // src/prisma/prisma.module.ts
    
    import { Module } from '@nestjs/common';
    import { PrismaService } from './prisma.service';
    
    @Module({
      providers: [PrismaService],
      exports: [PrismaService],
    })
    export class PrismaModule {}
    ```
    

Now, any module that *imports* the `PrismaModule` will have access to `PrismaService` and can inject it into its own components/services. This is a common pattern for NestJS applications. 

<aside>
ℹ️ Note: In some cases running the `nest generate` command with the server already running may result in NestJS throwing an exception that says: `Error: Cannot find module './app.controller'`. If you run into this error, run the following command from the terminal: `npm run prebuild` and restart the server.

</aside>

### Task 5: Set up Swagger

[Swagger](https://swagger.io/) is a tool to document your API using the [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification). Nest has a dedicated module for Swagger, which you will be using shortly.

Get started by installing the required dependencies:

```jsx
npm install --save @nestjs/swagger swagger-ui-express
```

Now open `main.ts` and initialize Swagger using the `SwaggerModule` class.  

- **Solution**
    
    ```tsx
    // src/main.ts
    
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
    
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
    
      const config = new DocumentBuilder()
        .setTitle('Median')
        .setDescription('The Median API description')
        .setVersion('0.1')
        .build();
    
      const document = SwaggerModule.createDocument(app, config);
      SwaggerModule.setup('api', app, document);
    
      await app.listen(3000);
    }
    bootstrap();
    ```
    

While the application is running, open your browser and navigate to http://localhost:3000/api. You should see the Swagger UI.

- Swagger UI
    
    ![swagger-ui.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c30b0c84-b20c-4fd3-b3f7-eb3a27f3a497/swagger-ui.png)
    

## Implement yourself

<aside>
⌨️ You get 5-10 minutes to complete the tasks.

</aside>