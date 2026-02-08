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
8. Query your database
Now you can use Prisma Client to query your database. Create a script.ts file:
```Bash
touch script.ts
```
```TS
import { prisma } from './lib/prisma'

async function main() {
  // Example: Fetch all records from a table
  // Replace 'user' with your actual model name
  const allUsers = await prisma.students.findMany()
  console.log('All users:', JSON.stringify(allUsers, null, 2))
}

main()
  .then(async () => {
    await prisma.$disconnect()
  })
  .catch(async (e) => {
    console.error(e)
    await prisma.$disconnect()
    process.exit(1)
  })
```
Run the script:
```Bash
npx tsx script.ts
# Output: ===============================
# Need to install the following packages:
# tsx@4.21.0
# Ok to proceed? (y) y
# All users: [
#   {
#     "id": 1,
#     "name": "Ahmed"
#   },
#   {
#     "id": 2,
#     "name": "Sara"
#   }
# ]
```

9. Evolve your schema
To make changes to your database schema:

9.1. Update your Prisma schema file
Update your Prisma schema file to reflect the changes you want to make to your database schema. For example, add a new model:
```Bash
nvim prisma/schema.prisma
```
```Prisma
model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String?
  posts Post[]
}
```

9.2. Create and apply a migration:
```Bash
npx prisma migrate dev --name your_migration_name
```
- This command will:
    1. Create a new SQL migration file
    2. Apply the migration to your database
    3. Regenerate Prisma Client

10. Explore your data with Prisma Studio
Prisma Studio is a visual editor for your database. Launch it with:
```Bash
npx prisma studio
```
This opens a web interface where you can view and edit your data.
