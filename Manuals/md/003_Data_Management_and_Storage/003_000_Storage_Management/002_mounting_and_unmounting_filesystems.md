# Mounting and Unmounting Filesystems

## Contents

- [Mounting and Unmounting Filesystems](#mounting-and-unmounting-filesystems)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Filesystems and Mount Points](#understanding-filesystems-and-mount-points)
  - [Manual Mounting of Filesystems](#manual-mounting-of-filesystems)
    - [Mount Options](#mount-options)
  - [Unmounting Filesystems](#unmounting-filesystems)
  - [Working with `/etc/fstab`](#working-with-etcfstab)
    - [Common `fstab` Entries](#common-fstab-entries)
      - [Mounting the Root Filesystem](#mounting-the-root-filesystem)
      - [Mounting a Separate `/home` Partition](#mounting-a-separate-home-partition)
      - [Mounting a Swap Partition](#mounting-a-swap-partition)
      - [Mounting a USB Drive or External Hard Drive (Permantently)](#mounting-a-usb-drive-or-external-hard-drive-permantently)
      - [Mounting a Network Filesystem (NFS)](#mounting-a-network-filesystem-nfs)
    - [Mount All Filesystems in `/etc/fstab`](#mount-all-filesystems-in-etcfstab)

## Introduction

### Understanding Filesystems and Mount Points

Think of a **filesystem** like the organizational system inside a filing cabinet. Different types of filing systems exist (like alphabetical, by date, by subject), and in the Linux world, we have different types of filesystems like ext4 (common for Linux), XFS (often used for large files), and others. The filesystem's job is to manage how data is stored, accessed, and organized on a storage device (like a hard drive or a USB stick). It keeps track of where each file starts and ends, its permissions, and other important details.

Now, where do we actually access these filing cabinets (filesystems) on our server? That's where **mount points** come in. A mount point is simply a directory within your existing Linux directory structure where a filesystem is attached, making its contents accessible.

Imagine you have that filing cabinet full of important documents (your filesystem on a storage device). To actually use those documents, you need to place the entire cabinet (or perhaps just a drawer from it) in a specific location in your office so you can open it and get to the files. That location in your office is like the mount point on your server.

For example, you might have a separate hard drive containing user data. You could "mount" this filesystem to a directory named `/home/users`. Once mounted, all the files and folders on that separate hard drive will appear as if they are directly inside the `/home/users` directory.

We'll touch briefly on a special file called `/etc/fstab` later, which is like a configuration file that tells your server which filesystems to automatically "plug in" (mount) to which locations every time it starts up. But for now, we'll focus on how to do this manually.

---

Now, let's briefly touch upon the `/etc/fstab` file. You can think of this file as a configuration roadmap for your server's filesystems. It lives in the `/etc` directory and contains a list of filesystems that the system should automatically mount during the boot process.

Each line in this file typically describes a filesystem, where it should be mounted, the type of filesystem, and some options on how it should be mounted. We'll delve deeper into the specifics of `/etc/fstab` in the last step of our learning plan.

For now, just remember that `/etc/fstab` is there to automate the mounting of filesystems so you don't have to manually mount them every time you start your server. However, understanding how to manually mount and unmount filesystems is crucial for troubleshooting, mounting temporary storage, or working with devices that aren't permanently listed in `/etc/fstab`.

## Manual Mounting of Filesystems

The primary command for manually mounting filesystems in Linux is, unsurprisingly, `mount`. The basic structure of this command looks like this:

```bash
sudo mount device mount_point
```

Let's break down these two crucial parts:

- `device`: This refers to the storage device or the filesystem you want to mount. Linux identifies these devices using special file paths, usually located in the `/dev` directory. For example:
  - A primary hard drive might have partitions like `/dev/sda1` (first partition on the first SATA drive), `/dev/sdb2` (second partition on the second SATA drive), and so on.
  - A USB drive or external hard drive might be recognized as `/dev/sdb1`, `/dev/sdc1`, etc., depending on what's already connected to the system.
  - Even network filesystems (like NFS shares) have their own way of being identified here.

- `mount_point`: This is the directory on your existing Linux system where you want to attach the filesystem. This directory needs to exist before you can mount something to it. Common places to create mount points for additional storage are within the `/mnt` directory (a general-purpose mount point location) or you might create specific directories like `/data`, `/backup`, etc., depending on the purpose of the mounted storage.

So, if you had a USB drive that Linux recognized as `/dev/sdb1` and you wanted to access its contents, you would first create a directory to serve as the mount point (let's say `/mnt/usb`). Then, you would use the mount command like this:

```bash
sudo mount /dev/sdb1 /mnt/usb
```

After running this command (and assuming everything goes well), you would be able to navigate into the `/mnt/usb` directory and see all the files and folders that are on your USB drive.

Remember that you typically need `sudo` to run the `mount` command because it involves making changes to the system's filesystem structure, which requires administrative privileges.

---

Let's walk through a practical example of manually mounting a filesystem.

Imagine you've just connected a new external hard drive to your Linux server. The system has recognized it as `/dev/sdb1`. Now, you want to be able to access the files on this drive.

**Step 1: Create a Mount Point Directory**

First, you need a directory where you'll "attach" the filesystem. A common place for this is within the `/mnt` directory. Let's create a directory named `external_drive` inside `/mnt`:

```bash
sudo mkdir /mnt/external_drive
```

This command uses `sudo` for administrative privileges and `mkdir` to create a new directory. Now, `/mnt/external_drive` exists on your system.

**Step 2: Mount the Filesystem**

Next, you'll use the `mount` command to connect the filesystem on `/dev/sdb1` to the mount point you just created:

```bash
sudo mount /dev/sdb1 /mnt/external_drive
```

After running this command (if the device and filesystem are healthy), the contents of the filesystem on `/dev/sdb1` will now be accessible within the `/mnt/external_drive` directory. You can navigate into this directory using the `cd` command and explore the files.

To see what filesystems are currently mounted on your system, you can use the command:

```bash
mount | less
```

This will display a list of all mounted filesystems and their corresponding mount points. You should see an entry for `/dev/sdb1` mounted on `/mnt/external_drive`. You can press `q` to exit the less pager.

Now, let's say you wanted to mount the same `/dev/sdb1` filesystem with read-only permissions. This can be useful if you only need to view the data and want to prevent any accidental modifications. You can achieve this by using the `-o` option with the `mount` command, followed by the mount option `ro` (for read-only):

```bash
sudo mount -o ro /dev/sdb1 /mnt/external_drive
```

If the filesystem was already mounted, you might need to unmount it first (which we'll cover in the next step) before remounting it with different options.

### Mount Options

Now that you know how to mount a filesystem, let's talk about **mount options**. These are like extra instructions you can give to the `mount` command to control how the filesystem is accessed and behaves. You specify these using the `-o` option followed by a comma-separated list of options.

Here are a few common and important mount options you'll likely encounter as a system administrator:

- `ro` **(read-only)**: As we saw earlier, this mounts the filesystem so that you can only read files and directories; you cannot make any changes. This is useful for accessing data that shouldn't be modified, like mounting an installation CD or a backup.

- `rw` **(read-write)**: This is the default option and allows you to both read and write to the filesystem.

- `exec` **(execute)**: This allows you to execute binary files that reside on the mounted filesystem.

- `noexec` **(no execute)**: This prevents the execution of any binary files on the mounted filesystem. This can be a security measure to prevent running potentially harmful executables from untrusted sources, like a USB drive.

- `suid` **(set user ID)**/`guid` **(set group ID)**: These options allow special permissions associated with a file's owner or group to take effect when the file is executed.

- `nosuid` **(no set user ID)/**`noguid` **(no set group ID)**: These options disable the `suid` and `guid` permissions on the mounted filesystem. This is another security measure to prevent privilege escalation.

- `dev` **(allow device files)**: This allows the interpretation of character or block special devices on the filesystem.

- `nodev` **(no device files)**: This prevents the interpretation of device files on the filesystem. This is often used for network filesystems for security reasons.

For example, if you wanted to mount `/dev/sdb1` to `/mnt/readonly_data` in read-only mode and disallow the execution of any files on it, you would use the following command:

```bash
sudo mount -o ro,noexec /dev/sdb1 /mnt/readonly_data
```

Understanding these options is crucial for controlling access and security on your server. You'll often need to choose the appropriate options based on how the mounted storage will be used.

## Unmounting Filesystems

Now that we've covered how to mount filesystems and control their behavior with options, it's equally important to understand how to safely **unmount** them.

The command for unmounting a filesystem is `umount`. It's quite straightforward and has two common ways to use it:

1. **Unmounting by mount point**: You can specify the directory where the filesystem is mounted. For example, to unmount the filesystem we mounted at `/mnt/external_drive`, you would use:

    ```bash
    sudo umount /mnt/external_drive
    ```

2. **Unmounting by device**: You can also specify the device file of the filesystem you want to unmount. For example, to unmount the filesystem that was on `/dev/sdb1`, you could use:

    ```bash
    sudo umount /dev/sdb1
    ```

Both methods achieve the same result. However, it's generally safer and more common to unmount by the mount point, as it's often easier to remember where you mounted something rather than the specific device name, especially if you have multiple storage devices connected.

**Why is unmounting important?**

When a filesystem is mounted, the operating system keeps track of which files are open and which processes are using the files on that filesystem. If you were to simply unplug a USB drive or detach a hard drive without properly unmounting it first, you could potentially lead to:

- **Data corruption**: Any data that was being written to the device might not be fully saved, leading to incomplete or corrupted files.
- **Lost data**: In some cases, abruptly removing a mounted device can even result in the loss of entire files or directories.
- **System instability**: Although less common with modern systems, improper unmounting could sometimes lead to system errors or instability.

Therefore, it's crucial to always unmount a filesystem before physically disconnecting the storage device.

---

Now, let's discuss what can happen if you try to unmount a filesystem while it's being used. Think of it like trying to pull out a drawer from our filing cabinet while someone is actively looking through files inside!

If files are open or a process is currently working within a mounted filesystem, the `umount` command will likely fail and you'll see an error message, often saying something like **"device is busy"**. This is the operating system's way of protecting the integrity of the data and preventing those active processes from crashing or corrupting data.

**How to identify if a filesystem is busy**:

When you encounter a "device is busy" error, you need to figure out which process or user is currently using the mounted filesystem. There are a couple of handy command-line tools for this:

1. `lsof` **(List Open Files)**: This powerful command can show you all the open files and the processes that have them open. To find out which processes are using a specific mount point (e.g., `/mnt/external_drive`), you can use:

    ```bash
    sudo lsof /mnt/external_drive
    ```

    This will list the process ID (PID), user, and the type of access for each file or directory within /mnt/external_drive that is currently open.

2. `fuser` **(Identify Processes Using Files or Sockets)**: The `fuser` command is specifically designed to identify processes that are using specified files or filesystems. To find out which processes are using our example mount point:

    ```bash
    sudo fuser -m /mnt/external_drive
    ```

    The `-m` option tells `fuser` to look for processes using the mount point. The output will typically show the PIDs of the processes that are using the filesystem. You might also see letters after the PIDs indicating the type of access (e.g., `c` for current directory, `r` for root directory, `f` for open file).

**What to do if a filesystem is busy**:

Once you've identified the processes that are using the filesystem, you have a couple of options:

1. **Terminate the processes**: If you know what the processes are and it's safe to do so, you can terminate them using the `kill` command followed by the process ID (PID) you found with `lsof` or `fuser`. For example, if `fuser` showed a PID of `1234` is using the filesystem, you could try:

    ```bash
    sudo kill 1234
    ```

    Sometimes, a process might not respond to a regular `kill` signal, in which case you might need to use `kill -9` (force kill), but be very cautious with this as it can lead to data loss if the process was in the middle of writing something.

2. **Change the current directory**: If your own terminal or another user's terminal is currently inside the mounted directory, simply navigating out of it using `cd ..` can often resolve the "device is busy" error.

After ensuring that no processes are actively using the filesystem, you should be able to unmount it successfully using the `umount` command.

## Working with `/etc/fstab`

The `/etc/fstab` file (short for "file system table") is a crucial configuration file that the system reads during startup to determine which filesystems should be automatically mounted and where. Each line in this file describes a single filesystem and how it should be handled.

A typical entry in `/etc/fstab` has the following six fields, separated by spaces or tabs:

1. `<file system>`: This specifies the device to be mounted. It can be the device file (like `/dev/sdb1`), the UUID (Universally Unique Identifier) of the filesystem (which is more robust as device names can sometimes change), or a network filesystem specification.

2. `<mount point>`: This is the directory where the filesystem should be mounted (e.g., `/mnt/data`, `/home/users`). This directory must exist.

3. `<type>`: This specifies the type of the filesystem (e.g., `ext4`, `xfs`, `nfs`, `vfat`). The system needs to know the filesystem type to handle it correctly.

4. `<options>`: This is a comma-separated list of mount options, similar to what we used with the `mount -o` command (e.g., `ro`, `rw`, `noexec`, `defaults`). The defaults option typically implies `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, and `async`.

5. `<dump>`: This field is used by the `dump` utility for backups. Usually, it's set to `0`, which means the filesystem is not dumped. Setting it to `1` would enable dumping.

6. `<pass>`: This field is used by the `fsck` (file system consistency check) utility to determine the order in which filesystem checks are performed at boot time. The root filesystem (`/`) should have a value of `1`. Other filesystems typically have a value of `2` (checked after root) or `0` (not checked).

Here's an example of a line in `/etc/fstab`:

```shell
UUID=a1b2c3d4-e5f6-7890-1234-567890abcdef  /data  ext4  defaults  0  2
```

**In this example**:

- `UUID=a1b2c3d4-e5f6-7890-1234-567890abcdef` is the unique identifier of the device.
- `/data` is the mount point.
- `ext4` is the filesystem type.
- `defaults` are the mount options.
- `0` means it's not dumped.
- `2` means it will be checked after the root filesystem.

### Common `fstab` Entries

Let's look at some common entries you might find in an `/etc/fstab` file on your Linux server. Understanding these examples will help you when you need to add or modify entries yourself.

Here are a few typical scenarios:

#### Mounting the Root Filesystem

- **Mounting the Root Filesystem**:

    You'll always see an entry for the root filesystem (/). It's essential for the system to boot and function. It might look something like this:

    ```shell
    UUID=your-root-partition-uuid  /  ext4  errors=remount-ro  0  1
    ```

  - `UUID=your-root-partition-uuid`: This identifies the root partition using its unique UUID. We'll learn how to find this shortly.
  - `/`: This is the mount point – the root directory.
  - `ext4`: This is the filesystem type, in this case, the common ext4.
  - `errors=remount-ro`: This is a mount option that tells the system to remount the filesystem in read-only mode if errors are detected.
  - `0`: The filesystem is not dumped.
  - `1`: This indicates that the root filesystem should be checked for errors first during boot.

#### Mounting a Separate `/home` Partition

- **Mounting a Separate** `/home` **Partition**:

    If you have a dedicated partition for user home directories, you might see an entry like this:

    ```shell
    UUID=your-home-partition-uuid  /home  ext4  defaults  0  2
    ```

  - `UUID=your-home-partition-uuid`: The UUID of the partition containing `/home`.
  - `/home`: The mount point for the home directories.
  - `ext4`: The filesystem type.
  - `defaults`: Using the default mount options (rw, suid, dev, exec, auto, nouser, async).
  - `0`: Not dumped.
  - `2`: Checked after the root filesystem.

#### Mounting a Swap Partition

- **Mounting a Swap Partition**:

    Swap space is used as virtual RAM. Its entry in /etc/fstab will look different as it's not mounted to a regular directory:

    ```shell
    UUID=your-swap-partition-uuid  none  swap  sw  0  0
    ```

  - `UUID=your-swap-partition-uuid`: The UUID of the swap partition.
  - `none`: Swap partitions don't have a traditional mount point.
  - `swap`: Specifies the filesystem type as swap.
  - `sw`: The options for swap (usually just `sw`).
  - `0`: Not dumped.
  - `0`: Not checked by fsck.

#### Mounting a USB Drive or External Hard Drive (Permantently)

- **Mounting a USB Drive or External Hard Drive (Permanently)**:

    If you want a USB drive or external hard drive to be automatically mounted to a specific location every time it's connected (and if its entry is in `/etc/fstab`), it might look like this:

    ```shell
    UUID=your-external-drive-uuid  /mnt/backup  ext4  defaults,user  0  0
    ```

  - `UUID=your-external-drive-uuid`: The UUID of the external drive's partition.
  - `/mnt/backup : The mount point.
  - `ext4`: The filesystem type.
  - `defaults,user`: Includes default options and allows a regular user to mount and unmount this filesystem (which can be useful for removable media).
  - `0`: Not dumped.
  - `0`: Not checked.

#### Mounting a Network Filesystem (NFS)

- **Mounting a Network Filesystem (NFS)**:

    Mounting an NFS share from another server would look something like this:

    ```shell
    server_ip:/path/to/share  /mnt/network_share  nfs  defaults  0  0
    ```

  - `server_ip:/path/to/share`: Specifies the server's IP address or hostname and the exported path.
  - `/mnt/network_share`: The local mount point.
  - `nfs`: The filesystem type is NFS.
    defaults: Mount options specific to NFS.
  - `0`: Not dumped.
  - `0`: Not checked.

Notice the use of UUIDs in many of these examples. Using UUIDs is generally preferred over device names like 1/dev/sdb1` because device names can sometimes change depending on the order in which devices are detected during boot. UUIDs are unique and persistent identifiers for each filesystem.

### Mount All Filesystems in `/etc/fstab`

The command to do this is quite simple:

```bash
sudo mount -a
```

The `-a` option tells the `mount` command to try and mount all filesystems that are listed in `/etc/fstab` (except those marked as `noauto`).

**When is** `mount -a` **useful?**

- **During system boot**: The system automatically runs `mount -a` (or a similar process) as part of its startup sequence to mount all the filesystems defined in `/etc/fstab`. This is how your regular partitions like `/`, `/home`, and `swap` get mounted without you having to do anything manually.
- **After editing** `/etc/fstab`: If you make changes to the `/etc/fstab` file (e.g., add a new entry for an external drive you want to mount automatically), you can run `sudo mount -a` to apply these changes without needing to reboot your entire server. This is a convenient way to test if your new entry is correctly configured. If there are any errors in your `/etc/fstab` file, the `mount -a` command will usually report them, helping you to identify and fix any issues.

**Important Considerations**:

- **Permissions**: Just like with manual mounting, you'll typically need `sudo` to run `mount -a` as it involves system-level changes.
- **Existing Mounts**: `mount -a` will try to mount all filesystems in `/etc/fstab` that are not already mounted. If a filesystem is already mounted and its entry is in `/etc/fstab`, running `mount -a` again won't cause any problems; it will simply skip that entry.
- **Errors**: If there's an error in an `/etc/fstab` entry (e.g., incorrect device name, wrong filesystem type, invalid options), `mount -a` will likely fail to mount that specific filesystem and will display an error message. It might also continue to try and mount the other valid entries.

So, if you've just added a line to `/etc/fstab` for a new backup drive mounted at `/mnt/backup`, after saving the file, you could run `sudo mount -a`. If the entry is correct and the device is available, you should then see your backup drive mounted at `/mnt/backup`. You can verify this using the `mount | less` command we used earlier.
