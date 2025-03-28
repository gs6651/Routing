Multi-Protocol
Can Transport different payloads
L2 Payloads
Ethernet
FR
ATM
PPP
HDLC
L3
IPv4
IPv6


Label-Switching
Switches traffic b/w interfaces based on locally significant labels
Somewhat similar as :
FR's DLCI
ATM's VPI/VPC


MPLS Label Header Format
4-Byte
20-bit Label = Locally significant to Router & neighbor Router
3-bit EXP = Class of Service
1-bit S = Indicates Bottom of Stack
8-bit TTL = Time to Live
 
----------------------------------------------------------------------
|| 20 bit LABEL ||  3-bit EXP || 1-bit S || 8-bit TTL ||
----------------------------------------------------------------------



Misc.
Implicit-NULL (Pop Label (Label = 3))  label is used for PHP
To remove top labels
Explicit-NULL = (Label =0) = No PHP
To preserve EXP values, which is not possible in Implicit-NULL
Router Alert Label (Label = 1) :  MPLS Operation & Maintenance
OAM Alert Label : (Label = 14), Failure detection, localization & Performance Monitoring, Cisco don't use it
No label / Unlabeled / Untagged = Not running MPLS (Towards CE)
To remove all labels (Entire Stack)
Aggregate Label = To perform routing lookup as per global routing table. In other words LSR is doing aggregation or summarization, and needs to send after removing labels, so that IP lookup can be done for a more specific route.
FEC = MPLS Labels + IP Prefixes (+Interfaces and NH)
An FEC is a set of packets that a single router:
(1)  Forwards to the same next hop;
(2)  Out the same interface; and
(3)  With the same treatment (such as queuing).




TDP vs. LDP 
Tag Distribution Protocol (TDP) 
Originally used with Cisco's Tag Switching 
Uses UDP broadcast to port 71 1 to discover neighbors 
Once discovered, TCP session is setup on port 711 
Label Distribution Protocol (LDP) 
Standard per RFC 3036 
Uses UDP multicast to 224.0.0.2 at port 646 to discover neighbors 
Once discovered, TCP session is setup on port 646



TDP & LDP Caveats 
Label protocol must match for adjacency 
debug ip packet detail can be used to discover remote label protocol 
Devices must have route to transport-address to establish TCP session 
Transport address comes from LDP Router-ID 
Router-ID selection similar to OSPF/BGP/etc. 
Can be modified with interface level commands... 
IOS  mpls ldp discovery transport-address 
IOS XR discovery transport-address 



Overlay VPNs 
Service Provider does not participate in customer routing 
Must be provisioned prior to communication 
Frame Relay & ATM PVCs 
Leased lines 
GRE Tunnels 
Overlay suffers from (n(n-l ))/2 scalability issues 
Allows customers to use flexible addressing scheme 

Peer to Peer VPNs
Service Provider does participate in customer routing 
No static provisioning required 
Service Provider required to keep customer traffic separate through route filtering and access-lists
Does not allow customers to use flexible addressing
Problems with default routing



Different MPLS Modes:
• Label Distribution Mode
	○ Downstream-on-Demand
○ Unsolicited Downstream
• Label Retention Mode
	○ Liberal
	○ Conservative
• LSP control mode
	○ Independent
	○ Control

 


LDP & It's Operations:
LDP Functions :
LSR Discovery
Session Establishment & Maintenance
Advertisement of Label Mappings
Housekeeping by means of Notifications
 
LDP Hello :
UDP to discover Neighbors on Port 646 and IP : 224.0.0.2
After discovery, communicate on TCP-646 & LDP-IDs, for session parameters like : Timers, LDP/TDP, VPI/VCI ranges, DLCI ranges
Timers : 5, 15 (Default, but can be changed)
If Timers are miss-matched, lower value will be consider, like BGP
 
LDP-ID
6 Bytes Field
4 Bytes used for identifying the LSR Uniquely
2 Bytes used to identify "Label Space"
If last 2 Bytes == 0, Per-Platform Label Space
If last 2 Bytes !=0, per-interface label space (LC-ATM)
LDP-ID must be reachable to neighbor
 
