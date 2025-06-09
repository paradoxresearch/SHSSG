# Setting Up a Network Bridge

## Contents

- [Setting Up a Network Bridge](#setting-up-a-network-bridge)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Network Bridging](#understanding-network-bridging)
  - [Prerequisites](#prerequisites)
    - [Identifying Network Interfaces in More Depth](#identifying-network-interfaces-in-more-depth)
  - [Setting Up Your Network Bridge](#setting-up-your-network-bridge)
    - [Setting Up a Ethernet Network Bridge](#setting-up-a-ethernet-network-bridge)
      - [Setting Up a Ethernet Network Bridge With NetworkManager](#setting-up-a-ethernet-network-bridge-with-networkmanager)
      - [Setting Up a Ethernet Network Bridge With Netplan](#setting-up-a-ethernet-network-bridge-with-netplan)
    - [Setting Up a Wi-Fi Network Bridge](#setting-up-a-wi-fi-network-bridge)
      - [Setting Up a Wi-Fi Network Bridge with NetworkManager](#setting-up-a-wi-fi-network-bridge-with-networkmanager)
      - [Setting Up a Wi-Fi Network Bridge with netplan](#setting-up-a-wi-fi-network-bridge-with-netplan)
  - [Testing the Network Bridge with a Virtual Machine](#testing-the-network-bridge-with-a-virtual-machine)
    - [Configuring your Virtual Machine's Network Settings](#configuring-your-virtual-machines-network-settings)
      - [Configuring a Network Bridge with VirtualBox](#configuring-a-network-bridge-with-virtualbox)
      - [Configuring a Network Bridge with KVM/QEMU](#configuring-a-network-bridge-with-kvmqemu)
        - [For KVM/QEMU Using virt-manager](#for-kvmqemu-using-virt-manager)
        - [For KVM/QEMU Using the Command Line](#for-kvmqemu-using-the-command-line)
    - [Test to Verify the Bridge is Working Correctly](#test-to-verify-the-bridge-is-working-correctly)
      - [Pinging a Device on your Local Network from within the Virtual Machine](#pinging-a-device-on-your-local-network-from-within-the-virtual-machine)
        - [Troubleshooting the ping test further](#troubleshooting-the-ping-test-further)
  - [Removing a Network Bridge](#removing-a-network-bridge)
    - [Reverting NetworkManager Configuration Changes](#reverting-networkmanager-configuration-changes)
    - [Reverting netplan Configuration Changes](#reverting-netplan-configuration-changes)

## Introduction

Network bridging is very useful, especially if you're planning to work with virtual machines. In simple terms, a network bridge allows you to connect different network segments together as if they were a single network. This lets your virtual machines communicate directly with your local network.

### Understanding Network Bridging

Network bridging is a way to connect different parts of a network so they can work together as one. Imagine a bridge connecting two islands—without it, people on either side would have trouble traveling between them. Similarly, a network bridge allows devices on separate network segments to communicate as if they were on the same network, making data transfer seamless.

When you set up a network bridge, it essentially listens to data moving through the network and determines where it needs to go. Instead of sending all data everywhere, the bridge keeps track of which devices are on each side and forwards information only when necessary. This prevents unnecessary congestion and improves network efficiency. In a virtual machine setup, bridging is particularly useful because it lets the virtual machines behave like regular devices on your local network. They receive IP addresses from the same router as your physical computers, making integration and communication effortless.

By using network bridging, you ensure that your virtual machines function as full members of your network, rather than being isolated from other devices. This is helpful for tasks like running server applications, testing network configurations, or simply accessing shared resources without extra setup. Whether in home networks or enterprise environments, bridging simplifies connectivity and improves performance.

## Prerequisites

Before we can set up a network bridge on your Linux system, there are a few prerequisites:

1. **`bridge-utils` Package**: This is a collection of command-line tools for managing bridge interfaces. You'll need to make sure this packages is installed on your system.

    ```bash
    sudo apt update
    sudo apt install bridge-utils
    ```

    These commands will update the package list and then installs the `bridge-utils` package if it's not already there.

2. **Identifying Network Interfaces**: You need to know the names of your network interfaces. For Ethernet, it's often something like `eth0` or `w1pXsY`. You can find these names using the command `ip addr` in your terminal. Look for the interfaces that are currently connected to your network and have an UP address (for example, under `eth0` or `wlan0`).

### Identifying Network Interfaces in More Depth

The most common way to identify your network interface and names is by using the `ip addr` command in your terminal. You should see a output similar to:

```shell
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether yy:yy:yy:yy:yy:yy brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.105/24 brd 192.168.1.255 scope global wlan0
       valid_lft forever preferred_lft forever
```

You should see a list of network interfaces. Look for the ones that are currently active and have an IP address assigned to them.

- **Ethernet**: Ethernet interfaces often start with `eth` (like `eth0`) or `enp` followed by some numbers (like `enp3s0`). You'll likely see a line under it that starts with `inet` followed by an IP address. This is your active Ethernet connection if you have one plugged in.
- **Wi-Fi**: Wi-Fi interfaces usually start with `wlan` (like `wlan0`) or `wlp` followed by numbers (like `wlp2s0`). Similarly, if you're connected to Wi-Fi, you should see an `inet` line with an IP address under the Wi-Fi interface.
- **Loopback**: You'll also see an interface called `lo` or "loopback". This is your system's internal network interface and is not what we're looking for for bridging to your physical network.

In the example output above, `eth0` is the Ethernet interface and `wlan0` is the Wi-Fi interface, both with IP addresses, indicating they are active.

Think of `bridge-utils` as your toolkit for building the network bridge, and knowing your interface names is like knowing which cables you need to connect.

## Setting Up Your Network Bridge

### Setting Up a Ethernet Network Bridge

#### Setting Up a Ethernet Network Bridge With NetworkManager

To see if your system is managing its network configuration with NetworkManager, run the following command in your terminal:

```bash
sudo systemctl status NetworkManager.service
```

If you see a status that says "active (running)", then NetworkManager is likely handling your network.

You can further confirm that NetworkManager is in control of interfaces with the command:

```bash
nmcli networking
```

If the output in your terminal says `enabled`, we've confirmed NetworkManager is handling your network.

Creating a network bridge using NetworkManager involves using the `nmcli` command-line tool.

Here's how we can create a network bridge for your Ethernet connection using `nmcli`:

1. **Create the bridge interface**: Create a new bridge connection named `br0`.

    ```bash
    sudo nmcli connection add type bridge con-name br0 ifname br0
    ```

    This command tells NetworkManager to create a new connection of type `bridge` with the connection `br0` and the interface name `br0`.

2. **Add your Ethernet interface to the bridge**: Now, we need to tell NetworkManager to make your Ethernet interface (`eth0`) a "slave" to the `br0` bridge. First, we need to find the connection name associated with your Ethernet interface. You can do this with:

    ```bash
    nmcli connection show
    ```

    Look for the entry corresponding to your Ethernet interface (`eth0`). It will have a "NAME" associated with it. Let's assume the connection name is `Wired connection 1` (it might be different on your system).

    Then, use this command, replacing `Wired connection 1` with the actual connection name:

    ```bash
    sudo nmcli connection add type ethernet slave master br0 con-name bridge-slave-eth0 ifname eth0
    ```

    This command creates a new Ethernet connection that is a slave to the `br0` bridge.

3. **Disable the original Ethernet connection**: We need to disable the original connection for your Ethernet interface so that the bridge can take over. Again, using the connection name we found (e.g., `Wired connection 1`):

    ```bash
    sudo nmcli connection down 'Wired connection 1'
    ```

4. **Enable the bridge connection**: Now, bring up the bridge connection:

    ```bash
    sudo nmcli connection up br0
    ```

After running these commands, a new network bridge interface named `br0` should be active, and your Ethernet interface should be part of it. You can verify this by using the `ip addr` command again. You should see the `br0` interface with an IP address (if your network uses DHCP) and your `eth0` interface should no longer have an IP address directly assigned to it.

#### Setting Up a Ethernet Network Bridge With Netplan

You can check if your system is currently using `netplan` to manage your network configuration with the command:

```bash
systemctl status systemd-networkd
```

If you see an output in your terminal with a heading like:

```shell
● systemd-networkd.service - Network Configuration
     Loaded: loaded (/lib/systemd/system/systemd-networkd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-04-28 21:17:23 PDT; 8min ago
```

This output would suggest your network is being managed with `netplan`. Otherwise if you see an output like:

```shell
● systemd-networkd.service - Network Configuration
     Loaded: loaded (/lib/systemd/system/systemd-networkd.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-04-28 21:17:23 PDT; 8min ago
```

The "disabled;" part of the "`Loaded:`" line suggests your network is being managed with a different service.

If you have confirmed that `netplan` is handling your network configuration, we can move forward with configuring the bridge.

If your system uses `netplan`, you would typically have a configuration file in `/etc/netplan/`. Let's assume you have a file named `01-netcfg.yaml` (the name might vary). You would edit this file (after backing it up!) to define the bridge.

1. **Create a backup**: Let's save our old configuration in case something goes wrong, and we need to revert back to the original configuration:

    ```bash
    sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.backup
    ```

2. **Edit your `netplan` configuration**: After creating a backup of the original, edit your `netplan` configuration to reflect this configuration:

    ```shell
    sudo nano /etc/netplan/01-netcfg.yaml
    ```

    **Original** (Before edit):

    ```YAML
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp4: yes
          # dhcp6: yes
    ```

    **Create a bridge named `br0` with your Ethernet interface `eth0` using DHCP** (After edit):

    ```YAML
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp: no
      bridges:
        br0:
          interfaces: [eth0]
          dhcp4: yes
          dhcp6: no
    ```

    Here's a breakdown of your configuration file:

    - `network:`: This is the top-level section that encapsulates all network configurations. Think of it as the main container for all your network settings.
    - `version: 2`: This line specifies the version of the `netplan` configuration syntax being used. Currently, version 2 is common. It helps `netplan` correctly interpret the instructions.
    - `renderer: networkd`: This is a crucial line. It tells `netplan` which network management backend to use. In this case, we're specifying `networkd`, which is the network service provided by `systemd`. Another common renderer is `NetworkManager` itself. If `renderer` were set to `NetworkManager`, then NetworkManager would interpret the `netplan` configuration.
    - `ethernets:`: This section is used to configure your physical Ethernet interfaces.
      - `eth0:`: This identifies your specific Ethernet interface by its name.
      - `dhcp: no`: This line is important. It tells the `eth0` interface not to obtain an IP address automatically. This is because the bridge interface (`br0`) will be responsible for getting the IP address. If you didn't include this, you might have IP address conflicts.
    - `bridges:`: This section is where you define your bridge interfaces.
      - `br0:`: This is the name we've chosen for our bridge interface. You can name it something else if you prefer, but `br0` is a common convention.
      - `interfaces: [eth0]`: This line specifies which physical network interfaces should be part of this bridge. You can include multiple interfaces in this list (e.g., [eth0, eth1]) if you want to bridge them together. In our case, we're only including the Ethernet interface.
      - `dhcp4: yes`: This tells the bridge interface (`br0`) to obtain an IPv4 address automatically using DHCP (Dynamic Host Configuration Protocol) from your router. This is usually what you want for your VMs to easily connect to your network.
      - `dhcp6: no`: This line disables IPv6 DHCP on the bridge. You can set it to `yes` if you need IPv6 connectivity for your VMs and your network supports it.

    So, in essence, this `netplan` configuration tells the system to:

    1. Not assign an IP address directly to the `eth0` Ethernet interface.
    2. Create a bridge interface named `br0`.
    3. Make `eth0` a part of the bridge.
    4. Have the `br0` bridge obtain an IP address using DHCP.

3. **Apply the `netplan` configuration and verify the bridge**.

    After saving the changes to the `/etc/netplan/01-netcfg.yaml` configuration file, we can apply the changes with the following command in the terminal:

    ```bash
    sudo netplan apply
    ```

    This command reads your configuration files, generates the necessary configuration for the network backend (like `networkd`), and then applies those settings to your network interfaces.

    After running this command, it's essential to verify that the bridge interface (`br0`) has been created correctly and has obtained an IP address if you configured it to use DHCP. You can do this using the `ip addr` command again:

    ```bash
    ip addr
    ```

    In the output you should look for a few things:

    1. **A new interface named `br0`**: You should see a section for the `br0` interface.
    2. **An IP address for `br0` (if using DHCP)**: If you configured `dhcp4: yes` under the `br0` section in your `netplan` file, the `br0` interface should have an IP address listed under it (usually starting with `inet`).
    3. **No IP address on the bridged Ethernet interface**: Your original Ethernet interface (`eth0`, or whatever it might be in the `netplan` system) should no longer have an IP address directly assigned to it. This is because it's now part of the bridge, and the bridge interface handles the IP configuration.

    If you see these things, it means your Ethernet bridge has likely been created and configured successfully using `netplan`.

### Setting Up a Wi-Fi Network Bridge

#### Setting Up a Wi-Fi Network Bridge with NetworkManager

Just like with Ethernet, here is the general approach to create a Wi-Fi network bridge with **NetworkManager** and the `nmcli` tool:

1. **Create the bridge interface**: Just like with Ethernet, we'll create a new bridge connection named `br0`:

    ```Bash
    sudo nmcli connection add type bridge con-name br0 ifname br0
    ```

2. **Add your Wi-Fi interface to the bridge**: Now, we need to add your Wi-Fi interface (let's assume it's `wlan0`, though it might be different if your actively using a different Wi-Fi adapter) as a slave to the `br0` bridge. First, find the connection name associated with your Wi-Fi interface:

    ```bash
    nmcli connection show
    ```

    Look for the entry corresponding to your Wi-Fi interface (`wlan0` or similar). Let's assume the connection name is `wireless-connection-1`. Then use this command, replacing `wireless-connection-1` with your actual Wi-Fi connection name:

    ```bash
    sudo nmcli connection add type wifi slave master br0 con-name bridge-slave-wlan0 ifname wlan0
    ```

    **Imortant Note for Wi-Fi**: When adding the Wi-Fi interface as a slave, you might need to specify the SSID and password of your Wi-Fi network within this command. It would look something like this:

    ```bash
    sudo nmcli connection add type wifi slave master br0 con-name bridge-slave-wlan0 ifname wlan0 ssid "YourWiFiNetworkName" password "YourWiFiPassword"
    ```

    Replace `"YourWifiNetworkName"` and `"YourWifiPassword"` with your actual Wi-Fi credentials.

3. **Disable the original Wi-Fi connection**: Disable the original connection for your Wi-Fi interface:

    ```bash
    sudo nmcli connection down wireless-connection-1
    ```

    Again, replace `wireless-connection-1` with the correct connection name.

4. **Enable the bridge connection**: Bring up the bridge connection:

    ```bash
    sudo nmcli connection up br0
    ```

    After these commands, NetworkManager will attempt to connect your Wi-Fi interface to the bridge. You can then verify using `ip addr` to see if `br0` has an IP address.

**Potential Issues with Wi-Fi Bridging using NetworkManager**:

- **Connection Stability**: Wi-Fi bridging can be less stable than Ethernet Bridging. You might experience disconnects or performance issues.
- **Driver Compatibility**: As mentioned before, your Wi-Fi driver might not fully support this configuration.
- **NetworkManager Behavior**: NetworkManager's behavior with Wi-Fi bridges can sometimes be unpredictable.

It's important to be aware of these potential challenges. If you encounter issues with a Wi-Fi bridge, you might need to research specific bridge configurations for your Wi-Fi adapter and drivers, or consider using an Ethernet connection for bridging if possible.

#### Setting Up a Wi-Fi Network Bridge with netplan

Similar to Ethernet bridging, you need to edit your `netplan` configuration file.  For this example we'll say it's `/etc/netplan/01-netcfg.yaml`, but the name of your `netplan` configuration file may vary.  Let's also assume  your Wi-Fi interface is named `wlan0` and you want to create a bridge named `br0` that includes this interface and obtains an IP address via DHCP. Here's and example of how your `netplan` configuration might look:

```YAML
network:
  version: 2
  renderer: networkd
  wifis:
    wlan0:
      dhcp: no
  bridges:
    br0:
      interfaces: [wlan0]
      dhcp4: yes
      dhcp6: no
```

Here are the key differences and points to note compared to the Ethernet bridge configuration:

- **`wifis:` section**: Instead of `ethernets:`, we now have a `wifis:` section to configure the Wi-Fi interface.
- `wlan0:`: This identifies your Wi-Fi interface.
- **`dhcp: no` under `wlan0:`**: Just like with the Ethernet interface, we disable DHCP directly on the Wi-Fi interface because the bridge will handle the IP configuration.
- **`interfaces: [wlan0]` under `bridges: br0:`**: We specify the Wi-Fi interface `wlan0` as the interface to be included in the bridge.

After saving this `netplan` configuration, you would apply it using:

```bash
sudo netplan apply
```

And then you would verify the bridge (`br0`) and the status of your Wi-Fi interface using `ip addr`, just as we did with the Ethernet bridge.  You'd look for the `br0` interface with an IP address and the `wlan0` interface without a direct IP address.

## Testing the Network Bridge with a Virtual Machine

Creating a network bridge is a great way to allow your virtual machines to communicate directly with your local network.

### Configuring your Virtual Machine's Network Settings

The exact steps for configuring your virtual machine's network will depend on the virtualization software you are using. Some popular options on Linux include:

- **VirtualBox**: In VirtualBox, when you configure the network settings for a virtual machine, you typically have a few options for the "Attached to" field. To use the network bridge you created, you would usually select "Bridged Adapter." After selecting this, a dropdown menu will appear where you can choose which physical network interface the VM should bridge to. In our case, you would select the bridge interface you created, which is likely named `br0`.
- **KVM/QEMU**: When using KVM (Kernel-based Virtual Machine) with QEMU, you would configure the network interface for your VM in its virtual machine definition. This often involves created a virtual network interface and then "attaching" it to the bridge interface (`br0`) on your host system. This can be done through command-line options when starting the VM or by editing the VM's XML configuration if you're using a management tool like `virt-manager`. You would typically specify the type of network device to be "bridge" and then provide the name of your bridge interface (`br0`).

No matter which virtualization software you're using, the key idea is to tell the VM's network settings to connect to the bridge interface (`br0`) that you've created on your host machine. This makes the VM appear as another device directly connected to your local network. It will then typically obtain its own IP address from your router via DHCP, just like your physical laptop or other devices on your network.

Here's how you would typically configure the network settings in these two popular virtualization platforms to use the `br0` network bridge:

#### Configuring a Network Bridge with VirtualBox

1. **Select your Virtual Machine**: In the VirtualBox Manager, select the virtual machine you want to configure.
2. **Go to Settings**: Click on the "Settings" button for the selected VM.
3. **Navigate to Network**: In the VM settings window, go to the "Network" tab.
4. **Attached to**: In the "Attached to" dropdown menu, select "Bridged Adapter."
5. **Name**: A new dropdown menu labeled "Name" will appear. Here, you need to select the network bridge interface you created on your host system. This will likely be named `br0`. If you have multiple network interfaces, make sure you choose the one corresponding to your bridge.
6. **Adapter Type and other settings (optional)**: You can usually leave the "Adapter Type" at its default setting. You might have other advanced options, but for basic bridging, the "Attached to" and "Name" settings are the most crucial.
7. **Click "OK"**: Save the changes to your VM's network settings.

When you start this virtual machine, its network interface will now be bridged to your `br0` interface, allowing it to communicate directly with your local network.

#### Configuring a Network Bridge with KVM/QEMU

##### For KVM/QEMU Using virt-manager

 1. **Open virt-manager**: If you're using the graphical management tool virt-manager, open it.
 2. **Select your Virtual Machine**: Double-click on the virtual machine you want to configure.
 3. **Open Virtual Machine Details**: In the VM's overview window, click on "Show virtual hardware details" (it looks like a lightbulb or an "i" icon).
 4. **Select Network Interface**: In the hardware list, select your virtual network interface (it might be labeled "Network" or something similar).
 5. **Connection type**: In the right-hand pane, look for the "Connection type" setting. Change this to "Bridge device."
 6. **Bridge name**: A field labeled "Bridge name" will appear. Enter the name of your bridge interface, which is `br0`.
 7. **Network source**: Ensure the "Network source" is set to the bridge you just specified (`br0`).
 8. **Click "Apply"**: Save the changes to your VM's network configuration.

When you start this virtual machine, its virtual network interface will be connected to the `br0` bridge on your host.

##### For KVM/QEMU Using the Command Line

If you are starting your VMs from the command line using `qemu-system-x86_64` (or similar), you would typically use the `-netdev` and `-device` options to configure networking. To connect to a bridge, you would define a network device of type `bridge` and specify the bridge name:

```bash
qemu-system-x86_64 -enable-kvm -m 2G -cdrom your_image.iso -drive file=your_disk.img,format=qcow2 -netdev type=bridge,br=br0,id=net0 -device virtio-net-pci,netdev=net0
```

In this example:

- `-netdev type=bridge,br=br0,id=net0` creates a network backend connected to the `br0` bridge and gives it the ID `net0`.
- `-device virtio-net-pci,netdev=net0` creates a virtual network card in the VM and connects it to the `net0` network backend.

Once your VM is configured to use the `br0` bridge, it should, upon starting, attempt to obtain an IP address from your local network's DHCP server.

### Test to Verify the Bridge is Working Correctly

Once your virtual machine is running and configured to use the `br0` bridge, it should behave like any other device on your local network. It should ideally obtain an IP address from your router (if you're using DHCP on the bridge, which is the common setup).

Here's a simple and effective way to test if the network bridge is working correctly:

#### Pinging a Device on your Local Network from within the Virtual Machine

 1. **Identify another device on your local network**: This could be your host laptop, another computer, a printer, or anything else connected to the same router. You'll need to know its IP address. You can usually find this information in your router's administration interface or by using network scanning tools. For your host laptop, you can use `ip addr` (or `ipconfig` on Windows if you happen to be running a Windows VM).

 2. **Open a terminal or command prompt in your virtual machine**: Once your VM has started up and has (hopefully) obtained an IP address, open a terminal or command prompt within the guest operating system.

 3. **Use the `ping` command**: Use the ping command followed by the IP address of the other device you identified in step 1. For example, if another device on your network has the IP address `192.168.1.5`, you would run:

    ```bash
    ping 192.168.1.5
    ```

 4. **Observe the output**: If the bridge is working correctly and your VM has network connectivity, you should see replies from the IP address you are pinging. The output will typically show the time it took for each "echo request" to be sent and a "reply" to be received.

 5. **Troubleshooting**: If you don't get any replies, it could indicate a problem with your bridge configuration, the VM's network settings, or general network connectivity. You'll need to double-check each step we've taken. Make sure the `br0` interface is up on your host, your VM is configured to use it, and your VM has obtained an IP address (you can check this within the VM using tools like `ip addr` on Linux or `ipconfig` on Windows).

##### Troubleshooting the ping test further

You've configured your VM to use the `br0` network bridge, and you've started the VM. You open a terminal in the VM and try to ping your host laptop's IP address, but you're not getting any replies. Here are some steps to help diagnose and fix the issue:

1. **VM Network Configuration**:

    - **Check the "Attached to" setting**: Ensure that your VM's network adapter is indeed set to "Bridged Adapter" and that the "Name" field correctly points to `br0` in VirtualBox. For KVM, double-check that the network interface is set to "Bridge device" and the bridge name is `br0`.

    - **Verify IP Address in VM**: Make sure your VM has actually obtained an IP address. Use `ip addr` (Linux) or `ipconfig` (Windows) within the VM. If it hasn't gotten an IP address, there might be an issue with the bridge not forwarding DHCP requests or a problem with your router's DHCP server. You might need to restart the VM or the networking service within the VM.

2. **Bridge Interface Status on Host**:

    - **Check if `br0` is up**: On your host machine, run `ip addr` show `br0`. Ensure the state is "UP" and that it has an IP address (if you configured it for DHCP).

    - **Check if the physical interface is enslaved**: Also on your host, run `brctl show`. This command (from `bridge-utils`) will show you the bridge interfaces and the ports (physical interfaces) they contain. Make sure your Ethernet (`eth0` or similar) or Wi-Fi (`wlan0` or similar) interface is listed as a port on `br0`.

3. **Firewall Issues**:

    - **Host Firewall**: Your host system might have a firewall (like `ufw`) enabled that is blocking traffic to or from the bridge interface. You might need to configure it to allow traffic on `br0`.

    - **Guest Firewall**: Similarly, the operating system inside your VM might have a firewall that's preventing it from responding to ping requests. You'll need to check the firewall settings within the VM.

4. **Network Connectivity**:

    - **Host Connectivity**: Ensure your host machine itself has a working network connection. If your host can't access the network, the bridge won't be able to pass traffic.

    - **Router Issues**: In rare cases, there might be an issue with your router not properly handling DHCP requests or ARP (Address Resolution Protocol) for bridged interfaces. A router restart might sometimes help.

## Removing a Network Bridge

It's crucial to know how to undo the changes you've made, whether you no longer need the bridge or if you encountered any issues.

The process for removing a network bridge depends on how you created it in the first place (using `netplan` or NetworkManager).

### Reverting NetworkManager Configuration Changes

This is the method you would use if you created the bridge using NetworkManager's `nmcli` tool, as we did for your system.

Here's how to remove the br0 bridge interface using command-line tools:

 1. **Disable the bridge interface**: First, you need to bring the `br0` interface down. This stops all network traffic going through it. Use the command:

    ```bash
    sudo ip link down br0
    ```

 2. **Delete the bridge interface**: Once the interface is down, you can delete it entirely using the command:

    ```bash
    sudo ip link del br0
    ```

However, if you created the bridge using NetworkManager (with `nmcli`), simply deleting the `br0` interface might not be enough. NetworkManager has its own configuration for the bridge and its member interfaces. To properly remove the bridge managed by NetworkManager, you should use `nmcli` to delete the connections you created:

1. **Identify the bridge and slave connections**: First, list your NetworkManager connections to find the ones we created for the bridge.

    ```bash
    nmcli connection show
    ```

    You'll be looking for connections named something like `br0` and `bridge-slave-eth0` (or `bridge-slave-wlan0` if you bridged Wi-Fi).

2. **Delete the slave connection**: Delete the connection for the physical interface that was part of the bridge. For example:

    ```bash
    sudo nmcli connection delete bridge-slave-eth0
    ```

    If you bridged Wi-Fi, you would delete the Wi-Fi slave connection instead.

3. **Delete the bridge connection**: Finally, delete the bridge connection itself:

    ```bash
    sudo nmcli connection delete br0
    ```

4. **Re-enable the original interface connection**: After deleting the bridge connections, you'll likely want to re-enable the original connection for your Ethernet or Wi-Fi interface so it can get an IP address again:

    ```bash
    sudo nmcli connection up ethernet-eth0
    ```

    or

    ```bash
    sudo nmcli connection up wifi-wlan0
    ```

    (Use the correct connection name for your interface).

### Reverting netplan Configuration Changes

If you created the network bridge by editing your netplan configuration file, the primary way to remove it is to **revert those changes**.

Here's what you would typically need to do:

 1. **Identify the changes you made**: Remember the modifications you made to your `.yaml` file in `/etc/netplan/`. This likely involved:
    - Adding a `bridges:` section.
    - Modifying the configuration of your Ethernet (`ethernets:`) or Wi-Fi (`wifis:`) interface to disable DHCP.
    - Listing the physical interface under the `interfaces:` of the bridge.

 2. **Restore the original configuration (if you backed it up)**: If you followed our earlier advice and backed up your original `netplan` configuration file, the easiest way to remove the bridge is to restore that backup. For example, if you backed up `01-netcfg.yaml` to `01-netcfg.yaml.backup`, you would use a command like:

    ```bash
    sudo cp /etc/netplan/01-netcfg.yaml.backup /etc/netplan/01-netcfg.yaml
    ```

    Then, you would need to apply this restored configuration:

    ```bash
    sudo netplan apply
    ```

 3. **Manually edit the configuration (if you didn't back up)**: If you didn't create a backup, you'll need to manually edit the `netplan` configuration file again to remove the bridge-related entries and re-enable the original settings for your Ethernet or Wi-Fi interface. This would involve:
    - Deleting the entire `bridges:` section.
    - Going back to the `ethernets:` or `wifis:` section for your physical interface and setting `dhcp4: yes` (and `dhcp6: yes` if you need IPv6) to re-enable DHCP on the physical interface directly.

    For example, if you had this before creating the bridge:

    ```YAML
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp4: yes
          # dhcp6: yes
    ```

    And you changed it to create a bridge, you would need to revert it back to this (or a similar state depending on your original configuration). After editing, remember to apply the changes:

    ```bash
    sudo netplan apply
    ```

After applying the reverted configuration, your system should no longer have the `br0` bridge interface, and your Ethernet or Wi-Fi interface should be back to obtaining an IP address directly. You can verify this using `ip addr`. You should not see the `br0` interface anymore, and your physical interface (`eth0`, `wlan0`, etc.) should have an IP address.
