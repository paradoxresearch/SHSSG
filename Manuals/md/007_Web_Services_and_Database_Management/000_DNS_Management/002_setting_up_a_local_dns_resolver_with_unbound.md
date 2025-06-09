# Setting Up a Local DNS Resolver with `unbound`

## Contents

- [Setting Up a Local DNS Resolver with `unbound`](#setting-up-a-local-dns-resolver-with-unbound)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding DNS Resolution](#understanding-dns-resolution)
    - [Introducing Unbound](#introducing-unbound)
  - [Installing Unbound](#installing-unbound)
  - [Basic Configuration of Unbound](#basic-configuration-of-unbound)
  - [Testing Your Local DNS Resolver](#testing-your-local-dns-resolver)
  - [Forwarding Queries (Optional)](#forwarding-queries-optional)
    - [Configuring Forwarders in the `unbound.conf` File](#configuring-forwarders-in-the-unboundconf-file)
  - [DNSSEC Validation with Unbound](#dnssec-validation-with-unbound)
    - [Check if DNSSEC Validation is Working using `dig`](#check-if-dnssec-validation-is-working-using-dig)

## Introduction

### Understanding DNS Resolution

magine you want to call a friend. You probably don't remember their phone number by heart, but you do remember their name. So, you open your phonebook, find their name, and then get their number to make the call.

DNS resolution works in a similar way for the internet. When you type a website address (like `example.com`) into your browser, your computer needs to find the actual address of the server that hosts that website. This actual address is a series of numbers called an **IP address** (like `93.184.216.34`).

The process of finding the IP address for a given website name is called **DNS resolution**. Here's a simplified view of the steps involved:

1. **Your Computer Asks a Resolver**: Your computer (the client) sends a request to a **DNS resolver**. Think of the resolver as your personal phonebook operator.
2. **The Resolver Looks Up the Address**: The resolver then goes through a process to find the IP address associated with the website name. It might have this information stored already (like remembering a frequently called number), or it might need to ask other DNS servers for help.
3. **Root Servers (The Master Phonebooks)**: If the resolver doesn't know the answer, it starts by asking special servers called **root servers**. These are like the main directories that know where to find other more specific phonebooks.
4. **TLD Servers (Top-Level Domain Phonebooks)**: The root servers direct the resolver to Top-Level Domain (TLD) servers. For `.com` addresses, it would go to a `.com` TLD server. This is like going to the section of the phonebook for businesses.
5. **Authoritative Name Servers (The Specific Listing)**: Finally, the TLD server directs the resolver to the authoritative name server for the specific domain (`example.com`). This server holds the definitive IP address for that website, like the specific listing for your friend in the phonebook.
6. **The Resolver Responds**: Once the resolver gets the IP address, it sends it back to your computer.
7. **Connection Established**: Your computer can now use the IP address to connect to the web server and display the website.

Here's a simple diagram to visualize this:

```shell
[Your Computer] --> Asks: "What's the IP for example.com?" --> [DNS Resolver]
    |
    v
[DNS Resolver] --> Asks Root Server --> [Root Server]
    |
    v
[Root Server] --> Tells Resolver to ask .com TLD Server --> [.com TLD Server]
    |
    v
[.com TLD Server] --> Tells Resolver to ask example.com's Name Server --> [example.com Name Server]
    |
    v
[example.com Name Server] --> Responds with IP Address --> [DNS Resolver]
    |
    v
[DNS Resolver] --> Sends IP Address back --> [Your Computer]
```

### Introducing Unbound

Unbound is a specific piece of software that acts as a **validating, recursive DNS resolver**. Let's break down what those terms mean:

- **Recursive**: When your computer asks Unbound for the IP address of a website, and Unbound doesn't have the answer stored, it will go through the entire process of asking the root servers, TLD servers, and authoritative name servers on its own, until it finds the answer. It does all the legwork for you, just like our diligent phonebook operator.
- **Validating**: Unbound can perform **DNSSEC (Domain Name System Security Extensions)** validation. Think of DNSSEC as a way to add security and authenticity to the DNS lookup process. It's like having a digital signature on the phonebook entry, so you can be sure the IP address you received is the correct and untampered one. Unbound checks these signatures to ensure the integrity of the DNS data.

Here are some key features and advantages of using Unbound as your local DNS resolver:

- **Security**: Its built-in DNSSEC validation helps protect you from certain types of cyberattacks that try to redirect you to malicious websites.
- **Speed and Efficiency**: By caching (storing) the IP addresses it has looked up recently, Unbound can answer subsequent requests for the same websites much faster. This can lead to a snappier browsing experience.
- **Lightweight**: Compared to some other DNS resolver software like BIND (Berkeley Internet Name Domain, often referred to as `named`), Unbound is designed to be more lightweight and easier to configure for simple resolving tasks. BIND is very powerful but can be more complex to set up for a basic local resolver.
- **Focus on Resolving**: Unbound's primary focus is on being a good resolver. While BIND can also act as an authoritative name server (hosting the DNS records for a domain), Unbound typically just handles the task of looking up addresses.

Think of it this way: If BIND is like a large telecommunications company that can handle everything from looking up numbers to managing entire phonebook directories, Unbound is like a specialized, fast, and secure personal assistant whose main job is to quickly and reliably find the phone numbers you need.

## Installing Unbound

To install Unbound, you'll need to use the apt-get command. This command interacts with the apt system. Here's the command you'll need to run:

```Bash
sudo apt-get update && sudo apt-get install unbound
```

Let's break down this command:

- `sudo`: This command allows you to run the following command with administrative privileges (as the "superuser"). Installing software requires these privileges because it makes changes to the core system. You'll likely be prompted to enter your password after running this.
- `apt-get`: This is the command-line tool for handling packages provided by the `apt` system.
- `update`: This part of the command tells `apt-get` to refresh the list of available packages from the repositories (online software sources). It's always a good idea to run this before installing new software to make sure you have the latest information. The && means "if the previous command was successful, then run the next command."
- `install`: This part of the command tells `apt-get` that you want to install a specific package.
- `unbound`: This is the name of the package we want to install – the Unbound DNS resolver software.

So, when you run this entire command, your server will first update its list of available software, and then it will download and install the Unbound package and any other software it needs to run.

Once the installation is complete, Unbound will typically start running automatically as a service on your server.

## Basic Configuration of Unbound

When software like Unbound is installed, it usually comes with a default set of instructions on how it should operate. These instructions are stored in one or more **configuration files**. Think of these files as the detailed manual for your Unbound operator, telling it how to behave.

For Unbound, the main **configuration** file is typically located at:

```shell
/etc/unbound/unbound.conf
```

You can use a text editor like `nano` or `vim` (which are common command-line editors on Ubuntu Server) to view and modify this file. For example, to open it with nano, you would use the command:

```Bash
sudo nano /etc/unbound/unbound.conf
```

**Important Note**: Be very careful when editing configuration files. Incorrect changes can prevent Unbound from working correctly or even cause other network issues. It's always a good idea to make a backup of the original file before making any changes. You can do this with the `cp` command:

```Bash
sudo cp /etc/unbound/unbound.conf /etc/unbound/unbound.conf.backup
```

Inside the `unbound.conf file`, you'll find various sections and options that control how Unbound works. For a basic local resolver setup, there are a few key options we'll want to pay attention to.

One of the most important is the `interface:` option. This tells Unbound which network address(es) it should listen on for DNS queries. For a local resolver that's only meant to be used by the server it's running on, it's common practice to set this to the loopback address:

```shell
interface: 127.0.0.1
```

The loopback address (`127.0.0.1`) is a special IP address that your computer uses to refer to itself. By setting the interface to this address, you're telling Unbound to only accept DNS queries that originate from the same server. This enhances security by preventing external machines from using your Unbound instance as a public resolver.

Another important option is the `port:` directive, which specifies the network port Unbound will listen on for DNS queries. The standard port for DNS is port `53`, so you'll usually see this set as:

```shell
port: 53
```

Unless you have a specific reason to change it, it's best to leave this at the default port 53.

So, a very basic `server:` section in your `unbound.conf` file might look something like this:

```shell
server:
    interface: 127.0.0.1
    port: 53
```

After making any changes to the configuration file, you'll need to restart the Unbound service for the changes to take effect. You can do this using the systemctl command:

```Bash
sudo systemctl restart unbound
```

## Testing Your Local DNS Resolver

We'll use command-line tools to send DNS queries and see how Unbound responds. Two common tools for this purpose are `dig` and `nslookup`. Both are usually available on Ubuntu Server, but dig tends to provide more detailed information.

Let's start with `dig`. You can use it to query a specific domain name and see which DNS server answers the query. To test if your local Unbound resolver is working, you can run the following command:

```Bash
dig @127.0.0.1 www.example.com
```

Let's break down this command:

- `dig`: This is the command-line tool itself.
- `@127.0.0.1`: This tells dig to send the DNS query to the DNS server running at the IP address 127.0.0.1 (which is your local Unbound resolver, as we configured).
- `www.example.com`: This is the domain name we want to look up.

When you run this command, you should see a section in the output called **ANSWER SECTION**. If your local Unbound resolver is working correctly, this section should contain the IP address for `www.example.com`. It will look something like this:

```shell
;; ANSWER SECTION:
www.example.com.    83333   IN      A       93.184.216.34
```

Here, `93.184.216.34` is the IP address for `www.example.com`. The `IN` stands for "Internet", and `A` indicates that this is an IPv4 address record. The number `83333` is the time-to-live (TTL) in seconds, indicating how long this record can be cached.

Another useful piece of information in the `dig` output is the **SERVER** line in the **HEADER** section. This should confirm that the query was answered by your local resolver:

```shell
;; SERVER: 127.0.0.1#53(127.0.0.1)
```

This line tells you that the DNS server at 127.0.0.1 on port 53 (our Unbound server) provided the answer.

You can also use `nslookup` to perform a similar test:

```Bash
nslookup www.example.com 127.0.0.1
```

Here, `www.example.com` is the domain you're querying, and `127.0.0.1` specifies the DNS server to use. The output from `nslookup` will be less detailed than dig, but it should still show you the IP address for `www.example.com` and indicate that the query was handled by `127.0.0.1`.

## Forwarding Queries (Optional)

So far, your Unbound resolver has been set up to be **recursive**. This means it does all the work of finding the IP address itself, by querying the root servers and so on. However, you can also configure Unbound to forward DNS queries to other DNS resolvers, often called **upstream resolvers**.

Think of it like this: Instead of your personal phonebook operator making all the calls themselves, you can tell them to first try asking a more experienced operator (the upstream resolver) who might already know the number. If the experienced operator doesn't know, then your personal operator will go through the whole process.

There are several reasons why you might want to configure forwarding:

- **Simplicity**: It can sometimes simplify the initial setup, as you don't have to rely entirely on your own server to handle all recursive lookups.
- **Leveraging Existing Infrastructure**: You might want to use the DNS resolvers provided by your Internet Service Provider (ISP), which might be optimized for speed within their network.
- **Using Public DNS Servers**: You might prefer to use well-known public DNS resolvers like Google's (8.8.8.8 and 8.8.4.4) or Cloudflare's (1.1.1.1), which are often fast and have good security features.

To configure forwarding in Unbound, you need to edit the `unbound.conf` file again. Within the `server:` section, you would add one or more `forward-addr:` lines, specifying the IP addresses of the DNS servers you want to forward queries to.

For example, to forward queries to Google's public DNS servers, you would add these lines to your server: section:

```shell
server:
    interface: 127.0.0.1
    port: 53
    forward-addr: 8.8.8.8
    forward-addr: 8.8.4.4
```

Similarly, to use Cloudflare's public DNS servers, you would use:

```shell
server:
    interface: 127.0.0.1
    port: 53
    forward-addr: 1.1.1.1
    forward-addr: 1.0.0.1
```

You can have multiple `forward-addr:` lines, and Unbound will typically try them in the order they are listed.

**Important**: If you configure forwarding, your Unbound server will no longer be performing fully recursive lookups itself for queries it doesn't have in its cache. Instead, it will rely on the upstream resolvers to do that work. However, Unbound can still perform DNSSEC validation on the responses it receives from the forwarders, if they support it.

After making these changes, remember to save the `unbound.conf` file and restart the Unbound service:

```Bash
sudo systemctl restart unbound
```

To test if forwarding is working, you can use `dig` again, querying a domain that your Unbound server likely doesn't have cached yet. You should still get the correct IP address in the ANSWER SECTION. You can also try to monitor network traffic on your server (using tools like `tcpdump` if you're comfortable with them) to see if queries are being sent to the forwarder addresses you configured.

### Configuring Forwarders in the `unbound.conf` File

As we discussed, you'll need to open the Unbound configuration file. Let's use `nano` again:

```Bash
sudo nano /etc/unbound/unbound.conf
```

Once the file is open, you'll typically find a `server:` section. If it doesn't exist, you can create one at the beginning of the file (after any comment lines, which start with `#`).

Within this `server:` section, you'll add the `forward-addr:` lines, one for each forwarder you want to use. For example, if you want to forward to Cloudflare's primary and secondary DNS servers, you would add:

```shell
server:
    interface: 127.0.0.1
    port: 53
    forward-addr: 1.1.1.1
    forward-addr: 1.0.0.1
```

If you wanted to use Google's public DNS servers instead, you would use:

```shell
server:
    interface: 127.0.0.1
    port: 53
    forward-addr: 8.8.8.8
    forward-addr: 8.8.4.4
```

You can mix and match, or add more forwarders if you like. Unbound will generally try to use the first one in the list and move to the next if the first one doesn't respond.

**Important Considerations**:

- **Order matters (somewhat)**: While Unbound will try different forwarders if one fails, the order in which you list them can influence which server is used most often. You might want to put the fastest or most reliable forwarders first.
- **Security and Privacy**: Be mindful of the privacy policies of the DNS resolvers you choose to forward to. Some public resolvers offer enhanced privacy features.
- **Loop Prevention**: Ensure that the forwarders you choose are not configured to forward back to your own Unbound server, as this could create a loop. Public DNS servers are generally safe in this regard.

After you've added the `forward-addr:` lines to your `server:` section, save the file and close the editor. Then, you must restart the Unbound service for the changes to take effect:

```Bash
sudo systemctl restart unbound
```

To verify that forwarding is working, you can use dig to query a domain as we did before:

```Bash
dig @127.0.0.1 www.example.com
```

The output will still show that your local Unbound server (`127.0.0.1`) answered the query. However, behind the scenes, Unbound will have forwarded the query to one of the upstream resolvers you configured (if it didn't have the answer cached). You can sometimes get a hint of this by looking at the query time in the `dig` output. If it's the first time you're querying a particular domain after restarting Unbound, and you're forwarding, the query time might be slightly longer as your server waits for the upstream resolver to respond. Subsequent queries for the same domain should be faster due to caching.

## DNSSEC Validation with Unbound

Think back to our phonebook analogy. What if someone malicious changed the phone number listed for your friend to a different number, one that connects you to a scammer? You'd unknowingly be calling the wrong person.

DNSSEC is like adding a system of digital signatures to the DNS records. These signatures allow your DNS resolver (like Unbound) to verify that the DNS information it receives is authentic and hasn't been tampered with during its journey across the internet. It's a way to ensure the "phonebook entry" you get is the real one, signed by the rightful owner of the domain.

**Why is DNSSEC important?**

Without DNSSEC, there's a possibility of "DNS spoofing" or "DNS cache poisoning" attacks. In these attacks, malicious actors can inject false DNS records into DNS resolvers, redirecting users to fake websites (e.g., a phishing site that looks like your bank). DNSSEC helps prevent these attacks by providing a way to cryptographically verify the authenticity of DNS data.

**Unbound as a Validating Resolver**:

One of the key strengths of Unbound is that it is a **validating** resolver. This means that when it receives DNS records, it can automatically check the DNSSEC signatures associated with those records. If the signatures are valid, Unbound knows the information is legitimate. If the signatures are missing or invalid, Unbound knows something is wrong and can refuse to provide that potentially tampered data to your computer.

**How does this relate to our setup?**

Since we're setting up Unbound as our local resolver, and it's a validating resolver by default, it will automatically try to perform DNSSEC validation on the DNS responses it receives. This adds an extra layer of security to your internet communication.

In most default configurations of Unbound, DNSSEC validation is enabled. You typically don't need to do any extra configuration to turn it on. Unbound will handle the validation process automatically.

### Check if DNSSEC Validation is Working using `dig`

We'll use the `dig` command again, but this time we'll specifically ask for DNSSEC-related information. A common way to do this is by using the `+dnssec` option. Let's query a domain that is known to be DNSSEC-signed, like `dnssec.works`.

Run the following command:

```Bash
dig @127.0.0.1 dnssec.works +dnssec
```

The `+dnssec` option tells `dig` to include DNSSEC records in the query and the response. If DNSSEC validation is working correctly, you should see a few key things in the output:

1. **The** `AD` **flag in the HEADER SECTION**: This flag stands for "Authenticated Data". If you see `ad` in the flags section of the header, it means that the resolver (your local Unbound) has successfully validated the DNSSEC signatures for the response. The header might look something like this:

    ```shell
    ;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
    ```

    Notice the ad in the flags.

2. **DNSSEC Records in the ANSWER SECTION**: You might see additional record types in the ANSWER SECTION, such as `RRSIG` (Resource Record Signature). These are the digital signatures that Unbound has verified. For example:

    ```shell
    dnssec.works.       3600    IN  A       192.0.2.1
    dnssec.works.       3600    IN  RRSIG   A 5 2 3600 20250617000000 20250517000000 35387 dnssec.works. [signature data...]
    ```

    The presence of the RRSIG record indicates that the A record (the IP address) is signed.

If you run this `dig` command and see the `ad` flag in the header, it's a strong indication that your local Unbound resolver is successfully performing DNSSEC validation.

Now, let's try querying a domain that is known to be not DNSSEC-signed, like a very new or simple domain. You can try something like `example.com`.

```Bash
dig @127.0.0.1 example.com +dnssec
```

In this case, you should still get an answer (the IP address for `example.com`), but you likely won't see the `ad` flag in the header, and there won't be any `RRSIG` records in the ANSWER SECTION. This is because `example.com` itself doesn't have DNSSEC records to be validated.
