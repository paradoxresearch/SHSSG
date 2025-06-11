# Using SSH On Ubuntu

## Contents

- [Using SSH On Ubuntu](#using-ssh-on-ubuntu)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Checking if SSH is Installed](#checking-if-ssh-is-installed)
  - [Installing openssh-server](#installing-openssh-server)
  - [Obtain Your Servers IP Address](#obtain-your-servers-ip-address)
  - [Setting Up a SSH Client to Connect to Your Server](#setting-up-a-ssh-client-to-connect-to-your-server)
    - [Running a SSH Client on Windows](#running-a-ssh-client-on-windows)
      - [For Windows 10 or Later](#for-windows-10-or-later)
      - [For Earlier Versions of Windows](#for-earlier-versions-of-windows)
    - [Running a SSH Client on Mac](#running-a-ssh-client-on-mac)
    - [Running a SSH Client on Linux](#running-a-ssh-client-on-linux)
  - [Transfer Files Securely With the SSH Protocol](#transfer-files-securely-with-the-ssh-protocol)
    - [Downloading a File from Your Server](#downloading-a-file-from-your-server)
    - [Uploading a File to Your Server](#uploading-a-file-to-your-server)
    - [SCP Command Options](#scp-command-options)
      - [Copy a directory with the recursive option](#copy-a-directory-with-the-recursive-option)
      - [Using scp when the server has password authentication disabled](#using-scp-when-the-server-has-password-authentication-disabled)
      - [Using scp on a remote server NOT using port 22](#using-scp-on-a-remote-server-not-using-port-22)
      - [Using scp with multiple options](#using-scp-with-multiple-options)
  - [Creating and Using SSH Keys](#creating-and-using-ssh-keys)
    - [Generating a SSH Key Pair](#generating-a-ssh-key-pair)
    - [Copying Your SSH Key To Your Server](#copying-your-ssh-key-to-your-server)
      - [Method 1 - Using ssh-copy-id (Recommended)](#method-1---using-ssh-copy-id-recommended)
      - [Method 2 - Manually copying your public key to your server](#method-2---manually-copying-your-public-key-to-your-server)
  - [SSH Client Configuration](#ssh-client-configuration)
    - [Advanced SSH Configuration Options](#advanced-ssh-configuration-options)
      - [Host Patterns](#host-patterns)
      - [ProxyJump](#proxyjump)
      - [ForwardAgent](#forwardagent)
      - [LocalForward](#localforward)
      - [RemoteForward](#remoteforward)
      - [ServerAliveInternal](#serveraliveinternal)
      - [Example Advanced Configurations](#example-advanced-configurations)
  - [SSH Server Configurations](#ssh-server-configurations)
    - [Disabling Password Based Authentication](#disabling-password-based-authentication)
    - [Enable or Disable Access To Your Server by User](#enable-or-disable-access-to-your-server-by-user)
      - [Enable Access to Certain Users](#enable-access-to-certain-users)
      - [Disable Access to Certain Users](#disable-access-to-certain-users)
  - [SSH Tunnels](#ssh-tunnels)
    - [Tunneling Locally](#tunneling-locally)
      - [Local Tunneling Syntax](#local-tunneling-syntax)
      - [Local Tunneling Uses](#local-tunneling-uses)
    - [Tunneling Remotely](#tunneling-remotely)
      - [Remote Tunneling Syntax](#remote-tunneling-syntax)
      - [Remote Tunneling Uses](#remote-tunneling-uses)
    - [Tunneling Dynamically](#tunneling-dynamically)
      - [Dynamic Tunneling Syntax](#dynamic-tunneling-syntax)
      - [Dynamic Tunneling Uses](#dynamic-tunneling-uses)
  - [Sending Shell Commands to Your Server via SSH](#sending-shell-commands-to-your-server-via-ssh)
    - [Example SSH Remote Shell Command](#example-ssh-remote-shell-command)
    - [Command Chaining and Control](#command-chaining-and-control)
      - [Sequencing](#sequencing)
      - [Conditional AND](#conditional-and)
      - [Conditional OR](#conditional-or)
      - [Combining Conditional Operators](#combining-conditional-operators)
      - [Grouping Chained Commands](#grouping-chained-commands)
      - [Execute Commands in a Subshell](#execute-commands-in-a-subshell)
    - [Example Command Chaining Scenarios](#example-command-chaining-scenarios)
      - [SSH Command Chaining Example](#ssh-command-chaining-example)
      - [SSH Command Chaining Example With Error Redirection](#ssh-command-chaining-example-with-error-redirection)
      - [Example of Checking Apache2 HTTP Webserver Log for Errors](#example-of-checking-apache2-http-webserver-log-for-errors)
      - [Example of Backing Up a MySQL Database and Logging Results](#example-of-backing-up-a-mysql-database-and-logging-results)
      - [Example of Remotely Updating Your Server with SSH](#example-of-remotely-updating-your-server-with-ssh)

## Introduction

Often, you will encounter servers that do not have a display monitor attached to them (this is referred to as a headless server), or you may need to access your server when you are not physically in the same location as your server. This is were a very powerful and handy tool called SSH comes into play. SSH, or Secure Shell is a network protocol that provides a secure way for you to access your server remotely over a your Local Area Network, or even through the open internet (Wide Area Network) with some additional port forwarding rules added to your internet router and/or firewall.  SSH allows you terminal access to your server to execute commands remotely, and transfer files. Once you are familiar and comfortable using SSH, you might find it a much more convenient way to set up your servers without needing to have monitor always attached to them. The other advantage to this, is managing your normal tasks more often through the terminal only makes you less reliant on a GUI (graphical user interface). Running your servers with no GUI also frees up more RAM, and hard drive space with no need for a display server and desktop environment.

Aside from just being able to access your server for maintenance, if you are familiar with setting up users and permissions with individuals you trust, along with router and/or firewall port forwarding rules: you can also grant other people access to your server for shared file storage, or collaborating on projects. Being able to access your machine remotely is quite liberating, and also leaves you your tools at your own disposal where ever you have a ssh client. In the case of having one on your phone, you now have you servers with you EVERYWHERE!

So let's dive in and learn the basics of using SSH to access your server. For now, these instructions are more geared towards access your server over LAN (your Local Area Network) and not over the open internet. For accessing your servers over WAN (Wide Area Network / "The Internet") please read more about Router and Firewall Port Forwarding rules.

## Checking if SSH is Installed

First, login to your server, or open a new terminal if working from the desktop. Check if your server has a SSH server already running with either command(s):

```bash
sshd -V
```

If installed, you should see a similar output:

```shell
OpenSSH_9.6p1 Ubuntu-3ubuntu13.8, OpenSSL 3.0.13 30 Jan 2024
```

**OR alternately you can also check if SSH is installed with:**

```bash
sudo systemctl status ssh
```

If you have SSH installed, you should see a output similar to:

```shell
○ ssh.service - OpenBSD Secure Shell server  
    Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)  
  Active: inactive (dead)  
  TriggeredBy: ● ssh.socket  
    Docs: man:sshd(8)  
      man:sshd_config(5)  
```

**Otherwise if you receive a error message like:**

```shell
Unit ssh.service could not be found.
```

If you received this error, this means you either do not have it installed, OR there is a configuration preventing it from running. At this point you can also rule out ssh not being installed by typing the command into your terminal:

```bash
sudo dpkg -l | grep openssh-server
```

If you have it installed, you should see a output similar to:

```shell
ii  openssh-server   1:9.6p1-3ubuntu13.8    amd64 secure shell (SSH) server, for secure access from remote machines
```

Otherwise, if you once again receive an error, you need to install openssh-server on your machine.

## Installing openssh-server

This is very straight forward when using Ubuntu Server.  Just type the commands into your terminal to install the openssh-server and all of its dependencies:

```shell
sudo apt-get update && sudo apt-get install openssh-server
```

Your computer should check for the latest list of packages to install, and make sure all the dependencies are met to install openssh-server.  It should show you a list of all the packages to be installed, and the total amount of hard drive space required.  If you are good to move forward installing openssh-server, press Y and Enter in your terminal, or N to abort the process.

If you pressed N, and aborted the process: Cool, there's no reason for you to keep reading this portion of the manual. Otherwise, let's move forward with first connecting to your server over SSH without changing any configurations. Normally, after install your server should be running the openssh-server program.  We will need to figure out what your server's IP address (IPv4) is, and we will also need to make sure you have another computer or device with an SSH Client to connect to your SSH Server. If you don't have an SSH client already, they are widely available for pretty much every operating system, and even your smart phone.

## Obtain Your Servers IP Address

While your still in front of your server terminal, let's discover what your IP address is over your Local Area Network.  We will use this information for connecting to your server with a ssh client.

Type the following command into your terminal:

```bash
ip route
```

You should see a output similar to:

```shell
default via 192.168.1.254 dev wlp1s0 proto dhcp src 192.168.1.239 metric 600
192.168.1.0/24 dev wlp1s0 proto kernel scope link src 192.168.1.239 metric 600 
```

If you received an output similar to this, we can make the following observations:

1. "default via 192.168.1.254" (or any combination of 192.168.x.x commonly): This is your routers gateway address in which your device connects to the Local Area Network. Your routers address may not even have an IP address like 192.168.x.x, just remember that the address given after "default via" is your routers gateway address.

2. "src 192.168.1.239" (or any combination of 192.168.x.x commonly): This is the address assigned to your machine either as a static fixed IP address, or a dynamic address through DHCP.  In this case from the following output, we will note that 192.168.1.239 is the IP address that we will use to connect to the server over the network. Your IP address may not look like 192.168.x.x, just remember that the address given after "src" is your machine's IP address.

Great, you should have now discovered what your machines IP address is.  If you want to confirm this, you can quickly ping your IP address from your server with the following command, let it run for seconds and then press `Ctrl-C`:

```bash
ping 192.168.1.239
```

You should see an output like:

```shell
PING 192.168.1.239 (192.168.1.239) 56(84) bytes of data.
64 bytes from 192.168.1.239: icmp_seq=1 ttl=64 time=0.099 ms
64 bytes from 192.168.1.239: icmp_seq=2 ttl=64 time=0.085 ms
64 bytes from 192.168.1.239: icmp_seq=3 ttl=64 time=0.067 ms
64 bytes from 192.168.1.239: icmp_seq=4 ttl=64 time=0.068 ms
...
```

Now ping your routers IP address that you observed:

```bash
ping 192.168.1.254
```

You should see an output like:

```shell
PING 192.168.1.254 (192.168.1.254) 56(84) bytes of data.
64 bytes from 192.168.1.254: icmp_seq=1 ttl=64 time=14.8 ms
64 bytes from 192.168.1.254: icmp_seq=2 ttl=64 time=5.13 ms
64 bytes from 192.168.1.254: icmp_seq=3 ttl=64 time=6.40 ms
64 bytes from 192.168.1.254: icmp_seq=4 ttl=64 time=6.13 ms
...
```

Notice the difference in ping time between pinging the address for your server from your machine, and pinging your router.  Pinging your own machine results in a ping back at less than 1 millisecond. Pinging your router gives a result greater than 1 millisecond each time. You can easily confirm this is your Local Area Network address from such a quick ping result.  If you have not yet set up a static IP address on your network, you may need to run this command from time to time to rediscover what your machine's IP address is if your router is serving you a IP address dynamically through DHCP (Dynamic Host Configuration Protocol), your address is not guaranteed to stay the same on the network every time you connect.

## Setting Up a SSH Client to Connect to Your Server

Now we need to connect to our SSH server on our local area network.  We have all the information we need from the last section to make a connection.  But before we can remotely connect to our server from another computer in our network, we need to make sure that we can use a SSH Client on that computer.  Here are some simple steps for running a ssh client on your Windows/Mac/Linux computer and connecting to your server:

Recap of the information needed to connect to your remote server:

- **IP Address:** 192.168.1.239
- **User:** username

### Running a SSH Client on Windows

#### For Windows 10 or Later

1. Open Command Prompt or PowerShell:
   - Search for "**cmd**" or "**PowerShell**" in the Windows search bar and open it.
2. Connect to the server:

    In the command prompt or PowerShell window type:
  
     ```bash
     ssh username@192.168.1.239
     ```

3. You may receive a prompt, Accept the new connection or press 'Y'
4. Now you will be prompted for the password you set for the user on the remote server
5. You should then see your remote servers terminal appear in your Command Prompt or PowerShell window

#### For Earlier Versions of Windows

1. Download and install PuTTY from [https://www.putty.org](https://www.putty.org)
2. Open PuTTY after installation
3. In the "Host Name (or IP address)" field, enter the IP address of the remote server
4. Click "Open"
5. Accept the security alert if it pops up for creating a new connection
6. Enter your username and password
7. You should then be connected to your remote server

### Running a SSH Client on Mac

1. Open the Terminal
   - Go to **Applications > Utilities > Terminal**
2. Connect to the remote server:

    In the terminal type:

    ```bash
    ssh username@192.168.1.239
    ```

3. You may be prompted to accept a new connection, Press 'yes' and Enter
4. Enter your password
5. You should now be connected to your remote server

### Running a SSH Client on Linux

1. Open a terminal
2. Most Linux distributions have SSH by default, but if you need to enable it, type in your terminal:
  
    ```bash
    sudo systemctl enable ssh --now
    ```

3. Now connect to your remote server:

    In your terminal type:
  
    ```bash
    ssh username@192.168.1.239
    ```

4. Accept the new connection by pressing 'yes' and Enter
5. Enter your password
6. You should now be connected to your remote server

If these steps worked for you, and you connected to your remote server, congrats!  Let's move on to some useful applications of SSH.

## Transfer Files Securely With the SSH Protocol

To securely download or upload files to or from your server, we will use '**scp**' or secure copy which uses the SSH protocol. **scp** itself is a pretty straight forward program.  We will cover a few ways to utilize **scp** for file transfer.

**scp** uses the following command structure in your terminal:

```bash
scp [options] [source] [destination]
```

- **options:** Various options to modify the command's behavior (we'll get to some important ones later).
- **source:** The file or directory you want to copy.
- **destination:** Where you want to copy the file or directory.

### Downloading a File from Your Server

```bash
scp username@192.168.1.239:/home/username/textfile.txt .
```

- This tells ***scp*** to connect as ***username*** to ***192.168.1.239*** and download ***/home/username/textfile.txt*** and the '**.**' tells the program to copy the file to your current working directory.  You can specify a directory such as ***/home/remote_user/*** like:

    ```bash
    scp username@192.168.1.239:/home/username/textfile.txt /home/remote_user/
    ```

### Uploading a File to Your Server

```bash
scp /home/remote_user/textfile.txt username@192.168.1.239:/home/username/
```

- This tells ***scp*** to upload ***/home/remote_user/textfile.txt*** to ***/home/username/*** on your server with the username ***username*** at the address ***192.168.1.239***

### SCP Command Options

#### Copy a directory with the recursive option

```bash
scp -r /home/remote_user/text_files/ username@192.168.1.239:/home/username/
```

- This tells ***scp*** to connect as ***username*** to ***192.168.1.239*** and transfer the directory ***/home/remote_user/text_files/*** and **all** its contents to **/home/username/** on ***192.168.1.239***.
- This can also be used in reverse to download a whole directory from your server:

    ```bash
    scp -r username@192.168.1.239:/home/username/text_files/ /home/username/
    ```

#### Using scp when the server has password authentication disabled

For security measures, a lot of servers have password authentication disabled by default. In order to use this step, your public key needs to exist in your servers **authorized_keys** file. Refer to the SSH Keys section for more information on obtaining and setting keys.

We can include an Identity key in the scp options with the ```-i [path to key]``` option:

```bash
scp -i /home/remote_user/.ssh/id_rsa /home/remote_user/textfile.txt username@192.168.1.239:/home/username/
```

- This tells ***scp*** to include the **identity key** file ***/home/remote_user/.ssh/id_rsa*** with the ***-i*** option when connecting as ***username*** at ***192.168.1.239*** and transferring ***textfile.txt*** to the server.

#### Using scp on a remote server NOT using port 22

Sometimes administrators may set services up on different port numbers.  By default, SSH runs on port 22, but it can be set to a wide range of port numbers if need be. To tell scp to connect with SSH over a different port, we use the ```-P [port number]``` option.

```bash
scp -P 1022 username@192.168.1.239:/home/username/textfile.txt .
```

- This tells ***scp*** to connect as ***username*** at ***192.168.1.239*** using ***port 1022*** instead of the default 22, and download ***textfile.txt*** to our current working directory

#### Using scp with multiple options

Now let's wrap up this section by demonstrating using the **scp** command with multiple options by sending a folder of files to our server that only accepts connections from port 1022 and ONLY accepts ssh keys for connection instead of a password.  We could type this command into the terminal:

```bash
ssh -i /home/remote_user/.ssh/id_rsa -P 1022 -r /home/remote_user/text-files/ username@192.168.1.239:/home/username/
```

## Creating and Using SSH Keys

**What are SSH Keys?**

SSH Keys are like a digital lock and key for your servers SSH connection.  When you generate a new SSH key, you create a "key pair" consisting of two parts:

- **Public Key:** This is the key you initially share with servers to gain access, you only send this key to a remote server once. Remote servers will store your public key.

- **Private Key:** This is the key you use to connect and log in to a remote SSH server.  Your private key is used to prove that you have a matching public key on the remote server. This key file should be treated as sensitive and not shared with others.

You are not required to use a SSH key pair to connect to your server. You can still use a username and password to connect to your server. Although, using a SSH key to access your server, as long as you keep your private key secure, is a much more secure option for remotely accessing your server. With the proper configurations set up, which we will discuss later, you can log into your remote server much more quick and easy.

These next steps are assuming your using another terminal separate from your server, but you are able to generate keys on your server as well.  Just make sure after you generate your key, and copy your public key to your sshd configuration, that you copy your private key to another private drive, and remove it from your server.

### Generating a SSH Key Pair

In this step, open a new terminal session(It would be best to use a remote computer than your server).

We will use the **ssh-keygen** program for this next step.  Let's go over a few options that you can use when generating a new SSH *Key/Pair*.

When using the **ssh-keygen** command, one of the first options you will want to set is "-t" to set the type of encryption for your key.  You can use the following encryption types:

- **RSA** (Rivest–Shamir–Adleman) Encryption Algorithm:
  - It's based on the difficulty of factoring large numbers (multiplying primes is easy, reversing it is hard).
  - Common in secure communication, like SSH and website encryption.
  - RSA is slower than other encryption types, so it's often used for small amounts of data, like encrypting other encryption keys.
  - When using large key sizes, like 4096 bits, RSA remains a very strong form of encryption.
- **Ed25519** (Edwards-curve digital):
  - Very secure and efficient.
  - Often considered the most secure and recommended option.
- **ECDSA** (Elliptic Curve Digital Signature Algorithm):
  - Uses elliptic curve cryptography.
  - Offers good security with shorter key lengths.
  - Becoming increasingly popular.
- **DSA** (Digital Signature Algorithm):
  - Older algorithm.
  - Generally considered less secure than RSA or Ed25519.
  - Should be avoided.

After defining you encryption type, you'll want to specify the key length with the "-b" option.  In most cases 4096 bits is a secure key length. Let's generate a new RSA key with a length of 4096 bits.  We will need to input the following command into the terminal:

```bash
ssh-keygen -t rsa -b 4096
```

Next you will be prompted for a location and name of your private key.  Normally, by default **ssh-keygen** will prompt you to store your key pair in the directory '/home/username/.ssh/id_rsa'. You can use this option, or use the directory and filename of your choice.  In this example, we will use the default path of '/home/username/.ssh/id_rsa'.

After that, will be given the option of setting a passphrase for your key.  This is an optional feature, but it is generally considered more secure to have a passphrase attached to your key.  In the instance of your private key being compromised, whomever possesses your private key will still have a difficult time connecting to your server if you have a strong passphrase attached to your key.  If no passphrase is added to the key, the user is not prompted for a password when connecting to your ssh server, and is granted immediate remote access. While this may be convenient, and quicker not to enter a password, just be aware of the security implications if your key is compromised somehow. Enter a passphrase, or just hit enter on this step to skip using a password.

If done correctly, you should see that the program reports back to you the locations of both the private, and public key. You should also see a key fingerprint, and your key's random art image generated.  You can copy this text and save it all in a text file to keep with your private key.

Once done, you should have seen an output similar to this in your terminal for generating your new key/pair:

```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): /home/username/.ssh/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/username/.ssh/id_rsa
Your public key has been saved in /home/username/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:CyV4K9dXGaCyJBRlLjTjpYkN4qULF2QCixUMMMkmgw username@testmachine
The key's randomart image is:
+---[RSA 4096]----+
|E*++  .+o.       |
|B++ ...o.        |
|== o. =o.        |
|o = ..**.. .     |
| o ..=o*S . . +  |
|  . .o+... . =   |
|     .. .   o    |
|       . . .     |
|        . .      |
+----[SHA256]-----+
```

Now that we have successfully generated a SSH key/pair, we can copy it to our server.

### Copying Your SSH Key To Your Server

#### Method 1 - Using ssh-copy-id (Recommended)

Earlier we covered finding the local ip address of your server.  This is where it will come it. For our purposes the ip address of the server is 192.168.1.239, and there is a user on that server with the user name: username (Really creative, I know). We also know that right now, on our computer that we are using to gain access to our server, our public and private key are stored in: '/home/username/.ssh/'. This is the important information we need to transfer our public key to the server.

We will do this with the following command:

```bash
ssh-copy-id -i /home/username/.ssh/id_rsa.pub username@192.168.1.239
```

This will tell ssh-copy-id to copy the identity file '/home/username/id_rsa.pub' to be used for the user username on your server located at the address of 192.168.1.239.  Now you should be prompted to enter the password for the user name you are connecting as on your remote server.  In this example, our remote
SSH server has a user named username that we are connecting as.

If done successfully, you should be prompted to enter a the passphrase for username on your remote SSH server.  You should see an output similar to this if you entered the correct key path, and passphrase:

```shell
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/username/.ssh/id_rsa_.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
username@192.168.1.239's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'username@192.168.1.239'" and check to make sure that only the key(s) you wanted were added.
```

Your key should have been copied to the authorized_keys file in /home/\<Server Username>/.ssh/authorized_keys

We can now log into the server using the newly generated key:

```bash
ssh -i /home/username/.ssh/id_rsa username@192.168.1.239
```

Note that if you set a password for your key (which is recommended, you should still be prompted for a passkey)

#### Method 2 - Manually copying your public key to your server

If you are unable to use ssh-copy-id or just want to manually copy your public key file to your server, we must first copy the content of our /home/username/.ssh/id_rsa.pub file.

You can display the contents to the terminal with:

```bash
cat /home/username/.ssh/id_rsa.pub
```

Or open the file with a text editor:

```bash
nano /home/username/.ssh/id_rsa.pub
```

Copy the contents of the entire text file, either in the editor, or what was displayed in the terminal.

Now log into your SSH server like you normally would first:

```bash
ssh username@192.168.1.239
```

Next open your servers authorized_keys file.

```bash
nano /home/<Server Username>/.ssh/authorized_keys
```

Simply paste the text content you copied from your id_rsa.pub file into a new line in your *auhtorized_keys* file. The key is very long, and should have pasted as 1 single line to work correctly.

Save the contents of the file, and exit. (**Ctrl-o** and then **Ctrl-x** with nano). Now log out of your server,and try logging in with your key now:

```bash
ssh -i /home/username/.ssh/id_rsa username@192.168.1.239
```

If you set a passphrase for your key, you should be prompted to enter the passphrase next:

```shell
Enter passphrase for key 'id_rsa':
```

Or, if you did not set a passphrase for your key, you should be automatically logged in.

If you still see the following prompt when trying to log in with your key, something went wrong:

```shell
username@192.168.1.239's password: 
```

If this is happening, make sure you are copying the contents of your id_rsa.pub file to your server's *authorized_keys* file. Make sure the key is one continuous line in the file. When you log in with ssh and the **'-i'** option, make sure you are providing the *id_rsa* file, not the *id_rsa.pub* file.

This wraps up generating SSH Keys, and copying them to your server.

## SSH Client Configuration

Now that we've covered logging into your remote server, and generating an SSH key for added security, let's put those both together for an easier experience logging in or sending commands to your remote server.

First let's check on your client computer (the one you'll be logging into your remote server from) and see if your SSH directory has a config file.  Open up a new terminal and type:

```bash
cd .ssh/ && ls
```

If 'ls' lists a file named 'config' in the directory, ignore this next step. If the config file is not present, we can simply create the new config file in your /home/username/.ssh/ directory:

```bash
touch config
```

Now that there is a config file present, lets go ahead and cover what we'll be adding to the configuration file. First we can create a name for our connection to the server, we will need the *ip address*, *username*, *port number* (optional), and *identity key file* (optional). With these, we can easily connect to our server or perform any operations with SSH by just referring to the name that we give the configuration.  So instead of connecting to our remote server with the command:

```bash
ssh username@192.168.1.239
```

**...Or...**

```bash
ssh -p 22 -i /home/username/.ssh/id_rsa username@192.168.1.239
```

We can edit the config file we created in the .ssh directory:

```bash
nano config
```

Inside that file we can now either create a new entry below and existing entries, or add a new one in the blank config file.  This will give all the relevant information needed for connection to our remote server:

```shell
Host remote_server
  HostName 192.168.1.239
  Port 22
  User username
  IdentityFile /home/username/.ssh/id_rsa
```

Now save and close the file with **Ctrl-o** and **Ctrl-x**.  You should now be able to connect to your SSH server with the command:

```bash
ssh remote_server
```

That simplifies logging into your SSH server, and you can add multiple configuration profiles to your SSH configuration for logging into different servers that may contain different ports, usernames, or key files.  Let's cover the properties we set for the configuration file:

- **Host**: The alias or name you would like to give that SSH connection
- **Port**: By default you do not need to include this if the server runs on port 22.  If the remote server ran on another port such as 1022, you could set the port to that number.
- **User**: The username for that SSH connection
- **IdentityFile**: This is optional, but you can set the path for your identity key file here

### Advanced SSH Configuration Options

#### Host Patterns

You can user wildcards (*, ?) to apply settings to multiple hosts:

```shell
Host *.my-server.net
  HostName 192.168.1.239
  User username
  Port 22
```

Now if you had two different servers like **s1.my-server.net** and **s2.my-server.net**, this configuration would apply to both of those servers. You could connect to both servers:

```bash
ssh s1.my-server.net
```

```bash
ssh s2.my-server.net
```

#### ProxyJump

Allows you to jump through an intermediate "jump" host

```shell
ProxyJump jump-server
```

#### ForwardAgent

Forwards your agent, allowing you to use your local keys on a remote SSH server

```shell
ForwardAgent yes
```

#### LocalForward

Allows you to forward a local port to a remote port.  Refer to SSH Tunneling and Local forwarding.

```shell
LocalForward 9000 localhost:8080
```

#### RemoteForward

Allows you to forward a remote port to a local port.  Refer to SSH Tunneling and Remote forwarding.

```shell
RemoteForward 8080 localhost:9000
```

#### ServerAliveInternal

Keeps the SSH connection alive by sending keepalive messages.

```shell
ServerAliveInternal 60
```

#### Example Advanced Configurations

```shell
Host *.internal
  User devops
  ProxyJump jump-host
  ForwardAgent yes
  ServerAliveInterval 60

Host web-server
  Hostname 192.168.1.50
  LocalForward 8080 localhost:3000
```

This concludes the SSH Client configurations.

## SSH Server Configurations

### Disabling Password Based Authentication

If you have generated keys for accessing your SSH server, you can further secure it by disabling the password authentication feature on your SSH server's configuration file, and force your SSH server to rely solely on key based authentication.  This can stop individuals from trying to gain access to your SSH server by brute forcing as many passwords as they can to try and log in to your server.  One scenario where this would not be recommended is if you do not have physical access to your server. If you disable password login on your server, and lose your key, you will no longer be able to login over SSH until you revert settings back, or manually copy another public key into your server's authorized_keys file. Even if you're locked out from being able to log in through SSH, you should still be able to physically log into your server just fine.

To disable password authentication on your server, first log in to your server.

```bash
ssh -i /home/username/.ssh/id_rsa username@192.168.1.239
```

Now, we need to access your ssh servers configuration file.  This file is located at `/etc/ssh/sshd_config`.  Modifying this file requires root access.  Let's open the file in a text editor:

```bash
sudo nano /etc/ssh/sshd_config
```

First, inside the sshd_config file, find the line that has the text:

```shell
PasswordAuthentication yes
```

Change that line to:

```shell
PasswordAuthentication no
```

Next, find a setting for PubkeyAuthentication. It may be already commented out with a '#' symbol in your configuration file, like:

```shell
#PubkeyAuthentication yes
```

**...Or...**

```shell
PubkeyAuthentication no
```

Add or change this line in your sshd_config file:

```shell
PubkeyAuthentication yes
```

Now save the changes, and exit the text editor. (**Ctrl-o**, and **Ctrl-x**)

Next we need to restart the SSH server for the new configuration changes to take effect.

```bash
sudo systemctl restart sshd
```

**...Or..**

```bash
sudo systemctl restart ssh
```

**Note**: Check the `/etc/ssh/sshd_config.d` directory for other configuration files.  If they contain the line `PasswordAuthentication yes` you need to comment it out as `#PasswordAuthentication yes` to make sure this feature stays disabled. Otherwise, this will **override** the line from your original sshd_config file, and still allow password based authentication.  If you are using Ubuntu Server, check if you have the file `/etc/ssh/sshd_config.d/50-cloud-init.conf`. By default, password authentication is enabled there also.

Now log out of your server, and try logging in without supplying your key:

```bash
ssh username@192.168.1.239
```

If your new settings are correct, you should be immediately rejected from your server with the following message:

```shell
username@192.168.1.239: Permission denied (publickey).
```

Now try logging in supplying the ssh key you generated:

```bash
ssh -i /home/username/.ssh/id_rsa username@192.168.1.239
```

If you were rejected from your server for connected without a key, but were able to connect with your SSH key, congratulations! Your SSH server is now much more secure with password authentication disabled.

### Enable or Disable Access To Your Server by User

#### Enable Access to Certain Users

First, connect to your SSH server:

```bash
ssh -i /home/username/.ssh/id_rsa
```

Then open your sshd_config file to edit:

```bash
sudo nano /etc/ssh/sshd_config
```

To only allow certain users log into your server over SSH, add the following line:

```shell
AllowUsers user1 user2 user3
```

Make sure to separate usernames with a space only, no commas

If you want to restrict which computers your users on the network can connect to your SSH server, you can specify the user and IP address strictly, and that user will only be able to connect from the IP address that you add in your **sshd_configuration** file such as:

```shell
AllowUsers user1@192.168.1.237 user2@192.168.1.236 user3@192.168.1.235
```

Now these users are restricted to only connecting to your SSH server from the addresses you supplied:

- **user1** can only connect from the computer with the IP address **192.168.1.137**
- **user2** can only connect from the computer with the IP address **192.168.1.136**
- **user3** can only connect from the computer with the IP address **192.168.1.135**

Attempts to log in from any other computer, and as any other user will be denied access.

#### Disable Access to Certain Users

To disable certain users from being able to log in over SSH, add the following line:

```shell
DenyUsers user4 user5
```

Now save and exit the text editor with **Ctrl-o** and **Ctrl-x**.

Restart your SSH server to apply the new configuration changes.

```shell
sudo systemctl restart ssh
```

If you have multiple users set up on your server, try allowing some, and denying some access.  Try to log in to your server and make sure the changes took effect properly.

Any other rules that apply to trying to allow or deny access by IP address only should be handled by changing your firewall settings, such as **ufw** which should be running on your Ubuntu Server by default.

## SSH Tunnels

What is "tunneling" with SSH, and how could this be a useful feature for your server?

SSH tunneling works 3 ways: locally, remotely, or dynamically.  By using a SSH tunnel, you can access services one a secure network that is normally not accessible by the internet. You can also securely use the internet over public Wi-Fi through a SSH tunnel, by forcing all of your internet traffic through a SSH connection to your remote server at home, protecting your information. You may be inside of a network that restricts access to certain websites that you would still like to gain access to; you can establish a secure SSH connection to your remote server and still gain access to those sites while still connected to the restrictive network.  Or vice versa, let's say you have a server or machine inside of a restrictive network, and you would like to tunnel a connection to a service running your computer outside of that network, you can share services from a port on your local machine to a defined port on your server behind the restrictive network.  Let's cover how we can achieve this with these **three methods**:

### Tunneling Locally

#### Local Tunneling Syntax

```shell
ssh -L local_port:remote_host:remote_port user@remote_server
```

- **-L** : defines local port forwarding
- **local_port**: The port on your local machine that you want to forward
- **remote_host**: The hostname or IP address of the destination server (accessible from the remote server)
- **remote_port** : The port on the destination server that you want to access
- **user@remote_server**: Your username and the address of your Ubuntu Server

#### Local Tunneling Uses

Let's say your remote server has web service that it only serves ***locally*** on ***port 8080***, and you would like to be able to access that service on your machine on port ***8888***.  You could set up a **tunnel** to ***username@192.168.1.239*** to forward its ***port 8080*** on ***localhost***, since it is accessing a service on that machine itself, and you could access that service on your machine through port ***8888***.

You can set up a local tunnel to your server with the following command:

```shell
ssh -L 8888:localhost:8080 username@192.168.1.239
```

Now if you pointed your web browser to **<http://localhost:8888/>** you would be accessing **<http://192.168.1.239:8080/>**

### Tunneling Remotely

#### Remote Tunneling Syntax

```shell
ssh -R remote_port:local_host:local_port user@remote_server
```

- **-R** : defines remote port forwarding
- **remote_port**: The port on the remote server that you want to forward
- **local_host**: The hostname or IP address of the destination server (accessible from your local machine)
- **local_port**: The port on the destination server that you want to access
- **username@remote_server**: Your username and the address of the remote server

#### Remote Tunneling Uses

In this scenario, lets say you have web service running on your *local machine* (**localhost**) that runs on port ***8080***, and you want your ***remote server*** *to be able to access it* ***locally*** *on its own port* ***4444***.

You can set up a remote tunnel on your server with the following command:

```bash
ssh -R 4444:localhost:8080 username@192.168.1.239
```

Your remote server (**username@192.168.1.239**) can connect **locally** to port **4444**, and its traffic is forwarded to **your local machine's** web service running on port **8080**.

### Tunneling Dynamically

#### Dynamic Tunneling Syntax

```shell
ssh -D local_port user@remote_server
```

- **-D** : defines dynamic port forwarding
- **local_port**: The port on your local machine that will be used for the SOCKS proxy
- **user@remote_server**: Your username and the address of the remote server

#### Dynamic Tunneling Uses

In this scenario, we are outside of our home network.  We know that the external IP address of our local network at home is 77.22.142.66, and we configured our router to forward SSH traffic (port 22) to our server in the network whose address is 192.168.1.239 in case we needed to access it remotely outside of the local area network.  In this situation, we only have internet access through Wi-Fi that we are not certain is secure.  We can force all of our internet traffic through an encrypted connection to our remote server, and keep all of our traffic encrypted and hidden away from the Wi-Fi network we are having to use for internet access.  We will establish this proxy server on port 8888 on our machine. Then in our browser or network settings, we will set our SOCK5 proxy server to localhost at port 8888.  Now we can safely access the internet on an unsecure connection.

You can setup a dynamic tunnel on your local machine with the following command:

```bash
ssh -D 8888 username@77.22.142.66
```

We establish a connection to our router at 77.22.142.66 that forwards SSH traffic to 192.168.1.239, and define the connection as dynamic port forwarding. This connection is stored on our local machine at port 8888.  We can then set browser or network proxy settings to force all traffic on multiple ports through port 8888 as a encrypted connection.

## Sending Shell Commands to Your Server via SSH

SSH has the ability to connect, and dispatch shell commands to your remote server to help save time.  You can write bash scripts to automate tasks such as running updates or checking the status of certain services running on your server.

This is simply done by using the SSH command, followed by any options, and then user and host address, and finally a set of parenthesis or quotes with your desired shell command.  Your computer will connect to the server, send the commands, and then terminate the connection:

```shell
ssh [options] [host] "[shell scripts]"
```

### Example SSH Remote Shell Command

```bash
ssh -p 1022 -i ~/.ssh/id_rsa username@192.168.1.239 "echo 'Created from my client device' > ~/remote_file.txt"
```

These commands can quickly become long winded.  It would be best to store as much information about your host server in your SSH configuration file to help shorten the command to something like:

```bash
ssh remote_server "echo 'Created from my client device' > ~/remote_file.txt"
```

This command would have created a file called remote_file.txt in your users home directory on the server, containing the text "Created from my client device".  That would cover the basics of sending commands to your server, but let's step it up a notch and learn how to add some logic to the way that your commands are handled, and the ability to write logs based on the commands you send to the server, and record them locally on your client computer.

### Command Chaining and Control

We can control what happens with different shell commands with 3 operators:

#### Sequencing

**Sequencing (' ; ')**: Executes commands in sequence, regardless of success or failure. Each individual command is separated with a semicolon ';'

```bash
`ssh username@remote_host "command_1; command_2; command_3"`
```

1. **command_1** will execute.
2. **command_2** will execute **REGARDLESS if command_1 failed or succeeded**
3. **command_3** will execute **REGARDLESS if command_2 failed or succeeded**

#### Conditional AND

**Conditional AND (' && ' )**: Executes the next command only if the previous command succeeds (Returns a 0 exit code)

```bash
ssh username@remote_host "command_1 && command_2 && command_3"
```

1. **command_1** will execute
2. **command_2 will ONLY** execute if **command_1 succeeds**
3. **command_3 will ONLY** execute if **command_2 succeeds**

#### Conditional OR

**Conditional OR (' || ')**: Executes the next command only if the previous command fails (Returns a non-zero exit code)

```bash
ssh username@remote_host "command_1 || command_2"
```

1. command_1 will execute
2. command_2 will **ONLY** execute if command_1 **FAILS**

#### Combining Conditional Operators

Operators can be combined to create more sophisticated workflows:

```bash
ssh username@remote_host "command_1 && command_2 || command_3"
```

1. **command_1** will execute
2. **command_2** will **ONLY** execute if **command_1 SUCCEEDS**
3. if **command_1 FAILS**, **command_3 will execute**

#### Grouping Chained Commands

You can use parentheses to group commands and control the order of execution:

```bash
ssh username@remote_host "(command_1 && command_2) || command_3"
```

1. **command_1** will execute
2. **command_2** will execute **ONLY if command_1 succeeds**. **command_2** can still **fail or succeed**
3. **command_3** will execute **ONLY if both command_1 and command_2 fail**

#### Execute Commands in a Subshell

You can also execute commands in a subshell using parenthesis or backticks.  This creates a separate environment for the commands:

```bash
ssh username@remote_host '$(command_1; command_2)'
```

1. Once connected to your server, **another shell environment opens**
2. **command_1** is executed
3. **command_2** is executed **REGARDLESS if command_1 fails or succeeds**

### Example Command Chaining Scenarios

#### SSH Command Chaining Example

Now lets imagine a scenario where we want to check if a application you wrote is running, and if not, restart the service. Finally, we will log the status of that service on the server.

We can write that command into our terminal as:

```bash
ssh username@remote_host '(sudo systemctl is-active my-app.service || sudo systemctl restart my-app.service) && sudo systemctl status my-app.service >> app-status.log'
```

1. This connects to remote host and first checks if my-app.service is active
2. If my-app.service is not active, the system restarts the application
3. After the application restart, the system checks the status of the application
4. The results from that status check are logged to a file called app-status.log in the users' home directory

**You can alternatively use an If statement in your shell command to repeat the same logic:**

```bash
ssh username@remote_host '(if ! sudo systemctl is-active my-app.service; then sudo systemctl restart my-app.service; sudo systemctl status my-app.service >> app-status.log)'
```

- '!' can be read as NOT
- if my-app.service is NOT active
- then restart my-app.service
- Check the status of my-app.service
- Log the results to app-status.log in the home directory

#### SSH Command Chaining Example With Error Redirection

```bash
ssh username@remote_host 'sudo systemctl is-active my-app.service || (sudo systemctl restart my-app.service && sudo systemctl status my-app.service >> app-status.log || echo "Restart failed." >> app-status.log)'
```

- If my-app.service is active, nothing happens.
- If it is not active, restart the service
- If the service restarts, record the status to app-status.log
- If the restart fails, write "Restart failed." to the app-status.log

#### Example of Checking Apache2 HTTP Webserver Log for Errors

Here is an example of checking the Apache2 HTTP Webserver journal for errors, and recording them to a log and displaying them at the same time using 'tee' for you to investigate:

```bash
ssh username@remote_host 'sudo journalctl -u apache2.service | grep -i error | tee apache_errors.log'
```

#### Example of Backing Up a MySQL Database and Logging Results

Here is a example of creating a Bash script to backup a mysql database, and write the results to a log. We will write this script using a SSH key with no password so it can automatically connect to the server and execute the database backup script.

**First we will write a new script:**

```bash
nano db_backup.sh
```

**And add these instructions to our script:**

```bash
#!/usr/bin/bash
# Create timestamp for backup
timestamp=$(date +%Y%m%d_%H%M%S)
# Create a backup of the database
mysqldump -u <username> -p <password> <database_name> > /path/to/backup/db_backup_$timestamp.sql
# Compress the backup file
gzip /path/to/backup/db_backup_$timestamp.sql
```

Save and close this shell script with **Ctrl-o**, **Ctrl-x**. Now we will copy it over to the remote_server:

```bash
scp db_backup.sh username@remote_server:~/
```

**Now we can execute the backup script remotely on the server and log the results on your local machine:**

```bash
ssh username@remote_host "/home/username/db_backup.sh" &> db_backup.log
```

**If you wanted the results to only be logged on the server:**

```bash
ssh username@remote_host "/home/username/db_backup.sh &> db_backup.log"
```

#### Example of Remotely Updating Your Server with SSH

Now let's create one more scenario where we want to remotely update our server.  This can be risky if your update hangs, or there is a prompt to continue or make a decision during the update.  If you have no way to reclaim this terminal session, and it hangs, and you restart your server not knowing what to do, you can potentially cause yourself some issues.  There is a great little program you can install called 'screen'. Screen starts a new terminal session that you can detach from, or re-attach to later.  If you are running an update on your remote server from another location, and something with your network connection or server network connection goes wrong; you can still reclaim that screen session, and get things back working again if no one decided to restart the server.

Screen is a powerful tool, so we will only cover a few basics for this SSH session.

**First let's install screen on our remote server:**

```bash
sudo apt-get update && sudo apt-get install screen -y
```

After the install completes, lets give screen a try.  In your terminal type:

```bash
screen
```

Alright, nothing too exciting happened.  The screen blanked out, and you are either in another empty terminal session, or you have a display message from the software asking you to press space to keep reading, or return to go to your new terminal session.

We can tell we are running a screen session by typing:

```bash
screen -ls
```

You should see a output in your terminal like:

```shell
There is a screen on:
  28674.pts-0.my_server       (03/19/25 05:34:13)     (Attached)
1 Socket in /run/screen/S-ubuntu.
```

Your screen session has an ID of 28674, and at the end of the line (Attached) let's you know that that is the current session you are working in.  You can have multiple screen sessions.  Now, if we wanted to disconnect from this session, but still keep it open to continue working in; we can simply press **Ctrl-A** and then press **' d '**. This will detach you from that screen session, and return you to your original terminal session. Check your screen sessions again:

```bash
screen -ls
```

You should see a terminal output like:

```shell
There is a screen on:
  28674.pts-0.rpi02 (03/19/25 05:34:13) (Detached)
1 Socket in /run/screen/S-ubuntu.
```

We can now see that the session with the ID 28674 is (Detached).  When we are ready to get back to work in that terminal session, we can easily reattach to that session by using its ID.

```bash
screen -r 28674
```

Once you are done working in that session, and no longer need it; just type:

```bash
exit
```

You should see a result like:

```shell
[screen is terminating]
```

And you can also verify there are no more sessions by checking again:

```bash
screen -ls
```

Terminal Output:

```shell
No Sockets found in /run/screen/S-ubuntu.
```

When working with multiple screen sessions on a server, you can easily distinguish each screen session from each other, by naming each session when you start in, instead of trying to remember each session's ID (which you can do if you really want).  Starting a new screen session with a name is as easy as:

```bash
screen -S update
```

Now check the results:

```bash
screen -ls
```

Terminal Output:

```shell
There is a screen on:
  28800.update    (03/19/25 05:52:06)     (Attached)
1 Socket in /run/screen/S-ubuntu.
```

Now if you needed to detach from that screen with Ctrl-A, and then press 'd'; you can always reattach with that session with:

```bash
screen -r update
```

Great!  This is a great way for creating a remote session that can be recovered locally if something goes wrong. I recommend you use software or something similar to screen if you are going to conduct updates on your server remotely.  Let's tie all of this together, and create a little script to perform a remote update on our server in a screen session, and then terminate the session when the update is finished:

```bash
ssh username@remote_server "screen -S update -dm bash -c 'sudo apt-get update && sudo apt-get upgrade -y && exit'"
```

- This remotely starts a screen session named "update" first
- The '-d' option starts the session detached.
- The '-m' option forces the screen creation.
- The bash -c '...commands...' executes the commands in the single quotes in the new screen session created.

Be sure to check in on your server periodically and make sure a update is not waiting for you to make a decision
to continue.
