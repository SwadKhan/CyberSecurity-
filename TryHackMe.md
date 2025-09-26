Network Fundamentals:
Intro to Lan(Local area network):

Topology ->the look of the network at hand.

Topology -> The main premise of a star topology is that devices are individually connected via a central networking device such as a switch or hub.

#Adv.
 -scalability, reliability, 
 #dis
   -Cost, Maintenance, Prone to failure albeit reduced, fall of central devices can hamper the whole network, though they are very robust.
   
Bus topology- This type of connection relies upon a single connection which is known as a backbone cable. This type of topology is similar to the leaf off of a tree in the sense that devices (leaves) stem from where the branches are on this cable.
#adv-
cost efficient, easier to install
 #dis-
single point of failure, difficult troubleshooting, slow.

Ring Topology- Also known as token topology.
*Interestingly, a device will only send received data from another device in this topology if it does not have any to send itself. If the device happens to have data to send, it will send its own data first before sending data from another device.*
 #Adv 
 Easy troubleshooting, easy to install, less dependence to the dedicated hardware.
 #Dis
 time consuming, still cut cable can cause hamper to the network.

#Switches- These are dedicated devices with in a network that are designed to aggregate multiple other devices such as computers, printers or any other networking capable devices.

Switches connect devices using the ports and more efficient than the hubs and repeaters.

*It keep track of its connecting devices and then rather than sending the packets to all its devices like the hub it just sends to the targeted one directly. It can also connect with the router , which actually increases the redundancy(reliability).

 #Router
 It's a router's job to connect networks and pass the data between them. Routing involves creating a path between networks so that this data can be successfully delivered. 



# Ip/Subnet Mask:
Ip address - actually of 4 octet and (32 bit) same goes for subnet mask(ranging from 0-255)

Network address -This address identifies the start of the actual network and is used to identify a network's existence.

Host address: An IP address here is used to identify a device on the subnet/network.

Default gateway- The default gateway address is a special address assigned to a device on the network that is capable of sending information to another network.
*Any device needs to go the other network can use any host address but either use the first or last host address of that current network to get into the other network.*

Subnetting Provides a range of benefits - efficiency 
                                -security
                                 -Full control



#ARP Address Resolution Protocol
It is a technology that is responsible for allowing devices to identify themselves on a network.

ARP allows a device to associate its mac address with an IP address  on the network. Each device on a network will keep a log of the MAC Addresses associated with other devices. 

MAC Address-Physical address   IP  Address -Logical Address.

How ARP Works-
ARP sends two types of messages:
-ARP Request
-ARP Reply

*When an ARP request is sent, a message is broadcasted on the network to other devices asking, "What is the mac address that owns this IP address?" When the other devices receive that message, they will only respond if they own that IP address and will send an ARP reply with its MAC address. The requesting device can now remember this mapping and store it in its ARP cache for future use.*


==IP Address

It can be assigned manually or physically using DHCP(Dynamic Host Configuration Protocol)

![[Screenshot 2025-09-26 100601.png]]

