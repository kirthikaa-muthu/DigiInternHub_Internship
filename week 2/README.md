# Windows and Network Security Lab

## Overview
This repository documents practical exercises and study notes covering network reconnaissance, traffic analysis, Windows security fundamentals, Windows event-log investigation, and PowerShell-based security hunting. The lab was performed in a controlled virtual-machine environment for coursework and learning.

## Learning Objectives
- Understand TCP connections and common network protocols.
- Distinguish Wireshark capture filters from display filters.
- Use Nmap for authorised host discovery and port scanning.
- Investigate selected Windows security events in Event Viewer.
- Export failed-logon events to CSV with PowerShell and summarise them by account.
- Understand the value of event correlation and security baselines.

## Topics Covered

### TCP and Network Protocols
- TCP three-way handshake: `SYN`, `SYN-ACK`, `ACK`.
- Common TCP flags: `SYN`, `ACK`, `FIN`, `RST`, `PSH`, and `URG`.
- Protocols reviewed: DNS, HTTP/HTTPS, ARP, ICMP, and SMB.
- A single unusual packet or event does not automatically prove malicious activity; interpret evidence in context.

### Wireshark
Examples of display filters reviewed:

```text
dns
http
tcp
udp
icmp
arp
tcp.port == 443
ip.addr == 192.168.1.10
tcp.flags.syn == 1
tcp.flags.reset == 1
dns.qry.name == "example.com"
http.request.method == "POST"
http.response.code == 404
tls.handshake.type == 1
ip.src == 192.168.1.10
ip.dst == 192.168.1.1
tcp.flags.fin == 1
tcp.flags.ack == 1
udp.port == 53
frame.len > 1000
```

Capture filters are applied while capturing traffic. Display filters are applied to packets already captured; they hide non-matching packets without deleting them from the capture.

### Nmap
| Option | Purpose |
| --- | --- |
| `-sn` | Host discovery without a port scan |
| `-sS` | TCP SYN scan |
| `-sT` | TCP connect scan |
| `-sU` | UDP scan |
| `-sV` | Service/version detection |
| `-O` | OS detection |
| `-A` | Enables OS detection, version detection, default scripts, and traceroute |

Only scan systems and networks for which you have explicit permission.

### Windows Fundamentals
- **Processes:** programs currently running.
- **Services:** background components that can run without an interactive user.
- **Registry:** hierarchical database of Windows and application settings.
- **Users and groups:** identities and permission assignments.
- **User Account Control (UAC):** helps limit unauthorised administrative changes.
- **Event Viewer:** used to inspect Application, Security, and System logs.

Useful tools include `eventvwr.msc`, `services.msc`, and `regedit`. Use Registry Editor carefully and make changes only in a lab when instructed.

## Practical Task 3: Generate and Investigate Windows Events

| Event ID | Meaning | Lab evidence |
| --- | --- | --- |
| `4625` | Failed logon | Failed-logon event in the Security log |
| `4624` | Successful logon | Successful logon; observed example used logon type 5 (Service) |
| `4720` | User account created | Test account `LabTest` |
| `4732` | Member added to a security-enabled local group | `LabTest` added to the local Administrators group |
| `7045` | A service was installed | Test service `LabDummyService` |
| `1102` | Security audit log was cleared | Audit-log-cleared event in the lab |

**Important:** Event IDs need context. Event `4624` is not necessarily an interactive desktop logon; inspect the logon type and other fields. Event `1102` should be generated only in an authorised lab because clearing the Security log removes prior audit records. The test service was deleted after the exercise.

Suggested screenshot filenames (use only for screenshots you actually captured):

```text
evidence/Task3_Event4625_FailedLogon.png
evidence/Task3_Event4624_SuccessfulLogon.png
evidence/Task3_Event4624_LogonType.png
evidence/Task3_Event4720_NewUser.png
evidence/Task3_Event4732_AdminGroup.png
evidence/Task3_Event7045_ServiceInstall.png
evidence/Task3_Event1102_SecurityLogCleared.png
```

## Practical Task 4: PowerShell Hunting

### Export failed logons to CSV
Run PowerShell as an account with permission to read the Security log. This script creates `C:\Lab\FailedLogons.csv` and extracts useful fields from Event ID `4625`.

```powershell
New-Item -Path C:\Lab -ItemType Directory -Force

$events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id      = 4625
}

$report = foreach ($event in $events) {
    $xml = [xml]$event.ToXml()
    $data = @{}

    foreach ($item in $xml.Event.EventData.Data) {
        $data[$item.Name] = $item.'#text'
    }

    [PSCustomObject]@{
        TimeCreated      = $event.TimeCreated
        EventID          = $event.Id
        TargetUserName   = $data['TargetUserName']
        TargetDomainName = $data['TargetDomainName']
        LogonType        = $data['LogonType']
        IpAddress        = $data['IpAddress']
    }
}

$report | Export-Csv -Path C:\Lab\FailedLogons.csv -NoTypeInformation
```

### Count failures by account

```powershell
Import-Csv C:\Lab\FailedLogons.csv |
    Group-Object -Property TargetUserName |
    Select-Object Name, Count |
    Sort-Object Count -Descending |
    Format-Table -AutoSize
```

### Observed result
The lab CSV contained **3 failed-logon events** for the account `Kirthikaa`. The grouping command returned:

```text
Name       Count
----       -----
Kirthikaa      3
```

The events were recorded with logon type `2` (Interactive). Observed IP values included `127.0.0.1` and `-`; interpret these fields in the context of the event details and the local lab configuration.

### Output file
- `C:\Lab\FailedLogons.csv` — exported failed-logon events.

If the CSV is required on the host computer, copy it from the Windows VM using a configured shared folder, shared clipboard/drag-and-drop if working, or another approved transfer method.

## Security Event ID Quick Reference

| Event ID | Description |
| --- | --- |
| `4624` | Successful logon |
| `4625` | Failed logon |
| `4634` | Account logged off |
| `4648` | Logon attempted using explicit credentials |
| `4672` | Special privileges assigned to a new logon |
| `4688` | New process created |
| `4697` | Service installed |
| `4720` | User account created |
| `4722` | User account enabled |
| `4724` | Attempt to reset an account password |
| `4732` | Member added to a security-enabled local group |
| `4768` | Kerberos ticket-granting ticket (TGT) requested |
| `4769` | Kerberos service ticket requested |
| `4776` | NTLM credentials validated |
| `1102` | Security audit log cleared |

Whether an event is recorded depends on Windows version, audit policy, permissions, and system configuration. Correlate multiple events rather than relying on one ID in isolation.

## Security Baselines
A security baseline describes expected system configuration, user activity, and network behaviour. Comparing observed activity with a baseline can help identify unexpected logons, unusual processes, configuration changes, and suspicious network patterns. Keep baselines current as legitimate systems and workflows change.

## Suggested Repository Structure

```text
windows-network-security-lab/
├── README.md
├── Windows_Network_Security_Study_Report.pdf
├── FailedLogons.csv
└── evidence/
    ├── Task3_Event4625_FailedLogon.png
    ├── Task3_Event4624_SuccessfulLogon.png
    └── ...
```

Only include files you actually have. Do not commit passwords, VM credentials, personal data, or unrelated sensitive logs.

## Disclaimer
This material is for educational purposes and authorised lab use only. Perform scans and security tests only on systems and networks you own or have explicit permission to assess.
