## NestJS Workshop Prisma Day 22 - Let's build a REST API with NestJS and Prisma! 

Welcome to the code repository for the Prisma Day 2022 NestJS Workshop! 

#### Resources

- [NestJS Documentation](https://docs.nestjs.com/)
- [Workshop Recording](https://youtu.be/LMjj1_EK4y8)

## Prerequisites

In order to successfully complete the tasks in the workshop, you should have

- `Node.js` installed on your machine (12.6.X / 14.X / 16.X) - see Prisma [system requirements](https://www.prisma.io/docs/reference/system-requirements) and `npm`
- `docker` and `docker-compose` installed on your machine (and available in your command line).
- Basic familiarity with NestJS is recommended for this workshop. (Optional)
- If you’re using VSCode, install the [Prisma extension](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma). (Optional)

***Note**: If you're not familiar with NestJS, you can quickly learn the basics by following the [overview section](https://docs.nestjs.com/first-steps) in the NestJS docs.*

## What you will do

In this hands-on workshop, you'll learn how to build a REST API with NestJS and the Prisma ORM in TypeScript. The application you will build is the backend REST API for a blog application called "Median" (a simple [Medium](https://medium.com/) clone).

Specifically, this workshop will cover:

🧩  Setting up Prisma with PostgreSQL database and integrating Prisma into NestJS architecture (Lesson 1)

✅  CRUD operations for REST API (Lesson 2)

🧹  Input Validation for your API (Lesson 3)

🧯  Handling errors in your API (Lesson 4)

🧪  End-to-End testing your API (Lesson 5)

## Lessons

[0. NestJS recap](./lesson-0.md)

[1. Set up Prisma, PostgreSQL and NestJS](./lesson-1.md)

[2. REST API - CRUD Operations](./lesson-2.md)

[3. Input Validation](./lesson-3.md)

[4. Error Handling ](./lesson-4.md)

[5. End-to-end Testing](./lesson-5.md)

[6. Conclusion & Additional Resources](./lesson-6.md) 

Lessons are structured in two parts: 

🎓  **Host walkthrough:** At the beginning of each *lesson*, your host will walk you through the different *tasks* you'll encounter in this lesson. Please *be attentive* during that time and follow the host's explanations to be sure that you can accomplish the tasks yourself when you're working on them later. **Do not code along or work on the tasks yourself yet!** Instead, you can think of questions or raise anything that you don't understand (e.g. in the **Q & A** section of Zoom).

⌨️  **Practice Time:** Once the host is done showing and explaining the different tasks, you get dedicated time to work on the tasks yourself! If you find yourself struggling, you can always look at the code in this notion document. The code is hidden inside toggle list items.

#### Get Started

Quickly get started with the appropriate lesson by cloning/checking out the relevant branch from this repo. 

1. [Lesson 1](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22/tree/lesson-1-begin): `git clone -b lesson-1-begin git@github.com:TasinIshmam/nestjs-workshop-prisma-day-22.git` 
2. [Lesson 2](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22/tree/lesson-2-begin): `git clone -b lesson-2-begin git@github.com:TasinIshmam/nestjs-workshop-prisma-day-22.git` 
3. [Lesson 3](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22/tree/lesson-3-begin): `git clone -b lesson-3-begin git@github.com:TasinIshmam/nestjs-workshop-prisma-day-22.git` 
4. [Lesson 4](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22/tree/lesson-4-begin): `git clone -b lesson-4-begin git@github.com:TasinIshmam/nestjs-workshop-prisma-day-22.git` 
5. [Lesson 5](https://github.com/TasinIshmam/nestjs-workshop-prisma-day-22/tree/lesson-5-begin): `git clone -b lesson-5-begin git@github.com:TasinIshmam/nestjs-workshop-prisma-day-22.git`  


#### Installation

1. Install dependencies: `npm install`
2. Start a PostgreSQL database with docker using: `docker-compose up -d`. 
    - If you have a local instance of PostgreSQL running, you can skip this step. However you will need to update the `DATABASE_URL` inside the `.env` file with the correct environment variable. 
3. Apply database migrations: `npx prisma migrate dev` 
4. Start the project:  `npm run start:dev`
5. Access the project at http://localhost:3000/api
