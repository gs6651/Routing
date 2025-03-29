
### Structure of Policy
 - List: Which Site (Branch 1, Branch 2, HUB etc.)
 - Definition: What Action
 - Application: Where to apply policy. (in-bound or out-bound)

### Centralized policies
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
