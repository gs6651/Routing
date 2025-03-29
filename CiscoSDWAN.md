## Architecture & Components

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

### Overlay Management Protocol

- TCP based protocol
- OMP peering is between devices’ system-IPs, it’s not bound to any of DTLS control connections to vSmart
- There will be only one peering between vEdge & vSmart, irrespective of number of WAN connections available

### OMP is responsible for distribution of:

- TLOCs among network sites
- Service side reachability information
- Service-Chaining information
- Data-Plane security parameters, VPN labels & crypto keys
- Data & Application Aware Routing (AAR) policies

### Routes advertisement with OMP: (3 Types)

- OMP routes (vRoutes): Prefixes at the local site that are redistributed into OMP and advertised towards the controllers. Basically the LAN routes. It can be OSPF, BGP, static or connected routes, or any other routing information present on the site. OMP routes resolve their next-hop to a TLOC. An OMP route is installed in the forwarding table only if the next-hop TLOC is known and there is a BFD session in UP state associated with that TLOC. By default, OMP only advertises the best route out of multiple equal ones. If no policy is applied, advertise all TLOCs to all vEdges. It carries below attributes:
  
  - VPN Number (1 -to - 510)
  - Originator: System-IP of vEdge
  - TLOC: Next-Hop identifier of the OMP routes
  - Site-IDs: Similar to BGP ASN. All sites must have a unique Site-ID, and all vEdges must have the same Site-ID on the same site.
  - Origin-Protocol: From where vEdge has learned it i.e., via connected, static or any dynamic routing protocol
  - Origin-Metric: It helps in the best-path algorithm when OMP calculates the most optimal routes toward destinations
  - OMP Preference: Similar to BGP’s local-preference. Higher is better. To influence the best path selection in OMP
  - Tag: Similar to route-tag. It’s a transitive attribute
  
  ### TLOC routes:
  
    These are the tunnel endpoints on the WAN Edge routers that connect to the transport networks. Basically the WAN routes. These routes are advertised along with additional attributes such as public and private IP addresses, color, TLOC preference, site ID, weight, tags, and encryption keys. System-IP address is used instead of the interface IP address as an identifier for a TLOC route. A TLOC route advertisement contains the following attributes:
  
  - Private IPv4/IPv6 addresses and ports: DHCP or manually configured IPs on WAN interfaces
  - Public IPv4/IPv6 addresses and ports: If the vEdge sits behind a NAT device, the outside NATed IP addresses and ports are included in the TLOC route advertisements. If not behind NAT, the public and private addresses and ports are the same
  - Color: For logical abstraction between multiple WAN types
  - Encapsulation Type: GRE or IPsec. Must match with remote TLOC
  - TLOC Preference: To influence OMP Best path when having multi-paths. Higher is better, default is 0.
  - Site-ID: To identify the originating site. If remote & local vEdges Site-ID is the same, they  will not form a tunnel.
  - Tag: User defined value that can be acted upon in a control policy
  - Weight: A parameter to achieve unequal traffic distribution across multiple TLOCs with equal preferences. Higher means more flows will go through that TLOC compared to other TLOC on vedge.

### Service routes:

- These are used to exchange embedded services such as firewall, IPS, application-specific optimizations, and load-balancers. Network Service (FW, IPS etc.) must be Layer-2 adjacent to vEdge (There must not be a Layer 3 device in-between).  vSmart does not advertise Service routes to vEdges. Service routes contains the following attributes:
- VPN-ID: VPN that the service applies to
- Service-ID: Type of service that is being advertised. 7 pre-defined
  - FW maps to svc-id 1;
  - IDS maps to svc-id 2;
  - IDP maps to svc-id 3;
  - Custom Services: For customer defined
    - netsvc1 maps to svc-id 4;
    - netsvc2 maps to svc-id 5;
    - netsvc3 maps to svc-id 6;
    - netsvc4 maps to svc-id 7;
  - Originator ID: System-IP of the vEdge that originates the service route
  - TLOC: Where service is located.

### OMP Best-Path Selection

