# Norbertino Networking Project

## A Secure, VLAN-Segmented Small-Office Network in Cisco Packet Tracer

**Author:** Ibeh Chukwuebuka Anthony  
**Platform:** Cisco Packet Tracer  
**Date:** 14<sup>th</sup> August, 2026.

## 1. Objective

The goal of this project was to design, build, and verify a small-office network that isolates departments at Layer 2 using VLANs, enforces access policy at Layer 3 using Access Control Lists (ACLs), and centralizes IP and name management through DHCP and DNS.

**Specifically, the project set out to:**

1\. Segment the network into five VLANs so that departments cannot see each other's traffic by default.

2\. Give IT the ability to administer the router and switch, but nobody else.

3\. Ensure only IT can reach the file server; HR and Sales may query it for DNS but must not otherwise reach it.

4\. Allow Guest Wi-Fi internet access only, with zero access to internal subnets.

5\. Provide automatic IP addressing (DHCP) per VLAN and internal hostname resolution (DNS).

6\. Prove the design works with repeatable ping and DNS tests.

## 2. Requirements

### 2.1 Hardware / Devices

| **Device**      | **Friendly Name**   | **Role**                                       |
| --------------- | ------------------- | ---------------------------------------------- |
| Router          | Norb's Router       | Inter-VLAN routing, DHCP, ACL enforcement, SSH |
| Switch          | Norb's Switch       | VLAN distribution, trunk to router             |
| Server          | Norb's Server       | File server + DNS server                       |
| PC              | HR's PC             | HR Department                                  |
| PC              | Sales PC            | Sales Department                               |
| PC              | IT Admin PC         | IT Department / management                     |
| Laptop          | Norb's Guest Laptop | Guest Wi-Fi client                             |
| Wireless Router | Norb's Guest WiFi   | Guest wireless gateway                         |

### 2.2 VLAN and IP Scheme

| **VLAN** | **Name**         | **Network**     | **Gateway**  |
| -------- | ---------------- | --------------- | ------------ |
| 10       | HR_Department    | 192.168.10.0/24 | 192.168.10.1 |
| 20       | Sales_Department | 192.168.20.0/24 | 192.168.20.1 |
| 30       | Server_Network   | 192.168.30.0/24 | 192.168.30.1 |
| 40       | Guest_WiFi       | 192.168.40.0/24 | 192.168.40.1 |
| 50       | IT_Department    | 192.168.50.0/24 | 192.168.50.1 |

###

### 2.3 Access Policy (Target Behavior)

| **Source** | **HR**      | **Sales**   | **Server**    | **IT** | **Guest** | **Internet** |
| ---------- | ----------- | ----------- | ------------- | ------ | --------- | ------------ |
| **HR**     | ✅          | ❌          | ❌ (DNS only) | ❌     | ❌        | ✅           |
| **Sales**  | ❌          | ✅          | ❌ (DNS only) | ❌     | ❌        | ✅           |
| **Server** | ❌          | ❌          | ✅            | ✅     | ❌        | ✅           |
| **IT**     | ✅ ping/RDP | ✅ ping/RDP | ✅ Full       | ✅     | ❌        | ✅           |
| **Guest**  | ❌          | ❌          | ❌            | ❌     | ✅        | ✅           |

###

### 2.4 Functional Requirements

- Router-on-a-Stick inter-VLAN routing on Gi0/0 sub-interfaces.
- Trunk (802.1Q) between router and switch carrying VLANs 10,20,30,40,50.
- DHCP pools per VLAN with correct gateways and DNS.
- DNS A-records hosted on Norbs-Server for all internal hosts.
- ACLs applied inbound on each VLAN sub-interface.
- SSH-only management of router and switch, restricted to VLAN 50 (IT).
- Default route to the guest Wi-Fi gateway for internet access.
- Save to startup-config on both router and switch.

### 2.5 Naming Convention

Human-friendly names (with apostrophes) are used in the topology diagram and documentation. Cisco configuration uses hyphens because IOS does not allow spaces or apostrophes in hostnames (e.g., HRs-PC, Norbs-Router).

##

## 3. Challenges Faced and How We Overcame Them

