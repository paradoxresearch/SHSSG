# Configuring Static IP Addresses On Your Linux System

## Contents

- [Configuring Static IP Addresses On Your Linux System](#configuring-static-ip-addresses-on-your-linux-system)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Gather Network Information](#gather-network-information)
    - [Get Your Routers Gateway Address](#get-your-routers-gateway-address)
    - [Get Your Routers Name Servers](#get-your-routers-name-servers)
  - [Configure Your Static IP Address](#configure-your-static-ip-address)
    - [Configure Your Static IP Address with Netplan](#configure-your-static-ip-address-with-netplan)
      - [Backup Your Netplan Configuration](#backup-your-netplan-configuration)
      - [Edit Your Netplan Configuration](#edit-your-netplan-configuration)
      - [Troubleshooting Your Netplan Configuration](#troubleshooting-your-netplan-configuration)
    - [Configure Your Static IP Address with NetworkManager](#configure-your-static-ip-address-with-networkmanager)
      - [Set A Static IP Address Using NetworkManager GUI](#set-a-static-ip-address-using-networkmanager-gui)
      - [Set A Static IP Address Using the Command-Line with NetworkManager](#set-a-static-ip-address-using-the-command-line-with-networkmanager)
      - [Troubleshooting Your NetworkManager Configuration](#troubleshooting-your-networkmanager-configuration)

## Introduction

Think of an IP address like a home address for your computer on a network. It allows devices to find each other and communicate. Now, there are two main ways a device can get an IP address: dynamically or statically.

**Dynamic IP addresses** are like temporary addresses assigned by a central system (usually your router) each time a device connects to the network. It's convenient, like a hotel assigning you a room number when you check in.

**Static IP addresses**, on the other hand, are like owning your home – the address stays the same. You manually configure it on your device, and it doesn't change unless you change it.

Why might you want a static IP? Well, for things like hosting a web server, running a file server, or even for consistent access to a printer on your home network. It ensures that the address of your service or device remains constant, making it easier for others (or other devices on your network) to find it reliably.

In the Linux world, different tools help manage these network addresses. Two common ones are **Netplan** and **NetworkManager**.

**Netplan** is a relatively newer configuration utility that uses YAML files to define network interfaces. It then works with a "renderer" (like `networkd` or `NetworkManager`) in the background to apply these configurations. It's often the default on newer Ubuntu server versions.

**NetworkManager** is a more traditional and widely used service, especially in desktop environments. It can manage network connections through both graphical interfaces and command-line tools like nmcli.

So, how do you tell which one is active? Here are a couple of ways:

- **Check for Netplan configuration files**: If you see files ending with `.yaml` in the `/etc/netplan/` directory, it's a strong indicator that Netplan is being used. You can check this using the command:

    ```bash
    ls /etc/netplan
    ```

    If you see an output like `50-cloud-init.yaml`, Netplan is likely managing the network configuration.

- **Check the status of NetworkManager service**: You can check if the NetworkManager service is running using the `systemctl` command:

    ```bash
    systemctl status NetworkManager
    ```

    If it's active and running, NetworkManager is likely managing your network. However, keep in mind that on systems using Netplan, NetworkManager might still be installed but acting as the renderer for Netplan's configurations.

    A key thing to remember is that often, only one of these will be actively managing the main network interfaces.

## Gather Network Information

The first step, is to gather some essential information about your current network setup while it's still connected via DHCP. This information includes:

- **Router Gateway Address**: This is the IP address of your router, which acts as the gateway to the outside internet. Your computer needs to know this address to send traffic beyond your local network. Think of it as the main exit door of your network.
- **DNS Servers**: These servers translate website names (like <https://www.google.com/search?q=google.com>) into IP addresses that computers understand. Without DNS servers, you'd have to remember the numerical IP address of every website you want to visit! Your router often provides DNS server addresses, or you can use public ones.

### Get Your Routers Gateway Address

To obtain your network's gateway address, type the following command in your terminal:

```bash
ip route
```

The line starting with `default via` will show you the gateway IP.

**Example Output**:

```shell
default via 192.168.1.254 dev eth0 proto dhcp src 192.168.1.240 metric 100
```

The network gateway address is: `192.168.1.254`

### Get Your Routers Name Servers

To obtain your routers name servers, type the following commands in your terminal:

```bash
resolvectl status | grep "DNS Servers"
```

**Example Output**:

```shell
DNS Servers: 192.168.1.254 2600:1701:400:df0::1
```

The network nameservers are: `192.168.1.254` `2600:1701:400:df0::1`

Why is this information important? When you set a static IP, you'll need to manually provide these details so your computer knows how to communicate with the rest of the network and the internet. It's like manually setting the coordinates for your home address and the directions to the main highway!

## Configure Your Static IP Address

### Configure Your Static IP Address with Netplan

#### Backup Your Netplan Configuration

Alright, now that you've gathered your gateway and DNS server information, the next crucial step is to take a look at your current Netplan configuration file. It's always a good idea to see what's there before you start making changes, and even better to make a backup, just in case!

The Netplan configuration files are usually located in the `/etc/netplan/` directory. The filenames can vary, but often they have names like `50-cloud-init.yaml` or `01-network-manager-all.yaml`.

To see the files in this directory, you'd use the command:

```bash
ls /etc/netplan
```

Once you identify the `.yaml` file (or files), it's wise to make a copy before you start editing. This way, if anything goes wrong, you can easily revert to the original configuration. You can do this using the `cp` command (copy):

```bash
sudo cp /etc/netplan/your_config_file.yaml your_config_file-backup.yaml
```

**Important**: You'll need to replace `your_config_file.yaml` with the actual name of the file you found in the `/etc/netplan/` directory. The `sudo` command is used because these configuration files require administrator privileges to modify or copy. You might be asked for your password when you run this command.

Making a backup is a simple step that can save you a lot of potential headaches later on!

#### Edit Your Netplan Configuration

Now for the part where we tell Netplan to use a static IP address instead of DHCP. This involves editing the YAML configuration file you found in `/etc/netplan/`. You'll use a text editor for this, and `nano` is a common and user-friendly option.

To open the configuration file for editing, you'd use a command like this (again, replace your_config_file.yaml with the actual filename):

```bash
sudo nano /etc/netplan/your_config_file.yaml
```

This will open the file in the `nano` text editor within your terminal. You'll likely see something similar to the default configuration you included in your notes:

```YAML
# This file is generated from information provided by the datasource. Changes
# to it will not persist across an instance reboot. To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
```

Now, you'll need to modify this file to configure your static IP address. Here's how you'd typically do it, incorporating the information you gathered earlier:

```YAML
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: false  # Disable DHCP for IPv4
            addresses: [192.168.1.20/24]  # Your desired static IP and subnet mask
            routes:
                - to: default
                  via: 192.168.1.254  # Your router's gateway address
            nameservers:
                addresses: [192.168.1.254, 8.8.8.8]  # Your router's DNS and a public DNS (optional)
            optional: true
```

Let's break down these changes:

- `dhcp4: false`: This line tells Netplan to disable DHCP for the IPv4 address on the eth0 interface (you might have a different interface name like enp0s3).
- `addresses: [192.168.1.20/24]`: Here, you specify the static IP address you want to use (`192.168.1.20` in this example). The `/24` indicates the subnet mask (`255.255.255.0`), which defines the range of IP addresses in your local network. You'll want to choose an IP address within your network's range that isn't already in use.
- `routes:`: This section defines how your computer should send traffic outside your local network.
- `- to: default`: This means the default route (for all traffic not within your local network).
- `via: 192.168.1.254`: This is where you put the gateway address of your router that you found earlier.
- `nameservers:`: This section specifies the DNS servers your computer should use.
- `addresses: [192.168.1.254, 8.8.8.8]`: Here, you list the IP addresses of your DNS servers. In this example, we've included the router's IP as the primary DNS server and Google's public DNS server (8.8.8.8) as a secondary option for redundancy. You can use the DNS servers you found earlier.

**Important Considerations**:

- **Interface Name**: Make sure that eth0 in the configuration matches the actual name of your network interface. You can usually find this using the `ip addr show` command.
- **IP Address Choice**: Choose a static IP address within the same network range as your router but outside the range that your router typically assigns via DHCP to avoid conflicts. For example, if your router assigns IPs in the range of `192.168.1.100` to `192.168.1.200`, an IP like `192.168.1.20` would likely be a safe choice.
- **Subnet Mask**: The `/24` subnet mask is common for home networks. If you're unsure about your subnet mask, it's usually the same as the one listed when you checked your IP configuration with `ip addr show`.

After making these changes in `nano`, you would save the file by pressing `Ctrl+O`, then press `Enter` to confirm, and then exit by pressing `Ctrl+X`.

Now, the changes you've made in the text file aren't active yet. You need to tell the system to apply this new configuration.

```bash
sudo netplan apply
```

This command reads the configuration files in `/etc/netplan/` and applies the settings to your network interfaces.

After running this command, your network interface should attempt to reconfigure itself with the static IP address you specified. In some cases, especially on a server without a graphical interface, you might need to restart the networking service or even the entire system to ensure the changes are fully applied.

```bash
sudo shutdown -r now
```

**Important Considerations After Applying**:

- **Connectivity Check**: After applying the configuration (and potentially restarting), it's crucial to check if your network connection is working as expected. You can do this by:
  - Trying to ping your gateway address (e.g., `ping 192.168.1.254`). If you get a response, it means your computer can communicate with your router.
  - Trying to ping an external website by its IP address (e.g., `ping 8.8.8.8`). If this works, it means you have internet connectivity and your DNS configuration is likely correct.
  - Trying to ping a website by its name (e.g., `ping google.com`). If this works, it confirms that your DNS resolution is functioning.

- **Potential Issues**: Sometimes, applying the new configuration might lead to issues, such as:
  - **IP Address Conflicts**: If the static IP address you chose is already being used by another device on your network, you might experience connectivity problems.
  - **Incorrect Configuration**: A typo in the Netplan YAML file (e.g., incorrect IP address, gateway, or subnet mask) can prevent the network from working correctly.
  - **Firewall Issues**: In some cases, firewall settings might need to be adjusted to allow traffic on the new static IP address, although this is less common with basic static IP configuration.

If you encounter issues, the first step is usually to review your Netplan configuration file for any errors and ensure that the IP address you've chosen is appropriate for your network. You can also try reverting to your backup configuration file if necessary.

#### Troubleshooting Your Netplan Configuration

If you find yourself without network connectivity after running sudo netplan apply and potentially restarting, here are some steps you can take to troubleshoot:

1. **Check Your Netplan Configuration File**:
   - Use the `cat /etc/netplan/your_config_file.yaml` command (again, replace `your_config_file.yaml` with your actual filename) to view the contents of the file.
   - Carefully review it for any typos or syntax errors. YAML is sensitive to indentation and spacing, so make sure everything is correctly aligned. For example, are the colons in the right place? Are there any extra spaces?
   - Double-check that the IP address, gateway, and DNS server addresses are correct and match the information you gathered earlier.
   - Ensure the interface name (eth0 or whatever it is on your system) is correct.

2. **Try Applying Again**: Sometimes, the configuration might not apply correctly on the first try. You can try running sudo netplan apply again.

3. **Check Network Interface Status**: Use the command `ip a` or `ip addr` show to see the status of your network interfaces. Look for your configured interface (e.g., `eth0`) and check if it has the static IP address you intended to set. If it doesn't, there's likely an issue with your Netplan configuration.

4. **Ping Your Localhost**: Try pinging your own computer using the command `ping 127.0.0.1`. This checks if the basic network interface is working. If this fails, there might be a more fundamental issue with your network setup.

5. **Revert to the Backup**: If you're still having trouble, the backup you created earlier can be a lifesaver! You can revert to the original configuration by copying the backup file back to the original filename:

    ```bash
    sudo cp /etc/netplan/your_config_file-backup.yaml /etc/netplan/your_config_file.yaml
    ```

   - Then, apply the configuration again with `sudo netplan apply` and restart if necessary. This will put your network configuration back to its previous state.

6. **Check System Logs**: The system logs can sometimes provide more detailed information about what went wrong during the network configuration process. You can view the logs using commands like `journalctl -u systemd-networkd` (if `networkd` is your Netplan renderer) or `journalctl -u NetworkManager` (if NetworkManager is the renderer). Look for any error messages related to network configuration.

7. **Restart Your Router**: Sometimes, network issues can be related to your router. A simple restart of your router can often resolve unexpected connectivity problems.

Troubleshooting is a skill that develops with practice. Don't get discouraged if things don't work perfectly the first time. The error messages and the process of finding the solution are valuable learning experiences.

### Configure Your Static IP Address with NetworkManager

You'll often encounter NetworkManager on desktop Linux distributions like Ubuntu Desktop, Fedora, Mint, etc. It provides both graphical and command-line tools for managing network connections.

#### Set A Static IP Address Using NetworkManager GUI

The most straightforward way to configure a static IP with NetworkManager in a desktop environment is usually through the **graphical user interface (GUI)**. The exact steps might vary slightly depending on your desktop environment (GNOME, KDE, XFCE, etc.), but the general process is similar:

1. **Open Network Settings**: Look for the network icon in your system tray (usually in the top or bottom right corner of your screen). Click on it to open the network menu. You'll typically find an option like "Network Connections," "Edit Connections," or "Network Settings."

2. **Select Your Connection**: In the network settings, you'll see a list of your network interfaces (e.g., Wi-Fi, Ethernet). Find the Ethernet connection you want to configure with a static IP and select it. There might be an "Edit" or "Settings" button associated with it.

3. **IPv4 Settings**: In the connection settings window, look for a tab or section related to "IPv4 Settings" or something similar. Here, you'll usually see a dropdown menu for "IPv4 Method" or "Address Type." By default, it's likely set to "Automatic (DHCP)."

4. **Manual Configuration**: Change the "IPv4 Method" or "Address Type" to "Manual." This will reveal fields where you can enter your static IP address, subnet mask, gateway, and DNS servers.

5. **Enter Details**:
    - **Address**: Enter your desired static IP address (e.g., `192.168.1.20`).
    - **Netmask**: Enter your network's subnet mask. Often, this will be `255.255.255.0` or you might see it represented as a prefix like /24.
    - **Gateway**: Enter the IP address of your router (the gateway you found earlier).
    - **DNS Servers**: Enter the IP addresses of your DNS servers, separated by commas if you have multiple. You can use your router's IP and/or public DNS servers like `8.8.8.8`.

6. **Apply Changes**: Once you've entered all the necessary information, click "Apply," "Save," or a similar button to save your changes. You might need to disconnect and reconnect to the network interface for the new settings to take effect. Sometimes, a system reboot might be necessary.

The key here is that NetworkManager provides a user-friendly graphical way to configure these settings without directly editing configuration files. It handles the underlying details for you.

#### Set A Static IP Address Using the Command-Line with NetworkManager

`nmcli` (NetworkManager Command Line Interface) allows you to control NetworkManager and manage network connections directly from your terminal. Here's how you can use it to set a static IP address:

1. **Identify Your Connection Name**: First, you need to know the name of the network connection you want to modify. You can list the active connections using the command:

    ```bash
    nmcli connection show --active
    ```

    This will display a table of active network connections. Look for the connection associated with your Ethernet interface (it might be something like "Wired connection 1" or the name of your interface like "eth0"). Note down the NAME of this connection.

2. **Modify the Connection**: Now, you'll use the `nmcli connection modify` command to set the static IP address. Here's the general syntax:

    ```bash
    nmcli connection modify "your_connection_name" ipv4.method manual ipv4.addresses "your_static_ip/subnet_mask" ipv4.gateway "your_gateway_ip" ipv4.dns "your_dns_ip1,your_dns_ip2"
    ```

    **Replace the placeholders with your actual information**:

   - `"your_connection_name"`: The name of the connection you identified in the previous step (e.g., "Wired connection 1" or "eth0").
   - `ipv4.method manual`: This tells NetworkManager to use manual IP configuration (i.e., static).
   - `ipv4.addresses "your_static_ip/subnet_mask"`: Replace "your_static_ip" with the desired static IP address and "subnet_mask" with your network's subnet mask (e.g., 192.168.1.20/24).
   - `ipv4.gateway "your_gateway_ip"`: Replace "your_gateway_ip" with the IP address of your router's gateway.
   - `ipv4.dns "your_dns_ip1,your_dns_ip2"`: Replace "your_dns_ip1" and "your_dns_ip2" with the IP addresses of your DNS servers, separated by a comma. You can specify one or more DNS servers.

    **Example**:

    Assuming your connection name is "eth0", you want to set the static IP to `192.168.1.25`, the subnet mask is `/24`, the gateway is `192.168.1.1`, and you want to use the DNS servers `192.168.1.1` and `8.8.8.8`, the command would look like this:

    ```bash
    nmcli connection modify eth0 ipv4.method manual ipv4.addresses "192.168.1.25/24" ipv4.gateway "192.168.1.1" ipv4.dns "192.168.1.1,8.8.8.8"
    ```

3. **Apply the Changes**: After modifying the connection, you need to tell NetworkManager to apply the new settings. You can do this by deactivating and then reactivating the connection:

    ```bash
    nmcli connection down "your_connection_name"
    nmcli connection up "your_connection_name"
    ```

    Again, replace "your_connection_name" with the name of your connection.

After these steps, your network interface should be configured with the static IP address you specified. You can verify this using the `ip a` or `ip addr show` command.

#### Troubleshooting Your NetworkManager Configuration

Just like with Netplan, things might not always go perfectly when you apply the static IP configuration using `nmcli`. If you lose network connectivity or encounter other issues, here are some troubleshooting steps you can take:

1. **Verify Your `nmcli` Command**: Double-check the command you used to modify the connection. Ensure there are no typos in the connection name, IP address, subnet mask, gateway, or DNS server addresses. Even a small mistake can cause problems. You can use `nmcli connection show "your_connection_name"` to review the current settings of the connection.

2. **Check Your Network Interface Status**: Use `ip a` or `ip addr show` to see if your network interface has the static IP address you intended to set. If it doesn't, the `nmcli connection up` command might not have applied the changes correctly. Try running it again.

3. **Ping Your Localhost**: As with Netplan, try `ping 127.0.0.1` to ensure your basic network interface is working.

4. **Ping Your Gateway**: Try pinging your router's IP address (`ping your_gateway_ip`). If this fails, there might be an issue with the IP address or gateway you configured.

5. **Check DNS Resolution**: Try pinging a public IP address like `8.8.8.8`. If this works but you can't access websites by name (e.g., `ping google.com` fails), there's likely an issue with your DNS server configuration. Double-check the IP addresses you provided for the DNS servers.

6. **Review NetworkManager Logs**: You can check the NetworkManager service logs for any error messages using `journalctl -u NetworkManager`. This might provide more specific details about what went wrong during the connection activation.

7. **Try Restarting NetworkManager**: Sometimes, restarting the NetworkManager service itself can resolve issues. You can do this with:

    ```bash
    sudo systemctl restart NetworkManager
    ```

    After restarting, try bringing your connection up again: `nmcli connection up "your_connection_name"`.

8. Revert to DHCP: If you're still having trouble, you can easily switch back to DHCP to regain internet access and further troubleshoot. Use the following nmcli command:

    ```bash
    nmcli connection modify "your_connection_name" ipv4.method auto
    nmcli connection down "your_connection_name"
    nmcli connection up "your_connection_name"
    ```

    This will set the connection back to automatically obtaining an IP address.