- By default, only the best 4 routes (according to OMP Best-Path Algorithm) are advertised out. Maximum 16 can be advertised. To change the behaviour on vSmart, use the command: `omp send-path-limit <1-16>`. Controllers can also be configured to send a backup path (`omp send-backup-paths`).

- The OMP best-path algorithm does not only select the best routes but also sorts them in descending order (from best to worst).

- The vSmart always inserts and keeps all routes sorted, with the best route at the top.

- The `omp send-path-limit <1-16>` parameter defines the maximum number of best-paths that can be advertised.

- The `omp ecmp-limit <5>` parameter defines the maximum number of best paths that can be installed in the routing table.

- The `controller-send-path-limit` defines the maximum number of best-paths that a vSmart can advertise to another vSmart.

- The `omp send-backup-paths` tell vSmart to advertise the first set of non-best routes.

- Route State: A route is ACTIVE when there is an OMP session in UP state with the peer that sent out the route. A route is STALE when the OMP session with the peer that sent out the route is in GRACEFUL RESTART mode

- Route Resolvability: Valid & Reachable next-hop TLOC

- Admin Distance: vEdge Only. Locally significant. OMP has AD of 250 on vEdges and 251 on cEdges. AD is not a parameter in OMP, is not advertised, and does not influence vSmart.

- Route Preference: Default OMP route preference is 0. 

- TLOC Preference: vEdge Ony. TLOC routes are not bound to VPN-id. Therefore, changing the TLOC preference affects vEdges path selection for all VPNs.

- Origin:
  
  - Connected 
  - Static
  - EIGRP summary
  - eBGP 
  - OSPF intra-area 
  - OSPF inter-area
  - IS-IS level 1
  - EIGRP external
  - OSPF external 
  - IS-IS level 2
  - iBGP 
  - Unknown

- Origin Metric: If Origin type is same, select the route with lower origin metric

- Tiebreaker 1 (Source Preference): vSmart only. Prefer vEdge sourced routes over vSmart sourced

- Tiebreaker 2 (System IP): Select the routes that have the lowest router-id (System-IP).

- Tiebreaker 3 (Private TLOC IP): For routes coming from the same vEdge, prefer the ones with a lower private TLOC IP address.

#### OMP Graceful Restart:

- When SD WAN control Plane becomes unavailable, data planes continue functioning and forwarding traffic. Cisco vEdge and vSmart devices cache the OMP information that they learn from peers. The cached information includes OMP, TLOC, and SERVICE routes, IPsec SA parameters, and the centralized data policies in place.

**TLOC Action** is a control-policy option that inserts an intermediate hop in an OMP route to a destination prefix. The intermediate TLOC and the ultimate TLOC must be the same color for the tloc-action to work. The intermediate router must be enabled for service TE. There are four tloc-action (s):

- Primary

- Backup

- ECMP

- Strict (default)

## TLOC

- Transport Locator

- Collection of entities making up a transport side connection. Think of it as a identity card given to WAN interfaces, it uniquely identifies with tuple of three values:
  
  - System-IP: It’s similar to router-ID and does not need to be routable or reachable across the fabric
  - Transport Color: Interface identifier on local vEdge. One color for MPLS and other one for Internet etc.
  - Encapsulation: The type of encapsulation this TLOC uses - IPsec or GRE.

- Transport interfaces (ones with TLOC configuration) do not forward traffic. They only serve as tunnel endpoints for the overlay tunnels

- TLOC Restrict: To restrict tunnels within same color

- TLOC Carrier: Between two clouds

- Tunnel Group:
  
  - Only one interface can be marked with a particular color per WAN edge router. That's a limitation of color, and Tunnel Group comes into picture.
  - TLOCs can only establish tunnels with remote TLOCs with the same tunnel-group IDs irrespective of the TLOC color.
  - TLOCs with any tunnel-group ID will also form tunnels with TLOCs that have no tunnel-group IDs assigned.
  - If the restrict-option is configured in conjunction with the tunnel-group option, then TLOCs will only form an overlay tunnel to remote TLOCs having the same tunnel-group ID and TLOC color.

- NAT Detection

