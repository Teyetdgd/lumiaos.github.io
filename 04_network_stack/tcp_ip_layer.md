# The TCP/IP Network Stack

LumiaOS does not rely on third-party networking libraries (like `lwIP` or `BSD sockets`). The entire networking stack, from Ethernet frame construction up to HTTP and DNS, was written from scratch in C++.

The stack follows the OSI model, with each layer processing incoming packets and stripping its specific headers before passing the payload up to the next layer.

## 1. Link Layer (Ethernet & ARP)

Located in `src/net/ethernet.cpp` and `src/net/arp.cpp`.

- **Ethernet:** The `ethernet_send_packet()` function constructs standard IEEE 802.3 frames, adding the Destination MAC, Source MAC, and EtherType before pushing it to the active NIC driver.
- **ARP (Address Resolution Protocol):** Since IP packets need MAC addresses to navigate the local subnet, the ARP module maintains an internal cache. If an IP is not in the cache, `arp_send_request()` broadcasts an ARP packet and halts the packet transmission until the MAC address is resolved.

## 2. Internet Layer (IPv4 & ICMP)

Located in `src/net/ip.cpp` and `src/net/icmp.cpp`.

- **IPv4:** The `ip_send_packet()` function handles IP header construction (TTL, Protocol, Checksum). It automatically calculates the correct Header Checksum using the standard one's complement sum algorithm.
- **ICMP:** Implements basic Ping functionality. The system responds to `ECHO_REQUEST` packets and can send `ECHO_REQUEST`s to diagnose network reachability.

## 3. Transport Layer (UDP & TCP)

### UDP (User Datagram Protocol)
Implemented in `src/net/udp.cpp`. UDP is a stateless, connectionless protocol used primarily in LumiaOS for DNS queries. It supports ephemeral port allocation and simple `udp_send` / `udp_socket_recv` bindings.

### TCP (Transmission Control Protocol)
Implemented in `src/net/tcp.cpp`. This is the most complex part of the networking stack, essential for the Web Browser.
- **State Machine:** Implements the core TCP states: `SYN_SENT`, `ESTABLISHED`, `FIN_WAIT`, and `CLOSED`.
- **Three-Way Handshake:** `tcp_connect()` manages the SYN -> SYN-ACK -> ACK sequence.
- **Sequence Tracking:** Maintains strict tracking of `seq_num` and `ack_num` to ensure packets are assembled in order.
- **Retransmission:** (In development) Basic retry logic for dropped segments.

## 4. Application Layer (DNS & HTTP)

- **DNS:** Implements a UDP-based DNS client (`src/net/dns_real.cpp`) capable of parsing `A` records to resolve domain names to IP addresses. It includes a caching mechanism to speed up subsequent requests.
- **HTTP:** A custom HTTP/1.1 client (`src/net/http_enhanced.cpp`). It supports chunked transfer encoding (`Transfer-Encoding: chunked`), parsing headers, and managing persistent connections for the Lumia Browser.
