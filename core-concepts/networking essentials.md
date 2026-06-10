>  *NOTE*: Networking tends to be a stronger focus on infrastructure and distributed systems interviews, but it's still useful to know the fundamentals well!

### What is Networking?

It's about connecting devices and enabling them to communicate. Networks are built on a layered architecture called the **OSI Model**. 

These network layers are essentially abstractions that allow us to reason about the communication between devices in a lot more simpler manner (E.g. We can just call `open` to read files instead of manually instructing the disk to read bytes.)

![[NetworkLayers.png]]

The 3 important layers that come up in system design interviews are: 
- Network Layer
- Transport Layer
- Application Layer

## Network Layer (3rd Layer)

This is where IP (Internet Protocol) operates. This is where routing and addressing is handled. While other protocols asides from IP exists (like InfiniBand), IP is the most common. 

What IP is responsible for:
- Breaking data down into packets. 
- Handling packet forwarding between networks,
- Providing best-effort delivery to any destination IP address on the network. 

### What exactly is IP?

IP is essentially the addressing system for the internet. It contains the set of rules that gives every single device a unique address (their IP address, e.g. 192.168.1.1). 

IP also defines how data gets routed from one IP address to another. For example, when I type in `google.com` from my device (my IP address), IP needs to route that request to Google's servers (their IP address). More about IP addresses: [[ip addresses]]


**"Breaking data down into packets"**

Instead of sending one giant chunk of data, it breaks the data down into small fixed-size chunks called packets. Each packet contains:
- Header (metadata including source IP, destination IP, packet number)
- Payload (actual chunk of the data)

*[Why is this done?]()*
- Efficiency - Packets from different sources can share the same network wire at the same time. 
- Resilience - If a packet gets lost, only that packet needs to be resent and not the whole data. 
- Flexibility - Packets from the same message can take different routes and still arrive at the same destination. 



**"Handling packet forwarding between networks"**

Routers - routers are specialized devices where there only job is to forward packets toward their destinations. 

Every router maintains its *routing table* which is a list that says "if packet X is headed to this range of IP addresses, send it out through this port toward that next router".

**IMPORTANT**: No single router knows the full path for a packet. It only knows the next best hop. This is called **hop-by-hop** routing. 

Example:
```
Your packet: "I need to reach 142.250.4.46 (Google)"

Router 1: "I don't know exactly where that is, 
           but my routing table says packets for 
           142.x.x.x go out through port 3" → forwards it

Router 2: "My table says 142.250.x.x goes toward 
           this next router" → forwards it

Router N: "I know exactly where 142.250.4.46 is,
           it's directly connected to me" → delivers it
```

*How are these routing tables formed in the first place?*
They are formed through complex *routing protocols* (no need to know in-depth). Popular ones are BGP (Border Gateway Protocol) and OSPF.  

 
## Transport Layer (4th Layer)

This is where TCP, QUIC, & UDP operates. They provide end-to-end communication services and features like reliability, ordering, and flow control on top of the network layer.


## Application Layer (7th Layer)

This is where application protocols like DNS, HTTP, Websockets, & WebRTC operates. These build on top of TCP (or UDP in the case of WebRTC) and they provide a layer of abstraction for different types of data that is typically associated with web applications. 

To see how these 3 layers work together, let's walk through an example of how a simple web request works:

![[SimpleWebRequest.png]]

1. **DNS Resolution**: Client starts by resolving the domain name of the website to an IP address using DNS. 
2. **TCP Handshake**: Client initiates a TCP connection with the server using a 3-way handshake.
	- SYN: Client sends a SYN (synchronize) packet to the server to request a connection
	- SYN-ACK: Server responds with a SYN-ACK (synchronize-acknowledge) packet to acknowledge the request. 
	- ACK: Client sends a ACK (acknowledge) packet to establish the connection. 
3. **HTTP Request**: Once the TCP connection is established, the client sends a HTTP GET request to the server to request the webpage. 
4. **Server Processing**: Server processes the request, retrieves the requested web page and prepares the HTTP response (This step is the only latency that most SWEs can control).
5. **HTTP Response**: Server sends the HTTP response over to the client, which included the requested webpage content. 
6. **TCP Teardown**: After the data transfer is complete, the client and server close the TCP connection using a 4-way handshake:
	- FIN: Client sends a FIN (finish) packet to the server to terminate the connection. 
	- ACK: The server acknowledges the FIN packet with an ACK. 
	- FIN: Server sends a FIN packet to the client to terminate its side of the connection. 
	- ACK: Client acknowledges the server's FIN packet with an ACK. 


