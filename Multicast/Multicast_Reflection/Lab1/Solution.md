# Solution

1. Ensure Multicast Routing is enabled on all devices
   1. ip multicast-routing
   2. ip pim auto-rp listener
   
2. One the Green and Blue Router Configure a non-routable IP and ACLs to only allow specific traffic through
### Green
```
ip access-list standard deny
 deny   any log
!
ip access-list extended mcast
 permit ip any 225.1.1.0 0.0.0.255
 permit ip any 226.1.1.0 0.0.0.255
 deny   ip any any log
!
interface Ethernet0/1
 no shutdown
 description x-connect to blue
 ip address 192.168.1.1 255.255.255.248
 ip access-group deny in
 ip access-group mcast out
 ip pim sparse-mode
```


### Blue_Edge
ip access-list standard deny
 deny   any log
!
ip access-list extended mcast
 permit ip any 225.1.1.0 0.0.0.255
 permit ip any 226.1.1.0 0.0.0.255
 deny   ip any any log
!
interface Ethernet0/1
 no shutdown
 ip address 192.168.1.2 255.255.255.248
 ip access-group mcast in
 ip access-group deny out
 ip pim sparse-mode

3. Create Class Maps to identify IGMP traffic
   
### Green

class-map type multicast-flows One_way_multicast
 group 225.1.1.1
 group 226.1.1.1

### Blue_Edge

class-map type multicast-flows One_way_multicast
 group 225.1.1.1
 group 226.1.1.1

4. Join those IGMP groups on the exit interface of the Green Router

### Green

interface Ethernet0/1
 ip igmp static-group class-map One_way_multicast

5. At this point you should be done with the Green network, now you need to reflect the multicast being pushed into the Blue edge into a group address that is native to that network

6. First this requires an operational RP on the Blue network, create loopbacks on Blue_Edge that are routable in that network

### Blue_Edge

interface Loopback0
 no shutdown
 ip address 3.3.3.3 255.255.255.255
 ip pim sparse-mode
!
interface Loopback10
 no shutdown
 description pim-bidir rp
 ip address 33.33.33.33 255.255.255.255
 ip pim sparse-mode

7. Create ACLS and RP Statements for these loopbacks

ip pim bidir-enable
ip pim rp-address 33.33.33.33 RP override bidir
ip pim rp-address 3.3.3.3 RP_Reflext override
ip pim send-rp-announce Loopback0 scope 5 group-list RP_Reflext
ip pim send-rp-discovery Loopback0 scope 5
!
ip access-list standard RP
 permit 225.1.1.0 0.0.0.255
 permit 226.1.1.0 0.0.0.255
ip access-list standard RP_Reflext
 permit 235.1.1.0 0.0.0.255
 permit 236.1.1.0 0.0.0.255

8. Why do we use bidirectional PIM?  No RPF check, and will act as the RP for the traffic that we do not have a route to.

9. Now we need to create class maps to identify our Group addresses on the blue network, remember reflection allows us to change groups we cant route in our network into groups that we can

class-map type multicast-flows One_way_multicast
 group 225.1.1.1
 group 226.1.1.1
!
class-map type multicast-flows One_way_multicast_reflect
 group 235.1.1.1
 group 236.1.1.1

10. Now we have to create a virtual interface with a routable IP that we can use as our source for the new reflected traffic

interface Vif1
 no shutdown
 ip address 10.10.10.1 255.255.255.252
 ip service reflect Ethernet0/1 destination 226.1.1.0 to 236.1.1.0 mask-len 24 source 10.10.10.2
 ip service reflect Ethernet0/1 destination 225.1.1.0 to 235.1.1.0 mask-len 24 source 10.10.10.2
 ip pim sparse-mode
 ip igmp static-group class-map One_way_multicast

11. Finally we have to force our reflected multicast down into the Blue network

interface Ethernet0/2
 no shutdown
 ip address 10.20.20.1 255.255.255.0
 ip pim sparse-mode
 ip igmp static-group class-map One_way_multicast_reflect

12. Within the blue network we need to apply the same static join to ensure the traffic continues to be routed with the network

class-map type multicast-flows One_way_multicast_reflect
 group 235.1.1.1
 group 236.1.1.1
!
interface Ethernet0/2
 ip igmp static-group class-map One_way_multicast_reflect
