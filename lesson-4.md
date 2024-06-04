# 4. Error Handling

## Goal

The goal of this lesson is to create a Nest [Exception filter](https://docs.nestjs.com/exception-filters) to filter for unhandled `PrismaClient` exceptions (`PrismaClientKnownRequestError`) such as **unique** constraint validations. You'll apply the filter to the `ArticlesController` to gracefully handle exceptions thrown by `PrismaClient`.

## Setup

The start for this lesson is the `lesson-4-begin` branch of [this repo](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22). Before you switch to that branch, you need to commit the current state of your project. For simplicity, you can use the `stash` command:

```bash
git add .
git stash
git checkout lesson-4-begin
```

## Tasks

Execute `POST /articles` with the following object **twice** and see what the server response for the second call looks like and also have a look into the terminal.

```json
{
  "title": "Let’s build a REST API with NestJS and Prisma!",
  "description": "NestJS Workshop announcement.",
  "body": "NestJS is one of the hottest Node.js frameworks around. In this workshop, you will learn how to build a backend REST API with NestJS, Prisma, PostgreSQL and Swagger.",
  "published": true
}
```

- Response and Terminal
    
    The first call responds with status code `201` and the second fails with
    
    ```json
    {
      "statusCode": 500,
      "message": "Internal server error"
    }
    ```
    
    `PrismaClient` throws a unique constraint validation error because of the `title` field, which is marked as `@unique` in the `schema.prisma` file. The exception is typeof `PrismaClientKnownRequestError` and is exported at the `Prisma` namespace level.
    
    ```
    [Nest] 6803  - 06/14/2022, 3:25:40 PM   ERROR [ExceptionsHandler] 
    Invalid `this.prisma.article.create()` invocation in
    /Users/tasinishmam/my-code/talks-and-presentations/prisma-day-22-nestjs-workshop/median/src/articles/articles.service.ts:11:32
    
       8 constructor(private prisma: PrismaService) {}
       9 
      10 create(createArticleDto: CreateArticleDto) {
    → 11   return this.prisma.article.create(
      Unique constraint failed on the fields: (`title`)
    Error: 
    Invalid `this.prisma.article.create()` invocation in
    /Users/tasinishmam/my-code/talks-and-presentations/prisma-day-22-nestjs-workshop/median/src/articles/articles.service.ts:11:32
    8 constructor(private prisma: PrismaService) {}
       9 
      10 create(createArticleDto: CreateArticleDto) {
    → 11   return this.prisma.article.create(
      **Unique constraint failed on the fields: (`title`)**
    ```
    

The above response is handled by NestJS's built-in [global exception filter](https://docs.nestjs.com/exception-filters), which gives a `500 Internal server error` response for any **unrecognized** exception. You are going to create a custom `ExceptionFilter` for `PrismaClientKnownRequestError`s. 

### Task 1: Create `PrismaClientExceptionFilter`

Create `PrismaClientExceptionFilter` to handle [`PrismaClientKnownRequestError`](https://www.prisma.io/docs/reference/api-reference/error-reference#prismaclientknownrequesterror) for the entire application. The filter will handle exceptions like above and return an appropriate error code instead of `500`.

Start by generating a filter class by using the Nest CLI:

```bash
npx nest generate filter prisma-client-exception

# or shorthand
npx nest g f prisma-client-exception
```

Update the new generated `PrismaClientExceptionFilter` as follows

- use `PrismaClientKnownRequestError` as the exception type and add it to the `@Catch()` decorator too
- extend the filter class with [`BaseExceptionFilter`](https://docs.nestjs.com/exception-filters#inheritance), to keep the default fallback (`500 Internal server error`) behavior.
- Solution
    
    Prepare `PrismaClientExceptionFilter` with `BaseExceptionFilter` and catch `PrismaClientKnownRequestError`
    
    ```tsx
    //src/prisma-client-exception.filter.ts
    import { ArgumentsHost, Catch } from '@nestjs/common';
    import { BaseExceptionFilter } from '@nestjs/core';
    import { Prisma } from '@prisma/client';
    
    @Catch(Prisma.PrismaClientKnownRequestError)
    export class PrismaClientExceptionFilter extends BaseExceptionFilter {
      catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
    		// TODO catch error exception.code
    		
    		// default 500 error code
        super.catch(exception, host);
      }
    }
    ```
    

### Task 2: Catch `unique constraint` error code

The next steps include finding the error code on the  [Prisma docs](https://www.prisma.io/docs/reference/api-reference/error-reference#prisma-client-query-engine) for **unique constraint failed.** Then ****

- catch the `unique constraint` error code
- return a `Conflict` response (`statusCode: 409`)
- return the `exception` message in the response body
- Solution
    
    The error code for **unique constraint failed** is `P2002`. Use a `switch` statement to handle this error code differently.
    
    ```tsx
    //src/prisma-client-exception.filter.ts
    import { ArgumentsHost, Catch } from '@nestjs/common';
    import { BaseExceptionFilter } from '@nestjs/core';
    import { Prisma } from '@prisma/client';
    
    @Catch(Prisma.PrismaClientKnownRequestError)
    export class PrismaClientExceptionFilter extends BaseExceptionFilter {
      catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
        **switch (exception.code) {
          case 'P2002':
            // TODO return conflict response
            break;
          default:
            // default 500 error code
            super.catch(exception, host);
            break;
        }**
      }
    }
    ```
    
    Create a new response with a status code of conflict and return the exception message.
    
    ```tsx
    //src/prisma-client-exception.filter.ts
    import { ArgumentsHost, Catch, **HttpStatus** } from '@nestjs/common';
    import { BaseExceptionFilter } from '@nestjs/core';
    import { Prisma } from '@prisma/client';
    **import { Response } from 'express';**
    
    @Catch(Prisma.PrismaClientKnownRequestError)
    export class PrismaClientExceptionFilter extends BaseExceptionFilter {
      catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
        **const ctx = host.switchToHttp();
        const response = ctx.getResponse<Response>();
        const message = exception.message.replace(/\n/g, '');**
    
        switch (exception.code) {
          case 'P2002':
            **const status = HttpStatus.CONFLICT;
            response.status(status).json({
              statusCode: status,
              message: message,
            });**
            break;
          // TODO catch other error codes (e.g. 'P2000' or 'P2025')
          default:
            // default 500 error code
            super.catch(exception, host);
            break;
        }
      }
    }
    ```
    

### Task 3: Apply `PrismaClientExceptionFilter`

An Exception filter can be scoped to individual routes (method-scoped), entire controllers (controller-scoped) or even across the entire application (global-scoped). For now, you are going to scope the `PrismaClientExceptionFilter` to the `ArticlesController`. 

- Solution
    
    Method-scoped and Controller-scoped filters that extend the `BaseExceptionFilter` should not be instantiated with `new`. Instead, let NestJS instantiate them automatically.
    
    ```tsx
    // src/articles/articles.controller.ts
    
    import {
      Controller,
      Get,
      Post,
      Body,
      Patch,
      Param,
      Delete,
      NotFoundException,
      ParseIntPipe,
      **UseFilters**,
    } from '@nestjs/common';
    import { ArticlesService } from './articles.service';
    import { CreateArticleDto } from './dto/create-article.dto';
    import { UpdateArticleDto } from './dto/update-article.dto';
    import { ApiCreatedResponse, ApiOkResponse, ApiTags } from '@nestjs/swagger';
    import { ArticleEntity } from './entities/article.entity';
    **import { PrismaClientExceptionFilter } from 'src/prisma-client-exception.filter';**
    
    @Controller('articles')
    @ApiTags('articles')
    **@UseFilters(PrismaClientExceptionFilter)**  
    export class ArticlesController {
      constructor(private readonly articlesService: ArticlesService) {}
    }
    ```
    

Execute the same `POST /articles` call again and review the error response

- Response
    
    The response is now status code `409 Conflict` and contains the exception message:
    
    ![Screen Shot 2022-06-14 at 3.56.23 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4e5d95f-4fec-4fde-82ac-720892f60674/Screen_Shot_2022-06-14_at_3.56.23_PM.png)
    

Awesome, you are successfully handling `PrismaClient` exception for **unique** constraints. You can identify other error codes and add them to the filter. Here are some ideas:

- "**Record not found**" for `update` and `delete`
- "**Value too long for the column**" for `create` and `update`
- See the [docs](https://www.prisma.io/docs/reference/api-reference/error-reference#prisma-client-query-engine) for more error codes

## Implement yourself

<aside>
⌨️ You get 5 minutes to complete the tasks.

</aside>