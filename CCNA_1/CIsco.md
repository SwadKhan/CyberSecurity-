same devices- cropper crossover

Different devices-cropper straight-through

Network :

Intermediate devices- Router, switches, firewall appliance

![[Screenshot 2025-09-27 222339.png]]

NIC= Network Interface Card (usually physically connects to the end devices)
physical port= media connect to the end devices.
Interface=specialized port on a networking device that connect to individual networks.

**The terms port and interface are often used interchangeably.

Topology diagram 
-physical topology diagram-(one can see that the rooms in which these devices are located are labelled in this physical topology.)
-logical topology diagram-(one can see which end devices are connected to which intermediary devices and what media is being used.)

*1. A NIC is a specialized port on a networking device that connects to individual networks.
2. An interface physically connects the end device to the network.
3. The logical topology lets you see which end devices are connected to which intermediary devices and what media is being used.
4. The physical topology lets you see the actual location of intermediary devices and cable installation.*


LAN:1.LANs interconnect end devices in a limited area such as a home, school, office building, or campus.
2.A LAN is usually administered by a single organization or individual. Administrative control is enforced at the network level and governs the security and access control policies.
3.LANs provide high-speed bandwidth to internal end devices and intermediary devices, as shown in the figure.

WAN:1)WANs interconnect LANs over wide geographical areas such as between cities, states, provinces, countries, or continents.
2)WANs are usually administered by multiple service providers.
3)WANs typically provide slower speed links between LANs.

Internet Engineering Task Force (IETF),
Internet Corporation for Assigned Names and Numbers (ICANN), and 
the Internet Architecture Board (IAB)


Intranet is a term often used to refer to a private connection of LANs and WANs that belongs to an organization. An intranet is designed to be accessible only by the organization's members, employees, or others with authorization.


*1. A LAN provides access to users and end devices in a small geographical area.
2.An extranet provides secure and safe access to individuals who work for a different organization but require access to the organization’s data.
3.A WAN provides access to other networks over a large geographical area.*


*there are four basic characteristics that network architects must address to meet user expectations:

- Fault Tolerance
- Scalability
- Quality of Service (QoS)
- Security


**Redundant connections allow for alternative paths if a device or a link fails. The user experience is unaffected.**

**As data, voice, and video content continue to converge onto the same network, QoS becomes a primary mechanism for managing congestion and ensuring reliable delivery of content to all users.**

Congestion occurs when the demand for bandwidth exceeds the amount available. Network bandwidth is measured in the number of bits that can be transmitted in a single second, or bits per second (bps).

*1. Scalability happens when designers follow accepted standards and protocols.  
2.Confidentiality, integrity, and availability are requirements of security.  
3.QoS means that a router will manage the flow of data and voice traffic, giving priority to voice communications.  
4.Redundancy is an example a fault-tolerant network architecture*




1.7.6 Cloud Computing

#Cloud computing is one of the ways that we access and store data.
For security, reliability, and fault tolerance, cloud providers often store data in distributed data centers. Instead of storing all the data of a person or an organization in one data center, it is stored in multiple data centers in different locations.

There are four primary types of clouds: Public clouds, Private clouds, Hybrid clouds, and Community clouds,

A Wireless Internet Service Provider (WISP) is an ISP that connects subscribers to a designated access point or hot spot using similar wireless technologies found in home wireless local area networks (WLANs). From the perspective of the home user, the setup is not much different than DSL or cable service. The main difference is that the connection from the home to the ISP is wireless instead of a physical cable.


**1. Video communications is a good conferencing tool to use with others who are located elsewhere in your city, or even in another country.  
2.BYOD feature describes using personal tools to access information and communicate across a business or campus network.  
3.Cloud computing contains options such as Public, Private, Custom and Hybrid.  
4.Powerline is being used when connecting a device to the network using an electrical outlet.  
5.Wireless broadband uses the same cellular technology as a smartphone.**


1.8.3 Check Your Understanding - Network Security
Correct

You have successfully identified the correct answers.

1. A DoS attack slows down or crashes equipment and programs.
2. A VPN creates a secure connection for remote workers.
3. A firewall blocks unauthorized access to your network.
4. A zero-day or zero-hour attack occurs on the first day that a vulnerability becomes known.
5. A virus, worm, or Trojan horse is malicious code running on user devices.

Module 2(Assessment)-

2.1.6 Check Your Understanding - Cisco IOS Access
1. Because a new switch would not have any initial configurations, it could only be configured through the console port.
2. Connecting a computer to a Cisco device through the console port requires a special console cable.
3. Both Telnet and SSH are in-band access methods that require an active network connection to the device.
4. The AUX port on a Cisco device provided out-of-band connections over a telephone line.

2.2.8 Check Your Understanding - IOS Navigation
1. The privileged EXEC mode allows access to all commands. Higher level commands like global configuration mode and sub configuration modes can only be reached from the privileged EXEC mode.  
2. Global configuration mode is identified by the **(config)#** prompt.  
3. The **>** prompt after the device name identifies user EXEC mode.  
4. To return from any prompt, all the way down to privileged EXEC mode, type the **end** command or by pressing the **CTRL+Z** keys simultaneously on the keyboard.

2.4.8 Check Your Understanding - Basic Device Configuration
1. The global configuration command to set the host name on a Cisco device is hostname. So, in this example the full command is Switch(config)# **hostname Sw-Floor-2**.  
2. Securing access to the EXEC mode on a Cisco switch is accomplished with the enable secret command followed by the password. In this example the command is Switch(config)# **enable secret class.**  
3. User EXEC mode access through the console port is enabled with the login command entered in line mode. For example: Switch(config-line)# **login**.  
4. The **service password-encryption** command entered in global configuration mode will encrypt all plaintext passwords.  
5. The command to set a banner stating "Keep out" that will be displayed when connection to a Cisco switch is Switch(config)# **banner motd $ Keep out $**


3.2.4 Check Your Understanding - Protocols
1. BGP and OSPF are routing protocols. They enable routers to exchange route information to reach remote networks.
2. Service discovery protocols, such as DNS and DHCP enable automatic detection of service. DHCP is used to discover services for automatic IP address allocation and DNS for name-to-IP address resolution services.
3. Sequencing uniquely identifies or labels each transmitted segment with a sequence number that is used by the receiver to reassemble the segments in the proper order.
4. Transmission Control Protocol (TCP) manages the conversation between end devices and guarantees the reliable delivery of information.


![[Pasted image 20251010171037.png]]
