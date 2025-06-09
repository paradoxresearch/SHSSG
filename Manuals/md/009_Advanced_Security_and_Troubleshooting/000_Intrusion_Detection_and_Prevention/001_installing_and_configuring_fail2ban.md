# Installing and Configuring Fail2ban

## Contents

- [Installing and Configuring Fail2ban](#installing-and-configuring-fail2ban)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Installing Fail2ban](#installing-fail2ban)
  - [Basic Configuration - Jails](#basic-configuration---jails)
    - [Fail2ban Configuration](#fail2ban-configuration)
  - [Understanding Filters](#understanding-filters)
  - [Taking Action - Banning and Unbanning](#taking-action---banning-and-unbanning)
    - [Banning](#banning)
    - [Unbanning](#unbanning)
  - [Applying Configurations and Restarting](#applying-configurations-and-restarting)
    - [Testing Your Configuration](#testing-your-configuration)

## Introduction

Think of Fail2ban as a diligent security guard for your server. It constantly watches the logs of various services, like a security camera feed. If it sees too many suspicious activities, like repeated failed login attempts from the same source, it acts by temporarily blocking the troublemaking IP address, just like a security guard might temporarily deny access to someone causing repeated issues.

## Installing Fail2ban

Our first step is to install this helpful security guard on your server. We'll use the `apt` package manager for this.

To install Fail2ban, you'll use the following command in your terminal:

```bash
sudo apt update && sudo apt install fail2ban
```

Let's break this command down:

- `sudo`: This command gives you temporary superuser (administrator) privileges, which are needed to install software. You'll be asked for your password.
- `apt update`: This command refreshes the list of available packages from the software repositories. It's always a good idea to run this before installing new software to make sure you have the latest information.
- `&&`: This means that the second command (`sudo apt install fail2ban`) will only run if the first command (`sudo apt update`) is successful.
- `sudo apt install fail2ban`: This is the command that actually downloads and installs the Fail2ban software and its dependencies.

## Basic Configuration - Jails

Let's talk about how you tell Fail2ban what to protect. This is where the concept of jails comes in.

Think of a jail as a specific set of rules that Fail2ban uses to monitor a particular service on your server. Each jail is like a dedicated security detail assigned to watch over a specific area, such as your SSH login attempts, your web server access logs, or your mail server.

For example, you might have one jail specifically configured to watch the logs of your SSH server. This jail will have its own rules defining what constitutes a suspicious activity (like too many failed password attempts) and what action to take when such activity is detected (like blocking the offending IP address).

You can have multiple jails running simultaneously, each monitoring a different service with its own tailored rules. This allows you to have granular control over the security of each part of your server.

### Fail2ban Configuration

The main configuration file for Fail2ban is located at `/etc/fail2ban/jail.conf`.

Think of `jail.conf` as the master blueprint for all the potential security measures Fail2ban can take. It contains the default configurations for various services. However, **it's strongly recommended that you do not directly edit this file**. Why? Because if Fail2ban gets updated, your changes in `jail.conf` might be overwritten, and you'd lose your custom settings.

Instead, the best practice is to create a local override file named `jail.local` in the same directory (`/etc/fail2ban/`). Any settings you put in `jail.local` will override the corresponding settings in `jail.conf`. This way, your configurations are preserved during updates.

You can create `jail.local` by copying the `jail.conf` file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Then, you can edit `jail.local` using a command-line text editor like `nano`:

```bash
sudo nano /etc/fail2ban/jail.local
```

Inside `jail.local`, you'll find sections for different services (jails), like `[sshd]`, `[apache-auth]`, `[nginx-http-auth]`, and many others. You can enable and configure these jails as needed.

Inside this file, you'll likely see a section that looks something like this (it might be commented out with `#` at the beginning of each line):

```Ini, TOML
[sshd]
enabled = false
port = ssh
logpath = /var/log/auth.log
bantime = 600
findtime = 600
maxretry = 3
```

Let's break down these key parameters:

- `[sshd]`: This is the name of the jail. It specifically applies to SSH connections.
- `enabled = false`: By default, most jails are disabled. To activate the SSH jail, you need to change this to `enabled = true`. Think of it as flipping the switch to turn on the security monitoring for SSH.
- `port = ssh`: This specifies the port that Fail2ban should monitor for the SSH service. `ssh` is a symbolic name that usually resolves to port 22, the standard SSH port. You can also specify the port number directly if your SSH server is running on a different port.
- `logpath = /var/log/auth.log`: This tells Fail2ban the location of the SSH server's log file. Fail2ban will scan this file for failed login attempts and other suspicious activities.
- `bantime = 600`: This sets the duration, in seconds, for which an offending IP address will be banned. In this case, 600 seconds equals 10 minutes. After this time, the ban is automatically lifted.
- `findtime = 600`: This defines the time window, in seconds, during which failed login attempts are counted. Here, Fail2ban will look at the last 600 seconds (10 minutes) of the log file.
- `maxretry = 3`: This is the maximum number of failed login attempts allowed from a single IP address within the `findtime` period before that IP is banned. In this example, if an IP tries to log in unsuccessfully 3 times within 10 minutes, it will be banned for 10 minutes.

To enable the SSH jail with these default settings, you would change `enabled = false` to `enabled = true` in your `jail.local` file.

Imagine `findtime` as the period the security guard reviews their camera footage, and `maxretry` as the number of suspicious actions they'll tolerate within that period before taking action (`bantime`).

## Understanding Filters

Think of filters as the specific instructions given to our security guard on how to identify a potential intruder in the security camera footage. These instructions are defined using regular expressions, which are like powerful search patterns that can match specific text within the log files.

For each jail you enable (like the `[sshd]` jail we just looked at), there's a corresponding filter that tells Fail2ban what patterns to look for in the log file (`/var/log/auth.log` in the case of SSH) that indicate a failed login attempt.

The configuration files for these filters are located in the `/etc/fail2ban/filter.d/` directory. Inside this directory, you'll find various `.conf` files, each corresponding to a different service. For example, the filter for the `[sshd]` jail is usually defined in a file named `sshd.conf`.

When the `[sshd]` jail is enabled, Fail2ban uses the rules defined in the `sshd.conf` filter to scan the `/var/log/auth.log file`. If it finds a line that matches one of the patterns defined in the filter (indicating a failed login), it increments a counter for the IP address that made the attempt. If this counter reaches the `maxretry` threshold within the `findtime` period, the `bantime` action (blocking the IP) is triggered.

So, to summarize, **jails** define *what* to protect and *how* to react, while **filters** define *how* to recognize the malicious activity within the logs of those services.

---

The filters that Fail2ban uses to identify suspicious activity are located in the `/etc/fail2ban/filter.d/` directory. If you were to list the contents of this directory using the `ls` command, you'd see a variety of `.conf` files, each named after a service that Fail2ban can monitor (e.g., `apache-auth.conf`, `nginx-http-auth.conf`, `sshd.conf`, etc.).

For the `[sshd]` jail we've been focusing on, the corresponding filter configuration is usually in the `sshd.conf` file. This file contains the regular expressions that Fail2ban uses to scan the `/var/log/auth.log` file for patterns indicating failed SSH login attempts.

## Taking Action - Banning and Unbanning

### Banning

When Fail2ban, guided by its jails and filters, detects that an IP address has exceeded the `maxretry` threshold within the `findtime` period, it takes action by **banning** that IP address. This means that Fail2ban adds a rule to your server's firewall (usually `iptables` or `firewalld`) to block all network traffic coming from that specific IP address for the duration specified by the `bantime`.

Think of it like our security guard spotting someone causing trouble repeatedly. They don't just watch; they actively prevent that person from entering the premises for a certain amount of time.

So, how do you know which IP addresses Fail2ban has currently banned? You can use the `fail2ban-client` command-line tool. This is your primary way to interact with the Fail2ban service once it's running.

To see the status of Fail2ban and the currently banned IPs for each jail, you can use the following command:

```bash
sudo fail2ban-client status
```

This command will give you an overview of all the active jails and, for each jail, it will show you the number of currently banned IPs and the list of those IPs.

For example, if the `[sshd]` jail has banned one IP address, the output might look something like this:

```shell
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 5
`- Actions
   |- Currently banned: 1
   `- IP list: 203.0.113.45
```

Here, you can see that under the "Actions" section for the "sshd" jail, there is one "Currently banned" IP, and the "IP list" shows the banned IP address (`203.0.113.45` is just an example).

This command is very useful for monitoring Fail2ban's activity and seeing who is being blocked.

### Unbanning

Fail2ban might block an IP address that you trust, or perhaps you've resolved the issue that caused the ban. In such cases, you'll need to know how to manually unban an IP address.

You can do this using the `fail2ban-client` tool as well. The command to unban an IP address is:

```bash
sudo fail2ban-client set <jailname> unbanip <ipaddress>
```

Let's break this down:

- `sudo fail2ban-client`: This is the main command to interact with the Fail2ban service.
- `set`: This tells `fail2ban-client` that you want to change something.
- `<jailname>`: Here, you need to specify the name of the jail from which you want to unban the IP address. For example, if you want to unban an IP from the SSH jail, you would use `sshd`.
- `unbanip`: This is the specific action you want to perform – to unban an IP address.
- `<ipaddress>`: This is the IP address that you want to unban.

So, if you wanted to unban the IP address `203.0.113.45` from the sshd jail, you would run the command:

```bash
sudo fail2ban-client set sshd unbanip 203.0.113.45
```

After running this command, Fail2ban will remove the blocking rule for that IP address from your server's firewall, and the IP will be able to connect again to the service monitored by that jail.

## Applying Configurations and Restarting

Simply editing `jail.local` doesn't automatically make Fail2ban use those new settings. You need to tell the Fail2ban service to reload its configuration. The standard way to manage system services on Ubuntu (and many other Linux distributions) is using `systemctl`.

To restart the Fail2ban service, which will cause it to reload its configuration files (including your `jail.local`), you'll use the following command:

```bash
sudo systemctl restart fail2ban
```

Think of this as telling our security guard to read the updated instructions you've given them in `jail.local`. After you run this command, Fail2ban will start using the enabled jails, the specified log paths, the filters, and the banning rules you've set.

It's also a good idea to check the status of the Fail2ban service after restarting to make sure there were no errors during the reload. You can do this with the command:

```bash
sudo systemctl status fail2ban
```

This command will show you whether the Fail2ban service is active (running) and if there were any issues during startup or the last reload.

Finally, and this is a very important point: always test your Fail2ban configuration. You can do this by intentionally triggering a ban (e.g., by trying to SSH into your server with the wrong password multiple times from an IP address you control). Then, use `fail2ban-client status` to verify that the IP address has been banned in the `[sshd]` jail. After confirming the ban, remember to unban your test IP address using the command we discussed earlier.

### Testing Your Configuration

The best way to test your Fail2ban setup is to simulate a failed login attempt. Here's how you can do it for the SSH jail we've been configuring:

1. **Identify an IP address you can use for testing**. This could be your home computer or another machine you control.
2. **Attempt to SSH into your server from that test IP address using an incorrect password multiple times**. Make sure you exceed the `maxretry` value you set in your `jail.local` file (which was 3 in our example).
3. **Wait for the** `findtime` **period to elapse**. In our example, this was 600 seconds (10 minutes).
4. **On your server, use the** `fail2ban-client status sshd` **command**. You should see the IP address you used for testing listed under the "Currently banned" section.

If you see your test IP address listed as banned, congratulations! Your basic Fail2ban configuration for SSH is working.

**Important**: After you've confirmed that Fail2ban is working correctly, remember to **unban your test IP** address using the `sudo fail2ban-client set sshd unbanip <your_test_ip>` command so you don't accidentally lock yourself out of your server.
