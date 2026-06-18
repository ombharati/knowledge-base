# Packet Journeys: Step-by-Step Traces through Network Topologies

This guide documents the technical journey of network packets through different topologies. Each section traces the exact path, header modifications, kernel function calls, and hardware interactions for a specific network scenario.

---

## Table of Contents
*   [[#Journey 1: Local Loopback (127.0.0.1 to 127.0.0.1)]]
*   [[#Journey 2: Same-Subnet LAN Communication (Host A to Host B)]]
*   [[#Journey 3: Cross-Subnet Internet Traversal (Host A to Web Server via Gateway)]]
*   [[#Journey 4: Container Network Namespace to Host Bridge]]
*   [[#Journey 5: Overlay Networking (VxLAN Cross-Host Tunneling)]]
*   [[#Journey 6: Load Balancer / NAT (DNAT & SNAT Flow)]]

---

## Journey 1: Local Loopback (127.0.0.1 to 127.0.0.1)

The loopback interface (`lo`) allows applications running on the same host to communicate using standard network protocols without transmitting data over a physical network.

### 1. Conceptual Flow & Path Diagram

```text
[ Client App ]                                       [ Server App ]
      │                                                    ▲
 1. send(fd, buf, len)                                6. recv(fd, buf, len)
      │                                                    │
      ▼                                                    │
[ Socket & VFS Layer ]                                     │
      │                                                    │
      ▼                                                    │
[ Transport (tcp_sendmsg) ]                                │
      │                                                    │
      ▼                                                    │
[ Network (ip_queue_xmit) ]                           [ tcp_v4_rcv() ]
      │                                                    ▲
      ▼                                                    │
[ Route Resolve: RTN_LOCAL ]                           [ ip_rcv() ]
      │                                                    ▲
      ▼                                                    │
[ loopback_xmit() ] ───► [ netif_rx() ] ───► [ SoftIRQ ] ──┘
 (Driver Short-Circuit)       (Queue)         (NET_RX_SOFTIRQ)
```

---

### 2. Step-by-Step Trace

#### Step 1: The Syscall and Transport Handoff
1.  A client application calls `send(fd, buffer, length, 0)` on a socket connected to `127.0.0.1:80`.
2.  The Virtual File System (VFS) routes the call to the protocol-specific handler, **`tcp_sendmsg()`**.
3.  `tcp_sendmsg()` allocates a socket buffer (`sk_buff`), copies the payload from user-space memory, and partitions it into TCP segments.

#### Step 2: Routing Decision
1.  The TCP layer passes the `sk_buff` to the IP layer via **`ip_queue_xmit()`**.
2.  The network layer queries the routing table via `__ip_route_output_key()`.
3.  The routing table resolves the destination address `127.0.0.1` as a local IP, returning a route type of **`RTN_LOCAL`**. This route references the virtual loopback device (`loopback_dev`, interface `lo`).
4.  The routing system sets the packet's destination cache output handler to `ip_output()`.

#### Step 3: Loopback Driver Short-Circuit
1.  The IP layer processes the packet through the Netfilter output hooks and calls **`ip_output()`**, which routes it to the device transmit queue.
2.  Because the interface is `lo`, the stack bypasses physical queuing disciplines (Qdiscs) and invokes the loopback driver's transmit method directly: **`loopback_xmit()`**.
3.  Inside **`loopback_xmit()`**:
    *   **Statistics Update**: The loopback driver updates its transmission statistics counters (TX packets, TX bytes).
    *   **Socket Buffer Orphaning**: The driver calls **`skb_orphan()`**. This detaches the `sk_buff` from the client socket's memory limits, freeing up buffer space so the client application does not block during reception.
    *   **Device Re-assignment**: The packet's incoming device pointer is changed: `skb->dev = skb->dst->dev;` (assigning the receiving device to `lo`).
    *   **Queue Injection**: The driver calls **`netif_rx()`** (or `netif_rx_ni()` if in thread context). This injects the packet directly into the CPU's local incoming backlog queue (`softnet_data.input_pkt_queue`).

#### Step 4: The Reception Path
1.  The call to `netif_rx()` schedules the receive SoftIRQ: **`NET_RX_SOFTIRQ`**.
2.  The SoftIRQ handler invokes **`net_rx_action()`**, which drains the CPU's input queue.
3.  The packet is passed to **`__netif_receive_skb_core()`**, which identifies the protocol (IPv4) and routes the packet to the IP handler: **`ip_rcv()`**.
4.  `ip_rcv()` verifies the packet, traverses Netfilter hooks, and calls `ip_local_deliver()`, which passes the packet to the TCP layer: **`tcp_v4_rcv()`**.
5.  `tcp_v4_rcv()` matches the packet to the server's listening socket, inserts the packet into the receive queue (`sk->sk_receive_queue`), and calls **`sk->sk_data_ready()`** to wake up the server application.

---

### 3. Header Transformations

```text
+-----------------------------+-----------------------------+--------------------+
| Loopback / L2 (Skipped)     | IP Header                   | TCP Header         |
|                             | Src: 127.0.0.1              | Src Port: 49152    |
| (No MAC addresses resolved) | Dest: 127.0.0.1             | Dest Port: 80      |
+-----------------------------+-----------------------------+--------------------+
```

*   **Layer 2 (Ethernet) Header**: No Layer 2 MAC addresses are resolved or built. The loopback driver skips L2 encapsulation because the packet never leaves the kernel.
*   **IP Header**: Remains unchanged during transit. The source IP is `127.0.0.1` and the destination IP is `127.0.0.1`.
*   **TCP Header**: Contains the client's ephemeral port (e.g., `49152`) and the server's listening port (e.g., `80`).

---

## Journey 2: Same-Subnet LAN Communication (Host A to Host B)

This scenario documents how two devices on the same physical Local Area Network (LAN) subnet communicate directly without crossing a router.

### 1. Conceptual Flow & Path Diagram

```text
[ Host A (192.168.1.10) ]                         [ Host B (192.168.1.20) ]
  MAC: 00:11:22:33:44:AA                           MAC: 55:66:77:88:99:BB
          │                                                  ▲
     (TX Path)                                           (RX Path)
          │                                                  │
          ▼                                                  │
 ┌─────────────────┐                                ┌─────────────────┐
 │   Host A NIC    │                                │   Host B NIC    │
 └────────┬────────┘                                └────────▲────────┘
          │                                                  │
          │             ┌──────────────────────┐             │
          └────────────►│  Layer 2 LAN Switch  │─────────────┘
                        │                      │
                        │ MAC Learning:        │
                        │ - Port 1: ...44:AA   │
                        │ - Port 2: ...99:BB   │
                        └──────────────────────┘
```

---

### 2. Step-by-Step Trace

#### Step 1: Subnet Identification
1.  An application on Host A (`192.168.1.10/24`) initiates a TCP connection to Host B (`192.168.1.20/24`).
2.  Host A's network stack performs a bitwise logical AND calculation on the target IP and its own subnet mask:
    `192.168.1.20 & 255.255.255.0 = 192.168.1.0`
    `192.168.1.10 & 255.255.255.0 = 192.168.1.0`
3.  Since the resulting network addresses match, the stack determines the destination resides on the **same local subnet**, meaning it can bypass routing gateways and deliver the packet directly.

#### Step 2: Neighbor Address Resolution (ARP)
Host A must determine Host B's physical Layer 2 MAC address.
1.  **ARP Cache Lookup**: Host A queries its internal ARP cache (kernel neighbor table, inspectable via `/proc/net/arp` or `ip neigh`).
    *   **Case A: Cache Hit**: If the MAC address `55:66:77:88:99:BB` is registered, the stack immediately constructs the L2 Ethernet header.
    *   **Case B: Cache Miss**:
        1.  The kernel suspends the outgoing packet, placing it on a queue inside the neighbor subsystem.
        2.  Host A sends an **ARP Request** broadcast:
            *   **L2 Destination MAC**: `FF:FF:FF:FF:FF:FF` (Broadcast)
            *   **EtherType**: `0x0806` (ARP)
            *   **Payload**: *"Who has 192.168.1.20? Tell 192.168.1.10."*
        3.  The physical Layer 2 Switch receives the broadcast frame on Port 1 and floods it out of all other active ports.
        4.  Host B receives the broadcast frame. Its ARP subsystem processes the query, notices its own IP matching the request, and inserts Host A's IP-to-MAC mapping into its own local ARP table.
        5.  Host B sends a unicast **ARP Reply** back to Host A:
            *   **L2 Destination MAC**: `00:11:22:33:44:AA` (Host A MAC)
            *   **L2 Source MAC**: `55:66:77:88:99:BB` (Host B MAC)
            *   **Payload**: *"192.168.1.20 is at 55:66:77:88:99:BB."*
        6.  The switch forwards this unicast frame directly to Host A.
        7.  Host A processes the reply, writes Host B's MAC address to its ARP cache, and releases the suspended packet for transmission.

#### Step 3: Frame Assembly & Driver Queueing
1.  The IP layer calls `dst_neigh_output()`. The MAC header is prepended to the packet via **`skb_push()`**:
    *   Destination MAC: `55:66:77:88:99:BB`
    *   Source MAC: `00:11:22:33:44:AA`
    *   EtherType: `0x0800` (IPv4)
2.  The packet is queued on the interface Qdisc (`dev_queue_xmit()`) and handed to the physical network card driver via **`ndo_start_xmit()`**.
3.  The driver registers the buffer's physical RAM coordinates via DMA, updates the TX ring buffer, and alerts the hardware controller.

#### Step 4: Switch Processing
1.  The physical network switch receives the frame on Port 1.
2.  **MAC Learning**: The switch inspects the source MAC (`00:11:22:33:44:AA`), notes it is attached to Port 1, and updates its local MAC address table.
3.  **Forwarding Lookup**: The switch inspects the destination MAC (`55:66:77:88:99:BB`) in its MAC table.
    *   It locates the entry pointing to Port 2 and forwards the frame exclusively out of Port 2 (avoiding network flooding).

#### Step 5: Ingress Execution
1.  Host B's physical NIC receives the electrical/optical signals.
2.  **MAC Filter Check**: The NIC validates that the destination MAC matches its own address (`55:66:77:88:99:BB`) or a broadcast address. Since it matches, it accepts the frame.
3.  The NIC initiates a DMA transfer to copy the frame into a pre-allocated receive ring buffer in host RAM and raises a HardIRQ.
4.  The kernel processes the packet up the stack, passing it from the driver to the network layer (`ip_rcv()`), and finally to the destination socket's receive buffer.

---

### 3. Header Transformations

```text
+---------------------------+---------------------------+-------------------+
| Ethernet / L2 Header      | IP Header                 | TCP Header        |
| Dest: 55:66:77:88:99:BB   | Src: 192.168.1.10         | Src Port: 50123   |
| Src:  00:11:22:33:44:AA   | Dest: 192.168.1.20        | Dest Port: 22     |
| EtherType: 0x0800         |                           |                   |
+---------------------------+---------------------------+-------------------+
```

*   **Ethernet Header**: Encapsulates the IP packet. The destination MAC points to Host B, and the source MAC is Host A.
*   **IP Header**: Source IP is `192.168.1.10`, destination IP is `192.168.1.20`.
*   **TCP Header**: Contains Host A's client port (e.g. `50123`) and Host B's SSH listening port (`22`).

---

## Journey 3: Cross-Subnet Internet Traversal (Host A to Web Server via Gateway)

This scenario traces a packet as it exits a local private subnet, crosses a routing default gateway (which performs Source NAT / Masquerading), traverses the public Internet, and reaches a remote server.

### 1. Conceptual Flow & Path Diagram

```text
[ Host A ] ──────► [ LAN Switch ] ──────► [ Default Gateway (Router) ]
192.168.1.10                                 Internal: 192.168.1.1
                                             External: 203.0.113.1
                                                        │
                                                        ▼
[ Web Server ] ◄────── [ ISP WAN Routers ] ◄────────────┘
93.184.216.34
```

---

### 2. Step-by-Step Trace

#### Step 1: DNS Resolution (The Initial Journey)
Before the HTTP request can begin, Host A must translate `example.com` into an IP address.
1.  Host A checks its local `/etc/hosts` file and resolver cache.
2.  If unresolved, it builds a UDP port 53 DNS query packet destined for its configured resolver (e.g. `8.8.8.8`).
3.  The DNS query travels through the gateway, resolves the domain, and returns an IP: **`93.184.216.34`**.

#### Step 2: Route Destination Check
1.  Host A compares the server IP (`93.184.216.34`) with its local network properties using its subnet mask `/24`:
    `93.184.216.34 & 255.255.255.0 = 93.184.216.0` (Mismatch with local `192.168.1.0`).
2.  Determining the destination is remote, Host A queries its routing table (`ip route`).
3.  The routing system matches the packet to the default gateway route entry:
    `default via 192.168.1.1 dev eth0`
4.  This specifies that the packet must be sent to the gateway router IP (`192.168.1.1`) to be forwarded to the Internet.

#### Step 3: Resolving Gateway Layer 2 Address
1.  Host A must build a Layer 2 Ethernet frame. The destination MAC address must be the MAC of the **default gateway** (`192.168.1.1`), **not** the destination web server.
2.  Host A looks up the gateway's IP in its ARP table.
3.  If the gateway MAC is missing, it sends an ARP Request broadcast to resolve `192.168.1.1` to its MAC address: **`AA:BB:CC:DD:EE:11`**.

#### Step 4: Framing & Local Transmission (Host A to Gateway)
1.  Host A constructs the packet.
    *   **Ethernet Header**: Dest MAC = `AA:BB:CC:DD:EE:11` (Gateway), Src MAC = `00:11:22:33:44:AA` (Host A).
    *   **IP Header**: Source IP = `192.168.1.10`, Destination IP = `93.184.216.34`.
2.  The packet is transmitted over the wire, traversing the local LAN switch, which forwards it to the router's internal port.

#### Step 5: Router Execution (Layer 3 Routing & NAT)
The router receives the frame on its internal interface (`eth0`).
1.  **MAC Check**: The router verifies the frame's destination MAC matches its interface MAC. Since it matches, it parses the frame.
2.  **Decapsulation**: The router strips the Layer 2 Ethernet header, leaving the IP packet.
3.  **TTL Decrement**: The router decrements the IP Time-to-Live (TTL) field by 1:
    `TTL: 64 -> 63`
    If the TTL reaches 0, the router discards the packet and sends an ICMP Type 11 (Time Exceeded) packet back to Host A.
4.  **IP Checksum Update**: The router updates the IP header checksum field because the TTL value changed.
5.  **Source NAT (Masquerading)**: Because `192.168.1.10` is a private, non-routable IP (RFC 1918), the router applies Source NAT (SNAT):
    *   It replaces the source IP with its own public WAN IP: **`203.0.113.1`**.
    *   It allocates an ephemeral port mapping in its NAT tracking table to route incoming replies back to Host A.
6.  **Next-Hop Lookup**: The router queries its routing table for `93.184.216.34`. It resolves the route to its external WAN interface (`eth1`) and identifies the next-hop ISP router's gateway IP.
7.  **L2 Re-encapsulation**: The router resolves the next-hop router's MAC address via ARP, prepends a new L2 header, and transmits the frame out of `eth1`.

#### Step 6: WAN Traversal & Ingress
1.  The packet traverses multiple Autonomous Systems (AS) and ISP routers. Each intermediate router strips the L2 header, decrements the TTL, performs a routing lookup, and prepends a new L2 header matching the next physical link (e.g. Ethernet, MPLS, or POS).
2.  The packet eventually arrives at the Web Server's network boundary switch, which delivers the frame to the server's NIC.
3.  The Web Server processes the packet, notices the source IP is `203.0.113.1` (the router's WAN IP), and sends its reply back to that address.

---

### 3. Header Transformations

#### Phase 1: Host A to Default Gateway
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header           | IP Header                 | TCP Header        |
| Dest: AA:BB:CC:DD:EE:11   | Src: 192.168.1.10         | Src Port: 51200   |
| Src:  00:11:22:33:44:AA   | Dest: 93.184.216.34       | Dest Port: 443    |
+---------------------------+---------------------------+-------------------+
```

#### Phase 2: Gateway to Next Hop (After SNAT)
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header (WAN Link)| IP Header (WAN Link)      | TCP Header        |
| Dest: Next-Hop Router MAC | Src: 203.0.113.1          | Src Port: 61001   |
| Src:  AA:BB:CC:DD:EE:22   | Dest: 93.184.216.34       | Dest Port: 443    |
+---------------------------+---------------------------+-------------------+
```
*(Note how the Source IP and Source Port are rewritten by the NAT gateway, and the Layer 2 addresses are completely replaced for the WAN link).*

---

## Journey 4: Container Network Namespace to Host Bridge

This scenario traces how packets move between two container network namespaces (Container A and Container B) located on the same physical host, connected via a virtual bridge (`docker0`).

### 1. Conceptual Flow & Path Diagram

```text
 [ Container A (netns_a) ]                 [ Container B (netns_b) ]
   IP:  172.17.0.2                           IP:  172.17.0.3
   MAC: 02:42:ac:11:00:02                    MAC: 02:42:ac:11:00:03
         │ (eth0)                                  ▲ (eth0)
         ▼ (veth_container)                        │ (veth_container2)
   ┌─────┴─────────────┐                     ┌─────┴─────────────┐
   │ Virtual Wire Pipe │                     │ Virtual Wire Pipe │
   └─────┬─────────────┘                     └─────┬─────────────┘
         │ (veth_host)                             ▲ (veth_host2)
         ▼                                         │
 ┌─────────────────────────────────────────────────┴─────────────────┐
 │                       Host Bridge (docker0)                       │
 │                                                                   │
 │ FDB table matches MAC ...00:03 to port veth_host2                 │
 └───────────────────────────────────────────────────────────────────┘
```

---

### 2. Step-by-Step Trace

#### Step 1: Container A Egress Check
1.  An application in Container A (`netns_a`) sends a packet to Container B's IP (`172.17.0.3`).
2.  Container A's network stack performs a routing table lookup. It matches the subnet local route:
    `172.17.0.0/16 dev eth0 proto kernel scope link src 172.17.0.2`
3.  The stack queries Container A's ARP cache to resolve `172.17.0.3` to its MAC address: **`02:42:ac:11:00:03`**. (If missing, it sends an ARP Request. The host bridge floods this request, and Container B responds).
4.  Container A builds the L2 frame with Container B's destination MAC, and sends it out of its `eth0` interface (which is the container-side endpoint of the virtual Ethernet pair `veth_container`).

#### Step 2: Namespace Boundary Cross (Veth Tunnel)
1.  The `eth0` interface driver calls **`veth_xmit()`**.
2.  Rather than pushing to hardware, `veth_xmit()` identifies the peer interface **`veth_host`** in the host network namespace.
3.  It updates the socket buffer metadata in software:
    *   **Device Reference**: `skb->dev = peer_dev;` (setting it to `veth_host`).
    *   **Namespace Reference**: Shifts the namespace tag from `netns_a` to the default host namespace.
4.  It calls **`netif_rx()`** to enqueue the packet on the host CPU's incoming backlog, causing the packet to transition seamlessly from Container A's namespace to the host namespace.

#### Step 3: Host Bridge Processing
1.  The host's receive loop (`__netif_receive_skb_core()`) retrieves the packet from the backlog.
2.  The stack detects that `veth_host` is enrolled as a port under the virtual bridge interface `docker0`.
3.  The core stack diverts execution to the bridge receive handler: **`br_handle_frame()`**.
4.  **MAC Learning**: The bridge inspects the packet's source MAC (`02:42:ac:11:00:02`) and records it in its Forwarding Database (FDB), mapping it to port `veth_host`.
5.  **FDB Lookup**: The bridge queries the FDB for the destination MAC address (`02:42:ac:11:00:03`).
6.  The lookup returns a match: the MAC is registered on port **`veth_host2`** (the host-side endpoint of Container B's virtual pair).
7.  The bridge calls **`br_forward()`**, routing the socket buffer to `veth_host2`'s transmission queue.

#### Step 4: Entering Container B Namespace
1.  Transmission on `veth_host2` triggers `veth_xmit()`.
2.  `veth_xmit()` identifies its peer interface `eth0` (which resides inside Container B's namespace `netns_b`).
3.  It updates the `skb` metadata in memory:
    *   `skb->dev = peer_dev;` (re-assigning it to `eth0` in `netns_b`).
    *   Sets the namespace tag to `netns_b`.
4.  It calls **`netif_rx()`** to drop the packet onto the CPU receive queue inside Container B's namespace.
5.  Container B's stack processes the frame, matches the destination IP `172.17.0.3`, and delivers the payload to the waiting application.

---

### 3. Header Transformations

```text
+---------------------------+---------------------------+-------------------+
| Ethernet / L2 Header      | IP Header                 | TCP Header        |
| Dest: 02:42:ac:11:00:03   | Src: 172.17.0.2           | Src Port: 59000   |
| Src:  02:42:ac:11:00:02   | Dest: 172.17.0.3          | Dest Port: 80     |
| EtherType: 0x0800         |                           |                   |
+---------------------------+---------------------------+-------------------+
```
*(Unlike physical network transitions, the Layer 2 Ethernet header remains completely unmodified from the source container to the destination container, because the bridge acts strictly as a Layer 2 forwarder).*

---

## Journey 5: Overlay Networking (VxLAN Cross-Host Tunneling)

This scenario documents how containers located on different physical hosts communicate across an overlay network using **VxLAN (Virtual Extensible LAN)** encapsulation.

### 1. Conceptual Flow & Path Diagram

```text
[ Container A (10.0.0.2) ]                     [ Container B (10.0.0.3) ]
  MAC: 02:42:0a:00:00:02                         MAC: 02:42:0a:00:00:03
          │                                              ▲
   (Inner Frame)                                   (Inner Frame)
          ▼                                              │
┌───────────────────────────┐                  ┌───────────────────────────┐
│     Host 1 (VTEP 1)       │                  │     Host 2 (VTEP 2)       │
│  IP: 192.168.50.10        │                  │  IP: 192.168.50.20        │
│                           │                  │                           │
│  Encapsulates inner frame │                  │  Decapsulates and strips  │
│  with Outer IP/UDP headers│                  │  outer encapsulation      │
└─────────────┬─────────────┘                  └─────────────▲─────────────┘
              │                                              │
              │         ┌─────────────────────────┐          │
              └────────►│ Physical Underlay Net   │──────────┘
                        │ (Standard Routers/LAN)  │
                        └─────────────────────────┘
```

---

### 2. Step-by-Step Trace

#### Step 1: Egress from Container A
1.  Container A (`10.0.0.2` on Host 1) transmits a packet to Container B (`10.0.0.3` on Host 2).
2.  The packet exits Container A's namespace and reaches the host's virtual bridge (`br0`).
3.  **Inner Packet State**:
    *   Source IP: `10.0.0.2` | Destination IP: `10.0.0.3`
    *   Source MAC: `02:42:0a:00:00:02` | Destination MAC: `02:42:0a:00:00:03`

#### Step 2: Bridge Forwarding to VTEP
1.  The host bridge (`br0`) receives the frame and queries its FDB for Container B's MAC address (`02:42:0a:00:00:03`).
2.  The lookup redirects the frame to the virtual VxLAN interface **`vxlan100`** (which acts as the Virtual Tunnel Endpoint, or **VTEP**).

#### Step 3: VTEP Encapsulation (Host 1)
The `vxlan100` interface driver captures the inner Ethernet frame and encapsulates it:
1.  **VTEP FDB Lookup**: The driver queries its own FDB table to find which physical host owns Container B's MAC. It resolves Host 2's physical IP address: **`192.168.50.20`**.
2.  **VxLAN Header Construction**: The driver prepends a **VxLAN Header** (8 bytes) containing the VxLAN Network Identifier flag: **`VNI = 100`**.
3.  **Outer UDP Header Construction**: It prepends a **UDP Header** (8 bytes) with:
    *   **Destination Port**: `4789` (The RFC 7348 standard port for VxLAN).
    *   **Source Port**: A flow-based hash computed from the inner packet headers (e.g. `53490`). Using a hash allows underlay routers to distribute VxLAN packets across multiple physical links using ECMP (Equal-Cost Multi-Path) routing.
4.  **Outer IP Header Construction**: It prepends an **IP Header** (20 bytes) with:
    *   Source IP: Host 1 physical IP (`192.168.50.10`).
    *   Destination IP: Host 2 physical IP (`192.168.50.20`).
5.  **Outer Ethernet Header Construction**: It prepends a standard Ethernet header:
    *   Source MAC: Host 1 physical NIC MAC (`00:11:22:aa:bb:01`).
    *   Destination MAC: Next-hop underlay gateway MAC (or Host 2 physical NIC MAC if on the same subnet).

#### Step 4: Underlay WAN/LAN Transmission
1.  The resulting encapsulated packet is treated by the Host 1 stack as a standard UDP packet.
2.  It traverses physical switches and routers on the underlay network. Intermediate devices inspect **only** the outer headers; they do not know the packet contains an inner container frame.

#### Step 5: VTEP Decapsulation (Host 2)
1.  Host 2 receives the physical frame on its NIC.
2.  **UDP Dispatch**: The network stack decapsulates the Layer 2/3 headers, processes the UDP packet, and notices the destination port is `4789`.
3.  The stack routes the packet payload to the registered VxLAN kernel module driver.
4.  **Decapsulation**: The VxLAN driver reads the VNI (`100`), validates it, and strips the outer headers (Outer Ethernet, Outer IP, Outer UDP, and the VxLAN control header).
5.  **Inner Packet Forwarding**: The driver extracts the original **inner Ethernet frame** and injects it into Host 2's virtual bridge (`br0`).
6.  The bridge reads the destination MAC (`02:42:0a:00:00:03`) and delivers the frame to Container B.

---

### 3. Header Transformations

```text
+----------------------+--------------------+--------------------+---------------+--------------------+------------------+-------------------+
| Outer Ethernet       | Outer IP           | Outer UDP          | VxLAN Header  | Inner Ethernet     | Inner IP         | Inner TCP         |
| Dest: Host 2 MAC     | Src:  192.168.50.10| Src: Flow-hash     | Flags: VNI 100| Dest: Container B  | Src:  10.0.0.2   | Src Port: 52110   |
| Src:  Host 1 MAC     | Dest: 192.168.50.20| Dest: 4789 (VxLAN) |               | Src:  Container A  | Dest: 10.0.0.3   | Dest Port: 80     |
| Type: 0x0800 (IPv4)  | Proto: 17 (UDP)    |                    |               | Type: 0x0800 (IPv4)|                  |                   |
+----------------------+--------------------+--------------------+---------------+--------------------+------------------+-------------------+
|<----------------------------------- Outer Envelope -------------------------->|<------------------ Inner Payload -------------------->|
```
*(The outer envelope is stripped by the destination host VTEP, delivering the inner payload intact to the target namespace).*

---

## Journey 6: Load Balancer / NAT (DNAT & SNAT Flow)

This scenario documents how a client connection to a public load balancer's Virtual IP (VIP) is dynamically translated via Destination NAT (DNAT) to a backend real server IP, and how the reverse translation handles the return path.

### 1. Conceptual Flow & Path Diagram

```text
 [ Client ]                                     [ Backend Server ]
 203.0.113.50                                   192.168.10.100
      │                                                ▲
  (Req to VIP)                                   (DNATed Req)
      ▼                                                │
┌──────────────────────────────────────────────────────┴───────────────────┐
│                      Load Balancer Gateway (VIP)                        │
│                      VIP: 198.51.100.10                                  │
│                                                                          │
│  - DNAT: Rewrites Dest VIP to Real IP 192.168.10.100                     │
│  - Tracks mapping in conntrack table                                     │
│  - Reverse NAT: Rewrites Source IP back to VIP for replies               │
└──────────────────────────────────────────────────────────────────────────┘
```

---

### 2. Step-by-Step Trace

#### Step 1: Outbound Request (Client to Load Balancer VIP)
1.  A client (`203.0.113.50`) connects to a public web service hosted on a Load Balancer's Virtual IP (VIP): **`198.51.100.10:443`**.
2.  **Outgoing Packet State**:
    *   Source IP: `203.0.113.50` | Destination IP: `198.51.100.10`
    *   Source Port: `54321` | Destination Port: `443`

#### Step 2: Load Balancer DNAT Rewrite
1.  The Load Balancer receives the packet on its public-facing interface.
2.  The kernel Netfilter/IPVS engine intercepts the packet at the `NF_INET_PRE_ROUTING` hook.
3.  The Load Balancer selects a healthy backend server using a configured algorithm (e.g. Round Robin), choosing Server A: **`192.168.10.100`** listening on port **`8080`**.
4.  **Destination NAT (DNAT)**: The LB rewrites the destination fields:
    *   Destination IP: `198.51.100.10 -> 192.168.10.100` (Real IP)
    *   Destination Port: `443 -> 8080` (Backend Port)
5.  **State Insertion**: The LB updates its connection tracking state table, registering the bidirectional mapping:
    `[Src: 203.0.113.50:54321, Dest: 198.51.100.10:443] <=> [Src: 203.0.113.50:54321, Dest: 192.168.10.100:8080]`
6.  The LB routes and transmits the rewritten packet out of its private interface to the backend local network.

#### Step 3: Backend Server Processing
1.  The backend server (`192.168.10.100`) receives the packet. Since the destination IP matches its network card interface, it passes the packet to the application listening on port `8080`.
2.  The application processes the request and sends a reply.
3.  **Reply Packet State**:
    *   Source IP: `192.168.10.100` (Backend Real IP) | Destination IP: `203.0.113.50` (Client IP)
    *   Source Port: `8080` | Destination Port: `54321`
4.  The backend's routing table points to the Load Balancer's private IP as its default gateway. The packet is sent back to the LB.

#### Step 4: Reverse NAT translation (Load Balancer to Client)
1.  The Load Balancer receives the backend server's reply on its private interface.
2.  The stack intercepts the packet at `NF_INET_PRE_ROUTING`.
3.  **State Match**: The tracking engine searches its state table for the tuple:
    `Src: 192.168.10.100:8080, Dest: 203.0.113.50:54321`
4.  **Reverse Translation**: Finding a match, the LB rewrites the source fields:
    *   Source IP: `192.168.10.100 -> 198.51.100.10` (VIP)
    *   Source Port: `8080 -> 443` (VIP Port)
5.  The LB routes the packet over the public interface, transmitting it back to the client across the WAN.

#### Step 5: Client Ingress
1.  The client receives the reply packet.
2.  **Incoming Packet State**:
    *   Source IP: `198.51.100.10` (VIP) | Destination IP: `203.0.113.50` (Client IP)
    *   Source Port: `443` | Destination Port: `54321`
3.  Because the source matches the exact socket connection the client initiated in Step 1, the client's TCP stack accepts the packet.
4.  *(Note: If the Load Balancer failed to perform reverse NAT and returned the packet with source `192.168.10.100`, the client stack would look up its socket table, find no connection to `192.168.10.100:8080`, and reject the packet with an unsolicited TCP `RST`).*

---

### 3. Header Transformations

#### Phase 1: Client to Load Balancer VIP
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header           | IP Header                 | TCP Header        |
| Dest: LB Public MAC       | Src: 203.0.113.50         | Src Port: 54321   |
| Src:  Client MAC/Router   | Dest: 198.51.100.10 (VIP) | Dest Port: 443    |
+---------------------------+---------------------------+-------------------+
```

#### Phase 2: Load Balancer to Backend (After DNAT)
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header (LAN Link)| IP Header (LAN Link)      | TCP Header        |
| Dest: Backend MAC         | Src: 203.0.113.50         | Src Port: 54321   |
| Src:  LB Private MAC      | Dest: 192.168.10.100      | Dest Port: 8080   |
+---------------------------+---------------------------+-------------------+
```

#### Phase 3: Backend Reply to Load Balancer
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header (LAN Link)| IP Header (LAN Link)      | TCP Header        |
| Dest: LB Private MAC      | Src: 192.168.10.100       | Src Port: 8080    |
| Src:  Backend MAC         | Dest: 203.0.113.50        | Dest Port: 54321  |
+---------------------------+---------------------------+-------------------+
```

#### Phase 4: Load Balancer to Client (After Reverse NAT)
```text
+---------------------------+---------------------------+-------------------+
| Ethernet Header (WAN Link)| IP Header (WAN Link)      | TCP Header        |
| Dest: Client/Gateway MAC  | Src: 198.51.100.10 (VIP)  | Src Port: 443     |
| Src:  LB Public MAC       | Dest: 203.0.113.50        | Dest Port: 54321  |
+---------------------------+---------------------------+-------------------+
```
