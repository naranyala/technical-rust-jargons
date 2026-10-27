# Company Computer Networking and Data Management Catalog

A practical field guide for IT support teams, service desk engineers, and operations staff who need to understand how company networks work, how business data flows, and how to troubleshoot issues without guesswork.

This catalog is designed to be used in real environments: office LANs, remote workers, shared file systems, cloud backups, user support, ticket triage, and day-to-day systems administration.

---

## Why this matters

In a modern company, technology problems are usually one of three things:

- a connectivity issue
- a data access or storage issue
- a configuration or permissions issue

Support teams are often the first line of response. The faster someone can classify the problem, the faster they can restore service.

This catalog helps answer questions like:

- Why can a user access Wi‑Fi but not the file server?
- Why does a machine have internet but not the company app?
- Why did a backup fail after a drive replacement?
- Why is a printer offline after a network change?
- Why is a user locked out of shared data after a permission update?

---

## Quick map: the IT support view

### 1. Networking layer

This is the physical and logical path that devices use to talk to each other.

Topics include:

- IP addresses and subnets
- DNS and DHCP
- routers, switches, firewalls
- Wi‑Fi and wireless troubleshooting
- VPN and remote access
- VLANs and segmentation
- basic cabling and patching
- latency, jitter, and packet loss

### 2. Data management layer

This is how company information is stored, protected, shared, and recovered.

Topics include:

- file shares and permissions
- backups and restores
- storage arrays and NAS/SAN
- retention and archive policies
- versioning and snapshots
- cloud sync and collaboration storage
- data cleanup and lifecycle management

### 3. Support operations layer

This is how the help desk keeps the environment stable.

Topics include:

- incident triage
- change control
- asset inventory
- onboarding and offboarding
- endpoint imaging and deployment
- security basics
- user communication and escalation

---

## Core networking concepts for IT support

### IP address

An IP address is the unique network identifier used by a device to send and receive traffic.

Common examples:

- IPv4: 192.168.1.25
- IPv6: 2001:db8::10

IT support use:

- Check whether a device has a valid address
- Confirm whether it is on the expected subnet
- Detect DHCP failures or address conflicts

### Subnet mask

The subnet mask defines which part of the address is the network and which part is the host.

Example:

- 192.168.1.25 /24 means the network is 192.168.1.0 and the host portion is .25

Support relevance:

- Helps determine if two devices are on the same LAN
- Useful for troubleshooting connectivity issues
- Common source of false “internet is down” reports when client is simply on the wrong network segment

### Default gateway

The default gateway is the device that lets the local network talk to other networks, usually the internet or another office site.

If a device has a valid IP but no gateway, it may have local connectivity but no outside access.

Typical checks:

- ping the gateway
- confirm route configuration
- verify VLAN or firewall rules

### DNS

DNS translates domain names like `portal.company.local` or `www.example.com` into IP addresses.

Support symptoms:

- website names do not resolve
- user can reach some services but not others
- Outlook or VPN fails when the corporate DNS is misconfigured

Useful checks:

- `nslookup`
- `ping` a hostname
- check whether local DNS suffixes are correct

### DHCP

DHCP automatically assigns IP addresses, subnet masks, DNS servers, and gateway information.

Problems often include:

- no IP address assigned
- duplicate IPs
- slow lease renewal
- rogue DHCP server on the network

Common support actions:

- check lease status
- verify scope configuration
- inspect reserved IPs and exclusions
- compare the device with the corporate subnet design

### LAN vs WAN

- LAN = local area network, usually inside an office or site
- WAN = wide area network, connecting sites or branches

In IT support this matters because:

- a user problem may be local to the office
- a site-to-site problem may affect all users at a branch office
- connectivity issues can be isolated to a link, router, or ISP issue

### Router

A router forwards traffic between networks.

Typical tasks:

- connect office networks to internet
- route traffic between VLANs or sites
- handle NAT and firewall policies

Support triage:

- is the link up?
- is the gateway reachable?
- are routes correct?
- are ACLs or firewall rules blocking traffic?

### Switch

A switch connects endpoints inside a LAN and moves traffic based on MAC addresses.

Support relevance:

- port not forwarding traffic
- port locked to wrong VLAN
- dead switch port or failed cable
- device not receiving DHCP

### Firewall

A firewall filters traffic based on policy, ports, and destinations.

Common issues:

- blocked RDP or file-sharing ports
- incorrect VPN access policy
- remote users cannot reach internal apps
- rule changes unexpectedly break a service

### Wi‑Fi

Wireless is often the most common user complaint source.

