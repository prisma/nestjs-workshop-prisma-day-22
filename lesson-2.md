# 2. REST API - CRUD Operations

## Goal

In this lesson, you will implement the Create, Read, Update, and Delete (CRUD) operations for the `Article` model in your REST API and any accompanying business logic.

## Setup

You are continuing to work with the same project that you set up in lesson 1. The start for this lesson is the `lesson-2-begin` branch of [this repo](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22). 

Before you switch to that branch, you need to commit the current state of your project. For simplicity, you can use the `stash` command:

```bash
git add .
git stash
git checkout lesson-2-begin

# clone repo from scratch
git clone -b lesson-2-begin git@github.com:prisma/nestjs-workshop-prisma-day-22.git
```

## Tasks: CRUD Operations

### Task 1: Generate REST Resource

You'll implement REST endpoints for all CRUD operations for the `Article` model. You'll need to generate a controller, a service, a module, entity class... wait there is an easier solution.

Nest CLI has **one** command to [generate all boilerplate code](https://trilon.io/blog/introducing-cli-generators-crud-api-in-1-minute#Generating-a-CRUD-API) for a new resource. Create a new resource for `articles` by running the following command and answering the CLI prompts:

```bash
npx nest generate resource

# CLI prompts
? What name would you like to use for this resource (plural, e.g., "users")? **articles**
? What transport layer do you use? **REST API**
? Would you like to generate CRUD entry points? (Y/n) **y**
```

You should now find a new `src/articles` directory with all the boilerplate for your REST endpoints. Inside the `src/articles/articles.controller.ts` file, you will see the definition of different routes (also called route handlers). The business logic for handling each request is encapsulated in the `src/articles/articles.service.ts` file. Currently, this file contains dummy implementations.

Start the Nest application if (`npm run start:dev`) and open the [Swagger API](http://localhost:3000/api/) you should see the new `articles` endpoints:

- Swagger UI
    
    ![articles-crud-1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/410fc40f-074f-4ffe-8cd9-b8cbea221daf/articles-crud-1.png)
    

Feel free to test out the dummy implementations of the REST endpoints via the Swagger API.

### Task 2: **Add `PrismaClient` to the `Articles` module**

To access `PrismaClient` inside the `Articles` module, you must add the `PrismaModule` as an import inside `articles.module.ts` 

- **Solution**
    
    ```bash
    // src/articles/articles.module.ts
    
    import { Module } from '@nestjs/common';
    import { ArticlesService } from './articles.service';
    import { ArticlesController } from './articles.controller';
    import { PrismaModule } from 'src/prisma/prisma.module';
    
    @Module({
      controllers: [ArticlesController],
      providers: [ArticlesService],
      imports: [PrismaModule],
    })
    export class ArticlesModule {}
    
    ```
    

You can now inject the `PrismaService` inside the `ArticlesService` and use it to access the database. To do this, add a constructor to `articles.service.ts` with an argument of type `PrismaService`.

- **Solution**
    
    ```bash
    // src/articles/articles.service.ts
    
    import { Injectable } from '@nestjs/common';
    import { CreateArticleDto } from './dto/create-article.dto';
    import { UpdateArticleDto } from './dto/update-article.dto';
    **import { PrismaService } from 'src/prisma/prisma.service';**
    
    @Injectable()
    export class ArticlesService {
      **constructor(private prisma: PrismaService) {}**
    
      // CRUD operations
    }
    ```
    

<aside>
ℹ️ If you need to access `PrismaService` in any other modules, you will follow the same pattern of importing the module and injecting the service. Alternatively, you can also declare a module as a **[Global Module](https://docs.nestjs.com/modules#global-modules).**

</aside>

### Task 3: **Define `GET /articles` endpoint**

The controller method for this endpoint is called `findAll`. This endpoint will return all published articles in the database.

Notice that the business logic for each controller is written in the `ArticlesService` class, in a function with the same name. So, you need to update `ArticlesService.findAll()` to return an array of all published articles in the database.

- **Solution**
    
    The `prisma.findMany` query will return all `article` records that match the `where` condition. 
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {}
    
      create(createArticleDto: CreateArticleDto) {
        return 'This action adds a new article';
      }
    
      findAll() {
        **return this.prisma.article.findMany({ where: { published: true } });**
      }
    ```
    

You can test out the endpoint by going to[`http://localhost:3000/api`](http://localhost:3000/api) and clicking on the **GET/articles** dropdown menu. Press **Try it out** and then **Execute** to see the result. 

- Test `GET /articles` endpoint
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c360b33-f5e0-4cc6-a6fb-52e0eee7b01a/Untitled.png)
    

<aside>
💡 **Note**: You can also run all requests in the browser directly or through a REST client (like [Postman](https://www.postman.com/)). Swagger also generates the curl commands for each request in case you want to run the HTTP requests in the terminal.

</aside>

### Task 4: **Define `GET /articles/drafts` endpoint**

You will define a new route to fetch all *unpublished* articles. NestJS did not automatically generate the controller route handler for this endpoint, so you have to write it yourself. Define the `findDrafts()` controller and service endpoints. 

- **Solution**
    
    ```tsx
    // src/articles/articles.controller.ts
    
    @Controller('articles')
    export class ArticlesController {
      constructor(private readonly articlesService: ArticlesService) {}
    
      @Post()
      create(@Body() createArticleDto: CreateArticleDto) {
        return this.articlesService.create(createArticleDto);
      }
    
      **@Get('drafts')
      findDrafts() {
        return this.articlesService.findDrafts();
      }**
    
      // ...
    
    }
    ```
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {}
    
      create(createArticleDto: CreateArticleDto) {
        return 'This action adds a new article';
      }
    
      **findDrafts() {
        return this.prisma.article.findMany({ where: { published: false } });
      }**
    
      // ...
    
    }
    ```
    

The `GET /articles/drafts` endpoint will now be available on the Swagger [API page](http://localhost:3000/api).

### Task 5: Define `GET /articles/:id` endpoint

The controller route handler for this endpoint is called `findOne`. It looks like this: 

```tsx
// src/articles/articles.controller.ts

@Get(':id')
findOne(@Param('id') id: string) {
  return this.articlesService.findOne(+id);
}  
```

The route accepts a dynamic `id` parameter, which is passed to the `findOne` controller route handler. Since the `Article` model has an integer `id` field, the `id` parameter needs to be cast to a number using the `+` operator. 

Similar to before, update the `findOne` method in the ArticlesService to return the article with the given id.

- **Solution**
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {}
    
      create(createArticleDto: CreateArticleDto) {
        return 'This action adds a new article';
      }
    
      findAll() {
        return this.prisma.article.findMany({ where: { published: true } });
      }
    
      findOne(id: number) {
        **return this.prisma.article.findUnique({ where: { id } });**
      }
    }
    ```
    

Once again, test out the endpoint by going to http://localhost:3000/api. Click on the **GET /articles/{id}** dropdown menu. Press **Try it out**, add a valid value to the **id** parameter, and press **Execute** to see the result.

- Test `GET /articles/:id` endpoint
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c9b72206-b92c-4ea4-ad76-6a6aeeb91213/Untitled.png)
    

Prisma, by default, does not raise exceptions when **`findUnique`** is called with an `id` that does not exist in the database. Currently, if an invalid `id` is provided to the endpoint, the API returns nothing but does not respond with an error code. To fix this, you can modify the `findOne` controller method to check if data is returned by the `findOne` service method. In case no data is returned, the controller can throw one of the NestJS **built-in HTTP exceptions**.

- **Solution**
    
    Normally NestJS will resolve any `promise` objects before sending data back to the client. However, here you need access to the `article` object inside the controller method. So both the controller and service `findOne` method need to be made `async`. 
    
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
      **NotFoundException,**
    } from '@nestjs/common';
    
    @Get(':id')
    **async findOne(@Param('id') id: string) {
      const article = await this.articlesService.findOne(+id);
    
      if (!article) {
        throw new NotFoundException(`Could not find article with ${id}.`);
      }
      return article;**
    }
    ```
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {}
    
      create(createArticleDto: CreateArticleDto) {
        return 'This action adds a new article';
      }
    
      findAll() {
        return this.prisma.article.findMany({ where: { published: true } });
      }
    
      **async findOne(id: number) {**
        return this.prisma.article.findUnique({ where: { id } });
      }
    }
    ```
    

Now, providing an invalid `id` to the `GET /articles/:id` endpoint will give an idiomatic error message. 

- Test `GET /articles/:id` endpoint with invalid `id`
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/879bf478-c5a0-450a-bf11-d04ab1afe114/Untitled.png)
    

