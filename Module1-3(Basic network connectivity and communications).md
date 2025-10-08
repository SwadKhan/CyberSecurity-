
Peer to peer- In small businesses and homes, many computers function as the servers and clients on the network. This type of network is called a peer-to-peer network.

The advantages of peer-to-peer networking:
Easy to set up
Less complex
Lower cost because network devices and dedicated servers may not be required
Can be used for simple tasks such as transferring files and sharing printers

The disadvantages of peer-to-peer networking:
No centralized administration
Not as secure
Not scalable
All devices may act as both clients and servers which can slow their performance


*An Ethernet hub is also known as a multiport repeater. Repeaters regenerate and retransmit communication signals. Notice that all intermediary devices perform the function of a repeater.*

- **Interface** - Specialized ports on a networking device that connect to individual networks. Because routers connect networks, the ports on a router are referred to as network interfaces.

**The operating system on home routers is usually called firmware. The most common method for configuring a home router is by using a web browser-based GUI.

![[Pasted image 20251008105659.png]]


 Similar to a console connection, the AUX port is out-of-band and does not require networking services to be configured or available.

Line Configuration Mode - Used to configure console, SSH, Telnet, or AUX access.
Interface Configuration Mode - Used to configure a switch port or router network interface


**To move from any sub configuration mode of the global configuration mode to the mode one step above it in the hierarchy of modes, enter the exit command.

**To move from any subconfiguration mode to the privileged EXEC mode, enter the end command or enter the key combination Ctrl+Z.


From global configuration mode, enter the command hostname followed by the name of the switch and press Enter. Notice the change in the command prompt name.

**Note: To return the switch to the default prompt, use the no hostname global config command.

**If power to the device is lost, or if the device is restarted, all configuration changes will be lost unless they have been saved. To save changes made to the running configuration to the startup configuration file, use the **copy running-config startup-config** privileged EXEC mode command.

*The downside to using the reload command to remove an unsaved running config is the brief amount of time the device will be offline, causing network downtime.

The startup config is removed by using the **erase startup-config** privileged EXEC mode command. After the command is issued, the switch will prompt you for confirmation. Press **Enter** to accept.

2.6.3 Check Your Understanding - Ports and Addresses

1. IPv4 addresses are written in dotted-decimal format. For example: 192.168.1.1.
2. IPv4 addresses are written as four groups of decimal numbers separated by periods. For example: 192.168.1.1.
3. Switch virtual interfaces (SVIs) are virtual and have no physical port. Layer 2 switches use SVIs for remote management.

3.1.12 Check Your Understanding - The Rules.
1. One of the first steps to sending a message is encoding. During the encoding process, information is converted from its original form into an acceptable form for transmission.
2. Messages sent over a computer network must be in the correct format for them to be delivered and processed. Part of the formatting process is properly identifying the source of the message and its destination.
3. Flow control is the managing of the rate of transmission. Response timeout is how long to wait for responses. Access methods determine when someone can send a message. These are the three components of message timing.
4. Multicast messages are addressed for transmission to one or more end devices on a network. Broadcast messages are addressed for transmission to all devices on the network. Unicast messages are addressed for transmission to one device on the network.
