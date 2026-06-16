# The Complete Networking Reference Manual: From Sockets to Cloud Architectures

Welcome to the ultimate networking reference manual. This document progresses from foundational physical communication concepts to low-level C socket programming, kernel internals, container networking, and security patterns. It is designed to serve as both an educational textbook and an active system design reference.

---

## Learning Roadmap

```text
[Part 1: Foundations] ──> [Part 2: Network Models] ──> [Part 3: Link Layer]
                                                                │
                                                                ▼
[Part 6: DNS] <────────── [Part 5: Transport Layer] <── [Part 4: Internet Layer]
     │
     ▼
[Part 7: HTTP] ──────────> [Part 8: HTTPS] ─────────> [Part 9: Server Engineering]
                                                                │
                                                                ▼
[Part 12: Modern Protocols] <─ [Part 11: Web Infra] <── [Part 10: Building Servers]
     │
     ▼
[Part 13: Linux Net Stack] ──> [Part 14: Containers] ──> [Part 15: Security]
                                                                │
                                                                ▼
[Part 17: Case Studies] <────── [Part 16: Observability] <──────┘
```

Use this roadmap to guide your progression. If you are building a server from scratch, focus heavily on Parts 5, 9, and 10. If you are troubleshooting routing issues or container deployments, focus on Parts 4, 13, and 14.

---

## Acronym Table

| Acronym | Expanded Form | OSI Layer | Description |
| :--- | :--- | :--- | :--- |
| **ARP** | Address Resolution Protocol | Layer 2 | Maps IPv4 addresses to MAC addresses. |
| **CIDR** | Classless Inter-Domain Routing | Layer 3 | IP address allocation and routing method. |
| **DHCP** | Dynamic Host Configuration Protocol | Layer 7 | Dynamically assigns IP addresses to hosts. |
| **DNS** | Domain Name System | Layer 7 | Translates human-readable domain names to IP addresses. |
| **ICMP** | Internet Control Message Protocol | Layer 3 | Used for diagnostic and error messages (e.g., ping). |
| **MAC** | Media Access Control | Layer 2 | Unique hardware identifier for network interfaces. |
| **MTU** | Maximum Transmission Unit | Layer 2/3 | The largest physical packet size that can be transmitted. |
| **NAT** | Network Address Translation | Layer 3 | Translates private IP addresses to public IP addresses. |
| **OSI** | Open Systems Interconnection | N/A | Seven-layer conceptual network model. |
| **RTT** | Round Trip Time | Layer 4 | Time taken for a packet to travel to destination and back. |
| **TCP** | Transmission Control Protocol | Layer 4 | Connection-oriented, reliable transport protocol. |
| **UDP** | User Datagram Protocol | Layer 4 | Connectionless, lightweight transport protocol. |
| **VLAN** | Virtual Local Area Network | Layer 2 | Logically segments a physical network into multiple domains. |

---

## Concept Map

```text
┌────────────────────────────────────────────────────────────────────────┐
│                              Application                               │
│                         (HTTP, DNS, TLS, gRPC)                         │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Socket API
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                               Transport                                │
│                         (TCP, UDP, QUIC/HTTP3)                         │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ IP Packets
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                               Internet                                 │
│                           (IPv4, IPv6, ICMP)                           │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Frames
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                                 Link                                   │
│                        (Ethernet, ARP, Switches)                       │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Bits / Signals
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                               Physical                                 │
│                       (Fiber, Copper, Wireless)                        │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Glossary

*   **Socket**: An abstraction provided by the operating system representing an endpoint for communication.
*   **Frame**: The unit of data at the Link Layer (Layer 2), containing physical addresses (MAC).
*   **Packet**: The unit of data at the Internet/Network Layer (Layer 3), containing logical addresses (IP).
*   **Segment**: The unit of data at the Transport Layer (Layer 4) when using TCP.
*   **Datagram**: The unit of data at the Transport Layer (Layer 4) when using UDP.
*   **Endianness**: The order in which bytes of a multi-byte word are stored in memory. Big-endian stores the most significant byte first; Little-endian stores the least significant byte first.
*   **Subnet**: A logical subdivision of an IP network.
*   **Collision Domain**: A physical network segment where packets can collide with one another during transmission (e.g., on a shared hub).
*   **Broadcast Domain**: A logical division of a computer network in which all nodes can reach each other by broadcast at the data link layer.

---

# Part 1 - Foundations

## What is a Network

### What It Is
A computer network is a collection of interconnected computing devices that can share resources and exchange data. These connections can be physical (copper cables, fiber optics) or wireless (Wi-Fi, cellular, satellite).

### Why It Exists
Networks exist to enable communication and resource sharing. Without networks, computer systems would be isolated islands, requiring physical media (like disks or tapes) to move data. Modern applications—from web browsing and streaming to distributed database replication—rely completely on fast, reliable, and standardized network communication.

### How It Works
At its core, networking is about translating data (files, keystrokes, audio) into physical signals, transmitting those signals over a distance, and translating them back into data. 

```text
[Device A] ──(Data)──> [Network Interface Card] ──(Electrical/Light Signals)──┐
                                                                              │
[Device B] <──(Data)── [Network Interface Card] <──(Electrical/Light Signals)─┘
```

### Real Packet Flow
1. **Application Generation**: An application generates data (e.g., a text string like `"Hello"`).
2. **Serialization**: The software converts this structured data into raw bytes.
3. **Transmission**: The operating system sends these bytes to the network interface card (NIC), which converts the bytes into voltage changes or light pulses.
4. **Reception**: The receiving NIC detects these physical variations, reconstructs the bytes, and alerts the receiving operating system.

### Common Mistakes
*   **Assuming instant transmission**: Data is bound by the speed of light in the physical medium. Copper transmission travels at roughly 2/3 the speed of light in a vacuum ($200,000 \text{ km/s}$ or about $200 \text{ km/ms}$).
*   **Treating the network as reliable**: Networks are inherently lossy. Electrical interference, physical damage, and congestion can drop or corrupt signals.

### Implementation Notes
In systems programming, networks are represented by virtual devices (e.g., `eth0`, `wlan0`) managed by the operating system kernel. The application developer rarely interacts with the hardware signals directly; instead, they use system calls like `socket()`, `send()`, and `recv()`.

### Debugging Notes
*   `ethtool`: Run `ethtool <interface_name>` (e.g., `ethtool eth0`) to inspect the physical link state, speed, duplex mode, and hardware statistics of the network card.

---

## Network History

### What It Is
The history of networking is the evolution of communication protocols from custom, proprietary systems to the open, globally-adopted TCP/IP protocol suite that powers the modern Internet.

### Why It Exists
Historically, early computer networks were proprietary and designed by individual manufacturers (such as IBM's SNA or DECnet). A computer made by one manufacturer could not communicate with a computer made by another. Standardizing on open protocols was necessary to build a global, decentralized network.

### How It Works
The standard historical progression is:
1.  **ARPANET (1969)**: The first packet-switching network, funded by the US Department of Defense. It introduced packet switching rather than traditional circuit-switching (used by telephone lines).
2.  **TCP/IP Standardized (1983)**: On January 1, 1983, ARPANET migrated entirely to the TCP/IP protocol suite, establishing the core mechanics of the modern Internet.
3.  **World Wide Web (1989-1991)**: Tim Berners-Lee invented HTTP and HTML, layering an accessible document-sharing system on top of the established TCP/IP infrastructure.

```text
[Circuit Switching: Dedicated Path]
Host A ──────(Reserved Line)──────> Host B  (Inefficient for bursty data)

[Packet Switching: Shared Infrastructure]
Host A ───[Packet 1]───┐
                       ├───(Shared Channel)───> Host B
Host C ───[Packet 2]───┘
```

### Real Packet Flow
In circuit switching, a connection reserves physical circuits along the path. In packet switching (which the Internet uses), data is broken into individual packets. Each packet is routed independently through the network using routing algorithms.

### Common Mistakes
*   **Confusing the Web with the Internet**: The Internet is the underlying infrastructure (cables, routers, IP addresses, TCP/UDP). The World Wide Web is an application layer system (HTTP, HTML) built on top of the Internet.

### Implementation Notes
The open design of TCP/IP is defined by RFCs (Requests for Comments) published by the IETF (Internet Engineering Task Force). Implementers must strictly adhere to these specifications to ensure interoperability.

### Debugging Notes
Modern operating systems still maintain support for older protocol families (e.g., AX.25 for amateur radio, IPX/SPX historical links), though they are disabled by default in modern environments.

---

## Physical Communication

### What It Is
Physical communication is the physical layer (Layer 1) transmission of bits using electrical signals over copper wires, light pulses over fiber optic cables, or radio waves through the air.

### Why It Exists
Data in a computer exists as logical 1s and 0s. The physical communication layer converts these logical states into physical properties that can traverse physical media.

### How It Works
Different media use different physical properties:
*   **Copper (e.g., Cat6 Ethernet)**: Uses voltage fluctuations. For example, a high voltage might represent a `1` and a low voltage a `0`.
*   **Fiber Optics**: Uses light pulses. Laser or LED transmitters send pulses down glass fibers. Photodetectors on the receiving end convert the light back to electrical signals.
*   **Wireless (Wi-Fi/Cellular)**: Uses radio frequency modulation. Data is mapped to changes in amplitude, frequency, or phase of an electromagnetic wave.

```text
[Digital State]    1      0      1      1      0
                   ┌──┐          ┌──┐  ┌──┐
[Copper Signal]  ──┘  └───┘  └──┘  └──┘  └───
```

### Real Packet Flow
At the physical level, there are no "packets." Data is sent as a continuous stream of symbols. To detect where a frame starts, the link layer adds a preamble—a specific pattern of alternating bits (such as `10101010` ending in `10101011`) that allows the receiver to synchronize its clock with the incoming signal.

### Common Mistakes
*   **Ignoring attenuation and noise**: Signals degrade over distance (attenuation) and suffer from interference (noise). This is why copper Ethernet cables are limited to 100 meters without active repeaters.

### Implementation Notes
In software development, physical layer details are handled by the NIC firmware and driver. However, network engineers must understand concepts like **Duplex**:
*   *Half-Duplex*: Devices can transmit and receive, but not at the same time (like walkie-talkies).
*   *Full-Duplex*: Devices can transmit and receive simultaneously.

### Debugging Notes
*   If a physical link is unstable, check the error rates on the interface:
    ```bash
    ip -s link show eth0
    ```
    Look for "errors", "dropped", "overrun", or "carrier" flags. High error counts usually indicate bad cabling or electromagnetic interference.

---

## Bits and Bytes

### What It Is
Bits (binary digits) are the fundamental unit of digital information, taking a value of `0` or `1`. A byte is a group of 8 bits.

### Why It Exists
Computers process data in groups of bits. The 8-bit byte became the standard unit of measurement because it is large enough to encode standard character sets (like ASCII) while matching the word size of early computer architectures.

### How It Works
A byte can represent integers from $0$ to $255$ (unsigned) or $-128$ to $127$ (signed). 
Standard prefixes:
*   **Kilobyte (KB)**: $1,000$ bytes (or $1,024$ bytes in binary notation, sometimes called KiB).
*   **Megabyte (MB)**: $1,000,000$ bytes.
*   **Gigabyte (GB)**: $1,000,000,000$ bytes.

> [!WARNING]
> Network speeds are measured in **bits** per second (bps, Kbps, Mbps, Gbps), while file sizes are measured in **bytes** (B, KB, MB, GB). A 100 Mbps internet connection can transfer at most 12.5 Megabytes of data per second ($100 / 8 = 12.5$).

### Real Packet Flow
When a stream of bytes is sent over the network, it is serialized bit-by-bit. The default transmission order on Ethernet is least significant bit (LSB) first within each byte, though high-level protocols generally deal with bytes as the atomic unit.

### Common Mistakes
*   Confusing bits (lowercase `b`, e.g., `Mb`) with bytes (uppercase `B`, e.g., `MB`).
*   Assuming that `100 Mbps` bandwidth guarantees `100 MB` file downloads in 8 seconds.

### Implementation Notes
In C, the `uint8_t` type defined in `<stdint.h>` represents exactly 1 byte (8 bits). For network programming, we must always use fixed-width types (`uint8_t`, `uint16_t`, `uint32_t`) rather than platform-dependent types like `int` or `long`, which can vary in size between architectures (e.g., 32-bit vs. 64-bit systems).

### Debugging Notes
Using a hex editor or a network analyzer like Wireshark allows you to view the raw bytes of a network frame. For example, the hex value `0x45` represents one byte, which in binary is `01000101`.

---

## Endianness

### What It Is
Endianness is the order in which bytes of a multi-byte word (such as a 16-bit or 32-bit integer) are stored in computer memory.
*   **Little-Endian**: The least significant byte (LSB) is stored at the lowest memory address. Used by x86 and ARM architectures.
*   **Big-Endian**: The most significant byte (MSB) is stored at the lowest memory address. This is the standard order for network protocols, also known as **Network Byte Order**.

### Why It Exists
Different CPU architectures were designed by different manufacturers using different conventions. Since modern internet routers and hosts must communicate regardless of their CPU design, a standard "Network Byte Order" (Big-Endian) was established.

### How It Works
Consider the 32-bit hex integer `0x12345678`. It consists of four bytes: `12`, `34`, `56`, and `78`.

```text
Memory Address:   0x00     0x01     0x02     0x03
                  ──────   ──────   ──────   ──────
