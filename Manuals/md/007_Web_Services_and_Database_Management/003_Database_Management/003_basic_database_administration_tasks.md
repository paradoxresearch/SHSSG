# Basic Database Administration Tasks

## Contents

- [Basic Database Administration Tasks](#basic-database-administration-tasks)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Connecting to the Database Server](#connecting-to-the-database-server)
  - [User and Privilege Management](#user-and-privilege-management)
    - [Creating Users](#creating-users)
      - [Creating Users with MySQL and MariaDB](#creating-users-with-mysql-and-mariadb)
      - [Creating Users with PostgreSQL](#creating-users-with-postgresql)
    - [Granting and Revoking Privileges](#granting-and-revoking-privileges)
      - [Granting and Revoking Privileges with MySQL and MariaDB](#granting-and-revoking-privileges-with-mysql-and-mariadb)
      - [Granting and Revoking Privileges with PostgreSQL](#granting-and-revoking-privileges-with-postgresql)
    - [Removing Users](#removing-users)
      - [Removing Users with MySQL and MariaDB](#removing-users-with-mysql-and-mariadb)
      - [Removing Users with PostgreSQL](#removing-users-with-postgresql)
  - [Database and Table Management](#database-and-table-management)
    - [Database Management](#database-management)
      - [Database Management with MySQL and MariaDB](#database-management-with-mysql-and-mariadb)
      - [Database Management with PostgreSQL](#database-management-with-postgresql)
        - [Schemas in PostgreSQL](#schemas-in-postgresql)
    - [Table Management](#table-management)
      - [Table Management with MySQL and MariaDB](#table-management-with-mysql-and-mariadb)
      - [Table Management with PostgreSQL](#table-management-with-postgresql)
  - [Basic Data Manipulation](#basic-data-manipulation)
    - [SELECT](#select)
    - [INSERT](#insert)
    - [UPDATE](#update)
    - [DELETE](#delete)
  - [Backup and Restore](#backup-and-restore)
    - [Backup Your Database](#backup-your-database)
      - [Backup with MySQL and MariaDB](#backup-with-mysql-and-mariadb)
      - [Backup with PostgreSQL](#backup-with-postgresql)
    - [Restore Your Database](#restore-your-database)
      - [Restore with MySQL and MariaDB](#restore-with-mysql-and-mariadb)
      - [Restore with PostgreSQL](#restore-with-postgresql)

## Introduction

This manual briefly covers basic database administration tasks for MySQL, MariaDB, and PostgreSQL on Debian-based Linux Distributions.

## Connecting to the Database Server

Each of the databases we're discussing – MySQL, MariaDB, and PostgreSQL – has its own command-line tool that allows you to interact with it directly from your Ubuntu Server terminal.

- For **MySQL** and **MariaDB**, the tool is usually `mysql` (since MariaDB was initially a fork of MySQL, they share a lot of client tools). To connect, you'll typically use a command like this:

    ```bash
    mysql -u <username> -p
    ```

    Here, `<username>` is the name you use to log in. The `-p` flag tells MySQL to prompt you for your password. If you want to connect to a specific database right away, you can add its name at the end:

    ```bash
    mysql -u <username> -p <database_name>
    ```

- For **PostgreSQL**, the command-line tool is `psql`. The basic command to connect is similar:

    ```bash
    psql -U <username> -W
    ```

    Here, `-U` specifies the username, and `-W` prompts for the password. Just like with MySQL/MariaDB, you can specify a database to connect to:

    ```bash
    psql -U <username> -W <database_name>
    ```

Think of these commands as your personal entry codes. You provide your username, and the server asks for the secret password to let you in.

## User and Privilege Management

This is a crucial aspect of database administration, as it allows you to control who can access your databases and what actions they can perform. Think of it as setting up a security system for your database, ensuring that only authorized personnel have the necessary permissions.

The principle of **least privilege** is key here. It means granting users only the specific permissions they need to do their job and nothing more. For example, a user who only needs to read data should only have `SELECT` privileges, not the ability to `DELETE` or `ALTER` tables. This helps prevent accidental or malicious damage to your data.

Here's how it generally works across the three database systems:

- **Creating Users**: Each system has its own SQL command to create new users. You'll typically specify a username and a password (often hashed for security).
- **Granting Privileges**: You then grant specific permissions to these users on particular databases or even specific tables. Common privileges include `SELECT` (read data), `INSERT` (add data), `UPDATE` (modify data), `DELETE` (remove data), `CREATE` (create new databases or tables), `ALTER` (modify the structure of databases or tables), and `DROP` (delete databases or tables).
- **Revoking Privileges**: Just as you can grant privileges, you can also revoke them if a user's role changes or if they no longer need certain access.

### Creating Users

#### Creating Users with MySQL and MariaDB

You use the `CREATE USER` statement. The basic syntax is:

```SQL
CREATE USER '<username>'@'<host>' IDENTIFIED BY '<password>';
```

- `'<username>'`: The name you want to give to the new user.
- `'<host>'`: Specifies where the user can connect from (e.g., `'localhost'` for connections from the same server, `'%'` for connections from any host - be cautious with this!).
- `'<password>'`: The password for the new user. It's important to choose a strong, unique password for security.

For example, to create a user named `john` who can connect from `localhost` with the password `securepassword`:

```SQL
CREATE USER 'john'@'localhost' IDENTIFIED BY 'securepassword';
```

Now you need to grant privileges to the user.

#### Creating Users with PostgreSQL

You use the `CREATE ROLE` command to create new users (in PostgreSQL, users are a type of role).

```SQL
CREATE ROLE <username> WITH LOGIN PASSWORD '<password>';
```

- `<username>`: The name for the new user.
- `WITH LOGIN`: This specifies that this role can be used to log in.
- `PASSWORD '<password>'`: Sets the password for the user.

For example, to create a user named jane with the password anotherstrongpassword:

```SQL
CREATE ROLE jane WITH LOGIN PASSWORD 'anotherstrongpassword';
```

### Granting and Revoking Privileges

#### Granting and Revoking Privileges with MySQL and MariaDB

- **Granting Privileges**: You use the `GRANT` statement. The basic syntax looks like this:

    ```SQL
    GRANT <privilege(s)> ON <database_name>.<table_name> TO '<username>'@'<host>';
    ```

  - `<privilege(s)>`: A comma-separated list of privileges (e.g., `SELECT`, `INSERT`, `UPDATE`). Use `ALL PRIVILEGES` to grant all permissions (generally not recommended for the principle of least privilege!).
  - `<database_name>`.`<table_name>`: Specifies where the privileges apply. You can use `*.*` for all databases and all tables, `<database_name>.*` for all tables in a specific database, or `<database_name>`.`<table_name>` for a specific table.
  - `'<username>'@'<host>'`: Identifies the user. The `'<host>'` part specifies where the user can connect from (e.g., `'localhost'` for connections from the same server, `'%'` for connections from any host - be cautious with this!).

    For example, to grant a user named `alice` connecting from `localhost` the ability to `SELECT` and `INSERT` data into the users table in the mydatabase database, you'd do:

    ```SQL
        GRANT SELECT, INSERT ON mydatabase.users TO 'alice'@'localhost';
    ```

    Don't forget to run `FLUSH PRIVILEGES;` after granting or revoking privileges to make the changes take effect.

- **Revoking Privileges**: You use the `REVOKE` statement. The syntax is very similar to `GRANT`:

    ```SQL
    REVOKE <privilege(s)> ON <database_name>.<table_name> FROM '<username>'@'<host>';
    ```

    For example, to remove the INSERT privilege from alice on the users table:

    ```SQL
    REVOKE INSERT ON mydatabase.users FROM 'alice'@'localhost';
    ```

    Again, remember to run `FLUSH PRIVILEGES;`.

- **Viewing Privileges**: To see the privileges granted to a user, you can query the `mysql.user` and `mysql.db` tables (and other privilege-related tables) or use the `SHOW GRANTS` command:

    ```SQL
    SHOW GRANTS FOR 'alice'@'localhost';
    ```

#### Granting and Revoking Privileges with PostgreSQL

- **Granting Privileges**: You use the `GRANT` command. The syntax is a bit different:

    ```SQL
    GRANT <privilege(s)> ON <object_type> <object_name> TO <username>;
    ```

  - `<privilege(s)>`: A comma-separated list of privileges (e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `USAGE` (for databases and schemas), etc.). `ALL PRIVILEGES` also exists here.
  - `<object_type>`: The type of database object the privilege applies to (e.g., `TABLE`, `DATABASE`, `SCHEMA`).
  - `<object_name>`: The name of the database object.
  - `<username>`: The user to whom the privilege is granted.

    For example, to grant a user named `bob` the ability to `SELECT` and `INSERT` data into the users table in the mydatabase database:

    ```SQL
    GRANT SELECT, INSERT ON TABLE mydatabase.users TO bob;
    ```

    To grant `bob` the ability to connect to the `mydatabase` database, you'd use the `USAGE` privilege:

    ```SQL
    GRANT USAGE ON DATABASE mydatabase TO bob;
    ```

- **Revoking Privileges**: You use the REVOKE command:

    ```SQL
    REVOKE <privilege(s)> ON <object_type> <object_name> FROM <username>;
    ```

    For example, to remove the `INSERT` privilege from `bob` on the `users` table:

    ```SQL
    REVOKE INSERT ON TABLE mydatabase.users FROM bob;
    ```

- **Viewing Privileges**: You can view privileges by querying the system catalogs, particularly tables in the `pg_catalog` schema, or by using the `\dp` command in `psql` to list access privileges for tables, schemas, and databases. For a specific user, you might look at the output of:

    ```SQL
    \dp <username>
    ```

It might seem like a lot of commands, but the underlying concept is the same across all three: you specify what a user can do (*privileges*) on what (*database/table/object*) for whom (*user*).

### Removing Users

#### Removing Users with MySQL and MariaDB

You use the `DROP USER` statement to remove a user. The syntax is:

```SQL
DROP USER '<username>'@'<host>';
```

- `'<username>'@'<host>'`: This precisely identifies the user you want to remove. You need to specify both the username and the host from which they connect.

For example, to remove the user `john` who connects from `localhost`:

```SQL
DROP USER 'john'@'localhost';
```

It's important to note that dropping a user will also revoke any privileges specifically granted to that user.

#### Removing Users with PostgreSQL

You use the `DROP ROLE` command to remove a user (remember, users are a type of role in PostgreSQL).

```SQL
DROP ROLE <username>;
```

- `<username>`: The name of the user you want to remove.

For example, to remove the user jane:

```SQL
DROP ROLE jane;
```

Similar to MySQL/MariaDB, dropping a role (user) will also revoke any privileges directly associated with that role. However, if the user is a member of any groups (roles), the privileges granted to those groups will still apply to other members.

## Database and Table Management

This is where you learn how to organize your data by creating and managing the containers that hold it. Think of databases as large filing cabinets, and tables as the individual drawers within those cabinets where you store specific types of information.

### Database Management

#### Database Management with MySQL and MariaDB

- **Creating Databases**: You use the `CREATE DATABASE` statement:

    ```SQL
    CREATE DATABASE <database_name>;
    ```

- **Listing Databases**: To see all the databases on the server, you use:

    ```SQL
    SHOW DATABASES;
    ```

- **Selecting a Database**: To work with a specific database, you use the `USE` statement:

    ```SQL
    USE <database_name>;
    ```

- **Dropping Databases**: To delete a database (and all the tables and data it contains - be very careful with this!), you use:

    ```SQL
    DROP DATABASE <database_name>;
    ```

#### Database Management with PostgreSQL

- **Creating Databases**:

    ```SQL
    CREATE DATABASE <database_name>;
    ```

- **Listing Databases**: You can use the `\l` command in `psql`:

    ```SQL
    \l
    ```

    Alternatively, you can query the system catalog:

    ```SQL
    SELECT datname FROM pg_database;
    ```

- **Connecting to a Database**: You specify the database when you connect with `psql`:

    ```bash
    psql -U <username> -W <database_name>
    ```

    Or, if you're already connected, you can use the `\c` command:

    ```SQL
    \c <database_name>
    ```

- **Dropping Databases**: Again, be cautious! This deletes the database and its contents:

    ```SQL
    DROP DATABASE <database_name>;
    ```

##### Schemas in PostgreSQL

PostgreSQL has an additional concept called **schemas**. Think of a schema as a namespace within a database that can hold tables, views, and other database objects. By default, PostgreSQL has a `public` schema. You can create additional schemas to further organize your database objects. This can be helpful for managing different applications or sets of data within the same database.

- **Creating Schemas**:

    ```SQL
    CREATE SCHEMA <schema_name>;
    ```

- When creating tables or other objects, you can specify the schema:

    ```SQL
    CREATE TABLE <schema_name>.<table_name> (...);
    ```

- Or you can set the current search path to work within a specific schema:

    ```SQL
    SET search_path TO <schema_name>;
    ```

### Table Management

#### Table Management with MySQL and MariaDB

- **Creating Tables**: You use the `CREATE TABLE` statement. You'll need to define the table name and the columns it will contain, along with their data types (e.g., `INT` for integers, `VARCHAR` for strings, `DATE` for dates). You can also specify constraints like `NOT NULL` (the column can't be empty) and primary keys (unique identifiers for each row).

    ```SQL
    CREATE TABLE users (
        id INT PRIMARY KEY AUTO_INCREMENT,
        username VARCHAR(50) NOT NULL UNIQUE,
        email VARCHAR(100) NOT NULL,
        registration_date DATE
    );
    ```

- **Listing Tables**: To see the tables in the currently selected database, you use:

    ```SQL
    SHOW TABLES;
    ```

- **Describing Tables**: To see the structure of a table (its columns, data types, and constraints), you use the `DESCRIBE` or `EXPLAIN` command:

    ```SQL
    DESCRIBE users;
    ```

    or

    ```SQL
    EXPLAIN users;
    ```

- **Altering Tables**: To modify the structure of an existing table (e.g., add or remove columns, change data types, add or remove constraints), you use the `ALTER TABLE` statement. This is a powerful command with various options.

  - **Adding a column**:

    ```SQL
    ALTER TABLE users ADD COLUMN city VARCHAR(50);
    ```

  - **Modifying a column's data type**:

    ```SQL
    ALTER TABLE users MODIFY COLUMN email VARCHAR(150);
    ```

  - **Adding a** `NOT NULL` **constraint**:

    ```SQL
    ALTER TABLE users MODIFY COLUMN city VARCHAR(50) NOT NULL;
    ```

  - **Adding a unique constraint to an existing column**:

    ```SQL
    ALTER TABLE users ADD UNIQUE (email);
    ```

  - **Dropping Tables**: To delete a table and all its data (again, be careful!), you use:

    ```SQL
    DROP TABLE users;
    ```

#### Table Management with PostgreSQL

- **Creating Tables**: Similar to MySQL, you use `CREATE TABLE` with column definitions, data types, and constraints.

    ```SQL
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        username VARCHAR(50) UNIQUE NOT NULL,
        email VARCHAR(100) NOT NULL,
        registration_date DATE
    );
    ```

    Note the use of `SERIAL` which automatically creates an auto-incrementing integer column and a related sequence.

- **Listing Tables**: To see tables in the current database (and schema), you can use the `\dt` command in `psql`:

    ```SQL
    \dt
    ```

    To see tables in all schemas, you can use:

    ```SQL
    \dt *.*
    ```

    Alternatively, query the system catalog:

    ```SQL
    SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'; -- Or another schema name
    ```

- **Describing Tables**: Use the `\d` command followed by the table name in `psql`:

    ```SQL
    \d users
    ```

- **Altering Tables**: The `ALTER TABLE` statement is also used in PostgreSQL, with similar capabilities:

  - **Adding a column**:

    ```SQL
    ALTER TABLE users ADD COLUMN city VARCHAR(50);
    ```

  - **Modifying a column's data type**:

    ```SQL
    ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(150);
    ```

  - **Adding a** `NOT NULL` **constraint**:

    ```SQL
    ALTER TABLE users ALTER COLUMN city SET NOT NULL;
    ```

  - **Adding a unique constraint to an existing column**:

    ```SQL
    ALTER TABLE users ADD UNIQUE (email);
    ```

  - **Dropping Tables**:

    ```SQL
    DROP TABLE users;
    ```

As you can see, the fundamental operations are quite similar across the systems, although the specific syntax might have slight variations. Understanding data types is also important here, as they determine what kind of data can be stored in each column.

## Basic Data Manipulation

This is where you'll learn how to interact with the data inside your tables. The four fundamental SQL commands for this are:

- **SELECT**: Used to retrieve data from one or more tables. It's how you ask the database to show you specific information.
- **INSERT**: Used to add new rows of data into a table. It's how you put new information into your database.
- **UPDATE**: Used to modify existing data in a table. It's how you change information that's already there.
- **DELETE**: Used to remove rows from a table. It's how you take information out of your database.

Let's look at a simple example for each:

### SELECT

Imagine you have a `users` table with columns like `id`, `username`, and `email`. To retrieve all the usernames and emails from this table, you would use:

```SQL
SELECT username, email FROM users;
```

To retrieve all columns, you can use `*`:

```SQL
SELECT * FROM users;
```

You can also add conditions using the `WHERE` clause. For example, to get the information for the user with the `ID` of `1`:

```SQL
SELECT * FROM users WHERE id = 1;
```

### INSERT

To add a new user to the `users` table, you would specify the table name and the values for each column:

```SQL
INSERT INTO users (username, email, registration_date) VALUES ('newuser', 'newuser@example.com', '2025-05-13');
```

If you are providing values for all columns in the table in their defined order, you can omit the column names:

```SQL
INSERT INTO users VALUES (NULL, 'anotheruser', 'another@example.com', '2025-05-13');
```

(Note: `NULL` is used here for the id column assuming it's auto-incrementing.)

### UPDATE

To change the email address of the user with the ID of 1, you would use the `UPDATE` command along with the `SET` clause to specify which column to modify and the `WHERE` clause to identify the row(s) to update:

```SQL
UPDATE users SET email = 'updated@example.com' WHERE id = 1;
```

### DELETE

To remove the user with the ID of 2 from the users table, you would use the `DELETE FROM` command with a `WHERE` clause to specify which rows to delete:

```SQL
DELETE FROM users WHERE id = 2;
```

**Important Note**: Be very careful when using `DELETE` without a `WHERE` clause, as it will remove all rows from the table!

These four commands form the basis of interacting with data in any relational database. The syntax is generally consistent across MySQL, MariaDB, and PostgreSQL.

## Backup and Restore

This is arguably one of the most critical aspects of database administration. Imagine all the valuable data you've been managing – losing it due to a hardware failure, a software glitch, or even a simple human error would be a major setback! Regular backups are your safety net, allowing you to restore your database to a previous working state.

There are different types of backups, but we'll focus on basic logical backups using command-line tools. Logical backups contain the structure of your database (schemas, tables) and the data itself in a format that can be re-executed to rebuild the database.

Here's how you can perform basic logical backups for each of our databases:

### Backup Your Database

#### Backup with MySQL and MariaDB

The primary tool for creating logical backups is `mysqldump`. You can use it to back up entire databases or specific tables.

- **Backing up an entire database**:

    ```Bash
    mysqldump -u <username> -p <database_name> > <backup_file.sql>
    ```

    Replace `<username>` with your MySQL user, `<database_name>` with the name of the database you want to back up, and `<backup_file.sql>` with the name you want to give to your backup file (it's common to use the `.sql` extension). You'll be prompted for the password.

- **Backing up specific tables**:

    ```Bash
    mysqldump -u <username> -p <database_name> <table1> <table2> > <backup_file.sql>
    ```

    Just list the names of the tables you want to include in the backup.

The resulting `<backup_file.sql>` is a text file containing SQL commands to recreate your database and insert the data.

#### Backup with PostgreSQL

The main tool for creating logical backups in PostgreSQL is `pg_dump`.

- **Backing up an entire database**:

    ```Bash
    pg_dump -U <username> -d <database_name> -f <backup_file.sql>
    ```

    Here, `-U` specifies the PostgreSQL user, `-d` specifies the database name, and `-f` specifies the output file. You might be prompted for the password.

- **Backing up specific tables**:

    ```Bash
    pg_dump -U <username> -d <database_name> -t <table1> -t <table2> -f <backup_file.sql>
    ```

    Use the `-t` option followed by the table name for each table you want to back up.

Just like `mysqldump`, `pg_dump` creates a `.sql` file with the necessary SQL commands.

It's a good practice to regularly perform backups and to store these backup files in a secure location separate from your database server. You might also consider automating this process using cron jobs.

### Restore Your Database

The process involves using the command-line tools we discussed earlier to execute the SQL commands stored in your backup file against the database server.

Here's how you generally restore:

#### Restore with MySQL and MariaDB

You use the mysql command-line client to execute the SQL file you created with mysqldump.

- **Restoring an entire database**:

    First, you might need to create an empty database if it doesn't already exist:

    ```SQL
    CREATE DATABASE <database_name>;
    ```

    Then, you can restore the data by redirecting the content of your backup file to the mysql command:

    ```bash
    mysql -u <username> -p <database_name> < <backup_file.sql>
    ```

    Replace `<username>` with your MySQL user, `<database_name>` with the name of the database you want to restore into, and `<backup_file.sql>` with the path to your backup file. You'll be prompted for the password.

#### Restore with PostgreSQL

You use the `psql` command-line client or the `pg_restore` tool. For a .sql format backup created with `pg_dump` (which we discussed), `psql` is typically used:

- **Restoring an entire database**:

    Similar to MySQL, you might need to create the database first if it's a full restore:

    ```SQL
    CREATE DATABASE <database_name>;
    ```

    Then, you can restore by using `psql` and redirecting the backup file:

    ```bash
    psql -U <username> -d <database_name> -f <backup_file.sql>
    ```

    Here, `-U` is your PostgreSQL user, `-d` is the target database, and `-f` specifies the backup file. You might be prompted for the password.

It's important to ensure that the database you are restoring into exists (or you create it first) and that the user you are using for the restore has the necessary privileges to create objects and insert data.
