# Installing and Securing MySQL and MariaDB

## Contents

- [Installing and Securing MySQL and MariaDB](#installing-and-securing-mysql-and-mariadb)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Installation](#installation)
    - [Installing MySQL](#installing-mysql)
    - [Installing MariaDB](#installing-mariadb)
  - [Secure Your Database](#secure-your-database)
  - [Basic Security Practices](#basic-security-practices)
  - [Firewall Configuration (UFW)](#firewall-configuration-ufw)
  
## Introduction

MySQL and MariaDB are both popular open-source Relational Database Management Systems (RDBMS). Think of them as well-organized digital filing cabinets that allow you to store, manage, and retrieve data efficiently. They are crucial for many applications, from websites and web applications to data analysis and more, because they provide a reliable and structured way to handle information. MariaDB was actually developed by the original creators of MySQL in response to Oracle's acquisition of MySQL, aiming to remain open-source. For many practical purposes, they function very similarly.

Before we get started with the installation, there are a few things you should have in place on your Linux server:

- **Basic command-line knowledge**: You should be comfortable navigating the terminal and running commands.
- `sudo` **privileges**: You'll need administrative rights to install software and make system changes. This usually means you have sudo access.
- A Linux disitribution that uses the `apt` package manager.

## Installation

### Installing MySQL

To install MySQL, you would typically run these commands in your terminal:

1. **Update the package lists**: This ensures you have the latest information about available packages.

    ```bash
    sudo apt update
    ```

2. **Install the MySQL server package**: This command will download and install the necessary MySQL server files and dependencies.

    ```bash
    sudo apt install mysql-server
    ```

### Installing MariaDB

To install MariaDB, you would use these commands:

1. **Update the package lists**:

    ```bash
    sudo apt update
    ```

2. **Install the MariaDB server package**:

    ```bash
    sudo apt install mariadb-server
    ```

Once you run either of these installation commands, `apt` will handle downloading the necessary files and setting up the database server on your system. You might be prompted to confirm the installation during the process.

## Secure Your Database

Both MySQL and MariaDB installations on Debian-based systems come with a handy utility called `mysql_secure_installation` (or sometimes `mariadb-secure-installation` for MariaDB). This script will walk you through some important security settings.

**It's highly recommended to run this script immediately after installing your database server.**

To run it, simply open your terminal and type:

```bash
sudo mysql_secure_installation
```

or, if you installed MariaDB:

```bash
sudo mariadb-secure_installation
```

You'll be guided through a series of questions. Let's go through each one and discuss the recommended answers:

1. **Validating Password Plugin**: The script will first ask if you want to set up the Validate Password plugin. This plugin can enforce password complexity rules.
   - **Recommendation**: It's a good security practice to say `Y` (yes) here. You'll then be asked to choose a level of password complexity (low, medium, strong). Choose a level that suits your security needs but ensures reasonably strong passwords.

2. **Change the root password?** This is asking if you want to change the password for the root user of your MySQL/MariaDB server. The root user has full administrative privileges.
   - **Recommendation**: Definitely say `Y` (yes) here. You'll then be prompted to enter and confirm a new, strong password. Make sure it's unique and not easily guessable!

3. **Remove anonymous users?** Anonymous users are default accounts that allow anyone to log in to MySQL/MariaDB without a username.
   - **Recommendation**: Say `Y` (yes) to remove them. These are a security risk.

4. **Disallow root login remotely?** This asks if you want to prevent the root user from logging in from any computer other than the server itself.
   - **Recommendation**: Say `Y` (yes) here. Allowing remote root login is a significant security vulnerability. You should manage your database remotely using specific, less privileged user accounts.

5. **Remove test database and access to it?** By default, MySQL/MariaDB comes with a test database that is accessible to all users.
   - **Recommendation**: Say `Y` (yes) to remove it. It's unnecessary and could be a potential entry point.

6. **Reload privilege tables now?** This ensures that the changes you've made are applied immediately.
   - **Recommendation**: **Say `Y` (yes).**

After you answer all these questions, the `mysql_secure_installation` (or `mariadb-secure-installation`) script will apply the security measures you've chosen.

## Basic Security Practices

Let's talk about some basic security practices that you should follow to keep your MySQL or MariaDB database secure in the long run. These practices go beyond the initial setup and are essential for maintaining a secure environment.

1. **Create Specific User Accounts with Limited Privileges**:

    The root user in your database has full control over everything. Just like the root user on your Linux system, you should avoid using the database root user for your applications. Instead, create separate user accounts for each application or user that needs to access the database. Grant these users only the specific privileges they need to perform their tasks. For example, an application might only need SELECT, INSERT, UPDATE, and DELETE privileges on certain tables, but not the ability to create or drop tables or manage users.

    - **Why is this important?** If an application with limited privileges is compromised, the attacker's access to your database will also be limited. If they had the `root` user credentials, the damage could be much more severe.

2. **Use Strong and Unique Passwords**:

    This might seem obvious, but it's crucial. All your database users, especially administrative users, should have strong, unique passwords. Avoid using common words, personal information, or the same password across multiple services. Consider using a password manager to generate and store complex passwords securely.

    - **Why is this important?** Weak passwords are the easiest way for attackers to gain unauthorized access to your database.

3. **Keep Your Database Software Updated**:

    Like any software, MySQL and MariaDB can have security vulnerabilities. The developers regularly release updates that often include fixes for these vulnerabilities. It's essential to keep your database server updated to the latest stable version.

    - **How to update**: On Ubuntu, you would typically use the following commands:

        ```bash
        sudo apt update
        sudo apt upgrade mysql-server  # If you installed MySQL
        ```

        or

        ```bash
        sudo apt update
        sudo apt upgrade mariadb-server # If you installed MariaDB
        ```

    It's a good practice to run these commands periodically to ensure all your system packages, including the database server, are up to date.

4. **Limit Network Exposure**:

    By default, MySQL and MariaDB listen for connections on a specific network port (usually 3306). You should configure your firewall (which we'll discuss in the next step) to allow connections to this port only from trusted sources. If your application server is on the same machine, you might even configure the database to only listen for local connections.

    - **Why is this important?** Limiting network exposure reduces the attack surface and makes it harder for unauthorized individuals to even attempt to connect to your database server.

## Firewall Configuration (UFW)

Now, let's talk about configuring a firewall. A firewall acts as a security guard for your server, controlling the incoming and outgoing network traffic. It allows you to specify which connections are permitted and which are blocked.

On Ubuntu, a common and easy-to-use firewall tool is called UFW (Uncomplicated Firewall). By default, after installing MySQL or MariaDB, your database server listens for connections on a specific network port. The standard port for both MySQL and MariaDB is 3306.

It's crucial to configure your firewall to only allow connections to this port from trusted sources. For example, if your web application that needs to access the database is running on the same server, you would only need to allow connections from the server's own IP address (or from all local processes). If your web application is on a different server, you would need to allow connections from the IP address of that specific server.

Here's how you can configure UFW to allow connections on port 3306:

1. **Enable UFW (if it's not already enabled)**:

    ```bash
    sudo ufw enable
    ```

    You might get a warning about existing SSH connections. If you are connected via SSH, make sure to allow SSH connections before enabling UFW, or you might lock yourself out! You can do this with: `sudo ufw allow OpenSSH`.

2. **Allow connections on port 3306**:

    To allow connections from any IP address to port 3306 (which might be suitable if you have a public-facing application), you can use:

    ```bash
    sudo ufw allow 3306
    ```

    However, for better security, it's recommended to allow connections only from specific IP addresses or network ranges. For example, to allow connections only from the IP address `192.168.1.100`, you would use:

    ```bash
    sudo ufw allow from 192.168.1.100 to any port 3306
    ```

    To allow connections from an entire network range (e.g., `192.168.1.0/24`), you would use:

    ```bash
    sudo ufw allow from 192.168.1.0/24 to any port 3306
    ```

3. **Verify UFW status**:

    To check if the rule has been added correctly and to see the current status of UFW, you can use:

    ```Bash
    sudo ufw status
    ```

    This will show you a list of the rules that are currently active. You should see a rule allowing traffic on port 3306 from the specified sources.

**Important Considerations**:

- If your application and database are on the same server, you might not even need to open port 3306 to external networks. MySQL and MariaDB can be configured to listen only on the local loopback interface (127.0.0.1), which means they will only accept connections originating from the same machine. This is a very secure setup.
- If you need to access your database from a remote machine (e.g., for administration), make sure to allow connections only from the specific IP address of that machine.
