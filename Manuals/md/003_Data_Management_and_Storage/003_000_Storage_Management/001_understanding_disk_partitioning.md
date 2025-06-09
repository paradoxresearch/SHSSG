# Understanding Disk Partitioning

## Contents

- [Understanding Disk Partitioning](#understanding-disk-partitioning)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Common Partitioning Schemes](#common-partitioning-schemes)
  - [Essential Partitions for Linux Systems](#essential-partitions-for-linux-systems)
  - [Command-Line Tools for Partitioning](#command-line-tools-for-partitioning)
  - [Creating Partitions with Command-Line Tools](#creating-partitions-with-command-line-tools)
    - [`fdisk`](#fdisk)
    - [`gdisk`](#gdisk)
    - [`parted`](#parted)
      - [`parted` - Interactive](#parted---interactive)
      - [`parted` - Non-Interactive](#parted---non-interactive)
  - [Formatting Partitions](#formatting-partitions)
  - [Mounting Partitions](#mounting-partitions)
    - [Temporary Mounting](#temporary-mounting)
    - [Permanent Mounting](#permanent-mounting)

## Introduction

Imagine you have a big filing cabinet – that's like your server's hard drive. Now, if you just throw all your documents in there without any order, it'll become a chaotic mess, right? **Disk partitioning** is like creating separate, organized drawers within that filing cabinet. Each drawer (partition) can hold different types of information and can be managed independently.

In the context of your Linux server, partitioning allows you to divide your physical hard drive into several logical sections. This brings several key benefits:

- **Organization**: You can keep different types of data separate, like your system files, user data, and application logs.
- **Security**: If one partition experiences an issue or corruption, it's less likely to affect the data on other partitions. For example, if your user data is on a separate partition from the core system, a problem with user files won't necessarily break your operating system.
- **Flexibility with File Systems**: Different partitions can use different file systems, which are like the indexing systems within our filing cabinet drawers. Some file systems are better suited for certain types of data or tasks.
- **Easier Backups and Recovery**: Backing up and restoring specific types of data becomes more manageable when they are isolated on separate partitions.

Think of it this way: having separate partitions is like having dedicated rooms in your house. The kitchen is for cooking, the bedroom is for sleeping, and the office is for work – each with its own purpose and organization.

## Common Partitioning Schemes

Now that we covered ***why*** we would want disk partitioning, let's begin to discuss the ***how*** – specifically, the common ways disks are partitioned. These are called partitioning schemes. The two main schemes you'll encounter are **MBR (Master Boot Record)** and **GPT (GUID Partition Table)**. Think of these as different blueprints for organizing those drawers in our filing cabinet.

**MBR** is an older standard. It's been around for a while and has a few limitations. Imagine an older blueprint that only allows for a limited number of drawers (specifically, only four primary partitions). Also, this blueprint can only address a certain size of filing cabinet (it has a 2 terabyte disk size limit).

**GPT** is the more modern scheme. It's like an updated blueprint that overcomes many of MBR's limitations. GPT allows for a significantly larger number of partitions (theoretically, many more than you'd likely ever need) and supports much larger disk sizes – way beyond 2 terabytes. It also includes features for better data integrity.

Another important concept related to partitioning schemes is the **boot partition**. This is a small section of your disk that contains the necessary files to start your operating system. Think of it as the special key to open the main filing cabinet and access all the drawers inside. On Linux systems using GPT, you'll often see an **EFI System Partition (ESP)** which serves this purpose.

In most modern servers, especially those with larger drives, **GPT is the preferred and recommended partitioning scheme** due to its flexibility and capabilities. However, you might still encounter systems using MBR, especially older ones.

## Essential Partitions for Linux Systems

With some knowledge of the "why" and the "how" of the blueprints (partitioning schemes), let's talk about the "what" – specifically, which partitions are generally considered essential for a Linux system, like your Linux Server.

While you can technically run a very basic Linux system with just one partition, it's highly recommended to have at least two, and often more, for better organization, stability, and security. Think of these as the most important drawers in your server's filing cabinet.

Here are the key partitions you'll typically encounter and why they are important:

- `/` **(Root Partition)**: This is the heart of your Linux system. It's where the operating system files, system configurations, installed applications, and everything else essential for running your server reside. Think of it as the main compartment of your filing cabinet, holding all the core documents and the system to run the entire office. It's usually mounted at the root directory (`/`).
  
- `swap` **Partition**: This is a special type of partition that acts as virtual RAM. If your server runs out of physical memory (RAM), the system can temporarily move less actively used data to the swap partition to free up RAM for more urgent tasks. Think of it as a temporary overflow drawer for when your main workspace is full. While modern systems with sufficient RAM might use a swap file instead of a dedicated partition, a swap partition is still a common and sometimes recommended practice, especially for servers. The size of your swap partition often depends on your RAM and intended use, but a general guideline is to have at least as much swap as you have RAM, or sometimes double that for systems with less RAM.

Optionally, you might also have these partitions:

- `/home` **Partition**: This is where user-specific data, like personal files, documents, and user configurations, are typically stored. Keeping `/home` on a separate partition is excellent for organization and security. For example, if you need to reinstall the operating system, you can usually do so without affecting the data in your `/home` partition. Think of this as individual user drawers within the filing cabinet.

- `/var` **Partition**: This partition often stores variable data such as logs, temporary files, printer queues, and email spools. Separating `/var` can be beneficial for stability, especially on busy servers that generate a lot of log data. If `/var` fills up, it can cause issues with system operation, and having it separate can prevent it from impacting the root partition. Think of this as a dedicated drawer for all the ongoing paperwork and temporary documents.

For a basic Linux Server setup, you'll almost always have a `/` (root) and a `swap` partition. A separate `/home` and `/var` are often recommended for better management, especially in multi-user environments or on servers handling significant data.

## Command-Line Tools for Partitioning

Now that you have a good understanding of the essential partitions, let's get familiar with the tools we'll use to actually manage them in your Ubuntu Server environment. Since we're focusing on command-line tools, you'll become best friends with a few powerful utilities.

The three main command-line tools you'll encounter for disk partitioning are:

- `fdisk`: This is a classic, widely used tool that primarily works with **MBR partition tables**. While it can handle GPT partitions to some extent, `gdisk` **is generally preferred for GPT**. `fdisk` provides an interactive, menu-driven interface for managing partitions.

- `gdisk`: As mentioned, `gdisk` is the go-to tool for managing disks with **GPT partition tables**. It offers a similar interactive interface to `fdisk` but is specifically designed for the features and capabilities of GPT.

- `parted`: This is a more versatile tool that can work with both **MBR and GPT partition tables**. Unlike `fdisk` and `gdisk`, `parted` can be used both interactively and non-interactively in scripts, making it very powerful for automation.

To see the current partitioning layout of your server's disks, you can use these commands:

- `sudo fdisk -l` : This command will list all the disks and their partitions that `fdisk` can recognize. You'll see information like the disk name (`/dev/sda`, `/dev/sdb`, etc.), the partition table type (if recognized), and the individual partitions with their sizes and types.

- `sudo parted -l` : This command provides a more comprehensive overview of your disks and their partitions, including the partition table type (like gpt or msdos - which is another name for MBR) and the file systems on each partition (if any).

It's important to note that you'll typically need `sudo` (superuser do) privileges to run these commands, as they involve accessing and potentially modifying system-level information.

Think of these tools as different types of wrenches in your server administration toolkit – some are better suited for specific types of bolts (partition tables) and some offer more advanced features.

## Creating Partitions with Command-Line Tools

We'll focus on using `fdisk` or `gdisk` for creating partitions, as they provide interactive ways to manage your disk layout.

**Important Note**: Creating or modifying partitions is a powerful operation that can lead to data loss if not done carefully. **Always ensure you have backups of any important data before proceeding with partitioning**. It's also a good idea to practice these commands in a virtual machine environment first if you're not completely comfortable.

Let's assume you have a new, unpartitioned disk connected to your Linux server, and it's identified as `/dev/sdb` (you can find the actual device name using `sudo fdisk -l` or `sudo parted -l`).

Here's a general outline of the steps you'd take using `fdisk` (for MBR) or `gdisk` (for GPT):

### `fdisk`

**Using** `fdisk` **(for MBR)**:

1. Open `fdisk` for the target disk:

    Open your terminal and type the following command, then press Enter:

    ```Bash
    sudo fdisk /dev/sdb
    ```

    You'll enter an interactive command prompt.

2. **View existing partitions (if any)**: Type `p` and press Enter.

3. **Create a new partition**: Type `n` and press Enter.
   - You'll be asked to choose a partition type: `p` for primary or `e` for extended (you can have at most four primary partitions, or fewer primary and one extended containing logical partitions).

        ```shell
        Partition type
           p   primary (0 primary, 0 extended, 4 free)
           e   extended
        Select (default p):
        ```

        Since we want to create a primary partition, just press Enter to accept the default `p`.

        Next, it will ask for the partition number:

        ```shell
        Partition number (1-4, default 1):
        ```

        Press Enter to accept the default `1` for the first primary partition.

   - Then, you'll be prompted for the **first sector**. You can usually accept the default value by pressing Enter.
   - Finally, you'll be asked for the **last sector**. Instead of specifying a sector number, it's often easier to define the size of the partition. You can usually accept the default value by pressing Enter. You can specify the size using `+<size>{M,G}` (e.g., `+20G` for a 20 GB partition).

4. **Set the partition type (optional but important for certain partitions like swap)**: Type `t` and press Enter. You'll be asked for the partition number. For a swap partition, you'd typically enter the corresponding code (e.g., `82` for Linux swap). You can list the available codes by typing `L`.

5. **Write the changes to disk**: Once you're satisfied with your partition layout, type `w` and press Enter. **This is the point where changes are actually applied to the disk, so be absolutely sure before you do this!**

6. **Quit without saving**: If you make a mistake or decide not to proceed, type `q` and press Enter.

### `gdisk`

**Using gdisk (for GPT)**:

1. Open `gdisk` for the target disk:

    Open your terminal and type the following command, then press Enter:

    ```Bash
    sudo gdisk /dev/sdb
    ```

    If the disk has a GPT partition table (or if `gdisk` detects it should), you'll see some information about the GPT structures. You'll then be presented with the `gdisk` command prompt: `Command (? for help):`.

2. **View existing partitions (if any)**:

    To see the current partition layout, type `p` and press Enter. If it's a new disk with a GPT table, you might not see any partitions listed initially.

3. **Create a new partition**:

   - Type `n` and press Enter to create a new partition.

   - `gdisk` will ask for the partition number. Let's enter `1` for the first partition and press Enter.

   - Next, it will prompt for the **first sector**. You can usually accept the default by pressing Enter.

   - Then, it will ask for the last sector. Similar to `fdisk`, you can specify the size using `+<size>{M,G}`. Let's create a 30GB partition this time, so enter `+30G` and press Enter.

   - Finally, `gdisk` will ask for the **type code or GUID**. For a standard Linux filesystem partition, you can often accept the default by pressing Enter (which is usually `8300` for "Linux filesystem"). If you wanted to create a swap partition, you would enter the code for that (which is `8200`). You can list the available type codes by typing `L`.

4. **Verify the new partition**:

    Type `p` and press Enter again to display the partition table. You should now see your newly created partition (`/dev/sdb1`) with its size and type.

5. **Write the changes to disk**:

    **Only proceed if you are absolutely sure about the changes!** To save the partition table to the disk, type `w` and press Enter. You'll be asked for confirmation – type `y` and press Enter. You should see a message indicating that the operation was successful.

6. **Quit without saving**:

    If you make a mistake or decide not to proceed, you can type q and press Enter to exit gdisk without saving any changes.

> Remember, these tools are interactive. Don't be afraid to explore the different commands by typing m at the prompt to see a menu of available options.

### `parted`

Now, let's explore the `parted` command, which offers a powerful and versatile way to manage disk partitions. We'll start with its interactive mode.

Think of parted as a more advanced tool in your kit, capable of handling both MBR and GPT partition tables with a consistent interface.

#### `parted` - Interactive

**Using** `parted` **in Interactive Mode**:

1. **Open** `parted` **for the target disk**:

    In your terminal, type the following command, replacing /dev/sdb with your target disk, and press Enter:

    ```bash
    sudo parted /dev/sdb
    ```

    You'll enter the parted interactive prompt, which usually looks like:

    ```shell
    GNU Parted 3.4
    Using /dev/sdb
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted)_
    ```

2. **View existing partitions:**

    To see the current partition table and partitions, type `print` and press Enter. This will display information about the disk, its size, the partition table type, and any existing partitions with their numbers, types, start and end points, sizes, and flags.

3. **Create a new partition**:

    To create a new partition, type `mkpart` and press Enter.

    `parted` will then ask you for several pieces of information:

   - **Partition type?** You can specify `primary`, `logical`, or `extended`. For most standard partitions, you'll use `primary`.
   - **File system type?** This is optional and doesn't actually format the partition, but it can be helpful for `parted` to set appropriate flags. You can enter something like `ext4` or just leave it blank and press Enter.
   - **Start?** You can specify the starting point of the partition in megabytes (MB), gigabytes (GB), or as a percentage of the disk. For example, `0%` or `0GB` to start at the beginning of the free space.
   - **End?** Similarly, you specify the end point of the partition using a size or percentage. For example, `20GB` to create a 20GB partition from the start point, or `50%` to make it half the disk.

    For example, to create a 20GB primary partition starting at the beginning of the free space, you might enter:

    ```shell
    mkpart primary ext4 0GB 20GB
    ```

4. **Set partition flags (optional but important for boot partitions)**:

    For certain partitions, like the boot partition in a GPT scheme, you might need to set flags. To see available flags, select a partition using `select <partition_number>` (e.g., `select 1`) and then type `help set`. To set a flag, use `set <flag_name> on` (e.g., `set boot on`). To unset a flag, use `set <flag_name> off`.

5. **Apply the changes**:

    Unlike `fdisk` and `gdisk`, **changes in** `parted` **are applied immediately by default in recent versions**. However, it's still good practice to be sure of your commands before executing them. You can use `print` again to verify your changes.

6. **Quit parted**:

    To exit the interactive mode, type `quit` and press Enter.

#### `parted` - Non-Interactive

This is where `parted` really shines for automation and scripting, as you can pass all the commands directly on the command line without entering the interactive prompt.

The general syntax for using `parted` non-interactively is:

```bash
sudo parted <device> <command> [parameters...]
```

Here, `<device>` is the disk you want to work with (e.g., `/dev/sdb`), `<command>` is the action you want to perform (e.g., `mklabel`, `mkpart`, `set`), and `[parameters...]` are the specific details for that command.

Let's look at some common examples:

1. **Creating a GPT partition table**:

    To initialize a disk with a GPT partition table, you can use the mklabel command:

    ```bash
    sudo parted /dev/sdb mklabel gpt
    ```

2. **Creating a primary partition with a specific size and file system type hint**:

    To create a 25GB primary partition, starting at 0GB and ending at 25GB, with a file system type hint of `ext4`, you would use the `mkpart` command:

    ```bash
    sudo parted /dev/sdb mkpart primary ext4 0GB 25GB
    ```

    Notice how all the parameters we entered interactively before are now provided directly on the command line.

3. **Setting a flag on a partition**:

    To set the `boot` flag on the first partition (partition number 1), you would first select the disk and then use the `set` command with the partition number, flag name, and state (`on` or `off`):

    ```bash
    sudo parted /dev/sdb set 1 boot on
    ```

4. **Combining multiple operations in a script**:

    You can put these commands sequentially in a shell script to perform multiple partitioning tasks at once. For example, a script to create a GPT partition table and two partitions might look like this:

    ```bash
    #!/bin/bash
    device="/dev/sdb"

    sudo parted "$device" mklabel gpt
    sudo parted "$device" mkpart primary ext4 0GB 50GB
    sudo parted "$device" mkpart primary swap 50GB 52GB # Create a 2GB swap partition
    ```

**Important Considerations for Non-Interactive Mode**:

- **Error Handling**: When using `parted` in scripts, it's crucial to include error handling to gracefully manage potential issues.
- **Be Careful with Parameters**: Double-check the device name, partition numbers, sizes, and other parameters in your commands or scripts to avoid unintended changes.
- **No Prompts**: Since it's non-interactive, parted won't ask for confirmation. Be absolutely sure your commands are correct before running them.

## Formatting Partitions

Now that you know how to create partitions using command-line tools, the next crucial step is to make them usable by the operating system. This is where **formatting** comes in.

Think of the partitions we've created as empty drawers in our filing cabinet. To actually store documents in an organized way, we need to set up an indexing system within each drawer. This indexing system is analogous to a **file system**.

A **file system** defines how data is stored and retrieved on a partition. It manages files, directories, and metadata (information about the files). Different operating systems and even different use cases often benefit from specific file systems.

For Linux systems, some of the most common file systems you'll encounter are:

- `ext4`: This is a very common and robust general-purpose file system that is the default for many Linux distributions. It's known for its performance and reliability.
- `XFS`: Another high-performance file system, often favored for large storage systems and parallel I/O workloads.
- `swap`: While technically not a "file system" in the same way as `ext4` or `XFS`, we need to format our swap partition as a swap area so the system can use it for virtual memory.

To actually format a partition, we use the `mkfs` command (short for "make file system"). The basic syntax is:

```bash
sudo mkfs.<filesystem_type> <device_partition>
```

For example:

- To format the first partition on `/dev/sdb` (which would be `/dev/sdb1`) with the ext4 file system, you would use:

    ```bash
    sudo mkfs.ext4 /dev/sdb1
    ```

    You'll likely see output indicating the progress of the formatting process.

- To format a partition as a swap area (let's say it's `/dev/sdb2`), you would use the mkswap command:

    ```bash
    sudo mkswap /dev/sdb2
    ```

    After formatting a swap partition, you usually need to enable it using the swapon command:

    ```bash
    sudo swapon /dev/sdb2
    ```

    To disable it, you'd use swapoff:

    ```bash
    sudo swapoff /dev/sdb2
    ```

**Important Note**: Formatting a partition will erase any existing data on it. **Double-check that you are formatting the correct partition!**

Once a partition is formatted with a file system, it's ready to have files and directories stored on it. The next step is to make it accessible within your system's file hierarchy, which is done through a process called **mounting**.

## Mounting Partitions

Now that we have our organized drawers (partitions) with their indexing systems (file systems), we need to actually be able to put files in and take them out. This is where **mounting** comes in.

In Linux, the entire file system is structured as a single, hierarchical tree, starting from the root directory (`/`). To access the data on a partition, you need to mount it at a specific location (a directory) within this tree. This location is called a **mount point**.

Think of it like this: you have your filing cabinet in one place, but to use the documents inside, you need to "mount" a drawer onto your desk. The desk is like a directory in your Linux file system, and the act of placing the drawer on the desk is like mounting the partition to that directory. Once mounted, the contents of the partition appear as if they are directly within that directory.

### Temporary Mounting

**Temporary Mounting**:

The `mount` command is used to temporarily attach a file system to a mount point. The basic syntax is:

```bash
sudo mount <device_partition> <mount_point>
```

For example, if you formatted `/dev/sdb1` with `ext4` and you want to access it, you might first create a directory to serve as the mount point:

```bash
sudo mkdir /mnt/mydata
```

Then, you would mount the partition to this directory:

```bash
sudo mount /dev/sdb1 /mnt/mydata
```

After this command, you can navigate to the `/mnt/mydata` directory, and you will see the contents (if any) of the `/dev/sdb1` partition.

To detach or **unmount** the partition, you use the `umount` command:

```bash
sudo umount /mnt/mydata
```

**Important Notes about Temporary Mounting**:

- The mount is only active until the system is rebooted. After a reboot, you'll need to mount the partition again if you want to access it.
- The mount point directory must exist before you can mount a partition to it.

### Permanent Mounting

Temporary mounting is useful for accessing a partition in the current session, but what if you want a partition to be automatically mounted every time your server starts up? That's where permanently mounting comes in, and it involves editing a crucial configuration file called `/etc/fstab`.

The `/etc/fstab` (file systems table) file contains a list of all the partitions and storage devices that should be automatically mounted at boot time. Each line in this file describes a single mount point and its associated device, file system type, and mount options.

Here's the typical format of an entry in `/etc/fstab`:

```shell
<device> <mount_point> <filesystem_type> <options> <dump> <pass>
```

Let's break down each of these fields:

1. `<device>`: This specifies the partition or storage device you want to mount. You can identify it either by its device name (e.g., `/dev/sdb1`) or by its **UUID (Universally Unique Identifier)**. Using UUIDs is generally recommended because device names can sometimes change depending on the order in which disks are detected during boot. You can find the UUID of your partitions using the `sudo blkid` command.

2. `<mount_point>`: This is the directory where you want to mount the partition within your system's file hierarchy (e.g., `/mnt/mydata`, `/home`, `/var`). This directory must exist.

3. `<filesystem_type>`: This specifies the type of file system on the partition (e.g., `ext4`, `swap`, `xfs`).

4. `<options>`: This field contains mount options that control how the file system is mounted. Common options include:
   - `defaults`: A set of commonly used options (rw, suid, dev, exec, nouser, async).
   - `rw`: Mounts the file system in read-write mode.
   - `ro`: Mounts the file system in read-only mode.
   - `noexec`: Prevents the execution of binaries on the file system.
   - `nosuid`: Disables the set-user-ID and set-group-ID bits.
   - `nodev`: Disables interpretation of character or block special devices.
   - `noatime`: Prevents updating the access time for files, which can improve performance.

5. `<dump>`: This field is used by the `dump` utility for backups. Usually, it's set to 0 to disable dumping.

6. `<pass>`: This field is used by `fsck` (file system check) to determine the order in which file systems are checked at boot time. The root file system (`/`) should be `1`, and other file systems are usually `2` or `0` to disable checking. Swap partitions are typically `0`.

**Example** `/etc/fstab` **entry**:

Let's say you have a partition with UUID `a1b2c3d4-e5f6-7890-1234-567890abcdef` that you want to permanently mount at `/data` with `ext4` file system and default options. Your `/etc/fstab` entry would look something like this:

```shell
UUID=a1b2c3d4-e5f6-7890-1234-567890abcdef /data ext4 defaults 0 2
```

**Editing** `/etc/fstab`:

You'll need to use a text editor with `sudo` privileges to edit the `/etc/fstab` file (e.g., `sudo nano /etc/fstab`). **Be very careful when editing this file, as incorrect entries can prevent your system from booting correctly**. It's always a good idea to make a backup of the /etc/fstab file before making changes.

After editing `/etc/fstab`, you can test your entries without rebooting using the command:

```bash
sudo mount -a
```

This command attempts to mount all file systems listed in `/etc/fstab`. If there are errors in your entries, you'll see error messages.