All-frame relay links will require only 1 LDP session for all links
LC-ATM will require 1 LDP session for 1 Link


LSR's working mode :
Advertisement Mode
Unsolicited Downstream (UD)
Downstream-on-Demand (DoD) advertisement mode
Label Retention Mode
Liberal Label Retention (LLR)
Conservative Label Retention (CLR) mode
LSP Control Mode
Independent LSP Control
Ordered LSP Control mode


Controlling VPNv4 Routes 
Route distinguisher used solely to make route unique 
Allows for overlapping IPv4 addresses between customers 
New BGP extended community "route-target" used to control what enters/exits VRF table 
"export" route-target 
What routes will be go from VRF into BGP 
"import" route-target 
What routes will go from BGP into VRF 
Allows granular control over what sites have what routes 
"import map" and "export map" allow control on a per prefix basis 





Traffic Engineering:
CSPF:
Router "advertises" below things for all interfaces that are involved in MPLS T.E.
Bandwidth
Affinity (Link Coloring) or Affinity Bits or 32-bits Bitmap
Administrative Weight (TE Metric)
Overrides the IGP Metric
A delay-sensitive metric on a per-tunnel basis
Two links A & B, both are of 10Mbps, but delay on A > B. Administrative Weight Can be set to measure delay on the link.
Explicitly Defined Path

 ** CSPF is designed to find the best path till Tunnel's End Point, not to all routers.


When is Information Flooded?
When Link goes UP or DOWN
When link's configuration is changed (For e.g. Cost is modified)
Periodically Flooded
When Link B/W changes "significantly"
Three rules of Flooding
Flood significant changes immediately (Threshold Values : 15, 30, 45, 60, 75, 80, 85, 90, 95, 96, 97, 98, 99, 100)
Flood significant changes periodically, but more often than IGP refresh interval - Default 3 Minutes
Changes that has not yet been flooded, is known to cause an error, flood immediately
For e.g. : When one router is having "stale topology information"
 

  
CSPF Path Option:
Explicit (Manual)
Dynamic
More than one path options can be configured for a tunnel
 
** TE Tunnels are unidirectional
**  OSPF uses Type-10 (area local) Opaque LSAs
** IS-IS uses TLVs (Wide Metric Type)

TE resources for a link are:
TE Head-End router must have "all topology information" and "all constraints or resources" information of the links
TE metric
To construct a TE topology
Maximum bandwidth
Total b/w of link
Maximum reservable bandwidth
B/W available to TE link
Unreserved bandwidth
#2 - #3
Administrative group
32-bit user defined field
 
OSPF Extensions for TE:
Three (3) new LSAs are defined for TE, called Opaque LSA
Type-9
Link scope
Type-10
Area scope
Stopped by ABR
Same Like Type-1 & Type-2
Type-11
OSPF Domain Scope
Same Like Type-5
Don't flow to stub areas
New bit "O-bit" is defined for TE which means router is capable of processing Opaque LSAs, in packets :
Hello
DBD
All LSA's
 
TE's LSA is Type-10 (Opaque LSA) that carries one or more TLVs.
TLVs carries MPLS TE specific data
Router Address TLV
Carries router ID for TE
Link TLV
Carries "set of sub-TLVs"
Describing single link for MPLS TE
 
TE "Link" Attributes:
Maximum reservable bandwidth
Attribute flags
Resources of the link
Capabilities of the link (like encrypted or not)
Administrative Policies
TE metric
IGP cost of the link (Default Behavior)
But can be changed on Headend router
Shared risk link groups (SRLG)
Maximum reservable sub-pool bandwidth
Diff-Serv aware TE tunnel gets bandwidth
Fraction of Global bandwidth
 
TE "Tunnel" Attributes:
Tunnel destination
MPLS TE Router-ID of Tail End Router
Desired bandwidth
B/W requirement of TE tunnel
Affinity
Setup and holding priorities
Re-Optimization
Path options
 