Big-Endian:        12       34       56       78      (Network Byte Order)
Little-Endian:     78       56       34       12      (Host Byte Order on x86)
```

If a Little-Endian machine sends `0x12345678` raw over the wire without translation, a Big-Endian receiver will interpret it as `0x78563412`.

### Real Packet Flow
1.  **Host Representation**: Host A (x86, Little-Endian) has the port number `80` (hex `0x0050`). In memory, it is stored as `50 00`.
2.  **Conversion**: Host A calls `htons(80)` (Host to Network Short). This swaps the bytes to `00 50`.
3.  **Transmission**: The bytes `00 50` travel over the network.
4.  **Reception**: Host B (x86, Little-Endian) receives `00 50`. It calls `ntohs()` (Network to Host Short), swapping the bytes back to `50 00` for native memory storage.

### Common Mistakes
*   **Failing to convert on local testing**: If you test your server code on a single x86 machine, sending data without conversion might work because both sender and receiver share the same Little-Endian host byte order. The code will fail as soon as it interacts with a Big-Endian system or strict network hardware.
*   **Converting single bytes**: Endianness does not apply to single-byte values (`char`, `uint8_t`). It only applies to multi-byte values (`uint16_t`, `uint32_t`, `float`, etc.).

### Implementation Notes
The standard POSIX socket API provides translation functions:
*   `htons(uint16_t hostshort)`: Host to Network Short (16-bit, e.g., ports)
*   `htonl(uint32_t hostlong)`: Host to Network Long (32-bit, e.g., IPv4 addresses)
*   `ntohs(uint16_t netshort)`: Network to Host Short
*   `ntohl(uint32_t netlong)`: Network to Host Long

Here is how you detect machine endianness in C:

```c
#include <stdio.h>
#include <stdint.h>

int main() {
    uint16_t val = 0x0001;
    uint8_t *ptr = (uint8_t*)&val;
    if (ptr[0] == 0x01) {
        printf("Host is Little-Endian\n");
    } else {
        printf("Host is Big-Endian\n");
    }
    return 0;
}
```

### Debugging Notes
When debugging binary protocols with toolchains, if integer values look completely wrong (e.g., port `80` appearing as port `20480`, which is `0x5000` instead of `0x0050`), you are likely facing an endianness conversion error.

---

# Part 2 - Network Models

## OSI Model vs. TCP/IP Model

### What It Is
The OSI (Open Systems Interconnection) model and the TCP/IP model are conceptual frameworks used to describe and standardize network communications by dividing them into logical layers.

```text
       OSI Model                       TCP/IP Model
┌───────────────────────┐
│  7. Application       │ ───┐
├───────────────────────┤    │
│  6. Presentation      │    ├─> Application Layer
├───────────────────────┤    │   (HTTP, DNS, TLS)
│  5. Session           │ ───┘
├───────────────────────┤
│  4. Transport         │ ─────> Transport Layer (TCP, UDP)
├───────────────────────┤
│  3. Network           │ ─────> Internet Layer (IP, ICMP)
├───────────────────────┤
│  2. Data Link         │ ───┐
├───────────────────────┤    ├─> Link / Network Interface Layer
│  1. Physical          │ ───┘   (Ethernet, ARP, Wi-Fi)
└───────────────────────┘
```

### Why It Exists
Designing a network stack is highly complex. Layering isolates responsibilities. An application developer writing code at the Application Layer does not need to know whether the physical transmission is copper, fiber, or satellite. Similarly, a switch forwarding packets at the Link Layer does not need to understand HTTP headers.

### How It Works
*   **OSI Model (7 Layers)**:
    1.  **Physical**: Transmission of raw bits over physical media.
    2.  **Data Link**: Physical addressing (MAC), error detection, framing.
    3.  **Network**: Logical addressing (IP), routing, path determination.
    4.  **Transport**: End-to-end connections, reliability, flow control (TCP, UDP).
    5.  **Session**: Managing sessions between applications.
    6.  **Presentation**: Data formatting, encryption, compression.
    7.  **Application**: User-facing application protocols.
*   **TCP/IP Model (4 Layers)**:
    1.  **Link (or Network Interface)**: Combines OSI Layers 1 and 2.
    2.  **Internet**: Maps to OSI Layer 3.
    3.  **Transport**: Maps to OSI Layer 4.
    4.  **Application**: Combines OSI Layers 5, 6, and 7.

### Real Packet Flow (Encapsulation)
When sending data, each layer takes the payload from the layer above and prepends its own header (and sometimes a trailer):

```text
[HTTP Data]                                                (Application Layer)
      │
      ▼
[TCP Header][HTTP Data]                                    (Transport Layer)
      │
      ▼
[IP Header][TCP Header][HTTP Data]                         (Internet Layer)
      │
      ▼
[Ethernet Header][IP Header][TCP Header][HTTP Data][FCS]    (Link Layer)
```

At the destination, the process is reversed (**Decapsulation**). Each layer parses and strips off its header, passing the remaining payload up the stack.

### Common Mistakes
*   **Assuming strict OSI compliance**: Modern operating systems implement the TCP/IP model. The distinctions between OSI Layers 5, 6, and 7 are often blurred. For example, TLS (encryption) can be viewed as Layer 6 (Presentation), but it is implemented directly within application-level libraries or combined into the application layer logic.

### Implementation Notes
In the Linux kernel network stack, encapsulation is managed using the `sk_buff` (socket buffer) structure. As a packet travels down the stack, the kernel shifts a pointer in `sk_buff` to allocate space for the new header, avoiding costly memory copies of the payload.

### Debugging Notes
Wireshark displays this layered architecture explicitly in its packet tree view:
*   Frame 1 (Physical properties)
*   Ethernet II (Link Layer)
*   Internet Protocol Version 4 (Internet Layer)
*   Transmission Control Protocol (Transport Layer)
*   Hypertext Transfer Protocol (Application Layer)

---

# Part 3 - Link Layer

## Ethernet

### What It Is
Ethernet is the primary wired technology used in local area networks (LANs). It defines the physical medium (cables, connectors) and the protocols for framing and MAC-level transmission.

### Why It Exists
Ethernet provides a standardized way for multiple devices on a single physical site to communicate directly with one another. It was designed to replace expensive, low-speed serial connections with a shared, high-speed medium.

### How It Works
Ethernet uses physical MAC addresses to identify source and destination nodes. 
Modern Ethernet is based on point-to-point connections through switches, eliminating packet collisions.
An **Ethernet II Frame** (the standard frame type) has the following structure:

```text
┌─────────────────┬─────────────────┬───────────┬─────────────────┬──────────┐
│ Destination MAC │   Source MAC    │ Type/Len  │ Payload (Data)  │   FCS    │
│    (6 Bytes)    │    (6 Bytes)    │ (2 Bytes) │ (46-1500 Bytes) │ (4 Bytes)│
└─────────────────┴─────────────────┴───────────┴─────────────────┴──────────┘
```

*   **Preamble & SFD**: 8 bytes used for hardware synchronization (stripped by the NIC, not visible in software captures).
*   **Destination MAC Address**: 6 bytes.
*   **Source MAC Address**: 6 bytes.
*   **EtherType**: 2 bytes. Identifies the network protocol contained in the payload (e.g., `0x0800` for IPv4, `0x0806` for ARP).
*   **Payload**: Variable size, minimum 46 bytes, maximum 1500 bytes (defined by the standard **MTU**).
*   **FCS (Frame Check Sequence)**: 4 bytes. A cyclic redundancy check (CRC) used to detect transmission errors.

### Real Packet Flow
1.  **Format Frame**: The operating system formats the frame, appending the EtherType `0x0800` (IPv4).
2.  **Append MACs**: It looks up the Destination MAC for the target IP (using the ARP cache) and sets the Source MAC to its own NIC hardware address.
3.  **Physical Sent**: The NIC serializes the frame, appends the preamble and FCS, and transmits it.
4.  **Hardware Validation**: The receiving NIC calculates the CRC of the incoming signal. If it matches the FCS, it strips the FCS and passes the frame to the kernel driver; otherwise, the NIC silently drops the frame.

### Common Mistakes
*   **Ignoring the minimum frame size**: Ethernet frames must be at least 64 bytes long (excluding preamble). If the payload is smaller than 46 bytes, the NIC driver must pad the payload with null bytes (`0x00`) to reach the minimum size limit.

### Implementation Notes
In C, the Ethernet header can be represented by `struct ethhdr` defined in `<linux/if_ether.h>`:

```c
struct ethhdr {
    unsigned char h_dest[6];   /* destination eth addr */
    unsigned char h_source[6]; /* source ether addr    */
    uint16_t      h_proto;    /* packet type ID field */
} __attribute__((packed));
```

### Debugging Notes
*   To see the MAC addresses of your local interfaces, run:
    ```bash
    ip link show
    ```

---

## MAC Addresses

### What It Is
A MAC (Media Access Control) address is a unique, 48-bit (6-byte) physical hardware address assigned to a network interface controller (NIC) at the time of manufacture.

### Why It Exists
While IP addresses are logical and can change depending on where a device connects, MAC addresses are physically bound to the network hardware. They provide a reliable hardware identifier to handle point-to-point delivery within a local network segment.

### How It Works
A MAC address is represented as six groups of two hexadecimal digits, separated by colons or hyphens (e.g., `00:1A:2B:3C:4D:5E`).
*   **Organizationally Unique Identifier (OUI)**: The first 3 bytes are assigned by the IEEE to the hardware manufacturer (e.g., `00:1A:2B` might be registered to Cisco).
*   **Network Interface Controller (NIC) Specific**: The last 3 bytes are a unique serial number assigned by the manufacturer.

```text
┌───────────────────────────┬───────────────────────────┐
│     OUI (Manufacturer)    │      NIC Serial Number    │
│  00  :  1A  :  2B  :  3C  │    4D  :  5E  :  6F  :  70   │
└───────────────────────────┴───────────────────────────┘
```

Types of MAC Addresses:
1.  **Unicast**: Targeted at a single NIC.
2.  **Multicast**: Targeted at a group of devices (the least significant bit of the first byte is set to `1`).
3.  **Broadcast**: Targeted at all devices on the local segment (`FF:FF:FF:FF:FF:FF`).

### Real Packet Flow
When a host transmits a frame, it sets the destination MAC. Switches on the path read the destination MAC to forward the frame only to the port where that MAC is connected. If the destination MAC is `FF:FF:FF:FF:FF:FF`, the switch floods the frame out of all ports.

### Common Mistakes
*   **Assuming MAC addresses are globally unique**: While designed to be unique, MAC addresses can be changed in software ("MAC spoofing") or duplicated due to manufacturing errors.
*   **Trying to route MAC addresses across routers**: MAC addresses only have meaning within the local layer-2 collision/broadcast domain. Routers strip Layer 2 headers and replace them with new ones when forwarding across network boundaries.

### Implementation Notes
To change a MAC address temporarily in Linux:
```bash
ip link set dev eth0 down
ip link set dev eth0 address 00:11:22:33:44:55
ip link set dev eth0 up
```

### Debugging Notes
To look up the manufacturer of a device using its MAC address, you can query public OUI databases (like `macvendors.com`).

---

## Switches

### What It Is
A network switch is a Layer 2 device that forwards Ethernet frames between devices on a local network.

### Why It Exists
Early networks used hubs. A hub simply replicates incoming electrical signals from one port and broadcasts them to all other ports, leading to frequent packet collisions and poor performance. A switch segments collision domains by tracking where MAC addresses reside, forwarding frames only to the relevant port.

### How It Works
Switches maintain a **CAM (Content Addressable Memory) Table**, mapping MAC addresses to physical switch ports.
1.  **MAC Learning**: When a frame arrives on Port 1 from MAC `A`, the switch records `MAC A -> Port 1` in its CAM table.
2.  **Forwarding Lookup**: If a frame is destined for MAC `B`, the switch searches its CAM table.
    *   *If found*: It forwards the frame only to the mapped port.
    *   *If not found (Unknown Unicast)*: It floods the frame out of all ports except the source port.
3.  **CAM Aging**: Entries are deleted if no frames are received from that MAC within a specific timeout (typically 300 seconds).

```text
CAM Table:
MAC Address        Port
─────────────────  ────
00:1A:2B:3C:4D:5E  1
00:1A:2B:3C:4D:5F  2

