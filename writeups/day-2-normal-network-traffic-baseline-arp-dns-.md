
# Day 2: Normal-Network-Traffic-Baseline-(ARP, DNS, DHCP)   
## 1. What I Expected   
- **ARP:** Expected the operating system to magically translate IP addresses into MAC addresses instantly behind the scenes without needing explicit visibility.   
- **DNS:** Expected domain name resolution to be a simple, abstracted API call ( `example.com` $\rightarrow$ IP) rather than an explicit packet exchange over the wire.   
- **DHCP:** Expected IP address assignment to be a static configuration process or a background daemon detail that rarely interacts with active network frames during daily use.   
   
## 2. What Actually Happened   
- **ARP:** Flushing the neighbor table ( `sudo ip neigh flush dev wlan0`) and pinging the gateway forced an immediate Ethernet broadcast ( `ff:ff:ff:ff:ff:ff`) asking *"Who has 192.168.100.1? Tell 192.168.100.18,"* which was followed instantly by a unicast reply from the router containing its hardware MAC address.   
- **DNS:** Running `dig example.com` generated a clean, connectionless UDP transaction over destination port 53. Wireshark revealed matching transaction IDs ( `0x0444`), specific query types ( `A` records), returned IP addresses, and a short Time to Live (TTL) of 23 seconds dictating local caching behavior.   
- **DHCP:** Explored the mechanics of temporary IP leases and mapped out the **DORA** sequence (**D**iscover, **O**ffer, **Request**, **ACK**), highlighting how network addresses are rented rather than permanently owned.   
   
## 3. Why It Happened   
- **ARP Layer 2/3 Bridge:** Network layer traffic (IP) relies on data link layer delivery (Ethernet frames). Because Ethernet switches and NICs target physical hardware addresses rather than logical IP addresses, ARP acts as the necessary mapping bridge on the local broadcast domain.   
- **DNS over UDP:** The observed DNS query used UDP destination port 53, avoiding the connection setup associated with TCP. DNS can also use TCP when required, including certain larger or operationally specific exchanges.   
- **DHCP Lease Constraints:** IPv4 addresses are finite resources. Routers assign temporary leases using broadcast-driven DORA exchanges so that disconnected devices automatically relinquish their addresses back into the available pool.   
   
## 4. How I Would Detect or Fix It   
- **ARP Spoofing / Poisoning Detection:** Unexpected or anomalous ARP replies, particularly those that change the expected IP-to-MAC mapping, can indicate ARP spoofing or poisoning. This is detected and mitigated using static ARP entries, Dynamic ARP Inspection (DAI) on managed switches, or monitoring tools like` arpwatc`h.   
- **DNS Tunneling Detection:** Anomalous spikes in DNS query volume, unusually long subdomain strings, or heavy reliance on TXT records can indicate data exfiltration or Command and Control (C2) communication hidden inside normal-looking port 53 traffic.   
- **Rogue DHCP Server Detection:** Unauthorized DHCP servers on a network can hand out malicious default gateways to hijack client traffic. This is mitigated in enterprise environments by enabling **DHCP Snooping** on network switches.   
   
## 5. How Someone Could Reproduce It   
1. Open a terminal in your Kali Linux lab environment and verify your active interface using `ip addr` and `ip route` (e.g., `wlan0`).   
2. Start a live capture in Wireshark targeting your active interface.   
3. **For ARP:** Flush your neighbor cache ( `sudo ip neigh flush dev wlan0`) and ping your local gateway ( `ping -c 1 <gateway-ip>`), filtering Wireshark for `arp`.   
4. **For DNS:** Run `dig example.com` in your terminal and filter Wireshark for `dns`.   
5. **For DHCP:** Inspect your current lease status using `nmcli device show wlan0` and document the DORA exchange mechanics.   
6. Export your filtered packet captures to your `network-labs/captures/` directory.   
