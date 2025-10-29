 **"What type of static route is created when the next-hop IP address and exit interface are specified?"**

The correct answer, is the **Fully specified static route**.

---

## 🧐 Understanding Static Route Types

When configuring a static route on a network device like a router, the level of detail provided determines the type of static route created.

- **Fully Specified Static Route:** This type is created when you explicitly specify **both** the **next-hop IP address** and the **exit interface** in the configuration command.
    
    - _Example (Cisco CLI concept):_ `ip route [destination-network] [mask] [next-hop-ip] [exit-interface]`
        
    - This provides the router with the most direct information for forwarding.
        
- **Recursive Static Route:** This route is configured using **only the next-hop IP address**. The router must perform a second lookup (recursion) in its routing table to determine the exit interface to use to reach that next-hop IP address.
    
    - _Example (Cisco CLI concept):_ `ip route [destination-network] [mask] [next-hop-ip]`
        
- **Directly Connected Static Route:** This route is configured using **only the exit interface**. The router assumes the destination network is directly reachable via that interface (e.g., in a Point-to-Point link scenario, or a common configuration in older or simplified systems). In modern Cisco practice, specifying only the exit interface for an Ethernet segment often creates a type of **Directly Connected Static Route**, where the router assumes it can send an ARP request for the destination's MAC address via that interface. However, the term "Fully Specified" is the definitive answer when _both_ the next-hop IP and exit interface are used.
    
- **Floating Static Route:** This is a standard static route configured with a higher **Administrative Distance (AD)** than the primary route (which could be dynamic or another static route). It is only used if the primary route fails, effectively acting as a **backup path**. The route type itself (recursive, directly connected, or fully specified) is determined by the next-hop information provided, regardless of the AD setting.
    

---

**In summary:** Specifying both the **next-hop IP address** and the **exit interface** defines a **Fully specified static route**.



The recursive IPv6 static route statement is:

**`Ipv6 route 2001:db8:cafe:1::/56 2001:db8:1000:10::1`**

---

## 🔁 Recursive Static Route Explanation

A **recursive static route** is defined by specifying the **destination network** and the **next-hop IP address** but **without** specifying the exit interface.

- **Syntax (Conceptual):** `ipv6 route [destination-network/prefix] [next-hop-ip-address]`
    

In the correct statement:

1. **`Ipv6 route`**: The command to define an IPv6 static route.
    
2. **`2001:db8:cafe:1::/56`**: The **destination network** and prefix.
    
3. **`2001:db8:1000:10::1`**: The **next-hop IP address**.
    

The router must now perform a **second lookup (recursion)** in its routing table to find the exit interface it should use to reach the next-hop IP address (`2001:db8:1000:10::1`).

### Comparison with Other Route Types:
![[Pasted image 20251029142630.png]]


A network administrator configuring a route to forward packets to a specific web server should configure A host route.

🖥️ Route Type Explanation
A host route is the most specific type of static route used to target a single device.

Definition: A host route is a route entry that has a /32 mask for IPv4 or a /128 mask for IPv6. This mask uniquely identifies a single IP address (the web server's IP) and ensures that all traffic destined for that specific address uses this explicitly configured path.

Use Case: Since the goal is to forward packets to one specific web server, a host route is the most efficient and precise method to direct traffic to that single destination.

Comparison with Other Route Types
![[Pasted image 20251029142900.png]]

The command that would create a valid **IPv6 default route** is:

**`ipv6 route ::/0 2001:db8:acad:2::a`**

---

## 🗺️ IPv6 Default Route Breakdown

A **default route** serves as the **gateway of last resort** for any packet whose destination network is not explicitly found in the router's routing table.

### 1. The Destination Network

- In IPv6, the special address used to represent **all networks** (the default route) is **`::/0`**.
    
    - The `::` is the shortened form of all zeros (`0000:0000:0000:0000:0000:0000:0000:0000`).
        
    - The `/0` prefix length means zero bits are required to match, making it match _every_ possible IPv6 address.
        
    - The syntax is similar to the IPv4 default route, which is `0.0.0.0 0.0.0.0` or `0.0.0.0/0`.
        

### 2. The Next-Hop Address

- The address immediately following the destination network is the **next-hop IP address**. This is the address of the neighboring router to which unknown traffic should be forwarded.
    
- In the correct command, **`2001:db8:acad:2::a`** is the Global Unicast Address (GUA) of the next-hop router.
![
|**Command**|**Destination Network**|**Next-Hop**|**Result**|
|---|---|---|---|
|`ipv6 route 2001:db8:acad:1::/64 ::1`|`2001:db8:acad:1::/64`|`::1` (Loopback)|Creates a route to a **specific subnet**, not a default route.|

|`ipv6 route ::/0 fe80::1`|`::/0` (Default)|`fe80::1` (Link-Local)|**Invalid** as a standalone recursive route. When using a **Link-Local Address (LLA)** (`fe80::...`) as the next-hop, you **must** also specify the exit interface, otherwise the router doesn't know _which_ local link to send the packet out on.|

|**`ipv6 route ::/0 2001:db8:acad:2::a`**|**`::/0` (Default)**|**`2001:db8:acad:2::a`** (GUA)|**Valid Recursive Default Route.** This tells the router to send all unknown traffic to the next-hop GUA, which the router will then recursively look up to find the exit interface.|

|`ipv6 route ::/128 2001:db8:acad:1::1`|`::/128` (Host)|`2001:db8:acad:1::1`|Creates a route to a **single host** (a host route), not a default route.|


The IPv6 static route that would serve as a **backup route** to a dynamic route learned through OSPF is:

**`Router1(config)# ipv6 route 2001:db8:acad:1::/32 2001:db8:acad:6::2 100`**

---

## 🛡️ The Backup Route Principle

A backup static route, also known as a **Floating Static Route**, is created by manipulating its **Administrative Distance (AD)**.

### 1. The Primary Route (OSPF)

The primary route is learned dynamically through **OSPF**, which has a default Administrative Distance of **$110$**.

### 2. The Backup Route (Static)

For a static route to be installed in the routing table **only if the OSPF route fails**, its Administrative Distance must be set to a value **higher** than $110$.

- **Option 1 (`... 100`):** AD of $100$ is **lower** than OSPF's $110$. This static route would be preferred over the OSPF route and installed as the _primary_ route.
    
- **Option 2 (`... 200`):** AD of $200$ is **higher** than OSPF's $110$. This route will be placed in the routing table **only** if the OSPF route (the primary route) is removed, making it a **backup (floating) route**.
    
- **Option 3 (`... 100`):** AD of $100$ is lower than OSPF's $110$. Same reason as Option 1; it would be the primary route.
    

The correct command is:

`Router1(config)# ipv6 route 2001:db8:acad:1::/32 2001:db8:acad:6::2 200`

This command specifies:

- **Destination Network:** `2001:db8:acad:1::/32`
    
- **Next-Hop Address:** `2001:db8:acad:6::2`
    
- **Administrative Distance (AD):** `200` (which is higher than the OSPF AD of $110$).
    