Port 1 (MAC A) ───[Switch]─── Port 2 (MAC B)
                     │
                  Forwarded only to Port 2
```

### Real Packet Flow
1. Host A transmits to Host B.
2. The Switch receives the frame, reads the source MAC of Host A, and updates its CAM table.
3. The Switch reads the destination MAC of Host B, finds it in the CAM table, and builds an internal circuit to route the frame directly to Host B's port.

### Common Mistakes
*   **Confusing Switches with Routers**: Switches work at Layer 2 (MAC addresses) and connect devices on the *same* network. Routers work at Layer 3 (IP addresses) and connect *different* networks.

### Implementation Notes
High-performance switches perform forwarding in hardware using ASICs (Application-Specific Integrated Circuits), allowing wire-speed forwarding with minimal latency.

### Debugging Notes
On managed switches, you can dump the CAM table. In Linux (if using a Linux system bridge acting as a switch), you can view MAC tables via:
```bash
brctl showmacs <bridge_name>
```
or
```bash
bridge fdb show
```

---

## VLANs

### What It Is
A VLAN (Virtual Local Area Network) is a technology that allows a single physical switch to be partitioned into multiple logical switches, creating isolated broadcast domains.

### Why It Exists
Without VLANs, separating departments or services (like public guest Wi-Fi from internal corporate servers) requires buying entirely separate physical switches and cabling. VLANs allow network administrators to isolate traffic logically over shared hardware.

### How It Works
VLANs use the **IEEE 802.1Q** protocol. When a frame travels over a link that carries traffic for multiple VLANs (a **Trunk Link**), an 802.1Q header is inserted into the Ethernet frame.

```text
Standard Ethernet Frame:
[Src MAC][Dst MAC][Type][Payload]

802.1Q Tagged Ethernet Frame:
[Src MAC][Dst MAC][802.1Q Tag (VLAN ID)][Type][Payload]
```

*   **VLAN ID (VID)**: A 12-bit field specifying the VLAN (values 1 to 4094).
*   **Access Port**: A switch port assigned to a single VLAN. Frames entering or leaving this port are untagged (the switch manages VLAN assignment internally).
*   **Trunk Port**: A port configured to carry traffic for multiple VLANs. Frames traversing this link must be tagged so the receiving switch knows which VLAN they belong to.

### Real Packet Flow
1.  **Untagged Entry**: Host A (VLAN 10) sends an untagged frame to Switch 1 on an access port.
2.  **Tagging**: Switch 1 tags the frame with VLAN ID `10`.
3.  **Trunk Transit**: The tagged frame travels across a trunk link to Switch 2.
4.  **Untagged Delivery**: Switch 2 receives the tagged frame, identifies it as VLAN `10`, strips the tag, and forwards it to Host B on an access port configured for VLAN `10`.

### Common Mistakes
*   **VLAN Leaking**: Configuring native VLANs incorrectly on trunk ports can cause frames from one VLAN to bleed into another, posing a security risk.

### Implementation Notes
In Linux, you can create a virtual VLAN interface linked to a physical network card:
```bash
ip link add link eth0 name eth0.10 type vlan id 10
ip link set dev eth0.10 up
```
This routes all traffic sent to `eth0.10` through the physical interface `eth0` with 802.1Q tag 10 automatically appended.

### Debugging Notes
Use `tcpdump` to capture VLAN tagged frames on a trunk interface:
```bash
tcpdump -i eth0 -e vlan 10
```

---

## ARP

### What It Is
ARP (Address Resolution Protocol) is a Layer 2 protocol used to resolve a known Layer 3 logical address (IPv4) to a Layer 2 physical address (MAC).

### Why It Exists
When an application wants to send data, it specifies a destination IP address (e.g., `192.168.1.5`). However, the local network card can only send Ethernet frames using destination MAC addresses. ARP acts as the translator between these two layers.

### How It Works
Hosts maintain an **ARP Cache** (a local lookup table mapping IP addresses to MAC addresses).
1.  **Cache Lookup**: Host A wants to send data to `192.168.1.5`. It checks its local ARP cache.
    *   *If found*: It builds the Ethernet frame and sends it.
    *   *If not found*: It sends an **ARP Request**.
2.  **ARP Request (Broadcast)**: The request asks: *"Who has 192.168.1.5? Tell 192.168.1.1."* This is sent as a broadcast frame to `FF:FF:FF:FF:FF:FF`.
3.  **ARP Reply (Unicast)**: The host with `192.168.1.5` receives the broadcast, records Host A's IP/MAC mapping in its own cache, and replies: *"I have 192.168.1.5, my MAC is 00:11:22:33:44:55."*
4.  **Cache Update**: Host A receives the reply, updates its ARP cache, and sends the queued Ethernet frame.

```text
Host A (192.168.1.1)                           Host B (192.168.1.5)
      │                                              │
      ├─────── ARP Request (Broadcast: Who has?) ────>│
      │                                              │
      <─────── ARP Reply (Unicast: I have it!) ──────┤
```

### Real Packet Flow
An ARP packet structure:
*   **Hardware Type**: Ethernet (`0x0001`).
*   **Protocol Type**: IPv4 (`0x0800`).
*   **Hardware Size**: 6 bytes.
*   **Protocol Size**: 4 bytes.
*   **Opcode**: Request (`1`) or Reply (`2`).
*   **Sender MAC** & **Sender IP**.
*   **Target MAC** & **Target IP**.

### Common Mistakes
*   **ARP Spoofing / Poisoning**: ARP is stateless and lacks authentication. A malicious host can send unsolicited ARP replies (Gratuitous ARP) claiming: *"I have the gateway IP 192.168.1.1, my MAC is 00:AA:BB:CC:DD:EE."* All hosts on the network will update their caches, sending their outbound internet traffic to the attacker (Man-in-the-Middle attack).

### Implementation Notes
ARP is only used for IPv4. In IPv6, ARP is replaced by **Neighbor Discovery Protocol (NDP)**, which runs over ICMPv6 multicast.

### Debugging Notes
*   To view the ARP cache in Linux, run:
    ```bash
    ip neighbor show
    ```
    or
    ```bash
    arp -an
    ```
*   To clear the ARP cache entry for an IP:
    ```bash
    ip neighbor delete 192.168.1.5 dev eth0
    ```

---

# Part 4 - Internet Layer

## IPv4 vs. IPv6

### What It Is
IPv4 (Internet Protocol Version 4) and IPv6 (Internet Protocol Version 6) are Layer 3 protocols that provide logical addressing and routing across network boundaries.

### Why It Exists
Every device on a routed network must have a unique logical identifier. IPv4 addresses are 32-bit, providing about 4.3 billion addresses. Because of the explosive growth of connected devices, IPv4 addresses have been exhausted. IPv6 was designed with a 128-bit address space to provide a virtually limitless supply of addresses.

### How It Works
*   **IPv4 Addresses**: 32-bit, written in dotted-decimal notation (e.g., `192.168.1.1`).
*   **IPv6 Addresses**: 128-bit, written in hexadecimal colon-separated groups (e.g., `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).

```text
IPv4 Header (Minimum 20 Bytes):
┌─────────┬────────┬─────────────┬───────────────────────────┐
│ Version │  IHL   │   DSCP/ECN  │       Total Length        │
├─────────┴────────┼─────────────┼───────────────────────────┤
│    Identification│    Flags    │      Fragment Offset      │
├──────────────────┼─────────────┼───────────────────────────┤
│   Time to Live   │   Protocol  │      Header Checksum      │
├──────────────────┴─────────────┴───────────────────────────┤
│                     Source IP Address                      │
├────────────────────────────────────────────────────────────┤
│                  Destination IP Address                    │
└────────────────────────────────────────────────────────────┘

IPv6 Header (Fixed 40 Bytes):
┌──────────────────┬─────────────┬───────────────────────────┐
│ Version │ Traffic Class │               Flow Label          │
├──────────────────┼─────────────┼───────────────────────────┤
│   Payload Length │ Next Header │         Hop Limit         │
├──────────────────┴─────────────┴───────────────────────────┤
│                                                            │
│                     Source IPv6 Address                    │
│                                                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│                  Destination IPv6 Address                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

Differences:
*   **Fragmentation**: In IPv4, routers on the path can fragment packets if they exceed the MTU. In IPv6, routers do not fragment; hosts must perform Path MTU Discovery (PMTUD) using ICMPv6 packet-too-big messages.
*   **Header Size**: IPv4 has a variable header size (due to options fields), requiring an Internet Header Length (IHL) field. IPv6 has a fixed 40-byte header, simplifying hardware routing speed.
*   **Checksum**: IPv6 removed the header checksum to speed up processing (since Layer 2 and Layer 4 already provide error detection).

### Real Packet Flow
When an IP packet is sent:
1.  **Route Match**: The OS looks up the destination IP in its routing table to find the next-hop router.
2.  **Decrement TTL**: For each router traversed, the router decrements the TTL (IPv4) or Hop Limit (IPv6) by 1.
3.  **Drop & ICMP**: If TTL reaches 0, the packet is discarded, and the router sends an ICMP "Time Exceeded" message back to the sender.

### Common Mistakes
*   **Assuming native compatibility**: IPv4 and IPv6 are not backward compatible. An IPv4-only device cannot communicate directly with an IPv6-only device without translation mechanisms like NAT64.

### Implementation Notes
In C, IPv4 and IPv6 use different address structures:
*   IPv4: `struct sockaddr_in` in `<netinet/in.h>`.
*   IPv6: `struct sockaddr_in6` in `<netinet/in.h>`.

```c
struct sockaddr_in {
    sa_family_t    sin_family; /* AF_INET */
    in_port_t      sin_port;   /* Port number (Big-Endian) */
    struct in_addr sin_addr;   /* IPv4 Address struct */
};

struct in_addr {
    uint32_t       s_addr;     /* 32-bit IPv4 address (Big-Endian) */
};
```

### Debugging Notes
*   View IPv4 and IPv6 addresses on interfaces:
    ```bash
    ip addr show
    ```
*   Force tools to use IPv4 (`-4`) or IPv6 (`-6`):
    ```bash
    curl -6 https://icanhazip.com
    ```

---

## CIDR and Subnetting

### What It Is
CIDR (Classless Inter-Domain Routing) is a method for allocating IP addresses and routing packets. It replaced the older class-based routing system (Class A, B, C) with variable-length subnet masking.

### Why It Exists
Class-based routing was wasteful. If an organization needed 300 IP addresses, it had to request a Class B block (65,536 addresses) because a Class C block (256 addresses) was too small. CIDR allows arbitrary block sizes.

### How It Works
A CIDR address is written as an IP address followed by a slash and a decimal number (the **Subnet Mask Prefix**), e.g., `192.168.1.0/24`.
*   The prefix (e.g., `/24`) specifies the number of bits reserved for the network portion of the address.
*   The remaining bits (e.g., $32 - 24 = 8$ bits) are available for host addresses.

For `192.168.1.0/24`:
*   **Subnet Mask**: `255.255.255.0` (in binary: 24 ones followed by 8 zeros).
*   **Total IP Addresses**: $2^{8} = 256$ addresses.
*   **Network Address**: `192.168.1.0` (the first address, used to identify the subnet).
*   **Broadcast Address**: `192.168.1.255` (the last address, used to send packets to all hosts on the subnet).
*   **Usable Host Range**: `192.168.1.1` to `192.168.1.254` ($256 - 2 = 254$ usable hosts).

```text
CIDR /26 Subnet Example: 192.168.1.0/26
Network bits: 26. Host bits: 6.
Total addresses: 2^6 = 64. Usable hosts: 64 - 2 = 62.
Netmask: 255.255.255.192
Network Addr: 192.168.1.0
Broadcast Addr: 192.168.1.63
Usable host range: 192.168.1.1 - 192.168.1.62
```

### Real Packet Flow
When Host A (`192.168.1.5/24`) wants to send to Host B (`192.168.1.10`), it applies its subnet mask to Host B's IP address:
`192.168.1.10 AND 255.255.255.0 = 192.168.1.0`
This matches Host A's subnet (`192.168.1.0`), so it communicates directly via Layer 2 ARP.
If Host A wants to send to Host C (`8.8.8.8`), the match fails:
`8.8.8.8 AND 255.255.255.0 = 8.8.8.0`
Since this is a different subnet, Host A sends the packet to its **Default Gateway** (router) instead.

### Common Mistakes
*   **Using network or broadcast addresses for hosts**: Configuring a host IP to `.0` or `.255` in a `/24` subnet will fail or cause routing loops.
*   **Forgetting to subtract 2**: Usable host calculations must always exclude the network and broadcast addresses.

### Implementation Notes
In software, subnet calculations are implemented using fast bitwise operations:
```c
uint32_t ip = inet_addr("192.168.1.5");
uint32_t mask = inet_addr("255.255.255.0");
uint32_t network = ip & mask;
```

### Debugging Notes
*   Use `ipcalc` to verify subnet boundaries, network addresses, and usable ranges:
    ```bash
    ipcalc 192.168.1.5/26
    ```

---

## Routing

### What It Is
Routing is the process of forwarding Layer 3 packets across network boundaries from a source host to a destination host.

### Why It Exists
Devices in different local networks cannot communicate directly via MAC addresses. Routers act as crossing points, reading the destination IP address of packets and determining the next logical hop along the path.

### How It Works
Routers make decisions using a local **Routing Table**. Each entry in the routing table has:
1.  **Destination Subnet** (CIDR block).
2.  **Subnet Mask**.
3.  **Gateway** (Next Hop IP).
4.  **Interface** (Physical card to send the traffic out).
5.  **Metric** (Cost value; lower metric routes are preferred).

When a packet arrives:
*   The router checks the destination IP against all subnets in the routing table.
*   It performs a **Longest Prefix Match** (LPM). If the destination matches multiple entries (e.g., `10.0.0.0/8` and `10.1.0.0/16`), it chooses the route with the most specific prefix (the larger mask value, `/16`).
*   It decrements the TTL, recalculates the IP header checksum, builds a new Layer 2 frame pointing to the next hop's MAC, and forwards the packet.

```text
                    [Routing Table: Longest Prefix Match]
                       Target IP: 10.1.5.100
                       Routes:
                       - 10.0.0.0/8    -> Gateway 192.168.1.1
                       - 10.1.0.0/16   -> Gateway 192.168.2.1  <-- MATCH (More specific)
