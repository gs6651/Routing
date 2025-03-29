
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