Typical Wi‑Fi issues:

- poor signal strength
- channel congestion
- wrong SSID or security type
- expired certificate or authentication failure
- roaming between access points

Useful checks:

- signal level and noise
- client sees the right SSID
- DHCP is working
- 2.4 GHz vs 5 GHz behavior
- AP placement and radio health

### VPN

A VPN creates a secure tunnel for remote users or branch sites.

Support issues:

- username or MFA failure
- split tunnel misconfiguration
- certificate expiry
- server-side routing issue
- remote user cannot access internal file shares

### VLAN

A VLAN segments a network logically even when physically connected to the same switch.

Why it matters:

- separate voice, guest, management, and user traffic
- reduce broadcast domains
- isolate sensitive systems

Support symptoms:

- phone works but data does not
- guest Wi‑Fi cannot reach internal resources
- printer is on the wrong VLAN

### NAT

Network Address Translation allows private IP space to access public networks via shared external addresses.

Support relevance:

- internet access
- external services behind a firewall
- port forwarding and remote access rules

### Latency, jitter, and packet loss

These are important performance indicators.

- latency = delay
- jitter = variation in delay
- packet loss = dropped packets

IT support impact:

- VoIP call quality
- video conferencing quality
- file transfers timing out
- remote desktop sessions freezing

### Cable and physical layer

A lot of “network issues” are physically caused.

Typical causes:

- bad patch cable
- damaged wall jack
- switch port failure
- poor crimping
- dust or corrosion in connectors

Support habit:

- check link lights
- swap known-good cable
- inspect patch panel labeling
- verify cable run and patch location

---

## Network troubleshooting workflow

When a user says “the network is down,” the support sequence should be:

### 1. Define the symptom

Ask:

- Is it one user or many users?
- Is it Wi‑Fi only or all connectivity?
- Is internet unavailable or only one internal app?
- Did it begin after a change or outage?

### 2. Check the basics

- device has power
- network cable or Wi‑Fi connected
- IP address assigned
- gateway reachable
- DNS resolves names

### 3. Isolate the layer

- local device
- switch/port
- wireless access point
- firewall or router
- ISP or upstream provider
- application server or service

### 4. Test with simple tools

Common tools:

- `ping` for reachability
- `tracert`/`traceroute` for path analysis
- `ipconfig` / `ifconfig` for network state
- `nslookup` for DNS
- `arp -a` for local address mapping
- `netstat` for active connections

### 5. Verify business impact

- Is one department down, or the entire site?
- Is it a SaaS outage or internal app outage?
- Are there recent changes or maintenance windows?

### 6. Escalate with evidence

Good escalations include:

- impacted users and locations
- whether DHCP is working
- whether the gateway is reachable
- whether DNS resolves
- recent change windows or maintenance

---

## Data management concepts for IT support

### File server

A file server stores shared documents, project files, and departmental resources.

Support concerns:

- slow access
- permission issues
- storage full conditions
- offline or disconnected shares
- quotas and audit logs

### Shared drives and permissions

Shared data relies on correct access control.

Common support patterns:

- user cannot open folder
- user sees too much data
- a new employee cannot access project files
- someone was removed from a team but still has access

Best practice:

- group-based access, not manual user-by-user permissioning
- least privilege
- regular access review
- clear ownership for shared folders

### NAS and SAN

- NAS = network-attached storage, usually file-based
- SAN = storage area network, usually block-based and used by servers

Support relevant questions:

- Is the volume healthy?
- Is there a failed drive or rebuild in progress?
- Are there capacity alerts?
- Is there IOPS or latency degradation?

### Backup and restore

Backups protect against:

- hard drive failure
- ransomware or malware
- accidental deletion
- user error
- site disruption

Support tasks:

- verify backup jobs run successfully
- check retention windows
- test restore procedures
- confirm backup destination health

### RPO and RTO

- RPO = Recovery Point Objective: how much data can be lost
- RTO = Recovery Time Objective: how quickly service must be restored

These metrics support business continuity planning.

### Snapshot and versioning

Snapshots capture a point-in-time state of storage, often used for fast recovery.

Examples:

- restore a deleted file from last night
- recover a VM from a known snapshot
- recover a database to an earlier point

### Retention policy

Retention defines how long data must be kept and when it must be deleted or archived.

Support needs:

- legal compliance
- contract records
- audit trail requirements
- managing company storage growth

### Data lifecycle

Data lifecycle management covers:

- creation
- classification
- access
- retention
- archive
- deletion

This is important when teams want to reduce storage cost and minimize risk.

