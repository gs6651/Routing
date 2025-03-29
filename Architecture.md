
- SD-WAN is an overlay technology to run between WAN edges, over the transports or underlays (MPLS, Internet, Cellular, Satellite links etc.) provided by ISPs.
- Underlay tunnels provide IP reachability with each other.
- **Management Plane**: Telnet, SSH, NETCONF
- **Control Plane**: Routing Protocols like OMP, which only runs between vSmart and vEdges.
- **Data Plane**: Physical Devices. OSPF, BGP etc.

### Components

#### vBond
- Orchestration Plane.
- Authenticator. It does authentication of devices. Think of it as a Gatekeeper.
- vEdges are pre-configured with vBond IPs. It exchanges certificates with vEdges. After successful authentication, shares the IPs of vManage & vSmart.
- It requires public IP reachability. Could sit behind 1:1 NAT.
- Highly resilient.
- Between vBond & vEdge there is a dynamic DTLS tunnel, which goes down after successful authentication.
- Facilitates NAT-Traversal. vBond acts as a STUN Server: Session Traversal Utilities for NAT


#### vManage
- It’s part of Management-Plane
- Virtual Device (VM). On-Prem or Cloud Installation
- It’s a single pane of glass for Day0, Day1 & Day2 operations.
- Centralized Provisioning
- Central Policies & Templates
- Central Troubleshooting & Monitoring
- Software Upgrades
- It does not establish OMP tunnels with vEdges.
- To push config to vEdges, it uses NETCONF APIs over DTLS Tunnels.
- vBond is also configured via vManage.

#### vAnalytics
- It’s an optional component of SD-WAN fabric and only available as a SAAS (same like O365).
- Offers insights into the performance of applications and the underlying SD-WAN network infrastructure. E.g., link utilization, performance, delay etc.
- vManage collects data from all vEdges and shares the raw data with - vAnalytics, then it presents it in a better way.

#### vSmart
- Control Plane
- Distributes control plane information (Routing) to the vEdges using OMP. Also implements control plane policies such as service-chaining, multi-topology and multi-hop.
- Distributes data plane and app aware routing policies to the vEdges.
- All the policies for Data Flow are defined centrally on vManage and distributed to vEdges using vSmart. vSmart does not store policies locally, it only loads the currently active policy in its running-configuration.
- vEdges talks to vSmart, but not with each other. It acts as a Route-Reflector and doesn’t participate in Data-Plane, which means if you’re on vEdge_A, and look for a route from vEdge_B, next hop would be vEdge_B and not vSmart.


#### vEdge
- Data-Plane
- Communicates to vSmart controllers using OMP to setup the Data Flow
- Each vEdge is uniquely identified by its Chassis ID and certificate serial number
- Establishes permanent tunnels with vManage & vSmart, but dynamic tunnels with vBond. Tunnels are established via each available transport.


### Misc.
#### VPN_0
- Transport VPN, can neither be deleted or modified
- The purpose is to enforce a separation between the WAN transport networks (the underlay) and network services (the overlay).
For Control Plane & Management
- Authentication traffic between vBond & vEdges
- Template deployment traffic between vManage & vEdges.
- Traffic between vManage & vBond 

#### VPN_512: Out of Band Mgmt VPN (Mgmt VRF)

#### Service VPN
- Where LAN is connected
- Ranges between 1 & 511. Depending on hardware or license can support a limited number of service VPNs.
- E.g., Data users in VPN10, Voice Users in VPN11. It's a kind of segments in the VeloCloud world.

#### Transport Side
- Controller or vEdges side (interfaces) connected to the underlay/WAN network. Always VPN0.
- Traffic typically tunneled / encrypted. In some special cases we need to specifically tell vEdges not to encrypt traffic but rather send it natively like in case of Direct Internet Access or split-tunnel.

#### Service Side
- vEdge interfaces facing LAN
- Traffic is forwarded as is from the original source

#### DTLS vs TLS
- Cisco SD WAN supports two Transport Layer Security protocols to provide end-to-end transport security
- DTLS uses UDP and implements its own sequence numbers, fragment offers and retransmissions because UDP doesn't guarantee reliable delivery of packets. DTLS is default, because it’s faster compared to TLS.
- TLS uses TCP.
- Both are used between vManage, vSmart & VBond and they use protocol NETCONF for communication with each other.

#### Gateway Tracking
This feature that is enabled by default and can’t be stopped or modified. Each device probes using ARP the next-hop IP of each underlay static route every 10 seconds. If the device receives an ARP response, it maintains the static route in the VPN0’s routing table. If the device misses ten consecutive ARP responses for a next-hop IP, the device removes the static route that points to this IP from the routing table.

#### NETCONF
Network management protocol to manage remote configurations. It works on the RPC layer (Remote Procedure Call) and uses XML or JSON for data encoding. Typically the protocol messages are exchanged over the TLS with mutual X.509 authentication.  