| **#** | **Challenge**                    | **Symptom**                                                      | **Root Cause**                                                        | **Resolution**                                                             |
| ----- | -------------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1     | Wrong VLAN ID on sub-interface   | `%Configuration of multiple subinterfaces ... with the same VID` | VLAN 20 was applied to Gi0/0.30 by mistake                            | Removed with no encapsulation dot1Q 20; reapplied correct VLAN IDs         |
| 2     | DHCP pool not configurable       | `% Invalid input detected at '^' marker`                         | Commands entered in privileged EXEC mode, not config mode             | Entered configure terminal first                                           |
| 3     | Wrong DHCP network address       | Sales clients got wrong IPs / pool inconsistent                  | Typo 132.168.20.0 and default-router 192.168.30.1                     | Removed pool, recreated with 192.168.20.0 and 192.168.20.1                 |
| 4     | No DNS server on router          | ip dns server → % Invalid input                                  | Packet Tracer 2911 IOS does not support DNS server                    | Hosted DNS on Norbs-Server; DHCP pools point DNS to 192.168.30.2           |
| 5     | IT couldn't ping HR              | 100% loss from IT to HR                                          | Deny rules were placed above the specific permit rules in ACL 110     | Reordered ACL: specific permits first, denies after, general permit last   |
| 6     | HR could ping IT (security hole) | HR→IT ping succeeded                                             | ACL 120 used permit icmp (all ICMP types, both directions)            | Changed to permit icmp ... echo-reply only, then deny ip                   |
| 7     | First ping always lost           | 1 of 4 packets lost on any first test                            | ARP resolution of destination MAC on first packet                     | Normal behavior; documented, not "fixed"                                   |
| 8     | Guest on wrong subnet            | Guest got 192.168.0.x instead of 192.168.40.x                    | Wi-Fi router performing NAT/DHCP on its own LAN                       | Put Wi-Fi router in bridge mode; uplink to VLAN 40; guest DHCP from router |
| 9     | SSH accessible from all VLANs    | HR could reach SSH on router                                     | ACL 10 not applied to vty lines                                       | Applied access-class 10 in under line vty 0 4                              |
| 10    | No internet reachability         | `Gateway of last resort is not set`                              | No default route                                                      | `ip route 0.0.0.0 0.0.0.0 192.168.2.2`                                     |
| 11    | Command typos                    | copy runnig-config → invalid                                     | Typo; also typed "yes" instead of pressing Enter for default filename | Corrected to copy running-config startup-config and pressed Enter          |
| 12    | Accidental ?                     | `% Unrecognized command`                                         | ? triggers IOS help                                                   | Not an error; commands retyped cleanly                                     |

