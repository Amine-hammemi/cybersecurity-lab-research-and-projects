---
tags:
  - cybersecurity/reference
  - networking
  - wireshark
  - command-reference
status: maintained
---

# Wireshark Display Filters

These are **Wireshark display filters and packet fields**, not shell commands. I keep them separate so I do not confuse a filter entered in Wireshark's display-filter bar with a command run in a terminal.

## Filters recorded in my ARP and ICMP lab

| Filter / field | What it selects or represents | General use and useful options | Where I used it |
|---|---|---|---|
| `arp` | Displays ARP packets. | Use to isolate address-resolution traffic on an Ethernet segment. | [Wireshark — ARP capture experiments](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) · [Wireshark — display-filter summary](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `icmp` | Displays ICMP packets. | Useful when analysing Echo Requests/Replies and other ICMP messages. | [Wireshark — ICMP and ping](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#icmp-and-ping) |
| `arp \|\| icmp` | Displays packets matching either ARP or ICMP. | `\|\|` is the logical OR operator in Wireshark display filters. Use `&&` for logical AND and `!` for negation. | [Wireshark — ARP and ICMP combined filter](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#experiments) · [Wireshark — repeatable capture filter](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `arp.opcode == 1` | Displays ARP requests (opcode 1). | ARP reply opcode is 2. Check packet details to understand sender/target protocol and hardware addresses. | [Wireshark — distinguish ARP requests](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |

## ARP fields inspected in packet details

| Field | Meaning | General use | Where I used it |
|---|---|---|---|
| `arp.src.proto_ipv4` | Sender protocol address (IPv4) in the ARP message. | Useful for identifying the IPv4 address the sender is announcing or using. | [Wireshark — ARP field inspection](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `arp.dst.proto_ipv4` | Target protocol address (IPv4) in the ARP message. | Useful for identifying the IPv4 address being queried or referenced as the target. | [Wireshark — ARP field inspection](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |
| `arp.opcode` | ARP operation code. | Common values include 1 for request and 2 for reply. It can be used as a display-filter field, e.g. `arp.opcode == 1`. | [Wireshark — ARP opcode analysis](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md#command-summary) |

## Important distinction

A packet filter controls which captured packets are displayed; it does not change network traffic. A missing ARP reply alone does not prove that a host is powered off. Consider interface state, VLAN/bridge placement, filtering, packet loss and capture location before drawing a conclusion.

## Related notes

- [Wireshark Module 1](../Cybersecurity/05-Network%20Analysis/Wireshark/Module%201.md)
- [02 - Networking Commands](02%20-%20Networking%20Commands.md)
