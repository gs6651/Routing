`BGP is NOT a routing protocol, but an application. Used to exchange NLRI.`

IPv4 NLRI contains:
• Prefix/Length
	• Attributes
		○ Local Preference
		○ AS-Path
		○ Etc.
	• Next Hop
BGP Message Format:
19 bytes -to- 4096 bytes
19 bytes = Header
16 bytes = Marker
To authenticate incoming message
OR
To detect loss of synchronization b/w peers
2 bytes = Length
Indicates total length of BGP message
1 byte = Type
OPEN
NOTIFICATION
UPDATE
KEEP-ALIVE



Messages Types:
OPEN
Version No. (3 or 4)
AS No. (contains Local AS no.)
Hold Time
BGP Identifier (BGP Router-ID)
Optional Parameters Length
1 = YES (Carries Optional Parameters)
0 = NO 
Optional Parameters
Authentication
Route-Refresh (C = Capability)
Enhanced Route-Refresh
Multi-protocol support extension (C)
Outbound Route Filtering (ORF)
Multi-route to a destination
Extended Next Hop
Graceful Restart (C)
4-Byte Octet AS Number (C)
Multi Session BGP (IPv4 & IPv6 with same neighbor)
ADD-PATH
Format



KEEP-ALIVE
If router accepts parameters send in OPEN message
Contains only header, NO data
NOTIFICATION
Only when something bad happens
Format:
2+ Bytes
Error Code (1 Byte)
Message Header Error
Connection not synchronized
Bad Message Length
Bad message type
Open Message Error
Unsupported version number
Bad peer AS number or IP address
Bad BGP identifier
Unsupported optional parameter
Authentication Failure
Unacceptable hold time
Update Message Error
Attribute list length error : I.e. too long or too short
Invalid network field, I.e. prefix included in the field is invalid
Invalid origin code
Missing well-known error
Invalid next-hop attribute
Malformed attribute list
Hold Time Expired
FSM Error
Unexpected event
Cease
Any fatal error
Error Subcode (1 byte)
More specific information about the nature of reported error
Error Data (Variable)
Databases on error code or Error subcode, used for diagnose the reason
UPDATE
NLRI
Path Attributes
MTU
DF bit is set in update message
RD & RT (when MP-BGP)
Feasible Routes 'OR' Withdrawn Routes
Format:



Path Selection
Mnemonic = We Love Oranges, AS Oranges Mean Pure Pulp

