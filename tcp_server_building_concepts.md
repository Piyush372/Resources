# Comprehensive Guide to Building a TCP Server
## **1. What is a Network?**
A network is a collection of interconnected computing devices that can exchange data and share resources. Think of it like a road system connecting different cities (computers):

- Devices can send information (like packages)
- Communication follows specific rules (protocols)
- Networks can be small (home network) or global (internet)

## **Types of Networks**

**LAN (Local Area Network):**

- Small geographic area
- Example: Office or home network
- Typically high-speed connections


**WAN (Wide Area Network):**

- Covers large geographic areas
- Internet is the largest WAN
- Connects multiple LANs

## **2. Network Communication Models**
### **OSI (Open Systems Interconnection) Model**
Seven layers describing network communication:

**1. Physical Layer:**

- Physical transmission of raw bits
- Cables, electrical signals
- Defines physical characteristics of network hardware


**2. Data Link Layer:**

- Manages data transfer between directly connected nodes
- Handles error detection
- MAC addressing
- Ethernet switches operate here


**3. Network Layer:**

- Routing of data packets
- IP addressing
- Determines best path for data transmission
- Routers operate at this layer


**4. Transport Layer:**

- Ensures reliable data transmission
- Segmentation and reassembly of data
- TCP and UDP protocols
- Manages end-to-end communication


**5. Session Layer:**

- Establishes, manages, and terminates connections
- Handles authentication
- Synchronizes communication


**6. Presentation Layer:**

- Data formatting and encryption
- Converts data between application and network formats
- Handles data compression


**7. Application Layer:**

- Closest to end-user
- Provides network services directly to applications
- Protocols like HTTP, FTP, SMTP

## **3. IP Addressing**
**IPv4 Address**

- 32-bit numeric address
- Divided into network and host portions
- Example: 192.168.1.1
- Classes: A, B, C, D, E
- Private IP ranges:
  - 10.0.0.0 to 10.255.255.255
  - 172.16.0.0 to 172.31.255.255
  - 192.168.0.0 to 192.168.255.255



**IPv6 Address**

- 128-bit address
- Larger address space
- Hexadecimal representation
- Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

---

## 1. Introduction to TCP
**Transmission Control Protocol (TCP)** is a connection-oriented communication protocol used in networking to ensure reliable data transmission between devices. It is part of the **TCP/IP protocol suite**.
It operates at the transport layer of the OSI model and provides reliable, ordered, and error-checked delivery of data between applications running on networked devices.

## TCP Headers and Fields
To understand how TCP operates, it’s essential to explore the structure of a TCP segment. The TCP header is the foundation of communication, carrying critical information that ensures reliable, ordered, and error-checked data transfer. Let's dive into the details.

### **What is a TCP Segment?**
A TCP segment consists of two parts:
- **Header:** Contains control information and metadata about the segment.
- **Data:** Contains the payload being transmitted (e.g., part of a file or message).
The header is always present, while the data might not be (e.g., in a control message like SYN or FIN).

### **Structure of a TCP Header**
```
  -----------------------------------------------------------------
 |     Source Port(16 bits)     |    Destination Port (16 bits)  | 
  -----------------------------------------------------------------
 |                      Sequence Number (32 bits)                |
  -----------------------------------------------------------------
 |                    Acknowledgment Number  (32 bits)           |
  -----------------------------------------------------------------
 |  Data Offset |Res.|  Flags (9 bits)  |  Window Size (16 bits) |
  -----------------------------------------------------------------
 |     Checksum          |    Urgent Pointer (if URG flag set)   |
  -----------------------------------------------------------------
 |                  Options (if any)            |    Padding     |
  -----------------------------------------------------------------
```

### Detailed Explanation of Key Fields

**1. Source Port and Destination Port**
- Purpose:
  - Identify specific services/processes
  - 16-bit number (0-65535)
  - Divided into ranges:
    - 0-1023: Well-known ports
    - 1024-49151: Registered ports
    - 49152-65535: Dynamic/private ports
  - Common Port Examples
    - HTTP: 80
    - HTTPS: 443
    - SSH: 22
- Example:
A web browser might use a random source port (e.g., 49512) to connect to a web server’s destination port (e.g., 80 for HTTP or 443 for HTTPS).

**2. Sequence Number**
- Purpose:
  - Ensures data is delivered in the correct order.
  - Helps reassemble out-of-order segments.
