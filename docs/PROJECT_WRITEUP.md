# Norbertino Networking Project

A secure, VLAN-segmented small-office network built in Cisco Packet Tracer.

## Quick Summary

- **5 VLANs** — HR, Sales, Server, Guest WiFi, IT
- **Router-on-a-Stick** inter-VLAN routing (802.1Q)
- **ACLs** enforcing department isolation + IT-only server/management access
- **DHCP** per VLAN, **DNS** hosted on the server
- **SSH-only** management restricted to IT

## Repo Structure

| Path | Contents |
|------|----------|
| `docs/PROJECT_WRITEUP.md` | Full project write-up (objective, requirements, challenges, testing) |
| `docs/verification-results.md` | Standalone test results *./configs/verification-results.md* |
| `images/Norbetinos-Network-Setup-Topology.png` | Topology diagram |
| `configs/` | `show running-config` files, ACL summary, VLAN summary |
| `configs/CLI-session-log.txt` | Raw typed command session (evidence) *./configs/CLI-session-log.txt* |
| `Norbertino-Network.pkt` | The main Packet Tracer file (in the root directory) |

## Access Policy at a Glance

| Source | HR | Sales | Server | IT | Guest | Internet |
|--------|-----|-------|--------|-----|-------|----------|
| **HR** | ✅ | ❌ | ❌ (DNS only) | ❌ | ❌ | ✅ |
| **Sales** | ❌ | ✅ | ❌ (DNS only) | ❌ | ❌ | ✅ |
| **Server** | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |
| **IT** | ✅ ping/RDP | ✅ ping/RDP | ✅ Full | ✅ | ❌ | ✅ |
| **Guest** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

## Result

22/22 verification checks passed. See `docs/verification-results.md`. *(File to be added)*

## How to Reproduce

1. Open the `Norbertino-Network.pkt` file in Cisco Packet Tracer.
2. Confirm all devices are powered and links are green.
3. Run the verification commands from `docs/verification-results.md`.
4. Compare against the configurations in the `configs/` folder.
