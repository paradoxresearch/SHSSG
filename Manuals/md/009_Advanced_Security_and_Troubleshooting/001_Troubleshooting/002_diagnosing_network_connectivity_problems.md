# Diagnosing Network Connectivity Problems

## Contents

- [Diagnosing Network Connectivity Problems](#diagnosing-network-connectivity-problems)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Network Basics](#understanding-network-basics)
  - [Initial Checks](#initial-checks)
  - [Verifying IP Configuration](#verifying-ip-configuration)
    - [Use `ip -a` and `ip -r` to Verify IP Address, Netmask, and Default Gateway Settings](#use-ip--a-and-ip--r-to-verify-ip-address-netmask-and-default-gateway-settings)
    - [Check DNS Server Configuration](#check-dns-server-configuration)
  - [Testing Reachability to Local Network Devices](#testing-reachability-to-local-network-devices)
    - [Using `ping` to Test Local Connectivity](#using-ping-to-test-local-connectivity)
      - [Success: "Reply from..." or "bytes from..."](#success-reply-from-or-bytes-from)
      - [Failure: "Destination Host Unreachable" or "Request timed out"](#failure-destination-host-unreachable-or-request-timed-out)
    - [View the ARP Cache](#view-the-arp-cache)
      - [Using `arp -a` or `ip neigh`](#using-arp--a-or-ip-neigh)
      - [Interpreting the ARP Cache Output](#interpreting-the-arp-cache-output)
      - [How it helps troubleshoot](#how-it-helps-troubleshoot)
  - [Testing Reachability to External Networks and Services](#testing-reachability-to-external-networks-and-services)
    - [Using `ping` to Check Connectivity to External Hosts](#using-ping-to-check-connectivity-to-external-hosts)
      - [Interpreting Output and Potential Issues](#interpreting-output-and-potential-issues)
    - [Identify where Break Downs Occur with `traceroute` or `mtr`](#identify-where-break-downs-occur-with-traceroute-or-mtr)
      - [The traceroute Command](#the-traceroute-command)
        - [Interpreting `traceroute` Output](#interpreting-traceroute-output)
      - [The mtr Command (My Traceroute)](#the-mtr-command-my-traceroute)
    - [Using `dig` and `nslookup` to Diagnose DNS Resolution Problems](#using-dig-and-nslookup-to-diagnose-dns-resolution-problems)
      - [The `dig` Command (Domain Information Groper)](#the-dig-command-domain-information-groper)
      - [The `nslookup` Command (Name Server Lookup)](#the-nslookup-command-name-server-lookup)
  - [Checking Firewall Rules](#checking-firewall-rules)
    - [Inspecting `iptables` Rules](#inspecting-iptables-rules)
    - [Inspecting `firewalld` Rules](#inspecting-firewalld-rules)
    - [Inspecting `ufw` Rules](#inspecting-ufw-rules)
  - [Analyzing Network Traffic](#analyzing-network-traffic)
    - [Capture and Analyze Packets with `tcpdump`](#capture-and-analyze-packets-with-tcpdump)
      - [What to Look For (and Why it's Advanced)](#what-to-look-for-and-why-its-advanced)

## Introduction

### Understanding Network Basics

Imagine your Linux server is like a specific desk in a very large office building. For this desk to communicate, it needs some essential pieces of information:

- **IP Address (Your Desk's Unique Number)**: Every desk in this building needs a unique number so people know exactly where to send a memo or where a visitor should go. Your server's IP address (e.g., `192.168.1.100`) is just like that unique number for your server on a computer network. It's how other computers find it and send information to it.

- **Subnet Mask (Your Section of the Office)**: This is like knowing which "section" or "department" your desk belongs to. If you want to talk to someone in your same section, you just walk over. If they're in a different section, you send the memo to the mailroom. The subnet mask (`255.255.255.0`) helps your server figure out if another computer is in its immediate "section" of the network or if it needs to send the information "outside its section."

- **Default Gateway (The Mailroom/Main Exit)**: If you need to send a memo to someone in a different section of the building, or even to a different building altogether, you send it to the mailroom. The mailroom knows how to get it where it needs to go. For your server, the default gateway (e.g., `192.168.1.1`) is like that mailroom or main exit. If your server wants to talk to a computer outside its local network (like a website on the internet), it sends that data to the default gateway, which then handles forwarding it.

- **DNS Server (The Office Directory)**: What if you want to send a memo to "Mr. Smith in Accounting," but you don't know his desk number? You'd look him up in the office directory, right? A DNS server is exactly that for computers. It translates human-friendly names like `google.com` (which is much easier for us to remember) into the computer's unique IP address (like `142.250.190.46`). Without a DNS server, your server would struggle to find websites or online services by their names.

Think of it this way:

- **IP Address**: Your unique identifier.
- **Subnet Mask**: Defines your local group.
- **Default Gateway**: Your path to the outside world.
- **DNS Server**: Your internet phone book for names.

## Initial Checks

When you're troubleshooting network problems on a Linux server, the very first thing you want to know is if the network adapter itself is even turned on and connected. Think of it like checking if your desk lamp is plugged in and switched on before you complain it's not giving light.

In Linux, we use commands like `ip a` (short for `ip address`) or the older, but still common, `ifconfig` to check this. These commands show you information about your server's network interfaces. An "interface" is basically your server's network card, the piece of hardware that allows it to connect to the network.

Let's look at what you might see:

1. `ip a` **(Recommended for modern Linux systems)**:

    When you type `ip a` and press `Enter`, you'll see output that looks something like this:

    ```shell
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
           valid_lft 86242sec preferred_lft 86242sec
        inet6 fe80::21a:2bff:fe3c:4d5e/64 scope link
           valid_lft forever preferred_lft forever
    ```

    - You'll typically see lo (loopback interface, for internal communication) and one or more ethX (e.g., eth0, eth1) or enpXsY interfaces (these are your actual network cards).
    - **Key things to look for**:
        - The word `UP` in the `<...>` brackets: This indicates the interface is administratively enabled.
        - The word `LOWER_UP`: This indicates that the physical link (like the network cable) is connected and active.
        - `state UP`: This explicitly confirms the interface is active and ready to transmit.

    If you don't see `UP` or `LOWER_UP`, it could mean the network cable is unplugged, or the interface has been disabled.

2. `ifconfig` **(Older but still common)**:

    The output looks similar:

    ```shell
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
            inet6 fe80::21a:2bff:fe3c:4d5e  prefixlen 64  scopeid 0x20<link>
            ether 00:1a:2b:3c:4d:5e  txqueuelen 1000  (Ethernet)
            RX packets 12345  bytes 1234567 (1.2 MB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 9876  bytes 987654 (987.6 KB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            loop  txqueuelen 1000  (Local Loopback)
    ```

    - Again, look for the `UP` and `RUNNING` flags. If `UP` is missing, the interface might be disabled. If `RUNNING` is missing, the physical link might be down.
    - Also, check "RX errors" and "TX errors." While some errors are normal, consistently high numbers here could indicate a faulty cable or network card.

So, in summary, the first step is to visually check if your server's network connection is up and active using `ip a` or `ifconfig`.

## Verifying IP Configuration

### Use `ip -a` and `ip -r` to Verify IP Address, Netmask, and Default Gateway Settings

Remember how we used `ip a` to see if the interface was "UP"? We can also use it to confirm the IP address, subnet mask, and other details.

Let's revisit the `ip a` output, focusing on the inet line for your active network interface (like `eth0` or `enp0s3`):

```shell
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86242sec preferred_lft 86242sec
```

- **IP Address**: Here, `192.168.1.100` is your server's IP address. This is its unique identifier on the network.
- **Subnet Mask (CIDR notation)**: The `/24` after the IP address is crucial. This is called CIDR (Classless Inter-Domain Routing) notation and it represents the subnet mask. A `/24` is equivalent to a subnet mask of `255.255.255.0`. This tells your server which part of the IP address identifies the network and which part identifies the specific host on that network.
  - **Common Misconfiguration**: An incorrect IP address or subnet mask means your server might think it's on a different network than it actually is, preventing it from communicating.

Next, let's look at the Default Gateway using the `ip r` (short for `ip route`) command. This command displays your server's routing table, which is like its internal map for sending traffic.

When you type `ip r` and press `Enter`, you'll likely see something similar to this:

```shell
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100 metric 100
```

- The most important line here for troubleshooting is the one starting with `default via`.
- **Default Gateway**: In this example, `192.168.1.1` is your default gateway. This is the "mailroom" IP address that your server sends all traffic to if the destination is not on its local network.
  - **Common Misconfiguration**: If the default gateway is missing, incorrect, or unreachable, your server won't be able to communicate with devices outside its local network (like the internet). This is a very common cause of "no internet" issues.

**In summary**:

- Use `ip a` to check the IP address and subnet mask (the `/XX` part).
- Use `ip r` to check the default gateway (the `default via` address).

Think of it like this: Your IP address is your house number, the subnet mask is the size of your block, and the default gateway is the exit road from your block. If any of these are wrong, you can't send or receive mail correctly!

### Check DNS Server Configuration

Even if your server has a correct IP address and can reach its default gateway, it might still struggle to access websites like `google.com` if it doesn't know how to translate that name into an IP address. That's where the DNS server comes in.

On Linux, the primary file that tells your system which DNS servers to use is `/etc/resolv.conf`. You can view the contents of this file using the `cat` command, which simply prints the file's content to your screen.

Type: `cat /etc/resolv.conf`

You might see output like this:

```shell
# Generated by NetworkManager
search example.com
nameserver 8.8.8.8
nameserver 8.8.4.4
```

Here's what to look for:

- **nameserver lines**: These lines list the IP addresses of the DNS servers your server will use. In this example, `8.8.8.8` and `8.8.4.4` are Google's public DNS servers. You might see IP addresses provided by your internet service provider (ISP) or your local network's DNS server.
- **Importance of Correct DNS Settings**: If these nameserver entries are incorrect, missing, or point to DNS servers that are unreachable or not functioning, your server will not be able to resolve domain names. This means you won't be able to `ping google.com` (you'd need to ping an IP address directly) or browse the web by name, even if your network connection is otherwise fine. It's like having a phone but no directory to look up numbers!
- `search` **line (Optional)**: The search line specifies domain suffixes that your system will try to append to hostnames when resolving them. For instance, if you type `ping myinternalserver` and you have `search mycompany.local` in `resolv.conf`, your system might automatically try to resolve `myinternalserver.mycompany.local`.

**Common Misconfiguration**: A common issue is having no `nameserver` entries, or entries that point to non-existent or blocked DNS servers. This will almost certainly prevent your server from accessing internet resources by name.

## Testing Reachability to Local Network Devices

### Using `ping` to Test Local Connectivity

The `ping` command is one of the most fundamental and widely used network troubleshooting tools. Think of it like shouting "Hello!" across the room and waiting to hear "Hello!" back. It sends small network packets to a target IP address or hostname and waits for a reply.

First, you'll want to test if your server can reach devices on its local network. The most important device to `ping` first is your **default gateway**. Remember, that's your "mailroom" or "main exit" that routes all traffic going outside your local network.

You identified your default gateway using `ip r` earlier. Let's assume it was `192.168.1.1`. To ping it, you would type:

```Bash
ping 192.168.1.1
```

Press `Ctrl+C` to stop the ping command after a few seconds.

**Interpreting** `ping` **Output**

Here's what you might see and what it means:

#### Success: "Reply from..." or "bytes from..."

```shell
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.5 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.4 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.4 ms
^C
--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.435/0.463/0.505/0.030 ms
```

- `64 bytes from 192.168.1.1: icmp_seq=X ttl=Y time=Z ms`: This means your server sent a packet, and the gateway responded successfully!
  - `icmp_seq`: The sequence number of the packet.
  - `ttl` (Time To Live): This indicates how many "hops" the packet can make before being discarded. Don't worry too much about the exact number for now, just know it decreases with each hop.
  - `time`: The time it took for the packet to travel to the destination and back (latency). Lower is better!
- `0% packet loss`: This is ideal. It means every packet you sent was received.
- **What it means**: If you can ping your default gateway successfully, it's a very good sign that your server's network card is working, its IP address is correctly configured, and its connection to the local network is sound.

#### Failure: "Destination Host Unreachable" or "Request timed out"

```Bash
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
From 192.168.1.100 icmp_seq=1 Destination Host Unreachable
From 192.168.1.100 icmp_seq=2 Destination Host Unreachable
^C
--- 192.168.1.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2004ms
```

- `Destination Host Unreachable`: This typically means your server tried to send the packet, but it couldn't find a path to the destination on its local network. This often points back to:
  - An incorrect IP address on your server.
  - An incorrect subnet mask.
  - The target device (your gateway) being offline or having a problem.
- `Request timed out`: This means your server sent the packet, but it never received a reply within a certain time limit. This could indicate:
  - The target device is offline.
  - A firewall (either on your server or the target) is blocking the `ping` request.
  - A faulty cable or network device between your server and the target.
- `100% packet loss`: This means none of your packets reached the destination and returned. This is a clear sign of a connectivity problem.

**Troubleshooting tip**: If you can't ping your default gateway, you need to go back and double-check your server's IP address, subnet mask, and routing table (`ip a` and `ip r`). Also, physically check the network cable!

### View the ARP Cache

Think of the "office building" analogy again. If you know Mr. Smith works in Accounting (his IP address), but you need to send him a physical memo, you also need to know his exact desk number or "physical address" within that section. In networking, this "physical address" is called a MAC address (Media Access Control address).

**ARP (Address Resolution Protocol)** is the mechanism computers use to find the MAC address of another device on the same local network when they only know its IP address. Your server keeps a temporary record of these IP-to-MAC address mappings in something called the **ARP cache**.

If your server can't communicate with a device on the local network, even if it has the correct IP address for it, sometimes the issue is that it can't resolve the MAC address. This is where checking the ARP cache comes in handy.

#### Using `arp -a` or `ip neigh`

You can inspect your server's ARP cache using one of these commands:

  1. arp -a (Older but common):

        ```Bash
        arp -a
        ? (192.168.1.1) at 00:11:22:33:44:55 [ether] on eth0
        ? (192.168.1.10) at 66:77:88:99:AA:BB [ether] on eth0
        ```

  2. `ip neigh` **(Recommended for modern Linux systems, short for** `ip neighbour`**)**:

        ```Bash
        ip neigh
        192.168.1.1 dev eth0 lladdr 00:11:22:33:44:55 REACHABLE
        192.168.1.10 dev eth0 lladdr 66:77:88:99:AA:BB STALE
        ```

#### Interpreting the ARP Cache Output

- **IP Address**: The first column shows the IP address of the device.
- **MAC Address (lladdr)**: This is the physical address (e.g., `00:11:22:33:44:55`).
- `dev eth0` **(or your interface name)**: Shows which network interface the entry is associated with.
- **Status (**`ip neigh` **only)**:
  - `REACHABLE`: The entry is valid and the device has been recently active. This is ideal.
  - `STALE`: The entry is still valid, but the system hasn't recently verified its reachability. It will try to re-verify if traffic is sent to it.
  - `FAILED`: This is a problem! It means the ARP resolution for that IP address failed. This can happen if the device is offline, if there's a misconfigured switch, or if your server can't physically reach the device at the MAC layer.

#### How it helps troubleshoot

If you can `ping` a device (meaning you're sending packets), but it's still not working as expected, and you see `FAILED` or a missing entry for its IP address in the ARP cache, it might indicate a low-level problem in the local network segment. This could be:

- The target device is truly offline.
- The target device's network card is faulty.
- There's an issue with the switch port it's connected to.
- The IP address is configured correctly, but there's a conflict or problem at the physical (Layer 2) level.

By checking the ARP cache, you can confirm if your server has successfully "learned" the physical address of local devices it's trying to communicate with.

## Testing Reachability to External Networks and Services

### Using `ping` to Check Connectivity to External Hosts

Being able to ping your default gateway confirms your local connection, but it doesn't guarantee you can reach the internet. You might have a working connection to your "mailroom," but the mailroom itself might be disconnected from the rest of the world!

**Using** `ping` **for External Hosts**

A common practice is to ping a well-known, reliable IP address on the internet, such as Google's public DNS server (`8.8.8.8`) or Cloudflare's public DNS server (`1.1.1.1`). These are usually very stable and widely reachable.

To test, you would type:

```Bash
ping 8.8.8.8
```

And again, press `Ctrl+C` to stop it.

#### Interpreting Output and Potential Issues

- **Success (Similar to local ping)**: If you see replies from `8.8.8.8` with `0% packet loss`, it's a great sign! It means your server can successfully send traffic out through your default gateway and receive replies from the internet. This confirms your server's IP configuration (including gateway) and the internet connection itself are generally working.

    ```shell
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=15.2 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=15.1 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1001ms
    ```

- **Failure (**`Request timed out` **or** `Destination Host Unreachable`**)**: If ping `8.8.8.8` fails, and you could successfully `ping` your default gateway, then the problem is likely beyond your server and its local network. Common reasons include:
  - **Incorrect Default Gateway**: Double-check your `ip r` output. If the gateway address is wrong, traffic can't leave your local network.
  - **Router/Firewall Issues**: Your router (which is your default gateway) might have issues, or there might be a firewall blocking outgoing traffic on the router/network level.
  - **ISP (Internet Service Provider) Problems**: There might be an outage or issue with your internet service provider.
  - **Firewall on your Linux Server**: Although we'll cover this in more detail later, your server's own firewall might be blocking outgoing `ping` requests, or incoming replies.

  This failure indicates that while your server can talk to its immediate neighbor (the gateway), the gateway isn't successfully forwarding that traffic to the internet, or the internet isn't sending replies back.

By starting with a `ping` to a known external IP address, you can quickly differentiate between local network issues and problems further upstream on the internet.

### Identify where Break Downs Occur with `traceroute` or `mtr`

Think of `traceroute` (or `mtr`) as a tool that maps the journey your network packets take from your server to a destination on the internet. Instead of just knowing if the mail arrived, this tool shows you every post office, sorting facility, and road segment the mail went through on its way to the destination. This is incredibly useful for pinpointing the exact "hop" or router where connectivity issues occur.

#### The traceroute Command

`traceroute` works by sending packets with increasing "Time To Live" (TTL) values. Each time a packet passes through a router (a "hop"), its TTL decreases by one. When the TTL reaches zero, the router sends an error message back to your server. By timing these error messages, `traceroute` can identify each router in the path.

To use it, you'll typically specify an IP address or a hostname. Let's use Google's public DNS server again (`8.8.8.8`):

```Bash
traceroute 8.8.8.8
```

The output will list a series of numbered lines, one for each "hop" or router encountered:

```shell
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.722 ms  0.706 ms  0.686 ms
 2  some-isp-router-1.com (203.0.113.1)  5.123 ms  5.245 ms  5.301 ms
 3  another-isp-router-2.com (198.51.100.1)  12.345 ms  12.398 ms  12.411 ms
 4  google-router-a.net (172.253.x.x)  15.001 ms  15.023 ms  15.045 ms
 5  google-router-b.net (142.250.y.y)  15.111 ms  15.132 ms  15.154 ms
 6  dns.google (8.8.8.8)  15.222 ms  15.243 ms  15.265 ms
```

##### Interpreting `traceroute` Output

- **Hop Number (1, 2, 3...)**: Each number represents a router your packets passed through.
- **Hostname/IP Address**: The name (if resolvable) and IP address of the router at that hop.
- **Three Time Readings (e.g.,** `0.722 ms 0.706 ms 0.686 ms`**)**: These are the times (latency) for three different probes sent to that router. Lower numbers are better.

**What to look for when troubleshooting**:

- **Asterisks (**`* * *`**)**: If you see three asterisks (`* * *`) for a hop, it means no response was received from that router. This is a strong indicator of a problem!
  - If asterisks appear at a specific hop and continue for all subsequent hops, it usually means the connection is breaking down at or after that point. The router at that hop might be down, misconfigured, or a firewall might be blocking traffic.
  - If you see asterisks for some hops but then responses resume later, it might just mean that particular router doesn't prioritize responding to `traceroute` probes, or there's temporary congestion. It's the sustained `* * *` that's the real concern.
- **Sudden Increase in Latency**: A sudden jump in the time readings at a specific hop (e.g., from 10ms to 200ms) can indicate network congestion or a problem with that particular router.

#### The mtr Command (My Traceroute)

`mtr` combines the functionality of `ping` and `traceroute` into a single, continuously updating display. It provides a real-time view of packet loss and latency to each hop. It's often preferred for continuous monitoring.

To use it:

```Bash
mtr 8.8.8.8
```

It will show a live updating table. Press `q` to quit.

```shell
                                 My traceroute  [v0.94]
your_server (0.0.0.0)                                     2025-05-27T14:06:32-0700
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                              Packets               Pings
 Host                                          Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. _gateway                                    0.0%    10    0.7   0.7   0.6   0.8   0.1
 2. some-isp-router-1.com                       0.0%    10    5.2   5.3   5.1   5.5   0.1
 3. another-isp-router-2.com                    0.0%    10   12.4  12.4  12.3  12.5   0.0
 4. google-router-a.net                         0.0%    10   15.0  15.0  15.0  15.1   0.0
 5. google-router-b.net                         0.0%    10   15.1  15.1  15.1  15.2   0.0
 6. dns.google                                  0.0%    10   15.2  15.2  15.2  15.3   0.0
```

- `Loss%`: This is the most crucial column. If you see high or increasing packet loss at a particular hop, it indicates a problem at that specific point in the network path.
- `Avg` **(Average Ping)**: The average latency to that hop.

**When to use** `traceroute`**/**`mtr`: After you've confirmed that your server's local network connection and default gateway are working, but you *still* can't reach external services (e.g., `ping 8.8.8.8` fails), `traceroute` or `mtr` is your next go-to tool. It helps you determine if the problem is with your ISP, an upstream router, or the destination itself.

### Using `dig` and `nslookup` to Diagnose DNS Resolution Problems

Remember our "office directory" analogy for DNS? Even if your server's network connection is perfectly fine (you can ping `8.8.8.8` successfully!), if its DNS server settings (`/etc/resolv.conf`) are wrong, or if the DNS server itself is having issues, your server won't be able to translate `google.com` into an IP address. This means you won't be able to access websites or services by their human-readable names.

#### The `dig` Command (Domain Information Groper)

`dig` is a powerful and flexible tool for querying DNS name servers. It's often preferred by administrators for its detailed output.

To check if your server can resolve a domain name like `google.com`, you'd type:

```Bash
dig google.com
```

**Successful** `dig` **Output Example**:

```shell
; <<>> DiG 9.16.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36726
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             299     IN      A       142.250.190.46

;; Query time: 15 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue May 27 14:06:01 PDT 2025
;; MSG SIZE  rcvd: 55
```

**Key things to look for in successful dig output**:

- `status: NOERROR`: This is the most important part. It means the DNS query was successful and no error occurred.
- `ANSWER SECTION`: This section will contain the IP address (or addresses) that correspond to the domain name you queried. In this example, `google.com` resolved to `142.250.190.46`.
- `Query time`: How long it took to get the answer.
- `SERVER`: The IP address of the DNS server that provided the answer. This should match one of the nameserver entries in your `/etc/resolv.conf`.

**Unsuccessful** `dig` **Output Example (Common issues)**:

If you see output like this, it indicates a DNS problem:

```shell
; <<>> DiG 9.16.1-Ubuntu <<>> non-existent-domain12345.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; WARNING: recursion requested but not available from type
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue May 27 14:07:01 PDT 2025
;; MSG SIZE  rcvd: 139
```

- `status: NXDOMAIN`: This means "Non-Existent Domain." The DNS server you queried responded that the domain name you asked for does not exist. This is usually what you'd see if you typed a typo like `googe.com` or a domain that truly doesn't exist.

Another common failure when the DNS server itself is unreachable:

```shell
; <<>> DiG 9.16.1-Ubuntu <<>> google.com
;; global options: +cmd
;; connection timed out; no servers could be reached
```

- `connection timed out; no servers could be reached`: This is critical. It means your server couldn't even talk to its configured DNS servers. This points to issues like:
  - Incorrect DNS server IP addresses in `/etc/resolv.conf`.
  - Your DNS server being down or unreachable (e.g., firewall blocking access to it).
  - A network issue preventing access to any DNS server.

#### The `nslookup` Command (Name Server Lookup)

`nslookup` is another command-line tool for querying DNS. It's simpler and often used for quick checks.

To query:

```Bash
nslookup google.com
```

**Successful** `nslookup` **Output Example**:

```shell
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.190.46
```

**Unsuccessful** `nslookup` **Output Example**:

```shell
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can't find non-existent-domain12345.com: NXDOMAIN
```

And if the DNS server is unreachable:

```shell
;; connection timed out; no servers could be reached
```

**When to Use** `dig` **or** `nslookup`:

Use these tools when:

- You can `ping` external IP addresses (like `8.8.8.8`) successfully, but you cannot access websites or services by their domain names (e.g., `ping google.com` fails, but `ping 142.250.190.46` works). This specifically points to a DNS resolution problem.
- You want to verify which DNS server your system is actually using to resolve names.

## Checking Firewall Rules

Think of your server's firewall like a security guard standing at the door of your "house" (your server). This guard has a strict set of rules about who can come in and who can go out, and what "packages" (network traffic) are allowed. If the guard's rules are too restrictive, even legitimate traffic might be blocked.

Linux servers commonly use one of two main firewall management tools: `iptables` or `firewalld` (which uses `firewall-cmd`). You'll typically find one or the other active on a modern Linux distribution.

### Inspecting `iptables` Rules

If your server uses `iptables` directly, you can list its rules with:

```Bash
sudo iptables -L -n -v
```

- `sudo`: You'll often need superuser privileges to view or modify firewall rules.
- `-L`: Lists all rules in all chains.
- `-n`: Shows IP addresses and port numbers in numeric format, which is easier to read than names.
- `-v`: Provides verbose output, including packet and byte counters, which can show if rules are actually being hit.

The output can be quite detailed, but here's what to look for:

```shell
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
   10  600 ACCEPT     all  --  lo     any     0.0.0.0/0            0.0.0.0/0
    5  300 ACCEPT     tcp  --  any    any     0.0.0.0/0            0.0.0.0/0            tcp dpt:22 state NEW,ESTABLISHED
    0    0 DROP       all  --  any    any     0.0.0.0/0            0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
```

- `Chain INPUT`: Rules for traffic coming into your server.
- `Chain OUTPUT`: Rules for traffic leaving your server.
- `Chain FORWARD`: Rules for traffic passing through your server (if it's acting as a router).
- `policy ACCEPT` **or** `policy DROP`: This is the default action for a chain if no specific rule matches. If it's `DROP` on `INPUT` or `OUTPUT` and you don't have explicit `ACCEPT` rules, that's a problem!
- `target` **(e.g.,** `ACCEPT`**,** `DROP`**,** `REJECT`**)**: What action to take if a packet matches the rule.
- `prot` **(protocol)**: `tcp`, `udp`, `icmp` (for ping), or `all`.
- `dpt` **(destination port)**: The port number the traffic is trying to reach (e.g., `22` for SSH, `80` for HTTP).

### Inspecting `firewalld` Rules

If your server uses `firewalld` (common on Red Hat-based systems like CentOS/RHEL/Fedora), you'll use the `firewall-cmd` command:

```Bash
sudo firewall-cmd --list-all
```

The output will be structured by zones, which are like different security profiles for different network connections:

```shell
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: ssh dhcpv6-client http
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

**Key things to look for in firewalld output**:

- `interfaces:`: Which network interfaces are assigned to this zone.
- `services:`: Allowed services (like `ssh`, `http`, `https`, `dns`). If a service you expect to work isn't listed, it's likely blocked.
- `ports:`: Specific port numbers allowed (e.g., `80/tcp`, `443/tcp`).

**Common Firewall Scenarios Leading to Connectivity Issues**:

- **Incoming Traffic Blocked**: If you can't connect to your server (e.g., SSH, web server), but your server can connect out, the `INPUT` chain in `iptables` or the `services/ports` in `firewalld` are likely blocking the inbound connection.
- **Outgoing Traffic Blocked**: Less common by default, but possible in highly secured environments. If your server can't reach anything even after passing all previous checks, the `OUTPUT` chain in `iptables` or specific outbound rules in `firewalld` might be the culprit. This would block things like ping and web requests originating from your server.
- **Specific Ports Blocked**: Even if general connectivity works, a specific application might fail if its required port is blocked (e.g., your web server on port 80 is unreachable, but SSH on port 22 works).

### Inspecting `ufw` Rules

If your server uses `ufw`, it's essentially a user-friendly front-end for `iptables`. While the underlying rules are still `iptables`, `ufw` simplifies their management.

To check `ufw`'s status and rules, you'll use:

```Bash
sudo ufw status verbose
```

Here's what you might see:

```shell
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
Anywhere                   ALLOW       192.168.1.0/24
```

Key things to look for in `ufw` output:

- `Status: active`: This indicates that `ufw` is running and enforcing its rules. If it says `inactive`, your firewall is off, and traffic isn't being filtered by `ufw`.
- `Default: deny (incoming), allow (outgoing), disabled (routed)`: These are the default policies. A common secure setup is to `deny` incoming traffic by default and `allow` outgoing traffic. If your incoming traffic is blocked, this `deny (incoming)` policy is likely why, unless you have explicit `ALLOW` rules.
- **The rule list (e.g.,** `22/tcp ALLOW Anywhere`)**:** These are the specific rules you've configured.
  - `To`: The destination port or service on your server.
  - `Action`: `ALLOW` (permit traffic), `DENY` (silently drop traffic), or `REJECT` (drop traffic and send an error back).
  - `From`: The source IP address or network that the rule applies to. `Anywhere` means from any source.

**How** `ufw` **helps troubleshoot (and common** `ufw` **scenarios)**:

- `Status: inactive`: If `ufw` is inactive, it's not the cause of your network issues (unless the underlying `iptables` have been manipulated directly). You'd need to look at `iptables` or `firewalld` if another firewall is active.
- **Default Deny, No Allow Rule**: If `Default: deny (incoming)` is set, but you don't have an `ALLOW` rule for the specific service/port you're trying to reach (e.g., you're trying to connect to a web server on port 80 but there's no `80/tcp ALLOW` rule), then the traffic will be blocked. This is probably the most common `ufw` issue.
- **Incorrect Source** `From` **Rule**: You might have an `ALLOW` rule, but it only allows traffic from a specific IP address or network, and the client trying to connect is coming from a different source.

When troubleshooting, checking firewall rules is crucial because they can override all other network settings. If a rule says "block this traffic," it will be blocked, regardless of correct IP addresses or routing.

## Analyzing Network Traffic

### Capture and Analyze Packets with `tcpdump`

Think of `tcpdump` as a very specialized, high-speed camera that sits on your server's network card and takes snapshots of every single "packet" (piece of data) that passes by. It allows you to see the raw network communication, which can be invaluable for diagnosing complex or subtle issues that aren't apparent with `ping` or `traceroute`. It's like checking the mail, but also watching the mail carrier to see exactly what they're doing with each envelope.
Basic Usage of tcpdump

Because `tcpdump` can generate a lot of output, you usually want to filter what it shows you. You also need `sudo` privileges to run it.

A common way to start is to specify the network interface you want to monitor (e.g., `eth0`) and a filter for the traffic you're interested in.

```Bash
sudo tcpdump -i eth0 host 8.8.8.8
```

- `sudo`: Required to capture network traffic.
- `-i eth0`: Specifies that you want to capture traffic on the eth0 network interface. Replace `eth0` with your server's actual interface name (which you can find using ip a).
- `host 8.8.8.8`: This is a filter. It tells `tcpdump` to only show you packets that are either coming from or going to the IP address `8.8.8.8`.

You can also filter by port (e.g., to see web traffic):

```Bash
sudo tcpdump -i eth0 port 80
```

- `port 80`: Shows only traffic on port 80 (standard HTTP web traffic).

Press `Ctrl+C` to stop `tcpdump`.

#### What to Look For (and Why it's Advanced)

The output of `tcpdump` can be very verbose and requires some understanding of network protocols. However, even a beginner can gain insight:

- **Seeing if Traffic is Reaching/Leaving**: If you're trying to send a `ping` to `8.8.8.8` but ping is timing out, you can run `tcpdump -i eth0 host 8.8.8.8` and then try to ping `8.8.8.8` in another terminal.
  - **If you see outgoing** `echo request` **packets**: Your server is sending the ping. This tells you the problem is not with your server sending the request.
  - **If you don't see outgoing** `echo request` **packets**: Your server isn't even trying to send the packet, which points to an issue with its local configuration (IP, routing, or perhaps an outbound firewall rule).
  - **If you see outgoing** `echo request` **but no incoming** `echo reply` **packets**: This means the ping left your server, but the reply never came back. This points to a problem further out on the network, a return path issue, or a firewall blocking the reply.

- **Confirming Connections**: If you expect a web server to be listening on port 80, you can use `tcpdump -i eth0 port 80` while trying to connect to it. You'd look for TCP `SYN` packets (for connection initiation) and `SYN-ACK` (for acknowledgment). If you only see `SYN` from the client but no `SYN-ACK` from your server, it suggests your server isn't responding, possibly due to a firewall on the server itself, or the web server application not running.

**When to Use** `tcpdump`:

`tcpdump` is typically used as a last resort when basic checks (`ping`, `traceroute`, `firewall` rules) don't fully explain the problem. It allows you to confirm assumptions about network flow directly at the packet level. It's powerful, but it requires patience and some knowledge of networking fundamentals.
