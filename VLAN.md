# VLAN

# VIRTUAL LOCAL AREA NETWORK
VLAN is the technology where we can connect a number of virtual LANs in order to make a LAN ecosystem.

The concept was suggested in order to balance the incoming traffic as well as access to easier management of the networks. We can also use VLANs for implementing WiFi guest networks. VLAN provides data-link connectivity for a subnet.

VLAN also allows us to break broadcast domains.


[![ijgm-07142017-port-to-vlan-interface-settings-step00.png](https://i.postimg.cc/HW03YV4S/ijgm-07142017-port-to-vlan-interface-settings-step00.png)](https://postimg.cc/G92vz3fY)
# Types of VLAN
## Default VLAN
At the initial boot of the switch, all switch ports become a member of the default VLAN.

## Data/User VLAN
Carries only user-generated traffic.

## Native VLAN
Assigned to a dot1q trunk port(more on this later) which supports traffic coming from other VLANs and non-VLANs.

Native VLAN also is used to transfer frames without the dot1q tags. So if we transfer frames over trunks without tagging them, the frames are sent over Native VLAN which is generally VLAN1.

# Management VLAN

The only way to access the management capabilities of the switch.

## Access and Trunk Links

An ethernet switch has two types of links (ports) depending on what we want to implement. Access link is the more common link since it connects the end hosts to the switch but can only have one VLAN network. In a flat network with one VLAN and a single switch, this does the job for us but once we need multiple VLANs, we need ways for VLANs to communicate. To avoid this issue we can use other VLAN networks with switches and connect the switches using a trunk link. A trunk link is essentially a cable which connects switches and supports multiple VLAN networks.

Recent implementations of switches let us have more than one VLAN in a single switch, preventing the need for a trunk link between switches.

VLANs operate in the data-link layer (layer 2) of the OSI model, the layer 2 ethernet switches provide creations of multiple VLANs (up to 4096 if IEEE.802.1Q protocol is used.) but are unable to provide communication between different VLANs. To solve this we need ways to forward traffic from one VLAN to another. Since routing isn’t a thing in the layer 2, we need layer 3 devices so we can forward packets.
# InterVLAN Routing

In order for the swtiches to communicate, a layer 3 device which has routing capabilities is needed for different VLANs to communicate. This process is called InterVLAN routing.
## Solution 1(A router with 2 ethernet interfaces)

A router simply operates in the layer 3 of the OSI model and finds the fastest route for the packet to be delivered. What we do in this solution is connect both switches to the router and we give IP adresses and subnetmasks to the two interfaces of the router so both switches can reach the router and let router forward the traffic.

[![070214-1824-Inter-VLANRo1.jpg](https://i.postimg.cc/8CBB1YL7/070214-1824-Inter-VLANRo1.jpg)](https://postimg.cc/Y40mRbn7)

This is  a solid solution but it scales poorly. When we have multiple VLANs and switcports we need to manually enable routing on each VLAN.
## Solution 2(A router with one ethernet  interface)

With this, trunk link is created between router and the switch using the same kind of encapsulation switches use (ISL or IEEE.802.1Q). Here we create subinterfaces under one interface each terminating their own VLAN. Here subinterfaces act as virtual trunk links for carrying traffic from multiple VLANs and must add VLAN tags after leaving the source interface. Also known as Router on a Stick but not frequently used since this is an expensive alternative.


## Solution 3(Cisco layer 3 switches)
Most used solution here we can use a router or a layer 3 switch created by Cisco which can handle both routing and multiple VLANs and is relatively lower cost.

# Ethernet II frame

An ethernet frame consists of 64 bytes composed of the following headers:

Destination MAC | Source MAC | Ether Type | Data | FCS

Here FCS (Frame Check Sequence) or CRC (Cyclic redundancy check) checks the checksum and depending on the checksum, it either accepts the frame or discards it.

Data consists of upper layer TCP/IP headers and the user data.

When we operate in a single VLAN this frame is used to identify the source and destination addresses. Once the packet is sent to an other VLAN on another switch, it goes through the trunk link. To identify which VLAN the frame belongs to, Cisco developed VLAN tagging which is inserted into the frame in different protocols. Once VLAN header is added the FCS is altered and new FCS must be calculated for checksum.

What this essentially does is it puts a header into the frame to identify the VLAN portion. Two well known protocols for VLAN tagging are ISL(Inter Switch Link) and IEEE.802.1Q(dot1q) trunk link protocols. After the header passes through the trunk link the header is dropped so the receiving end does not receive information about the sender VLAN.
# IEEE.802.1Q

The ethernet II frame is 64 bytes of data. Dot1q inserts its header which is 4 bytes, making the overall frame 68 bytes.

TPID | Priority | CFI | VLAN ID

Priority + CFI + VLAN ID= TCI (Tag Control Info)

TPID(16 bits) is used to identify that the frame is a dot1q tagged frame.

Priority(3 bits), used for prioritisation of data. This means that the specified bandwidth is allocated for these critical services to pass them through the link without any delay.

CFI(1 bit) is mainly used because of compatibility between token ring (which waits for a token in order to transmit data) and ethernet networks (has CSMA/CD which means it listens to the cable and if there hasn’t been traffic for a while it permits itself to transfer data.). If set to one means MAC address is in NON-canonical format means the order of bits is reversed.

VLAN ID(12 bits) is the most important part since it holds information about which VLAN the frame belongs to.

# ISL(Inter Switch Link)

Cisco’s way of VLAN tagging. Instead of putting a header into the frame, ISL puts two headers both to the start and end of the frame with a total of 30 bytes.

ISL(26 bytes) | Ethernet II frame | ISL FCS(4 bytes)

Again they hold data about where the frame belongs to.

After seeing the difference of size in bytes the headers have, one might think that dot1q is the go-to protocol solely because it’s so much smaller compared to the size of ISL. While it is true that dot1q is widely more common, it is not because of its size but because the engineers believe that dot1q approach is safer, ensuring maximum compatibility. Nowadays even Cisco prefers using dot1q.