Valid Next Hop (Ignore Routes with incomplete information of Next-hop)
W - Weight // Highest // 32768 // Incoming Update // Local to router
L - Local Preference // Highest // 100 // Incoming Update
O - Origin-Type -- Locally Originated (Network > redistribute > aggregate > not locally originated)
AS - shortest AS-path // Outgoing Update (AS SET & CONFED does not count here)
O - Origin Code  //  i > e > ?
M - Lowest MED // Outgoing Update (Only for same AS, until deterministic & always compare MED are configured)
P - Paths // eBGP > iBGP
P - Multi-Path -- Prefer the Path with LOWEST IGP metric to BGP next-hop (If it's equal, can use Multi-Path)
# bgp bestpath as-path multipath-relax
 
Tie Breakers :
Oldest Route
Lowest Router-ID
Shortest Cluster-List
Lowest neighbor IP
 
Bla.. Bla.. Bla..


BGP States:
Finite State Machine (FSM)
Idle
BGP Starts in this state
All incoming connections will be ignored
Start Event (IE1) : Configuring or Resetting BGP will trigger IE1
Initialize ConnectRetry timer
Initialize TCP session to neighbor
Listens TCP initialization from neighbor
Changes state to connect
After first fail to reset BGP Process initiates ConnectRetry timers (60s), Double next time and will keep on increasing 
Connect:
Waits for TCP session to finish
If TCP connection is Successful :
Clear ConnectRetry timer
Completes initialization
Sends an Open Message to neighbor and transition to OpenSent state
If TCP connection is Unsuccessful : (TCP RST, etc.)
Continue listening for any incoming TCP connection from neighbor
Resets ConnectRetry
Transition to Active State
If TCP connection taking too long to respond or for any other reason it's getting delayed, then :
ConnectRetry Timer will expires in Connect state only
Timer will be reset
Another attempt will be made to establish TCP Connection and Process stays in Connect State
 
*Any other Input Event (IE) will transition to idle state.
 
Active
Actively try to establish TCP session (initiated by local router)
If TCP connection is successful
Clears ConnectRetry timer
Completes initialization
Sends an Open Message to neighbor and transition to OpenSent state
If TCP connection is Unsuccessful
Means ConnectRetry timers expires in Active State, the process transition back to Connect State
Resets ConnectRetry timer
Initiates & Listens TCP connection
*Any other input Event (IE), except START, triggers the idle state
Open Sent
Hold timer negotiation
Decision of eBGP or iBGP
Open Confirm
If Keepalive is received, state will transit to Established
If Notification or TCP Disconnect received, state will transit to IDLE
Established
Peers are ready to exchange updates, keepalives or notifications
 


BGP Attributes:
Well-Known Mandatory
Well-Known Discretionary
Optional Transitive
Optional Non-Transitive
Must be recognized by all BGP routers

Must be included in all  updates messages
Must be recognized by all BGP routers

May or May-not carry in  updates messages
BGP process should accept the update in which it is included and pass it to its peers, whether supported or not
 
Origin (i, e, ?)
Local Preference
Aggregator: It tells where the aggregation was done, by including the RID & AS number.
 
When Aggregation happens, below will still tell about ASes from where updates have been through, so that loop can be avoided.
AS_SEQUENCE:  Ordered list, will contain all "common" ASes
AS_SET: Unordered list, will contain all "uncommon" ASes
MED:
BGP's Metric, lower is better
Can be advertised to routers that are one AS away, not beyond that.
Deterministic-MED: It groups the same next AS routes in one group, so that MED can be compared.
Always-Compare-MED: So that MED can be compared even if the AS-Path list is different.
AS-PATH
Atomic-Aggregate: Loss of path-information has occurred.
Community: 32-bits (4 Octets)
NONE: No Community
NO_EXPORT: Not outside of the AS
NO_ADERTISE: Not to ANY peer
NO_SUBCONFED:
Originator-ID: It's the router-ID of the originator of the route.
Next-Hop
 
Extended Communities: 64-bits (8 Octets)
Route-Target
Route-Origin
Link Bandwidth
Opaque : Allows Opaque data to be carried in BGP extended communities
 
Cluster-ID: It's the router-ID of the route-reflector.




Misc.
BGP supports graceful neighbor shutdown 
Updates are incremental only (Means whenever there is change in network)
Withdrawn routes are part of UPDATE message
Updates include NLRI and Attributes, but Withdrawn includes NLRI only.
There may be multiple routes in NLRI Field, but each update message describes only a single BGP route. (Reason: PATH Attribute describe only single path, but that PATH Attribute may lead to multiple destination)
If route information has been changed, then there is no withdrawn message required, only advertisement of route replacement will be enough.
The KEEP-ALIVE message contains only a header, but no data.
Note that the negotiation done for the Version Number (by actually resetting the session until both nodes agree on a common Version) AND the one for the Hold Timer (use the minimum value of the two BGP speakers) are very different. In both cases, only the OPEN message is sent by each router. However, if the values don't match (in the case of Hold Timer), the session is not reset.
Much more extensible via AFI/SAFI
IPv4 Unicast (By Default)
IPv6 Unicast, IPv4 & IPv6 Multicast, IPv4 & IPv6 MPLS, MDT, VPLS etc.
After successful process of a prefix, below BGP Path Attributes (PAs) are established:
Connected Network
Next-hop BGP attribute – 0.0.0.0
BGP origin attribute – IGP (i)
BGP weight – 32,768.
Static Route/Routing Protocol
Next-hop BGP attribute – next-hop IP address in the RIB
BGP origin attribute – IGP (i)
BGP weight – 32,768
Multi Exit Discriminator (MED) – IGP metric

![image](https://github.com/user-attachments/assets/d09865a2-4963-4a2c-a71c-987633616e8d)

