>  *NOTE*: Networking tends to be a stronger focus on infrastructure and distributed systems interviews, but it's still useful to know the fundamentals well!

### What is Networking?

It's about connecting devices and enabling them to communicate. Networks are built on a layered architecture called the **OSI Model**. 

These network layers are essentially abstractions that allow us to reason about the communication between devices in a more simpler manner (E.g. We can just call `open` to read files instead of manually instructing the disk to read bytes.)

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

**IMPORTANT FACTS:**
DHCP (**Dynamic Host Configuration Protocol**) is a service whose job is to automatically assign IP addresses to devices when they join a network. This IP address typically has a lease (a set duration) where it will expire.

RIR (**Regional Internet Registry**) is an organization that manages and distributes public IP address blocks within a geographic region.

There are 5 RIRs that cover the entire world:
```ARIN → North America
RIPE → Europe, Middle East, Central Asia
APNIC → Asia Pacific (this is yours — Indonesia falls here)
LACNIC → Latin America and Caribbean
AFRINIC → Africa
```

How IP Address allocation works:
```
IANA (Internet Assigned Numbers Authority)
  — the global authority, allocates large blocks to RIRs
        ↓
RIRs (ARIN, RIPE, APNIC, etc.)
  — allocate smaller blocks to ISPs and large organizations
        ↓
ISPs (Telkom Indonesia, Indosat, etc.)
  — allocate individual IPs to their customers
        ↓
You / Your servers
```

 
## Transport Layer (4th Layer)

This is where TCP, QUIC, & UDP operates. They provide end-to-end communication services and features like reliability, ordering, and flow control on top of the network layer.

For most system design interviews, the choice of the primary protocols will usually fall down to **TCP** Vs. **UDP**. QUIC is not that ubiquitous and it's goal is to be a better TCP (modernization and performance benefits). 


### UDP (User Datagram Protocol) - Fast But Unreliable.

**Characteristics of UDP**:
1. **Connectionless**: No handshake or connection setup. 
2. **No guarantee of delivery**: Packets may be lost without any notification of them being lost. 
3. **No ordering**: Packets may arrive in a different order than they were sent. 
4. **Lower latency**: Because its connectionless (less overhead), this means faster transmission.  Also the side of the header (8 bytes) is a lot less compared to TCP (20 bytes). 

UDP is absolutely *perfect* for applications where speed is more important than reliability. Certain applications like: VoIP (Voice Calling), Online Gaming, Live Video Streaming, DNS Lookups. In these applications, these applications are equipped to handle the occasional packet loss / out of order packet. 

For example, in VoIP (Voice Calling), some packet loss might lead to a slight hiccup in the audio, but the conversation is still overall quite clear. 

>NOTE: Browsers don't have widespread support for UDP yet, outside of WebRTC. 


### TCP (Transmission Control Protocol) - Reliable But Have Overhead

**Characteristics of TCP**:
1. **Connection-Oriented**: Establishes a direct connection before any data transfer happens. 
	 - This connection is called a *stream*. It is a stateful connection between the client and the server. 
2. **Guaranteed Delivery**: Packets are guaranteed to arrive + in order + no errors. 
	- TCP ensures that the recipients of the messages (packets) will acknowledge the receipt of those packets, and if no receipt, TCP will retransmit the messages until they are acknowledged. 
3. **Flow Control**: Prevents the sender from sending too much data and overwhelming the receiver's buffer (could lead to packet loss). (UDP don't have this)
	- The receiver has this thing called a *receive buffer* and if too much data is being sent from the sender, its receive bugger will fill up and packets get dropped.

	Flow control solves this with a receive window:
		```
		Receiver tells sender: "My buffer can currently handle 64KB"
		Sender: only sends up to 64KB before waiting for acknowledgment
		Receiver processes data, buffer frees up
		Receiver tells sender: "Now I can handle 128KB"
		Sender: increases how much it sends
		```
	The receiver continuously advertises how much space it has left in its buffer, and the sender respects that limit. It's a feedback loop between sender and receiver.
