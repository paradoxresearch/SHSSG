# Configuring `logrotate`

## Contents

- [Configuring `logrotate`](#configuring-logrotate)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Examining `logrotate` Configuration](#examining-logrotate-configuration)
    - [Common `logrotate` Configuration Directives](#common-logrotate-configuration-directives)
  - [Creating a Custom `logrotate` Configuration](#creating-a-custom-logrotate-configuration)
  - [Testing and Debugging `logrotate`](#testing-and-debugging-logrotate)
    - [Using `logrotate -d` for Debugging](#using-logrotate--d-for-debugging)
  - [Advanced `logrotate` Concepts](#advanced-logrotate-concepts)
    - [Prerotate and Postrotate Scripts](#prerotate-and-postrotate-scripts)
    - [`size` and `dateext` Directives](#size-and-dateext-directives)
    - [`logrotate` and Cron Integration](#logrotate-and-cron-integration)

## Introduction

In server administration, log files are essential records of system events, application activities, and potential errors. Effective log management is a cornerstone of maintaining a stable, secure, and well-performing server environment.

Key reasons why log management is critical:

1. **Resource Conservation**: Log files can grow indefinitely if unmanaged, consuming valuable disk space. This can lead to system instability or failure if the disk becomes full.
2. **Performance Optimization**: Accessing and analyzing extremely large log files can be time-consuming and resource-intensive, hindering efficient troubleshooting.
3. **Problem Diagnosis** (Troubleshooting): When system issues or application errors occur, logs provide the primary data source for diagnosing the root cause. Organized logs facilitate quicker identification of relevant information.
4. **Security and Auditing**: Logs are vital for security monitoring, recording access attempts, system changes, and potential security incidents. They form the basis for security audits and forensic analysis.
5. **Regulatory Compliance**: Many organizations are subject to regulations that mandate the retention and management of log data for specific periods.

**The** `logrotate` **Utility:**

`logrotate` is a standard Linux utility designed to automate the administration of log files. Its primary function is to systematically process log files by rotating, compressing, deleting, and optionally mailing them. This automation ensures that log management tasks are performed consistently without manual intervention.

## Examining `logrotate` Configuration

The behavior of logrotate is controlled by configuration files located in specific directories:

1. **Primary Configuration File**: `/etc/logrotate.conf`
   - This file establishes the global default settings and operational parameters for `logrotate`. Directives set in this file apply to all log rotation processes unless explicitly overridden by more specific configurations.

2. **Service-Specific Configuration Directory**: `/etc/logrotate.d/`
   - This directory contains individual configuration files for specific applications or services (e.g., web servers like Apache or Nginx, system services like `syslog` or `apt`). Each file within `/etc/logrotate.d/` defines how the logs for that particular service should be managed. These service-specific configurations can augment or override the global settings defined in `/etc/logrotate.conf`.

Essentially, /etc/logrotate.conf provides the general framework, while files in /etc/logrotate.d/ provide tailored instructions for individual services.

### Common `logrotate` Configuration Directives

With an understanding of where `logrotate` gets its instructions, let's explore the actual commands, or directives, that you'll use within these configuration files to define how logs are managed. These directives act as specific instructions for `logrotate`.

 Here are some directives with explicit examples as they would appear in a file like `/etc/logrotate.d/apache2` or `/etc/logrotate.conf`

```shell
# Example inside of /etc/logrotate.conf

/var/log/apache2/*.log {
    # Directives for Apache web server logs
}
```

Here are some of the most frequently used and essential `logrotate` directives:

- `rotate <count>`: This directive specifies how many old log files should be kept before they are removed.
  - Example: `rotate 4` means `logrotate` will keep the current log file and up to four older, rotated versions. The fifth oldest rotated log file will be deleted.
  - Example in config:

    ```shell
    /var/log/myapp/access.log {
        rotate 4
        # ... other directives
    }
    ```

    > **Explanation**: This means logrotate will keep the current access.log and the 4 most recent rotated versions (e.g., `access.log.1`, `access.log.2`, ..., `access.log.4`). The 5th oldest will be removed.

- `daily` / `weekly` / `monthly` / `yearly`: These directives define the frequency of log rotation.
  - `daily`: Logs will be rotated once every day.
  - `weekly`: Logs will be rotated once every week.
  - `monthly`: Logs will be rotated once every month.
  - `yearly`: Logs will be rotated once every year.
  - **Example**: If a log file is rotated `daily` and `rotate 7` is set, you will have the current log file and seven previous daily logs, effectively keeping a week's worth of data.
  - **Example in config (Daily)**:

    ```shell
    /var/log/apache2/access.log {
        daily
        rotate 30
        # ... other directives
    }
    ```

    > **Explanation**: The access.log will be rotated every day. With rotate 30, you would keep 30 days of rotated log files.

  - **Example in config (Monthly)**:

    ```shell
    /var/log/database/error.log {
        monthly
        rotate 6
        # ... other directives
    }
    ```

    > **Explanation**: The `error.log` for the database will be rotated once a month, and 6 months of historical log files will be retained.

- `compress` / `nocompress`: These directives control whether rotated log files are compressed.
  - `compress`: Rotated log files will be compressed using gzip (or a similar utility) to save disk space. They will typically have a .gz extension.
  - `nocompress`: Rotated log files will not be compressed.
  - **Example**: `compress` is highly recommended for most logs to conserve disk space, especially if you are keeping many rotations.
  - **Example in config (Compression)**:

    ```shell
    /var/log/syslog {
        weekly
        rotate 4
        compress
        # ... other directives
    }
    ```

    > **Explanation**: After `syslog` is rotated weekly, the old `syslog.1` (and subsequent numbers) will be compressed to `syslog.1.gz`, `syslog.2.gz`, etc., saving disk space.

  - **Example in config (No Compression)**:

    ```shell
    /var/log/uncompressed.log {
        daily
        nocompress
        # ... other directives
    }
    ```

    > **Explanation**: The `uncompressed.log` will be rotated daily, but the old files will remain uncompressed. This might be used if another tool expects logs to be uncompressed.

- `missingok`: This directive tells `logrotate` not to output an error if a log file is missing. This is useful for optional log files that may not always exist.
  - **Example**: If a log file for a rarely used service might not be present, `missingok` prevents `logrotate` from reporting an error unnecessarily.
  - **Example in config**:

    ```shell
    /var/log/optional-app/activity.log {
        weekly
        missingok
        # ... other directives
    }
    ```

    > **Explanation**: If `/var/log/optional-app/activity.log` is not present when `logrotate` runs, it will simply skip this entry without generating an error message.

- `notifempty`: This directive prevents `logrotate` from rotating a log file if it is empty.
  - **Example**: If an application hasn't generated any new log entries since the last rotation, `notifempty` saves resources by skipping an unnecessary rotation.
  - **Example in config**:

    ```shell
    /var/log/cron.log {
        monthly
        notifempty
        # ... other directives
    }
    ```

    > **Explanation**: If `cron.log` has had no new entries since the last rotation, logrotate will not create a new empty log file, saving a trivial operation.

- `create <mode> <owner> <group>`: This directive is used in conjunction with rotation. After the original log file is rotated, `create` instructs `logrotate` to create a new, empty log file with specified permissions, owner, and group.
  - **Example**: `create 0640 root adm` would create the new log file with read/write permissions for the owner (root), read-only for the group (adm), and no permissions for others.
  - **Example in config**:

    ```shell
    /var/log/nginx/access.log {
        daily
        rotate 14
        create 0640 www-data adm
        # ... other directives
    }
    ```

    > **Explanation**: After `nginx/access.log` is rotated, `logrotate` will create a brand new `/var/log/nginx/access.log` file with permissions `rw-r-----`, owned by user `www-data` and group `adm`. This ensures the web server can continue writing to a fresh log.

- `olddir <directory>`: This directive specifies a separate directory where old, rotated log files should be moved. This helps keep the primary log directory clean.
  - **Example**: `olddir /var/log/apache2/old_logs` would move rotated Apache logs into the old_logs subdirectory instead of keeping them in /var/log/apache2. This makes navigating the current log directory simpler.
  - **Example in config**:

    ```shell
    /var/log/webapp/error.log {
        weekly
        olddir /var/log/webapp/archive
        rotate 8
        compress
        # ... other directives
    }
    ```

    > **Explanation**: When `error.log` is rotated, the old `error.log.1.gz`, `error.log.2.gz`, etc., will be moved into the `/var/log/webapp/archive` directory, keeping the main `/var/log/webapp/` directory cleaner. The archive directory must exist and be writable by `logrotate`.

## Creating a Custom `logrotate` Configuration

To illustrate how these directives coalesce into a functional configuration, let's create a hypothetical scenario. Imagine you've developed a custom web application that logs its activities to `/var/log/mywebapp/access.log` and `/var/log/mywebapp/error.log`. You want to ensure these logs are properly managed to prevent disk space issues and to maintain a reasonable history for troubleshooting.

Here's a sample logrotate configuration file that you would typically place in `/etc/logrotate.d/mywebapp`:

```Bash
# Configuration for My Web Application Logs
/var/log/mywebapp/access.log
/var/log/mywebapp/error.log
{
    daily                 # Rotate logs daily
    rotate 7              # Keep 7 rotations (i.e., 7 days of logs)
    compress              # Compress old log files to save space
    missingok             # Don't throw an error if the log file is missing
    notifempty            # Don't rotate if the log file is empty
    create 0640 www-data adm # Create a new log file with specific permissions after rotation

    # This is a post-rotation script. It runs after the logs have been rotated.
    # In this case, it sends a signal to the web application to reopen its log files.
    # This is crucial so the application continues logging to the new, empty file
    # instead of the old, rotated one.
    postrotate
        /usr/bin/systemctl reload mywebapp.service > /dev/null || true
    endscript
}
```

Let's break down this example:

- `/var/log/mywebapp/access.log`  
  `/var/log/mywebapp/error.log`

  These lines specify the log files that this particular configuration block will manage. You can list multiple log files that share the same rotation policy within a single block.

- `daily`: This directive ensures that `logrotate` will check these logs once every day and perform a rotation if the conditions are met (e.g., it hasn't been rotated that day yet).

- `rotate 7`: After rotation, `logrotate` will keep the current log file and the seven most recent compressed archives of these logs. Older archives will be removed.

- `compress`: All rotated log files (e.g., `access.log.1`, `error.log.2`) will be compressed into `.gz` files (e.g., `access.log.1.gz`, `error.log.2.gz`) to save disk space.

- `missingok`: If, for some reason, one of these log files doesn't exist when `logrotate` runs, it will simply skip it without generating an error message.

- `notifempty`: If a log file is empty (meaning no new entries have been written since the last rotation), `logrotate` will not perform an unnecessary rotation.

- `create 0640 www-data adm`: After `access.log` and `error.log` are rotated and effectively moved aside, logrotate will create new, empty files named `access.log` and `error.log`. These new files will have permissions set to `0640` (read/write for owner, read-only for group, no access for others), owned by the user `www-data` and the group `adm`. This is crucial because your web application (which likely runs as `www-data`) needs to be able to write to these new log files.

- `postrotate ... endscript`: This is a powerful feature. Any commands placed between `postrotate` and `endscript` will be executed after the log files have been rotated. In this example:
  - `/usr/bin/systemctl reload mywebapp.service`: This command tells the system's service manager (`systemd`) to reload the `mywebapp.service`. For many applications, this action prompts the application to "reopen" its log files, causing it to start writing to the newly created, empty log file instead of continuing to write to the old, rotated one. Without this, your application might continue writing to the old log, defeating the purpose of rotation.
  - `> /dev/null || true`: This part suppresses any output from the `systemctl` command and ensures that `logrotate` doesn't report an error if the reload command itself fails (which might happen if the service isn't running, though in a real scenario you'd want to ensure the service is always operational).

This complete configuration demonstrates how various directives work together to achieve a specific log management policy for your application.

## Testing and Debugging `logrotate`

When you create or modify a `logrotate` configuration file, it's crucial to test it before deploying it to a production environment. This prevents unexpected behavior, such as logs not rotating correctly or applications failing to write to their new log files.

The primary tool for testing your `logrotate` configuration is the `logrotate` command itself, specifically with the `-d` (debug) option.

### Using `logrotate -d` for Debugging

The `logrotate -d` command performs a "dry run." It tells you what `logrotate` would do if it were to run, without actually making any changes to your files. This is invaluable for verifying your configuration.

Here's how you'd typically use it:

```Bash
sudo logrotate -d /etc/logrotate.d/mywebapp
```

Let's break down this command:

- `sudo`: `logrotate` typically requires root privileges to operate on system log files, so you'll need to run it with `sudo`.
- `logrotate`: The command itself.
- `-d`: This is the debug option. It simulates the rotation process without actually modifying any log files or creating new ones.
- `/etc/logrotate.d/mywebapp`: This specifies the specific configuration file you want to test. If you omit this and just run `sudo logrotate -d /etc/logrotate.conf`, it will process all configurations, which can produce a lot of output. For testing a new or modified application-specific configuration, targeting just that file is more efficient.

When you run this command, `logrotate` will output a detailed description of the actions it would take based on your configuration. This output includes:

- Which log files it identifies.
- Which directives it applies (e.g., `daily`, `compress`, `rotate 7`).
- Whether it determines a rotation is needed.
- What filenames would be used for old logs (e.g., `access.log.1`).
- Whether postrotate scripts would be executed.

**Example Output (Simplified)**:

```shell
reading config file /etc/logrotate.d/mywebapp
running logrotate program /usr/sbin/logrotate
considering log /var/log/mywebapp/access.log
  log needs rotating
considering log /var/log/mywebapp/error.log
  log needs rotating
rotating log /var/log/mywebapp/access.log
...
compressing log /var/log/mywebapp/access.log.1
creating new /var/log/mywebapp/access.log mode 0640 owner www-data group adm
running postrotate script
...
```

By examining this output, you can confirm that `logrotate` interprets your configuration as intended. Look for messages indicating that logs "need rotating," that files are being "compressed," and that "creating new" log files and "running postrotate script" commands are correctly identified. If there are any errors in your configuration syntax, `logrotate` will usually report them here.

It's also important to remember that `logrotate` is typically executed daily by a `cron` job (usually `/etc/cron.daily/logrotate`). The `-d` option helps you preview the next run.

## Advanced `logrotate` Concepts

### Prerotate and Postrotate Scripts

While we briefly touched upon `postrotate` in our practical example, it's worth exploring both `prerotate` and `postrotate` in more detail as they offer powerful control over the log rotation process. These directives define scripts that `logrotate` executes at specific points during the rotation.

- `prerotate` / `endscript`:
  - **Purpose**: Commands placed between `prerotate` and `endscript` are executed before logrotate begins the actual rotation of the log file.
  - **Common Use Case**: This is ideal for tasks that need to be completed before the log file is moved or renamed. For example, you might need to stop an application gracefully if it writes continuously and cannot handle log file renames while running. Or, you might want to flush any buffered log data to the file system.
  - **Example in config**:

    ```shell
    /var/log/mycriticalapp/app.log {
        daily
        prerotate
            /usr/bin/systemctl stop mycriticalapp.service
        endscript
        # ... other directives
        postrotate
            /usr/bin/systemctl start mycriticalapp.service
        endscript
    }
    ```

    > In this example, the `mycriticalapp` service is stopped before log rotation to ensure data integrity, and then restarted afterward.

- `postrotate` / `endscript`:
  - **Purpose**: Commands placed between `postrotate` and `endscript` are executed after the log file has been rotated and, if specified, compressed and moved.
  - **Common Use Case**: This is essential for signaling applications to reopen their log files, as shown in our previous example. Without this, an application might continue writing to the old, now-rotated log file, or stop logging altogether if it's unable to write to a file that has been moved.
  - **Example in config**: (Reiterating for emphasis, as this is very common)

    ```shell
    /var/log/nginx/access.log {
        weekly
        postrotate
            /usr/bin/nginx -s reopen
        endscript
        # ... other directives
    }
    ```

    > Here, after Nginx's access log is rotated, the nginx -s reopen command tells the Nginx web server to gracefully close its old log file and start writing to the new one that logrotate created.

### `size` and `dateext` Directives

These two directives provide more granular control over when logs are rotated and how rotated files are named.

- `size <size_threshold>`:
  - **Purpose**: This directive tells `logrotate` to rotate a log file only if it exceeds a specified size, regardless of the rotation frequency (`daily`, `weekly`, etc.).
  - **Units**: You can specify size in bytes, kilobytes (`k)`, megabytes (`M`), or gigabytes (`G`).
  - **Interaction with Frequency**: If both `size` and a frequency (e.g., `daily`) are specified, rotation occurs when either the size threshold is met or the frequency period has passed. The `size` directive effectively sets a maximum size between rotations.
  - **Example in config**:

    ```shell
    /var/log/largeapp/debug.log {
        size 100M        # Rotate if the log file reaches 100 MB
        rotate 5
        compress
    }
    ```

    > This configuration ensures that `debug.log` is rotated as soon as it hits 100MB, even if a daily rotation hasn't occurred yet. It will keep 5 compressed historical files.

- `dateext`:
  - **Purpose**: This directive changes the naming convention for rotated log files. Instead of appending a simple number (e.g., `access.log.1`, `access.log.2`), it appends a date stamp.
  - **Format**: The date format is typically `YYYYMMDD` (e.g., `access.log-20250521`).
  - **Benefit**: Using `dateext` makes it much easier to identify when a specific log file was rotated, which can be invaluable for historical analysis and troubleshooting.
  - **Example in config**:

    ```shell
    /var/log/mail/maillog {
        monthly
        dateext
        rotate 12
        compress
    }
    ```

    > With this, after monthly rotation, you'd see files like `maillog-20240501.gz`, `maillog-20240601.gz`, etc., clearly indicating the month each log covers.

These advanced directives give you finer control over `logrotate`'s behavior, allowing you to tailor log management precisely to your application's needs and compliance requirements.

### `logrotate` and Cron Integration

On Linux systems, routine automated tasks are typically handled by a daemon called **cron**. `cron` allows you to schedule commands or scripts to run automatically at specified intervals (e.g., daily, weekly, monthly, or at specific times).

`logrotate` is almost universally integrated with `cron` to ensure that log files are rotated regularly and automatically. You won't typically need to manually set up a `cron` job for `logrotate` on your server, as it's usually pre-configured during installation.

The most common setup involves a `cron` job that runs `logrotate` daily. You can typically find this in the `/etc/cron.daily/` directory. If you list the contents of that directory, you'll likely see a file named logrotate:

```Bash
ls -l /etc/cron.daily/logrotate
```

This file is a simple script that executes the `logrotate` command, usually pointing to the main configuration file:

```Bash
#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit $EXITVALUE
```

**Key takeaways from this integration**:

- **Automation**: This `cron` job ensures that `logrotate` runs automatically, typically once a day, processing all the configurations found in `/etc/logrotate.conf` and its included directory /etc/logrotate.d/.
- **No Manual Intervention (Usually)**: As a system administrator, your primary task regarding `logrotate` will be creating or modifying the configuration files in `/etc/logrotate.d/` for your specific applications, rather than scheduling the `logrotate` command itself.
- **System Standard**: This daily execution by `cron` is a standard and robust mechanism for maintaining log hygiene across the system.

Understanding this integration helps you grasp why `logrotate` operates seamlessly in the background without constant manual oversight.
