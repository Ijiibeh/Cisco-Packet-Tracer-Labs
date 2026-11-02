# Verification Results — Norbertino Networking Project

All checks below were run in Packet Tracer against the final, clean build.
Pass = behavior matches Section 2.3 of the write-up.

## Layer 2 — Switch
| Check | Command | Expected | Result |
|-------|---------|----------|--------|
| VLANs exist | `show vlan brief` | 10,20,30,40,50 active | PASS |
| Port assignments | `show vlan brief` | Fa0/1→10, Fa0/2→20, Fa0/3→30, Fa0/4→50 | PASS |
| Trunk | `show interfaces trunk` | Gi0/1 trunking, VLANs 10-50 | PASS |

## Layer 3 — Router
| Check | Command | Expected | Result |
|-------|---------|----------|--------|
| Sub-interfaces up | `show ip interface brief` | Gi0/0.10–.50 all up/up | PASS |
| Routing table | `show ip route` | 5 connected nets + default | PASS |
| ACLs applied | `show running-config` | 100,101,110,120,130 on correct sub-ifs | PASS |

## Connectivity
| From | To | Expected | Result |
|------|-----|----------|--------|
| IT | HR (10.2) | ✅ | PASS |
| IT | Sales (20.2) | ✅ | PASS |
| IT | Server (30.2) | ✅ | PASS |
| IT | Internet (8.8.8.8) | ✅ | PASS |
| HR | Gateway (10.1) | ✅ | PASS |
| HR | Internet | ✅ | PASS |
| HR | Server | ❌ | PASS (blocked) |
| HR | Sales | ❌ | PASS (blocked) |
| HR | IT | ❌ | PASS (blocked) |
| Sales | Gateway (20.1) | ✅ | PASS |
| Sales | Internet | ✅ | PASS |
| Sales | Server | ❌ | PASS (blocked) |
| Sales | HR | ❌ | PASS (blocked) |
| Sales | IT | ❌ | PASS (blocked) |
| Guest | Internet | ✅ | PASS |
| Guest | HR/Sales/Server/IT | ❌ | PASS (blocked) |

## DNS
| From | Query | Expected | Result |
|------|-------|----------|--------|
| IT | `nslookup Norbs-Server` | 192.168.30.2 | PASS |
| IT | `nslookup HRs-PC` | 192.168.10.2 | PASS |
| IT | `nslookup Sales-PC` | 192.168.20.2 | PASS |
| HR | `nslookup Norbs-Server` | 192.168.30.2 | PASS |
| HR | `ping Norbs-Server` | 100% loss | PASS (blocked) |

## Management
| From | Action | Expected | Result |
|------|--------|----------|--------|
| IT | `ssh admin@10.1` | Succeeds | PASS |
| HR | `ssh admin@10.1` | Times out | PASS (blocked) |

## DHCP
| Client | Subnet | Gateway | DNS | Result |
|--------|--------|---------|-----|--------|
| HRs-PC | 192.168.10.0/24 | 192.168.10.1 | 192.168.30.2 | PASS |
| Sales-PC | 192.168.20.0/24 | 192.168.20.1 | 192.168.30.2 | PASS |
| IT-Admin-PC | 192.168.50.0/24 | 192.168.50.1 | 192.168.30.2 | PASS |

**Summary: 22/22 checks passed.**
