# What I'd Change Before Production

What's still exposed after everything I implemented, in the order I'd raise it.

---

## 1. The tunnel protects the wrong traffic

The IPsec tunnel covers Human Resources traffic to the external server site. That's what the brief asked for, and it's what I built. Customer data crosses the same WAN link with nothing protecting it.

I'd raise this first because it isn't a misconfiguration. I checked the tunnel and it works — correct parameters, established security association, counters moving, selectors mirrored on both sides. Every box ticked. The problem is what's inside the scope.

I only noticed it because of a test. I sent Marketing traffic to the server site expecting to see the counters move, and they didn't — the traffic went straight across the link in the clear, exactly as designed. That's the moment the gap became visible, and I don't think I'd have found it by reading the configuration.

What I take from that: checking whether a control works and checking whether it covers the right thing are two different activities, and the second one is the harder one. A financial services company moving customer records over an unprotected WAN segment has a problem no matter what the requirements document said.

**What I'd do.** Widen the crypto ACL to cover all inter-site traffic. If there's a reason not to — cost, performance, something else — then write down why selective encryption is acceptable for this data, and get someone to agree to it. There's an ethics angle here too where special-category data is involved, which I've written about below.

---

## 2. IKEv1 and Dated Cryptographic Parameters

Phase 1 uses:

- AES-256 encryption
- SHA-1 hashing
- Pre-shared-key authentication
- Diffie-Hellman Group 5 (1536-bit)
- 3,600-second lifetime

IKEv1 works in Packet Tracer but is deprecated and should not be used for a new production deployment.

SHA-1 has known collision weaknesses and is being phased out. Those collision attacks are more serious for digital signatures than for its HMAC/PRF use in IKE, but SHA-1 is still a legacy choice and SHA-256 should be preferred for new deployments.

DH Group 5 also has too small a security margin. It uses 1536-bit finite-field Diffie-Hellman, while current IETF guidance recommends stronger groups such as Group 14 (2048-bit) or Group 19 (256-bit elliptic curve). Reused finite-field groups can also benefit from expensive precomputation attacks; Group 19 avoids the same fixed finite-field weakness while providing stronger security with a much smaller key.

**What I would use in production:**
For a two-site organisation like this, I would use **IKEv2, AES-256-GCM, SHA-256, DH Group 19 and a strong unique pre-shared key**. A certificate-based design would be preferable if the organisation expanded to many VPN peers, but it would introduce unnecessary PKI overhead for only two sites.

---

## 3. Router-on-a-stick: one link, one router, everything

All inter-departmental traffic goes through a single physical link between SW-DISTRIBUTION and R1-HQ, and crosses it twice. R1-HQ is also the only inter-VLAN routing device, the DHCP relay, and one end of the VPN.

At seventeen hosts this isn't a measured problem, and I'd rather accept a limit I understand than pay for capacity I can't justify. But accepting it only counts as a decision if I write down when it stops being acceptable.

**What I'd do.** Layer 3 switching with SVIs at the distribution layer removes the bottleneck and the single routing dependency at the same time. The trigger for that migration is inter-VLAN traffic volume approaching the capacity of that one link.

---

## 4. The ACLs are stateless

IOS extended ACLs match source, destination, protocol and port. The `established` keyword inspects TCP flags rather than session state, so a packet with the ACK bit set passes a rule that looks like it only permits return traffic.

**What I'd do.** Stateful inspection at the site boundary.

---

## 5. The OSPF Key Never Changes

The OSPF MD5 authentication key was configured once and has no automatic rotation process.

To rotate it without dropping the OSPF adjacency, I would:

1. Add a new key with a different key ID on both routers.
2. Keep the existing key active during the transition.
3. Confirm the OSPF neighbour relationship remains `FULL`.
4. Remove the old key.

In production, I would use managed key rotation and prefer **HMAC-SHA-256** over MD5 where the platform supports it.

---

## 6. Nothing in this design knows who anyone is

This is the one that bothers me most, and it's structural rather than fixable with a config change.

Port security binds a port to a MAC address. VLANs segment by switch port. ACLs filter by address and port. IPsec protects by traffic selector. Every single control is topological — they all answer "where is this traffic coming from" and none of them answers "who is this".

So someone sitting at a permitted port, with a spoofed MAC, holding an address in a permitted subnet, satisfies the entire design. ARP spoofing operates underneath all of it, invisible to every control above. VLAN membership isn't identity assurance and I was treating it as though it were.

NIST's zero-trust guidance makes the same point — don't grant trust based on network location, and apply network-tier and identity-tier policy together rather than relying on either alone.

**What I'd do.** 802.1X at the access edge for port-based authentication, and identity-aware policy above the network tier. With the caveat that zero trust is an approach rather than a product, and the benefit comes from implementing it rather than from describing a design that way.

---

## Legal and Ethical Considerations

### Article 25 – Data Protection by Design

VLAN segmentation and ACLs were built into the network design to reduce unnecessary access between departments and limit lateral movement if one device is compromised.

### Article 32 – Security of Processing

IPsec, ACLs, port security and device hardening improve confidentiality and integrity.

However, the design has limited security monitoring and little redundancy. A production environment should include centralised logging, accurate NTP time synchronisation and SIEM monitoring so that security incidents can be detected and investigated.

### Article 33 – Breach Notification

Article 33 requires qualifying personal-data breaches to be reported within 72 hours of becoming aware of them.

Without effective logging and monitoring, DFS may take longer to discover and understand a breach. The missing monitoring controls are therefore mainly an Article 32 security weakness, but they could also make it harder for the organisation to meet its Article 33 reporting obligation.

### Selective Encryption

The project encrypts traffic between HR and the external server site because this was required by the scenario.

However, a financial-services company may also handle sensitive customer and financial information. In production, I would recommend protecting all sensitive inter-site traffic rather than assuming that HR data is the only traffic requiring encryption.

### Professional Responsibility

The BCS Code of Conduct requires an engineer to follow business requirements while also identifying and communicating risks.

Therefore, although the HR-only VPN satisfies the project requirement, the remaining exposure of other sensitive traffic should still be documented and raised.

### Employee Privacy

Security information such as MAC-address records, ACL counters and system logs can also reveal employee activity.

This information should only be collected for a clear security purpose, restricted to authorised personnel and retained only for an appropriate period.

---

## What I couldn't test

Packet Tracer simulates. The cryptography is modelled rather than performed, so nothing in this project demonstrates that the parameters I picked are cryptographically sound — only that the configuration is correct and the policy behaves the way I expected. No real throughput, no genuine attack traffic, no hardware failures.

I'd rather say that than let someone assume otherwise.