The two most instructive failures were **ACL rule ordering** (#5) and **ICMP type selection** (#6). Together they taught the same lesson: an ACL is evaluated top-down, and "permit ICMP" ≠ "permit only replies." Getting policy right required thinking in terms of _direction_ and _type_, not just _source_ _and destination_.

## 4. Testing / Verification

All tests were run inside Packet Tracer using the PC Command Prompt and the router/switch CLI. Results below reflect the final, clean build.

###

### 4.1 Layer 2 — Switch Verification

### Norbs-Switch# Prompt: "_show vlan brief"_

### Result: VLANs 10, 20, 30, 40, 50 active; Fa0/1→10, Fa0/2→20, Fa0/3→30, Fa0/4→50

### Norbs-Switch# Prompt: "_show interfaces trunk"_

### Result: Gi0/1 trunking, allowed VLANs 10,20,30,40,50

### Norbs-Switch# Prompt: "_show vlan brief"_

### Result: VLANs 10, 20, 30, 40, 50 active; Fa0/1→10, Fa0/2→20, Fa0/3→30, Fa0/4→50

###

### 4.2 Layer 3 — Router Verification

### Norbs-Switch# Prompt: "s_how ip interface brief"_

### Result: All sub-interfaces Gi0/0.10–.50 up/up with correct gateways

### Norbs-Switch# Prompt: "show ip route_"_

### Result: Directly connected routes for 192.168.10.0/24, .20.0/24, .30.0/24

### .40.0/24, .50.0/24, .1.0/24, .2.0/24; default route via 192.168.2.2

### 4.3 Connectivity Tests

**From IT-Admin-PC (all should succeed):**

| **Target**             | **Expected** | **Actual** |
| ---------------------- | ------------ | ---------- |
| 192.168.50.1 (gateway) | ✅           | ✅ 0% loss |
| 192.168.30.2 (server)  | ✅           | ✅ 0% loss |
| 192.168.10.2 (HR)      | ✅           | ✅ 0% loss |
| 192.168.20.2 (Sales)   | ✅           | ✅ 0% loss |
| 8.8.8.8 (internet)     | ✅           | ✅ 0% loss |

**From HR's PC:**

| **Target**             | **Expected** | **Actual**   |
| ---------------------- | ------------ | ------------ |
| 192.168.10.1 (gateway) | ✅           | ✅ 0% loss   |
| 8.8.8.8 (internet)     | ✅           | ✅ 0% loss   |
| 192.168.30.2 (server)  | ❌           | ❌ 100% loss |
| 192.168.20.2 (Sales)   | ❌           | ❌ 100% loss |
| 192.168.50.2 (IT)      | ❌           | ❌ 100% loss |

**From Sales PC:**

| **Target**             | **Expected** | **Actual**   |
| ---------------------- | ------------ | ------------ |
| 192.168.20.1 (gateway) | ✅           | ✅ 0% loss   |
| 8.8.8.8 (internet)     | ✅           | ✅ 0% loss   |
| 192.168.30.2 (server)  | ❌           | ❌ 100% loss |
| 192.168.10.2 (HR)      | ❌           | ❌ 100% loss |
| 192.168.50.2 (IT)      | ❌           | ❌ 100% loss |

**From Guest Laptop:**

| **Target**            | **Expected** | **Actual**   |
| --------------------- | ------------ | ------------ |
| 8.8.8.8 (internet)    | ✅           | ✅ 0% loss   |
| 192.168.10.1 (HR gw)  | ❌           | ❌ 100% loss |
| 192.168.30.2 (server) | ❌           | ❌ 100% loss |
| 192.168.50.2 (IT)     | ❌           | ❌ 100% loss |

### 4.4 DNS Tests

**From IT-Admin-PC:**

### nslookup Norbs-Server → _"192.168.30.2"_

### nslookup HRs-PC → _"192.168.10.2"_

### nslookup SALEs-PC → _"192.168.20.2"_

### From HR's PC

### nslookup Norbs-Server → 192.168.30.2 (DNS allowed)

### ping Norbs-Server → 100% loss (ICMP blocked)

### _This pairing confirms ACL 100 permits UDP/TCP 53 but denies ICMP to the server subnet._

### 4.5 Management Access Tests

### From IT-Admin-PC

### ssh -l admin 192.168.10.1

### Password: \*\*\*\*\*\*\*\*

### Norbs-Router> → login succeeds

### From IT-Admin-PC

### ssh -l admin 192.168.10.1

### → connection times out (ACL 10 denies)

### 4.6 DHCP Verification

### Norbs-Router# _"show ip dhcp binding"_

### Result: HRs-PC, Sales-PC, and IT-Admin-PC each hold leases in their correct subnets with the correct gateway and DNS values

### 4.7 Summary of Results

All 22 verification checks passed:

- 5/5 VLAN isolation checks pass (HR≠Sales, HR≠IT, Sales≠IT, Guest≠all internal)
- 5/5 IT privilege checks pass (IT reaches all internal + internet)
- 4/4 internet-access checks pass (HR, Sales, IT, Guest)
- 3/3 DNS checks pass
- 2/2 SSH management checks pass (IT yes, others no)
- 3/3 DHCP binding checks pass

_The network is functioning as designed and securely segmented._

## 5. Configuration Files

Full show running-config output for the router and switch is committed under  
[View Router Configuration](./configs/Norbs-Router-running-config.cfg). The raw CLI session log (evidence of typed commands) is at  
[View Raw CLI Session Log](./configs/CLI-session-log.txt).