### Task 6: **Define `POST /articles` endpoint**

This is the endpoint for creating new articles. The controller route handler for this endpoint is called create. It looks like this:

```tsx
// src/articles/articles.controller.ts

@Post()
create(@Body() createArticleDto: CreateArticleDto) {
  return this.articlesService.create(createArticleDto);
}
```

Notice that it expects arguments of type `CreateArticleDto` in the request body. A DTO (Data Transfer Object) is an object that defines how the data will be sent over the network. Currently, the `CreateArticleDto` is an empty class. You will add properties to it to define the shape of the request body. Moreover, you can automatically document it in Swagger by using the Swagger `@ApiProperty` decorator. 

Add the `title`, `description`, `body`, and `published` properties to the `CreateArticleDto`. 

- **Solution**
    
    ```tsx
    // src/articles/dto/create-article.dto.ts
    
    import { ApiProperty } from '@nestjs/swagger';
    
    export class CreateArticleDto {
      @ApiProperty()
      title: string;
    
      @ApiProperty({ required: false })
      description?: string;
    
      @ApiProperty()
      body: string;
    
      @ApiProperty({ required: false, default: false })
      published?: boolean = false;
    }
    ```
    

The **CreateArticleDto** should now be defined in the Swagger API page under **Schemas**. The shape of `UpdateArticleDto` is automatically inferred from the `CreateArticleDto` class definition. So **UpdateArticleDto** is also defined inside Swagger.

