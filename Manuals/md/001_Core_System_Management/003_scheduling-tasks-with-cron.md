# Scheduling Tasks With Cron

## Contents

- [Scheduling Tasks With Cron](#scheduling-tasks-with-cron)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Understanding Cron Syntax](#understanding-cron-syntax)
    - [Special Characters in Cron Syntax](#special-characters-in-cron-syntax)
  - [Working with Crontab](#working-with-crontab)
    - [Edit Your Crontab](#edit-your-crontab)
    - [List Existing Cron Jobs](#list-existing-cron-jobs)
    - [Removing Cron Jobs](#removing-cron-jobs)
    - [Redirect the Output of a Script to a Log File](#redirect-the-output-of-a-script-to-a-log-file)
  - [Best Practices and Troubleshooting](#best-practices-and-troubleshooting)

## Introduction

Maintaining a server can consist of a series of repetitive tasks that you know need to be routinely repeated. With task scheduling your system can automatically run essential services. These services can include:

- **Backing up data**: Automatically copying you important files to a safe location regularly.
- **Rotate log files**: Manage your server logs by archiving old ones and starting new ones to prevent disk space issues.
- **Sending out reports**: Automatically generating and emailing summaries of server activity or website statistics.
- **Running maintenance scripts**: Performing routine system checks and optimizations.

The most widely used tool for scheduling tasks in Debian based GNU/Linux systems is called **cron**.  **Cron** directs commands and scripts to run a scheduled times.

The way **cron** works is by reading instructions from special files called **crontabels**.  A crontable (often just referred to as a "crontab") contains a list of commands that cron will execute, along with a a schedule for when each command should run. Each user on the system can have their own crontab, allowing for personalized scheduling of tasks. There's also a system-wide crontab for tasks that need to be run with administrative privileges.

## Understanding Cron Syntax

Each line in a crontab file represents a single scheduled task, and it follows a specific format. Here is the basic structure of a task:

```shell
minute hour day_of_month month day_of_week command_to_execute
```

Here is a breakdown of each field:

- **minute**: This specifies the minute of the hour when the command should run. It can be a value range from **0 to 59**.
- **hour**: This specifies the hour of the day (using a 24-hour clock) when the command should run. Its value range is from **0 to 23**.
- **day_of_month**: This specifies the day of the month when the command should run. It can be a value in the range of **1 to 31**.
- **month**: This specifies the month of the year when the command should run. Its value range starts at **1 (January) and goes to 12 (December)**, or you can use abbreviations like `jan`, `feb`, `mar`, etc.
- **day_of_week**: This specifies the day of the week when the command should run. It can be a value from **0 (Sunday) to 6 (Saturday)**, or you can use abbreviations like `sun`, `mon`, `tue`, etc.
- **command_to_execute**: This is the actual command or the path to the script that you want cron to run.

The first five fields in a task act as a filter, and when all of their criteria is met, `command_to_execute` will run.

For example, if you wanted to run a command every day at 3:30 PM, the cron entry might look something like this:

```shell
30 15 * * * /path/to/your/command
```

In the above example:

- `30` is the minute (30 minutes past the hour).
- `15` is the hour (3 PM in 24-hour format).
- `*` in the day of the month field means "every day of the month".
- `*` in the month field means "every month of the year".
- `*` in the day of the week field means "every day of the week".
- `/path/to/your/command` is the actual command you want to run.

### Special Characters in Cron Syntax

There are some special characters that give you a lot more flexibility in defining when your tasks should run.

These are the most common special characters:

- `*` **(Asterisk)**: This is like a wildcard. It means "all possible values" for that field. For example, `*` in the "minute" field means "every minute".
- `,` **(Comma)**: This allows you to specify a list of discrete values. For example, if you wanted a script to run at 8 AM, 12 PM, and 5 PM, you could use `0 8,12,17 * * * /path/to/script/`.
- `-` **(Hyphen)**: This allows you to specify a range of values. For example, if you wanted a script to run every hour from 9 AM to 5 PM, you could use `0 9-17 * * * /path/to/script`.
- `/` **(Slash)**: This allows you to specify step values within a range. For example, `*/5` in the "minute" field means "run every 5 minutes" (starting from minute 0). Similarly, `0 */3 * * *` would mean "run at the beginning of every 3rd hour".

Here are some more examples using special syntax:

- Run a script at 00:00 (midnight) every day:
  
    ```shell
    0 0 * * * /path/to/daily_script
    ```

- Run a script at 15 minutes past every hour:
  
    ```shell
    15 * * * * /path/to/hourly_script
    ```

- Run a script every Monday at 10:30 AM:

    ```shell
    30 10 * * 1 /path/to/weekly_report
    ```

- Run a script on the 1st and 15th of every month at 6:00 AM:

    ```shell
    0 6 1,15 * * /path/to/monthly_task
    ```

- Run a script every 2 hours:

    ```shell
    0 */2 * * * /path/to/every_two_hours
    ```

## Working with Crontab

The primary way to interact with cron is through the crontab command. Think of `crontab` as the tool that allows you to view, edit, and manage your list of scheduled tasks.

### Edit Your Crontab

To access and edit your personal crontab, you'll use the following command in your terminal:

```bash
crontab -e
```

The `-e` option stands for "edit". When you run this command for the first time, it will likely ask you to choose a text editor. Popular choices include `nano`, `vim`, and `gedit`. If you're new to command-line editors, `nano` is generally considered the easiest to learn. Just make a selection and press Enter.

Once you've chosen an editor, a file will open. This is your crontab file. It might be empty if you haven't scheduled any tasks before, or it might contain a list of your existing cron jobs. You can then add new lines following the cron syntax that was just covered, or modify existing ones.

**Important Note**: It's crucial to use the `crontab -e` command to edit your cron jobs. **Do not directly edit the system-wide crontab files** (which are usually located in `/etc/crontab` or `/etc/cron.d`). Using `crontab -e` ensures that the system correctly updates and registers your changes with the cron daemon (the background process that runs scheduled tasks).

After you've made your changes in the editor, you'll need to save the file and exit. If you're using `nano`, you can do this by pressing `Ctrl-O` (write out) followed by `Enter`, and then `Ctrl-X` (exit). Other editors will have their own save and exit commands.

Once you save and exit, cron will automatically notice changes to your crontab file and will start following the new schedule.

### List Existing Cron Jobs

To see a list of all the cron jobs you currently have scheduled, you can use the following command in your terminal:

```bash
crontab -l
```

The `-l` option stands for "list". When you run this command, it will display the contents of your crontab file. If you haven't scheduled any jobs yet, it might say something like "no crontab for your_username". This is a good way to quickly check what tasks are set to run automatically.

### Removing Cron Jobs

There might come a time when you want to remove some or all of your scheduled tasks. To remove your *entire* crontab (all scheduled jobs), you can use the following command:

```bash
crontab -r
```

The `-r` option stands for "remove". **Be very careful when using this command!** It will delete all of your scheduled tasks, and you won't get a confirmation prompt. Once you run this, your crontab will be empty.

**Think of `crontab -r` as hitting the "reset" button for all of your scheduled tasks.** Make sure you want to do this before you run the command!

If you only want to remove specific cron jobs, you'll need to open your crontab using `crontab -e` and manually delete the lines corresponding to the tasks you want to remove. Then, save and exit the editor.

### Redirect the Output of a Script to a Log File

By default, any output produced by a script run by cron (both standard output and standard error) is usually sent via email to the user who owns the crontab. While this can be helpful for important notifications, if your script runs frequently or produces a lot of output, your inbox can quickly become flooded!

A much better practice is to **redirect the output of your script to a log file**. This allows you to keep a record of when your script ran, what it did, and if any errors occurred. This is invaluable for debugging and monitoring the health of your scheduled tasks.

Here's how you can redirect the output of your script when scheduling with crontab:

- **Redirecting Standard Output (`>`)**: This will send only the normal output of your script to the specified file. If the file exists, its contents will be overwritten.

    ```shell
     * * * * * /home/your_username/my_script.sh > ~/my_script.log
     ```

- **Appending Standard Output (`>>`)**: This will also send the normal output to the file, but it will append it to the end of the file if it already exists, preserving the history. This is generally preferred for log files.

    ```shell
    * * * * * /home/your_username/my_script.sh >> ~/my_script.log
    ```

- **Appending Standard Error (`2>>`)**: This will append error messages to the specified file.

    ```shell
    * * * * * /home/your_username/my_script.sh 2>> ~/my_script_error.log
    ```

- **Redirecting Both Standard Output and Standard Error (`&>` or `2>&1`)**: This will send both normal output and error messages to the same file. The `&>` syntax is often simpler, but `2>&1` is more universally understood.

    ```shell
    * * * * * /home/your_username/my_script.sh &> ~/my_script.log
    ```

    ```shell
    * * * * * /home/your_username/my_script.sh >> ~/my_script.log 2>&1
    ```

A good practice is to append both the standard output (which is the date and time) and any potential errors to a log file:

```shell
* * * * * /home/your_username/my_script.sh >> ~/my_script.log 2>&1 
```

Now, instead of getting emails, you can check the `~/my_script.log` file to see when your script ran and if there were any issues. This is a much cleaner and manageable way to keep track of your scheduled tasks.

## Best Practices and Troubleshooting

Here are some best practices to keep in mind with working with cron:

- **Use Full Paths**: When specifying the command or script to run in your crontab, always use the **full path** to the executable. This is because cron runs in a limited environment and might not have the same `PATH` settings as your interactive shell. To find the full path to a command, you can use the `which` command in your terminal. For example, `which date` will tell you the full path to the `date` command (usually `/bin/date`). So, instead of just `date` in a script run by cron, it's safer to use `/bin/date`. Similarly, always use the full path to your scripts (e.g., `/home/your_username/my_script.sh`).
- **Be Mindful of the Environment**: Cron jobs run in a minimal environment, meaning they might not have access to the same environment variables (settings that programs use) that your interactive shell does. If your script relies on specific environment variables, you might need to set them within the script itself or the crontab entry. You can set environment variables in your crontab like this:
  
```shell
SHELL=/bin/bash
HOME=/home/your_username
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin/:/bin

* * * * * /home/your_username/my_script.sh >> my_script.log 2>&1
```

It's often a good idea to explicitly set `SHELL`, `HOME`, and `PATH` at the top of your crontab. You can output your current environment variables with commands in your terminal like: `echo $SHELL`, `echo $HOME`, `echo $PATH`

- **Ensure Scripts are Robust**: Your scripts should be written to handle potential errors gracefully. This includes:
  - **Error Handling**: Use conditional statements and error checking within your script to catch and manage potential issues.
  - **Exit Codes**: Make sure your script exits with an appropriate exit code (0 for success, non-zero for failure). This can be useful if other programs or scripts depend on the outcome of your cron job.
  - **Idempotency (if applicable)**: For some tasks, it's important that running the script multiple times doesn't cause unintended side effects. Such scripts are called idempotent. For example, a backup script should ideally not create multiple identical backups if run more than once for the same time period.
- **Keep Logs**: We already discussed redirecting output to log files. Regularly review these logs to ensure your cron jobs are running correctly and to identify any errors.
- **Start with Less Frequent Schedules**: When you first set up a new cron job, especially if it's running a complex script it's a good idea to schedule it to run less frequently (e.g., once a day) until your confident it's working correctly. You can then increase the frequency as needed.
- **Comment Your Crontab**: If you have multiple cron jobs, it's helpful to add comments to your crontab file to explain what each job does. You can add comments by starting a line with a `#` symbol. These lines will be ignored by cron.

Sometimes, even when you think you've set everything up correctly, your cron jobs might not run as expected. Here are some common things to check when troubleshooting:

- **Check Your Logs**: This is usually the first and most important step. If you've correctly redirected the output of your script to a log file, examine that file for any error messages or indications of what might be going wrong. Even if you didn't explicitly redirect output, the system might still log some information. Check the system logs, which can often be found in `/var/log/syslog` or a similar location (the exact location may very slightly depending on your Ubuntu version). Look for any messages related to cron or your script.
- **Verify Permissions**: Make sure your script has execute permissions (`chmod +x`). Also, ensure that the user under whose crontab the job is running has the necessary permissions to execute the script adn access any files or directories it needs. For example, if your script tries to write to a directory where the user doesn't have write access, it will fail.
- **Check the Cron Syntax**: Double-check the syntax of your cron entry in your crontab file. Even a small typo can prevent the job from running. Use the format `minute hour day_of_month month day_of_week command_to_execute` and ensure the values are within the valid ranges. You can even use online cron expression validators to check if your syntax is correct.
- **Cron Daemon Status**: Ensure that the cron daemon is actually running on your server. You can check its status using the following command:

```bash
sudo systemctl status cron
```

This will tell you if the service is active (running), inactive (dead), or has encountered any errors. If it's not running, you can start it with `sudo systemctl start cron` and enable it to start automatically on boot with `sudo systemctl enable cron`.
