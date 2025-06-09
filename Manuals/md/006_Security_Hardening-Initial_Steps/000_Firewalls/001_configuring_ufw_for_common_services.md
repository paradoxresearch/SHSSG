# Configuring `ufw` for Common Services

## Contents

- [Configuring `ufw` for Common Services](#configuring-ufw-for-common-services)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Basic UFW Commands](#basic-ufw-commands)
    - [How to Check the Current UFW Status and Interpret the Output](#how-to-check-the-current-ufw-status-and-interpret-the-output)
  - [Allowing Connections for Common Services](#allowing-connections-for-common-services)
    - [How to Allow Connections on Specific Ports and Protocols](#how-to-allow-connections-on-specific-ports-and-protocols)
    - [How to Allow Connections from Specific IP Addresses or Subnets](#how-to-allow-connections-from-specific-ip-addresses-or-subnets)
  - [Denying Connections](#denying-connections)
    - [When Denying Specific Connections Might Be Necessary for Server Hardening](#when-denying-specific-connections-might-be-necessary-for-server-hardening)
  - [Managing UFW Rules](#managing-ufw-rules)
    - [How to Delete UFW Rules Using Their Line Number](#how-to-delete-ufw-rules-using-their-line-number)
  - [Logging with UFW](#logging-with-ufw)
    - [Different Logging Levels](#different-logging-levels)

## Introduction

In the context of server administration, security is paramount. One fundamental aspect of securing an Ubuntu Server is implementing a robust firewall. A **firewall** functions as a network security system that monitors and controls incoming and outgoing network traffic based on pre-defined security rules. Its primary purpose is to establish a protective barrier between your server and potentially malicious external entities, thereby safeguarding sensitive data and ensuring system integrity.

**UFW (Uncomplicated Firewall)** is a user-friendly, command-line interface designed to simplify the management of `iptables`, the underlying packet filtering framework in the Linux kernel. For a headless server environment, where graphical interfaces are absent, UFW provides an efficient and straightforward means to configure your server's firewall rules directly from the terminal. It abstracts much of the complexity of `iptables` into a more intuitive command set, making it an ideal tool for system administrators to quickly and effectively manage network access to their servers.

Think of UFW as a well-organized gatekeeper for your server. You define the rules about who is allowed to pass through the gate (access specific services or ports) and who is denied. This controlled access is essential for preventing unauthorized intrusion and mitigating various security threats.

## Basic UFW Commands

Think of these commands as the basic instructions you give to your server's gatekeeper.

Here are some of the most essential UFW commands:

- `ufw enable`: This command is like telling your gatekeeper to start actively checking everyone who wants to come in. It activates the UFW firewall.
- `ufw disable`: This is like telling your gatekeeper to take a break and let everyone pass freely. It deactivates the UFW firewall. Be very careful when using this command on a production server, as it leaves it unprotected!
- `ufw status`: This command asks your gatekeeper for a report on who is currently allowed or denied access. It shows you the current status of the firewall and the active rules.
- `ufw default allow incoming`: This sets the default policy for incoming traffic. Imagine it as the gatekeeper's general instruction: "By default, let everyone in unless I have specific instructions otherwise." This is generally not recommended for servers as it leaves them open to potential attacks.
- `ufw default deny incoming`: This is the more secure default policy for incoming traffic. It tells the gatekeeper: "By default, don't let anyone in unless I specifically say they can." You'll then add rules to allow specific services or connections.
- `ufw default allow outgoing`: This sets the default policy for traffic going out of your server. It's usually safe to allow all outgoing traffic by default, as your server needs to communicate with other systems.
- `ufw default deny outgoing`: This is a more restrictive policy for outgoing traffic, where you would only allow connections to specific destinations. This is less common for typical server setups but can be used in highly controlled environments.

Think of the "default" policies as the initial stance of your gatekeeper before you give them specific instructions for certain people or services. It's generally much safer to start with `ufw default deny incoming` and then explicitly allow the traffic your server needs.

### How to Check the Current UFW Status and Interpret the Output

Once you've enabled UFW or made changes to its rules, you'll want to know what the current configuration looks like. The command for this is, as we mentioned earlier:

```bash
ufw status
```

When you run this command, you'll typically see output that looks something like this:

```shell
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
Anywhere                   ALLOW       192.168.1.10
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
Anywhere (v6)              ALLOW       2001:db8::10
```

Let's break down what this output tells us:

- `Status: active`: This clearly indicates that the UFW firewall is currently enabled and actively filtering traffic. If it said `Status: inactive`, the firewall would not be doing anything.
- `To`: This column specifies the destination port and protocol on your server. For example, `22/tcp` refers to TCP traffic on port 22 (which is typically used for SSH).
- `Action`: This column shows what action UFW is taking for traffic matching the rule. `ALLOW` means that connections to the specified port and protocol from the specified source are permitted. `DENY` would mean they are blocked.
- `From`: This column indicates the source of the traffic that the rule applies to. `Anywhere` means that connections from any IP address are allowed. You might also see specific IP addresses or network ranges listed here.
- **(v6)**: These entries indicate rules that apply to IPv6 addresses.

So, in this example, the server is allowing incoming TCP connections on ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) from any IP address. It's also allowing all traffic from the specific IPv4 address `192.168.1.10` and the IPv6 address `2001:db8::10`.

Understanding the output of `ufw status` is crucial for verifying that your firewall rules are configured correctly and that your server is protected as intended.

## Allowing Connections for Common Services

One of the great things about UFW is that it understands common network services by name. Instead of having to remember the specific port numbers for these services, you can often just use their names.

For example:

- **SSH (Secure Shell)**: This is the primary way you'll remotely access your server via the command line. UFW knows that SSH typically uses port 22 with the TCP protocol. To allow SSH connections, you can simply use the command:

    ```bash
    sudo ufw allow ssh
    ```

- **HTTP (Hypertext Transfer Protocol)**: This is the standard protocol for serving web pages. It usually uses port 80 with the TCP protocol. To allow incoming HTTP requests, use:

    ```bash
    sudo ufw allow http
    ```

- **HTTPS (HTTP Secure)**: This is the secure version of HTTP, often used for websites that handle sensitive information. It typically uses port 443 with the TCP protocol. To allow incoming HTTPS requests, use:

    ```bash
    sudo ufw allow https
    ```

When you use these service names with `ufw allow`, UFW looks up the corresponding port and protocol in its predefined list and creates the necessary firewall rule. This makes it much easier to manage common services.

It's important to only allow the services that your server actually needs to be accessible from the network. For example, if your server is only used for running a web server, you would typically need to allow HTTP and HTTPS, but you might restrict SSH access to specific IP addresses for added security (which we'll discuss in the next substep of this section).

### How to Allow Connections on Specific Ports and Protocols

Sometimes, the service you need to allow doesn't have a predefined name in UFW, or it might be running on a non-standard port. In these cases, you need to specify the port number and the protocol (either TCP or UDP).

Here's how you do it:

```bash
sudo ufw allow <port>/<protocol>
```

Let's break this down:

- `<port>`: This is the numerical port number that the service uses. For example, the default port for a custom web application might be 8080.
- `<protocol>`: This specifies the network protocol. The most common ones you'll encounter are `tcp` and `udp`. Most services use TCP, but some, like DNS or certain game servers, might use UDP. You'll need to know which protocol the service you're allowing uses.

**Example**:

Let's say you have a web application running on port 8080 using the TCP protocol. To allow incoming connections to this application, you would use the command:

```bash
sudo ufw allow 8080/tcp
```

Similarly, if you had a game server running on port 27015 using the UDP protocol, you would use:

```bash
sudo ufw allow 27015/udp
```

It's important to know the port number and the protocol of the service you want to allow. This information is usually provided in the service's documentation or configuration files.

When would you need to use this method instead of allowing by service name? You'd use it when:

- The service doesn't have a predefined name in UFW.
- The service is running on a non-standard port (i.e., not the usual port associated with its name).

### How to Allow Connections from Specific IP Addresses or Subnets

Sometimes, you might want to restrict access to certain services to only specific IP addresses or ranges of IP addresses for enhanced security. For example, you might want to allow SSH access only from your home or office IP address. UFW allows you to do this.

Here's how:

- **Allowing from a specific IP address**:

    ```bash
    sudo ufw allow from <IP address> to any port <port>/<protocol>
    ```

    Replace `<IP address>` with the specific IP you want to allow, `<port>` with the port number, and `<protocol>` with either tcp or udp.

    **Example**: To allow SSH (port 22, TCP) connections only from the IP address `192.168.1.100`, you would use:

    ```bash
    sudo ufw allow from 192.168.1.100 to any port 22/tcp
    ```

- **Allowing from a specific network (subnet)**:
You can also allow traffic from an entire range of IP addresses using CIDR (Classless Inter-Domain Routing) notation.

    ```bash
    sudo ufw allow from <network>/<mask> to any port <port>/<protocol>
    ```

    Here, `<network>` is the base IP address of the network, and `<mask>` is the subnet mask in CIDR format (e.g., `/24`). A `/24` mask typically represents a range of 256 IP addresses.

    **Example**: To allow HTTP (port 80, TCP) connections from the entire `192.168.1.0` network (with a subnet mask of 255.255.255.0, which is `/24` in CIDR), you would use:

    ```bash
    sudo ufw allow from 192.168.1.0/24 to any port 80/tcp
    ```

This level of control is very important for security. By restricting access to essential services like SSH to only trusted IP addresses, you significantly reduce the risk of unauthorized access.

Why is this more secure than just allowing a service from "Anywhere"? Because it limits the potential attackers to only those who might be able to route traffic from the allowed IP addresses.

## Denying Connections

The `ufw deny` command works very similarly to the `ufw allow` command. You can deny connections based on service name, port and protocol, or source IP address/subnet.

Here are some examples:

- **Denying a service by name**: Let's say you had previously allowed a service like Telnet (which is generally insecure) for testing, and now you want to explicitly block it. You could use:

    ```bash
    sudo ufw deny telnet
    ```

    This would block any incoming connections to the standard Telnet port (usually port 23 TCP) from any IP address.

- **Denying traffic on a specific port and protocol**: If you know a specific port and protocol are being used for malicious activity, you can block them. For example, to block all UDP traffic on port 12345:

    ```bash
    sudo ufw deny 12345/udp
    ```

- **Denying traffic from a specific IP address**: If you notice suspicious activity coming from a particular IP address, you can block all traffic from it:

    ```bash
    sudo ufw deny from <suspicious IP address>
    ```

    For example:

    ```bash
    sudo ufw deny from 203.0.113.15
    ```

    You can also specify a port and protocol if you only want to block traffic from that IP on a specific service:

    ```bash
    sudo ufw deny from 203.0.113.15 to any port 22/tcp
    ```

    This would block SSH connections specifically from the IP address `203.0.113.15`.

- **Denying traffic from a specific subnet**: Similar to allowing, you can also deny traffic from an entire network range:

    ```bash
    sudo ufw deny from <malicious network>/<mask>
    ```

    For example, to block all traffic from the `10.0.0.0/8` network:

    ```bash
    sudo ufw deny from 10.0.0.0/8
    ```

**Important Note**: `deny` rules take precedence over `allow` rules. So, if you have a rule allowing traffic from a specific IP address on port 80, but you also have a rule denying all traffic from that same IP address, the deny rule will be enforced.

Why would you need to explicitly deny connections?

- **Security Hardening**: To block potentially insecure or unnecessary services.
- **Mitigating Attacks**: To block traffic from known malicious IP addresses or networks.
- **Granular Control**: To override broader `allow` rules for specific sources.

### When Denying Specific Connections Might Be Necessary for Server Hardening

Explicitly denying connections, beyond just not allowing them, can be a crucial part of hardening your server's security posture. Here are a few scenarios where using `ufw deny` is particularly important:

- **Blocking Known Malicious Actors**: If you identify specific IP addresses or entire networks that are consistently trying to attack your server (e.g., through log analysis or intrusion detection systems), you can use ufw deny from `<IP address>` or ufw deny from `<network>/<mask>` to block all communication from them. This proactively prevents them from even attempting to connect to any services on your server.

- **Disabling Insecure or Unnecessary Services**: Even if you haven't explicitly allowed a service, if it's running and listening on a port, there's a potential (though often small with default configurations) risk. Using ufw deny `<service name>` or ufw deny `<port>/<protocol>` ensures that these services are actively blocked at the firewall level, adding an extra layer of security. For example, you might want to explicitly deny Telnet or other outdated protocols that you know you'll never use.

- **Overriding Broader Allow Rules**: Imagine you've allowed connections from an entire internal network (`192.168.1.0/24`) to your web server on port 80. However, you know that a specific IP address within that network (`192.168.1.50`) has been compromised. Instead of completely blocking the entire internal network, you can create a specific `ufw deny from 192.168.1.50 to any port 80/tcp` rule. This will block the compromised host from accessing the web server while still allowing other hosts on the same network. Remember, `deny` rules take precedence.

- **Geoblocking (with caution)**: In some advanced scenarios, if you know that the vast majority of legitimate traffic to your server comes from a specific geographic region, you could potentially deny traffic from other parts of the world. However, this needs to be done with extreme caution as IP address geolocation isn't always perfectly accurate, and you might inadvertently block legitimate users. UFW itself doesn't have built-in geoblocking capabilities, but this illustrates the concept of denying based on origin.

In essence, `ufw deny` provides you with a powerful tool to actively block unwanted communication, adding a significant layer of defense to your server. It's about being proactive in identifying and blocking potential threats or unnecessary access points.

## Managing UFW Rules

As you add more rules to your firewall, it becomes important to be able to see them in an organized way and to make changes when necessary.

UFW provides a convenient way to list your rules with line numbers. This is particularly useful when you want to delete a specific rule. To see your rules with line numbers, you use the command:

```bash
sudo ufw status numbered
```

The output will look similar to the regular `ufw status`, but with an added column for the rule number:

```shell
Status: active

     To                         Action      From
[ 1] 22/tcp                     ALLOW       Anywhere
[ 2] 80/tcp                     ALLOW       Anywhere
[ 3] 443/tcp                    ALLOW       Anywhere
[ 4] Anywhere                   ALLOW       192.168.1.10
[ 5] 22/tcp (v6)                ALLOW       Anywhere (v6)
[ 6] 80/tcp (v6)                ALLOW       Anywhere (v6)
[ 7] 443/tcp (v6)               ALLOW       Anywhere (v6)
[ 8] Anywhere (v6)              ALLOW       2001:db8::10
```

Notice the `[ ]` brackets at the beginning of each rule line containing a number. This is the line number associated with that specific rule.

Why are these line numbers important? Because you use them to delete rules. Instead of having to remember the exact syntax of a rule you want to remove, you can simply refer to its line number.

### How to Delete UFW Rules Using Their Line Number

To delete a rule, you use the command:

```bash
sudo ufw delete <number>
```

You simply replace `<number>` with the line number of the rule you want to remove, as shown in the output of `ufw status numbered`.

**Example**:

Let's say you want to remove the rule that allows HTTP traffic (port 80/tcp) from anywhere, and when you ran `sudo ufw status numbered`, it showed up as line number 2:

```shell
Status: active

     To                         Action      From
[ 1] 22/tcp                     ALLOW       Anywhere
[ 2] 80/tcp                     ALLOW       Anywhere
[ 3] 443/tcp                    ALLOW       Anywhere
...
```

To delete this rule, you would use the command:

```bash
sudo ufw delete 2
```

UFW will then ask you to confirm if you really want to delete the rule:

```shell
Deleting:
 allow 80/tcp
Proceed with operation (y|n)?
```

Type `y` and press Enter to confirm the deletion. After the rule is deleted, if you run `sudo ufw status`, you should see that the rule for port 80 is no longer listed.

**Important Caution**: Be very careful when deleting rules, especially if you're working on a production server. Accidentally deleting a rule that allows access to a critical service (like SSH) can lock you out of your server. Always double-check the rule number before deleting it.

Also, be extremely cautious about deleting the **default incoming/outgoing policies** unless you fully understand the implications. These default policies are the foundation of your firewall's security stance. Removing or changing them incorrectly can have unintended and potentially severe security consequences.

## Logging with UFW

Think of UFW logging as keeping a record of who tried to come to your server's door and whether they were allowed in or turned away. This can be incredibly valuable for monitoring your server's security and troubleshooting connection issues.

UFW has its own logging mechanism that records firewall activity. You can control whether logging is enabled or disabled using the following commands:

- **Enable Logging**:

    ```bash
    sudo ufw logging on
    ```

    This command starts the UFW logging service. After enabling it, UFW will begin recording firewall-related events in your system logs.

- **Disable Logging**:

    ```bash
    sudo ufw logging off
    ```

    This command stops the UFW logging service. You might want to temporarily disable logging if you are performing a lot of network activity and don't need the logs at that moment, or if you're troubleshooting something specific. However, for continuous security monitoring, it's generally a good idea to keep logging enabled.

Why is logging important?

- **Security Analysis**: Logs can help you identify potential security threats, such as repeated failed connection attempts from a specific IP address, which might indicate a brute-force attack.
- **Troubleshooting**: If a legitimate user is having trouble connecting to a service on your server, the logs can provide clues about whether the firewall is blocking their connection and why.
- **Auditing**: Logs provide a record of network activity and firewall decisions, which can be useful for security audits and compliance requirements.

Once logging is enabled, the firewall logs are typically stored in the standard system log files, often located in `/var/log/ufw.log` or `/var/log/syslog` (depending on your system configuration). You can use standard Linux command-line tools like `cat`, `grep`, `less`, or `tail` to view and analyze these logs.

### Different Logging Levels

UFW offers different levels of logging, which control the verbosity of the information recorded in your logs. Choosing the right logging level is a balance between getting enough detail for security analysis and not overwhelming your logs with excessive information.

Here are the common logging levels in UFW:

- `off`: As we discussed, this disables logging entirely. You would typically only use this temporarily for specific troubleshooting or performance testing where log activity might interfere. Generally, you'll want logging to be enabled for security reasons.

- `low`: This level logs only blocked packets and dropped invalid packets. It's the least verbose level and is often sufficient for general security monitoring. You'll see records of attempts to connect to ports that are not open or packets that are malformed and discarded by the firewall. This level can help you identify basic attack attempts.

- `medium`: This level includes everything logged in `low` mode, plus allowed packets and new connections. This provides a more comprehensive view of the network traffic passing through your firewall. While it gives you more detail, it can also generate a larger volume of log data, especially on a busy server. This level can be useful for more detailed troubleshooting or when you need to understand which connections were successfully established.

- `high`: This is the most verbose logging level. It logs all packets, including those that are blocked, allowed, and dropped due to invalid state. This level provides the most detailed information about every network interaction. However, it can generate a significant amount of log data very quickly, potentially impacting disk space and system performance if not managed properly (e.g., through log rotation). You might use this level temporarily when investigating a specific security incident or network issue that requires very granular details.

**When to use each level**:

- `off`: Use sparingly, primarily for temporary troubleshooting or specific scenarios where logging is not needed.
- `low`: Suitable for general, ongoing security monitoring on most production servers. It provides essential information about blocked attempts without being overly verbose.
- `medium`: Useful for more in-depth troubleshooting of network connectivity issues or when you need to see both blocked and allowed connections. Be mindful of the increased log volume.
- `high`: Best used temporarily when actively investigating a security incident or a complex network problem where you need to see every single packet. Remember to switch back to a lower level afterward to avoid excessive log growth.

You can set the logging level using the command:

```bash
sudo ufw logging <level>
```

Replace `<level>` with `off`, `low`, `medium`, or `high`. For example, to set the logging level to medium, you would use:

```bash
sudo ufw logging medium
```

Understanding these different logging levels allows you to tailor the amount of information you collect to your specific needs and the resources of your server.