```

### Real Packet Flow
1.  **Generate Packet**: Host A (`192.168.1.5`) sends a packet to `8.8.8.8`.
2.  **Send to Gateway**: No local subnet match. Host A sends the packet to Router 1 (`192.168.1.1`).
3.  **Forwarding Loop**: Router 1 decrements TTL, checks its routing table, and forwards it to Router 2.
4.  **Delivery**: The final router finds a local match (e.g., `8.8.8.8` is directly connected to one of its interfaces) and sends it directly to the host.

### Common Mistakes
*   **Asymmetric Routing**: Packets from Host A to Host B travel along Path 1, but replies from Host B to Host A travel along Path 2. While valid, this can cause stateful firewalls on the path to drop packets because they only see one half of the connection.
*   **Routing Loops**: If Router A forwards a packet to Router B, and Router B's table points back to Router A, the packet will bounce back and forth until its TTL reaches 0.

### Implementation Notes
In Linux, the routing table is managed by the kernel. You can inspect it with:
```bash
ip route show
```

### Debugging Notes
*   `traceroute`: Sends packets with incrementing TTL values ($1, 2, 3...$). Each hop decrements the TTL, drops the packet, and sends back an ICMP "Time Exceeded" message, allowing you to map the path to a host:
    ```bash
    traceroute -n 8.8.8.8
    ```

---

## ICMP

### What It Is
ICMP (Internet Control Message Protocol) is a Layer 3 protocol used by network devices to send error messages and operational information.

### Why It Exists
IP is a best-effort, connectionless protocol with no built-in feedback loop. If a packet cannot be delivered because a gateway is down or the packet is too large for the link MTU, the sender needs to be notified. ICMP provides this diagnostic and error channel.

### How It Works
ICMP does not use TCP or UDP. It is encapsulated directly inside IP packets (IP Protocol field is set to `1`).
ICMP messages consist of:
*   **Type** (8 bits): The broad category of message (e.g., Type `8` is Echo Request, Type `0` is Echo Reply, Type `3` is Destination Unreachable).
*   **Code** (8 bits): Sub-category details (e.g., Type `3` Code `1` is Host Unreachable, Type `3` Code `4` is Fragmentation Needed).
*   **Checksum** (16 bits): Error verification.
*   **Data**: Contains the original IP header and the first 8 bytes of the packet that caused the error, helping the sender identify which connection failed.

```text
┌─────────────────┬─────────────────┬────────────────────────────────┐
│   Type (8 bits) │   Code (8 bits) │       Checksum (16 bits)       │
├─────────────────┴─────────────────┴────────────────────────────────┤
│                     Rest of Header (Variable)                      │
├────────────────────────────────────────────────────────────────────┤
│                       Data (Variable Size)                         │
└────────────────────────────────────────────────────────────────────┘
```

### Real Packet Flow (Ping)
1.  **Echo Request**: Host A sends an ICMP Type `8` (Echo Request) to `192.168.1.5`.
2.  **Processing**: The receiver's OS intercepts the ICMP request and constructs an ICMP Type `0` (Echo Reply) response.
3.  **Echo Reply**: The receiver sends the reply back to Host A.

### Common Mistakes
*   **Blocking all ICMP**: Many system administrators configure firewalls to block all ICMP packets to prevent discovery scans. However, blocking Type `3` Code `4` (Destination Unreachable / Fragmentation Needed) breaks **Path MTU Discovery** (PMTUD), causing connections to hang or fail when transferring large packets.

### Implementation Notes
Because ICMP uses raw IP transmission, creating ICMP packets in user space requires creating a raw socket, which usually requires administrative privileges:
```c
int raw_socket = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
```

### Debugging Notes
*   To check if a host is alive:
    ```bash
    ping -c 4 8.8.8.8
    ```
*   To perform MTU testing by setting the "Don't Fragment" flag:
    ```bash
    ping -M do -s 1472 8.8.8.8
    ```
    (1472 bytes payload + 8 bytes ICMP header + 20 bytes IP header = 1500 bytes MTU limit).

---

## NAT

### What It Is
NAT (Network Address Translation) is a method of mapping private IP addresses to a single public IP address (or group of public IPs) during transmission across network boundaries.

### Why It Exists
The IPv4 address space is exhausted. To solve this, **RFC 1918** reserved three IP ranges for private local networks:
*   `10.0.0.0/8`
*   `172.16.0.0/12`
*   `192.168.0.0/16`
Private IPs cannot be routed on the public Internet. NAT translates these private IPs to a public IP, allowing thousands of local network hosts to share a single public IPv4 address.

### How It Works
The most common implementation is **PAT (Port Address Translation)**, also known as NAT Overload:
1.  **Outgoing Packet**: Host A (`192.168.1.100`) sends a request from source port `12345` to a public web server (`8.8.8.8:80`).
2.  **Translation**: The NAT router intercepts the packet. It changes the Source IP to its public IP (`203.0.113.1`) and changes the source port to a unique value (`50001`) to track the session.
3.  **NAT Table Update**: The router writes this mapping to its translation table.
4.  **Reply Routing**: When the web server replies to `203.0.113.1:50001`, the NAT router checks its table, translates the destination IP back to `192.168.1.100:12345`, and forwards the packet to Host A.

```text
Host A (192.168.1.100:12345) ───> [NAT Router: Public 203.0.113.1] ───> Destination (8.8.8.8:80)
                                    Maps: 192.168.1.100:12345 <-> 203.0.113.1:50001
```

| Source IP | Source Port | Translated IP | Translated Port | Destination IP | Destination Port |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `192.168.1.100` | `12345` | `203.0.113.1` | `50001` | `8.8.8.8` | `80` |

### Real Packet Flow
*   **Outbound**: `Source: 192.168.1.100:12345, Destination: 8.8.8.8:80` $\to$ NAT Router $\to$ `Source: 203.0.113.1:50001, Destination: 8.8.8.8:80`.
*   **Inbound**: `Source: 8.8.8.8:80, Destination: 203.0.113.1:50001` $\to$ NAT Router $\to$ `Source: 8.8.8.8:80, Destination: 192.168.1.100:12345`.

### Common Mistakes
*   **Assuming NAT is a security firewall**: While NAT blocks unsolicited incoming traffic (since there is no translation table mapping for incoming connections that aren't initiated locally), it is not a stateful firewall. It is a translation mechanism.
*   **Breaking Protocols**: Some protocols (like FTP or SIP) embed IP addresses inside their application payloads. A standard NAT router only changes headers, which can break these protocols unless the router has an active Application Layer Gateway (ALG) helper enabled.

### Implementation Notes
NAT is managed in Linux using `iptables` or `nftables` via the `netfilter` subsystem:
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
This configures Linux to perform PAT on all packets exiting the `eth0` public interface.

### Debugging Notes
*   To view current NAT translation states in Linux, run:
    ```bash
    conntrack -L
    ```

---

# Part 5 - Transport Layer

## Ports

### What It Is
A port is a 16-bit unsigned integer (ranging from `0` to `65535`) used by transport protocols (like TCP and UDP) to identify specific application processes on a host.

### Why It Exists
IP addresses route packets to a specific physical host. However, a single host runs multiple network services (e.g., a web server, an SSH daemon, a database). Ports act as logical sub-addresses, allowing the OS to multiplex incoming traffic to the correct application process.

### How It Works
Ports are categorized by the Internet Assigned Numbers Authority (IANA):
*   **Well-Known Ports (0 - 1023)**: Reserved for system/privilege services (e.g., System privileges are required to bind to these in Unix).
    *   `22`: SSH
    *   `53`: DNS
    *   `80`: HTTP
    *   `443`: HTTPS
*   **Registered Ports (1024 - 49151)**: Used by user processes or applications (e.g., `3306` for MySQL, `5432` for PostgreSQL).
*   **Dynamic/Ephemeral Ports (49152 - 65535)**: Temporarily allocated by the operating system for client connections.

```text
[Client Host]                                        [Server Host (IP: 203.0.113.5)]
  ├── App (Browser) ──> Source Port: 53123 ───┐        ├── Web Server <── Bind Port: 80
  └── App (SSH Client) ──> Source Port: 53124 ┼───>    └── SSH Server <── Bind Port: 22
                                              │
                         Routed to IP ────────┘
