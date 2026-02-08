1. Set up Prisma ORM
npm install prisma @types/node --save-dev
npm install @prisma/client @prisma/adapter-mariadb dotenv

### Here's what each package does:
    1. prisma - The Prisma CLI for running commands like prisma init, prisma db pull, and prisma generate
    2. @prisma/client - The Prisma Client library for querying your database
    3. @prisma/adapter-mariadb - The MySQL/MariaDB driver adapter that connects Prisma Client to your database
    4. dotenv - Loads environment variables from your .env file

2. Initialize Prisma ORM
npx prisma init --datasource-provider mysql --output ../generated/prisma
### This command does a few things:
    1. Creates a prisma/ directory with a schema.prisma file containing your database connection configuration
    2. Creates a .env file in the root directory for environment variables
    2. Creates a prisma.config.ts file for Prisma configuration