### Cloud storage and sync

Cloud tools are common in modern support environments.

Examples:

- OneDrive
- SharePoint
- Google Drive
- Dropbox
- enterprise sync services

Support concerns:

- sync errors
- conflicts from offline edits
- storage limit reached
- leftover stale files after offboarding
- permissions not inherited correctly

### Encryption and confidentiality

Data management is not only about storage; it is also about protection.

Common controls:

- disk encryption
- encrypted backups
- secure file transfer
- access logging
- classification labels

---

## Common IT support scenarios

### User cannot connect to the internet

Check:

- Wi‑Fi connected?
- device has IP address?
- default gateway responds?
- DNS resolves names?
- firewall or proxy policy blocking it?

### User cannot access internal file share

Check:

- DNS name resolves to the file server
- network path reachable
- user has permissions
- file server is online
- share mapping still valid
- VPN or split tunnel state if remote

### Printer is offline

Check:

- print server is reachable
- correct IP or hostname
- port status on switch or Wi‑Fi
- queue not stuck
- driver or spooler issue
- print permissions and authentication problems

### Email fails after a network change

Check:

- DNS records and MX configuration
- SMTP/IMAP/POP connectivity
- SSL certificate issues
- proxy or firewall rules
- user account or mailbox permission status

### Backup job failed

Check:

- storage is not full
- connectivity to backup target is healthy
- credentials still valid
- retention schedule not corrupted
- job logs show failed patch or missed interval

### Office branch has no internet but local LAN works

Likely causes:

- ISP outage
- router failure
- WAN link failure
- incorrect route or NAT policy
- firewall blocking upstream traffic

### New employee cannot get onto the network

Check:

- account exists and is active
- DHCP lease assigned
- VLAN or switch port correct
- wireless profile available
- required security policy applied
- onboarding ticket and asset inventory updated

---

## IT support checklist templates

### Basic network checklist

- Device has power and link light
- IP address assigned
- subnet matches expected range
- gateway is reachable
- DNS resolves names
- internet access tested
- device can access required services

### Data access checklist

- file share name resolves correctly
- user account is active
- permissions match role
- no stale mapping or disconnected drive
- requested data path exists
- file server health is normal
- backup or snapshot available if data loss occurred

### Incident triage checklist

- one user or many users?
- affected site or department?
- local or remote access?
- app-specific or full connectivity?
- recent changes or maintenance?
- evidence captured before remediation?

### Onboarding checklist

- account created
- email, VPN, and file access granted
- workstation or laptop assigned
- network access tested
- local policy configured
- data storage mapped correctly
- backup and device security state validated

### Offboarding checklist

- account disabled or removed
- VPN and email access revoked
- device returned or wiped
- shares and permissions reviewed
- data archived or transferred appropriately
- company-owned files secured

---

## Security and governance essentials

Even in a support catalog, security is part of stability.

### Least privilege

Users should only have access to the resources needed for their roles.

### Asset inventory

Every networked device should be tracked:

- user assignment
- location
- warranty or support status
- software version
- patch compliance

### Change management

Changes should be:

- documented
- tested
- approved when required
- rolled back if they cause outage

### Logging and audit trails

Useful for:

- identifying who changed a permission
- showing when a firewall rule was updated
- reviewing storage access anomalies
- tracing outage history

### Endpoint protection

IT support must work with security teams to reduce risk from:

- malware
- ransomware
- unauthorized software
- non-compliant devices

---

## Useful commands and tools

These are common in IT support and operations work.

### Windows

- `ipconfig /all`
- `ping <host>`
- `tracert <host>`
- `nslookup <domain>`
- `arp -a`
- `netstat -an`
- `route print`
- `tasklist`
- `sc query`
- `chkdsk`
- `diskpart`
- `fsutil volume diskfree c:`

### Linux / Unix

- `ifconfig` or `ip addr`
- `ping`
- `traceroute`
- `nslookup` or `dig`
- `netstat` or `ss`
- `df -h`
- `du -sh`
- `ls -l`
- `mount`
- `journalctl`

### Storage and backup checks

- free space percentage
- file growth rates
- restore validation
- backup completion logs
- snapshot status
- retention report

---

## Real-world support heuristics

### If a single user has a problem

Likely local or account-specific:

- profile issue
- permissions problem
- VPN profile issue
- local network adapter issue
- stale user mapping

### If several users at one site have a problem

Likely network or infrastructure issue:

- switch or AP problem
- firewall rule issue
- ISP link failure
- site router outage
- shared service outage

### If one application fails but internet works

