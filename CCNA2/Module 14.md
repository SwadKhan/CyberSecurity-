Ethernet switches are used to connect end devices and other intermediary devices, such as other Ethernet switches, to the same network.


Router have multiple interfaces that each belong to a different IP network.

The primary function of a router are to determine the best path to forward packets based on the information in its routing table.


Routing best path is also called as longest match. for this the destination ip address of a packet and a route in the routing table , a minimum number of far-left bits must match.
The route with the greatest number if equivalent far-left bits, or the longest match, is always the preferred route.

*For any of these routes to be considered a match there must be at least the number of matching bits indicated by the subnet musk of the route.*

Directly connected network-that are configured on the active interface of a router


Remote networks- not directly connected to the network.
static routes- manually configured
dynamic routing protocols- EIGRP, OSPF


Default Route- when the routing table does not contain a specific route that matches the dynamic IP address.

ipv4 default route entry of 0.0.0.0/0 and 
Ipv6 - ::/0.

It is sometimes referred as a getaway of last resort.

![[Pasted image 20251022195355.png]]

