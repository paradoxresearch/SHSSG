# Installing, Setting Up and Securing PostgreSQL

## Contents

- [Installing, Setting Up and Securing PostgreSQL](#installing-setting-up-and-securing-postgresql)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [What is PostgreSQL?](#what-is-postgresql)
    - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [Verify the Installation](#verify-the-installation)
  - [Initial Security Configuration](#initial-security-configuration)
    - [Changing the PostgreSQL 'postgres' User Password](#changing-the-postgresql-postgres-user-password)
    - [Configuring Authentication Methods (pg\_hba.conf)](#configuring-authentication-methods-pg_hbaconf)
  - [Basic Security Practices](#basic-security-practices)
    - [Creating Dedicated User Roles](#creating-dedicated-user-roles)
    - [Limiting User Privileges](#limiting-user-privileges)
    - [Keeping PostgreSQL Updated](#keeping-postgresql-updated)
  - [Firewall Configuration (UFW)](#firewall-configuration-ufw)
    - [Installing UFW](#installing-ufw)
    - [Enabling UFW](#enabling-ufw)
    - [Allowing SSH Traffic](#allowing-ssh-traffic)
    - [Enabling and Checking Firewall Status](#enabling-and-checking-firewall-status)
  
## Introduction

### What is PostgreSQL?

Imagine you have a vast library of information – maybe all the books in your favorite bookstore. PostgreSQL is like the librarian and the organizational system for that library, but instead of books, it manages data.

More technically, PostgreSQL is a powerful, open-source relational database management system (RDBMS). That's a bit of a mouthful, so let's break it down:

- **Database Management System (DBMS)**: This is software that allows you to create, manage, and access databases. Think of it as the software that runs our digital library.
- **Relational**: This means the data is organized into tables, and these tables can be related to each other. It's like having different sections in our library (fiction, non-fiction, etc.) and being able to find connections between them (e.g., books by the same author might appear in multiple sections).
- **Open-source**: This means the software is free to use, distribute, and modify. It's like our library having open membership and allowing anyone to contribute to its organization.

PostgreSQL is known for its reliability, robustness, and adherence to standards. It's used by everyone from small startups to large enterprises to store and manage all sorts of data. It's like a super-librarian that can handle millions of books and keep everything organized and accessible!

### Prerequisites

Before we jump into installing PostgreSQL on your Ubuntu server, there are a few things you should have in place:

1. **A Debian-based Linux distribution**: This manual is specifically tailored for Debian-based Linux distributions.
2. **SSH Access**: You'll need to be able to connect to your Ubuntu server remotely using SSH (Secure Shell). This allows you to execute commands on the server from your local machine. You should have your server's IP address and your SSH credentials (username and password or an SSH key).
3. **Basic Command-Line Knowledge**: We'll be working with the Linux command line, so a basic understanding of commands like `sudo`, `apt update`, `apt install`, etc., will be helpful. Think of it as knowing how to navigate the basic functions of your computer.
4. `sudo` **Privileges**: The commands we'll be using to install and configure software will often require administrative rights. Make sure the user you're logged in as has `sudo` privileges. This allows you to run commands as the superuser (root).

## Installation

Before we install any new software, it's always a good practice to update the package lists. This ensures that your system has the most up-to-date information about available software packages.

To do this, you'll use the following command in your terminal, after SSHing into your server (if required):

```bash
sudo apt update
```

To install PostgreSQL and its associated packages, we'll use the `apt install` command. Specifically, we'll install the `postgresql` package, which includes the PostgreSQL server, client utilities, and development libraries.

Run the following command in your terminal:

```bash
sudo apt install postgresql postgresql-contrib
```

Here's what this command does:

- `sudo`: As we discussed, this gives us the necessary administrative privileges to install software.
- `apt install`: This is the command used to install new packages on Ubuntu.
- `postgresql`: This is the main package for the PostgreSQL server.
- `postgresql-contrib`: This package includes additional utilities and extensions that can be very helpful, so it's a good idea to install it as well. Think of it as getting a helpful companion guide along with our main PostgreSQL book.

Your server will then download the necessary files and install PostgreSQL on your server. You might be prompted to confirm the installation – just type `y` and press Enter to continue.

Once the installation is complete, PostgreSQL should be running on your server. We'll verify this in the next step.

### Verify the Installation

Now that we've (hopefully!) installed PostgreSQL, we need to make sure it's actually running correctly.

There are a few ways to verify the installation:

1. **Check the Service Status**: We can use the `systemctl` command to check the status of the PostgreSQL service. Run the following command:

    ```bash
    sudo systemctl status postgresql
    ```

    This command will show you information about the PostgreSQL service, including whether it's active (running), when it started, and any recent logs. Look for a line that says something like `Active: active (running)` – this indicates that PostgreSQL is up and running.

2. **Connect to the PostgreSQL Server**: Another way to verify is to try and connect to the PostgreSQL server using one of its command-line tools. The `psql` command is a terminal-based front-end to PostgreSQL. After installation, a default PostgreSQL user named `postgres` is created. You can try to switch to this user and then connect to the PostgreSQL prompt.

    First, switch to the `postgres` user:

    ```bash
    sudo su - postgres
    ```

    You'll notice your terminal prompt change to postgres@your_hostname:~$. Now, try to connect to the PostgreSQL prompt by running:

    ```bash
    psql
    ```

    If the connection is successful, you'll see a prompt that looks like `postgres=#`. You can then exit this prompt by typing `\q` and pressing Enter, and then exit the `postgres` user session by typing `exit` and pressing Enter to return to your regular user.

If you can successfully check the service status and connect to the `psql` prompt, it means PostgreSQL has been installed and is running on your server.

## Initial Security Configuration

### Changing the PostgreSQL 'postgres' User Password

When PostgreSQL is installed, it creates a default administrative user named `postgres`. This user has superuser privileges within the database, meaning it can do just about anything. For security reasons, it's crucial to set a strong password for this user as soon as possible. Leaving it with the default settings is like leaving the master key to our library out in the open!

Here's how you can change the password for the postgres user:

1. **Switch to the** `postgres` **user**: Just like we did when verifying the installation, first switch to the postgres Linux user:

    ```bash
    sudo su - postgres
    ```

2. **Access the PostgreSQL prompt**: Once you're logged in as the `postgres` user, access the PostgreSQL command-line interface:

    ```bash
    psql
    ```

3. Change the password: At the `postgres=#` prompt, you can change the password for the `postgres` database user using the `ALTER ROLE` command:
SQL

    ```sql
    ALTER ROLE postgres WITH PASSWORD 'your_new_strong_password';
    ```

    **Important**: Replace 'your_new_strong_password' with a strong, unique password. A strong password typically includes a mix of uppercase and lowercase letters, numbers, and special characters.

4. **Exit PostgreSQL and the** `postgres` **user**: After successfully changing the password, exit the PostgreSQL prompt by typing `\q` and pressing Enter, and then exit the `postgres` user session by typing `exit` and pressing Enter.

Changing the default password is a fundamental security practice. It's like the first lock you put on the library door!

### Configuring Authentication Methods (pg_hba.conf)

Now that we've secured the `postgres` user with a strong password, the next crucial step in our initial security configuration is to manage how users are allowed to connect to the PostgreSQL database. This is like deciding who gets a key to our library and what kind of key they get.

This is configured in a file called `pg_hba.conf`. The hba stands for "host-based authentication." This file tells PostgreSQL which hosts are allowed to connect, which users they can connect as, and what authentication methods to use.

The `pg_hba.conf` file is typically located in the PostgreSQL data directory. You can usually find its location by running the following command as the `postgres` user:

```bash
psql -c 'SHOW hba_file;'
```

This command will output the full path to the `pg_hba.conf` file. You'll need to edit this file using a text editor with `sudo` privileges (since it's usually owned by the `postgres` user). For example, you might use `nano`:

```bash
sudo nano /etc/postgresql/your_version/main/pg_hba.conf
```

**(Replace** `your_version` **with the actual version number of PostgreSQL you installed, e.g.,** `14`**,** `15`**, etc.)**

The `pg_hba.conf` file consists of a series of records, each specifying an authentication rule. Each line typically has the following format:

```shell
type  database  user  address  method
```

**Let's break down these fields**:

- `type`: Specifies the connection type. Common values include `local` (for connections from the same host via Unix domain sockets) and `host` (for TCP/IP connections).
- `database`: Specifies the database(s) the rule applies to. `all` means all databases. You can also specify individual database names.
- `user`: Specifies the PostgreSQL user(s) the rule applies to. `all` means all users. You can also specify individual usernames.
- `address`: Specifies the client IP address or range of addresses the rule applies to. `all` means any IP address. You can also specify specific IP addresses (e.g., `192.168.1.10`) or network ranges (e.g., `192.168.1.0/24`). For local connections, this field is usually `all` or left empty.
- `method`: Specifies the authentication method to be used. Some common methods include:
  - `trust`: Allows connections without a password (generally not recommended for production environments!).
  - `password`: Requires a password for authentication (passwords are sent in clear text, so it's less secure for remote connections).
  - `md5`: Requires an MD5-hashed password for authentication (more secure than `password`).
  - `scram-sha-256`: A more modern and secure password hashing method (recommended).
  - `peer`: Uses the operating system's user identity for local connections.

**Important Security Considerations in** `pg_hba.conf`:

- **Local Connections**: For local connections (from the same server), the default `peer` authentication is usually fine. This relies on the operating system's user credentials.
- **Remote Connections**: For connections from other machines, you'll likely want to use a strong authentication method like `md5` or, preferably, `scram-sha-256`. You'll also want to restrict the `address` to only the IP addresses or networks that should be allowed to connect remotely.
- **Order Matters**: The rules in `pg_hba.conf` are processed in order from top to bottom. The first rule that matches the connection attempt is used.

After making any changes to `pg_hba.conf`, you must reload the PostgreSQL configuration for the changes to take effect. You can do this using the following command:

```bash
sudo systemctl reload postgresql
```

Configuring `pg_hba.conf` correctly is essential for controlling who can access your database and how they authenticate. It's like carefully managing who gets a key to which sections of our library and what kind of identification they need to show.

## Basic Security Practices

### Creating Dedicated User Roles

Think back to our library analogy. Instead of everyone using the master key (like the `postgres` user), it's much safer to give different staff members specific keys that only allow them access to the sections they need to manage. In PostgreSQL, we achieve this by creating dedicated user roles with specific permissions.

The `postgres` user is like the 'root' or 'administrator' user – it has ultimate power. For day-to-day operations and for different applications that need to access the database, it's much better to create separate user roles with only the necessary privileges. This principle is known as the **principle of least privilege**.

Here's how you can create new user roles in PostgreSQL:

1. **Access the PostgreSQL prompt**: First, you'll need to connect to the PostgreSQL server. You can do this by switching to the `postgres` user and then using `psql`:

    ```bash
    sudo su - postgres
    psql
    ```

2. **Create a new role**: At the `postgres=#` prompt, you can create a new user role using the `CREATE ROLE` or `CREATE USER` command. `CREATE USER` is essentially a shortcut for `CREATE ROLE` with the `LOGIN` attribute.

    For example, let's say you have an application called "my_app" that needs to read and write data to a database named "my_database". You could create a dedicated user for this application like this:

    ```SQL
    CREATE USER my_app_user WITH PASSWORD 'a_strong_app_password';
    ```

    This command creates a new user named `my_app_user` and sets a password for it. Make sure to replace `'a_strong_app_password'` with a strong, unique password for this user.

    You can also create roles that don't have login privileges (these are useful for grouping permissions). For example:

    ```SQL
    CREATE ROLE my_app_read_only;
    ```

3. **Grant privileges**: Once you've created a user or role, you need to grant it the necessary privileges to interact with the database. For our `my_app_user`, we'd need to grant it privileges on the `my_database` database. For example, to allow it to connect to the database and select, insert, update, and delete data, you would use the `GRANT` command:

    ```SQL
    GRANT CONNECT ON DATABASE my_database TO my_app_user;
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO my_app_user;
    ```

    This grants the `CONNECT` privilege on the `my_database` and the `SELECT`, `INSERT`, `UPDATE`, and `DELETE` privileges on all tables within the `public` schema of that database. You can adjust these privileges based on the specific needs of the user or application.

4. **Exit PostgreSQL**: After creating the user and granting privileges, exit the PostgreSQL prompt using `\q`.

By creating dedicated user roles with specific privileges, you limit the potential damage if one of these accounts is compromised. It's like giving each library staff member a key that only opens the sections they need, so if a key is lost, the entire library isn't vulnerable.

### Limiting User Privileges

Creating separate users is only half the battle. The other crucial part is to ensure that these users only have the **minimum privileges** necessary to perform their tasks. This reinforces the principle of least privilege we touched on earlier. It's like giving our library staff keys that only open the specific rooms they need to access, and nothing more.

In PostgreSQL, you control user privileges using the `GRANT` and `REVOKE` commands. We saw an example of `GRANT` when we gave `my_app_user` permission to connect to the database and interact with tables.

Here are some important concepts related to limiting privileges:

- **Object-Level Privileges**: You can grant or revoke privileges on specific database objects, such as databases, tables, views, sequences, and functions. This allows for very fine-grained control. For example, you might grant `SELECT` privilege on a specific table but not `INSERT` or `DELETE`.
- **Schema Privileges**: Schemas are like namespaces within a database that help organize objects. You can grant privileges on schemas, such as the ability to create objects within a schema or to access objects in it. The `public` schema is the default one.
- **Role Membership**: You can create roles that represent a set of privileges and then grant membership in these roles to other users or roles. This makes it easier to manage permissions for groups of users.

**Examples**:

- To grant `SELECT` privilege on a table named `users` in the `public` schema to `my_app_user`:

    ```SQL
    GRANT SELECT ON TABLE public.users TO my_app_user;
    ```

- To revoke `INSERT` privilege on the same table from `my_app_user`:

    ```SQL
    REVOKE INSERT ON TABLE public.users FROM my_app_user;
    ```

- To create a read-only role and grant `SELECT` privilege on all tables in the `public` schema to it:

    ```SQL
    CREATE ROLE read_only_access;
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only_access;
    ```

- To then grant membership in the `read_only_access` role to the `my_app_user`:

    ```SQL
    GRANT read_only_access TO my_app_user;
    ```

By carefully managing privileges, you can significantly reduce the risk of accidental or malicious data modification or access. It's all about giving each user the right key for their specific job and no more.

### Keeping PostgreSQL Updated

Think of PostgreSQL as any other important piece of software on your server, like your operating system itself. Just as you regularly update your OS to get the latest security patches and bug fixes, it's equally important to keep your PostgreSQL installation up to date.

Software updates often include:

- **Security Patches**: These fix vulnerabilities that could be exploited by attackers. Applying these patches promptly is crucial to protect your database from known threats. It's like fixing weak spots in the walls of our library.
- **Bug Fixes**: Updates also address bugs that could cause instability or unexpected behavior in the database.
- **Performance Improvements**: Sometimes, updates include optimizations that can make your database run faster and more efficiently.
- **New Features**: While security and stability are the primary reasons for updates, they can also introduce new and useful features.

On Debian-based distributions, you can typically update your PostgreSQL installation using the standard system update commands. First, you'd update the package lists:

```bash
sudo apt update
```

Then, you can upgrade all outdated packages on your system, including PostgreSQL:

```bash
sudo apt upgrade
```

Alternatively, if you want to upgrade only the PostgreSQL packages, you can specify them:

```Bash
sudo apt upgrade postgresql postgresql-contrib
```

It's a good practice to regularly run these update commands to ensure your system, including PostgreSQL, is running the latest stable and secure versions. Many administrators even automate this process.

Keeping your software updated might seem like a routine task, but it's a fundamental aspect of maintaining a secure environment. It's like regularly checking our library for any signs of damage and making necessary repairs to keep it safe and sound.

## Firewall Configuration (UFW)

### Installing UFW

Think of a firewall as a security guard for your server, controlling who can enter and who can't. Uncomplicated Firewall (UFW) is a user-friendly front-end for managing iptables, which is the underlying firewall system in Linux. It makes it much easier to configure your firewall rules.

By default, UFW might not be installed on your Ubuntu server. You can check its status by running:

```bash
sudo ufw status
```

If it's not installed, you'll likely see a message indicating that the command isn't found. In that case, you can install UFW using the following command:

```bash
sudo apt install ufw
```

You'll be asked to confirm the installation. Just type `y` and press Enter.

Once the installation is complete, you can again check the status using `sudo ufw status`. It will likely show that the firewall is inactive.

Installing UFW is the first step in setting up this security guard for our PostgreSQL server. In the next steps, we'll configure it to allow only the necessary traffic.

### Enabling UFW

By default, even after installation, UFW is usually disabled. To enable it, you use the following command:

```bash
sudo ufw enable
```

You might see a warning about potential disruption of existing SSH connections. This is because if you're connected to your server via SSH, and you haven't explicitly allowed SSH traffic through the firewall, enabling UFW might block your own connection!

Therefore, **before enabling UFW**, it's crucial to ensure that SSH traffic is allowed. We'll do that in the next step. However, if you're absolutely sure that SSH is allowed (perhaps you've configured it previously or are working directly on the server console), you can proceed with enabling UFW.

Once enabled, UFW will start enforcing the rules you configure. If you want to disable it at any point (for example, for troubleshooting), you can use the command:

```bash
sudo ufw disable
```

For now, let's hold off on enabling UFW until we've explicitly allowed SSH access. This is like telling our security guard to make sure they don't lock themselves out of the building!

### Allowing SSH Traffic

By default, SSH typically uses port 22. To allow incoming SSH connections, you can use the following UFW command:

```bash
sudo ufw allow ssh
```

This command tells UFW to allow traffic on the SSH port. UFW is smart enough to usually know the standard port for services like SSH by name. Alternatively, you can specify the port number directly:

```bash
sudo ufw allow 22/tcp
```

This command explicitly allows TCP traffic on port 22. Both commands achieve the same result in most standard SSH configurations.

After running this command, UFW will be configured to allow incoming SSH connections. Now it's safe to enable the firewall.

Once you've allowed SSH, we need to allow traffic for PostgreSQL itself. By default, PostgreSQL listens for connections on port 5432. To allow connections to the PostgreSQL server, you can use the following UFW command:

```bash
sudo ufw allow 5432/tcp
```

This command tells UFW to allow TCP traffic on port 5432, which is the standard port for PostgreSQL. This is like telling our security guard, "And it's okay to let people in who are trying to access the library's services on this specific door!"

It's important to allow both SSH (so you can manage your server) and PostgreSQL (so applications can connect to the database).

### Enabling and Checking Firewall Status

First, let's enable UFW. As we discussed earlier, you can do this with the command:

```bash
sudo ufw enable
```

You might get a warning about disrupting existing SSH connections if you haven't allowed it yet. Since we just did that, it should be safe to proceed. Type `y` to confirm.

Once UFW is enabled, it's crucial to check its status to ensure that the rules we've added (allowing SSH and PostgreSQL) are active. You can do this with the command:

```bash
sudo ufw status
```

This command will display a list of the firewall rules. You should see entries similar to these:

```shell
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
5432/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
5432/tcp (v6)              ALLOW       Anywhere (v6)
```

This output indicates that the firewall is active and is allowing TCP traffic on port 22 (for SSH) and port 5432 (for PostgreSQL) from any IP address (`Anywhere`).

For enhanced security, especially for PostgreSQL, you might want to restrict the allowed IP addresses to only those that actually need to connect to your database. For example, if your application server has a specific IP address, you could modify the rule to only allow connections from that IP. However, for this basic setup, allowing from anywhere is a starting point.

After confirming that the firewall is active and the necessary rules are in place, your server will have a basic firewall configured to protect it. This is like having our security guard actively monitoring who comes in and out of the library through the designated doors!
