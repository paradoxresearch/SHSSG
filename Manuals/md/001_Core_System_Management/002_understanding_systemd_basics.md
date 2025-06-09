# Understanding systemd Basics

## Contents

- [Understanding systemd Basics](#understanding-systemd-basics)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [What is `systemd`?](#what-is-systemd)
    - [`systemd` Units](#systemd-units)
  - [Managing Services with `systemctl`](#managing-services-with-systemctl)
    - [List Services and Check Status](#list-services-and-check-status)
      - [Listing Services](#listing-services)
      - [Checking the Status of a Specific Service](#checking-the-status-of-a-specific-service)
        - [Interpreting Service Status Output](#interpreting-service-status-output)
  - [Understanding Unit Files](#understanding-unit-files)
    - [Common Sections in a Service Unit File](#common-sections-in-a-service-unit-file)
    - [Where Unit Files are Stored and the Precedence Rules](#where-unit-files-are-stored-and-the-precedence-rules)
  - [Creating a Simple Service Unit File](#creating-a-simple-service-unit-file)
    - [Troubleshooting Common `systemd` Issues](#troubleshooting-common-systemd-issues)
      - [The `systemd` Journal and `journalctl`](#the-systemd-journal-and-journalctl)
      - [Identifying Service Failures](#identifying-service-failures)

## Introduction

### What is `systemd`?

In the world of Linux, when your server boots up, it needs a way to get all its processes and services running in an organized fashion. That's where an "init system" comes in. Historically, many Linux distributions used a system called `SysVinit` (System V init) for this purpose.

However, `systemd` emerged as a more modern and powerful replacement. Imagine `SysVinit` as a traditional, linear assembly line, where tasks are performed one after another. `systemd`, on the other hand, is like a highly efficient, parallel processing plant. It can start services simultaneously, which significantly speeds up boot times. It also offers more robust dependency management (making sure Service A starts only after Service B is ready) and better overall process control.

This efficiency and advanced management are why `systemd` has become the standard init system for most major Linux distributions, including popular server choices like CentOS, Fedora, Debian, and Ubuntu. As a system administrator, understanding `systemd` is no longer optional; it's a fundamental skill.

### `systemd` Units

If `systemd` is the conductor, then "units" are the musical scores for each instrument. In systemd, everything it manages is represented as a "unit." A unit is essentially a configuration file that tells `systemd` what to do. These units are the fundamental building blocks for controlling various aspects of your system.

There are several types of `systemd` units, each designed for a specific purpose. Here are some of the most common ones you'll encounter as a system administrator:

- **Service units (**`.service`**)**: These are probably the most common and represent system services or applications, like a web server (Apache or Nginx) or a database (MySQL/PostgreSQL). When you start, stop, or restart an application, you're usually interacting with a service unit.

- **Target units (**`.target`**)**: Think of these as groups of other units or "runlevels" in older init systems. For example, the `multi-user.target` unit pulls in all the services needed for a typical command-line server environment. They help manage the system's state.

- **Socket units (**`.socket`**)**: These units describe network sockets or IPC (inter-process communication) sockets that `systemd` can use to activate services on demand. This means a service doesn't have to be constantly running; it can be started only when a connection to its socket is made, saving resources.

- **Device units (**`.device`**)**: These represent kernel devices. `systemd` can automatically generate these based on devices detected by the kernel.

- **Mount units (**`.mount`**)**: These define mount points for filesystems.

- **Automount units (**`.automount`**)**: Similar to mount units, but they automatically mount a filesystem only when it's accessed, rather than at boot.

- **Swap units (**`.swap`**)**: These define swap partitions or files.

- **Path units (**`.path`**)**: These units are used to activate other units (like service units) when changes occur in a specific file or directory.

- **Timer units (**`.timer`**)**: These are like `cron` jobs but integrated within `systemd`. They are used to activate other units (typically service units) at specific times or intervals.

- **Slice units (**`.slice`**)**: Used for managing and grouping processes for resource control (e.g., CPU, memory) using Linux Control Groups (cgroups).

- **Scope units (**`.scope`**)**: Similar to slice units, but typically used for external processes not started by systemd itself, such as user sessions.

Understanding that everything in `systemd` revolves around these "units" is a crucial mental model. They are the fundamental configuration pieces that `systemd` reads to understand what needs to be done.

## Managing Services with `systemctl`

The `systemctl` command is your primary interface for interacting with the `systemd` init system. Think of it as your remote control for all those `systemd` units we just discussed, especially service units. It's the command you'll use daily as a system administrator.

Here's the basic `syntax` for systemctl and some of its most common and essential commands:

- `systemctl start <unit_name>`: This command is used to start a service immediately. For example, `systemctl start apache2` would start the Apache web server.

- `systemctl stop <unit_name>`: This command is used to stop a running service immediately. For example, `systemctl stop apache2` would stop the Apache web server.

- `systemctl restart <unit_name>`: This command is used to stop and then start a service. It's often used after making configuration changes. For instance, `systemctl restart nginx` would restart the Nginx web server.

- `systemctl reload <unit_name>`: Some services can reload their configuration without a full restart. This is faster and prevents service downtime. If a service supports it, `systemctl reload <unit_name>` is generally preferred over restart.

- `systemctl enable <unit_name>`: This command enables a service to start automatically at boot. It creates a symbolic link from the unit file to the appropriate systemd startup directory. For example, `systemctl enable ssh` ensures the SSH server starts every time the system boots.

- `systemctl disable <unit_name>`: This command prevents a service from starting automatically at boot. It removes the symbolic link created by enable.

- `systemctl status <unit_name>`: This is an incredibly useful command to check the current status of a service. It tells you if it's running, if it failed, its process ID (PID), recent log entries, and more. For instance, systemctl status firewalld shows you the status of the firewall service.

Remember, replacing `<unit_name>` with the actual name of the service you want to manage (e.g., `apache2`, `nginx`, `sshd`, `mariadb`). You usually don't need to include the .service extension; `systemd` is smart enough to figure that out.

To make this more concrete, imagine you've just installed a new web server. You'd likely use `systemctl start apache2` to get it running, and then `systemctl enable apache2` to make sure it comes up after a reboot. If you ever need to troubleshoot, `systemctl status apache2` would be your first stop.

### List Services and Check Status

As a system administrator, you'll often need to quickly see what's running, what's stopped, or if there are any issues. `systemctl` provides commands to do just that:

#### Listing Services

- `systemctl list-units --type=service`: This command will show you all loaded service units on your system, regardless of their current state (active, inactive, failed, etc.). This gives you a comprehensive list.

- `systemctl list-units --type=service --state=running`: If you only want to see services that are currently active and running, you can filter the output.

- `systemctl list-units --type=service --all`: This will display all units, including those that are inactive or have failed.

- `systemctl list-unit-files --type=service`: This command shows you all installed service unit files (whether enabled or disabled) and their "enabled" status (e.g., `enabled`, `disabled`, `static`, `masked`). This helps you understand which services are configured to start at boot.

#### Checking the Status of a Specific Service

We touched on this briefly, but it's worth emphasizing:

- `systemctl status <unit_name>`: This command is your best friend for diagnosing issues with a particular service. When you run it, you'll get detailed information.

##### Interpreting Service Status Output

Let's look at a typical output from `systemctl status <unit_name>`:

```shell
● sshd.service - OpenSSH Daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-06-06 09:00:00 PDT; 1 day ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1234 (sshd)
      Tasks: 1
     Memory: 5.6M
        CPU: 123ms
     CGroup: /system.slice/sshd.service
             └─1234 /usr/sbin/sshd -D
```

Here's a breakdown of what each line typically means:

- `● sshd.service - OpenSSH Daemon`: This is the service name and a brief description. The `●` indicates a healthy status. A red `×` would indicate a failed service.
- `Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)`:
  - `loaded`: Means `systemd` has successfully read the unit file.
  - `(/usr/lib/systemd/system/sshd.service)`: Shows the path to the unit file.
  - `enabled`: Indicates if the service is configured to start automatically at boot (remember `systemctl enable`?).
  - `vendor preset: enabled`: Shows the default setting from the distribution.
- `Active: active (running) since Fri 2025-06-06 09:00:00 PDT; 1 day ago`:
  - `active (running)`: This is the most crucial part! It tells you the service is currently running successfully. Other states could be `inactive (dead)`, `failed`, `activating`, etc.
  - `since...`: Shows when the service started.
- `Docs:`: Pointers to documentation.
- `Main PID: 1234 (sshd)`: The Process ID of the main process for this service.
- `Tasks:`, `Memory:`, `CPU:`: Resource usage statistics for the service.
- `CGroup: /system.slice/sshd.service ...`: Shows the control group the service belongs to, useful for resource management.
- **Additional lines**: Often, `systemctl status` will also show the last few log entries related to the service, which is incredibly helpful for quick troubleshooting.

Understanding this output is vital for quickly diagnosing issues. If a service isn't working as expected, the `Active:` line and the recent log entries are often the first place to look.

## Understanding Unit Files

While `systemctl` is the command you use to interact with `systemd`, the actual instructions for how a service or other unit should behave are defined in text files called **unit files**. These are like the blueprints or detailed recipes that `systemd` follows.

Unit files are typically located in specific directories, and they follow a clear, structured format, making them quite readable once you know what you're looking for. A unit file is essentially a plain text file ending with the unit type extension (e.g., `.service`, `.target`, `.mount`).

Let's look at the general structure of a `.service` unit file, which is the most common type you'll encounter for applications:

A unit file is divided into sections, each enclosed in square brackets, like `[Unit]`, `[Service]`, and `[Install]`. Each section contains directives (key-value pairs) that configure specific aspects of the unit.

### Common Sections in a Service Unit File

1. `[Unit]` **Section**:
  
    This section contains generic information about the unit and its dependencies.

    - `Description=`: A human-readable description of the unit. This is what you see when you run `systemctl status`.
    - `Documentation=`: Provides links to documentation for the service (e.g., man pages, URLs).
    - `After=`: Specifies that this unit should start after the listed units. This establishes a start-up order dependency. For example, a web server might list `network.target` or `mariadb.service` here.
    - `Requires=`: Similar to `After=`, but stronger. If a unit listed in `Requires=` fails to start, this unit will also fail.
    - `Wants=`: A weaker version of `Requires=`. It expresses a desire for the listed units to start, but if they fail, this unit will still attempt to start.

    Think of `After=`, `Requires=`, and `Wants=` as setting up prerequisites, ensuring your service has everything it needs before it tries to launch.

2. `[Service]` **Section**:

    This is the core section for service units and defines how the service itself behaves.

      - `Type=`: Defines the process startup type. Common types include:
        - `simple`: The main process is specified by `ExecStart=`.
        - `forking`: The service forks into a background process (daemon). `systemd` expects the parent process to exit after the child has forked.
        - `oneshot`: A process that runs once and then exits. `systemd` waits for it to complete.
      - `ExecStart=`: The command or script executed to start the service. This is often the most important line!
      - `ExecStop=`: The command to stop the service.
      - `ExecReload=`: The command to reload the service's configuration.
      - `WorkingDirectory=`: Sets the working directory for the executed commands.
      - `User=` **/** `Group=`: Specifies the user and group under which the service's process should run for security reasons.
      - `Restart=`: Defines when and how the service should be automatically restarted if it exits. Common options: `no`, `on-failure`, `always`.
      - `TimeoutStartSec=`: The maximum time `systemd` will wait for the service to start.

3. `[Install]` **Section**:

    This section is read by `systemctl enable` and `systemctl disable` commands to manage the service's boot-time behavior.
      - `WantedBy=`: Specifies the target units (e.g., `multi-user.target`) that will pull in this service when they are started. This is how `systemctl enable` works: it creates a symlink from the target's `.wants` directory to your service unit file.

Here's a simplified example of what an `apache2.service` file might look like:

```Ini, TOML
[Unit]
Description=The Apache HTTP Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl stop
ExecReload=/usr/sbin/apachectl graceful
Restart=on-failure
User=apache
Group=apache

[Install]
WantedBy=multi-user.target
```

Understanding these sections and directives is key to customizing services or troubleshooting why they might not be starting or behaving as expected. It tells you exactly what `systemd` is trying to do.

### Where Unit Files are Stored and the Precedence Rules

`systemd` unit files are not all stored in one single directory. Instead, they are distributed across several locations, which is a key design choice that allows for flexibility and proper system administration practices. The most common directories you'll encounter are:

1. `/usr/lib/systemd/system/`:
   - This is where unit files provided by **installed software packages** typically reside. These are the "vendor" unit files.
   - You generally **should not modify** files in this directory directly, as your changes might be overwritten during package updates.

2. `/etc/systemd/system/`:
   - This is the primary location for administrator-created or modified unit files.
   - If you create a new custom service, or if you want to override settings of a vendor-provided service, you should place your unit files or override files here.
   - Files in this directory take precedence over those in `/usr/lib/systemd/system/`. This is crucial for customizing your system without breaking future updates.

3. `/run/systemd/system/`:
   - This directory is for runtime-generated unit files. These are typically created dynamically by scripts or other processes and exist only while the system is running. They are not persistent across reboots.

**Precedence Rules (How systemd chooses)**:

When `systemd` needs to load a unit, it looks in these directories in a specific order. The general rule of thumb is: **files in** `/etc` **override files in** `/run`**, which override files in** `/usr/lib`.

This means if you have an `apache2.service` file in both `/usr/lib/systemd/system/` (from the package) and `/etc/systemd/system/` (your custom version), `systemd` will use the one from `/etc/systemd/system/`. This hierarchical structure is very powerful because it allows you to:

- **Customize vendor services**: You can create a file with the same name in `/etc/systemd/system/` to override specific directives or even the entire unit file provided by the distribution.
- **Create new services**: Your custom services belong in `/etc/systemd/system/`.
- **Maintain system stability**: Your changes are kept separate from the core system files, making updates safer.

**How to Override (a common scenario)**:

Instead of copying an entire unit file from `/usr/lib` to `/etc` and modifying it (which can make future updates harder to merge), systemd offers a cleaner way to override specific directives: `systemctl edit <unit_name>`.

When you run `systemctl edit apache2.service`, systemd opens a temporary file where you can add only the directives you want to change. It automatically creates a directory like `/etc/systemd/system/apache2.service.d/` and saves your changes in a file like `override.conf` within it. This ensures that only your specific changes are applied, while the rest of the original unit file remains intact. This is the recommended way to modify existing service behavior.

Understanding these locations and the precedence order is vital for effective `systemd` management, as it dictates where you should place your configuration files and how your changes will be applied.

## Creating a Simple Service Unit File

As a system administrator, you'll often encounter situations where you need to run a custom script or a simple application as a background service. This is where creating your own `systemd` service unit file becomes incredibly useful.

Let's imagine you have a simple Python script, `my_app.py`, that just writes a timestamp to a log file every few seconds. We want to run this script as a `systemd` service.

**Our Goal**: Create a `systemd` service unit file for a simple script, enable it to start at boot, and manage it.

**Steps to Create a Custom Service**:

1. **Prepare Your Script**:

    First, let's create our dummy script. We'll put it in a common location like /opt/my_app/.

    ```Bash
    # Create the directory
    sudo mkdir -p /opt/my_app

    # Create the script file
    sudo nano /opt/my_app/my_app.py
    ```

    Inside `my_app.py`, copy the following simple Python code:

    ```Python
    #!/usr/bin/env python3
    import time
    import datetime

    log_file = "/var/log/my_app.log"

    def log_message(message):
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with open(log_file, "a") as f:
            f.write(f"[{timestamp}] {message}\n")

    if __name__ == "__main__":
        log_message("My custom application started.")
        while True:
            log_message("Still running and logging...")
            time.sleep(5) # Log every 5 seconds
    ```

    Save and exit the editor (`Ctrl+X`, `Y`, `Enter`).

    Make the script executable:

    ```Bash
    sudo chmod +x /opt/my_app/my_app.py
    ```

2. **Create the Unit File**:

    Now, we'll create our `systemd` service unit file. As we learned, custom unit files should go into `/etc/systemd/system/`. Let's call our service `my-app.service`.

    ```Bash
    sudo nano /etc/systemd/system/my-app.service
    ```

    Inside this file, paste the following content:

    ```Ini, TOML
    [Unit]
    Description=My Custom Python Application
    After=network.target

    [Service]
    ExecStart=/opt/my_app/my_app.py
    StandardOutput=syslog
    StandardError=syslog
    Restart=on-failure
    User=nobody
    Group=nobody # Or a specific user if needed, for security

    [Install]
    WantedBy=multi-user.target
    ```

    - `[Unit]`: We define a Description and set `After=network.target` to ensure the network is up before our app tries to run.
    - `[Service]`:
      - `ExecStart=/opt/my_app/my_app.py`: This is the crucial line that tells systemd what command to run to start our application.
      - `StandardOutput=syslog / StandardError=syslog`: This redirects the script's output (both standard output and standard error) to the system journal, which means you can view its logs with `journalctl` (which we'll cover soon!).
      - `Restart=on-failure`: If our script crashes, `systemd` will try to restart it automatically.
      - `User=nobody / Group=nobody`: Running services as a non-privileged user like nobody is a good security practice. You could create a dedicated user for your application if it needs specific permissions.
    - `[Install]`: `WantedBy=multi-user.target` means this service will be enabled to start when the system reaches the standard multi-user operating state (which is the default for servers).

    Save and exit the editor.

3. **Reload** `systemd` **Daemon**:

    After creating or modifying any unit file, you must tell `systemd` to reload its configuration so it becomes aware of the new or changed unit:

    ```Bash
    sudo systemctl daemon-reload
    ```

4. **Enable and Start the Service**:

    Now, enable the service so it starts on boot, and then start it immediately:

    ```Bash
    sudo systemctl enable my-app.service
    sudo systemctl start my-app.service
    ```

5. **Check the Status**:

    Finally, check the status to ensure it's running:

    ```Bash
    sudo systemctl status my-app.service
    ```

    You should see `Active: active (running)`!

6. **Verify Logs (Optional, but good practice)**:

    You can also check the log file we configured in the script:

    ```Bash
    tail -f /var/log/my_app.log
    ```

    You should see new timestamped entries appearing every 5 seconds.

This process covers the end-to-end creation and management of a simple custom systemd service. This is a very common task for system administrators.

### Troubleshooting Common `systemd` Issues

No matter how well you configure your services, issues will inevitably arise. Being able to quickly diagnose and resolve problems is what sets apart a good administrator. `systemd` provides powerful tools to help you with this, primarily through its integrated logging system, the **Journal**.

#### The `systemd` Journal and `journalctl`

Unlike older Linux systems that often logged to plain text files in `/var/log` (like syslog or messages), systemd uses a structured, centralized logging system called the **Journal**. This journal collects logs from the kernel, initrd, services, and applications. The primary command-line utility for interacting with the Journal is `journalctl`.

Here are some essential `journalctl` commands for troubleshooting:

1. `journalctl`:
   - Running `journalctl` without any arguments will display all logs from the earliest available entry to the latest. This can be overwhelming, so it's rarely used alone. The output is paginated, allowing you to scroll.

2. `journalctl -u <unit_name>`:
   - This is arguably the most useful command for troubleshooting services. It displays only the log entries related to a specific `systemd` unit.
   - **Example**: `journalctl -u nginx.service` will show all logs from the Nginx web server. If Nginx failed, this is the first place you'd look for error messages.

3. `journalctl -u <unit_name> -f`:
   - The `-f` (follow) option works like `tail -f`. It displays new log entries as they are written in real-time. This is incredibly useful when you're starting or restarting a service and want to see any immediate errors or output.
   - **Example**: You start a service, then run `journalctl -u my-app.service -f` in another terminal to watch its output.

4. `journalctl -u <unit_name> --since "YYY-MM-DD HH:MM:SS" or --since "today", --since "yesterday"`:
   - This allows you to filter logs by time. You can specify a precise timestamp or use relative terms.
   - **Example**: `journalctl -u sshd --since "2 hours ago"` or `journalctl -u apache2 --since "2025-06-01 10:00:00"`.

5. `journalctl -u <unit_name> -p err or -p warning`:
   - The `-p` (priority) option lets you filter by log level. Common levels include `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`.
   - **Example**: `journalctl -u docker -p err` to see only error messages from the Docker service.

6. `journalctl -b or journalctl -b -1`:
   - The `-b` (boot) option shows logs from the current boot. `-b -1` shows logs from the previous boot, `-b -2` from the one before that, and so on. This is vital for diagnosing issues that occur during system startup.

#### Identifying Service Failures

When a service fails, `systemctl status <unit_name>` will often report `Active: failed`. The crucial next step is to use `journalctl -u <unit_name>` to investigate why it failed. Look for:

- **Error messages**: Keywords like `error`, `failed`, `permission denied`, `address already in use`, `segmentation fault`.
- **Missing files/directories**: The service might be looking for a configuration file or executable that doesn't exist or isn't accessible.
- **Incorrect paths or commands**: Check your `ExecStart=` or other `Exec` directives in the unit file for typos or incorrect paths.
- **Dependency issues**: Look for messages indicating that a required service or resource wasn't available.

**Remember**: Always consult the `systemctl status` output first for a quick overview, and then dive into `journalctl -u <service_name>` for the detailed explanation of any failure.
