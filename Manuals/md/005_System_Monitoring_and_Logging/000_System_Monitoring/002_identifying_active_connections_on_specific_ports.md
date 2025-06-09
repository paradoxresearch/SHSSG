# Identifying Active Connections On Specific Ports

## Contents

- [Identifying Active Connections On Specific Ports](#identifying-active-connections-on-specific-ports)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Using netstat](#using-netstat)
  - [Using ss (Socket Statistics)](#using-ss-socket-statistics)
  - [Using lsof](#using-lsof)

## Introduction

You can use command-line tools to inspect the network connections and filter for those related to the port you want to monitor. This gives you a real-time snapshot.

## Using netstat

This command displays network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.  

```bash
netstat -an | grep ':80' | grep 'ESTABLISHED' | wc -l
```

- `netstat -an`: Shows all active network connections and listening ports.
- `grep ':80'`: Filters the output to show connections on port 80 (the default HTTP port).   
- `grep 'ESTABLISHED'`: Further filters to show only currently active (established) connections.
- `wc -l`: Counts the number of lines in the output, which corresponds to the number of active connections.

For `HTTPS` connections, you would use port **443**:

```bash
netstat -an | grep ':443' | grep 'ESTABLISHED' | wc -l
```

Here is an example output without the line count:

```shell
tcp        0      0 192.168.192.226:49394   34.107.243.93:443       ESTABLISHED
tcp        0      0 192.168.192.226:32842   142.250.72.174:443      ESTABLISHED
```

## Using ss (Socket Statistics)

`ss` is another utility to investigate sockets. It can provide more detailed information than `netstat` and is often preferred on newer Linux systems.  

```bash
ss -o state established '( dport = :80 or sport = :80 )' | wc -l
```

- `ss -o state established`: Shows only established TCP connections and includes timer information.   
- `'( dport = :80 or sport = :80 )'`: Filters for connections where either the **destination port** (`dport`) or the **source port** (`sport`) is 80. This accounts for both incoming and outgoing connections on that port.  
- `wc -l`: Counts the lines, giving the number of established connections on port 80.

Similarly, for `HTTPS` on port 443:

```Bash
ss -o state established '( dport = :443 or sport = :443 )' | wc -l
```

If your output is including the count of the row of header labels, you can subtract from that number, and output an accurate number:

```bash
echo $(( $(ss -o state established '( dport = :443 or sport = :443 )' | wc -l) - 1 ))
```

> This bash command just subtracts 1 from the original output of the line count (`wc -l`) of the `ss` command, and outputs that number.

Without doing a line count, the `ss` utility can also give you some great information about the connections on a certain port:

```bash
ss -o state established '( dport = :443 or sport = :443 )'
```

Output:

```shell
Netid  Recv-Q  Send-Q      Local Address:Port        Peer Address:Port   Process                         
tcp    0       0         192.168.192.226:49394      34.107.243.93:https   timer:(keepalive,8min26sec,0)  
tcp    0       0         192.168.192.226:54590     34.149.100.209:https                                  
tcp    0       0         192.168.192.226:32842     142.250.72.174:https 
```

## Using lsof
 
`lsof` **(List Open Files)** is a powerful command that lists all open files and the processes that opened them. In Linux, *everything is treated as a file*, including network sockets.   

You can use `lsof` to filter for network connections related to a specific port:

```bash
sudo lsof -i :443 -n | grep ESTABLISHED | wc -l
```

- `sudo lsof -i :443`: Lists all network files using port 443.
- `-n`: Prevents DNS resolution, which can speed up the output.   
- `grep ESTABLISHED`: Filters for TCP connections in the ESTABLISHED state. lsof might show different states for TCP (like LISTEN, SYN_SENT, etc.).  
- `wc -l`: Counts the lines. Again, check for a header line.  

For both TCP and UDP on a port:

```bash
sudo lsof -i UDP:443 -i TCP:443 -n | grep ESTABLISHED | wc -l
```

You might need to adjust the filtering based on the output to get only active connections.

Output without line count:

```shell
firefox-b 285525 username  146u  IPv4 644828      0t0  TCP 192.168.192.226:49394->34.107.243.93:https (ESTABLISHED)
firefox-b 285525 username  157u  IPv4 668884      0t0  TCP 192.168.192.226:49882->142.250.141.84:https (ESTABLISHED)
```