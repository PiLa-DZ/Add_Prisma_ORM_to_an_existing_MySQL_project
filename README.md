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

3. Connect your database
### Update the .env file with your MySQL connection string details:
```.env
DATABASE_URL="mysql://username:password@localhost:3306/mydb"
//add-start
DATABASE_USER="username"
DATABASE_PASSWORD="password"
DATABASE_NAME="mydb"
DATABASE_HOST="localhost"
DATABASE_PORT=3306
//add-end
```
### Replace the placeholders with your actual database credentials:
    1. username: Your MySQL username
    2. password: Your MySQL password
    3. localhost:3306: Your MySQL host and port
    4. mydb: Your database name

4. Introspect your database
- NOTE: Step 1: You have to create database and tables
```Bash
mysql -u admin -pyour_password
```
```MySQL
create database db1;
use db1;
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL
);
CREATE TABLE courses (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL
);
-- The Junction Table (The "Bridge")
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrolled_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    -- Composite Primary Key (Prevents a student from enrolling in the same course twice)
    PRIMARY KEY (student_id, course_id),
    -- Foreign Keys
    CONSTRAINT fk_student FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    CONSTRAINT fk_course FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);
INSERT INTO students (name) VALUES ('Ahmed'), ('Sara');
INSERT INTO courses (title) VALUES ('Database Design'), ('Web Development'), ('UI/UX');
-- Ahmed (1) takes Database (1) and Web (2)
INSERT INTO enrollments (student_id, course_id) VALUES (1, 1), (1, 2);
-- Sara (2) takes Web (2) and UI/UX (3)
INSERT INTO enrollments (student_id, course_id) VALUES (2, 2), (2, 3);
```
- NOTE: Step 2: Run the following command to introspect your existing database:
npx prisma db pull
This command reads the DATABASE_URL environment variable, connects to your database, and introspects the database schema. It then translates the database schema from SQL into a data model in your Prisma schema.

5. Baseline your database
To use Prisma Migrate with your existing database, you need to baseline your database.

    1. First, create a migrations directory:
    ```Bash
    mkdir -p prisma/migrations/0_init
    ```

    2. Next, generate the migration file with prisma migrate diff:
    ```Bash
    npx prisma migrate diff --from-empty --to-schema prisma/schema.prisma --script > prisma/migrations/0_init/migration.sql
    ```

    - Review the generated migration file to ensure it matches your database schema.
    3. Then, mark the migration as applied:
    ```Bash
    npx prisma migrate resolve --applied 0_init
    ```

You now have a baseline for your current database schema.

6. Generate Prisma ORM types
Generate Prisma Client based on your introspected schema:
```Bash
npx prisma generate
```
This creates a type-safe Prisma Client tailored to your database schema in the generated/prisma directory.

7. Instantiate Prisma Client
Create a utility file to instantiate Prisma Client. You need to pass an instance of Prisma ORM's driver adapter to the PrismaClient constructor:
```Bash
mkdir lib
touch prisma.ts
```
```TS
import "dotenv/config";
import { PrismaMariaDb } from '@prisma/adapter-mariadb';
import { PrismaClient } from '../generated/prisma/client';

const adapter = new PrismaMariaDb({
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  connectionLimit: 5
});
const prisma = new PrismaClient({ adapter });

export { prisma }
```