4. **Congestion Control**: Prevents the sender from sending too much data and overwhelming the network (UDP don't have this).
	- The network between the sender and receiver has limits (routers have finite buffer space, links have finite bandwidth).
	- If many senders are sending data simulatenously, the routers can start dropping packets and causing the network to be congested.

TCP is absolutely *perfect* for applications where data integrity is important (this is basically every other application where UDP is not a good fit!).

## Choosing Between TCP Vs. UDP. 

Most cases, the answer will be TCP. But, there are cases where UDP might win:
- Applications where low latency is critical. 
- Applications where some packet loss is fine.
- Applications that handle high-volume telemetry / logs where occasional loss is acceptable. 
- No need support for web browsers. 


## Application Layer (7th Layer)

This is where application protocols like DNS, HTTP, Websockets, & WebRTC operates. These build on top of TCP (or UDP in the case of WebRTC) and they provide a layer of abstraction for different types of data that is typically associated with web applications. 

> NOTE: Application Layer operates in the *User Space*, whereas layers beneath it operate in the *Kernel Space*. Application Layer is also more flexible and easy to be modified compared to the lower layers. 


### HTTP (Hypertext Transfer Protocol)

The standard for data communication on the web. It operates as a request-response protocol (clients send requests, server respond with the requested data). 

HTTP is a **stateless** protocol, meaning each request is independent and the server does not need to maintain information about previous requests. 

![[SimpleHTTPReqResp.png]]

**HTTP Versions**

1. HTTP/1.1 : Default for years. One request at a time / connection. It uses keep-alive to reuse connections, but still head-of-line blocking (if one request is slow, everything behind it waits). 
2. HTTP/2 : Multiplexing. Multiple requests simultaneously over 1 TCP connection significantly reduces latency. Most modern websites use this. 


**Key Concepts in HTTP:**

1. **Request Methods**
	- GET: Request data from the server. Should be idempotent and no body. 
	- POST: Send data to the server. 
	- PUT: Update data on the server. 
	- PATCH: Update a resource partially. 
	- DELETE: Delete data from the server.  Should be idempotent. 

2. **Status Codes**
	- 2xx (Success):
		- 200 OK: Request was successful. 
		- 201 Created: Request was successful and a new resources was created. 
	- 3xx (Moved):
		- 301 Moved Permanently: Requested resource has been moved permanently. 
		- 302 Found: Requested resource has been moved temporarily
	- 4xx (Client Error):
		- 401 Unauthorized: Request requires authentication. 
		- 403 Forbidden: Server understood the request but refuses to authorize it. 
		- 404 Not Found: Requested resource was not found. 
		- 429 Too Many Requests: Client has sent too many requests in a given amount of time. 
	- 5xx (Server Error):
	    - 500 Server Error: The server encountered an error
	    - 502 Bad Gateway: The server received an invalid response from the upstream server

3. **Headers**
	- Metadata about the request or response (Flexible). 

HTTP headers are a great example of how to design an interface that is flexible to unknown future use-cases and provides a good lesson for API design. Content negotiation is a perfect case study.

The HTTP Accepts-Encoding header as an example provides clients a way to indicate they can handle different types of content encoding. This allows servers to provide (e.g.) gzip or br (brotli) encoded responses if they're available. Servers can then respond with the most efficient encoding for that client with Content-Encoding: X providing both backward compatibility and graceful degradation.

4. **Body**
	- The actual content being transferred. 

**HTTPS** adds a security layer (TLS/SSL) to encrypt communications. This protects against eavesdropping and man-in-the-middle attacks. HTTPS is always used when building websites. It basically ensures the contents of the HTTP requests and responses are encrypted and safe in transit. 

*However*, even though the contents of our HTTP requests and responses are encrypted, they aren't guaranteed to be generated by our client. 

Our API endpoint should never trust the contents of the request body before vailidating it. 

A common mistake: Including the user's ID in the request body to use it to make a DB call. 
- An attacker that can change the request body would simply just need to change the user ID in order to make DB calls.





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


