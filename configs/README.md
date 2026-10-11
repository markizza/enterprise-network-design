# Device Configurations

Full `show running-config` output for every device in the topology.

| File | Device | Role |
|---|---|---|
| `R1-HQ.txt` | R1-HQ | Inter-VLAN routing, DHCP relay, ACL 100, IPsec endpoint |
| `R2-REMOTE.txt` | R2-REMOTE | Server-site routing, ACL 110, IPsec endpoint |
| `SW-DISTRIBUTION.txt` | SW-DISTRIBUTION | Trunk aggregation toward R1-HQ |
| `SW-ACCESS-1.txt` | SW-ACCESS-1 | Human Resources and Marketing access |
| `SW-ACCESS-2.txt` | SW-ACCESS-2 | Customer Centric and Information Systems access |
| `SW-SERVER.txt` | SW-SERVER | Server site |

These are unmodified exports from a Cisco Packet Tracer simulation. The credentials and keys in them protect nothing — there's no device to reach and no traffic to intercept — so I've left them as they are rather than redacting a simulated network.

If you're reading these to check my work, the parts worth looking at are the four 802.1Q subinterfaces and helper addresses on R1-HQ, the differing allowed-VLAN lists across the three trunks, the two access lists and where they're applied, and the mirrored crypto ACLs at either end of the WAN link.

Everything here is explained in [docs/design.md](../docs/design.md) and [docs/security-controls.md](../docs/security-controls.md).