- Swagger definition of **CreateArticleDto** and **UpdateArticleDto**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8927bfff-cd46-41f6-ab55-99dee00d1c2e/Untitled.png)
    

Now update the `create` method in the `ArticlesService` to create a new article in the database.

- **Solution**
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {
      }
    
      create(createArticleDto: CreateArticleDto) {
        **return this.prisma.article.create({ data: createArticleDto });**
      }
    
      // ...
    }
    ```
    

### Task 7: **Define `PATCH /articles/:id` endpoint**

This endpoint is for updating existing articles. The route handler for this endpoint is called `update` and it has an argument of type `UpdateArticleDto`:

```tsx
// src/articles/articles.controller.ts

@Patch(':id')
update(@Param('id') id: string, @Body() updateArticleDto: UpdateArticleDto) {
  return this.articlesService.update(+id, updateArticleDto);
}
```

The `updateArticleDto` definition is defined as a [`PartialType`](https://docs.nestjs.com/openapi/mapped-types#partial) of `CreateArticleDto`. So it can have all the properties of `CreateArticleDto`.

Just like before, you must update the corresponding `ArticlesService` method for this operation. 

- **Solution**
    
    The `article.update` operation will try to find an `Article` record with the given `id` and update it with the data of `updateArticleDto`.
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) {}
    
      // ...
    
      update(id: number, updateArticleDto: UpdateArticleDto) {
        **return this.prisma.article.update({
          where: { id },
          data: updateArticleDto,
        });**
      }
    
      // ...
    }
    ```
    

<aside>
ℹ️ If no  `Article` record is found in the database with the given `id`, Prisma will return an error. In such cases, the API does not return a user-friendly error message. You will improve the error messages in Lesson  [4. Error Handling ](https://www.notion.so/4-Error-Handling-0c394e3c1e3040ca8b636f651771a18c?pvs=21). ****

</aside>

### Task 8: **Define `DELETE /articles/:id` endpoint**

The final endpoint is to delete existing articles. The route handler for this endpoint is called `remove`. 

You know the drill by now! Head over to `ArticlesService`and update the corresponding method. 

- **Solution**
    
    ```tsx
    // src/articles/articles.service.ts
    
    @Injectable()
    export class ArticlesService {
      constructor(private prisma: PrismaService) { }
    
      // ...
    
      remove(id: number) {
        **return this.prisma.article.delete({ where: { id } });**
      }
    }
    ```
    

You’ve finished implementing all the `Articles` endpoints! Congratulations 🎉 

### Task 9: **Group endpoints together in Swagger**

You can add an `@ApiTags` decorator to the `ArticlesController` class, to group all the `articles` endpoints together in Swagger. 

- **Solution**
    
    ```tsx
    // src/articles/articles.controller.ts
    
    **import { ApiTags } from '@nestjs/swagger';**
    
    @Controller('articles')
    **@ApiTags('articles')**
    export class ArticlesController {
      // ...
    }
    ```
    

The API page looks a bit more organized now. Grouping endpoints like this is very useful for navigating a large API with hundreds of endpoints. 

- Swagger UI with `articles` endpoints grouped together
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b7ddede-e69b-4b8c-be6a-1558353cef4e/Untitled.png)
    

