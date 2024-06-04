# 5. End-to-end Testing

## Goal

The goal of this lesson is to integrate [end-to-end testing](https://docs.nestjs.com/fundamentals/testing#end-to-end-testing) into your application. As an application grows, it becomes hard to manually test the end-to-end behavior of each API endpoint. Automated end-to-end tests help ensure that the overall behavior of the system is correct. 

## Setup

The start for this lesson is the `lesson-5-begin` branch of [this repo](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22). Before you switch to that branch, you need to commit the current state of your project. For simplicity, you can use the `stash` command:

```bash
git add .
git stash
git checkout lesson-5-begin
```

## Tasks

NestJS has a lot of nice features that make setting up testing relatively painless. In particular: 

- NestJS automatically scaffolds starting code and configuration for e2e tests
- Provides integration with [Jest](https://github.com/facebook/jest) and [Supertest](https://github.com/visionmedia/supertest) out-of-the-box
- Makes it easy to mock different components thanks to the dependency injection system

For e2e testing, NestJS creates a `test` directory at the root of the project. The project has two files: 

- The `test/app.e2e-spec.ts` file contains the scaffolding for a test suite. This is where you will be writing your e2e tests.
- The `test/jest-e2e.json` contains the jest configuration file.

<aside>
ℹ️ You can learn more about testing in NestJS [in the docs](https://docs.nestjs.com/fundamentals/testing). The [testing-nestjs](https://github.com/jmcdo29/testing-nestjs) repository on GitHub has some great testing examples with different types of applications.

</aside>

### Task 1: Configure e2e testing

Firstly, you will create a test database for running all your tests. This database will be set up from scratch every time you run your test suite to ensure that previous data does not affect your tests. The connection string for the database will be inside a new `.env.test` file, which you will now create: 

```bash
touch .env.test
```

Inside this file, add the connection string for the new database. Since you have admin credentials to the database running inside docker, you can easily create a new database on the fly. 

```bash
# env.test

DATABASE_URL="postgres://postgres:postgres@localhost:5432/**nestjs-workshop-test**"
```

- **Alternative code for SQLite**
    
    ```bash
    # env.test
    DATABASE_URL="file:./test.db"
    ```
    

You will need some way to tell Prisma to use the `.env.test` file for environment variables when running the test suite. You can do this using the `dotenv-cli` package. Go ahead and install this package. 

```bash
npm i dotenv-cli --save-dev
```

Inside `package.json` there is a `script` command called `test:e2e` for running e2e tests. You’ll need to modify it to do the following: 

1. Use `dotenv` to use `.env.test` for loading environment variables 
2. Reset the database and run `prisma migrate` to get the database in sync with your prisma schema. 
3. Run e2e tests with your test-runner (`jest`) using the proper jest configuration. 
- **Solution**
    
    Let’s break down the script step by step: 
    
    1. Use `dotenv` to use `.env.test` for loading environment variables: `dotenv -e .env.test` 
    2. Reset the database and run `prisma migrate` to get the database in sync with your prisma schema: `prisma migrate reset --force --skip-seed` 
    3. Run e2e tests with your test-runner (`jest`) using the proper jest configuration : `dotenv -e .env.test -- jest --runInBand --config ./test/jest-e2e.json`
    
    ```json
    "scripts": {
    		// ... 
        "test:e2e": "dotenv -e .env.test -- npx prisma migrate reset --force --skip-seed  && dotenv -e .env.test -- jest --runInBand --config ./test/jest-e2e.json;"
      },
    ```
    

Finally, you will need to update the jest configuration to add a [`moduleDirectories`](https://jestjs.io/docs/configuration#moduledirectories-arraystring) option. This is necessary to ensure absolute imports (eg: `import { PrismaModule } from '**src/prisma/prisma.module'**;`) work properly. 

- **Solution**
    
    ```json
    // test/jest-e2e.json
    
    {
      "moduleFileExtensions": [
        "js",
        "json",
        "ts"
      ],
      "rootDir": ".",
      "testEnvironment": "node",
      "testRegex": ".e2e-spec.ts$",
      "transform": {
        "^.+\\.(t|j)s$": "ts-jest"
      },
      **"moduleDirectories": [
        "<rootDir>/../",
        "node_modules"
      ]**
    }
    ```
    

Since NestJS has already created a basic test suite, you can go ahead and run the `test:e2e` command and it should work. 

```bash
npm run test:e2e
```

- The output should look like this
    
    ```
    > median@0.0.1 test:e2e
    > dotenv -e .env.test -- npx prisma migrate reset --force --skip-seed  && dotenv -e .env.test -- jest --runInBand --config ./test/jest-e2e.json ;
    
    Environment variables loaded from .env
    Prisma schema loaded from prisma/schema.prisma
    Datasource "db": PostgreSQL database "nestjs-workshop-test", schema "public" at "localhost:5432"
    
    PostgreSQL database nestjs-workshop-test created at localhost:5432
    
    Applying migration `20220613111522_init`
    
    Database reset successful
    
    The following migration(s) have been applied:
    
    migrations/
      └─ 20220613111522_init/
        └─ migration.sql
    
    ✔ Generated Prisma Client (3.15.1 | library) to ./node_modules/@prisma/client in 48ms
    
     PASS  test/app.e2e-spec.ts
      AppController (e2e)
        ✓ / (GET) (116 ms)
    
    Test Suites: 1 passed, 1 total
    Tests:       1 passed, 1 total
    Snapshots:   0 total
    Time:        1.879 s
    Ran all test suites.
    ```
    

Congrats, your test suite runs! Now if it was actually testing anything… 

### Task 2: Set up `beforeAll` for tests

In order to run end-to-end tests on the application, you will need access to an instance of your application. You can do this using the `createNestApplication()` function which will return a nest application instance based on some module reference. Module references can be generated using the `createTestingModule()` function. All of this is conveniently already done for you inside `test/app.e2e-spec.ts`! The only change you will make is replacing `beforeEach` with `beforeAll`.

- **Solution**
    
    ```tsx
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
    
      **beforeAll**(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        await app.init();
      });
    
      it('/ (GET)', () => {
        return request(app.getHttpServer())
          .get('/')
          .expect(200)
          .expect('Hello World!');
      });
    });
    ```
    

Next, you will want to populate your database with some dummy `articles`. You will want to do this directly using `prismaService` instead of through the API. As the API is one of the things you want to test, a bug in `POST /articles` may cause other tests to fail. You can retrieve any [providers](https://docs.nestjs.com/providers) (services, repositories, factories, helpers, etc) available in the `appModule` from its module reference. Using this feature, you can retrieve the `PrismaService` instance and insert some `articles` into the database. 

- **Solution**
    
    ```tsx
    // test/app.e2e-spec.ts
    
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    **import { PrismaService } from 'src/prisma/prisma.service';**
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        **prisma = app.get<PrismaService>(PrismaService);**
    
        await app.init();
    
        **await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });**
      });
    
      it('/ (GET)', () => {
        return request(app.getHttpServer())
          .get('/')
          .expect(200)
          .expect('Hello World!');
      });
    });
    ```
    

You also need to make a slight modification to bind the global `ValidationPipe` to the application instance and get it working properly with the `class-validator` package. 

- **Solution**
    
    Here’s a [StackOverflow post](https://stackoverflow.com/a/71989929/8855844) that explains a bit about why this is needed (it’s kind of confusing in my opinion 😭)
    
    ```tsx
    // test/app.e2e-spec.ts
    
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication, **ValidationPipe** } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    import { PrismaService } from 'src/prisma/prisma.service';
    **import { useContainer } from 'class-validator';**
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        prisma = app.get<PrismaService>(PrismaService);
    
        **useContainer(app.select(AppModule), { fallbackOnErrors: true });
        app.useGlobalPipes(new ValidationPipe({ whitelist: true }));**
    
        await app.init();
    
        await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });
      });
    
      it('/ (GET)', () => {
        return request(app.getHttpServer())
          .get('/')
          .expect(200)
          .expect('Hello World!');
      });
    });
    ```
    

Now, you will create just jest object matchers. This will be useful for testing return values from your API. Create an object matcher using `expect.objectContaining` for `Article` objects. 

- **Solution**
    
    ```tsx
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication, ValidationPipe } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    import { PrismaService } from 'src/prisma/prisma.service';
    import { useContainer } from 'class-validator';
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
      **const articleShape = expect.objectContaining({
        id: expect.any(Number),
        body: expect.any(String),
        title: expect.any(String),
        published: expect.any(Boolean),
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      });**
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        prisma = app.get<PrismaService>(PrismaService);
    
        useContainer(app.select(AppModule), { fallbackOnErrors: true });
        app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    
        await app.init();
    
        await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });
      });
    
      it('/ (GET)', () => {
        return request(app.getHttpServer())
          .get('/')
          .expect(200)
          .expect('Hello World!');
      });
    });
    ```
    

 Your testing setup is now ready! It’s time to write some tests! 

### Task 3: Add tests for `GET /articles` and `GET /articles/drafts`

You will be using [Supertest](https://github.com/visionmedia/supertest) to simulate HTTP requests. You will write two tests:

1. Fetch all published articles from the `GET /articles` endpoint and then ensure that the response is as expected (status code is `200`, the response is an array of `article` objects, articles are published, etc. 
2. Fetch all unpublished articles from the `GET /articles/drafts` endpoint and then ensure that the response is as expected (status code is `200`, the response is an array of `article` objects, articles are unpublished, etc. 
- Solution
    
    ```tsx
    // test/app.e2e-spec.ts
    
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication, ValidationPipe } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    import { PrismaService } from 'src/prisma/prisma.service';
    import { useContainer } from 'class-validator';
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
      const articleShape = expect.objectContaining({
        id: expect.any(Number),
        body: expect.any(String),
        title: expect.any(String),
        // Todo figure out how to have description be nullable
        published: expect.any(Boolean),
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      });
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        prisma = app.get<PrismaService>(PrismaService);
    
        useContainer(app.select(AppModule), { fallbackOnErrors: true });
        app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    
        await app.init();
    
        await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });
      });
    
      **describe('GET /articles', () => {
        it('returns a list of published articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[0].id);
          expect(body[0].published).toBeTruthy();
        });
    
        it('returns a list of draft articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles/drafts',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[1].id);
          expect(body[0].published).toBeFalsy();
        });
      });**
    });
    ```
    

 Most of the tests will be quite similar, they will run an HTTP request followed by assertions to make sure the response is as expected. 

### Task 4:  Add tests for `GET /articles/{id}`

You will be writing three more tests that: 

1. Successfully returns an article with a valid `id` from the database
2. Returns HTTP `404` error when trying to fetch an article that not exist in the database.
3. Returns HTTP `400` error when providing `id` of incorrect data type. 
- **Solution**
    
    ```tsx
    // test/app.e2e-spec.ts
    
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication, ValidationPipe } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    import { PrismaService } from 'src/prisma/prisma.service';
    import { useContainer } from 'class-validator';
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
      const articleShape = expect.objectContaining({
        id: expect.any(Number),
        body: expect.any(String),
        title: expect.any(String),
        // Todo figure out how to have description be nullable
        published: expect.any(Boolean),
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      });
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        prisma = app.get<PrismaService>(PrismaService);
    
        useContainer(app.select(AppModule), { fallbackOnErrors: true });
        app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    
        await app.init();
    
        await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });
      });
    
      describe('GET /articles', () => {
        it('returns a list of published articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[0].id);
          expect(body[0].published).toBeTruthy();
        });
    
        it('returns a list of draft articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles/drafts',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[1].id);
          expect(body[0].published).toBeFalsy();
        });
      });
    
      describe('GET /articles/{id}', () => {
        it('returns a given article', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/${articlesData[0].id}`,
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(articleShape);
          expect(body.id).toEqual(articlesData[0].id);
        });
    
        it('fails to return non existing user', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/100`,
          );
    
          expect(status).toBe(404);
        });
    
        it('fails to return article with invalid id type', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/string-id`,
          );
    
          expect(status).toBe(400);
        });
      });
    });
    ```
    

### Task 5:  Add tests for `POST /articles/{id}`

You will write two tests for this endpoint:

1. Successfully creates and returns an article with valid input. 
2. Returns HTTP `400` error when providing invalid data (no `title` ).
- **Solution**
    
    ```tsx
    // test/app.e2e-spec.ts
    
    import { Test, TestingModule } from '@nestjs/testing';
    import { INestApplication, ValidationPipe } from '@nestjs/common';
    import * as request from 'supertest';
    import { AppModule } from './../src/app.module';
    import { PrismaService } from 'src/prisma/prisma.service';
    import { useContainer } from 'class-validator';
    
    describe('AppController (e2e)', () => {
      let app: INestApplication;
      let prisma: PrismaService;
      const articleShape = expect.objectContaining({
        id: expect.any(Number),
        body: expect.any(String),
        title: expect.any(String),
        // Todo figure out how to have description be nullable
        published: expect.any(Boolean),
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      });
    
      const articlesData = [
        {
          id: 100001,
          title: 'title1',
          description: 'description1',
          body: 'body1',
          published: true,
        },
        {
          id: 100002,
          title: 'title2',
          description: 'description2',
          body: 'body2',
          published: false,
        },
      ];
    
      beforeAll(async () => {
        const moduleFixture: TestingModule = await Test.createTestingModule({
          imports: [AppModule],
        }).compile();
    
        app = moduleFixture.createNestApplication();
        prisma = app.get<PrismaService>(PrismaService);
    
        useContainer(app.select(AppModule), { fallbackOnErrors: true });
        app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    
        await app.init();
    
        await prisma.article.create({
          data: articlesData[0],
        });
        await prisma.article.create({
          data: articlesData[1],
        });
      });
    
      describe('GET /articles', () => {
        it('returns a list of published articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[0].id);
          expect(body[0].published).toBeTruthy();
        });
    
        it('returns a list of draft articles', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            '/articles/drafts',
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(expect.arrayContaining([articleShape]));
          expect(body).toHaveLength(1);
          expect(body[0].id).toEqual(articlesData[1].id);
          expect(body[0].published).toBeFalsy();
        });
      });
    
      describe('GET /articles/{id}', () => {
        it('returns a given article', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/${articlesData[0].id}`,
          );
    
          expect(status).toBe(200);
          expect(body).toStrictEqual(articleShape);
          expect(body.id).toEqual(articlesData[0].id);
        });
    
        it('fails to return non existing user', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/100`,
          );
    
          expect(status).toBe(404);
        });
    
        it('fails to return article with invalid id type', async () => {
          const { status, body } = await request(app.getHttpServer()).get(
            `/articles/string-id`,
          );
    
          expect(status).toBe(400);
        });
      });
    
      **describe('POST /articles', () => {
        it('creates an article', async () => {
          const beforeCount = await prisma.article.count();
    
          const { status, body } = await request(app.getHttpServer())
            .post('/articles')
            .send({
              title: 'title3',
              description: 'description3',
              body: 'body3a',
              published: false,
            });
    
          const afterCount = await prisma.article.count();
    
          expect(status).toBe(201);
          expect(afterCount - beforeCount).toBe(1);
        });
    
        it('fails to create article without title', async () => {
          const { status, body } = await request(app.getHttpServer())
            .post('/articles')
            .send({
              description: 'description4',
              body: 'body4',
              published: true,
            });
    
          expect(status).toBe(400);
        });
      });**
    });
    ```
    

Try running the test with `npm run test:e2e` 

- The output should look like this
    
    ```
    PASS  test/app.e2e-spec.ts
      AppController (e2e)
        GET /articles
          ✓ returns a list of published articles (22 ms)
          ✓ returns a list of draft articles (5 ms)
        GET /articles/{id}
          ✓ returns a given article (6 ms)
          ✓ fails to return non existing user (4 ms)
          ✓ fails to return article with invalid id type (2 ms)
        POST /articles
          ✓ creates an article (33 ms)
          ✓ fails to create article without title (2 ms)
    
    Test Suites: 1 passed, 1 total
    Tests:       7 passed, 7 total
    Snapshots:   0 total
    Time:        1.982 s, estimated 2 s
    Ran all test suites.
    ```
    

I think you get the idea of how this works. I will leave tests for **`PATCH /articles/:id`** and **`DELETE /articles/:id`** as exercises. Try writing them yourself! 

## Implement yourself

<aside>
⌨️ You get 5-7.5 minutes to complete the tasks.

</aside>