TE Tunnel Path Calculation:
Path setup option
Explicit
On all routers including Tail End
Dynamic
Calculation doe on Head End router
Only need to configure about Destination (Tail End Router on Head End)
Setup and holding priority
Setup-priority : So that they can preempt other Tunnels (Lower is better)
Holding-Priority : So that other don't preempt them (Lower is better)
Attribute flags and affinity bits
Re-optimization
Periodic Re-Optimization  : Every One Hour
Event-Driven Re-Optimization
Manual Re-Optimization
 

  

  


RSVP:
 **** After a path is calculated with CSPF, that path needs to be signaled, that's done with RSVP.  ****
Uses its own Protocol (46)
Can be encapsulated in UDP, but never implemented
USED FOR : To Signal & Resource reservation throughout the network
Three Basics Functions
Path SETUP & MAINTENANCE
Path Teardown
Error Signaling
Soft-State Protocol : Means, periodically needs to refresh reservation
Signaling in TE (Messages Types)
PATH Message
Resv Message
PATH Error Message
Resv Error Message
PATH Tear Message
Resv Tear Message
ResvConf Message (Optional)
ResvTearConf Message (Cisco Proprietary)
Hello : Local Keep alive between 2 direct connected neighbor
   
Shared Explicit (SE) Reservation style:
All RSVP reservations are uniquely identified with :
Sender Address : Headend RID
LSP ID
Endpoint Address : Tail-end RID
Tunnel ID : Source Tunnel Interface No.
Extended Tunnel ID
 

RSVP Signaling Objects:
Object
Message
Function
LABEL_REQUEST
PATH
Used to request a label mapping to the TE tunnel or LSP; generated by the headend router in the PATH message.
LABEL
RESERVATION
Used to allocate labels mapping to the TE tunnel or LSP; generated by the tail-end router in the RESERVATION message and propagated upstream.
EXPLICIT_ROUTE
PATH
Carried in PATH messages and is used to either request or confirm a specific path/route for the tunnel.
RECORD_ROUTE
PATH, RESERVATION
Similar to a record option with ICMP ping. It is added to the PATH or RESERVATION messages to notify the originating node about the actual route/path that the LSP TE tunnel traverses.
SESSION_ATTRIBUTE
PATH
Used to define specific session parameters local to the TE LSP tunnel.



  

  

Objects in PATH Message
Object
Message
SESSION
Defines the source and the destination of the LSP tunnel. Usually identified by IP addresses of corresponding loopback interfaces on headend and tail-end routers.
SESSION_ATTRIBUTE
Defines the characteristics of the specific LSP tunnel, such as the bandwidth requirements and resources that would need to be allocated to the tunnel.
EXPLICIT_ROUTE
Populated by the list of next hops that are either manually specified or calculated using constraint-based SPF. The previous hop (PHOP) is set to the router's outgoing interface address. The Record_Route (RRO) is populated with the same address as well.
RECORD_ROUTE
Populated with the local router's outgoing interface address in the path of the LSP tunnel.
SENDER_TEMPLATE
In addition to the previously mentioned attributes, the sender template object in the path message depicts the interface address that will be used as the LSP-ID for the tunnel. This value is defined by the headend router.



  

  

  

  

 ** You don't run IGP over MPLS TE Tunnel Interfaces :


TE Tunnels are unidirectional and can't receive routing updates
Topology is already learnt with underlying IGP
 
Protection Types:
Path Protection
Local Protection
Link Protection
Node Protection
Link Protection: Four Section
Pre-failure configuration
After enabling FRR, headend routers sets SESSION_ATTRIBUTE flag (Link Protection Desired), Total Flag count will be 3 i.e. Local_Protection_Desired, SE Style, Label Recording (Only in case of Node Protection)
Failure detection
Connectivity restoration
Post-failure signaling
Upstream-Signaling
After Failure :
With No Local Protection : PLR will send PATH_ERR Message to Headend router containing Values 24 & 5 which means : Routing_Problem & No_Route_to_Destination.
With Local Protection : PLR will send PATH_ERR Message to Headend router containing values 25 & 3 which means : Notification & Tunnel_Locally_Repaired.
IGP Notification
Downstream-Signaling
 


