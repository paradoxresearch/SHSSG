# Setting Up `firewalld`

## Contents

- [Setting Up `firewalld`](#setting-up-firewalld)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Key `firewalld` Concepts](#key-firewalld-concepts)
  - [Basic `firewalld` Management](#basic-firewalld-management)
    - [Changing the Default Zone](#changing-the-default-zone)
  - [Working with Zones](#working-with-zones)
    - [Assign Network Interfaces to Specific Zones](#assign-network-interfaces-to-specific-zones)
  - [Managing Services](#managing-services)
    - [Allowing Specific Services in a Zone](#allowing-specific-services-in-a-zone)
  - [Working with Ports and Protocols](#working-with-ports-and-protocols)
    - [TCP and UDP Protocols](#tcp-and-udp-protocols)
  - [Advanced Rules and Concepts (Optional)](#advanced-rules-and-concepts-optional)
    - [ICMP Blocking](#icmp-blocking)

## Introduction

In the context of server administration, security hardening is paramount. One of the foundational elements of this is controlling network access to your server. This is where a firewall comes into play. Think of your server as a critical business asset, like a secure data vault. This vault has entry points (network interfaces) that need to be carefully guarded to prevent unauthorized access and potential breaches.

`firewalld` is a robust and dynamically managed firewall service available on many Linux distributions. It provides a structured and more manageable approach to configuring your server's network security policies compared to directly manipulating lower-level tools such as `iptables`. While `iptables` operates directly on the Linux kernel's network filtering rules, `firewalld` offers an abstraction layer with concepts like zones and services, making rule management more intuitive and less error-prone for administrators.

The importance of a properly configured firewall cannot be overstated. Without it, your server is exposed to a multitude of network-based threats. For instance, malicious actors could attempt to gain unauthorized access to sensitive data, disrupt services through denial-of-service attacks, or exploit vulnerabilities in running applications. `firewalld` allows you to define precise rules that dictate what types of network traffic are permitted to enter or leave your server, significantly reducing its attack surface and enhancing its overall security posture.

### Key `firewalld` Concepts

Understanding these concepts is fundamental to effectively managing your firewall. The three main concepts we'll cover are:

- **Zones**: Think of zones as different security levels or trust levels for your network interfaces. For example, your home network might be considered a trusted zone, while a public Wi-Fi network would be an untrusted zone. `firewalld` comes with several pre-defined zones, each with a default set of rules. We'll explore these in more detail later. For now, just understand that a zone dictates what traffic is allowed.

- **Services**: These are pre-defined rules for common network applications or protocols. Instead of manually specifying ports and protocols for services like SSH (for remote access) or HTTP (for web servers), `firewalld` provides these as easy-to-use service names. This simplifies rule creation and makes it more readable.

- **Rules**: These are the specific instructions that determine whether network traffic is allowed or blocked. Rules can be based on source/destination IP addresses, ports, protocols, and the zone the network interface belongs to. Services are essentially collections of pre-defined rules.

Let's relate this to a real-world scenario: Imagine you're setting up a web server on your Ubuntu Server. You want it to be accessible from the internet but also want to secure it. You might assign the network interface connected to the internet to the `public` **zone**. Then, you would **allow the** `http` **and** `https` **services** in this zone, which automatically opens the necessary ports (80 and 443 respectively) for web traffic. You would likely want to block all other incoming traffic by default in the `public` zone to protect your server.

## Basic `firewalld` Management

To start using `firewalld`, you'll need to know a few essential commands. We'll cover how to check its status and see what its current configuration looks like.

Here are a couple of fundamental commands:

1. **Checking the status of** `firewalld`: You can use the `systemctl` command to check if the `firewalld` service is running on your server. Open your terminal and type:

    ```bash
    systemctl status firewalld
    ```

    This command will tell you if the service is active (running), inactive (stopped), or in a failed state. As a system administrator, you'll often use `systemctl` to manage various system services.

2. **Checking the state of** `firewalld`: `firewalld` also has its own command-line tool, `firewall-cmd`, which you'll use extensively. To check the overall state of the firewall, use:

    ```bash
    firewall-cmd --state
    ```

    This command should simply output "running" if `firewalld` is active.

3. **Listing active zones**: Remember those zones we talked about? To see which zones are currently active and which network interfaces are associated with them, use:

    ```bash
    firewall-cmd --get-active-zones
    ```

    This will show you a list of zones and the network interfaces (like `eth0` or `ens3`) that are currently assigned to each zone. This is useful for understanding how your network traffic is being categorized.

4. **Getting the default zone**: `firewalld` has a default zone that new network interfaces will be assigned to automatically unless you specify otherwise. To find out what the default zone is, use:

    ```bash
    firewall-cmd --get-default-zone
    ```

    Understanding the default zone is important because it dictates the initial security posture of any new network connection to your server.

Think of these commands as your basic diagnostic tools for understanding how `firewalld` is currently configured and running on your server.

### Changing the Default Zone

The default zone in `firewalld` is like the standard security setting for any new network interface that gets added to your server. When a network interface starts up, if you haven't explicitly assigned it to a specific zone, it will automatically be placed in the default zone.

To see what the current default zone is, as we learned, you use:

```bash
firewall-cmd --get-default-zone
```

Let's say the output is `public`. This means that by default, any new network interface will have the security rules associated with the `public` zone applied to it. The `public` zone is generally intended for untrusted networks, like the internet, and has a more restrictive set of allowed traffic by default.

To change the default zone, you use the following command:

```bash
firewall-cmd --set-default-zone=<zone>
```

Replace `<zone>` with the name of the zone you want to set as the new default. For example, if you have a server that primarily interacts with a trusted internal network, you might want to set the default zone to `private`:

```bash
firewall-cmd --set-default-zone=private
```

**Important Considerations**:

- **Implications**: Changing the default zone affects any new network interfaces that are added to your system after you run this command. It does not change the zone of any interfaces that are already active. You'll need to change those explicitly, which we'll cover in the next step.
- **Common Zones**: You'll often encounter zones like `public`, `private`, and `dmz` (Demilitarized Zone - often used for servers exposed to the internet but isolated from the internal network). We'll discuss these zones in more detail in the next step as well.
- **Persistence**: Like many `firewall-cmd` operations, this change is usually runtime only. To make it permanent across reboots, you'll typically need to add the `--permanent` flag to the command. However, for setting the default zone, the change usually persists.

Think of setting the default zone as choosing the standard security posture for any new network connection to your server. Choosing the right default zone depends on the primary role and network environment of your server.

## Working with Zones

As we touched upon earlier, zones are a core concept in `firewalld`. They represent different levels of trust in the networks your server is connected to. `firewalld` comes with several pre-defined zones, each with its own default rules regarding what traffic is allowed. Understanding these zones is crucial for applying the right security policies to your network interfaces.

Here are some of the most common pre-defined zones in firewalld:

- `public`: This zone is intended for untrusted public networks, like the internet. By default, it's very restrictive, typically only allowing explicitly defined incoming connections. This is the zone you'd usually assign to your server's internet-facing network interface.

- `private`: This zone is for trusted private networks, such as your home or internal office network. It generally allows more incoming connections from other devices on the same network.

- `dmz` **(Demilitarized Zone)**: This is used for servers that are exposed to the internet but need to be isolated from your internal network. It typically allows only specific incoming services.

- `trusted`: This zone allows all network traffic. You would typically only use this for highly trusted networks, and you should be very cautious when assigning an interface to this zone.

- `block`: Any incoming connections are rejected with an `icmp-host-prohibited` message for IPv4 and `icmp6-adm-prohibited` for IPv6. Only outgoing connections are allowed.

- `drop`: All incoming connections are silently dropped without any response. Only outgoing connections are allowed. This is the most restrictive zone.

Think of these zones like security profiles you can apply to your network interfaces. For example, if your server has one network card connected to the internet and another connected to your internal network, you would likely assign the internet-facing interface to the `public` zone and the internal interface to the `private` zone. This allows different security rules to be applied to each network based on its trust level.

### Assign Network Interfaces to Specific Zones

Your server might have multiple network interfaces, each connecting to a different network. For example, you might have `eth0` connected to the internet and `eth1` connected to your internal network. It's crucial to assign the appropriate zone to each interface to enforce the correct security policies. You wouldn't want your internal network interface to have the same wide-open rules as an internet-facing interface!

To assign a network interface to a specific zone, you use the `firewall-cmd` command with the `--change-interface` option. Here's the basic syntax:

```bash
firewall-cmd --zone=<zone> --change-interface=<interface>
```

Replace `<zone>` with the name of the zone you want to assign (e.g., `public`, `private`) and `<interface>` with the name of the network interface (e.g., `eth0`, `ens3`).

**Important**: This command, by default, only makes the change in the current runtime. This means that if you reboot your server, the changes will be lost. To make the assignment permanent across reboots, you need to add the `--permanent` flag:

```bash
firewall-cmd --zone=<zone> --change-interface=<interface> --permanent
```

After making permanent changes, you need to reload the firewalld configuration for them to take effect in the running firewall:

```bash
firewall-cmd --reload
```

Let's look at an example. Suppose you want to assign your internet-facing network interface, `eth0`, to the `public` zone permanently. You would use these commands:

```bash
firewall-cmd --zone=public --change-interface=eth0 --permanent
firewall-cmd --reload
```

Similarly, if you wanted to assign your internal network interface, `eth1`, to the `private` zone permanently:

```bash
firewall-cmd --zone=private --change-interface=eth1 --permanent
firewall-cmd --reload
```

Think of this as putting different security guards at different doors of your server, depending on who you expect to be coming through that door. Assigning the correct zone to each interface is a fundamental step in network segmentation, which is a key security practice to limit the impact of a potential security breach.

## Managing Services

One of the really convenient features of `firewalld` is its concept of pre-defined **services**. These are essentially bundles of rules that define the necessary ports and protocols for common network applications. Instead of having to remember and manually configure the specific port and protocol for, say, an SSH server, `firewalld` provides a ready-made "ssh" service.

To see a list of all the `pre-defined` services that firewalld knows about, you can use the following command:

```bash
firewall-cmd --get-services
```

This will give you a long list of service names. Some common ones you might recognize include:

- `ssh`: For secure remote access using SSH (usually on TCP port 22).
- `http`: For standard web traffic (usually on TCP port 80).
- `https`: For secure web traffic using SSL/TLS (usually on TCP port 443).
- `smtp`: For sending email (usually on TCP port 25).
- `dns`: For DNS name resolution (usually on UDP port 53).

Using these services simplifies firewall configuration. Instead of remembering that SSH uses TCP port 22, you can just allow the `ssh` service in a specific zone. `firewalld` will then automatically handle the underlying port and protocol rules for you in that zone.

Think of these services as shortcuts for common applications. Instead of writing out the full address and room number every time you want to allow someone into your server's "room" (port), you can just say "let in anyone who's here for the 'SSH service'."

### Allowing Specific Services in a Zone

To allow a specific service in a particular zone, you use the `firewall-cmd` command with the `--add-service` option. Here's the syntax:

```bash
firewall-cmd --zone=<zone> --add-service=<service>
```

Replace `<zone>` with the name of the zone where you want to allow the service (e.g., `public`, `private`) and `<service>` with the name of the service you want to enable (e.g., `ssh`, `http`).

Just like with assigning interfaces to zones, this command only applies to the current runtime by default. To make the rule permanent across reboots, you need to add the `--permanent` flag:

```bash
firewall-cmd --zone=<zone> --add-service=<service> --permanent
```

And again, after making permanent changes, you need to reload the `firewalld` configuration:

```bash
firewall-cmd --reload
```

Let's consider our web server example again. Your server's internet-facing interface (`eth0`) is in the `public` zone. To allow web traffic (both standard HTTP on port 80 and secure HTTPS on port 443), you would enable the `http` and `https` services in the public zone like this:

```code snippet
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --reload
```

Similarly, if you need to allow SSH access to your server (which is crucial for remote administration), you would typically allow the `ssh` service in the appropriate zone. If you're accessing it from your trusted internal network (which might be assigned to the `private` zone), you would do:

```bash
firewall-cmd --zone=private --add-service=ssh --permanent
firewall-cmd --reload
```

**Security Implications**: It's crucial to only enable the services that are absolutely necessary for your server's function. Allowing unnecessary services opens up potential attack vectors. For example, if your server doesn't host email, you should not enable the `smtp` service. This principle of only allowing what is needed is a fundamental aspect of security hardening.

## Working with Ports and Protocols

Sometimes, the pre-defined services in `firewalld` might not cover all your needs. You might have a custom application running on a specific port or need to allow a protocol that isn't associated with a standard service. In these cases, you can directly specify the port and protocol you want to allow in a particular zone.

To allow traffic on a specific port and protocol, you use the `--add-port` option with the `firewall-cmd` command. The syntax is as follows:

```bash
firewall-cmd --zone=<zone> --add-port=<port>/<protocol>
```

Replace `<zone>` with the zone where you want to allow the traffic, `<port>` with the port number, and `<protocol>` with either `tcp` or `udp`.

Again, remember to add the `--permanent` flag to make the rule persistent across reboots and then reload `firewalld`:

```bash
firewall-cmd --zone=<zone> --add-port=<port>/<protocol> --permanent
firewall-cmd --reload
```

Here are a few common examples:

- To allow standard HTTP traffic (port 80) using the TCP protocol in the `public` zone:

    ```bash
    firewall-cmd --zone=public --add-port=80/tcp --permanent
    firewall-cmd --reload
    ```

- To allow secure HTTPS traffic (port 443) using the TCP protocol in the `public` zone:

    ```bash
    firewall-cmd --zone=public --add-port=443/tcp --permanent
    firewall-cmd --reload
    ```

- To allow a custom application running on UDP port 12345 in the `private` zone:

    ```bash
    firewall-cmd --zone=private --add-port=12345/udp --permanent
    firewall-cmd --reload
    ```

Think of this as opening a specific numbered door (`port`) and specifying who is allowed to use that door (`protocol` - either the "TCP people" or the "UDP people"). This gives you very granular control over the network traffic allowed to your server.

### TCP and UDP Protocols

Understanding the difference between TCP and UDP is important when configuring firewall rules at the port level. They are the two most common transport layer protocols used on the internet.

Think of them like two different ways of sending packages:

- **TCP (Transmission Control Protocol)**: Imagine sending a registered letter. With TCP:
  - A connection is established between the sender and receiver before any data is sent (like saying "Hello, are you ready to receive?").
  - Data is broken down into packets, and each packet is numbered.
  - The receiver acknowledges the receipt of each packet.
  - If a packet is lost or arrives out of order, it is retransmitted and reassembled correctly.
  - This makes TCP reliable and ensures that all data arrives in the correct order.
  - Common uses: Web browsing (HTTP/HTTPS), email (SMTP, POP3, IMAP), file transfer (FTP, SFTP), and secure shell (SSH). These applications require reliable data transmission.

- **UDP (User Datagram Protocol)**: Imagine sending postcards. With UDP:
  - No connection is established beforehand.
  - Data is sent in packets (datagrams) without numbering or guarantees of delivery.
  - The sender doesn't know if the packets arrived, and the receiver doesn't acknowledge them.
  - If packets are lost or arrive out of order, there's no mechanism for retransmission or reordering at the UDP level.
  - This makes UDP faster and more efficient for applications where occasional data loss is acceptable or where low latency is critical.
  - **Common uses**: Online gaming, streaming video and audio, Voice over IP (VoIP), and DNS lookups. These applications often prioritize speed and real-time data over guaranteed delivery of every single packet.

When configuring firewall rules, you need to know which protocol your application or service uses. For example, web servers primarily use TCP on ports 80 and 443 because reliable delivery of web pages is essential. DNS, on the other hand, often uses UDP on port 53 because quick lookups are important, and lost packets can be easily retransmitted by the application if needed. SSH uses TCP on port 22 because a reliable connection is crucial for secure remote access.

## Advanced Rules and Concepts (Optional)

So far, we've been allowing traffic based on zones, services, and ports. `firewalld` also allows you to create much more specific rules using what are called "rich rules." These rules let you define criteria based on source or destination IP addresses, network ranges, MAC addresses, and more.

Think of basic rules as saying "anyone can come to this door if they are here for the 'web service'." Rich rules are like saying "only people with this specific ID card (IP address) can come to this door, and only if they are here for the 'web service' during these specific hours."

Here's a simplified example of how you might add a rich rule to allow TCP traffic on port 80 only from the IP address `192.168.1.100` in the `public` zone (remember to use `--permanent` and `--reload` to make it persistent):

```bash
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port protocol=tcp port=80 accept' --permanent
firewall-cmd --reload
```

This rule is much more specific than simply allowing the `http` service in the `public` zone. Rich rules provide a powerful way to fine-tune your firewall policies for very specific security requirements.

While we won't go into all the details of rich rule syntax right now, it's good to be aware that this capability exists for more advanced scenarios where you need very granular control over network access based on various criteria.

### ICMP Blocking

**ICMP (Internet Control Message Protocol)** is used for various network diagnostic and control purposes. Common ICMP messages include "ping" requests (echo requests) and responses (echo replies), as well as error messages like "destination unreachable."

While ICMP can be useful for network troubleshooting, allowing all ICMP traffic can sometimes pose a security risk. For example, attackers might use ICMP echo requests (pings) to discover active hosts on your network (a process called "ping sweeping"). They might also exploit other types of ICMP messages for denial-of-service attacks or to gather information about your network infrastructure.

Because of these potential risks, some system administrators choose to block certain types of ICMP traffic on their servers, especially on internet-facing interfaces. However, it's important to be cautious when blocking ICMP entirely, as some legitimate network functions rely on it. For instance, Path MTU Discovery, a process that determines the optimal packet size for a network path, uses ICMP messages. Blocking necessary ICMP types can sometimes lead to connectivity issues.

**How to block ICMP using** `firewalld`:

You can use rich rules to block specific ICMP types or all ICMP traffic in a particular zone. For example, to block all incoming ICMP echo requests (pings) in the `public` zone permanently, you could use a rich rule like this:

```bash
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" protocol=icmp type=echo-request drop' --permanent
firewall-cmd --reload
```

To block all incoming ICMP traffic in the `public` zone, you could use:

```bash
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" protocol=icmp drop' --permanent
firewall-cmd --reload
```

**Important Considerations**:

- **Think carefully before blocking ICMP**. Ensure that blocking specific types won't negatively impact legitimate network communication.
- **Monitor your network**. If you do block ICMP, be aware that you might lose some basic network diagnostic capabilities.

This is just a brief introduction to the concept of ICMP blocking. It's a more advanced topic, and whether or not you choose to implement it depends on your specific security requirements and network environment.