```

### Real Packet Flow
1.  **Client Bind**: A browser opens a connection to `example.com:80`. The client OS selects an unused ephemeral port (e.g., `53120`) as the source port.
2.  **Request Packet**: The packet contains: `Src IP: 192.168.1.5, Src Port: 53120, Dst IP: 93.184.216.34, Dst Port: 80`.
3.  **Response Packet**: The server replies with: `Src IP: 93.184.216.34, Src Port: 80, Dst IP: 192.168.1.5, Dst Port: 53120`. The client OS uses the destination port `53120` to route the data to the browser process.

### Common Mistakes
*   **Port Exhaustion**: A high-traffic client initiating thousands of outbound connections can exhaust the ephemeral port range. The OS will return an `EADDRNOTAVAIL` error when trying to open new sockets.
*   **Binding Permission Denied**: Attempting to run a server binding to port `80` or `443` as a non-root user will fail with `Permission denied` (`EACCES`).

### Implementation Notes
The ephemeral port range on Linux can be viewed or configured via `sysctl`:
```bash
sysctl net.ipv4.ip_local_port_range
```
(Default is often `32768 60999`, though IANA recommends `49152 65535`).

### Debugging Notes
*   To see which process is listening on a specific port (e.g., `80`):
    ```bash
    ss -lntp sport = :80
    ```
    or
    ```bash
    lsof -i :80
    ```

---

## UDP

### What It Is
UDP (User Datagram Protocol) is a simple, connectionless, lightweight transport protocol defined in **RFC 768**.

### Why It Exists
TCP provides reliability but introduces overhead (handshakes, order preservation, retransmissions). UDP exists for applications that prioritize speed and low latency over perfect reliability (e.g., DNS, live media streaming, gaming, routing protocols).

### How It Works
UDP is a thin wrapper around IP. It adds only 8 bytes of header overhead:

```text
UDP Header (8 Bytes):
┌───────────────────────────────┬───────────────────────────────┐
│          Source Port          │       Destination Port        │
│           (2 Bytes)           │           (2 Bytes)           │
├───────────────────────────────┼───────────────────────────────┤
│            Length             │           Checksum            │
│           (2 Bytes)           │           (2 Bytes)           │
└───────────────────────────────┴───────────────────────────────┘
```

*   **Length**: The length of the UDP header plus data (minimum value is `8`).
*   **Checksum**: Optional in IPv4 (but mandatory in IPv6), used for detecting packet corruption.

Properties of UDP:
*   **Connectionless**: No handshakes. Packets are sent blindly.
*   **Unreliable**: Packets can be lost, duplicated, or arrive out of order.
*   **Message-Oriented**: Preserves boundaries. One `send()` call on a UDP socket maps to exactly one IP packet.

### Real Packet Flow
1.  **Application Send**: A DNS client calls `sendto()` with a 30-byte request payload to port `53`.
2.  **UDP Layer**: Prepends the 8-byte UDP header.
3.  **IP Layer**: Prepends IP header (total `30 + 8 + 20 = 58` bytes) and transmits.
4.  **Delivery**: The receiver gets the packet intact or not at all. If it arrives, the application gets exactly the 30-byte payload.

### Common Mistakes
*   **Expecting fragmentation handling**: If a UDP datagram exceeds the path MTU, the IP layer must fragment it. If a single fragment is lost, the entire datagram is discarded. To avoid this, UDP payloads should be kept below `1472` bytes (or `512` bytes for DNS to guarantee transit).

### Implementation Notes
A UDP socket is created in C using `SOCK_DGRAM`:
```c
int fd = socket(AF_INET, SOCK_DGRAM, 0);
```

### Debugging Notes
*   Monitor UDP statistics (e.g., packet receive errors or buffer overflows):
    ```bash
    netstat -su
    ```

---

## TCP

### What It Is
TCP (Transmission Control Protocol) is a connection-oriented, reliable, byte-stream transport protocol defined in **RFC 793**.

### Why It Exists
IP is unreliable and packet-based. TCP exists to provide applications with a reliable virtual byte-stream interface. It guarantees that data is delivered in order, without duplicates, and without errors, managing underlying network congestion automatically.

### How It Works
TCP uses a 20-byte minimum header:

```text
TCP Header (Minimum 20 Bytes):
┌───────────────────────────────┬───────────────────────────────┐
│          Source Port          │       Destination Port        │
├───────────────────────────────┴───────────────────────────────┤
│                        Sequence Number                        │
├───────────────────────────────────────────────────────────────┤
│                     Acknowledgment Number                     │
├───────┬───────┬───────────────┬───────────────────────────────┤
│ Data  │Reser- │     Flags     │            Window             │
│Offset │  ved  │               │                           │
├───────┴───────┼───────────────┴───────────────────────────────┤
│   Checksum    │         Urgent Pointer        │
└───────────────┴───────────────────────────────────────────────┘
```

*   **Sequence Number**: Tracks the number of bytes sent. Allows ordering and duplicate detection.
*   **Acknowledgment Number**: Identifies the next expected byte from the peer.
*   **Flags (Control Bits)**:
    *   `SYN`: Synchronize sequence numbers (connection setup).
    *   `ACK`: Acknowledgment field is valid.
    *   `FIN`: Finish (orderly shutdown).
    *   `RST`: Reset (abrupt connection termination).
*   **Window**: Advertises the sender's current receive buffer size (used for flow control).

---

## TCP Handshake & Teardown

### 3-Way Handshake
Establishes connection state and negotiates initial sequence numbers (ISN) and MSS (Maximum Segment Size).

```text
Client                                  Server
  │                                       │
  │ ─── SYN (Seq=X, MSS=1460) ──────────> │  (Server enters SYN_RCVD)
  │                                       │
  │ <── SYN-ACK (Seq=Y, Ack=X+1) ─────────┤  (Client enters ESTABLISHED)
  │                                       │
  │ ─── ACK (Seq=X+1, Ack=Y+1) ─────────> │  (Server enters ESTABLISHED)
```

### 4-Way Teardown
Terminates a connection gracefully. Both directions of the full-duplex link must be closed independently.

```text
Active Close (Client)                    Passive Close (Server)
  │                                       │
  │ ─── FIN (Seq=U, Ack=V) ─────────────> │  (Server enters CLOSE_WAIT)
  │                                       │
  │ <── ACK (Seq=V, Ack=U+1) ─────────────┤  (Client enters FIN_WAIT_2)
  │                                       │
  │ <── FIN (Seq=W, Ack=U+1) ─────────────┤  (Server enters LAST_ACK)
  │                                       │
  │ ─── ACK (Seq=U+1, Ack=W+1) ─────────> │  (Client enters TIME_WAIT)
  │                                       │
  │ ─── (Wait 2 * MSL) ─────────────────> │  (Socket fully closed)
```

---

## Flow & Congestion Control

### Flow Control (Sliding Window)
Prevents the sender from overwhelming the receiver's buffer. The receiver advertises its available buffer space in the `Window` field of every ACK. The sender must not have more unacknowledged bytes in-flight than this advertised window size.

```text
Sender Buffer:
[ Sent & Acked ] [ Sent & Unacked ] [ Can Send Immediately ] [ Cannot Send Yet ]
                 └─────── Window Size (In-Flight Limit) ─────┘
```

### Congestion Control
Prevents the sender from overwhelming the intermediate network routers.
*   **Slow Start**: Starts with a small Congestion Window (`cwnd`). For every received ACK, `cwnd` is doubled (exponential growth) until it hits the slow start threshold (`ssthresh`).
*   **Congestion Avoidance**: Once above `ssthresh`, `cwnd` grows linearly (adds 1 segment size per RTT) to probe network limits safely.
*   **Fast Retransmit**: If the sender receives three duplicate ACKs for a segment, it assumes that segment was lost and retransmits it immediately without waiting for a retransmission timeout (RTO).
*   **Fast Recovery**: Temporarily reduces `cwnd` rather than resetting it to `1` upon duplicate ACK detection (maintaining pipe throughput).

---

## TCP State Machine

```text
                   ┌──────────────┐
                   │    CLOSED    │
                   └──────┬───────┘
                          │ listen() / connect()
                          ▼
                   ┌──────────────┐
                   │    LISTEN    │
                   └──────┬───────┘
                          │ recv SYN / send SYN-ACK
                          ▼
                   ┌──────────────┐
                   │   SYN_RCVD   │
                   └──────┬───────┘
                          │ recv ACK
                          ▼
                   ┌──────────────┐
                   │ ESTABLISHED  │
                   └──────┬───────┘
            active close  │   passive close (recv FIN)
            ┌─────────────┴─────────────┐
            ▼                           ▼
     ┌─────────────┐             ┌─────────────┐
     │ FIN_WAIT_1  │             │ CLOSE_WAIT  │
     └──────┬──────┘             └──────┬──────┘
            │ recv ACK                  │ close()
            ▼                           ▼
     ┌─────────────┐             ┌─────────────┐
     │ FIN_WAIT_2  │             │  LAST_ACK   │
     └──────┬──────┘             └──────┬──────┘
            │ recv FIN                  │ recv ACK
            ▼                           ▼
     ┌─────────────┐             ┌─────────────┐
     │  TIME_WAIT  │             │   CLOSED    │
     └──────┬──────┘             └─────────────┘
            │ 2 * MSL timeout
            ▼
     ┌─────────────┐
     │   CLOSED    │
     └─────────────┘
```

### Common Mistakes
*   **Assuming `TIME_WAIT` is a leak**: The `TIME_WAIT` state is normal and necessary. It prevents delayed duplicate packets from a closed connection from corrupting a new connection using the same ports. It lasts for $2 \times \text{MSL}$ (Maximum Segment Lifetime, typically 1 to 2 minutes).
*   **Underestimating `CLOSE_WAIT` accumulation**: If your server socket shows thousands of connections stuck in `CLOSE_WAIT`, it means the client initiated a close (sent FIN), but your application code has not closed its side of the socket (forgot to call `close()` on the client file descriptor).

### Implementation Notes
The default TCP congestion control algorithm on modern Linux kernels is **BBR** (Bottleneck Bandwidth and RTT) or **cubic**. You can inspect and change this:
```bash
sysctl net.ipv4.tcp_congestion_control
```

---

# Part 6 - DNS

## DNS Architecture

### What It Is
The Domain Name System (DNS) is a hierarchical, distributed database used to translate human-readable hostnames (e.g., `www.example.com`) to machine-routable IP addresses (e.g., `93.184.216.34`).

### Why It Exists
Humans remember names, but network routers route packets using binary IP addresses. Maintaining a static local mapping file (like `/etc/hosts`) is impossible at global scale. DNS provides a scalable, decentralized, and cached look-up infrastructure.

### How It Works
The DNS namespace is organized as a tree structure:

```text
                              . (Root)
                                 │
           ┌─────────────────────┴─────────────────────┐
          .com                                        .org
           │                                           │
      example.com                                 wikipedia.org
           │                                           │
    www.example.com                             en.wikipedia.org
```

*   **Root Servers**: Manage the root zone (`.`). They know the authoritative servers for all Top-Level Domains (TLDs).
*   **TLD Servers**: Authoritative for specific extensions (e.g., `.com`, `.net`, `.org`).
*   **Authoritative Name Servers**: Managed by domain owners or DNS hosting providers. They store the actual mapping records for a domain.

---

## Resolution Process

```text
Browser              Stub Resolver         Recursive Resolver          Authoritative
  │                        │                       │                         │
  ├─ Query example.com ───>│                       │                         │
  │                        ├─ Query (No cache) ───>│                         │
  │                        │                       ├─ Ask Root ('.') ───────>│
  │                        │                       <─ Refer to TLD ('.com') ─┤
  │                        │                       │                         │
  │                        │                       ├─ Ask TLD ('.com') ─────>│
  │                        │                       <─ Refer to Auth ns ──────┤
  │                        │                       │                         │
  │                        │                       ├─ Ask Authoritative ────>│
  │                        │                       <─ Return IP (1.2.3.4) ───┤
  │                        │                       │                         │
  │                        <─ Return IP (1.2.3.4) ─┤                         │
  <─ Return IP (1.2.3.4) ──┤                       │                         │
```

### DNS Record Types
*   **A**: Maps a hostname to an IPv4 address.
*   **AAAA**: Maps a hostname to an IPv6 address.
*   **CNAME** (Canonical Name): Alias pointing to another domain name.
*   **MX** (Mail Exchanger): Identifies mail servers for the domain.
*   **TXT**: Arbitrary text records (used for SPF, DKIM, site verification).
*   **NS**: Identifies the authoritative name servers for a zone.
*   **SRV**: Identifies service locations (port and hostname).

### Common Mistakes
*   **CNAME loops**: Creating CNAME records that point to each other (e.g., `A` points to `B` and `B` points to `A`) will crash DNS resolvers attempting to resolve the query.
*   **TTL caching lag**: Changing an authoritative record with a high TTL (Time to Live) means resolvers worldwide will serve the cached stale IP until the TTL expires.

### Implementation Notes
DNS queries are typically sent over UDP port `53`. If the response size exceeds 512 bytes, the client retries the query over TCP port `53`.

### Debugging Notes
*   Query domain records directly using `dig`:
    ```bash
    dig A www.example.com +trace
    ```
    This shows the delegation path from root down to authoritative.

---

# Part 7 - HTTP

## HTTP Fundamentals

### What It Is
HTTP (Hypertext Transfer Protocol) is an application-layer protocol used to transfer hypermedia documents on the Web.

### Why It Exists
HTTP provides a common grammar for web browsers and servers. It defines how requests are formatted, how resources are identified (via URIs), and how servers must communicate status and metadata.

### How It Works
HTTP/1.1 is a plain-text, request-response protocol running over TCP.
A typical HTTP Request consists of:

```text
GET /index.html HTTP/1.1\r\n
Host: www.example.com\r\n
User-Agent: Mozilla/5.0\r\n
Accept: text/html\r\n
\r\n
```

A typical HTTP Response consists of:

```text
HTTP/1.1 200 OK\r\n
Content-Type: text/html; charset=UTF-8\r\n
Content-Length: 125\r\n
Connection: keep-alive\r\n
\r\n
<html>...
```

> [!IMPORTANT]
> The line terminator for HTTP headers MUST be a Carriage Return and Line Feed: `\r\n` (CRLF). An empty line (`\r\n\r\n`) separates the headers from the payload body.

---

## HTTP Methods, Headers, Status Codes

### Methods
*   `GET`: Retrieve a resource. Must be safe and idempotent.
*   `POST`: Create a new resource or submit data. Neither safe nor idempotent.
*   `PUT`: Replace a resource completely or create it if missing. Idempotent.
*   `DELETE`: Remove a resource. Idempotent.
*   `HEAD`: Retrieve only the response headers (payload excluded).

### Common Headers
*   `Host`: Specifies the target domain (mandatory in HTTP/1.1 to support virtual hosting).
*   `Content-Length`: The size of the request/response body in bytes.
*   `Transfer-Encoding: chunked`: Indicates that data is sent in dynamic chunks, avoiding the need for a pre-calculated `Content-Length`.
*   `Cookie` / `Set-Cookie`: Manages state across requests.
*   `Cache-Control`: Controls client and proxy caching behavior.

### Status Codes
*   `1xx` (Informational): Request received, continuing process (e.g., `101 Switching Protocols`).
*   `2xx` (Success): Request successfully processed (e.g., `200 OK`, `201 Created`).
*   `3xx` (Redirection): Action required to complete request (e.g., `301 Moved Permanently`, `304 Not Modified`).
*   `4xx` (Client Error): Request contains bad syntax or cannot be fulfilled (e.g., `400 Bad Request`, `401 Unauthorized`, `404 Not Found`).
*   `5xx` (Server Error): Server failed to fulfill an apparently valid request (e.g., `500 Internal Server Error`, `502 Bad Gateway`).

### Common Mistakes
*   **Failing to handle chunked encoding**: When writing an HTTP parser, assuming that `Content-Length` is always present will cause your parser to crash or hang on chunked streams.
*   **Parsing body before headers are complete**: Trying to parse the request body before validating that the entire double CRLF (`\r\n\r\n`) separating headers has arrived can lead to truncation bugs.

### Implementation Notes
HTTP/1.1 uses **Keep-Alive** by default. A TCP connection remains open after a request-response cycle, allowing subsequent requests to reuse the same connection to save TCP handshake overhead.

### Debugging Notes
*   Send manual HTTP requests using `curl`:
    ```bash
    curl -v http://www.example.com/
    ```

---

# Part 8 - HTTPS

## Cryptography Foundations

### What It Is
HTTPS is the secure version of HTTP. It uses TLS (Transport Security Layer) to encrypt all communications.

### Why It Exists
Plaintext HTTP traffic can be sniffed, modified, or hijacked by anyone on the network path. HTTPS guarantees:
1.  **Encryption (Confidentiality)**: Eavesdroppers cannot read the data.
2.  **Integrity**: Data cannot be modified on the wire without detection.
3.  **Authentication**: Verification that you are talking to the real owner of the site.

### How It Works
HTTPS combines:
*   **Symmetric Cryptography** (e.g., AES, ChaCha20): Fast encryption using a shared secret key. Used for bulk data transfer.
*   **Asymmetric Cryptography** (e.g., RSA, ECDH): Slow encryption using public/private key pairs. Used to authenticate the server and securely exchange the shared symmetric key.
*   **Cryptographic Hash Functions** (e.g., SHA-256): Ensures data integrity.

---

## TLS Handshake

### TLS 1.2 Handshake (2 RTT)

```text
Client                                             Server
  │                                                  │
  ├─ ClientHello (Supported Ciphers, ClientRandom) ─>│
  │                                                  │
  │ <─ ServerHello (Selected Cipher, ServerRandom) ──┤
  │ <─ Certificate & ServerKeyExchange ──────────────┤
  │ <─ ServerHelloDone ──────────────────────────────┤
  │                                                  │
  ├─ ClientKeyExchange (PreMasterSecret) ───────────>│
  ├─ [ChangeCipherSpec] & Finished ─────────────────>│
  │                                                  │
  │ <─ [ChangeCipherSpec] & Finished ────────────────┤
  │                                                  │
  ├─ Encrypted Application Data ────────────────────>│