- All Cisco SD-WAN devices have an embedded STUN client and the vBond orchestrator acts as a STUN Server (Session Traversal Utilities for NAT).

- 

- NAT Types:
  
  - Full-Cone NAT: A full-cone is one where all packets from the same internal IP address are mapped to the same NAT IP address. External hosts can send packets to the internal host, by sending packets to the mapped NAT IP address. In other words, One-to-One NAT.
  - Restricted-Cone NAT: Same as Full-Cone NAT, only difference is that an external host can send packets to the internal host only if the internal host had previously sent a packet to that IP address. Once the NAT mapping state is created, the external destination can communicate back to the internal host on any port. It is also known as Address-Restricted-Cone.
  - Port-Restricted-Cone NAT: Same as Restricted-Cone NAT, but the restriction also includes port numbers. The difference is that an external destination can send back packets to the internal host only if the internal host had previously sent a packet to this destination on this exact port number. It is known as Port Address Translation (PAT).
  - Symmetric: In this NAT, all requests from the same internal IP & port to a specific destination IP & port, are mapped to a unique NAT IP & and NAT port. Only the external destination that received a packet can send packets back to the internal host. It is the most restrictive of all other types. It is also known as Port Address Translation (PAT) with port-randomization.

- Recommendation is to have Full-Cone NAT at the Hubs, DC, DR sites.

- In GRE tunnels, any type of NAT with port overloading is not supported since GRE packets lack an L4 header.

- Loopback TLOC: When loopback interface is used as local TLOC endpoint.
  
  - Standard Mode: Loopback interface bound to multiple physical interfaces. The service provider network must have IP reachability to the loopback IP. Tunnel config would be done under loopback instead of physical interface.
  - Bind Mode: Loopback interface is strictly bound to a single physical link. Used when the last mile provider assigns an IP address to the WAN link, and this IP is filtered within the MPLS cloud or in another transit WAN along the way. Command is:  bind <interface>.

#### Structure of Policy

- List: Which Site (Branch 1, Branch 2, HUB etc.)
- Definition: What Action
- Application: Where to apply policy. (in-bound or out-bound)

#### Centralized policies

- One type of policy can be applied to a site-list. For example, one control-policy in and one control-policy out but not two control policies in the outbound direction.
- Cisco does not recommend including a site in more than one site-list. Doing this may result in unpredictable behaviour of the policies applied to these site-lists.
- Centralized-*Control*-policy is unidirectional applied either inbound or outbound. For example, If we need to manipulate OMP routes that the controller sends and receives, we must configure two control policies.
- Centralized-*Data*-policy is directional and can be applied either to the traffic received from the service side of the vEdge router, traffic received from the transport side, or both.
- VPN membership policy is always applied to traffic outbound from the vSmart controller.

### Order-of-Operation on vEdge

- *IP lookup*: 1st thing 1st, IP address lookup, because vEdges at the heart are routers 🙂.
- *Local Ingress Policy*: Localized policies are typically used to create ACLs and tie them to vEdge interfaces. ACLs can be used for filtering, marking, and traffic policing.
- *Centralized App-Route Policy*: or App-Aware Routing. If it’s configured, the packet being inspected makes a routing decision based on the defined SLA characteristics such as packet loss, latency, jitter, load, cost, and bandwidth of a link.
- *Centralized Data Policy*: It is evaluated after the App-Aware Routing policy and is able to override the App-Aware Routing forwarding decision.
- *Forwarding*: At this point, the destination IP address is compared against the routing table, and the output interface is determined.
- *Security Policy*: If there are security services attached to the vEdge, they are processed in the following sequence - Firewall, IPS, URL-Filtering, and lastly AMP (Advanced Malware Protection). The necessary tunnel encapsulations are performed and VPN labels are inserted.
- *Local Egress Policy*: If traffic is denied or manipulated by the egress ACL, those changes will take effect before the packet is forwarded.
- *Queueing and Scheduling*: Egress traffic queueing services such as Low-Latency (LLQ) and Weighted Round Robin (WRR) queueing are performed before the packet leaves.
