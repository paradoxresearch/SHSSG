# Managing Logical Volumes with LVM

## Contents

- [Managing Logical Volumes with LVM](#managing-logical-volumes-with-lvm)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Creating Physical Volumes (PVs)](#creating-physical-volumes-pvs)
    - [Display Physical Volumes (PVs) Information](#display-physical-volumes-pvs-information)
  - [Creating Volume Groups (VGs)](#creating-volume-groups-vgs)
    - [Display Volume Groups (VGs) Information](#display-volume-groups-vgs-information)
  - [Creating Logical Volumes (LVs)](#creating-logical-volumes-lvs)
    - [Display Logical Volumes (LVs) Information](#display-logical-volumes-lvs-information)
  - [Managing Logical Volumes](#managing-logical-volumes)
    - [Resizing Logical Volumes](#resizing-logical-volumes)
      - [Extend a filesystem](#extend-a-filesystem)
      - [Reduce a filesystem](#reduce-a-filesystem)
    - [Creating Snapshots](#creating-snapshots)
    - [Activate, Deactivate, and Remove Logical Volumes](#activate-deactivate-and-remove-logical-volumes)
      - [Deactivate Logical Volumes](#deactivate-logical-volumes)
      - [Activate Logical Volumes](#activate-logical-volumes)
      - [Remove Logical Volumes](#remove-logical-volumes)
  - [Using Logical Volumes](#using-logical-volumes)

## Introduction

LVM allows you to manage your storage in a much more flexible way than traditional partitioning. You can easily resize, move, and take snapshots of your storage without needing to take the system offline.

Think of your physical hard drives as individual building blocks, like LEGO bricks. These are your **Physical Volumes (PVs)**. On their own, they might not be the exact size or configuration you need for your server's storage.

Now, imagine you have a big baseplate where you can combine these LEGO bricks into larger, more useful structures. This baseplate is like a **Volume Group (VG)**. A VG pools the storage capacity of one or more PVs, creating a flexible storage space.

Finally, from this large pool of storage (the VG), you can carve out specific storage areas of the size and configuration you need for different purposes, like file systems or databases. These carved-out sections are your **Logical Volumes (LVs)**. They behave just like regular partitions but with much more flexibility.

So, to recap with our analogy:

- **Physical Volumes (PVs)** are like individual LEGO bricks.
- **Volume Groups (VGs)** are like the baseplates where you combine the bricks.
- **Logical Volumes (LVs)** are like the specific structures you build on the baseplate.

---

LVM offers several key advantages over traditional partitioning:

- **Flexible Resizing**: Imagine you've allocated 50GB to a partition, and suddenly you need more space. With traditional partitioning, this can be a real headache, often requiring downtime, data migration, and potentially reformatting. LVM allows you to easily extend or even shrink logical volumes (within the limits of the VG's free space) while the system is running! It's like being able to add or remove LEGO bricks from your structure without taking the whole thing apart.

- **Snapshots**: LVM enables you to create "snapshots" of your logical volumes. Think of it as taking a digital photograph of your data at a specific point in time. These snapshots are very useful for backups or testing changes, as you can quickly revert to the previous state if something goes wrong. The cool thing is, snapshots initially take up very little space, only storing the changes made after the snapshot was taken.

- **Storage Pooling**: With LVM, you can combine multiple physical disks into a single volume group. This creates a large pool of storage that can be flexibly allocated to different logical volumes. So, if you add a new hard drive to your server, you can easily add it to an existing volume group and extend your logical volumes without having to worry about which physical disk has the free space. It's like having one big bin of LEGOs instead of several smaller, separate boxes.

Consider this scenario: You have a database server running on a traditional partition that's almost full. To increase its size, you'd likely have to:

- Take the database offline.
- Unmount the partition.
- Resize the underlying physical partition (which might not even have contiguous free space).
- Resize the filesystem.
- Mount the partition.
- Bring the database back online.

This whole process can be time-consuming and risky. With LVM, you could potentially extend the logical volume and resize the filesystem online, minimizing downtime and complexity.

## Creating Physical Volumes (PVs)

Our first step in using LVM is to identify the physical disks or partitions that we want to use as part of our LVM setup. These will become our **Physical Volumes (PVs)**.

Imagine you have a brand new hard drive connected to your Linux server, say it's recognized as `/dev/sdb`. To tell LVM that this drive should be used as a PV, we use the command `pvcreate`.

Here's how you would initialize `/dev/sdb` as a physical volume:

```bash
sudo pvcreate /dev/sdb
```

You might see an output like:

```shell
Physical volume "/dev/sdb" successfully created.
```

This command essentially writes some metadata onto the disk, identifying it as a physical volume that LVM can now understand and manage. You can repeat this command for any other block devices (like `/dev/sdc`, `/dev/sdd`, or even partitions like `/dev/sda1`) that you want to include in your LVM configuration.

Think of it like putting a special LVM sticker on each of your LEGO bricks so that the LVM baseplate (our next step, the Volume Group) knows they can be used.

### Display Physical Volumes (PVs) Information

Now that you know how to create Physical Volumes, it's important to be able to see the information about them. For this, we have a couple of handy command-line tools: `pvdisplay` and `pvs`.

`pvdisplay`: This command gives you a detailed view of each physical volume, including its name, size, the volume group it belongs to (if any), and other technical details. If you run `sudo pvdisplay /dev/sdb`, you'll see a lot of information about the `/dev/sdb` PV we just created. It's like looking at the detailed specifications sheet for that particular LVM LEGO brick.

`pvs`: This command provides a more concise, table-like output of all physical volumes on your system. It shows key information like the PV name, its size, how much of it is being used, and the name of the volume group it belongs to. It's like having a quick inventory list of all your LVM LEGO bricks and which projects (Volume Groups) they are currently part of.

Here's an example of what the output of `pvs` might look like:

```shell
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda2  vg00      lvm2 a--    99.51g    0
  /dev/sdb             lvm2 ---  <100.00g 100.00g
```

In this example, `/dev/sda2` is a PV that's part of a Volume Group named `vg00` and is fully utilized. `/dev/sdb` is a PV that we likely just created (since it has no VG listed and shows 100% free space). The `<` symbol before the size indicates that the size might not be exactly 100GB but close to it.

## Creating Volume Groups (VGs)

A **Volume Group** is like that big baseplate where you can stick multiple LEGO bricks together to create a larger, unified workspace. It pools the storage capacity of all the PVs it contains into one big virtual disk. This allows you to create Logical Volumes (our next step) of varying sizes across potentially multiple physical disks.

To create a Volume Group, we use the command `vgcreate`. You need to give your Volume Group a name (something descriptive like `vg00`, `data_vg`, or `storage_pool`). Then, you specify the physical volumes you want to include in it.

For example, if you have initialized `/dev/sdb` and `/dev/sdc` as physical volumes, you can create a volume group named my_vg using this command:

```bash
sudo vgcreate my_vg /dev/sdb /dev/sdc
```

You might see output similar to this:

```shell
Volume group "my_vg" successfully created
```

This command takes the storage capacity of `/dev/sdb` and `/dev/sdc` and combines it into a single, manageable pool named `my_vg`. Now, `my_vg` has a total size equal to the sum of the usable space on `/dev/sdb` and `/dev/sdc`.

Think of it as taking your individual 100GB LEGO bricks (`/dev/sdb` and `/dev/sdc`) and placing them on the `my_vg` baseplate, giving you a total of approximately `200GB` of space to work with.

### Display Volume Groups (VGs) Information

Just like with Physical Volumes, we have commands to get the information for Volume Groups: `vgdisplay` and `vgs`.

`vgdisplay`: This command provides detailed information about a specific Volume Group. If you run `sudo vgdisplay user_data`, you'll see a comprehensive overview, including the VG's name, its status, the total size, how much free space it has, the number of Physical Volumes it contains, and much more. It's like getting a detailed blueprint of your LVM baseplate.

`vgs`: This command gives you a more summarized, table-like view of all Volume Groups on your system. It shows key details like the VG name, the number of PVs included, the total size, how much space is free, and its overall status. It's like having a quick inventory list of all your LVM baseplates and their current capacity.

Here's an example of what the output of `vgs` might look like:

```shell
  VG        #PV #LV #SN Attr   VSize   VFree
  my_vg     2   0   0   wz--n- <199.99g <199.99g
  user_data 1   0   0   wz--n- <99.99g  <99.99g
  vg00      1   2   1   wz--n- 99.51g      0
```

In this example, you can see the `my_vg` we created earlier, the `user_data` VG, and another VG named `vg00`. The `#PV` column shows the number of Physical Volumes in each VG, `#LV` shows the number of Logical Volumes, `VSize` is the total size, and `VFree` is the available free space.

To see the detailed information for user_data, you would use `sudo vgdisplay user_data`.

## Creating Logical Volumes (LVs)

Think of Logical Volumes as the specific structures you build on your LVM baseplate. They are the equivalent of traditional partitions but are created from the free space within a Volume Group. When you create an LV, you specify its size and give it a name. This name, combined with the Volume Group name, will form the path you use to access the LV (e.g., `/dev/my_vg/my_logical_volume`).

To create a Logical Volume, we use the command `lvcreate`. Here are a couple of important options you'll commonly use:

- `-L`: This option specifies the size of the logical volume. You can use units like G for gigabytes, M for megabytes, etc. For example, `-L 50G` would create a 50-gigabyte logical volume.
- `-n`: This option specifies the name you want to give to your logical volume.

So, if you have a Volume Group named `user_data` and you want to create a 20GB logical volume named `home_partition` within it, you would use the following command:

```bash
sudo lvcreate -L 20G -n home_partition user_data
```

You might see output like this:

```shell
  Logical volume "home_partition" created.
```

This command carves out 20GB of space from the `user_data` Volume Group and names it `home_partition`. You can create multiple logical volumes within the same Volume Group, each with its own size and name, as long as there is enough free space in the VG. It's like building different sized rooms (LVs) within the space provided by your baseplate (VG).

### Display Logical Volumes (LVs) Information

Just like we had `pvdisplay`/`pvs` for Physical Volumes and `vgdisplay`/`vgs` for Volume Groups, we have `lvdisplay` and `lvs` to get information about our **Logical Volumes (LVs)**.

`lvdisplay`: This command provides detailed information about a specific Logical Volume. If you run `sudo lvdisplay /dev/user_data/home_partition` (assuming you created an LV with that name in the `user_data` VG), you'll see a wealth of details. This includes the LV's name, its size, the Volume Group it belongs to, its current state, and even information about snapshots if they exist. It's like getting a detailed spec sheet for a particular storage room you built on your LVM baseplate.

`lvs`: This command gives you a more concise, table-like output of all Logical Volumes on your system. It shows key information such as the LV name, the Volume Group it belongs to, its size, how much space is allocated, and its current status. It's like having a quick inventory list of all the storage rooms you've built and their current size and occupancy.

Here's an example of what the output of `lvs` might look like:

```shell
  LV           VG        Attr       LSize   Pool Origin Data%  Meta%  Move Cpy%Sync Inactive PV
  home_partition user_data -wi-a-----  20.00g
  database       data_vg   -wi-a-----  30.00g
  root         vg00      -wi-ao----  <80.00g
  swap_1       vg00      -wi-ao----   1.51g
  snap_root    vg00      swi-cow---   4.00g root   0.00
```

In this example, you can see the `home_partition` LV we created in the `user_data` VG, the `database` LV in `data_vg`, and a couple of other LVs (`root` and `swap_1`) that are part of the `vg00` VG. You can also see an example of a snapshot named `snap_root` associated with the `root` LV. The `LSize` column shows the size of each logical volume.

To see more detailed information about the `home_partition` LV, you would use `sudo lvdisplay /dev/user_data/home_partition`.

## Managing Logical Volumes

### Resizing Logical Volumes

One of the most significant advantages of LVM is the ability to dynamically resize your Logical Volumes. We use two primary commands for this: `lvextend` to increase the size and `lvreduce` to decrease it.

#### Extend a filesystem

To **increase** the size of a Logical Volume, you use the `lvextend` command. You can specify the amount you want to add (using a `+` sign) or the new total size. For example, to add 10GB to a logical volume named `home_partition` in the `user_data` Volume Group, you'd use:

```bash
sudo lvextend -L +10G /dev/user_data/home_partition
```

After extending the LV, you'll usually need to resize the filesystem on it to utilize the new space. You can often do this online using commands like resize2fs (for ext4) or xfs_growfs (for XFS). The -r option with lvextend can sometimes automate this process.

**For ext4, you would typically use**:

```bash
sudo resize2fs /dev/user_data/home_partition
```

**For XFS, you would use**:

```bash
sudo xfs_growfs /dev/user_data/home_partition
```

#### Reduce a filesystem

To **decrease** the size of a Logical Volume, you use the `lvreduce` command. **However, this is a potentially dangerous operation that can lead to data loss if not done correctly**. You should always back up your data before attempting to shrink an LV. Similar to `lvextend`, you can specify the amount to remove (with a `-` sign) or the new total size. **Crucially, before reducing the LV size, you MUST first shrink the filesystem on it to a smaller size using filesystem-specific tools like** `resize2fs`.

**Before reducing the LV, you MUST first shrink the filesystem on it to a size smaller than the target LV size**. For an ext4 filesystem, you would do something like:

```bash
sudo resize2fs /dev/user_data/home_partition 22G
```

To reduce the size of an LV, for example, to remove 5GB from our (now 30GB) `home_partition` LV, you would use:

```bash
sudo lvreduce -L -5G /dev/user_data/home_partition
```

It's generally safer and more common to extend Logical Volumes when you need more space.

### Creating Snapshots

Another powerful feature of Logical Volumes is the ability to create **snapshots**. Think of a snapshot as a "point-in-time" copy of your LV. It allows you to capture the state of your data at a specific moment, which is incredibly useful for backups or for testing changes without risking your live data.

The cool thing about LVM snapshots is that they use a technique called copy-on-write. This means that when you first create a snapshot, it doesn't actually copy all the data. Instead, it only records the original state of the blocks. As data on the original Logical Volume changes, only the original blocks are copied to the snapshot before being overwritten. This makes snapshot creation very fast and initially consumes very little disk space. However, as more changes occur on the original LV, the snapshot will grow in size as it stores these original blocks.

To create a snapshot, you use the `lvcreate` command with the `-s` (for snapshot) option and the `-L` option to specify the size you want to allocate for the snapshot. You also need to specify the name for your snapshot and the original Logical Volume you want to snapshot.

For example, to create a snapshot named `my_snapshot` of our `home_partition` LV, and allocate 5GB of space for it, you would use:

```bash
sudo lvcreate -s -L 5G -n my_snapshot /dev/user_data/home_partition
```

Here, `-s` indicates it's a snapshot, `-L 5G` sets the maximum size the snapshot can grow to, `-n my_snapshot` gives the snapshot a name, and `/dev/user_data/home_partition` is the original Logical Volume.

You can then mount this snapshot just like a regular Logical Volume to access the data as it was at the moment you took the snapshot. For example:

```bash
sudo mount /dev/user_data/my_snapshot /mnt/snapshot_mount
```

Remember that the size you allocate for the snapshot determines how much change the original LV can undergo before the snapshot runs out of space and becomes invalid. For critical systems, you'll need to monitor the snapshot size.

### Activate, Deactivate, and Remove Logical Volumes

Sometimes, you might need to temporarily make a Logical Volume inaccessible (deactivate it) or bring it back online (activate it). The `lvchange` command is used for this.

#### Deactivate Logical Volumes

To **deactivate** a Logical Volume, you use the `-a n` option followed by the LV's path:

```bash
sudo lvchange -a n /dev/user_data/home_partition
```

When an LV is deactivated, it's no longer accessible by the system (it will be unmounted if it was mounted). This might be necessary for certain maintenance tasks.

#### Activate Logical Volumes

To **activate** a Logical Volume, you use the `-a y` option followed by the LV's path:

```bash
sudo lvchange -a y /dev/user_data/home_partition
```

This makes the LV accessible again, and you can then mount it if needed.

#### Remove Logical Volumes

Finally, if you no longer need a Logical Volume, you can **remove** it using the lvremove command. **This is a destructive operation, and all data on the LV will be lost!** Make absolutely sure you want to do this before proceeding.

To remove a Logical Volume, use the command followed by the LV's path:

```bash
sudo lvremove /dev/user_data/home_partition
```

You will likely be prompted to confirm this action.

It's important to remember that you cannot remove a Logical Volume that is currently mounted or active. You'll need to unmount it and deactivate it first.

## Using Logical Volumes

Once you've created a Logical Volume, it's like having a new, blank hard drive partition. To actually store files on it, you need to put a **filesystem** on it. This organizes the space so your operating system knows how to read and write data.

The command we use to create a filesystem is `mkfs` (make filesystem), followed by the type of filesystem you want and the path to your Logical Volume. For example, if you want to use the common ext4 filesystem on a Logical Volume named `data_lv` within a Volume Group called `storage_vg` (so its path is `/dev/storage_vg/data_lv`), you would use the command:

```bash
sudo mkfs.ext4 /dev/storage_vg/data_lv
```

After running this command, the Logical Volume `/dev/storage_vg/data_lv` will be formatted with the ext4 filesystem, and you'll be able to mount it and start storing data.

To make the storage accessible, you need to **mount** the Logical Volume to a directory on your system. For example, if you want to access the data in `/dev/storage_vg/data_lv` through a directory named `/data`, you would use the `mount` command:

```bash
sudo mount /dev/storage_vg/data_lv /data
```

This command links the Logical Volume to the `/data` directory, and anything you read or write in `/data` will now be on your LVM-managed storage. However, this mount is temporary and will not persist after a reboot.

To make the mount permanent, so the Logical Volume is automatically mounted every time your server starts, you need to add an entry to the `/etc/fstab` file. This file tells the system how to handle different storage devices. You would add a line similar to this to `/etc/fstab`:

```shell
/dev/storage_vg/data_lv  /data   ext4    defaults    0   2
```

Let's break down this line:

- `/dev/storage_vg/data_lv`: This is the device path of your Logical Volume.
- `/data`: This is the mount point, the directory where you want to access the storage.
- `ext4`: This is the filesystem type.
- `defaults`: These are the mount options (common defaults usually work well).
- `0`: This specifies whether to dump the filesystem for backups (0 means no).
- `2`: This specifies the order for filesystem checks during boot (root filesystem is usually 1, others are 2 or 0 for no check).

**Be very careful when editing** `/etc/fstab` **as errors can prevent your system from booting correctly**. It's always a good idea to make a backup of this file before making changes.

Once you've added this line to `/etc/fstab`, you can test it by running:

```bash
sudo mount -a
```

This command will mount all filesystems listed in `/etc/fstab`. If there are no errors, your Logical Volume should now be permanently mounted at `/data`.
