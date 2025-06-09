# Setting Up a Chroot Environment

## Contents

- [Setting Up a Chroot Environment](#setting-up-a-chroot-environment)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Chroot Environments](#understanding-chroot-environments)
      - [The Limitations of Chroot](#the-limitations-of-chroot)
  - [Planning the Chroot Setup](#planning-the-chroot-setup)
    - [Deciding on the Location For the Chroot Directories](#deciding-on-the-location-for-the-chroot-directories)
  - [Creating the Chroot Directory Structure](#creating-the-chroot-directory-structure)
  - [Copying Necessary Binaries and Libraries](#copying-necessary-binaries-and-libraries)
    - [sftp-server](#sftp-server)
    - [git](#git)
    - [nano](#nano)
    - [bash](#bash)
  - [Setting Up User Accounts](#setting-up-user-accounts)
  - [Configuring SSH for Chroot](#configuring-ssh-for-chroot)
  - [Testing the Chroot Environment](#testing-the-chroot-environment)
    - [Understanding Potential Issues and Troubleshooting](#understanding-potential-issues-and-troubleshooting)
  - [Change a Users Shell to RSSH](#change-a-users-shell-to-rssh)

## Introduction

Setting up chroot environments is a good way to enhance security of your server. In a nutshell, a chroot environment restricts a user's access to a specific directory tree, making it seem like that directory is the root of their file system. This limits what they can see and do on your server.

### Understanding Chroot Environments

Imagine a child playing in a sandbox. The sandbox is their limited world - they can play with the sand and toys inside, but they can't wander off into the wider garden or the house. A chroot environment is similar to this sandbox for users on your server.

**Definition**: A chroot environment, short for "change root", is a way to isolate a user or process to a specific directory tree on your system. For a user inside a chroot jail, this designated directory becomes their apparent root directory (`/`). They cannot access any files or directories outside of this restricted view.

**Security Benefits**: This isolation provides significant security benefits:

- **Limited Damage**: If a chrooted user's account is compromised, the attacker's access is limited to the chroot environment. They cannot easily access or modify critical system files or other user's data outside of this jail.
- **Restricted Actions**: By controlling which files and commands are available within the chroot, you can limit what actions a user can perform on your server.

Think of it like giving someone a locked briefcase with only the tools they need for a specific task. They can use those tools, but they can't access anything else you haven't provided.

#### The Limitations of Chroot

It's important to understand that while chroot adds a significant layer of security, it's not a completely foolproof solution. Think of our sandbox analogy again. While the child can't easily get out of the sandbox, a determined and clever child might find ways to dig under the walls or use tools in unexpected ways.

Here are some limitations of chroot:

- **Escape Vulnerabilities**: Historically, there have been ways for skilled users or attackers to "escape" the chroot jail by exploiting vulnerabilities in the kernel or in the setuid/setgid binaries within the chroot. While modern systems are more robust, it's not impossible.
- **Kernel Access**: The chrooted environment still shares the same kernel as the host system. Therefore, a user inside the chroot could potentially exploit kernel vulnerabilities to gain broader access.
- **Resource Exhaustion**: A malicious user within the chroot could potentially try to exhaust system resouces (like CPU or memory), affecting other users and processes on the server.

Chroot is a valuable *defense in depth* mechanism. It makes it significantly harder for unauthorized users to cause harm, but it should ideally be used in conjunction with other security best practices, such as keeping your system updated, using strong passwords, and limiting privileges of chrooted users as much as possible.

## Planning the Chroot Setup

This is a crucial step as it will determine how useful and secure your chroot environments will be. Think back to the "locked briefcase" analogy. Before we can even create the briefcase, we need to decide what tools the user *actually* needs inside to do their specific job. Putting in too many tools increases the risk if the briefcase is compromised. Putting in too few will make it impossible for the user to do their work.

Here are some key questions to consider during this planning phase:

1. **What specific tasks will these users need to perform on the server via SSH?** Will they be transferring files (using `scp` or `sftp`)? Will they need to run specific commands? Do they need a shell prompt at all?
2. **Based on these tasks, what are the absolute minimum set of commands and utilities they will require?** For example, if they only need to transfer files using `sftp`, they might not need a full shell like `bash`.
3. **What level of access do they need to the file system *within* their restricted environment?** Do they need to be able to create directories? Read and write files in specific locations within their chroot?

Answering these questions will help you determine which directories, binaries (executable programs), and libraries will be necessary to include in the chroot environment. The goal is to keep the chroot environment as minimal as possible, providing only what is strictly required for the intended purpose. This reduces the potential attack surface.

For example, if a user only needs to upload and download files securely, you might only need to provide the `sftp` subsystem of SSH and the necessary supporting libraries, without giving them a shell or other commands.

### Deciding on the Location For the Chroot Directories

A common and logical place to create chroot environments is within the `/home` directory. We can create a dedicated subdirectory there, perhaps prefixed with `chroot-`, followed by the username. For example:

- For a username `alice`, the chroot directory could be `/home/chroot-alice`.
- For a username `bob`, the chroot directory could be `/home/chroot-bob`.

This approach keeps the chroot environments organized and separate from the user's actual home directories (if they had any non-chrooted access, which in this case they won't).

Why `/home`? It's a standard location for user-related data, and it often has reasonable default permissions.

## Creating the Chroot Directory Structure

Think of building the foundation for our "sandbox". We need to create the basic folders that will make the chroot environment functional. Inside the `/home/chroot-username` directory (where `username` is the name of the user), we'll need to create some essential subdirectories. The most common ones are:

- `bin`: This directory will hold the essential executable files (the commands) that the user will be allowed to run within the chroot. For our case, this will likely include `sftp-server` (part of the SSH suite), `git`, and perhaps the executable for your chosen editor (`nano` or `vim`), and a shell environment which is typically `bash`.
- `lib` and `lib64`: These directories will contain the shared libraries that the executables in the `bin` directory depend on. Think of libraries as supporting files that the main programs need to run correctly.
- **Potentially other directories**: Depending on the specific needs of  your chosen executables, you might need other directories like `/usr`, `/etc`, or `/dev`.

To create these basic directories for our example user `alice`, you would use the `mkdir` command in the terminal.  Assuming you are logged in with `sudo` privileges, you would run something like this:

```bash
sudo mkdir -p /home/chroot-alice/{bin,lib,lib64}
```

The `-p` flag ensures that if the parent directory (`/home/chroot-alice`) doesn't exist, it will be created as well.  We use curly braces `{}` to create multiple directories at once.

## Copying Necessary Binaries and Libraries

The next crucial step is copying the necessary binaries and libraries.  This is where we put the actual "tools" and their "helper parts" into the chroot "briefcase."

The key is to copy only the *essential executables* and the *exact* libraries they depend on. Copying too much will make the chroot environment larger and potentially introduce security risks.

How do we know which libraries a program needs? This is where the handy command `ldd` comes in. `ldd` (short for "list dynamic dependencies") shows you the shared libraries that a given executable depends to run.

For example, we'll first figure out the dependencies for `sftp-server`. Open your terminal and run the following command:

```bash
ldd /usr/lib/openssh/sftp-server
```

(Note: The exact path to `sftp-server` might be slightly different on your system, but it's usually in a directory related to `openssh`.)

The output of this command will be a list of shared libraries that `sftp-server` needs. Each line will typically show the name of a library (like libc.so.6) and the path to where that library is located on your system (like /lib/arm-linux-gnueabihf/libs.so.6).

Our next step will be to copy the `sftp-server` executable itself into the `/home/chroot-alice/bin` directory, and then copy all the listed libraries into the appropriate `lib` or `lib64` directory within the chroot.

We'll repeat this process for `git` and your chosen editor (`nano` or `vim`).

> Why is it important to use `ldd`? Because simply copying the executable file isn't enough. Without its dependent libraries, the program won't be able to run inside the isolated chroot environment. It's like having a car (the executable) but not the engine or wheels (the libraries) - it's not going anywhere!

Now you should have the list of libraries that `sftp-server` depends on, let's proceed with **Copying Necessary Binaries and Libraries** into our chroot environment for `alice` (which we created at `/home/chroot-alice`).

### sftp-server

Here's what you need to do:

1. **Copy the `sftp-server` executable**:

    ```bash
    sudo cp /usr/lib/openssh/sftp-server /home/chroot-alice/bin/
    ```

    (or `sudo cp /bin/sftp-server /home/chroot-alice/bin/` if that's the path `ldd` showed for you).

2. **Examine the output of `ldd` again**. For each library listed (e.g., `/lib/arm-linux-gnueabihf/libc.so.6`), you need to copy that library file to the corresponding directory within your chroot environment.

    - If the library path starts with `/lib/`, copy it to `/home/chroot-alice/lib`. For example:

        ```bash
        sudo cp /lib/arm-linux-gnueabifh/libc.so.6 /home/chroot/alice/lib/
        ```

    - If the library path starts with `/usr/lib/`, copy it to `/home/chroot-alice/usr/lib/` (you might need to create the `/usr/lib` directory inside your chroot first if it doesn't exist: `sudo mkdir -p /home/chroot-alice/lib`). For example:
  
        ```bash
        sudo mkdir -p /home/chroot-alice/usr/lib
        sudo cp /usr/lib/your-library.so.X /home/chroot-alice/usr/lib/
        ```

        (Replace `your-library-.so.X` with the actual library name from the `ldd` output).

    - If the library path starts with `/lib64/`, copy it to `/home/chroot-alice/lib64/`. For example:

        ```bash
        sudo cp /lib64/your-library.so.X /home/chroot-alice/lib64/
        ```

        (Replace your-library.so.X with the actual library name from the ldd output).

It's important to copy the *actual library files*, not just the symbolic links (which are like shortcuts). `ldd` usually shows the resolved path to the actual library.

This might seem a bit tedious, but its crucial for creating a self-contained chroot environment.

Let's run through a output for `ldd /usr/lib/openssh/sftp-server` and go through the dependencies.  Here is a sample output:

```shell
  linux-vdso.so.1 (0x0000792d50bd6000)
  libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x0000792d50600000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000792d50200000)
  /lib64/ld-linux-x86-64.so.2 (0x0000792d50bd8000)
```

> **What about the library `linux-vdso.so.1`**? `linux-vdso.so.1` is a bit special. It's a virtual dynamic shared object provided by the Linux kernel itself. It's not a real file on your file system that you can copy in the traditional way. So, for the `linux-vdso.so.1`, you don't need to do anything! You can simply ignore it in the `ldd` output when your thinking about which files to copy.

Let's move on to the first real library dependency: `libcrypto.so.3` located at `/lib/x86_64-linux-gnu/libcrypto.so.3`.

Now, to copy this library into our chroot environment for `alice` (`/home/chroot-alice`), we need to put it in the corresponding `lib` directory within the chroot. Since the original path starts with `/lib/`, we will copy it to `/home/chroot-alice/lib/`.

Here's the command you would need to run:

```bash
sudo cp /lib/x86_64-linux-gnu/libcrypto.so.3 /home/chroot-alice/lib/
```

After running this command, the `libcrypto.so.3` library will be present within the `lib` directory of Alice's chroot environment. This ensures that when `sftp-server` tries to use this library, it will be able to find it within its restricted view of the file system.

Let's move on to the next library listed from our `ldd` command: `libc.so.6` located at `/lib/x86_64-linux-gnu/libc.so.6`. This is a very common and fundamental library that almost every program on a Linux system relies on.

Just like we did with `libcrypto.so.3`, we need to copy `libc.so.6` into the corresponding `lib` directory within our chroot environment for `alice`:

```bash
sudo cp /lib/x86_64-linux-gnu/libc.so.6 /home/chroot-alice/lib/
```

This ensures that sftp-server will have access to the core system functions it needs to run.

We successfully copied all the necessary libraries for the `sftp-server` command into the chroot environment for the user `alice`. This was a crucial step in making sure that `sftp` will be functional within the restricted environment.

### git

Now, we need to repeat this process for the other commands we want to allow: `git` and your chosen editor.

Let's move on to `git`.

First let's copy the git binary into Alice's chroot environment:

```bash
sudo cp /usr/bin/git /home/chroot-alice/bin/
```

Then we need to list its dependencies:

```bash
ldd /usr/bin/git
```

Here is an example output:

```shell
  linux-vdso.so.1 (0x00007160248ef000)
  libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x0000716024838000)
  libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x000071602481c000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000716024000000)
  /lib64/ld-linux-x86-64.so.2 (0x00007160248f1000)
```

Our first library to copy for `git` is: `libpcre2-8.so.0` located at `/lib/x86_64-linux-gnu/libpcre2-8.so.0`. This library provides support for Perl Compatible Regular Expressions, which git uses for pattern matching.

Let's copy this library into the `lib` directory within Alice's chroot:

```bash
sudo cp /lib/x86_64-linux-gnu/libpcre2-8.so.0 /home/chroot-alice/lib/
```

The next library for `git` is: `libz.so.1` located at `/lib/x86_64-linux-gnu/libz.so.1`. This library provides data compression and decompression functionalities, which git uses for handling repository data efficiently.

Let's copy this library into the `lib` directory within Alice's chroot:

```bash
sudo cp /lib/x86_64-linux-gnu/libz.so.1 /home/chroot-alice/lib/
```

The next library for `git` is: `libc.so.6`. We already copied this library for `sftp-server`, so we can skip this one.

The last library for `git` is: `ld-linux-x86-64.so.2` located at `/lib64/ld-linux-x86-64.so.2`. This is a dynamic linker dependency.  We need to copy this crucial file into the `/home/chroot-alice/lib64/` directory:

```bash
sudo cp /lib64/ld-linux-x86-64.so.2 /home/chroot-alice/lib64/
```

That should cover the dependencies needed for `git`.  

### nano

We can now move on to our text editor.  For this example we will use `nano`. We'll first copy the executable and get a list of dependencies for `nano`.

Copy the executable into Alice's chroot:

```bash
sudo cp /usr/bin/nano /home/chroot-alice/bin/
```

Now list the dependencies for `nano`:

```bash
ldd /usr/bin/nano
```

Here is an example output in the terminal:

```shell
  linux-vdso.so.1 (0x0000736c6e544000)
  libncursesw.so.6 => /lib/x86_64-linux-gnu/libncursesw.so.6 (0x0000736c6e4a1000)
  libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x0000736c6e46f000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000736c6e200000)
  /lib64/ld-linux-x86-64.so.2 (0x0000736c6e546000)
```

From this output, it looks like the only new libraries we need to add to Alice's chroot is `libncursesw.so.6`, and `libtinfo.so.6`.

Let's get those dependencies copied over:

```bash
sudo cp /lib/x86_64-linux-gnu/libncursesw.so.6 /home/chroot-alice/lib/
sudo cp /lib/x86_64-linux-gnu/libtinfo.so.6 /home/chroot-alice/lib/
```

### bash

Finally, so that our user can interact with the server, we will provide them with the shell prompt `bash`.

Let's copy over the executable to Alice's chroot:

```bash
sudo cp /bin/bash /home/chroot-alice/bin/
```

Now, let's take a look at the dependencies we will need for `bash`:

```bash
ldd /bin/bash
```

Here is a example output:

```shell
  linux-vdso.so.1 (0x00007de3b301d000)
  libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007de3b2e6a000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007de3b2c00000)
  /lib64/ld-linux-x86-64.so.2 (0x00007de3b301f000)
```

It looks like we have already copied these libraries into Alice's chroot environment, so we don't need to copy any additional libraries.

We've now covered gathering all of the necessary binaries and libraries.

## Setting Up User Accounts

Think of this step as creating the "inhabitants" for our carefully constructed "sandboxes". We need to create the actual user accounts on the server that will be restricted to these chroot environments.

The standard way to create new user accounts on most Linux distributions is using the `adduser` command. This command is interactive and will guide you through the process of creating a new user, setting their password, and optionally adding some basic user information.

To create a new user named, for example, `alice` (who will be confined to the `/home/chroot-alice` directory we've been working on), you would run the following command in your terminal:

```bash
sudo adduser alice
```

When you run this, you will be prompted to:

1. Enter a password for `alice`. It's crucial to choose a **strong and unique password** for each user to maintain the security of your server.
2. Re-enter the password to confirm it,
3. Enter optional full name, room number, work phone, home phone, and other information. You can usually leave these fields blank by pressing Enter if you don't need them.
4. Finally, you'll be asked to confirm the information you've entered. Type `y` and press Enter to create the user account.

You'll need to repeat this process for each new user you want to create who will have a chroot environment (e.g., for a user named `bob`, you'd run `sudo adduer bob`, and so on, making sure you've created a corresponding chroot directory like `/home/chroot-bob` and copied the necessary files into it).

> Why is creating separate user accounts important? Each user account has its own unique identifier and permissions on the system. This allows you to manage access and track actions on a per-user basis.

After creating the user account, we'll later link it to its specific chroot directory.

## Configuring SSH for Chroot

Now we need to tell the SSH server (`sshd`) to actually enforce the chroot restriction for the new users.

Think of it like setting the rules for our "sandbox". We've built the sandbox (the chroot directory) and put the toys (binaries and libraries) inside. Now, we need to tell the "playground supervisor" (the SSH server) that certain kids (our new users) are only allowed to play in their specific sandbox and can't wander off.

We configure the SSH server by editing its main configuration file, which is usually located at `/etc/ssh/sshd_config`. **Be very careful when editing this file**, as incorrect changes can lock you out of your server! It's always a good idea to make a backup of the file before making any modifications. You can do this with the command:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

Once you have a backup, you'll need to open the `sshd_config` file with a text editor that has sudo privileges (like `sudo nano /etc/ssh/sshd_config` or `sudo vim /etc/ssh/sshd_config`).

Inside this file, you'll be looking for a section that starts with a comment like `#Subsystem sftp / usr/lib/openssh/sftp-server`. We need to modify this and add a new directive for chrooting.

The key directive we'll be using is `ChrootDirectory`. This directive specifies the path to the directory that should become the root directory for the user or group.

There are a couple of ways to apply the `ChrootDirectory` directive:

1. **Globally for a specific group**: You can create a dedicated group (e.g., `chrooted_users`) and then specify that all members of this group should be chrooted to a particular directory (though typically, each user will have their own chroot directory).
2. **For specific users**: You can specify the `ChrootDirectory` for individual users. This is likely what we'll want to do in our case, where each user (`alice`, `bob`, etc.) will have their own unique chroot environment.

To apply the chroot for a specific user, you'll typically use a `Match` block in the `sshd_config` file. This allows you to apply specific configurations only to users or groups that match certain criteria.

Here's an example of how you might configure our user `alice`. Add these lines at the end of your `sshd_config` file:

```bash
Match User alice
    ChrootDirectory /home/chroot-alice
    #ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

Let's break down these lines:

- `Match User alice`: This tells SSH that the following settings should apply to the user named `alice`.
- `ChrootDirectory /home/chroot-alice`: This is the crucial part! It specifies that for `alice`, the `/home/chroot-alice` directory will become their root directory with they log in via SSH.
- `ForceCommand internal-sftp`: This line is important if you *only* want to allow SFTP access and not a full shell, even if you copied `bash`. The `internal-sftp` is a built-in SFTP server within OpenSSH. If you want to allow a shell as well, you might omit or comment out this line. **Since we added git and a text editor, we will comment out the `ForceCommand` line.**
- `AllowTcpForwarding no` and `X11Forwarding no`: These are additional security measures that disable TCP forwarding and X11 forwarding for this user, further limiting their potential actions. These are generally good practices for chrooted users.

You'll need to add a similar `Match` block for each user you want to chroot, making sure to replace `alice` with their username and `/home/chroot-alice` with their respective chroot directory.

After you've made these changes to the `/etc/ssh/sshd_config` file, you need to **restart the SSH service** for the changes to take effect. You can do this with the command:

```bash
sudo systemctl restart sshd
```

> **Important Security Note**: The owner of the chroot directory itself (`/home/chroot-alice` in our example) **must be root**, and it should not be writable by the chrooted user or any other non-root user. If the chroot directory is writable by the user, they might be able to escape the chroot.

After restarting the SSH service, it's a good idea to quickly check if the service is running correctly. You can do this with the command:

```bash
sudo systemctl status sshd
```

Look for the line that says "Active: active (running)" to confirm that the service restarted without any erros.

## Testing the Chroot Environment

We should test if our "sandbox" is working as intended and if our "playground supervisor" (the SSH server) is enforcing rules correctly.

To test this, you'll need to try logging into your server using the `alice` user account via SSH from another computer on your network. You'll use the standard `ssh` command followed by the username and the IP address of your server:

```bash
ssh alice@your_ip_or_hostname
```

When you connect, you'll be prompted for the password you set for the `alice` user.

Once you're logged in, the first thing to check is your current location in the file system. If the chroot is working correctly, when you run the command `pwd` (print working directory), you should see `/` as the output. This is because, from `alice`'s perspective within chroot, the `/home/chroot-alice` directory is now the root of the file system.

Next, try listing the contents of the root directory using the command `ls -l`. You should only see the `bin`, `lib`, `lib64`, and any other directories you created inside `/home/chroot-alice`. You should **not** be able to see the regular top-level directories of your server like `/etc`, `/var`, `/usr` (unless you specifically created these inside `/home/chroot-alice`).

Try running some of the commands we copied into the `bin` directory, like `ls`, `pwd`, `git --version`, or `nano --version` (or `vim --version`). These should hopefully work.

Now, try to navigate outside of what should be the chroot environment. For example, try to change directory to `/home` or `/etc` using the `cd` command:

```bash
cd /home
ls -l
```

If the chroot is working correctly, you should either get an error message (like "No such file or directory") or you might still appear to be in the `/` directory of the chroot. You definitely should not be able to access the actual `/home` directory of your server and see other directories.

### Understanding Potential Issues and Troubleshooting

Here are some common issues you might encounter when testing your chroot environment and how to troubleshoot them:

- **"Command not found" errors**: If you try to run a command within the chroot and get this error, it likely means that either the executable for that command wasn't copied into the `bin` directory within the chroot, or one of its required libraries is missing from the `lib` or `lib64` directories. In this case, you'll need to go back, and use the `ldd` command, and make sure all the listed dependencies are present in the chroot.
- **Login failure via SSH**: If you can't even log in as the chrooted user, double-check the sshd_config file for any typos in the username or the `ChrootDirectory` path. Also, ensure that you have restarted the `sshd` service after making changes to the configuration file. The permissions on the chroot directory itself are also crucial; it should be owned by `root` and not writable by the chrooted user.
- **Unexpected access to the file system**: If, after logging in, you can navigate to directories outside of `/` (the apparent root of the chroot), it could indicate that an issue with the `ChrootDirectory` configuration in `sshd_config` or incorrect permissions on the chroot directory. Review the `sshd_config` and ensure the chroot directory's ownership and permissions are correctly set (owned by root, not writable by the user).
- **SFTP not working (if you intended only SFTP)**: If you used the `ForceCommand internal-sftp` directive and SFTP isn't working, ensure that the `sftp-server` binary and its libraries were correctly copied to the chroot.

## Change a Users Shell to RSSH

For even tighter control, you could explore using restricted shells like `rssh`. `rssh` is a shell that only allows certain limited operations (like `scp` and `sftp`) and prevents users from executing arbitrary commands. This can be a good alternative to providing a full `bash` shell if your users primarily need file transfer capabilities. If you decide to use `rssh`, you could configure SSH to use `rssh` as the user's shell instead of `bash`.

To change a user's shell to `rssh` in the `/etc/ssh/sshd_config` file, you would typically modify the `Match User` block for that specific user. Instead of just setting the `ChrootDirectory`, you would also add a `ForceCommand` directive that specifies the `rssh` command with the allowed options.

First, you would need to make sure that the `rssh` package is installed on your server.

```bash
sudo apt update
sudo apt install rssh
```

Once `rssh` in installed, you can edit your `/etc/ssh/sshd_config` file (remember to make a backup first!):

```bash
sudo nano /etc/ssh/sshd_config
```

Then, within the `MatchUser` block for the user you want to restrict (e.g., `alice`), you would add the `ForceCommand` line. For example, if you only want to allow SFTP access with `rssh`, the block might look like this:

```shell
Match User alice
    ChrootDirectory /home/chroot-alice
    ForceCommand /usr/bin/rssh -sftp
    AllowTcpForwarding no
    X11Forwarding no
```

Here, `/usr/bin/rssh` is the path to the `rssh` executable, and -sftp is an option that tells `rssh` to only allow SFTP connections.

If you wanted to allow both SFTP and `scp` (secure copy) with `rssh`, the `ForceCommand` would be:

`ForceCommand /usr/bin/rssh -sftp -scp`

After making these changes, you would again need to restart the SSH service:

```bash
sudo systemctl restart sshd
```

Now, when `alice` logs in via SSH, instead of getting a regular shell (if we had allowed it), the `rssh` command will be executed, limited them to only the specified actions (in these examples, SFTP and/or SCP) within their chroot environment. Any attempt to run other commands will be blocked by `rssh`.

Remember to only include the `rssh` options for the functionality you want to allow for that specific user.
