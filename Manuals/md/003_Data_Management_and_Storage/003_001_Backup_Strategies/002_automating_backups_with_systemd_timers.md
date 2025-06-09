# Automating Backups with Systemd Timers

## Contents

- [Automating Backups with Systemd Timers](#automating-backups-with-systemd-timers)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Systemd Timers](#understanding-systemd-timers)
  - [Creating Backup Scripts](#creating-backup-scripts)
  - [Setting Up Systemd Timer Units](#setting-up-systemd-timer-units)
    - [`.service` Unit File](#service-unit-file)
    - [`.timer` Unit File](#timer-unit-file)
    - [Enabling, Starting, and Checking the Status of the Timer and Service Units](#enabling-starting-and-checking-the-status-of-the-timer-and-service-units)
  - [Monitoring and Maintenance](#monitoring-and-maintenance)
    - [Monitoring](#monitoring)
    - [Maintenance](#maintenance)

## Introduction

Think of systemd timers as the evolved, more sophisticated cousin of the traditional cron jobs you might be familiar with in Linux. Both help you schedule tasks to run automatically at specific times or intervals. However, systemd timers offer some cool advantages, especially for server administration.

One key benefit is their tight integration with systemd, the system and service manager in modern Linux distributions like Ubuntu. This means timers are managed by the same system that controls your server's services, making them more robust and easier to manage. For example, you can easily check the status and logs of your timers using the same `systemctl` and `journalctl` commands you use for other services. This centralized management can simplify your administrative tasks.

Another advantage is better event-based scheduling. While cron primarily works on fixed time intervals (like every day at 3 AM), systemd timers can be triggered by various events, such as system boot, specific unit activation, or even file system events. This flexibility can be really useful for backups – for instance, you could potentially trigger a backup after a major system update completes.

### Understanding Systemd Timers

Let's look at the two main players involved when you set up an automated task like a backup: the timer unit file and the service unit file. They work together like a director and an actor in a play.

1. **The Timer Unit File (.timer)**: This file is like the director. It's responsible for defining when a specific action should occur. You configure the schedule in this file – for example, "run this task every day at 2 AM" or "run this task 1 hour after the system boots." The timer unit doesn't actually do the work itself; instead, it tells systemd when to activate a corresponding service unit. These files typically end with the `.timer` extension.

2. **The Service Unit File (.service)**: This file is like the actor. It defines what action should be performed when the timer triggers. In our case, this file will contain the instructions on how to run your backup script. It specifies things like which user should run the script, the working directory, and most importantly, the command to execute your backup script. These files usually end with the `.service` extension.

So, to automate your backups, you'll need to create both a `.timer` file to define the backup schedule and a `.service` file to define what the backup process actually entails (running your backup script). The `.timer` file will then be linked to the `.service` file, telling systemd to run the service according to the schedule defined in the timer.

---

Now, let's discuss how you actually tell the timer when to trigger the service. This is where the scheduling options within the `.timer` file come into play.

Systemd offers several ways to define a schedule, giving you a lot of flexibility for your backup automation needs. Here are a few common and useful options:

- `OnBoot=`: This option specifies a delay after the system has finished booting before the timer is triggered for the first time. For example, `OnBoot=5min` would start the associated service 5 minutes after the server boots up. This can be useful if your backup relies on other services being fully operational after a reboot.

- `OnUnitActiveSec=`: This option defines a time interval relative to when the associated service was last activated and finished. For instance, if your backup service takes about 30 minutes to run, you could set `OnUnitActiveSec=12h` to run the backup every 12 hours after the previous backup completed. This is useful for ensuring a certain interval between backups, regardless of how long the backup process takes.

- `OnCalendar=`: This is a very powerful and flexible option that lets you define calendar-like schedules, similar to cron but with some extensions. You can specify times, dates, weekdays, etc., using a specific format. For example:
  - `OnCalendar=daily` would run the service every day at midnight.
  - `OnCalendar=weekly` would run it on the first day of the week (usually Sunday) at midnight.
  - `OnCalendar=Mon,Wed,Fri 2:00` would run it at 2:00 AM on Mondays, Wednesdays, and Fridays.
  - `OnCalendar=*-*-15 03:30` would run it at 3:30 AM on the 15th of every month.

For automating backups, `OnCalendar=` is often the most convenient and commonly used option as it allows you to set specific times and days for your backups to occur.

## Creating Backup Scripts

The next crucial step is to think about the backup script itself. This script will contain the actual commands to copy your important data to a backup location.

When creating a backup script, there are a few essential things to consider:

1. **What data do you need to back up?** This seems obvious, but it's important to be specific. Are you backing up entire directories like `/etc`, `/home`, or specific databases? Knowing exactly what needs to be saved will help you define the commands in your script.

2. **Where will you store the backups?** This is also critical for data safety. Ideally, backups should be stored in a different location than the original data. This could be a separate hard drive, a network-attached storage (NAS) device, or even a remote server. The destination will influence how you structure the backup commands in your script.

3. **Basic Error Handling**: While we won't delve into complex error handling right now, it's good practice to include some basic checks in your script. For example, you might want to ensure that the backup destination is mounted or that the backup command completed successfully. You can often do this by checking the exit code of the commands you run in the script.

4. **Choosing the right tools**: Linux offers several powerful command-line tools for creating backups. Two of the most common are:
    - `tar` **(Tape Archive)**: This utility can archive multiple files into a single file (often called a "tarball") and can also compress these archives using tools like gzip or bzip2 to save space. It's very versatile for creating full or incremental backups.
    - `rsync` **(Remote Sync)**: This tool is excellent for synchronizing files and directories between two locations. It's particularly efficient for incremental backups because it only copies the changes made since the last backup.

For a basic automated backup using systemd, either `tar` or `rsync` can be a good starting point.

Let's say, for example, you want to back up the `/etc` directory to a directory named `/backup/etc_backup` on the same server using `tar` and gzip compression. A very basic command for this would be:

```bash
tar -czvf /backup/etc_backup/etc_$(date +%Y%m%d).tar.gz /etc
```

In this command:

- `tar` is the command itself.
- `-c` tells `tar` to create an archive.
- `-z` tells `tar` to compress the archive using gzip.
- `-v` makes `tar` list the files it's processing (verbose output).
- `-f /backup/etc_backup/etc_$(date +%Y%m%d).tar.gz` specifies the name of the archive file. `$(date +%Y%m%d)` is a neat trick to include the current date in the filename, so you get a new backup file each day (e.g., `etc_20250515.tar.gz`).
- `/etc` is the directory you want to back up.

## Setting Up Systemd Timer Units

### `.service` Unit File

Let's first discuss creating the `.service` **unit file. This file tells systemd how to run your backup script. Let's assume you've saved the `tar` command we discussed earlier (or a similar `rsync` command) in a script file, for example, `/usr/local/bin/backup_etc.sh`.

Here's a basic example of what your `backup_etc.service` file might look like:

```Ini, TOML
[Unit]
Description=Backup the /etc directory

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/backup_etc.sh
```

**Let's break down what each line does**:

- `[Unit]`: This section contains general information about the service.
  - `Description=Backup the /etc directory`: This is a human-readable description of what the service does.

- `[Service]`: This section defines the behavior of the service.
  - `User=root`: This specifies that the script should be run as the root user. Since backing up system directories often requires elevated privileges, this is common. Be cautious when running scripts as root and ensure your script is secure.
  - `Group=root`: Similarly, this specifies the group under which the script will run.
  - `ExecStart=/usr/local/bin/backup_etc.sh`: This is the crucial line! It tells systemd the exact command to execute when this service is started. In this case, it's the path to your backup script.

You would typically save this file in the `/etc/systemd/system/` directory. It's important that the script `/usr/local/bin/backup_etc.sh` exists and has execute permissions. You can give it execute permissions using the command `chmod +x /usr/local/bin/backup_etc.sh`.

### `.timer` Unit File

Now, let's create a timer unit file named `backup_etc.timer` (it's good practice to give it the same base name as the service it triggers). Here's a basic example that will run our `backup_etc.service` daily at 2:00 AM:

```Ini, TOML
[Unit]
Description=Daily backup of the /etc directory
After=network.target

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

Let's break down this file section by section:

- `[Unit]`: This section contains general information about the timer.
  - `Description=Daily backup of the /etc directory`: A human-readable description of the timer's purpose.
  - `After=network.targe`t: This directive ensures that the timer only starts after the network is up and running. This might be important if your backup destination is on a network.

- `[Timer]`: This is the crucial section where you define the scheduling.
  - `OnCalendar=daily`: As we discussed earlier, this tells systemd to trigger the associated service (in this case, `backup_etc.service`) every day at midnight (which is the default time for `daily`). You could be more specific, like `OnCalendar=02:00` to run it at 2:00 AM.
  - `Persistent=true`: This is a handy option. If your server is down at the time the timer was supposed to trigger, `Persistent=true` will make systemd run the service the next time the system boots up. This helps ensure your backups don't get skipped due to downtime.

- `[Install]`: This section defines how the timer should be enabled.
  - `WantedBy=timers.target`: This tells systemd that this timer should be started when the `timers.target` unit is activated during system boot. This is the standard way to enable timers.

Just like the `.service` file, you'll save this `backup_etc.timer` file in the `/etc/systemd/system/` directory.

### Enabling, Starting, and Checking the Status of the Timer and Service Units

You've now created both the `.service` file (defining the backup action) and the `.timer file` (defining the backup schedule). The next step is to learn how to tell systemd to actually use these files. This is where the `systemctl` command comes in.

Here are the essential `systemctl` commands you'll use to manage your systemd timer:

1. **Enabling the timer**: Before a timer can run, you need to tell systemd to enable it. This essentially links the timer to the `timers.target` we saw in the `[Install]` section of the `.timer` file. You enable the timer using the following command:

    ```bash
    sudo systemctl enable backup_etc.timer
    ```

    This command will likely create a symbolic link in a systemd configuration directory, telling systemd to start this timer during the boot process.

2. **Starting the timer**: Enabling the timer doesn't immediately start it. To start the timer right away (without needing to reboot), you use the `start` command:

    ```bash
    sudo systemctl start backup_etc.timer
    ```

    After running this, systemd will activate the timer, and it will then wait for its scheduled time to trigger the associated `backup_etc.service`.

3. **Checking the status of the timer**: To see if your timer is active and to find out when it's scheduled to run next, you can use the `status` command:

    ```bash
    systemctl status backup_etc.timer
    ```

    The output of this command will give you information like whether the timer is loaded, active, when it last ran (if it has), and when it's scheduled to run next. Pay close attention to the "Active:" line and the "Next Elapse:" line.

4. **Checking the status of the service**: After the timer has triggered, you can check the status of the associated service to see if the backup ran successfully. Use the same `status` command but for the service unit:

    ```bash
    systemctl status backup_etc.service
    ```

    This will show you if the service started, if it exited without errors, and any relevant output or error messages.

5. **Stopping and disabling the timer**: If you need to temporarily stop the timer, you can use:

    ```bash
    sudo systemctl stop backup_etc.timer
    ```

    To prevent the timer from starting on subsequent reboots, you need to disable it:

    ```bash
    sudo systemctl disable backup_etc.timer
    ```

Remember to run these commands with `sudo` if you're not logged in as the root user, as they involve modifying systemd configurations.

## Monitoring and Maintenance

### Monitoring

Once you've automated your backups with systemd timers, it's essential to ensure they are running correctly and that your backups are actually useful when you need them.

One of the significant advantages of using systemd is its integrated logging system, the journal. You can easily check the logs of your timer and service units to see if the backups are running as expected and if there were any errors.

To view the logs for your `backup_etc.service`, you can use the `journalctl` command followed by the unit name:

```bash
journalctl -u backup_etc.service
```

This command will display the logs specifically for the `backup_etc.service`. You can see when it started, when it finished, and any output or error messages that might have been generated by your backup script.

Similarly, you can check the logs for the timer unit itself:

```bash
journalctl -u backup_etc.timer
```

This will show you when the timer activated the service and any messages related to the timer's operation.

Using `journalctl` is crucial for:

- `Verifying successful backups`: You can check the logs for confirmation that your backup script ran without errors.
- `Identifying issues`: If a backup fails, the logs will often provide clues about what went wrong, such as permission errors, insufficient disk space, or problems with the backup command itself.
- `Troubleshooting`: When setting up your automated backups, you might encounter issues. The logs are your best friend for diagnosing and resolving these problems.

### Maintenance

Just setting up the automation isn't the end of the story; regular maintenance is key to ensuring your backups remain reliable and effective.

Here are a couple of crucial maintenance aspects to keep in mind:

1. **Log Rotation**: Over time, the logs generated by your backup service can grow and consume disk space. It's good practice to set up log rotation. Systemd's journald usually handles log rotation automatically based on size and time limits configured in `/etc/systemd/journald.conf`. You might want to review these settings to ensure they are appropriate for your server's disk space and your need to retain backup logs for a certain period.

2. **Testing Backup Restoration**: This is arguably the most critical maintenance task. Regularly testing your backup restoration process is the only way to be sure that your backups are actually working and that you can recover your data in case of a failure. This involves:
   - Selecting a backup to restore.
   - Identifying a safe location to restore the data (ideally not overwriting your live data initially).
   - Using the appropriate command (e.g., `tar -xzvf` for a `.tar.gz` archive or `rsync` to copy files back) to restore the data.
   - Verifying that the restored data is complete and correct.

    Make it a habit to perform test restores periodically. This will give you confidence in your backup strategy and help you identify any potential issues with your backup process or the integrity of your backup files.

While these are just a couple of key maintenance points, they are essential for a robust backup strategy.