### Task 10: **Update Swagger response types**

If you look at the **Responses** tab under each endpoint in Swagger, you will find that the **Description** is empty. This is because Swagger does not know the response types for any of the endpoints. To do this you will need to do the following: 

1. Update the `ArticleEntity` class in the `articles.entity.ts` file and annotate the fields with the Swagger `@ApiProperty()` decorator. 
- **Solution**
    
    Notice that `ArticleEntity` is an implementation of the `Article` type generated by Prisma Client, with `@ApiProperty` decorators added to each property.
    
    ```tsx
    // src/articles/entities/article.entity.ts
    
    import { Article } from '@prisma/client';
    import { ApiProperty } from '@nestjs/swagger';
    
    export class ArticleEntity implements Article {
      @ApiProperty()
      id: number;
    
      @ApiProperty()
      title: string;
    
      @ApiProperty({ required: false, nullable: true })
      description: string | null;
    
      @ApiProperty()
      body: string;
    
      @ApiProperty()
      published: boolean;
    
      @ApiProperty()
      createdAt: Date;
    
      @ApiProperty()
      updatedAt: Date;
    }
    ```
    
1. Annotate the controller route handlers with the correct response types for each endpoint. using the `ArticleEntity` class and … more decorators!  
- **Solution**
    
    ```tsx
    **import { ApiCreatedResponse, ApiOkResponse, ApiTags } from '@nestjs/swagger';
    import { ArticleEntity } from './entities/article.entity';**
    
    @Controller('articles')
    @ApiTags('articles')
    export class ArticlesController {
      constructor(private readonly articlesService: ArticlesService) {}
    
      @Post()
      **@ApiCreatedResponse({ type: ArticleEntity })**
      create(@Body() createArticleDto: CreateArticleDto) {
        return this.articlesService.create(createArticleDto);
      }
    
      @Get()
      **@ApiOkResponse({ type: ArticleEntity, isArray: true })**
      findAll() {
        return this.articlesService.findAll();
      }
    
      @Get('drafts')
      **@ApiOkResponse({ type: ArticleEntity, isArray: true })**
      findDrafts() {
        return this.articlesService.findDrafts();
      }
    
      @Get(':id')
      **@ApiOkResponse({ type: ArticleEntity })**
      async findOne(@Param('id') id: string) {
        const article = await this.articlesService.findOne(+id);
    
        if (!article) {
          throw new NotFoundException(`Could not find article with ${id}.`);
        }
        return article;
      }
    
      @Patch(':id')
      **@ApiOkResponse({ type: ArticleEntity })**
      update(@Param('id') id: string, @Body() updateArticleDto: UpdateArticleDto) {
        return this.articlesService.update(+id, updateArticleDto);
      }
    
      @Delete(':id')
      **@ApiOkResponse({ type: ArticleEntity })**
      remove(@Param('id') id: string) {
        return this.articlesService.remove(+id);
      }
    }
    ```
    

Now, Swagger should properly define the response type for all endpoints on the API page.

- Response types for endpoints in Swagger
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b362e833-dbd4-43ae-8267-c86e5bb072e6/Untitled.png)
    

<aside>
ℹ️ You can find all the response decorators that NestJS provides in the [NestJS docs](https://docs.nestjs.com/openapi/operations#responses)

</aside>

## Implement yourself

<aside>
⌨️ You get 10-15 minutes to complete the tasks.

</aside>