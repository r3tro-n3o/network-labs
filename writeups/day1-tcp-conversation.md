---
# day1-tcp-conversation.md   
# Lab Write-up: Day 1 TCP Session Dissection   
**Target File:** day1-handshake-filtered.pcapng   
**Environment:** Localhost Loopback (127.0.0.1)   
**Stream Index:** 20   
 --- 
## 1. What did I expect?   
I expected to observe a textbook TCP session lifecycle executing a standard HTTP web transaction. The theoretical model predicts a precise sequence of control flags and data transfers:   
1. **Three-Way Handshake:** `SYN` ➔ `SYN/ACK` ➔ `ACK`   
2. **Application Data Exchange:** `HTTP Request` ➔ `HTTP Response`   
3. **Four-Step Connection Teardown:** `FIN` ➔ `ACK` ➔ `FIN` ➔ `ACK` (or an optimized variant combining the client's confirmation and closing signals).   
 --- 
   
## 2. What happened?   
The actual packet capture (Packets 117–128) tracked an active local HTTP session operating over the localhost loopback interface (`127.0.0.1`) targeting destination port `8000`. The session progressed as follows:   
- **The Connection Establishment (Packets 117–119):** A successful three-way handshake was executed. The loopback endpoints advertised an MSS of 65,495 bytes, which is substantially larger than the 1460-byte MSS commonly observed with a 1500-byte Ethernet MTU. This reflects the different characteristics of the loopback interface and should be treated as an observed value rather than a universal loopback value. Additional advanced TCP features were established, includin**g Window Scali**ng (factor of 10)**, SACK Permitt**ed, an**d Timestam**ps.   
- **The Application Layer Transaction (Packets 120–125):** The client issued an outbound`  HTTP  `GET/ web request. The server acknowledged the request and delivered its response. Notably, rather than a single atomic transfer, the server's HTTP response layout was split across two distinct TCP data segments**: a 155-byte s**egment followed** **by** a 256-byte **segment. Wireshark successfully performed** TCP Reassembly **to present the full 411-byte HTTP payload to the application layer.   
- **The Connection Teardown (Packets 126–128):** The conversation concluded with a clean `FIN/ACK` teardown sequence. The client optimized the process by combining its acknowledgment of the server's close and its own termination request into a single packet.   
 --- 
   
## 3. Why?   
The packet values and sequence numbers observed are governed by TCP's sequencing, acknowledgment, flow-control, and connection-management mechanisms.   
### Handshake & Request Tracking:   
- **SYN Phase:** The client starts at relative `Seq 0`. Because a `SYN` flag consumes exactly one logical sequence number, the client's subsequent handshake `ACK` and its `HTTP Request` begin at relative `Seq 1`.   
- **Request Payload:** The `HTTP Request` carries a payload length of `78 bytes` (`Seq 1` to `Seq 78`). Wireshark calculates the client's `Next Seq = 79`.   
- **Server Acknowledgment:** The server acknowledges this data stream completely by returning `Ack 79`.   
   
### Server Response & Teardown Arithmetic:   
- **First Response Segment:** The server responds at relative `Seq 1` with a `155-byte` segment, advancing its stream to `Next Seq = 156`.   
- **Second Response Segment:** The server continues transmission from `Seq 156` with a `256-byte` HTML segment. The math handles the continuity seamlessly:
\[\text{156 (Start)} + \text{256 (Payload Length)} = \text{412}\]   
- **Client Validation:** The client balances the ledger by returning a pure acknowledgment holding `**Ack 412**`.   
- **The Connection Shutdown:** The server initiates termination by sending a `FIN` packet at `Seq 412`. Because control flags like `SYN` and `FIN` consume exactly one sequence number, the client must increment this value during its receipt. The client replies with `**Ack 413**`, proving that the `FIN` flag accounted for the final sequence slot.   
   
### The PSH Flag Role:   
During the data segments, the `PSH` flag was set. This indicates that the sender is requesting that the receiving TCP stack deliver the data to the application promptly rather than waiting for additional incoming data to fill a traditional buffer.   
 --- 
## 4. How would I detect or fix it?   
As a security analyst, understanding this normal baseline allows me to detect anomalies across a production enterprise network:   
- **Detection :** A healthy TCP session can be isolated in Wireshark by using targeted display filters (such as `tcp.port == 8000`) and verifying the structural presence of sequential `SYN`, `SYN/ACK`, and `ACK` packets.   
- **Investigating Deviations:** While the localhost behavior captured here is entirely benign, discovering abnormal patterns, such as a flood of `SYN` attempts without completing an `ACK`(SYN Flooding), an excessive volume of `RST` packets, or high retransmission ratios, would indicate infrastructure connectivity issues, firewall filtering behavior, or active malicious scanning.   
 --- 
   
## 5. How would I reproduce it?   
This exact local baseline environment can be fully reproduced using standard native command-line utilities:   
1. **python3 -m http.server 8000**
   
2. **Initiate a packet capture** inside Wireshark, targeting the local loopback interface (`lo` or `Loopback`).   
3. **curl http://localhost:8000**   
4. **tcp.port == 8000
**   