```

### TLS 1.3 Handshake (1 RTT)
TLS 1.3 optimizes the flow by guessing key exchange parameters in the first message.

```text
Client                                             Server
  │                                                  │
  ├─ ClientHello + Key Share (ClientRandom) ────────>│
  │                                                  │
  │ <─ ServerHello + Key Share (ServerRandom) ───────┤
  │ <─ EncryptedExtensions & Certificate ─────────────┤
  │ <─ Finished ─────────────────────────────────────┤
  │                                                  │
  ├─ Encrypted Application Data ────────────────────>│
```

---

## Public Key Infrastructure (PKI)

A client validates the server's identity using **Certificates** issued by trusted **Certificate Authorities (CAs)**.
1.  **Certificate Details**: Contains the server's public key, the domain name, expiration date, and the CA's digital signature.
2.  **Chain of Trust**:
    *   The browser has pre-installed trusted **Root CA Certificates** (self-signed).
    *   The server sends its certificate along with **Intermediate CA Certificates**.
    *   The browser verifies the signature on the server's certificate using the intermediate public key, then verifies the intermediate certificate using the root public key.

```text
[Root CA Certificate (Pre-installed in OS)]
         │ Signs
         ▼
[Intermediate CA Certificate]
         │ Signs
         ▼
[Server Certificate (e.g., example.com)]
```

### Common Mistakes
*   **Ignoring Certificate Expiry/Validation**: Failing to properly validate peer certificates in custom C code (e.g., bypassing certificate verification flags in `libcurl` or OpenSSL) leaves applications completely vulnerable to Man-in-the-Middle attacks.
*   **Mismatch Domain**: Generating a certificate for `example.com` but serving it on `sub.example.com` will trigger browser security warnings.

### Implementation Notes
In a C application, OpenSSL is the standard library used for TLS integration. Socket file descriptors are wrapped in `SSL` pointers:
```c
SSL_CTX *ctx = SSL_CTX_new(TLS_client_method());
SSL *ssl = SSL_new(ctx);
SSL_set_fd(ssl, socket_fd);
SSL_connect(ssl);
```

### Debugging Notes
*   Inspect a remote TLS certificate chain:
    ```bash
    openssl s_client -connect www.example.com:443 -showcerts
    ```

---

# Part 9 - Server Engineering

## Sockets & I/O Models

### What It Is
A socket is the software abstraction provided by the OS kernel that represents an endpoint for bidirectional communication. I/O models define how applications interact with these sockets to read and write data.

### Why It Exists
Network traffic is asynchronous and latency-bound. The OS kernel needs to bridge the gap between fast CPU execution and slow, unpredictable network transit. Different I/O models allow developers to choose between simple synchronous logic (blocking I/O) and high-performance asynchronous designs (non-blocking I/O and multiplexing).

### How It Works
*   **Blocking I/O**: The default model. When an application calls `recv()`, the thread blocks (is suspended by the scheduler) until data arrives in the kernel receive queue.
*   **Non-Blocking I/O**: Sockets configured with the `O_NONBLOCK` flag. If no data is available in the receive queue, `recv()` returns immediately with the error `EAGAIN` or `EWOULDBLOCK`. The application must poll or use multiplexing.
*   **I/O Multiplexing**: Monitoring multiple file descriptors simultaneously using a single system call.
    *   `select()` (Legacy): Uses a fixed-size bitmask (`fd_set`). Limited to 1024 file descriptors by default (`FD_SETSIZE`). $O(n)$ complexity because the kernel must scan the entire set on every call, and the application must re-initialize the sets.
    *   `poll()` (Legacy): Replaces bitmasks with an array of `pollfd` structs, lifting the 1024 limit. Still $O(n)$ complexity.
    *   `epoll()` (Linux Specific): Highly scalable. Uses an internal event-poll structure in the kernel (implemented as a red-black tree and a ready list). Registration is $O(\log n)$, and retrieving ready events is $O(1)$. It supports **Level-Triggered** (notifies as long as data remains in the queue) and **Edge-Triggered** (notifies only when state changes from empty to readable).
    *   `kqueue()` (BSD/macOS Specific): Similar to `epoll`, uses `kevent` structures to track generic kernel events.

```text
                     [epoll Mechanism (O(1) Concurrency)]
                       User Space         Kernel Space
                           │                   │
                     epoll_create() ─────────> Creates Red-Black Tree
                           │                   │
                     epoll_ctl(ADD) ─────────> Registers FD to monitor
                           │                   │ (Kernel appends to tree)
                           │                   │
                     epoll_wait()  ──────────> Blocks on Ready List
                           │                   │ (Interrupt handlers add
                           <── Returns Ready ──┤  active FDs to ready list)
```

---

## Event Loops & The Reactor Pattern

High-performance servers use the **Reactor Pattern**. A single thread runs an **Event Loop** that uses `epoll` to monitor multiple sockets. When an I/O event occurs, the loop dispatches the event to a registered callback handler.

```text
                  [Reactor Event Loop]
                  ┌─────────────────┐
                  │   Event Loop    │ <─── Events (e.g., FD readable)
                  └────────┬────────┘
                           │ Dispatches
                           ▼
                  ┌─────────────────┐
                  │   Dispatcher    │
                  └────────┬────────┘
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        [Handler 1]   [Handler 2]   [Handler 3]
```

### Common Mistakes
*   **Blocking the event loop thread**: Since a single thread manages all connections, running a heavy computation or blocking database call inside a handler will freeze the entire server, stalling all active connections.
*   **Level-Triggered vs. Edge-Triggered bugs**: When using Edge-Triggered epoll, you must read/write to a socket in a loop until it returns `EAGAIN` or `EWOULDBLOCK`. If you read only once, you will never receive subsequent notifications, leaving data stuck in the kernel queue.

### Implementation Notes
`epoll` uses three core system calls:
*   `epoll_create1(int flags)`: Creates the epoll instance.
*   `epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)`: Configures events (ADD, MOD, DEL).
*   `epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)`: Waits for events.

### Debugging Notes
*   Monitor system calls in real-time to inspect multiplexer activity:
    ```bash
    strace -e trace=epoll_wait,accept4,read,write -p <PID>
    ```

---

# Part 10 - Building Servers

## Low-Level C Socket Server

Below is a complete, production-grade C implementation of an asynchronous TCP echo server using `epoll` in edge-triggered mode. It demonstrates non-blocking socket configuration, robust handling of partial `recv`/`send` buffers, ignoring `SIGPIPE`, and avoiding resource leaks.

### C Implementation (`server.c`)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <signal.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#define MAX_EVENTS 64
#define BUFFER_SIZE 4096
#define PORT 8080

/* Helper function to set a socket to non-blocking mode */
int set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}

/* Gracefully handle signals to prevent zombie processes and clean sockets */
void setup_signals() {
    // Ignore SIGPIPE to prevent the server from crashing when clients disconnect abruptly
    struct sigaction sa;
    sa.sa_handler = SIG_IGN;
    sa.sa_flags = 0;
    sigemptyset(&sa.sa_mask);
    if (sigaction(SIGPIPE, &sa, NULL) == -1) {
        perror("sigaction SIGPIPE");
        exit(EXIT_FAILURE);
    }
}

int main() {
    setup_signals();

    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    // Set SO_REUSEADDR to bypass the TIME_WAIT socket bind lock on restart
    int opt = 1;
    if (setsockopt(listen_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)) == -1) {
        perror("setsockopt SO_REUSEADDR");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    struct sockaddr_in address;
    memset(&address, 0, sizeof(address));
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY; // Bind to all interfaces
    address.sin_port = htons(PORT);

    if (bind(listen_fd, (struct sockaddr *)&address, sizeof(address)) == -1) {
        perror("bind");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    if (listen(listen_fd, SOMAXCONN) == -1) {
        perror("listen");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    if (set_nonblocking(listen_fd) == -1) {
        perror("set_nonblocking listen_fd");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    struct epoll_event ev, events[MAX_EVENTS];
    ev.events = EPOLLIN | EPOLLET; // Edge-Triggered read monitoring
    ev.data.fd = listen_fd;

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_fd, &ev) == -1) {
        perror("epoll_ctl ADD listen_fd");
        close(listen_fd);
        close(epoll_fd);
        exit(EXIT_FAILURE);
    }

    printf("Server listening on port %d...\n", PORT);

    while (1) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            if (errno == EINTR) continue; // Restart on signal interrupt
            perror("epoll_wait");
            break;
        }

        for (int i = 0; i < nfds; ++i) {
            int current_fd = events[i].data.fd;

            if (current_fd == listen_fd) {
                /* Accept incoming connections */
                while (1) {
                    struct sockaddr_in client_addr;
                    socklen_t client_len = sizeof(client_addr);
                    int client_fd = accept(listen_fd, (struct sockaddr *)&client_addr, &client_len);

                    if (client_fd == -1) {
                        if (errno == EAGAIN || errno == EWOULDBLOCK) {
                            // Handled all pending connections in queue
                            break;
                        } else if (errno == EINTR) {
                            continue; // Retry
                        } else {
                            perror("accept");
                            break;
                        }
                    }

                    if (set_nonblocking(client_fd) == -1) {
                        perror("set_nonblocking client_fd");
                        close(client_fd);
                        continue;
                    }

                    // Register client with Edge-Triggered and EPOLLONESHOT to prevent multi-thread races
                    struct epoll_event client_ev;
                    client_ev.events = EPOLLIN | EPOLLET | EPOLLRDHUP;
                    client_ev.data.fd = client_fd;

                    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &client_ev) == -1) {
                        perror("epoll_ctl ADD client_fd");
                        close(client_fd);
                    } else {
                        printf("Connected client: %d (IP: %s)\n", client_fd, inet_ntoa(client_addr.sin_addr));
                    }
                }
            } else {
                /* Handle data on client socket */
                int closed = 0;
                char buffer[BUFFER_SIZE];

                // Client closed connection or errored
                if (events[i].events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
                    closed = 1;
                } else if (events[i].events & EPOLLIN) {
                    /* Read loop (Edge-triggered MUST read until EAGAIN) */
                    while (1) {
                        ssize_t bytes_read = recv(current_fd, buffer, sizeof(buffer), 0);
                        if (bytes_read == -1) {
                            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                                // Buffer empty, done reading
                                break;
                            } else if (errno == EINTR) {
                                continue; // Interrupted, retry
                            } else {
                                perror("recv");
                                closed = 1;
                                break;
                            }
                        } else if (bytes_read == 0) {
                            // Client closed connection cleanly
                            closed = 1;
                            break;
                        }

                        /* Echo data back (handling partial send writes) */
                        ssize_t bytes_written = 0;
                        while (bytes_written < bytes_read) {
                            ssize_t written = send(current_fd, buffer + bytes_written, bytes_read - bytes_written, MSG_NOSIGNAL);
                            if (written == -1) {
                                if (errno == EAGAIN || errno == EWOULDBLOCK) {
                                    // In a production server, you would buffer the remaining unsent data
                                    // and register for EPOLLOUT to write it when socket is ready.
                                    // For simplicity in this demo, we spin or log.
                                    usleep(1000);
                                    continue;
                                } else if (errno == EINTR) {
                                    continue;
                                } else {
                                    perror("send");
                                    closed = 1;
                                    break;
                                }
                            }
                            bytes_written += written;
                        }
                    }
                }

                if (closed) {
                    printf("Client %d disconnected\n", current_fd);
                    epoll_ctl(epoll_fd, EPOLL_CTL_DEL, current_fd, NULL);
                    close(current_fd);
                }
            }
        }
    }

    close(listen_fd);
    close(epoll_fd);
    return 0;
}
```

