# network-labs

Hands-on networking and network security labs, built in my own home lab. Each lab produces packet captures, a short write-up, and (where it fits) a small script.

I'm a self-directed cybersecurity learner working toward a career in security engineering and ethical hacking. This repo is where I turn theory into evidence: what I expected, what actually happened on the wire, and how I'd detect or fix it.

## Repo layout

```
network-labs/
├── captures/   # sanitized .pcap/.pcapng files from each lab
├── writeups/   # one short write-up per lab
└── README.md
```

## Labs

| Day | Lab | Status |
|-----|-----|--------|
| 1 | TCP on loopback: serve a page with `python3 -m http.server 8000`, capture it in Wireshark, annotate the handshake, data transfer and teardown | Done |
| 2 | Capture and document normal ARP, DHCP and DNS traffic | Planned |
| 3 | Nmap scan types (SYN, connect, UDP, version, OS) and what each looks like in a capture | Planned |
| 4-6 | Malware-traffic capture analysis, ARP trust between lab VMs, Suricata rules | Planned |
| 8-12 | Segmented mini network (WAN/LAN/DMZ) with a firewall VM, Suricata monitoring, and a Python alert summarizer | Planned |

## Write-up format

Every write-up answers five questions in 250 to 500 words:

1. What did I expect?
2. What happened?
3. Why?
4. How would I detect or fix it?
5. How do I reproduce it?

## Tools

- Wireshark / tshark
- Python 3
- Nmap
- Suricata
- KVM/libvirt virtual machines
- pfSense or OPNsense (firewall VM)

## Scope and safety

- All labs run on my own machines and virtual networks only.
- Captures are sanitized before publishing. Real-network captures that contain MAC addresses or LAN IPs stay local.
- Nothing here is aimed at systems I don't own or have permission to test.

## Notes

This is a learning log, so expect small mistakes and corrections in the commit history. Feedback is welcome through issues.

## Author

r3tro-n3o