- How It Works:
  - If a segment has a sequence number of 1000 and contains 500 bytes of data, the next segment will start with sequence number 1500.

**3. Data Offset**
- Purpose:
  - Specifies where the data begins in the segment.
  - Indicates the size of the header (minimum 20 bytes, but can be larger if options are present).

 **4. Flags (Control Bits)**
- Purpose:
  - Control the state of the connection.
- Common Flags:
  - SYN: Synchronize sequence numbers (used during connection establishment).
  - ACK: Acknowledgment field is valid.
  - FIN: Finish the connection (used during termination).
  - RST: Reset the connection (used for errors).
  - PSH: Push data to the receiving application immediately.
  - URG: Urgent pointer field is valid.
  - ECE/CWR: Used for congestion notification.
 
**5. Window Size**
- Purpose:
  - Implements flow control by specifying the amount of data the sender is willing to receive.
- Example:
  - A window size of 1024 means the sender can send up to 1024 bytes before waiting for an acknowledgment.

**6. Checksum**
- Purpose:
  - Ensures the integrity of the header and data.
- How It Works:
  - The checksum is calculated at the sender and verified at the receiver.
  - If the checksum fails, the segment is discarded.

**7. Padding**
- Purpose:
  - Ensures the header length is a multiple of 32 bits.
- How It Works:
  - Extra zeros are added if necessary.

**8. Options**
- Purpose:
  - Provides additional features or optimizations.
- Examples:
  - Maximum Segment Size (MSS): Indicates the largest amount of data a segment can carry.
  - Window Scaling: Allows for larger window sizes (used in high-speed networks).

### Key Features of TCP:
- **Reliable Communication:** Ensures data is delivered without errors and in the correct order. If a packet is lost or corrupted during transmission, TCP automatically retransmits it.
- **Connection-Oriented:** Establishes a connection before data transfer. This process is called the three-way handshake.
- **Flow Control:** Manages the rate of data transmission. TCP uses mechanisms like the sliding window protocol to regulate the rate of data transfer based on the receiver's capacity.
- **Error Detection:** Uses checksums to detect errors in data and retransmit them if necessary.
- **Congestion Control:** TCP detects network congestion and adjusts the transmission rate to prevent packet loss.
- **Full Duplex Communication:** TCP allows simultaneous data transmission in both directions.
  - Example:
  During a video call:
    - Audio and video streams are sent and received at the same time.

### How TCP Works
TCP divides data into smaller chunks called **segments**, adds control information to each segment (like sequence numbers and checksums), and sends them over the network.

At the receiving end:

- TCP reassembles the segments in the correct order using the sequence numbers.
- It verifies data integrity using the checksums.
- If a segment is missing or corrupted, TCP requests retransmission.

### The Three-Way Handshake
The three-way handshake establishes a reliable connection between two devices. Here's how it works:

**SYN (Synchronize):**
The client sends a SYN packet to the server to initiate a connection. This packet includes a randomly generated sequence number.

**SYN-ACK (Synchronize-Acknowledgment):**
The server responds with a SYN-ACK packet, acknowledging the client's sequence number and providing its own sequence number.

**ACK (Acknowledgment):**
The client sends an ACK packet back to the server, confirming the server's sequence number. At this point, the connection is established.

```
Client                Server
CLOSED                CLOSED
  |                    |
  |       SYN          |
  |----------------→   | 
SYN_SENT              LISTEN
  |                    |
  |                  SYN_RCVD
  |     SYN-ACK        |
  |   ←----------------| 
  |                    |
  |       ACK          |
  |----------------→   |
ESTABLISHED         ESTABLISHED
  |                    |
```

**Here’s a simplified sequence of the handshake:**

- Client → Server: SYN (Seq=1000)
- Server → Client: SYN-ACK (Seq=2000, Ack=1001)
- Client → Server: ACK (Seq=1001, Ack=2001)
- At this point, the connection is established, and data transfer can begin.

### Key Components
**1. Sequence Numbers:**
- A unique number assigned to each packet to ensure proper reordering and reliability.
  
**2. Acknowledgment Numbers:**
- Confirms receipt of the previous packet by referencing its sequence number.
  
**3. Flags (SYN, ACK):**
- Indicate the type of message (e.g., SYN for connection initiation).

### TCP States