---

## Connection Lifecycle Management

*   **SYN Backlog Queue**: The kernel buffer containing connections that have sent a `SYN` packet but have not yet completed the 3-way handshake.
*   **Accept Queue**: Completed 3-way handshake connections waiting to be retrieved by the application calling `accept()`.
*   **Keep-Alive Persistence**: Reusing a single TCP socket for multiple requests. Implementations use timer threads to close idle connections after a configurable window (e.g., Nginx `keepalive_timeout 65s`).

---

# Part 11 - Modern Web Infrastructure

## Reverse Proxies vs. Load Balancers

```text
               ┌───────────────┐
               │    Clients    │
               └──────┬────────┘
                      │ Internet
                      ▼
               ┌───────────────┐
               │ Load Balancer │ (Layer 4/7 Traffic Distribution)
               └──────┬────────┘
                      │ Routed
                      ▼
               ┌───────────────┐
               │ Reverse Proxy │ (Nginx: SSL Termination, Caching)
               └──────┬────────┘
             ┌────────┴────────┐
             ▼                 ▼
     [Backend Server 1]  [Backend Server 2]
```

### Reverse Proxy
An intermediary server positioned between client devices and backend web applications.
*   **Purpose**: SSL/TLS termination, request routing, static caching, header manipulation, rate limiting, and masking backend servers from direct public exposure.

### Load Balancer
Distributes incoming application traffic across multiple backend servers to ensure high availability and reliability.
*   **L4 Load Balancing**: Works at the transport layer (TCP/UDP). Makes routing decisions based on IP address and port fields without inspecting application payloads (very fast, low CPU usage).
*   **L7 Load Balancing**: Works at the application layer (HTTP/HTTPS). Inspects HTTP headers, cookies, and URI paths to make intelligent routing decisions (e.g., sending `/api` traffic to Server pool A, and `/static` to Server pool B).

### CDNs (Content Delivery Networks)
A distributed network of proxy servers deployed globally (edge locations). CDNs cache static files (images, JS, CSS, videos) close to end-users to reduce latency and load on origin servers.

---

# Part 12 - Modern Protocols

## HTTP/2

### What It Is
HTTP/2 is a major revision of the HTTP network protocol, standardized in **RFC 7540**.

### Why It Exists
HTTP/1.1 suffers from **Head-of-Line (HOL) Blocking**. In HTTP/1.1, a client can only send one request at a time over a single TCP connection; subsequent requests must wait in queue. While browsers bypassed this by opening up to six concurrent TCP connections per domain, this was inefficient and resource-heavy.

### How It Works
HTTP/2 replaces plaintext messages with a binary framing layer and multiplexes multiple streams over a single TCP connection:
*   **Binary Framing**: Data is split into smaller, binary-encoded frames (`HEADERS` frame, `DATA` frame, `SETTINGS` frame).
*   **Multiplexing**: Streams are bidirectional sequences of frames. Multiple streams co-exist on a single TCP connection. A lost packet still stalls the entire TCP connection (transport-level HOL blocking), but application-level HOL is solved.
*   **HPACK Header Compression**: Compresses HTTP headers using static and dynamic tables, reducing redundant header transfer.
*   **Pseudo-headers**: Prefixed with colons (e.g., `:method`, `:path`, `:authority`, `:status`) to replace HTTP/1.1 start lines.

```text
HTTP/1.1 Sequential:
Connection: [Request 1] ──> [Response 1] ──> [Request 2] ──> [Response 2]

HTTP/2 Multiplexed:
Connection: [Stream 1: Frame][Stream 3: Frame][Stream 1: Frame][Stream 5: Frame]
```

---

## HTTP/3 and QUIC

### What It Is
HTTP/3 is the upcoming third major version of the HTTP protocol, designed to run over **QUIC** (Quick UDP Internet Connections) rather than TCP.

### Why It Exists
Although HTTP/2 multiplexes streams over a single TCP connection, it suffers from **TCP Head-of-Line Blocking**. If a single packet carrying HTTP/2 data is lost on the network, TCP halts delivery of *all* streams in the connection until the lost packet is retransmitted and acknowledged.

### How It Works
HTTP/3 replaces TCP with QUIC, a transport layer protocol built on top of UDP:
*   **UDP Base**: Avoids kernel TCP stack limitations, allowing fast deployment of protocol updates in user space.
*   **Stream-Level Flow**: QUIC manages reliability and packet recovery on a *per-stream* basis. If a packet for Stream 1 is lost, only Stream 1 blocks. Streams 2, 3, and 4 continue processing without interruption.
*   **Connection Migration**: QUIC identifies connections using a 64-bit **Connection ID** rather than the traditional TCP 4-tuple (source IP, source port, dest IP, dest port). If a user switches from Wi-Fi to cellular data, their IP changes, but their Connection ID remains the same, allowing the transfer to continue without renegotiating a handshake.
*   **0-RTT Handshake**: Integrates the TLS 1.3 handshake directly into the transport layer handshake, allowing clients to send data in their very first packet (zero round trip time).

```text
TCP + TLS Handshake (HTTP/2):
Client                     Server
SYN ─────────────────────>
    <───────────────────── SYN-ACK
ACK ─────────────────────>
ClientHello ─────────────>
    <───────────────────── ServerHello + Cert
Finished ────────────────>
    <───────────────────── Finished  (3 RTT total before first request)

QUIC Handshake (HTTP/3):
Client                     Server
ClientHello (KeyShare) ──>
    <───────────────────── ServerHello (KeyShare) + Finished
Data Sent Immediately ───>           (1 RTT to establish; 0 RTT on resume)
```

---

## WebSockets vs. gRPC

*   **WebSockets**: A protocol running over TCP that provides full-duplex communication channels over a single connection. Initiated via an HTTP Upgrade request (`Upgrade: websocket`). Ideal for real-time web applications (chat, notifications).
*   **gRPC**: A high-performance RPC framework designed by Google. It runs over HTTP/2, uses **Protocol Buffers (Protobuf)** as its interface definition language and binary serialization format, and supports streaming APIs. Ideal for microservice-to-microservice backend communication.

### Common Mistakes
*   **Using WebSockets for simple updates**: WebSockets require maintaining stateful, open TCP connections on servers, which limits scalability. For unidirectional updates, **Server-Sent Events (SSE)** is simpler and runs over standard HTTP.
*   **Failing to configure Ping/Pong**: WebSocket connections can be silently dropped by intermediate NAT routers or stateful firewalls if idle. You must implement periodic ping/pong frames to keep connections alive.

### Debugging Notes
*   Inspect HTTP/2 or HTTP/3 frames in Wireshark by configuring keys to decrypt TLS sessions, or inspect streams in Chrome Developer Tools under the "Network" tab by enabling the "Protocol" column.

---

# Part 13 - Linux Networking

## Kernel Network Stack Path

When a packet arrives at the network interface card (NIC), it travels up the kernel network stack before reaching the user-space application socket.

```text
               User Space
                   │
  [Application] <──┴── Sockets API (sys_recv, sys_send)
───────────────────┼────────────────────────────────────
               Kernel Space
                   │
            [Sockets Layer] (socket, proto_ops)
                   │
           [Transport Layer] (tcp_v4_rcv, udp_rcv)
                   │
            [Internet Layer] (ip_rcv, ip_forward)
                   │
              [Traffic Control / Netfilter / Qdisc]
                   │
             [Device Queue] (napi_gro_receive, netif_receive_skb)
                   │
            [Driver (NIC)] (DMA transfer, ring buffers)
───────────────────┼────────────────────────────────────
              Hardware Layer
                   │
             [Physical NIC]
```

### Path of an Incoming Packet
1.  **Hardware Frame Reception**: The physical NIC receives the bits, performs frame validation (CRC), and copies the frame into kernel memory (the **Ring Buffer**) using Direct Memory Access (DMA).
2.  **Interrupt Handling**: The NIC triggers a hardware interrupt. The CPU runs the driver's interrupt handler, which schedules a soft interrupt (SoftIRQ) via **NAPI** (New API) to poll the ring buffer without thrashing the CPU.
3.  **Socket Buffer (`sk_buff`) Allocation**: The driver wraps the raw data in an `sk_buff` structure and passes it to `netif_receive_skb()`.
4.  **Network Layer Processing**: The packet enters the IP layer (`ip_rcv()`). Netfilter hooks (iptables) are executed. If the packet is destined for the local host, it is passed up; otherwise, it is routed or dropped.
5.  **Transport Layer Processing**: The packet enters the transport handler (e.g., `tcp_v4_rcv()`). The kernel locates the matching socket state using the packet's 4-tuple.
6.  **Socket Queues**: The payload is appended to the socket's **Receive Queue**. The kernel wakes up any thread blocked in `accept()`, `recv()`, or `epoll_wait()`.
7.  **User Space Copy**: The application invokes a read system call (`read`/`recv`), copying the data from the kernel space socket buffer to user-allocated memory.

---

## Network Namespaces & Virtual Interfaces

Linux uses namespaces to implement lightweight virtualization (used by Docker and Kubernetes).

*   **Network Namespaces (`netns`)**: An isolated instance of the network stack, complete with its own routing table, firewall rules, and network interfaces.
*   **Virtual Ethernet Pairs (`veth`)**: A virtual wire connecting two network namespaces. Packets entering one end of a `veth` pair automatically emerge from the other end.
*   **Linux Bridges**: A software implementation of an Ethernet switch. It connects multiple virtual and physical interfaces, forwarding frames using MAC address learning.
*   **TUN/TAP Devices**: Virtual network devices backed by a user-space program rather than a physical wire.
    *   *TUN (Network Tunnel)*: Simulates a Layer 3 (IP) device. Used for VPNs.
    *   *TAP (Network Tap)*: Simulates a Layer 2 (Ethernet) device. Used for virtual machine links.

```text
                  [Container Networking Isolation]
     ┌───────────────────────┐       ┌───────────────────────┐
     │  Namespace: Container │       │    Namespace: Host    │
     │  ┌─────────────────┐  │       │  ┌─────────────────┐  │
     │  │      eth0       │  │       │  │     veth_host   │  │
     └──┴────────┬────────┴──┘       └──┴────────┬────────┴──┘
                 │                               │
                 └────── Virtual veth Link ──────┘
```

### Common Mistakes
*   **Forgetting to set interfaces UP**: Creating a `veth` pair or a bridge interface does not make it active. You must explicitly configure them to the `UP` state:
    ```bash
    ip link set dev veth0 up
    ```
*   **Missing bridge forward rules**: By default, the Linux kernel may block packet forwarding between interfaces on a bridge due to `iptables` rules. You must configure forwarding flags to allow bridged container traffic.

