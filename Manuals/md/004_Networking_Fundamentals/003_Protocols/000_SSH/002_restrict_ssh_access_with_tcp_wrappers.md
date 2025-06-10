# Restrict SSH Access with TCP Wrappers

## Contents

- [Restrict SSH Access with TCP Wrappers](#restrict-ssh-access-with-tcp-wrappers)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Setup your TCP Wrappers](#setup-your-tcp-wrappers)
    - [Edit your hosts deny File](#edit-your-hosts-deny-file)
    - [Edit your hosts allow File](#edit-your-hosts-allow-file)

## Introduction

Restricting SSH with TCP Wrappers is another common method alongside `ufw` and `iptables`. The `/etc/hosts.allow` file is part of the tcp wrappers system.  While `iptables` and `ufw` offer more fine grained control, this can be used alongside them to help harden your server.

- **tcp wrappers**: This is a host-based access control system that allows you to control which hosts can connect to network services on your system.

- **`/etc/hosts.allow`**: This file contains rules that specify which hosts are allowed to connect to specific services.

- **`/etc/hosts.deny`**: This file contains rules that specify which hosts are denied access.

## Setup your TCP Wrappers

### Edit your hosts deny File

To only allow certain users or addresses to connect to your SSH server, you must first deny access to **ALL** users.  Afterward, you can configure which users are allowed to connect to your server.

To **DENY** access to **ALL** users, we need to first **create a backup of** `/etc/hosts.deny`, and then edit the `/etc/hosts.deny` file.

1. **Create a backup**

    In your terminal, type the following command:

    ```bash
    sudo cp /etc/hosts.deny /etc/hosts.deny.backup
    ```

    We can use this file if we mess something up, or just want to revert back to the original settings.

2. **Edit your hosts.deny file**:

    In your terminal type:

    ```bash
    sudo nano /etc/hosts.deny
    ```

    Add the following line to the file:

    ```bash
    sshd,sshdfwd-X11:ALL
    ```

    Let's break down the elements that we added to the `hosts.deny` file:

    - **sshd**: This specifies the SSH daemon.
    - **sshdfwd-X11**: This relates to ssh X11 forwarding.
    - **ALL**: DENY Access to all addresses, this can be overridden in hosts.allow

    Now, save the document with `Ctrl-O` followed by `Enter` and `Ctr-X`.

### Edit your hosts allow File

Now that we have set our `hosts.deny` file to deny access to all incoming SSH traffic, we can use the `hosts.allow` file to override that rule, and allow specific addresses to connect to your ssh server. In this example, we will only allow the computer with the IP address `192.168.1.5` to connect to our server in the local network. All other addresses will be denied access.

To do this, we need to first **make a backup** of `/etc/hosts.allow`, and then edit the `/etc/hosts.allow` file:

1. **Create a backup**

    In your terminal, type the following command:

    ```bash
    sudo cp /etc/hosts.allow /etc/hosts.allow.backup
    ```

    Just like the hosts.deny.backup file, we can use this to restore previous settings if we need to.

2. **Edit your `host.allow` file**:

    In your terminal, type the following command:

    ```bash
    sudo nano /etc/hosts.allow
    ```

    Add the following line to the file:

    ```bash
    sshd,sshdfwd-X11: 192.168.1.5
    ```

    Let's break down the elements that we added to the `hosts.allow` file:

    - **sshd**: This specifies the SSH daemon.
    - **sshdfwd-X11**: This relates to ssh X11 forwarding.
    - `192.168.1.5`: This is the IP address that is allowed to connect.

    Now, save the document with `Ctrl-O` followed by `Enter` and `Ctr-X`.

You should now have set up restricted access for SSH with the TCP wrappers.  You may need to restart your system for the changes to take effect.
