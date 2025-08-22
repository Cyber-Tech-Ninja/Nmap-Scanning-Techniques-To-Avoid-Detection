# Nmap-Scanning-Techniques-To-Avoid-Detection By Firewall
Nmap Scanning Techniques To Avoid Detection
When scanning a host behind a firewall, the firewall will usually detect and block port scans. This situation would require you to adapt your network and port scan to evade the firewall. A network scanner like Nmap provides few features to help with such a task. In this room, we group Nmap techniques into three groups:

Evasion via controlling the source MAC/IP/Port
Evasion via fragmentation, MTU, and data length
Evasion through modifying header fields
Nmap allows you to hide or spoof the source as you can use:

Decoy(s)
Proxy
Spoofed MAC Address
Spoofed Source IP Address
Fixed Source Port Number

Decoy
Hide your scan with decoys. Using decoys makes your IP address mix with other “decoy” IP addresses. Consequently, it will be difficult for the firewall and target host to know where the port scan is coming from. Moreover, this can exhaust the blue team investigating each source IP address.
Using the -D option, you can add decoy source IP addresses to confuse the target. Consider the following command, nmap -sS -Pn -D 10.10.10.1,10.10.10.2,ME -F MACHINE_IP. The Wireshark capture is shown in the following figure.
You can also set Nmap to use random source IP addresses instead of explicitly specifying them. By running nmap -sS -Pn -D RND,RND,ME -F MACHINE_IP, Nmap will choose two random source IP addresses to use as decoys. Nmap will use new random IP addresses each time you run this command. In the screenshot below, we see how Nmap picked two random IP addresses in addition to our own, 10.14.17.226.

Proxy
Use an HTTP/SOCKS4 proxy. Relaying the port scan via a proxy helps keep your IP address unknown to the target host. This technique allows you to keep your IP address hidden while the target logs the IP address of the proxy server. You can go this route using the Nmap option --proxies PROXY_URL. For example, nmap -sS -Pn --proxies PROXY_URL -F MACHINE_IP will send all its packets via the proxy server you specify. Note that you can chain proxies using a comma-separated list. nmap -sS -Pn --proxies 10.10.13.37 MACHINE_IP

Spoofed MAC Address
Spoof the source MAC address. Nmap allows you to spoof your MAC address using the option --spoof-mac MAC_ADDRESS. This technique is tricky; spoofing the MAC address works only if your system is on the same network segment as the target host. The target system is going to reply to a spoofed MAC address. If you are not on the same network segment, sharing the same Ethernet, you won’t be able to capture and read the responses. It allows you to exploit any trust relationship based on MAC addresses. Moreover, you can use this technique to hide your scanning activities on the network. For example, you can make your scans appear as if coming from a network printer.

Spoofed IP Address
Spoof the source IP address. Nmap lets you spoof your IP address using -S IP_ADDRESS. Spoofing the IP address is useful if your system is on the same subnetwork as the target host; otherwise, you won’t be able to read the replies sent back. The reason is that the target host will reply to the spoofed IP address, and unless you can capture the responses, you won’t benefit from this technique. Another use for spoofing your IP address is when you control the system that has that particular IP address. Consequently, if you notice that the target started to block the spoofed IP address, you can switch to a different spoofed IP address that belongs to a system that you also control. This scanning technique can help you maintain stealthy existence; moreover, you can use this technique to exploit trust relationships on the network based on IP addresses.

To mislead the opponent, you decided to make your port scans appear as if coming from a local access point that has the IP address 10.10.0.254. What option needs to be added to your Nmap command to spoof your address accordingly?

Fixed Source Port Number
Use a specific source port number. Scanning from one particular source port number can be helpful if you discover that the firewalls allow incoming packets from particular source port numbers, such as port 53 or 80. Without inspecting the packet contents, packets from source TCP port 80 or 443 look like packets from a web server, while packets from UDP port 53 look like responses to DNS queries. You can set your port number using -g or --source-port options.
The following Wireshark screenshot shows a Nmap scan with the fixed source TCP port number 8080. We have used the following Nmap command, nmap -sS -Pn -g 8080 -F MACHINE_IP. You can see in the screenshot how it is that all the TCP connections are sent from the same TCP port number.

This is a quick summary of the Nmap options discussed in this task.
Evasion Approach	Nmap Argument
Hide a scan with decoys	-D DECOY1_IP1,DECOY_IP2,ME
Hide a scan with random decoys	-D RND,RND,ME
Use an HTTP/SOCKS4 proxy to relay connections	--proxies PROXY_URL
Spoof source MAC address	--spoof-mac MAC_ADDRESS
Spoof source IP address	-S IP_ADDRESS
Use a specific source port number	-g PORT_NUM or --source-port PORT_NUM

#2 Evading via modifying header fields
Nmap allows you to control various header fields that might help evade the firewall. You can:

Set IP time-to-live
Send packets with specified IP options
Send packets with a wrong TCP/UDP checksum
Set TTL
Nmap gives you further control over the different fields in the IP header. One of the fields you can control is the Time-to-Live (TTL). Nmap options include --ttl VALUE to set the TTL to a custom value. This option might be useful if you think the default TTL exposes your port scan activities.

In the following screenshot, we can see the packets captured by Wireshark after using a custom TTL as we run our scan, nmap -sS -Pn --ttl 81 -F 10.201.35.151. As with the previous examples, the packets below are captured on the same system running Nmap.

Set IP Options
One of the IP header fields is the IP Options field. Nmap lets you control the value set in the IP Options field using --ip-options HEX_STRING, where the hex string can specify the bytes you want to use to fill in the IP Options field. Each byte is written as \xHH, where HH represents two hexadecimal digits, i.e., one byte.

