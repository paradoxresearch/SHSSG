# Using KVM

## Contents

- [Using KVM](#using-kvm)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Checking If Your System Supports Virtualization](#checking-if-your-system-supports-virtualization)
  - [Installing KVM and its Tools](#installing-kvm-and-its-tools)
  - [Creating the Virtual Machine](#creating-the-virtual-machine)
    - [Using the Graphical virt-manager Interface to Create a Virtual Machine](#using-the-graphical-virt-manager-interface-to-create-a-virtual-machine)
      - [Configuring a Bridged Network with virt-manager](#configuring-a-bridged-network-with-virt-manager)
      - [Basic VM Management with virt-manager](#basic-vm-management-with-virt-manager)
      - [Connect to the Console of a Running VM with virt-manager](#connect-to-the-console-of-a-running-vm-with-virt-manager)
      - [Modifying an Existing NAT Network to Bridged with virt-manager](#modifying-an-existing-nat-network-to-bridged-with-virt-manager)
    - [Using virt-install in the Command Line to Create a Virtual Machine](#using-virt-install-in-the-command-line-to-create-a-virtual-machine)
      - [Basic VM Mangement in the Command-Line Interface](#basic-vm-mangement-in-the-command-line-interface)

## Introduction

**KVM** is a widely adopted and powerful virtualization technology on Linux. Many cloud providers and enterprise environments rely on KVM or technologies built upon it.

**KVM**, which stands for **Kernel-based Virtual Machine**, is a powerful virtualization technology built right into the Linux kernel. Think of your Linux kernel as the core of your operating system. KVM essentially allows this core to act as a **hypervisor**.

Now, there are two main types of hypervisors. **Type 1** hypervisors run directly on the system's hardware (like KVM or VMware ESXi), offering excellent performance because they have direct access to resources. **Type 2** hypervisors, like VirtualBox, run as an application on top of a traditional operating system.

Here's a simple way to visualize it:

```shell
+-----------------+      +-----------------+
|   Applications  |      |   Applications  |
+-----------------+      +-----------------+
| VirtualBox      |      |      VMs        |
+-----------------+      +-----------------+
| Host OS (Linux) |      | KVM (in Kernel) |
+-----------------+      +-----------------+
|     Hardware    | <--> |     Hardware    |
+-----------------+      +-----------------+
      Type 2                    Type 1
```

**Benefits of KVM**:

- **Performance**: Because it's integrated into the kernel, KVM offers near-native performance for your virtual machines.
- **Integration**: It works seamlessly with other Linux features.
- **Industry Standard**: As mentioned, it's a widely used technology in professional environments.

**Key components you'll encounter with KVM**:

- **QEMU**: This is a generic machine emulator and virtualizer. KVM uses QEMU for the actual emulation of the virtual machine's hardware.
- **libvirt**: This is a toolkit and API to manage virtual machines. It provides a consistent way to interact with different virtualization technologies, including KVM.
- **virt-manager**: This is a user-friendly graphical tool built on top of libvirt, making it easier to manage your virtual machines.

## Checking If Your System Supports Virtualization

Checking if your Linux system supports virtualization is a pretty straightforward process.

Open your terminal, and run the following command:

```bash
grep -E --color 'vmx|svm' /proc/cpuinfo
```

Let's break down what this command does:

- `grep`: This is a powerful command-line utility used for searching plain-text data sets for lines matching a regular expression.
- `-E`: This tells `grep` to interpret the pattern as an extended regular expression.
- `--color`: This will highlight the matching text in the output, making it easier to see.
- `'vmx|svm'`: This is the regular expression we're searching for:
  - `vmx`: This indicates Intel's virtualization technology (Virtualization Technology eXtension).
  - `svm`: This indicates AMD's virtualization technology (Secure Virtual Machine).
  - `|`: This actions as an "OR" operator, so we're searching for either "vmx" or "svm".
- `/proc/cpuinfo`: This is a virtual file in Linux that contains detailed information about your system's CPU.

**What to look for in the output**:

- If you see lines with `vmx` (for Intel) or `svm` (for AMD) highlighted in the output, it means your CPU supports hardware virtualization, and it's likely enabled in your BIOS/UEFI settings.
- If you don't see any output after running the command, it could mean that your CPU doesn't support hardware virtualization, or it might be disabled in your BIOS/UEFI.

## Installing KVM and its Tools

The process for installing KVM and its related tools can vary slightly depending on which Linux distribution you're using. These are instructions for a common distribution like Ubuntu.

For Ubuntu, you'll need to install the following packages:

- `qemu-kvm`: This provides the core KVM functionality.
- `libvirt-daemon-system`: This is the system service for managing virtual machines.
- `libvirt-clients`: This provides the command-line tools for managing VMs (like virsh).
- `virt-manager`: This is the graphical management tool we talked about.
- `bridge-utils`: This is needed for setting up bridged networking, which allows your VMs to communicate directly with your network.

To install these packages, open your terminal and run the following command:

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients virt-manager bridge-utils
```

Let's break down this command:

- `sudo`: This allows you to run commands with administrator privileges. You'll likely be prompted to enter your password.
- `apt`: This is the package management tool used in Ubuntu (and other Debian-based distributions).
- `update`: This command refreshes the list of available packages from the software repositories. It's always a good idea to run this before installing new software.
- `install`: This command tells apt to download and install the specified packages.
- `qemu-kvm libvirt-daemon-system libvirt-clients virt-manager bridge-utils`: These are the names of the packages we want to install, separated by spaces.

After running this command, `apt` will download and install the necessary software. You might see some output in the terminal as it progresses. Once it's finished, KVM and its tools should be installed on your system.

## Creating the Virtual Machine

There are a couple of ways to create virtual machines with KVM:

1. **Graphical using `virt-manager`**: This provides a user-friendly interface to create and manage your virtual machines. It's often the easier way to get started.
2. **Command Line using `virt-install`**: This is a powerful tool that allows you to define all the VM parameters directly in the terminal. It's very flexible but can be a bit daunting for beginners.

### Using the Graphical virt-manager Interface to Create a Virtual Machine

To start `virt-manager`, simply open your application menu (or use a command launcher like Alt+F2) and search for "Virtual Machine Manager". Click on it to open the application.

Once `virt-manager` is open, you should see a window that might be empty if you haven't created any VMs before. To create a new virtual machine, you'll usually see a button that looks like a "+" or says "Create a new virtual machine". Click on that button.

A new window will pop up, guiding you through the steps of creating your virtual machine.

Once you've clicked the "Create a new virtual machine" button in `virt-manager`, you should see a window with several options. The first screen usually asks you to choose how you want to install the operating system. You'll likely see options like:

1. **Local install media (ISO image or CDROM)**
2. **Network install (HTTP, FTP, NFS)**
3. **Import existing disk image**

Since we want to install Ubuntu Server from scratch, we'll use an ISO image. You'll need to have already downloaded the Ubuntu Server ISO file to your computer. You can usually find the latest version on the official Ubuntu website.

**On this first screen of the wizard**:

- Select the option **"Local install media (ISO image or CDROM)"**.
- Click the **"Forward"** button.

The next screen will ask you to locate your installation media.

**On the second screen**:

- Click the **"Browse..."** button next to the "ISO image or CDROM" path.
- Navigate to the directory where you saved the Ubuntu Server ISO file, select it, and click **"Open"**.

Once you've selected the ISO file, `virt-manager` might automatically detect the operating system type. If it doesn't, you can manually select "Linux" as the OS type and "Ubuntu" as the version.

After you've selected the ISO, you should see the next screen in the "Create a new virtual machine" wizard. This screen is where you'll allocate CPU and memory resources for your virtual machine.

**You'll typically see options to specify**:

- **Memory (RAM)**: This determines how much of your computer's RAM will be dedicated to the virtual machine when it's running. The recommended amount will depend on the operating system you're installing and what you plan to do with the VM. For a basic Ubuntu Server installation, 1GB (1024 MB) or 2GB (2048 MB) is usually sufficient to get started. You can always adjust this later.

- **CPUs**: This determines how many of your computer's processor cores the virtual machine can use. Starting with 1 or 2 CPUs is usually a good idea for a server VM. Again, you can adjust this later if needed.

`virt-manager` will likely show you the total amount of RAM and the number of CPUs available on your host system. It's important not to allocate all of your resources to the VM, as your host operating system needs resources to run as well!

After allocating CPU and memory, the next crucial step is to **create a virtual disk** for your Ubuntu Server VM. This virtual disk will act as the hard drive for your virtual machine, where the operating system and your data will be stored.

On the next screen of the `virt-manager` wizard, you'll typically see options related to storage. You'll likely have a choice to:

1. **Create a disk image for the virtual machine**: This is the most common option for a new VM. You'll specify the size and location of the virtual disk file on your host system.

2. **Select or create custom storage**: This allows you to use an existing disk partition or create more advanced storage configurations.

For our Ubuntu Server setup, we'll choose the first option: **"Create a disk image for the virtual machine"**.

**You'll then need to specify**:

- **Size**: This is the capacity of your virtual hard drive. For a basic Ubuntu Server installation, 20GB is usually a good starting point. You can always resize it later if needed, but it's generally easier to start with a reasonable size.

- **Location**: This is where the virtual disk file will be stored on your host system. `virt-manager` will usually suggest a default location, which is often fine.

Think of this virtual disk as an empty hard drive that we're creating within a file on your computer. The operating system installer will then format this virtual drive and install Ubuntu Server onto it.

After configuring the virtual disk, the `virt-manager` wizard will usually present you with a summary of your virtual machine's configuration before you finalize the creation.

On this final screen, you'll see all the settings you've chosen:

- **Name**: The name you gave your virtual machine.
- **Installation Source**: The path to your Ubuntu Server ISO file.
- **Memory**: The amount of RAM allocated.
- **CPUs**: The number of virtual CPUs allocated.
- **Storage**: The size and location of your virtual disk.
- **Network**: The network configuration (it might be set to a default like "NAT" or "Virtual network").

Before clicking "Finish," it's a good idea to review these settings to make sure everything looks correct.

One important setting to pay attention to here is the Network configuration. By default, `virt-manager` might set your VM to use "NAT" (Network Address Translation). While this allows your VM to access the internet, it might not allow other machines on your local network to directly communicate with it.

For a server VM, you often want it to have its own IP address on your local network, just like a physical machine. To achieve this, you'll typically want to configure **bridged networking**.

#### Configuring a Bridged Network with virt-manager

Setting up bridged networking is a key step for a server VM. Let's go over what it is and how to configure it in virt-manager.

**NAT (Network Address Translation)**, which is often the default, works like this: your VM shares the IP address of your host machine. When the VM sends out network traffic, your host machine rewrites the source address so it appears to be coming from the host itself. The host then forwards the responses back to the correct VM. This is simple to set up and works well for internet access, but it makes it difficult for other machines on your local network to directly connect to services running on your VM.

**Bridged Networking**, on the other hand, creates a virtual network bridge on your host machine. This bridge allows your VM to have its own IP address on the same network as your host machine, as if it were a separate physical computer connected to your router. This enables direct communication between your VM and other devices on your local network.

Here's how to configure bridged networking in `virt-manager` before you click "Finish" on the VM creation wizard:

1. **Before clicking "Finish" on the summary screen, look for a section related to "Network source"**. It will likely say something like "Network: NAT" or list a "Virtual network" name (like "default").
2. **Click on the "Network source" line**. This should open up more detailed network options.
3. **In the dropdown menu for "Network source," you should see an option called "Bridge device"**. Select this option.
4. **Another dropdown menu will appear below "Bridge device"**. This menu will list the network interfaces (like `eth0`, `wlan0`, or similar) that are active on your host machine and can be bridged. Choose the network interface that your host machine uses to connect to your local network. For example, if you're connected via Ethernet cable, it might be `eth0`. If you're using Wi-Fi, it might be `wlan0`.
5. **Once you've selected the correct bridge device, click "Apply" (if there's an "Apply" button) or simply proceed**. The summary screen should now show your chosen bridge device as the network source.

Now, when your Ubuntu Server VM starts, it will try to obtain an IP address from your router's DHCP server, just like any other device on your network.

---

Now that you've configured the network settings, you should be back on the summary screen of the "Create a new virtual machine" wizard in `virt-manager`.

Take one last look at all the settings to ensure they are as you want them. You should see the name you chose for the VM, the path to the Ubuntu Server ISO, the allocated memory and CPUs, the size and location of the virtual disk, and the network source set to your bridge device.

At the bottom of this summary screen, you should see a checkbox that says something like **"Start installation before finishing"** or **"Begin installation"**.

**Make sure this checkbox is selected!** This will automatically start the virtual machine as soon as you click the **"Finish"** button, and it will boot from the Ubuntu Server ISO image we selected.

Once you're ready, click the **"Finish"** button.

A new window will likely pop up. This is the **virtual machine's console**. It's like having a direct monitor and keyboard connected to your virtual Ubuntu Server machine. You should see the Ubuntu Server installation process begin in this window.

The first screen you'll likely see will give you language options. You can use your arrow keys to navigate and press Enter to select. Follow the prompts in the installer. It will guide you through steps like choosing your language, configuring your keyboard layout, setting up networking (it should hopefully pick up an IP address via DHCP if bridged networking is configured correctly), creating a user account, and partitioning the virtual disk.

Go ahead and proceed through the Ubuntu Server installation process in the virtual machine's console. It's very similar to installing an operating system on a physical machine.

#### Basic VM Management with virt-manager

`virt-manager` provides a straightforward way to manage the lifecycle of your virtual machines. Here's how you can perform some basic actions:

1. **Starting a VM**:
    - If your VM is shut down, you can start it by selecting it in the `virt-manager` window and clicking the "Play" button (it looks like a right-pointing triangle).
2. **Stopping (Shutting Down) a VM**:
    - To gracefully shut down your VM, it's best to do it from within the guest operating system (Ubuntu Server in this case) using the appropriate command-line command (e.g., `sudo shutdown -h now`).
    - However, if the VM is unresponsive, you can force it to shut down by selecting it in `virt-manager` and clicking the **"Power Off"** button (it looks like a circle with a vertical line at the top). Be aware that this is like pulling the power cord on a physical machine and can lead to data loss if not done carefully.
3. **Pausing a VM**:
    - Pausing a VM saves its current state in memory and stops its execution. You can resume it later from the exact same point. To pause, select the VM and click the **"Pause"** button (it looks like two vertical lines).
    - To resume a paused VM, select it and click the **"Play"** button again.
4. **Deleting a VM**:
    - To delete a VM, first, make sure it's shut down. Then, right-click on the VM in the `virt-manager` window and select **"Delete"** (or sometimes "Remove").
    - A dialog box will appear asking if you also want to delete the storage volumes (the virtual disk files) associated with the VM. **Be very careful here!** If you only want to remove the VM configuration but keep the virtual disk, uncheck the option to delete storage. If you want to completely remove the VM and its data, leave the option checked.

#### Connect to the Console of a Running VM with virt-manager

You've already seen the console during the Ubuntu Server installation. Here's how to access it again if you close the window:

1. **Open `virt-manager`**: If it's not already open, find it in your application menu and launch it.
2. **Select your running VM**: In the virt-manager window, you'll see a list of your virtual machines. The ones that are running will likely have a green "Play" icon next to them. Click on your Ubuntu Server VM to select it.
3. **Open the virtual console**: Once the VM is selected, you'll see various buttons and information in the main `virt-manager` window. Look for a button that says **"Open"** or has a monitor icon. Clicking this button will open a new window displaying the console of your virtual machine.

This console gives you direct text-based access to your VM, allowing you to log in, run commands, and interact with the server. It's incredibly useful for managing your server, especially if you don't have a graphical interface installed on the VM (which is the case with Ubuntu Server by default).

#### Modifying an Existing NAT Network to Bridged with virt-manager

Let's go over **configuring bridged networking if you didn't do it during the VM creation** or **if you need to modify it later**.

You can adjust the network settings of an existing virtual machine in `virt-manager` by following these steps:

1. **Open `virt-manager`**: Launch the Virtual Machine Manager application.
2. **Select the VM**: Right-click on your Ubuntu Server virtual machine in the list.
3. **Open Virtual Machine Details**: In the context menu that appears, select "Open" or "Details". This will open a new window showing the configuration of your VM.
4. **Navigate to Network Interface**: In the left-hand pane of the details window, you should see a list of hardware components. Select the one that represents your network interface. It will likely be labeled something like "Network Interface" or have an icon of a network card.
5. **Change Network Source**: In the right-hand pane, you'll see the current network configuration. Look for the "Network source" setting. It might currently say "NAT" or the name of a virtual network. Use the dropdown menu next to it to select "Bridge device".
6. **Select Bridge Name**: Another dropdown menu will appear labeled "Bridge name". Here, you need to select the network bridge that corresponds to your host's physical network interface (e.g., `br0`, or the name of your Ethernet or Wi-Fi interface if a bridge hasn't been explicitly created yet). If a bridge doesn't exist, `virt-manager` might offer to create one for you, or you might need to create it manually using command-line tools (which is a more advanced topic we can discuss later if needed).
7. **Click "Apply"**: Once you've selected the "Bridge device" and the appropriate bridge name, click the "Apply" button at the bottom of the details window to save your changes.
8. **Restart the VM**: For the new network configuration to take effect, you'll likely need to shut down and then restart your Ubuntu Server virtual machine.

After the VM restarts, you can again use the `ip a` or `ip r` commands within the VM's console to verify that it has obtained an IP address on your local network.

Understanding how to reconfigure your VM's network settings is essential for adapting to different networking needs. It's like being able to change how a physical computer connects to the network without having to physically move cables!

### Using virt-install in the Command Line to Create a Virtual Machine

`virt-install` is a powerful tool that allows you to create virtual machines directly from your terminal. It takes various command-line options to define the VM's configuration, such as its name, memory, CPU allocation, disk size, and installation source.

While it might seem a bit more complex than the graphical `virt-manager`, using `virt-install` can be very efficient, especially when you want to automate VM creation or when you're working on a server without a graphical environment.

Here's a basic structure of a `virt-install` command for creating an Ubuntu Server VM:

```bash
virt-install \
--name=ubuntu-server-cli \
--memory=2048 \
--vcpus=2 \
--disk size=20,format=qcow2,bus=virtio \
--cdrom=/path/to/your/ubuntu-server.iso \
--network bridge=br0 \
--os-type=linux \
--os-variant=ubuntuserver-22.04 \
--graphics none \
--console pty,target_type=serial \
--extra-args="console=ttyS0,115200n8"
```

**Let's break down some of these options**:

- `--name`: Specifies the name of your virtual machine (ubuntu-server-cli in this case).
- `--memory`: Allocates RAM in megabytes (2048 MB = 2 GB).
- `--vcpus`: Assigns the number of virtual CPUs (2 in this case).
- `--disk`: Defines the virtual disk.
- `size`: The size of the disk in gigabytes (20 GB).
- `format`: The disk image format (qcow2 is a common and efficient format).
- `bus`: The type of virtual disk controller (virtio generally offers better performance).
- `--cdrom`: Specifies the path to your Ubuntu Server ISO file. You'll need to replace /path/to/your/ubuntu-server.iso with the actual path to your downloaded ISO.
- `--network bridge=br0`: Configures the network to use the br0 bridge (assuming you have one configured for bridged networking. If your bridge has a different name, use that instead).
- `--os-type`: Specifies the type of operating system (linux).
- `--os-variant`: Provides a more specific OS variant for better configuration (ubuntuserver-22.04). You might need to adjust this based on the exact version of Ubuntu Server you're installing. You can often find a list of valid variants using osinfo-query os.
- `--graphics none`: Indicates that we don't want a graphical display for the installation (since it's a server).
- `--console pty,target_type=serial`: Sets up a serial console for interacting with the installer.
- `--extra-args`: Passes kernel arguments to enable the serial console.

To use this command, you would open your terminal, replace the placeholder path to the ISO file with the correct path on your system, and then run the entire command. The `virt-install` tool will then create the virtual machine based on these specifications and start the installation process. You'll then interact with the Ubuntu Server installer through the serial console in your terminal.

#### Basic VM Mangement in the Command-Line Interface

For those who prefer the command line or are working in environments without a GUI, `libvirt` provides a powerful tool called `virsh` that allows you to manage your KVM virtual machines. Think of `virsh` as the command-line equivalent of `virt-manager`.

Here's how you can perform some of the actions we discussed using `virsh`:

- **Listing VMs**: To see a list of your defined virtual machines (both running and stopped), you can use the command:

    ```bash
    virsh list --all
    ```

This will show you the ID, name, and state of each VM.

- **Starting a VM**: To start a stopped virtual machine, use the command:

    ```bash
    virsh start <VM_NAME>
    ```

    Replace <VM_NAME> with the actual name of your virtual machine (e.g., `ubuntu-server-cli`).

- **Stopping a VM (Gracefully)**: To attempt a clean shutdown of a running VM, you can use:

    ```bash
    virsh shutdown <VM_NAME>
    ```

    This sends a shutdown signal to the guest operating system. It relies on the guest OS to respond and shut down properly.

- **Forcefully Stopping a VM**: If a VM is unresponsive, you can forcefully stop it using:

    ```bash
    virsh destroy <VM_NAME>
    ```

    **Warning**: This is like pulling the power cord and can lead to data loss. Use it as a last resort.

- **Connecting to the Console**: To connect to the text console of a running VM (especially one without a GUI, like Ubuntu Server), you can use:

    ```Bash
    virsh console <VM_NAME>
    ```

    You might need to press `Ctrl+]` to exit the console and return to your host terminal.

- **Getting VM Information**: To see detailed information about a specific VM, use:

    ```bash
    virsh dominfo <VM_NAME>
    ```

- **Suspending (Pausing) a VM**: To pause a running VM and save its state:

    ```bash
    virsh suspend <VM_NAME>
    ```

- **Resuming a VM**: To resume a suspended VM:

    ```bash
    virsh resume <VM_NAME>
    ```

- **Return to your terminal while leaving the VM running in the background**:

    To exit the virsh console and return to your host terminal while leaving the virtual machine running in the background, you need to use a specific key combination. This combination is:

    `Ctrl + ]`

    That is, press and hold the Ctrl key, and then press the right square bracket key (]). This will disconnect you from the VM's console, and the virtual machine will continue to run.

These are just some of the basic commands you can use with `virsh`. It's a very powerful tool that gives you complete control over your virtual machines from the command line.