### Implementation Notes
Creating an isolated container network namespace in C:
```c
#define _GNU_SOURCE
#include <sched.h>
#include <unistd.h>

int main() {
    // CLONE_NEWNET creates a new, empty network namespace for the child process
    unshare(CLONE_NEWNET);
    // The child now has its own loopback ('lo') interface, which is offline by default
    execl("/bin/bash", "bash", NULL);
    return 0;
}
```

### Debugging Notes
*   List all network namespaces on a system:
    ```bash
    ip netns list
    ```
*   Run a command inside a specific namespace (e.g., `ns1`):
    ```bash
    ip netns exec ns1 ip addr show
    ```

---

# Part 14 - Containers & Cloud

## Container Networking Architecture

Containers share the same host kernel but run in isolated namespaces. To communicate, they must be linked to the host's physical network using virtual bridges.

```text
 Container A (netns A)                   Container B (netns B)
┌───────────────────────┐               ┌───────────────────────┐
│         eth0          │               │         eth0          │
└──────────┬────────────┘               └──────────┬────────────┘
           │ vethA                                 │ vethB
           ▼                                       ▼
 ┌─────────┴───────────────────────────────────────┴─────────┐
 │                       Host Bridge                         │ (br0 or docker0)
 └─────────────────────────┬─────────────────────────────────┘
                           ▼
                 Host NIC (eth0 / Physical)
                           ▼
                       Internet
```

### Docker Networking Drivers
*   **Bridge (Default)**: Creates a virtual bridge (`docker0`) on the host. Each container gets a `veth` interface, with one end inside the container and the other end attached to the host bridge. Containers communicate via private IPs in the bridge range.
*   **Host**: Removes the network namespace isolation. The container shares the host's network stack directly, binding ports directly to the host's IP address.
*   **Overlay**: Used in multi-host clustering (Swarm/Kubernetes). Encapsulates Layer 2/3 traffic inside standard UDP packets (using **VXLAN**), allowing containers on different physical hosts to communicate on a shared virtual subnet.

---

## Kubernetes Networking & CNIs

Kubernetes implements a **IP-per-Pod** model. Every Pod (a group of containers) has a unique, routable IP address within the cluster, avoiding the need for host port mapping.

*   **CNI (Container Network Interface)**: A pluggable interface standard used by Kubernetes to configure pod networking.
    *   *Flannel*: A simple overlay network provider using VXLAN encapsulation.
    *   *Calico*: A high-performance provider that routes traffic natively without encapsulation using BGP (Border Gateway Protocol) and enforces security policies.
*   **Kubernetes Services**: An abstract way to expose an application running on a set of Pods.
    *   *ClusterIP*: Exposes the service on a cluster-internal IP (load balanced via `iptables` or `IPVS` rules in the host kernel).
    *   *NodePort*: Exposes the service on a static port on each Node's IP.
    *   *LoadBalancer*: Exposes the service externally using a cloud provider's load balancer.

---

# Part 15 - Network Security

## Firewalls & Netfilter

Linux firewalls are implemented via **Netfilter**, a framework inside the kernel that allows kernel modules to register handler hooks at specific stages of the IP stack.

```text
                          [Netfilter Hook Traversal]
                               Incoming Packet
                                     │
                                     ▼
                               [PREROUTING]
                                     │
                     ┌───────────────┴───────────────┐
                     ▼ Route Target                  ▼ Route Target
               (Local Host)                    (Forward/Routing)
                     │                               │
                     ▼                               ▼
                  [INPUT]                        [FORWARD]
                     │                               │
                     ▼                               ▼
                Local Process                  Outgoing Interface
                     │                               │
                     ▼                               │
                 [OUTPUT]                            │
                     │                               │
                     └───────────────┬───────────────┘
                                     ▼
                               [POSTROUTING]
                                     │
                                     ▼
                              Outgoing Packet
```

### Netfilter Tables
1.  **Filter**: Default table for blocking/allowing packets (Chains: `INPUT`, `FORWARD`, `OUTPUT`).
2.  **NAT**: Used for translating source/destination addresses (Chains: `PREROUTING`, `POSTROUTING`, `OUTPUT`).
3.  **Mangle**: Used for specialized packet alterations (e.g., modifying TTL or TOS fields).
4.  **Raw**: Used for configuring exemptions from connection tracking.

---

## DDoS Mitigation

*   **SYN Flood**: A denial-of-service attack where an attacker floods a server with `SYN` requests but never completes the handshakes, filling the server's SYN backlog queue and blocking legitimate connections.
*   **SYN Cookies**: A mitigation technique where the server does not allocate memory state when a `SYN` arrives. Instead, it encodes connection parameters inside the initial Sequence Number (ISN) of the `SYN-ACK` reply. When the client returns the final `ACK`, the server validates the cookie in the acknowledgment number and allocates the connection state.
*   **Rate Limiting**: Dropping packets from IPs that exceed connection limits using token-bucket filters.

---

# Part 16 - Observability

## Command-Line Tool Reference

### 1. `tcpdump`
A command-line packet analyzer. Uses `libpcap` to capture traffic.
*   *Capture all HTTP traffic on port 80*:
    ```bash
    tcpdump -i eth0 -nn -vv 'tcp port 80'
    ```
*   *Save capture to file for Wireshark inspection*:
    ```bash
    tcpdump -i any -w capture.pcap
    ```

### 2. `ss` / `netstat`
Inspect socket statistics and active connections. `ss` queries kernel interfaces directly, making it much faster than `netstat` (which parses `/proc`).
*   *List all listening TCP sockets with process IDs*:
    ```bash
    ss -lntp
    ```
*   *View connection state statistics*:
    ```bash
    ss -s
    ```

### 3. `dig`
Perform DNS lookups.
*   *Resolve domain and trace resolution delegation*:
    ```bash
    dig A www.example.com +trace
    ```
*   *Query a specific DNS server (e.g., Cloudflare)*:
    ```bash
    dig @1.1.1.1 google.com
    ```

### 4. `traceroute`
Discover the path packets take to a host.
*   *Fast traceroute using ICMP Echo rather than UDP*:
    ```bash
    traceroute -I -n 8.8.8.8
    ```

### 5. `curl`
Perform HTTP requests.
*   *Print request/response headers, SSL details, and performance statistics*:
    ```bash
    curl -ivs https://example.com/
    ```

---

# Part 17 - Case Studies

## Scenario 1: Accessing `https://example.com`

This case study traces the complete lifecycle of a single secure web request from a clean host boot to page load, showing the interaction across all seven layers of the OSI model.

```text
               [Scenario 1: Complete Packet Journey]
Host                                Router                     Server
 │                                    │                          │
 ├─ Broadcast: Who has Gateway? ─────>│ (ARP Request)            │
 │ <─ Gateway is at MAC_X ────────────┤ (ARP Reply)              │
 │                                    │                          │
 ├─ UDP Query: Where is example.com? ─┼─────────────────────────>│ (DNS Query)
 │ <─ IP Address is 93.184.216.34 ────┼──────────────────────────┤ (DNS Reply)
 │                                    │                          │
 ├─ SYN ──────────────────────────────┼─────────────────────────>│ (TCP Handshake)
 │ <─ SYN-ACK ────────────────────────┼──────────────────────────┤
 ├─ ACK ──────────────────────────────┼─────────────────────────>│
 │                                    │                          │
 ├─ ClientHello (TLS 1.3 Key Share) ──┼─────────────────────────>│ (TLS Handshake)
 │ <─ ServerHello + Certificate ──────┼──────────────────────────┤
 ├─ Finished ─────────────────────────┼─────────────────────────>│
 │                                    │                          │
 ├─ Encrypted HTTP: GET / ────────────┼─────────────────────────>│ (HTTP Request)
 │ <─ Encrypted HTTP: 200 OK ─────────┼──────────────────────────┤ (HTTP Response)
```

### 1. ARP Resolution for Gateway
*   *Trigger*: The host needs to send a packet to a remote IP (`93.184.216.34`), but it does not know the MAC address of its default gateway (`192.168.1.1`).
*   *Action*: Host broadcasts an ARP request: `"Who has 192.168.1.1? Tell 192.168.1.50"`.
*   *Result*: The default gateway router replies with its MAC address (`00:11:22:33:44:55`). The host caches this mapping.

### 2. DNS Name Resolution
*   *Trigger*: The system resolver needs to translate `example.com` to an IP.
*   *Action*: Host constructs a UDP query packet destined for its configured DNS server (e.g., `8.8.8.8:53`). 
*   *Routing*:
    *   *Link Layer*: Frames are addressed to the default gateway's MAC.
    *   *Internet Layer*: IP packets are addressed to the DNS server (`8.8.8.8`).
*   *Result*: The DNS server replies with a DNS response containing the A record: `example.com -> 93.184.216.34`.

### 3. TCP 3-Way Handshake
*   *Trigger*: The browser initiates a connection to the target web server on port `443` (HTTPS).
*   *Action*:
    *   Host sends a `SYN` packet containing its Initial Sequence Number (ISN) and MSS options.
    *   The web server replies with a `SYN-ACK` containing its own ISN and confirming the client's parameters.
    *   Host returns an `ACK`, completing the handshake and transitioning the connection state to `ESTABLISHED`.

### 4. TLS 1.3 Cryptographic Handshake
*   *Action*:
    *   Host sends a `ClientHello` packet containing supported TLS versions, encryption ciphers, and a temporary public key share (ECDH).
    *   The server responds with a `ServerHello` agreeing on the cipher, returning its own public key share, sending its certificate chain, and generating a shared secret key.
    *   Both sides switch to symmetric encryption immediately. The server completes with an encrypted `Finished` verification packet.
    *   Host validates the server's certificate chain using pre-installed root CA certificates and returns its own `Finished` packet.

### 5. Encrypted HTTP Request & Response
*   *Action*:
    *   The browser formats an HTTP/1.1 request: `GET / HTTP/1.1\r\nHost: example.com\r\n\r\n`.
    *   The TLS layer encrypts this payload and wraps it in a TLS application data record.
    *   The TCP layer segments this encrypted payload, appends sequence headers, and transmits.
    *   The server decapsulates the frames, decrypts the TLS record, parses the HTTP request, and generates a response payload (HTML content).
    *   The response travels back along the same path, decrypted by the host browser for page rendering.

---

## Scenario 2: Container-to-Container Communication Across Hosts

This case study traces how a container on Host A communicates with a container on Host B across a physical network using an overlay overlay VXLAN tunnel.

```text
Container A (IP: 10.0.0.2)                                Container B (IP: 10.0.0.3)
   │                                                                         ▲
   │ (Raw Frame)                                                             │ (Decapsulated)
   ▼                                                                         │
[vethA_host]                                                            [vethB_host]
   │                                                                         ▲
   ▼                                                                         │
[VXLAN Tunnel Interface]                                                 [VXLAN Interface]
   │ Encapsulates Layer 2 Frame in Layer 4 UDP                               │
   ▼                                                                         │
[Host A NIC (IP: 192.168.1.10)] ───> [Physical Network] ───> [Host B NIC (IP: 192.168.1.11)]
```

### 1. Packet Emission
*   Container A (`10.0.0.2`) on Host A attempts to send an IP packet to Container B (`10.0.0.3`) on Host B.
*   Since `10.0.0.3` is on the same logical overlay subnet, Container A broadcasts an ARP request for `10.0.0.3`.
*   The CNI agent on Host A (running a proxy ARP daemon) replies with a virtual MAC address representing the destination overlay endpoint.
*   Container A constructs the Ethernet frame and writes it to its `eth0` interface. The data exits the container via the host-side `vethA_host` virtual pair.

### 2. VXLAN Encapsulation
*   The frame is routed to the host's virtual VXLAN interface.
*   The VXLAN driver reads the destination virtual MAC and looks up the physical IP address of Host B in its overlay routing table.
*   The driver wraps the entire original Layer 2 Ethernet frame inside a standard Layer 4 UDP packet:
    *   *Outer Destination IP*: Host B's physical IP (`192.168.1.11`).
    *   *Outer Destination Port*: Standard VXLAN port `4789`.
    *   *VXLAN Header*: Contains the VXLAN Network Identifier (VNI, e.g., `100`).
*   The outer packet is sent out Host A's physical network card.

### 3. Transit and Decapsulation
*   The physical network routes the standard UDP packet from Host A to Host B.
*   Host B's physical NIC receives the packet and passes it to the kernel UDP stack on port `4789`.
*   The VXLAN driver on Host B intercept the payload, reads the VNI, strips the outer UDP, IP, and VXLAN headers, and recovers the exact original Layer 2 frame sent by Container A.
*   Host B forwards the raw frame to the virtual switch (bridge) associated with VNI `100`.
*   The bridge forwards the frame down `vethB_host` into Container B's network namespace, where it is read by Container B's application.

 
[diff_block_end]