The table below outlines the various TCP states, their descriptions, and when they occur during the TCP lifecycle.

| State             | Description                                                                                   | When It Occurs                                                                |
|-------------------|-----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| **CLOSED**        | No connection exists.                                                                         | Initial state or after a connection is fully terminated.                      |
| **LISTEN**        | Server is waiting for an incoming connection request.                                         | Passive open by a server (e.g., using `listen()` system call).                |
| **SYN-SENT**      | Client has sent a SYN (synchronize) request to establish a connection.                        | After a client performs an active open (e.g., using `connect()` system call). |
| **SYN-RECEIVED**  | Server has received a SYN and sent a SYN-ACK in response.                                     | After the server receives a connection request and responds to it.            |
| **ESTABLISHED**   | Connection is fully established, and data can be transmitted.                                 | After the three-way handshake is completed.                                   |
| **FIN-WAIT-1**    | One side has sent a FIN (finish) request to terminate the connection.                         | After a connection close is initiated by the application.                     |
| **FIN-WAIT-2**    | Waiting for the other side to send a FIN after acknowledging the first FIN.                   | After the FIN is acknowledged by the peer.                                    |
| **CLOSE-WAIT**    | One side has received a FIN and is waiting for the application to close the connection.       | After receiving a FIN from the peer.                                          |
| **CLOSING**       | Both sides have sent FIN requests simultaneously.                                             | Rare condition when both sides close at the same time.                        |
| **LAST-ACK**      | Waiting for the final acknowledgment of a FIN sent to terminate the connection.               | After sending a FIN in response to a received FIN.                            |
| **TIME-WAIT**     | Waiting for a period to ensure all duplicate packets have been received and acknowledged.     | After the final acknowledgment is sent during connection termination.         |

## Notes
- The **CLOSED** state is both the start and end state of a TCP connection.
- The **TIME-WAIT** state ensures the connection is fully terminated and prevents delayed packets from interfering with new connections.
- Not all states are traversed in every connection; for example, the **CLOSING** state is uncommon.

---
### **Connection Termination (4-Way Handshake)**
When the communication ends, TCP uses a 4-Way Handshake to close the connection. We'll cover this in detail later.

### **Why is Connection Termination Important?**
**1. Resource Management:**
Both the client and server free up resources (e.g., sockets, memory) after the connection ends.

**2. Orderly Shutdown:**
Ensures all pending data is transmitted and acknowledged before closing the connection.

### Steps in the 4-Way Handshake
TCP uses four distinct messages to close a connection: FIN, ACK, and optionally another FIN and ACK.

**Step 1: FIN (Finish)**
- One side (usually the client) sends a FIN message to signal it has finished sending data.
- Purpose:
  - Indicates the sender will not transmit any more data but is still ready to receive.
    
**Step 2: ACK (Acknowledge FIN)**
- The receiver acknowledges the FIN by sending an ACK message.
- Purpose:
  - Confirms that the sender’s request to stop transmitting data has been received.
    
**Step 3: FIN (Finish from Receiver)**
- The receiver sends its own FIN message once it finishes sending data.
- Purpose:
  - Signals that it has also completed its data transmission.
    
**Step 4: ACK (Acknowledge FIN from Receiver)**
- The sender acknowledges the receiver’s FIN message by sending a final ACK.
- At this point, the connection is fully closed.

```
Client                Server
ESTABLISHED         ESTABLISHED
  |                    |
  |       FIN          |
  |----------------→   |
FIN_WAIT_1         CLOSE_WAIT
  |                    |
  |       ACK          |
  |   ←----------------| 
FIN_WAIT_2         CLOSE_WAIT
  |                    |
  |                    |
  |       FIN          |
  |   ←----------------| 
TIME_WAIT           LAST_ACK
  |                    |
  |       ACK          |
  |----------------→   |
TIME_WAIT            CLOSED
  |                    |
  |(wait 2MSL)         |
CLOSED                 |
```

## **Flow Control and Congestion Control in TCP**
TCP is designed to ensure reliable data transfer, and two critical mechanisms—Flow Control and Congestion Control—play key roles in achieving this. Both help maintain efficiency and stability in communication.

### **Flow Control**
**Definition:**
- Flow control ensures that a sender does not overwhelm a receiver with more data than it can process or store in its buffer.

**How It Works:**

