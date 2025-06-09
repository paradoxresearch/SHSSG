# Introduction to Intrusion Detection Systems (IDS) Using Snort

## Contents

- [Introduction to Intrusion Detection Systems (IDS) Using Snort](#introduction-to-intrusion-detection-systems-ids-using-snort)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Intrusion Detection Systems (IDS)](#understanding-intrusion-detection-systems-ids)
    - [Differentiating Between Network-based IDS (NIDS) and Host-based IDS (HIDS)](#differentiating-between-network-based-ids-nids-and-host-based-ids-hids)
    - [How an IDS Works](#how-an-ids-works)
  - [Introduction to Snort](#introduction-to-snort)
    - [Three Primary Modes of Operation](#three-primary-modes-of-operation)
    - [Snort's Architecture](#snorts-architecture)
  - [Basic Snort Rule Structure](#basic-snort-rule-structure)
    - [The Rule Header](#the-rule-header)
      - [Action](#action)
      - [Protocol](#protocol)
      - [Source and Destination IP Addresses and Port Numbers](#source-and-destination-ip-addresses-and-port-numbers)
    - [Rule Options](#rule-options)

## Introduction

### Understanding Intrusion Detection Systems (IDS)

Think of an Intrusion Detection System like a sophisticated home security system. Imagine you have sensors on your doors and windows (that's like the IDS monitoring network traffic), and a rulebook that says things like "if a window opens at 3 AM, that's suspicious!" (those are like the rules in an IDS).

The main purpose of an IDS is to **monitor network traffic or system activity for any malicious or suspicious behavior**. It acts as a detective, observing what's happening and raising an alert if it sees something that matches its rulebook of known threats or anomalies. It doesn't usually block the activity (that's more the job of an Intrusion Prevention System - IPS), but it lets you know something potentially bad is going on so you can take action.

### Differentiating Between Network-based IDS (NIDS) and Host-based IDS (HIDS)

Think of it this way: our home security system analogy can be expanded.

- **Network-based IDS (NIDS)** is like having security cameras monitoring all the traffic coming into and going out of your house (your network). It sits at a strategic point in your network and analyzes the data flowing across it, looking for suspicious patterns. A key advantage is that it can often detect attacks targeting multiple systems on your network. However, it might not have detailed visibility into what's happening on each individual computer.

- **Host-based IDS (HIDS)**, on the other hand, is like having an alarm system installed directly on each door and window (each individual server or computer). It monitors the activity on that specific system, looking at things like system logs, file integrity, and running processes for signs of intrusion. HIDS can provide very detailed information about what's happening on a particular machine, but managing them on many servers can become complex.

Imagine a simple network with a router and a couple of servers. A NIDS would typically be placed at the router, watching all the traffic passing through. A HIDS would be installed as software on each of the servers, monitoring their individual activities.

### How an IDS Works

Imagine our NIDS sitting at the network's doorway. It's constantly **"sniffing"** all the network traffic that passes by. Think of it like eavesdropping on every conversation happening through that door. This "sniffing" involves capturing network packets, which are like the individual letters or words that make up those conversations.

Once the IDS captures these packets, it needs to understand what they're saying. This is where **rule-based detection** comes in. The IDS has a set of predefined **rules**, which are like specific patterns or signatures of known malicious activity. These rules look for specific sequences of bytes, particular protocols being used in unusual ways, or other characteristics that are associated with attacks.

For example, a rule might say: "Alert if there's a connection to a specific port that is known to be used by a particular type of malware, and if the data being sent contains this specific text string." This text string is like a **signature** – a unique characteristic that identifies that specific malicious activity.

So, the IDS compares the captured network traffic against its library of rules or signatures. If a packet or a series of packets matches a rule, the IDS triggers an alert, letting the administrator know that something suspicious has been detected.

Think of it like a spam filter for your email. It examines the content and characteristics of each email against a set of rules to identify and flag potential spam. An IDS does a similar thing for network traffic.

## Introduction to Snort

Snort is a very popular **open-source Network Intrusion Detection System (NIDS)**. Think of it as one of the most widely used and trusted security guard dogs for your network. It was initially developed by Martin Roesch in 1998 and has since grown into a powerful and highly flexible tool, thanks to a large and active community of users and developers.

Being open-source means that its code is freely available, and anyone can contribute to its development, share rules, and help improve it. This massive community support is one of Snort's biggest strengths. It means there's a wealth of documentation, tutorials, and pre-written rule sets available, which is incredibly helpful for new users. Plus, because so many people use and scrutinize it, potential security flaws are often quickly identified and fixed.

For a system administrator, Snort is a valuable tool because it's command-line based, highly configurable, and can be tailored to the specific security needs of your servers.

### Three Primary Modes of Operation

**Snort has three primary modes of operation**: Sniffer mode, Packet Logger mode, and Network Intrusion Detection System mode. Think of these as different ways Snort can be configured to do its job.

1. **Sniffer Mode**: In this mode, Snort acts like a network traffic viewer. It captures packets flowing through the network interface you tell it to monitor and displays them on your screen in real-time. It's like watching the raw network "conversations" as they happen. This mode is very useful for understanding network traffic and troubleshooting network issues. You can see the source and destination of the communication, the protocols being used, and even the data being transmitted.

2. **Packet Logger Mode**: This mode takes the "traffic viewing" a step further. Instead of just displaying the packets on the screen, Snort captures and saves them to a log file. This is like recording all the network "conversations" for later analysis. This mode is invaluable for investigating security incidents after they've occurred or for long-term monitoring and analysis of network behavior.

3. **Network Intrusion Detection System (NIDS) Mode**: This is Snort's most active and powerful mode. In this mode, Snort not only captures and optionally logs packets but also analyzes them in real-time against a set of rules. When a packet matches a rule, Snort generates an alert, notifying you of potential malicious activity. This is like our security guard dog barking when it sees someone suspicious according to its training (the rules).

So, to summarize:

- **Sniffer Mode**: Real-time traffic viewing.
- **Packet Logger Mode**: Capturing and saving traffic for later.
- **NIDS Mode**: Real-time traffic analysis and alerting based on rules.

As a system administrator, you'll likely spend most of your time using Snort in NIDS mode to actively monitor your server's network traffic for threats. However, the other modes can be very useful for debugging and analysis.

### Snort's Architecture

Think of Snort as having several key departments working together to analyze network traffic. The main components are:

1. **Packet Decoder**: This is like the receptionist. It's the first point of contact for incoming network packets. Its job is to take the raw network data and break it down into a format that the rest of Snort can understand. It identifies the different layers of network protocols (like Ethernet, IP, TCP, UDP) and extracts the relevant information.

2. **Preprocessors**: Think of these as specialized assistants who prepare the information for the main analyst. Preprocessors perform various tasks on the decoded packets before they reach the detection engine. This might involve things like reassembling fragmented packets, decoding application layer protocols (like HTTP), or detecting port scans. Preprocessors help to normalize the data and make the detection engine's job more efficient.

3. **Detection Engine**: This is the heart of Snort – the main analyst. It takes the preprocessed packets and compares them against the ruleset that you've configured. If a packet matches a rule, the detection engine generates an alert.

4. **Output Modules**: Once an alert is generated, the output modules decide what to do with it. They are like the reporting and action department. They can log the alerts to various formats (like text files, databases, or even send them to other security tools). You can configure different output modules depending on how you want to be notified and how you want to store the alert information.

Here's a simple way to visualize this flow:

```shell
Network Traffic --> Packet Decoder --> Preprocessors --> Detection Engine (Ruleset) --> Output Modules (Alerts/Logs)
```

Understanding this basic architecture will help you later when you want to fine-tune Snort's behavior and configure it effectively.

## Basic Snort Rule Structure

Think of a Snort rule as a precise instruction you give to Snort's detection engine. Each rule essentially says, "If you see network traffic that matches this specific pattern, then generate an alert!"

A Snort rule has two main parts:

1. **The Rule Header**: This part defines the basic characteristics of the network traffic the rule is looking for. It's like the "who, what, where, and how" of the traffic. It specifies things like the action to take (e.g., alert), the network protocol involved (e.g., TCP, UDP, ICMP), the source and destination IP addresses and port numbers, and the direction of the traffic.

2. **The Rule Options**: This part contains more detailed instructions about what to look for within the content of the network traffic. These options are enclosed in parentheses and are separated by semicolons. They allow you to specify things like specific text patterns to search for, the size of the packet, and much more.

So, a basic Snort rule looks something like this:

```shell
action protocol source_ip source_port -> destination_ip destination_port (rule options);
```

Don't worry if this looks a bit technical right now. We'll break down each part in detail. Understanding this structure is fundamental to writing your own Snort rules and customizing Snort to your specific security needs.

### The Rule Header

#### Action

The **action** keyword tells Snort what to do when traffic matches the rest of the rule. Here are some of the most common actions:

- `alert`: This is the most frequently used action. It tells Snort to generate an alert when a matching packet is found. The alert will typically be logged and can also be displayed in real-time if Snort is run in console mode. Think of this as the security guard shouting, "Hey, I saw something suspicious!"

- `log`: This action tells Snort to simply log the matching packet but not generate an immediate alert. This can be useful for passively monitoring specific types of traffic without being overwhelmed by alerts. It's like the security guard making a quiet note of something that might be worth reviewing later.

- `pass`: This action tells Snort to ignore the matching traffic and stop processing it against any subsequent rules. This can be useful for explicitly excluding certain types of traffic that you know are safe and don't want Snort to analyze further, improving efficiency. It's like telling the security guard, "It's okay, that person has permission to be here, you can ignore them."

- `drop`: This action (often used when Snort is integrated as an Intrusion Prevention System - IPS) tells Snort to block the matching packet from reaching its destination. This requires Snort to be running in a specific inline mode. It's like the security guard physically stopping the suspicious person from entering. We'll likely touch more on this when we discuss IPS.

- `sdrop`: Similar to `drop`, but it also logs the dropped packet.

#### Protocol

The **protocol** field specifies the network protocol that the rule should inspect. The most common protocols you'll encounter are:

- `tcp`: Transmission Control Protocol. This is a connection-oriented protocol used for reliable data transfer, like browsing the web (HTTP/HTTPS), sending emails (SMTP), and transferring files (FTP). Think of it like a phone call where both parties establish a connection before talking and can confirm if the other person heard them correctly.

- `udp`: User Datagram Protocol. This is a connectionless protocol that prioritizes speed over reliability. It's often used for things like streaming video, online gaming, and DNS lookups. Think of it like sending a postcard – you send it out, but you don't necessarily know if it arrived or if all the information was received correctly.

- `icmp`: Internet Control Message Protocol. This protocol is used for sending control and error messages between network devices. Common uses include the `ping` command to check network connectivity. Think of it like network devices sending each other status updates or error reports.

- `ip`: This is a more general keyword that matches any IP packet, regardless of the higher-level protocol. You might use this in specific cases where you want to inspect IP layer information.

So, if you want to write a rule that looks for suspicious web traffic, you would likely use `tcp` as the protocol and specify port 80 (for HTTP) or 443 (for HTTPS). If you were looking for unusual `ping` activity, you would use `icmp`.

#### Source and Destination IP Addresses and Port Numbers

These tell Snort where the traffic is coming from and where it's going.

- **IP Addresses**: These identify the specific devices involved in the network communication. You can specify a single IP address (e.g., `192.168.1.100`), a range of IP addresses (e.g., `192.168.1.0/24` for the entire 192.168.1.x network), or use the keyword `any` to match any IP address.

- **Port Numbers**: These identify the specific applications or services running on those devices. For example, web servers typically listen on port 80 (HTTP) or 443 (HTTPS), and email servers often use port 25 (SMTP) or 110 (POP3). Similar to IP addresses, you can specify a single port (e.g., `80`), a range of ports (e.g., `1024:`), or use `any` to match any port.

- **Direction of Traffic (**`->`**)**: The arrow in the rule header (`->`) indicates the direction of the traffic being inspected *relative to the source and destination IP addresses and ports specified*.
  - `source_ip source_port -> destination_ip destination_port`: This looks at traffic going from the source to the destination.
  - `source_ip source_port <> destination_ip destination_port`: The `<>` (bidirectional) operator tells Snort to inspect traffic going in either direction between the specified source and destination IP addresses and ports.

So, for example, a rule looking for web traffic coming from any IP address to your server (let's say its IP is `192.168.1.200`) on the standard HTTP port would look something like this:

```shell
alert tcp any any -> 192.168.1.200 80 (msg:"Potential web access to server!"; sid:1000001; rev:1;);
```

In this rule:

- `alert`: is the action.
- `tcp`: is the protocol.
- `any any`: means any source IP address and any source port.
- `->`: indicates traffic going to...
- `192.168.1.200`: the destination IP address (your server).
- `8`0: the destination port (HTTP).
- The part in parentheses are the rule options, which we'll discuss next!

### Rule Options

The **Rule Options** are enclosed in parentheses `()` and provide much more specific instructions to the detection engine about what to look for within the packet's content and other characteristics. These options are separated by semicolons `;`.

Here are a few fundamental rule options that you'll encounter frequently:

- `msg:"<text>";`: This option allows you to include a custom message that will be displayed in the alert when the rule is triggered. This is very helpful for understanding what the rule is designed to detect. For example: `msg:"Potential SQL Injection Attack!";`

- `content:"<pattern>";`: This is a very powerful option that allows you to search for specific sequences of bytes or text within the packet's payload (the actual data being transmitted). For example: `content:"SELECT * FROM";` would look for that specific SQL query string within the packet's data. You can have multiple `content` options in a single rule, and you can use modifiers with them to specify things like case sensitivity or the starting position of the search.

- `sid:<number>;`: SID stands for Snort ID. This is a unique identifier for each Snort rule. It's important for tracking and managing rules. User-defined rules typically have SIDs starting at 1,000,000 or higher to avoid conflicts with official Snort rules. For example: `sid:1000001;`

- `rev:<number>;`: REV stands for revision. This is a counter that you increment each time you modify a rule. It helps with version control of your rules. For example: `rev:1;`

So, let's revisit our previous example and add some rule options:

```shell
alert tcp any any -> 192.168.1.200 80 (msg:"Potential web access to server!"; content:"GET /index.html"; sid:1000002; rev:1;);
```

In this enhanced rule:

- We've added a `msg` that will appear in the alert.
- We've used the `content` option to look for the specific text "GET /index.html" within the HTTP traffic. This might indicate someone is trying to access the main webpage.
- We've assigned a unique `sid` to this rule.
- We've set the initial `rev` to 1.

These are just a few basic rule options. Snort has many more powerful options that allow for very sophisticated pattern matching and analysis. As you get more comfortable with the basics, you can explore these advanced options.

**Remember**:

- `msg:"<text>";`: This is your way of adding a descriptive label to the alert. When this rule triggers, the text within the quotes will be part of the alert output, making it easier to understand what was detected. Think of it as adding a note to yourself about why this rule exists. For example, `msg:"Possible FTP brute-force attempt!";` clearly indicates the potential threat.

- `content:"<pattern>";`: This is where you tell Snort to look for specific bytes or text within the data portion of the network packet. This is incredibly powerful for detecting known attack patterns or specific application behavior. You can have multiple `content` options in a rule, and they are often used in combination to look for a sequence of patterns. For instance, to detect a specific type of web request, you might have:

    ```shell
    content:"GET /admin.php";
    content:"admin_user=";
    ```

    Snort would only trigger the rule if both of these content patterns are found in the packet.

- `sid:<number>;`: As we discussed, the Snort ID (sid) is a unique numerical identifier for each rule. This is crucial for managing and referencing rules in logs and configurations. Official Snort rules have SIDs below 1,000,000. When you write your own custom rules, it's best practice to start your SIDs at 1,000,000 or higher to avoid conflicts. For example, your first custom web server rule might have `sid:1000001;`, the next `sid:1000002;`, and so on.

Let's put it all together in another example. Imagine you want to detect if anyone on your network is trying to access a specific, potentially malicious file on a remote server (let's say the file is `bad_script.sh` on IP address `203.0.113.45` on port 80). Your rule might look like this:

```shell
alert tcp any any -> 203.0.113.45 80 (msg:"Attempt to access malicious script!"; content:"GET /bad_script.sh"; sid:1000003; rev:1;);
```

In this rule:

- We're alerting on TCP traffic going to the specific IP and port.
- The `msg` tells us what the rule is looking for.
- The `content` option specifies the exact HTTP GET request for the malicious script.
- We've given it a unique `sid`.

You can create more specific and powerful rules by combining different criteria in both the rule header and the rule options. For example, you can specify a particular protocol and a specific port, and look for specific content within that traffic.

Let's consider a scenario where you want to detect someone trying to use a specific command (`rm -rf /`) over a telnet connection to your server (assuming your server's IP is `192.168.1.200` and telnet uses port 23). A rule to detect this might look like:

```shell
alert tcp any any -> 192.168.1.200 23 (msg:"Potential malicious telnet command!"; content:"rm -rf /"; sid:1000004; rev:1;);
```

In this rule:

- We're looking for `tcp` traffic specifically destined for your server (`192.168.1.200`) on the `telnet` port (23).
- The `content` option tells Snort to look for the exact string `"rm -rf /"` within the data transmitted over that telnet connection.
- The `msg` clearly indicates the potential danger.

Important Considerations:

- **Rule Order**: The order of your Snort rules can sometimes be important. Snort processes rules in the order they appear in the configuration file. The `pass` action, in particular, can stop further rule processing for a matching packet.
- **Performance**: The more complex and numerous your rules are, the more processing power Snort will require. It's important to write efficient rules and only monitor the traffic that is relevant to your security concerns.
- **False Positives**: Sometimes, legitimate traffic can accidentally match a rule, resulting in a "false positive" alert. Tuning your rules to minimize false positives is an ongoing process.

This should give you a basic understanding of how to structure and write simple Snort rules. We've covered the rule header (action, protocol, IP addresses, ports, direction) and some fundamental rule options (`msg`, `content`, `sid`, `rev`).
