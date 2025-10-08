Module-11.0

The bits within the network portion of the address must be identical for all devices that reside in the same network. The bits within the host portion of the address must be unique to identify a specific host within a network.


The actual process used to identify the network portion and host portion is called ANDing.
*Note: In digital logic, 1 represents True and 0 represents False. When using an AND operation, both input values must be True (1) for the result to be True (1).*



The prefix length is the number of bits set to 1 in the subnet mask. It is written in “slash notation”, which is noted by a forward slash (/) followed by the number of bits set to 1. Therefore, count the number of bits in the subnet mask and prepend it with a slash.


![[Pasted image 20251003114624.png]]

11.1.8 Check Your Understanding - IPv4
1. The network address for 10.5.4.100 with a subnet mask of 255.255.255.0 is 10.5.4.0.
2. The network address for 172.16.4.100 with a subnet mask of 255.255.0.0 is 172.16.0.0.
3. Host A is on network 10.5.4.0. Therefore, devices with the IPv4 addresses 10.5.4.1 and 10.5.4.99 are on the same network.
4. Host A is on network 172.16.0.0. Therefore, devices with the IPv4 addresses 172.16.4.99 and 172.16.0.1 are on the same network.
5. Host A is on network 192.168.1.0. Therefore, devices with the IPv4 addresses 192.168.1.1 and 192.168.1.100 are on the same network.