TCP uses the receiver’s advertised window size to implement flow control. This value is communicated in the Window Size field of the TCP header and is dynamically adjusted based on the receiver’s buffer availability.

1. **Receiver Buffer:**
- Every TCP connection has a buffer for storing incoming data.
- If the buffer is full, the receiver advertises a zero window size, telling the sender to pause data transmission.

2. **Sliding Window Protocol:**
- TCP uses a sliding window mechanism for flow control.
- The sender can transmit only as much data as allowed by the current window size.
- The window "slides" forward as the receiver acknowledges received data and frees up buffer space.
  
**Example of Flow Control**
- Receiver advertises a window size of 10 KB.
- Sender sends 10 KB of data.
- Receiver processes 5 KB and advertises a new window size of 5 KB.
- Sender adjusts its transmission rate accordingly.

### **Congestion Control**
**Definition:**
  - Congestion control prevents the network itself (routers, switches, etc.) from becoming overwhelmed, ensuring fairness and stability.

**Why It’s Needed**
- Networks have limited bandwidth and can experience congestion when too many devices send data simultaneously.
- Congestion leads to:
  - Packet loss.
  - Increased latency.
  - Decreased throughput.
    
**How It Works**

TCP implements congestion control using the following algorithms:

1. **Slow Start:**
  - When a connection starts, TCP assumes the network's capacity is unknown.
  - The sender begins by sending a small amount of data (1 segment).
  - It doubles the amount of data sent in each round-trip time (RTT) until packet loss occurs or a threshold is reached.
    
2. **Congestion Avoidance:**
  - After detecting the network's capacity, TCP grows the congestion window linearly instead of exponentially.
  - This ensures a gradual increase in throughput without causing congestion.
    
3. **Fast Retransmit:**
  - If a packet is lost, the sender retransmits it immediately without waiting for a timeout.
  - This is triggered by receiving duplicate acknowledgments (ACKs).
    
4. **Fast Recovery:**
  - Instead of returning to slow start after a loss, TCP reduces the congestion window size and switches to congestion avoidance.
    
**Key Metrics**
- Congestion Window (cwnd):
  - The sender maintains a congestion window that limits how much data can be in flight at any time.
  - The value is adjusted dynamically based on network conditions.
- Threshold (ssthresh):
  - Defines the transition point between slow start and congestion avoidance.
    
**Congestion Control in Action**
- Sender starts with a small congestion window (e.g., 1 segment).
- If no loss occurs, the window grows exponentially (slow start).
- Upon reaching the threshold, growth becomes linear (congestion avoidance).
- If packet loss occurs, the window size is reduced, and recovery begins.

---
### **Visualization: TCP in Action**
Consider an example of downloading a web page:

- The client requests a web page.
- TCP establishes a connection with the server.
- Packets (HTML, CSS, images) are sent.
- TCP ensures all packets are received, ordered, and uncorrupted.
- The web page is rendered in your browser.

### TCP vs. IP
While TCP ensures reliable communication, Internet Protocol (IP) handles the delivery of packets between devices. Together, they form the backbone of internet communication, where IP is responsible for routing, and TCP ensures data reliability.

### Common Use Cases of TCP
- **Web Browsing:** TCP powers HTTP and HTTPS, ensuring that web pages are delivered reliably.
- **Email:** Protocols like SMTP, IMAP, and POP3 use TCP for sending and receiving emails.
- **File Transfers:** FTP and SFTP use TCP for error-free file exchange.
- **Remote Access:** Protocols like SSH and Telnet rely on TCP for secure communication.
- **Streaming:** Video and audio streaming platforms often use TCP when buffering or data integrity is crucial.

## **UDP (User Datagram Protocol)**
Characteristics:

- Connectionless protocol
- No guaranteed delivery
- Low overhead
- Faster transmission
- No error recovery
- Used for time-sensitive communications

## 2. TCP vs. UDP
TCP and UDP (User Datagram Protocol) are transport layer protocols, but they differ in functionality:

| Feature           | TCP                         | UDP                      |
|-------------------|-----------------------------|--------------------------|
| Connection Type   | Connection-oriented         | Connectionless           |
| Reliability       | Reliable                    | Unreliable               |
| Speed             | Slower (overhead for reliability) | Faster                  |
| Use Cases         | File transfer, HTTP, Email | Streaming, DNS, VoIP     |

---

