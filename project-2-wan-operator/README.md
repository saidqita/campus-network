# Project 2 — WAN & Operator Network

Where Project 1 stopped at the core, this one starts. Two ISR 4331s above the core, two links up to an ISP, and a whole different set of questions: who do you peer with, what do you advertise, and who gets to reach what.

![Topology](project-2-wan-operator/images/Topology2.png)

### Getting out to the internet at all
Two ISRs deployed as a dual-homed edge, 1000BASE-LX fiber to the ISP. Static defaults outbound, NAT overload on both edges so the six user VLANs share the public addresses. `default-information originate` on IE1 pushes that default into OSPF, giving the rest of the network somewhere to send its internet-bound traffic.

### Making OSPF scale
Single-area worked, until it obviously wouldn't. Migrated to multi-area — each faculty became its own area (10, 20, 30), backbone stayed as 0. The DLS switches act as ABRs and summarize the user networks outward, so the core ends up with one route per faculty rather than a stack of /24s per module.

### Deciding who gets to SSH in
A standard ACL on both edge routers permits only the three MGMT VLANs. Everything else gets dropped before the router even thinks about the session.

### Shaping what leaves and what comes back
Two extended ACLs, named `Outgoing-Traffic` and `Ingoing-Traffic`. Students get the protocols they actually need — HTTP/HTTPS to the campus web server, DNS to the campus DNS, ICMP where it fits — and inbound only allows the return traffic for those. Stateless policy doing a rough impression of a stateful firewall, applied at the ingress closest to the source in each direction.

### Peering with the ISP
eBGP replaces the static default. AS43665 peering with AS3301 from both edges, and a loopback `1.2.3.4` on IE1 advertised out. Internet hosts reach it; IE2 still learns the path internally over OSPF. The inbound ACL from the previous module needed loosening to permit BGP — one of those lessons you only really learn by watching your edge go silent and spending a minute figuring out why.
