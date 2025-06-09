# Setting Up Local Backups with `rsync`

## Contents

- [Setting Up Local Backups with `rsync`](#setting-up-local-backups-with-rsync)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Basic Backup Concepts and `rsync`](#understanding-basic-backup-concepts-and-rsync)
  - [Installing `rsync` and Basic Syntax](#installing-rsync-and-basic-syntax)
    - [Basic Syntax](#basic-syntax)
    - [`rsync` Options](#rsync-options)
  - [Performing Full Backups with `rsync`](#performing-full-backups-with-rsync)
    - [Important Considerations when Choosing a Backup Destination for Your Local Backups](#important-considerations-when-choosing-a-backup-destination-for-your-local-backups)
  - [Implementing Incremental Backups with `rsync`](#implementing-incremental-backups-with-rsync)
    - [A Practical Example of Incremental Backups](#a-practical-example-of-incremental-backups)
  - [Automating Backups with `cron`](#automating-backups-with-cron)
    - [Example Automated Backup Script Scheduled with `cron`](#example-automated-backup-script-scheduled-with-cron)
  - [Backup Strategies and Best Practices](#backup-strategies-and-best-practices)
    - [Backup Strategies](#backup-strategies)
    - [Best Practices](#best-practices)

## Introduction

### Understanding Basic Backup Concepts and `rsync`

Think of backups as your server's safety net. Imagine accidentally deleting important configuration files or experiencing a hard drive failure. Without a backup, that data could be lost forever – a real headache for any system administrator! Backups are essentially copies of your data that you can restore in case of such disasters. They provide peace of mind and business continuity.

Now, let's talk about `rsync`. It's a command-line tool in Linux that's like a super-efficient file copier. But it's much smarter than a simple `cp` command. Here are some of its key strengths:

- **Efficiency**: `rsync` only copies the differences between the source and destination files. If a file hasn't changed since the last backup, `rsync` skips it, saving time and bandwidth. This is especially useful for regular backups.
- **Incremental Backups**: As mentioned, it can perform incremental backups, meaning after the first full backup, it only copies the changes.
- **Versatility**: It works for both local backups (copying files on the same machine) and remote backups (copying files to another server). We'll focus on local backups for now.
- **Preservation of Attributes**: `rsync` can preserve file permissions, ownership, timestamps, and other important attributes, ensuring your backups are faithful copies of the original data.

Think of `rsync` as a meticulous librarian who, after the first full cataloging of your books (full backup), only notes down any new books or changes to existing ones (incremental backup) in subsequent visits.

## Installing `rsync` and Basic Syntax

Since these guides are based on Debian-based Linux distributions, you can easily install `rsync` using the `apt` package manager. Open your terminal and run the following command:

```bash
sudo apt update
sudo apt install rsync
```

The first command `sudo apt update` refreshes the package lists, ensuring you have the latest information about available software. The second command `sudo apt install rsync` downloads and installs the `rsync` tool. You'll likely be prompted to enter your password to authorize the installation.

Once installed, you can verify it by running:

```bash
rsync --version
```

This should display the version of `rsync` that's installed on your system.

### Basic Syntax

Now, let's look at the basic syntax of the `rsync` command:

```bash
rsync [options] source destination
```

- `rsync`: This is the command itself.
- `[options]`: These are flags that modify how `rsync` works. We'll learn about some important ones shortly.
- `source`: This is the file or directory you want to back up.
- `destination`: This is where you want to store the backup. It can be another directory on the same machine or a different storage device mounted on your server.

Think of it like saying, "Hey `rsync`, using these `[options]`, please copy this `source` to this `destination`."

### `rsync` Options

Now that you know the basic structure of the rsync command, let's explore some essential options that will significantly enhance its functionality.

Here are three fundamental options you'll use frequently:

- `-v` **(verbose)**: This option tells `rsync` to be more talkative and display detailed information about what files are being transferred and what actions are being taken. It's very helpful for understanding what `rsync` is doing, especially when you're starting out or troubleshooting. Think of it as the librarian giving you a detailed list of every book they handle.

- `-a` **(archive mode)**: This is a very powerful and commonly used option. It's actually a shorthand for several other options that ensure a comprehensive backup. Specifically, it includes:
  - `-r` **(recursive)**: Copies directories and their contents recursively.
  - `-l` **(links)**: Preserves symbolic links.
  - `-p` **(perms)**: Preserves permissions.
  - `-t` **(times)**: Preserves modification times.
  - `-g` **(group)**: Preserves group ownership.
  - `-o` **(owner)**: Preserves owner (requires superuser privileges).
  - `-D`: Preserves special and block devices. Essentially, `-a` aims to create an exact copy of your source data, preserving most of its attributes. It's like telling the librarian to make sure all the details about the books (like their genre, when they were last shelved, etc.) are also copied.
- `-n` **(dry run)**: This is an incredibly useful option for testing your rsync commands without actually making any changes to the destination. It shows you exactly what rsync would do if you ran the command without `-n`. This allows you to preview the actions and catch any potential mistakes before they happen. It's like asking the librarian to tell you which books they would move without actually moving them.

Here are a couple of examples to illustrate their usage:

To see a detailed output of what would happen when backing up a directory named `important_files` to a directory named `backup` in the current location, you would use:

```bash
rsync -avn important_files/ backup/
```

Notice the trailing slashes. They can be important and affect whether `rsync` creates a subdirectory within the destination.

To perform an actual backup while preserving attributes and seeing the progress, you'd use:

```bash
rsync -av important_files/ backup/
```

## Performing Full Backups with `rsync`

Imagine you have a crucial directory on your Ubuntu Server called `/etc/nginx/sites-available` that contains the configuration files for your websites. You want to create a backup of this entire directory to another local directory, say `/var/backup/nginx-config-full`.

Here's the `rsync` command you would use:

```bash
sudo rsync -av /etc/nginx/sites-available/ /var/backup/nginx-config-full/
```

Let's break down this command:

- `sudo`: We use `sudo` because backing up system configuration files often requires administrator privileges.
- `rsync`: The command itself.
- `-a`: This is our trusty archive mode, ensuring that all the files and directories within `/etc/nginx/sites-available` are copied recursively, and their permissions, ownership, timestamps, and symbolic links are preserved.
- `-v`: The verbose option, which will show you the details of each file and directory being copied.
- `/etc/nginx/sites-available/`: This is the **source** directory – the one we want to back up. Notice the trailing slash. It's important! If you omit it, `rsync` will copy the `sites-available` directory itself into the destination, creating `/var/backup/nginx-config-full/sites-available`. With the trailing slash, it copies the contents of `sites-available` directly into `/var/backup/nginx-config-full`.
- `/var/backup/nginx-config-full/`: This is the **destination** directory – where the backup will be stored. `rsync` will create this directory if it doesn't already exist. Again, the trailing slash here means `rsync` will copy the contents of the source into this directory.

When you run this command, `rsync` will go through all the files and directories within `/etc/nginx/sites-available` and copy them to `/var/backup/nginx-config-full`, preserving their attributes.

### Important Considerations when Choosing a Backup Destination for Your Local Backups

- **Separate Physical Storage**: Ideally, your backup destination should be on a completely separate physical storage device from your primary data. This protects against hardware failures of your main hard drive. Imagine if both your original files and their backups were on the same failing drive – that wouldn't be very helpful! This could be a second internal hard drive, an external USB drive, or even a Network Attached Storage (NAS) device on your local network (though that blurs the line a bit between local and remote).

- **Sufficient Capacity**: Your backup storage needs to have enough capacity to hold all the data you plan to back up, and ideally, some extra space for future growth and multiple backup versions (which we'll discuss later with incremental backups). Running out of space mid-backup is a common and frustrating issue.

- **Accessibility**: The backup location needs to be accessible to the user or process performing the backup. This usually means having the correct file system permissions.

- **Mount Point**: If you're using a separate drive or partition, it needs to be properly mounted to a directory on your system (like `/mnt/backupdrive` as we mentioned).

- **File System**: The file system on your backup destination should be compatible with the types of files and attributes you are backing up. Most Linux file systems (like ext4) work well with `rsync`.

Think of it this way: if your main server is a house, you wouldn't want to keep the emergency blueprints inside the same house! You'd want them in a separate, fireproof safe (the backup storage) that's easy to get to if something goes wrong.

## Implementing Incremental Backups with `rsync`

Now that we've covered full backups, let's talk about **incremental backups**. This is where `rsync` really shines in terms of efficiency for ongoing backups.

Imagine you've done a full backup of your `/etc/nginx/sites-available directory`, which we'll call your base backup. Now, a week later, only a few configuration files have been changed. Instead of copying the entire `sites-available` directory again (which would be time-consuming and use a lot of space), an incremental backup only copies the files that have changed since the last backup.

`rsync` achieves this by comparing the files in the source directory with the files in the destination directory from the previous backup. It then only transfers the new or modified files. This saves significant time and storage space, especially for frequently changing data.

To implement incremental backups with `rsync`, we often use a combination of the `--delete` option and hard links (achieved using `-H` and `--link-dest`). Let's break these down:

- `--delete`: This option tells `rsync` to delete files in the destination directory that no longer exist in the source directory. This is important for keeping your backup synchronized with the original data. If you delete a file in `/etc/nginx/sites-available`, using `--delete` in your backup command will also remove it from your backup on the next run. Think of it as the librarian removing records of books that have been discarded.

- **Hard Links (**`-H` **and** `--link-dest`**)**: This is the clever part that saves space.
  - `-H` tells `rsync` to preserve hard links in the source.
  - `--link-dest=DIR` tells `rsync` that if a file in the source has not changed since the backup in `DIR`, instead of copying the entire file again, it should create a **hard link** to the unchanged file in `DIR`.

A **hard** link is essentially another name for the same file data on the disk. It doesn't take up extra space. So, in our incremental backup, if a configuration file hasn't changed since the last backup, `rsync` just creates a new "pointer" (the hard link) to the existing version of the file in your previous backup directory. Only the truly new or modified files are actually copied, saving a ton of space!

### A Practical Example of Incremental Backups

Let's say we want to back up our `/etc/nginx/sites-available` directory to `/var/backup/nginx-config-incremental`. We'll create a new subdirectory within this for each backup.

1. **Initial Full Backup**

    First, we perform a full backup. We'll name the first backup directory `backup.0`.

    ```bash
    sudo rsync -av /etc/nginx/sites-available/ /var/backup/nginx-config-incremental/backup.0/
    ```

    This command will copy all the files and directories from `/etc/nginx/sites-available` to `/var/backup/nginx-config-incremental/backup.0/`.

2. **First Incremental Backup**

    Now, let's say some changes have been made to the configuration files. To create our first incremental backup (let's call it `backup.1`), we'll use the `--link-dest` option, pointing it to our base backup (`backup.0`), and also include the `--delete` option.

    ```bash
    sudo rsync -av --delete --link-dest=/var/backup/nginx-config-incremental/backup.0/ /etc/nginx/sites-available/ /var/backup/nginx-config-incremental/backup.1/
    ```

    Here's what's happening:

   - `--link-dest=/var/backup/nginx-config-incremental/backup.0/`: This tells rsync to look at the backup.0 directory. If a file in `/etc/nginx/sites-available/` has the same content and attributes as a file in `backup.0`, it will create a hard link to that file in `backup.1` instead of copying the entire file again.
   - `--delete`: This ensures that if any files were deleted from `/etc/nginx/sites-available/` since the last backup, they will also be deleted from `backup.1`.
   - The other options (`-a` and `-v`) work as we discussed before.

    Only the files that have been added or modified since the `backup.0` will be fully copied into the `backup.1` directory. The rest will be hard links to the files in `backup.0`, saving space.

3. **Subsequent Incremental Backups**

    For the next incremental backup (e.g., `backup.2`), you would repeat the process, but this time, you would point `--link-dest` to the previous successful backup (`backup.1` in this case).

    ```bash
    sudo rsync -av --delete --link-dest=/var/backup/nginx-config-incremental/backup.1/ /etc/nginx/sites-available/ /var/backup/nginx-config-incremental/backup.2/
    ```

    And so on. Each new backup directory will only contain the changes since the immediately preceding backup, with the unchanged files being hard links to the versions in the previous backup.

    This way, you can maintain a history of your backups without taking up excessive disk space.

## Automating Backups with `cron`

Now that we've covered how to perform full and incremental backups with `rsync`, the next logical step is to **automate** this process. You don't want to have to manually run these `rsync` commands every day (or however often you need to back up)!

This is where `cron` comes in. `cron` is a time-based job scheduler in Unix-like operating systems. It allows you to schedule commands or scripts to run automatically at specific times, dates, or intervals. Think of it as your server's personal assistant that takes care of routine tasks for you, like running your backups.

To use `cron`, you need to edit a special file called a **crontab** (cron table). Each line in the crontab represents a scheduled job. The syntax for a crontab entry looks like this:

```shell
minute hour day_of_month month day_of_week command_to_run
```

Let's break down each field:

- `minute`: The minute of the hour when the command will run (0-59).
- `hour`: The hour of the day when the command will run (0-23, where 0 is midnight).
- `day_of_month`: The day of the month when the command will run (1-31).
- `month`: The month of the year when the command will run (1-12, or you can use abbreviations like jan, feb, etc.).
- `day_of_week`: The day of the week when the command will run (0-6, where 0 is Sunday, or you can use abbreviations like `sun`, `mon`, etc.).
- `command_to_run`: The actual command or script you want cron to execute.

For example, if you wanted to run your incremental backup script every day at 2:00 AM, your crontab entry might look something like this:

```shell
0 2 * * * /path/to/your/backup_script.sh
```

Here, `0` in the `minute` field means the job will run at the beginning of the hour. `2` in the `hour` field means 2 AM. The asterisks `*` in the `day_of_month`, `month`, and `day_of_week` fields mean that the job will run every day, every month, and every day of the week. `/path/to/your/backup_script.sh` is the full path to a script that contains your `rsync` command.

### Example Automated Backup Script Scheduled with `cron`

Here is an example of a script that will

You can use a text editor like `nano` to create this script. Let's say you want to create a script called `backup_nginx.sh` in your home directory (`~`).

1. **Open a new file in** `nano`:

    ```bash
    nano ~/backup_nginx.sh
    ```

2. **Enter your** `rsync` **command into the file**. For example, to perform an incremental backup as we discussed, you might put something like this in your script:

    ```bash
    #!/bin/bash
    BACKUP_DIR="/var/backup/nginx-config-incremental"
    SOURCE_DIR="/etc/nginx/sites-available"
    DATE=$(date +%Y-%m-%d_%H-%M-%S)
    PREVIOUS_BACKUP=$(ls -d "$BACKUP_DIR"/backup.* | sort -rV | head -n 1)

    if [ -z "$PREVIOUS_BACKUP" ]; then
      # First backup: perform a full backup
      rsync -av "$SOURCE_DIR"/ "$BACKUP_DIR"/backup."$DATE"/
    else
      # Subsequent backups: perform an incremental backup
      rsync -av --delete --link-dest="$PREVIOUS_BACKUP" "$SOURCE_DIR"/ "$BACKUP_DIR"/backup."$DATE"/
    fi
    ```

    Let's break down this script:

    - `#!/bin/bash`: This is a shebang line, which tells the system to execute the script using Bash.
    - `BACKUP_DIR="/var/backup/nginx-config-incremental"`: This sets a variable for your main backup directory.
    - `SOURCE_DIR="/etc/nginx/sites-available"`: This sets a variable for the directory you're backing up.
    - `DATE=$(date +%Y-%m-%d_%H-%M-%S)`: This creates a variable containing the current date and time, which we'll use to name our backup directories.
    - `PREVIOUS_BACKUP=$(ls -d "$BACKUP_DIR"/backup.* | sort -rV | head -n 1)`: This line finds the most recent previous backup directory.
    - The `if` statement checks if it's the first backup (if `$PREVIOUS_BACKUP` is empty). If it is, it performs a full backup.
    - Otherwise, it performs an incremental backup using `--link-dest` pointing to the previous backup.

3. **Make the script executable**: After you've saved the script in `nano` (Ctrl+O, then Enter, then Ctrl+X), you need to give it execute permissions:

    ```bash
    chmod +x ~/backup_nginx.sh
    ```

Now you have a script that will perform either a full backup (if it's the first run) or an incremental backup (for subsequent runs).

**Next, you need to tell** `cron` **to run this script at your desired schedule**. To edit your crontab, run:

```bash
crontab -e
```

This will open your crontab file in a text editor (usually `nano` by default). Add a line like this (adjusting the time as needed):

```shell
0 2 * * * ~/backup_nginx.sh
```

This will run your `backup_nginx.sh` script every day at 2:00 AM.

After adding the line, save and close the crontab file. `cron` will automatically pick up the changes.

## Backup Strategies and Best Practices

### Backup Strategies

Let's talk about different backup strategies. Understanding these will help you make informed decisions about how you want to protect your data.

We've already discussed **full backups** (copying everything every time) and **incremental** backups (copying only changes since the last backup). These are two fundamental strategies, but there's another important one called **differential backups**.

Here's a quick comparison:

- **Full Backup**: As we know, this copies all selected data. It's straightforward to restore from a full backup, as everything you need is in one place. However, it takes the longest time and the most storage space for each backup.

- **Incremental Backup**: This copies only the data that has changed since the last backup (which could be a full or a previous incremental backup). They are fast and use minimal storage space for each run. However, restoring from an incremental backup requires having the last full backup and all subsequent incremental backups in the correct order. If any one of the incremental backups is corrupted or missing, the restore process can fail.

- **Differential Backup**: This is like a middle ground. A differential backup copies all the data that has changed since the last full backup. So, on Monday, you might do a full backup. On Tuesday, the differential backup will contain all changes since Monday. On Wednesday, the differential backup will again contain all changes since Monday (it doesn't care about Tuesday's backup).

Think of it like this:

- **Full**: Making a complete photocopy of a book every day.
- **Incremental**: Each day, only photocopying the pages that were changed since the last photocopy. To get the complete book, you need the original full copy and all the changed pages in order.
- **Differential**: After the first full photocopy, each day you photocopy all the pages that have changed since that original full copy. To get the complete book, you only need the original full copy and the latest set of changed pages.

**When to use which?**

- **Full backups** are good for initial backups or for less frequently changing data where simplicity of restore is paramount.
- **Incremental backups** are excellent for daily backups of large datasets with small daily changes, as they are fast and space-efficient. However, the more incrementals you have, the longer and more complex the restore process becomes.
- **Differential backups** offer a balance. They take more space and time than incrementals (but less than fulls after the first one), and the restore process is simpler than with incrementals (you only need the last full and the latest differential).

You can even combine these strategies. For example, you might do a full backup weekly, with daily incremental backups in between, or perhaps a full backup monthly with daily differentials.

The best strategy depends on factors like the volume of your data, how frequently it changes, your storage capacity, and your desired restore time.

### Best Practices

Let's discuss some best practices for setting up and maintaining your local backups. These are like the rules of the road that ensure your backups are reliable and will actually save you when disaster strikes!

Here are some key best practices to keep in mind:

- **Verification**: Don't just assume your backups are working correctly! Regularly verify your backups to ensure the files are being copied as expected and are not corrupted. You can do this by occasionally listing the contents of your backup directories and comparing them to your source data. For more critical systems, you might even consider using `rsync`'s `--checksum` option, which performs a more thorough verification by comparing the content of files, though it can be slower. Think of it as occasionally checking if the emergency blueprints are actually readable and complete.

- **Testing Restores**: This is arguably the most crucial step. A backup is only as good as your ability to restore from it. Periodically practice restoring files or even a small directory from your backups to a temporary location to ensure the process works and you know what to do in a real recovery situation. This will also help you identify any potential issues with your backup process. It's like having a fire drill to make sure everyone knows how to evacuate safely.

- **Security Considerations**: Even though these are local backups, security is still important. Ensure that your backup destination has appropriate file permissions so that only authorized users (like the system administrator) can access and modify the backup data. If your backup destination is an external drive, consider storing it in a secure location to protect against physical theft or damage.

- **Retention Policy**: Decide how long you need to keep your backups. Do you need daily backups for a week, weekly backups for a month, and monthly backups for a year? Implementing a retention policy helps you manage storage space and ensures you have backups available for different recovery scenarios. You might need to periodically prune older backups according to your policy.

- **Monitoring**: If possible, set up some form of monitoring for your backup process. This could be as simple as checking the `cron` logs for any errors or using more sophisticated tools to track backup success and storage usage. Knowing if a backup failed is crucial for addressing the issue promptly.

- **Documentation**: Keep a record of your backup strategy, the scripts you use, where the backups are stored, and the restore procedure. This documentation will be invaluable if you need to recover data, especially if someone else needs to do it in your absence.

By following these best practices, you can significantly increase the reliability and usefulness of your local backup strategy with `rsync`.
