# 0. NestJS recap

Nest is a framework for building efficient and scalable server-side applications with Node.js using Express (default) or Fastify as the HTTP Server.

Nest's architecture is heavily inspired by Angular. Nest architecture contains:

- Modules (`nest generate module`) organize and establish clear boundaries, grouping related Controllers/Resolvers/Services together. Modules can import other modules. There is a root module called `AppModule` which is responsible for starting the application.
    - Diagram
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/508d4b0d-b0fb-4781-87b8-54ac195ed300/Untitled.png)
        
- Controller (`nest generate controller`) defining REST endpoints. Each controller has one or more route handlers for each endpoint.
    - Diagram
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1b654789-7aad-4f7b-9e76-a5d377e9e75c/Untitled.png)
        
- Services (`nest generate service`) implement and isolate business logic. Services are a kind of [provider](https://docs.nestjs.com/providers).