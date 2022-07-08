## NestJS Workshop Prisma Day 22 - Let's build a REST API with NestJS and Prisma! 

Welcome to the code repository for the Prisma Day 2022 NestJS Workshop! 

#### Resources

- [Notion Workshop Document](http://pris.ly/day-22-nestjs)
- [NestJS Documentation](https://docs.nestjs.com/)
- [Workshop Recording](https://youtu.be/LMjj1_EK4y8) 


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
