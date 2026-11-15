# Week 1 — VirtualBox Cybersecurity Lab

## Overview
This repository documents Week 1 of a hands-on cybersecurity lab. The goal was to build a small virtual network, verify communication between virtual machines, and practise basic Linux/network troubleshooting commands.

## Lab Environment

- **Virtualisation platform:** Oracle VirtualBox
- **Network type:** NAT Network
- **Network name:** `cyberlab`
- **Subnet:** `192.168.50.0/24`
- **Default gateway configured in the lab:** `192.168.50.1`

### Virtual machines

| Virtual machine | IP address | Purpose |
|---|---:|---|
| Kali Linux | `192.168.50.4` | Security testing workstation |
| Ubuntu Linux | `192.168.50.5` | Linux host for administration and testing |
| Windows 2000 Professional | `192.168.50.3` | Legacy Windows host for isolated lab practice |

## Network Topology

The three virtual machines are connected to the same VirtualBox NAT Network, `cyberlab`.

![Week 1 network topology](network-diagram.png)

> If the image is stored with a different filename, update the image path above. The diagram image supplied with the Week 1 report can be renamed to `network-diagram.png` and placed in the repository root.

## Work Completed

### 1. Virtual network configuration
- Created and started the `cyberlab` NAT Network using the `192.168.50.0/24` subnet.
- Connected Kali Linux, Ubuntu Linux, and Windows 2000 Professional to the shared network.
- Checked each machine's network configuration using `ip addr` on Linux and `ipconfig` on Windows.

### 2. Connectivity testing
Successful ping tests were recorded between the three virtual machines:

| Source | Destination | Result |
|---|---|---|
| Kali Linux (`192.168.50.4`) | Ubuntu (`192.168.50.5`) | Successful — 0% packet loss |
| Kali Linux (`192.168.50.4`) | Windows 2000 (`192.168.50.3`) | Successful — 0% packet loss |
| Ubuntu (`192.168.50.5`) | Kali Linux (`192.168.50.4`) | Successful — 0% packet loss |
| Ubuntu (`192.168.50.5`) | Windows 2000 (`192.168.50.3`) | Successful — 0% packet loss |
| Windows 2000 (`192.168.50.3`) | Kali Linux (`192.168.50.4`) | Successful — 0% packet loss |
| Windows 2000 (`192.168.50.3`) | Ubuntu (`192.168.50.5`) | Successful — 0% packet loss |

### 3. Network diagnostic commands
Commands practised and/or captured in the lab included:

- `ip addr` — displays Linux network interfaces and IP addresses.
- `ipconfig` — displays Windows IP configuration.
- `ping` — tests reachability between hosts.
- `traceroute` / `tracert` — investigates the route to a destination.
- `nslookup google.com` — tests DNS name resolution.

DNS lookup returned IP addresses for `google.com` on the Linux hosts. Traceroute results varied by host and probe; one Ubuntu-to-Kali trace reached the destination in one hop, while some other probes timed out.

### 4. Linux command-line practice
Basic Kali/Linux commands were also practised for files, permissions, users/processes, network information, and command history. Example tasks included changing file permissions, listing file details, checking the current user and identity, viewing processes, checking neighbour entries, and locating tools.

## Key Outcomes
- All three virtual machines were assigned distinct addresses in the same `/24` subnet.
- Bidirectional communication between the machines was verified with ping tests.
- Basic IP configuration and diagnostic commands were used to collect evidence.
- The lab provides a foundation for later networking and cybersecurity exercises.

## Safety Notes
- Windows 2000 is unsupported and should remain isolated in this lab environment.
- Only test systems and networks you own or have explicit permission to assess.
- Keep screenshots and logs free of passwords, tokens, and other sensitive information.

## Suggested Repository Structure

```text
week1-cybersecurity-lab/
├── README.md
├── network-diagram.png
├── Week1_Cybersecurity_Lab_Report.pdf
└── Week1_Cybersecurity_Lab_Presentation.pptx
```

## Evidence
Screenshots used during the lab included IP configuration from each VM, ping tests between VMs, and traceroute/DNS lookup output. Add your original screenshots to an `evidence/` folder if you want the repository to include the full evidence set.