A shortcut provided by Nmap is using the letters to make your requests:

R to record-route.
T to record-timestamp.
U to record-route and record-timestamp.
L for loose source routing and needs to be followed by a list of IP addresses separated by space.
S for strict source routing and needs to be followed by a list of IP addresses separated by space.
The loose and strict source routing can be helpful if you want to try to make your packets take a particular route to avoid a specific security system.

Use a Wrong Checksum
Another trick you can use is to send your packets with an intentionally wrong checksum. Some systems would drop a packet with a bad checksum, while others won’t. You can use this to your advantage to discover more about the systems in your network. All you need to do is add the option --badsum to your Nmap command.

Using nmap -sS -Pn --badsum -F 10.201.35.151, we scanned our target using intentionally incorrect TCP checksums. The target dropped all our packets and didn’t respond to any of them.

This is a quick summary of the Nmap options discussed in this task.

Evasion Approach	Nmap Argument
Set IP time-to-live field	--ttl VALUE
Send packets with specified IP options	--ip-options OPTIONS
Send packets with a wrong TCP/UDP checksum	--badsum

#EVASION USING PORT HOPPING
Three common firewall evasion techniques are:

Port hopping
Port tunneling
Use of non-standard ports
Port hopping is a technique where an application hops from one port to another till it can establish and maintain a connection. In other words, the application might try different ports till it can successfully establish a connection. Some “legitimate” applications use this technique to evade firewalls. In the following figure, the client kept trying different ports to reach the server till it discovered a destination port not blocked by the firewall.
There is another type of port hopping where the application establishes the connection on one port and starts transmitting some data; after a while, it establishes a new connection on (i.e., hopping to) a different port and resumes sending more data. The purpose is to make it more difficult for the blue team to detect and track all the exchanged traffic.

On the AttackBox, you can use the command ncat -lvnp PORT_NUMBER to listen on a certain TCP port.

-l listens for incoming connections
-v provides verbose details (optional)
-n does not resolve hostnames via DNS (optional)
-p specifies the port number to use
-lvnp PORT_NUMBER listens on TCP port PORT_NUMBER. If the port number is less than 1024, you need to run ncat as root.
For example, run ncat -lvnp 1025 on the AttackBox to listen on TCP port 1025, as shown in the terminal output below.

Evasion Using Port Tunneling
Port tunneling is also known as port forwarding and port mapping. In simple terms, this technique forwards the packets sent to one destination port to another destination port. For instance, packets sent to port 80 on one system are forwarded to port 8080 on another system.

Port Tunneling Using ncat
Consider the case where you have a server behind the firewall that you cannot access from the outside. However, you discovered that the firewall does not block specific port(s). You can use this knowledge to your advantage by tunneling the traffic via a different port.

Consider the following case. We have an SMTP server listening on port 25; however, we cannot connect to the SMTP server because the firewall blocks packets from the Internet sent to destination port 25. We discover that packets sent to destination port 443 are not blocked, so we decide to take advantage of this and send our packets to port 443, and after they pass through the firewall, we forward them to port 25. Let’s say that we can run a command of our choice on one of the systems behind the firewall. We can use that system to forward our packets to the SMTP server using the following command.

ncat -lvnp 443 -c "ncat TARGET_SERVER 25"

The command ncat uses the following options:

-lvnp 443 listens on TCP port 443. Because the port number is less than 1024, you need to run ncat as root in this case.
-c or --sh-exec executes the given command via /bin/sh.
"ncat TARGET_SERVER 25" will connect to the target server at port 25.
As a result, ncat will listen on port 443, but it will forward all packets to port 25 on the target server. Because in this case, the firewall is blocking port 25 and allowing port 443, port tunneling is an efficient way to evade the firewall.

#EVASION USING NON STANDDARD PORTS
ncat -lvnp PORT_NUMBER -e /bin/bash will create a backdoor via the specified port number that lets you interact with the Bash shell.

-e or --exec executes the given command
/bin/bash location of the command we want to execute
On the AttackBox, we can run ncat 10.201.35.151 PORT_NUMBER to connect to the target machine and interact with its shell.

Considering the case that we have a firewall, it is not enough to use ncat to create a backdoor unless we can connect to the listening port number. Moreover, unless we run ncat as a privileged user, root, or using sudo, we cannot use port numbers below 1024.

Answer the questions below
We’re continuing to use the web-form from Task 6 to set up the ncat listener. Knowing that the firewall does not block packets to destination port 8081, use ncat to listen for incoming connections and execute Bash shell. Use the AttackBox to connect to the listening shell. What is the user name associated with which you are logged in?


Evasion Approach	Nmap Argument
Hide a scan with decoys	-D DECOY1_IP1,DECOY_IP2,ME
Use an HTTP/SOCKS4 proxy to relay connections	--proxies PROXY_URL
Spoof source MAC address	--spoof-mac MAC_ADDRESS
Spoof source IP address	-S IP_ADDRESS
Use a specific source port number	-g PORT_NUM or --source-port PORT_NUM
Fragment IP data into 8 bytes	-f
Fragment IP data into 16 bytes	-ff
Fragment packets with given MTU	--mtu VALUE
Specify packet length	--data-length NUM
Set IP time-to-live field	--ttl VALUE
Send packets with specified IP options	--ip-options OPTIONS
Send packets with a wrong TCP/UDP checksum	--badsum

#example commands
nmap -sS -Pn -F <ip address>
-sS =steath scan
-Pn = proceed of no ping reply
-F = scan first well know 100 port
