# How to read a Wireshark TCP/HTTP log

In this reading, you’ll learn how to read a Wireshark TCP/HTTP log for network traffic between employee website visitors and the company’s web server. Most network protocol/traffic analyzer tools used to capture packets will provide this same information.

## ***Log entry number and time***

| No. | Time |
|---|---|
| 47 | 3.144521 |
| 48 | 3.195755 |
| 49 | 3.195755 |

This Wireshark TCP log section provided to you starts at log entry number (No.) 47, which is three seconds and .144521 milliseconds after the logging tool started recording. This indicates that approximately 47 messages were sent and received by the web server in the 3.1 seconds after starting the log. This rapid traffic speed is why the tool tracks time in milliseconds. 

## ***Source and destination IP addresses***

| Source | Destination |
|---|---|
| 198.51.100.23 | 192.0.2.1 |
| 192.0.2.1 | 198.51.100.23 |
| 198.51.100.23 | 192.0.2.1 |

The source and destination columns contain the source IP address of the machine that is sending a packet and the intended destination IP address of the packet. In this log file, the IP address 192.0.2.1 belongs to the company’s web server. The range of IP addresses in 198.51.100.0/24 belong to the employees’ computers. 

## ***Protocol type and related information***

| Protocol | Info |
|---|---|
| TCP | 42584->443 [SYN] Seq=0 Win-5792 Len=120... |
| TCP | 443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| TCP | 42584->443 [ACK] Seq=1 Win-5792 Len=120... |

The Protocol column indicates that the packets are being sent using the TCP protocol, which is at the transport layer of the TCP/IP model. In the given log file, you will notice that the protocol will eventually change to HTTP, at the application layer, once the connection to the web server is successfully established.

The Info column provides information about the packet. It lists the source port followed by an arrow → pointing to the destination port. In this case, port 443 belongs to the web server. Port 443 is normally used for encrypted web traffic.

The next data element given in the Info column is part of the three-way handshake process to establish a connection between two machines. In this case, employees are trying to connect to the company’s web server: 

*	The [SYN] packet is the initial request from an employee visitor trying to connect to a web page hosted on the web server. SYN stands for “synchronize.” 
*	The [SYN, ACK] packet is the web server’s response to the visitor’s request agreeing to the connection. The server will reserve system resources for the final step of the handshake. SYN, ACK stands for “synchronize acknowledge.”
*	The [ACK] packet is the visitor’s machine acknowledging the permission to connect. This is the final step required to make a successful TCP connection. ACK stands for “acknowledge.”

The next few items in the Info column provide more details about the packets. However, this data is not needed to complete this activity. If you would like to learn more about packet properties, please visit [Microsoft’s Introduction to Network Trace Analysis](https://techcommunity.microsoft.com/blog/coreinfrastructureandsecurityblog/introduction-to-network-trace-analysis-3-tcp-performance/3737115). 

## ***Normal website traffic***

A normal transaction between a website visitor and the web server would be like:

| **No.** | **Time** | **Source** | **Destination** | **Protocol** | **Info** |
|---|---|---|---|---|---|
| 47 | 3.144521 | 198.51.100.23 | 192.0.2.1 | TCP | 42584->443 [SYN] Seq=0 Win=5792 Len=120... |
| 48 | 3.195755 |	192.0.2.1 |	198.51.100.23 |	TCP |	443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| 49 | 3.246989 |	198.51.100.23 |	192.0.2.1 |	TCP |	42584->443 [ACK] Seq=1 Win-5792 Len=120... |
| 50 | 3.298223 |	198.51.100.23 |	192.0.2.1 |	HTTP |	GET /sales.html HTTP/1.1 |
| 51 | 3.349457 |	192.0.2.1 |	198.51.100.23 |	HTTP |	HTTP/1.1 200 OK (text/html) |

Notice that the handshake process takes a few milliseconds to complete. Then, you can identify the employee’s browser requesting the sales.html webpage using the HTTP protocol at the application level of the TCP/IP model. Followed by the web server responding to the request.

## The Attack

As you learned previously, malicious actors can take advantage of the TCP protocol by flooding a server with SYN packet requests for the first part of the handshake. However, if the number of SYN requests is greater than the server resources available to handle the requests, then the server will become overwhelmed and unable to respond to the requests. This is a network level denial of service (DoS) attack, called a SYN flood attack, that targets network bandwidth to slow traffic. A SYN flood attack simulates a TCP connection and floods the server with SYN packets. A DoS direct attack originates from a single source. A distributed denial of service (DDoS) attack comes from multiple sources, often in different locations, making it more difficult to identify the attacker or attackers. 

![image](https://github.com/user-attachments/assets/c92bb0a8-83d9-413e-acc5-e080f15de110)

There are two tabs at the bottom of the log file. One is labeled “Color coded TCP log.” If you click on that tab, you will find the server interactions with the attacker’s IP address (203.0.113.0) marked with red highlighting (and the word “red” in column A). 

| Color as text |	No. | Time | Source (x = redacted) | Destination (x = redacted) | Protocol | Info |
|---|---|---|---|---|---|---|		
| red| 52 |	3.390692 | 203.0.113.0 |192.0.2.1	| TCP |	54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red	| 53 |	3.441926 | 192.0.2.1 |	203.0.113.0 |	TCP |	443->54770 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| red |	54 |	3.493160 |	203.0.113.0 |	192.0.2.1 |	TCP |	54770->443 [ACK Seq=1 Win=5792 Len=0... |
| green |	55 |	3.544394 | 198.51.100.14	| 192.0.2.1 |	TCP |	14785->443 [SYN] Seq=0 Win-5792 Len=120... |
| green |	56 | 3.599628 |	192.0.2.1 |	198.51.100.14 |	TCP |	443->14785 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| red |	57 |	3.664863 |	203.0.113.0 |	192.0.2.1 |	TCP |	54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green |	58 | 3.730097 |	198.51.100.14 |	192.0.2.1 |	TCP |	14785->443 [ACK] Seq=1 Win-5792 Len=120... |
| red |	59 |	3.795332 |	203.0.113.0	| 192.0.2.1 |	TCP	| 54770->443 [SYN] Seq=0 Win-5792 Len=120... |
| green |	60 |	3.860567 |	198.51.100.14 |	192.0.2.1 |	HTTP |	GET /sales.html HTTP/1.1 |
| red	| 61 |	3.939499 |	203.0.113.0 |	192.0.2.1	| TCP |	54770->443 [SYN] Seq=0 Win-5792 Len=120... |
| green |	62 |	4.018431 |	192.0.2.1 |	198.51.100.14 |	HTTP |	HTTP/1.1 200 OK (text/html) |




