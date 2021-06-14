# Multicast Reflection Instructions

## Why do we need to do Multicast Service Reflection

 - From Cisco's implementation guide 
   - Allows users to translate externally received multicast or unicast destination addresses to multicast or
unicast addresses that conform to their company’s internal addressing policy. This allows the seperation
of the private addressing scheme used by the content provider from the public addressing used by the
service provider. The following types of translations are supported:
    - Multicast-to-Multicast Destination Translation
    - Multicast-to-Unicast Destination Translation
    - Multicast-to-Multicast Destination Splitting
    - Multicast-to-Unicast Destination Splitting
    - Provides logical separation between private and public multicast networks.
    - Provides the flexibility to forward multicast packets--translated or untranslated--out the same outgoing
    interface.
    - Provides redundancy by allowing users to get identical feeds from two ingress points in the network and
    route them independently.
    - Allows users to use the subnet of their choice to be the source network and scope it appropriately.
    - 

## Instructions 

 - Given a multicast source from Multicast Source 1 and Multicast Source 2, along with a configured RP at Green RP
 - A cross connect that only allows for transmission from green to blue
 - Reflect the multicast streams into the blue network
  

## Results

### From Green Network

Green#show ip mroute

(*, 226.1.1.1), 00:01:27/stopped, RP 1.1.1.1, flags: SJC
  Incoming interface: Ethernet0/3, RPF nbr 100.200.200.1
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse, 00:01:25/00:02:26

(100.100.100.2, 226.1.1.1), 00:00:30/00:02:29, flags: JT
  Incoming interface: Ethernet0/3, RPF nbr 100.200.200.1
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse, 00:00:30/00:02:29

(*, 225.1.1.1), 00:01:27/stopped, RP 1.1.1.1, flags: SJC
  Incoming interface: Ethernet0/3, RPF nbr 100.200.200.1
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse, 00:01:25/00:02:26

(100.100.100.1, 225.1.1.1), 00:00:32/00:02:27, flags: JT
  Incoming interface: Ethernet0/3, RPF nbr 100.200.200.1
  Outgoing interface list:
    Ethernet0/1, Forward/Sparse, 00:00:32/00:02:27

### From Blue Network

Blue#show ip mroute

(*, 236.1.1.1), 00:02:36/stopped, RP 3.3.3.3, flags: SJPC
  Incoming interface: Ethernet0/2, RPF nbr 10.20.20.1
  Outgoing interface list: Null

(10.10.10.2, 236.1.1.1), 00:01:33/00:01:26, flags: PTX
  Incoming interface: Ethernet0/2, RPF nbr 10.20.20.1
  Outgoing interface list: Null

(*, 235.1.1.1), 00:02:36/stopped, RP 3.3.3.3, flags: SJPC
  Incoming interface: Ethernet0/2, RPF nbr 10.20.20.1
  Outgoing interface list: Null

(10.10.10.2, 235.1.1.1), 00:01:35/00:01:24, flags: PTX
  Incoming interface: Ethernet0/2, RPF nbr 10.20.20.1
  Outgoing interface list: Null