Likely:

- app server issue
- DNS resolution issue
- dependency outage
- certificate problem
- firewall rule or NAT issue

### If no one has internet but local devices can still talk

Likely:

- gateway or router failure
- WAN or ISP outage
- upstream firewall issue
- DNS issue if only name resolution fails

---

## Glossary of common IT support terms

### Active Directory

A directory service used to manage users, computers, and permissions in a Windows-based environment.

### Authentication

The process of confirming that a user or device is who it claims to be.

### Bandwidth

The amount of data that can pass through a connection over time.

### Bare metal

A physical machine, as opposed to a virtual machine or cloud instance.

### Cache

Temporary storage used to speed up repeated access to data or files.

### Client

The device or software that requests a service from a server.

### Domain

A logical grouping of devices and users managed under a common directory or identity system.

### Endpoint

A user device such as a laptop, workstation, tablet, or phone.

### Fault domain

A scope where a failure can occur, such as a rack, switch, or site.

### Hostname

The human-readable device name used on a network.

### Log

A record of system or application events used for monitoring and troubleshooting.

### MDM

Mobile Device Management, used to enforce policies on managed devices.

### Port

A communication endpoint or logical channel used by network services.

### Proxy

An intermediary service used to filter, cache, or route traffic.

### Queue

A waiting line for print jobs or service requests.

### RAID

Redundant Array of Independent Disks, used to improve reliability or performance.

### Restore

The recovery of data or a system from backup or snapshot media.

### Service desk

The team or function responsible for handling user support requests and incident intake.

### Ticket

A tracked support request used to document issue details, work, and resolution.

### Uptime

The amount of time a system or service is available and operational.

### WAN link

A connection between sites or networks over a telecom or internet provider.

---

## Recommended operating habits

### Keep a clean mental map

For each issue, answer:

- what is the device?
- what network is it on?
- what service is it trying to reach?
- what changed recently?
- who is impacted?

### Troubleshoot in layers

Do not jump between network, app, and storage causes without isolating the problem.

### Record evidence

Capture:

- exact time of failure
- affected users and locations
- error messages
- recent change history
- test results

### Test before and after change

A good support process always confirms the problem before and the result after remediation.

### Prefer repeatable fixes

Document standard actions for common issues so help desk workflow improves over time.

---

## Example support playbook

### Example: user reports “I cannot access the shared drive”

1. Ask whether internet and email work.
2. Check whether the user is on the corporate Wi‑Fi or VPN.
3. Confirm the device has a valid IP address and gateway.
4. Verify whether the file server name resolves via DNS.
5. Check if the user has correct permissions on the share.
6. Try opening the same path from another machine.
7. Validate file server status and storage health.
8. Document cause and resolution in the ticket.

### Example: office internet is down

1. Check if the issue affects only one site or multiple sites.
2. Verify ISP or WAN link status.
3. Confirm router and firewall are online.
4. Check gateway reachability and route health.
5. Inspect upstream traffic and recent configuration changes.
6. Notify impacted stakeholders and follow the site restoration plan.

---

## Final mindset

The best IT support engineers do not memorize every device model. They understand patterns.

They ask:

- where is the problem?
- what layer is failing?
- what is the business impact?
- what evidence proves the cause?

Networking and data management are not isolated fields. In real support work, they are tied together:

- user access depends on network health
- file availability depends on storage health
- rapid recovery depends on backup strategy
- security depends on identity and permission integrity

If you can reason across these layers, you can keep business systems stable even when the environment is complex.

---

## Quick reference

### Network essentials

- IP address
- subnet
- gateway
- DNS
- DHCP
- router
- switch
- firewall
- VLAN
- VPN
- Wi‑Fi
- packet loss

### Data essentials

- file share
- permissions
- NAS/SAN
- backup
- restore
- snapshot
- retention
- archive
- encryption
- storage health

### Support essentials

- triage
- change management
- escalation
- documentation
- incident ticket
- root cause
- recovery verification

---

## Conclusion

This catalog is meant to be used as a practical reference, not a theoretical textbook. In day-to-day IT support, the best results come from clear thinking, strong fundamentals, and disciplined troubleshooting.

If you need to restore service quickly, start with the simplest facts:

- Is the endpoint online?
- Is it on the right network?
- Can it reach the required service?
- Does the user have the right access?
- Is the data available and backed up?

Those questions, answered carefully, usually lead to the fix.

---

This catalog is meant to be practical, concise, and operational. Use it as a field guide for troubleshooting, support readiness, and communication with end users and technical teams.
