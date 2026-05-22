# OPNSenseVlan-DHCP

# This is more of an advanced lab I have been working on. I have failed a few time but am putting it here for when I am successful.



| WARNING FOR WHEN YOU GET TO THIS POINT |

“If another virtual machine available type in default gateway into browser to finish opnsense config.”
# Setup:
I created a windows server machine with static ips of:

IP:192.168.1.10

Mask:/24

GW:192.168.1.1

DNS: loop back address
-------------------------

1.Install opnsense

Login: root

Password for installaion: installer 

2.Continue with default

3.Select UFS then harddisk

4.Click yes

5.Set your own password> 

6.Reboot> 

devices and remove optical disk

pw: Password1

login:root

Password1

1> no> no> wan em0> lan em1 set ips 2> lan 192.168.1.1 /24 gw blank dhcp no for now> access web gui via 192.168.1.1 our assignment: 

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/59c858455f739c0ced9afc013349179c460df73b/OPNsetuppng)

In GUI welcome> next > dns 1.1.1.1> enable DNSSEC > next > 

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/e8963e680e432d0ad24524e6ac2cce78f1d433e5/GUIsetup1.png)
![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/91b7db3136b107047f659c4f345b3e4e6ceb404b/GUIsetup2.png)

Keep going then reset password for GUI> Password1

1. System > Firmware > Updates check if any > Plugins check for virtualbox one
2. For VLAN: Interfaces, Devices, VLAN, LAN will be the parent and tag is the subnet save and apply. Assignment:

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/183a508f5e5af50f4b3581a278497ae368ba9002/VLANassignments1.png)

1. Assignment of VLANS:

In assignments under '+' sign make sure parent is correct and name like this for correct vlan: 

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/1bc52c02538ba01afb8a23fc44d7704c0f22eaa4/VLANassignment2.png)

1. Enable the VLANS: 

After you add and apply click the vlan it should look like this [VLAN10], enable, static ip, and at bottom assign the static IP. Save and apply. Do for the rest.

1. Next to set up firewall rule and DHCP server.

Firewall, Rules: 

VLAN1 

Staff can reach servers + internet, but not guest or management.

| Rule | Source | Destination | Port | Action | Purpose |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN10 net | LAN20 net | any | Pass | Allow staff to reach servers |
| 2 | LAN10 net | any | any | Pass | Allow internet access |
| 3 | LAN10 net | LAN30 net | any | Block | Block guest VLAN |
| 4 | LAN10 net | LAN40 net | any | Block | Block management VLAN |

VLAN2 Servers can only do DNS, NTP, updates, and internal server‑to‑server communication.

| Rule | Source | Destination | Port | Action | Purpose |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN20 net | LAN20 net | any | Pass | Allow server‑to‑server traffic |
| 2 | LAN20 net | LAN10 net | any | Block | Servers cannot initiate to staff |
| 3 | LAN20 net | LAN30 net | any | Block | Servers cannot talk to guest |
| 4 | LAN20 net | LAN40 net | any | Block | Servers cannot talk to management |
| 5 | LAN20 net | any | 53 (DNS) | Pass | DNS lookups |
| 6 | LAN20 net | any | 123 (NTP) | Pass | Time sync |
| 7 | LAN20 net | any | 443 (HTTPS) | Pass | Windows updates |
| 8 | LAN20 net | any | any | Block | Default deny |

The rules for ports left out some details this is rule 5 looks:

- Also be sure to know your TCP/UDP protocols for their associated ports.
- **Action:** Pass
- **Interface:** LAN20
- **Direction:** In
- **TCP/IP Version:** IPv4
- **Protocol:** TCP/UDP
- **Source:** LAN20 net
- **Destination:** any
- **Destination Port:** 53 (DNS)
- **Description:** Allow DNS from Servers

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/9fa1edb991d205b98f8711174d85a5d6443e7741/FirewallEx.png)

![image alt](https://github.com/Rgonzales92/OPNSenseVlan-DHCP/blob/e5012cdfdb7598dd8a839eb3c86f32573cf2c614/Port.png)

Rule 6

- **Action:** Pass
- **Interface:** LAN20
- **Direction:** In
- **TCP/IP Version:** IPv4
- **Protocol:** UDP
- **Source:** LAN20 net
- **Destination:** any
- **Destination Port:** 123 (NTP)
- **Description:** Allow NTP from Servers

This is for time sync and kerberos

Rule 7

- **Action:** Pass
- **Interface:** LAN20
- **Direction:** In
- **TCP/IP Version:** IPv4
- **Protocol:** TCP
- **Source:** LAN20 net
- **Destination:** any
- **Destination Port:** 443 (HTTPS)
- **Description:** Allow HTTPS from Servers

“Make sure your rules are in the order of the table.”

VLAN3 Guests can only reach internet

| Rule | Source | Destination | Port | Action | Purpose |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN30 net | LAN net | any | Block | Block access to all LANs |
| 2 | LAN30 net | RFC1918 networks | any | Block | Block private networks |
| 3 | LAN30 net | any | any | Pass | Allow internet only |

For RF1918 private networks we had to make an alias in the Aliases section to cover IPs 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16 the type was networks

- **Action:** Block
- **Interface:** LAN30
- **Direction:** In
- **TCP/IP Version:** IPv4
- **Protocol:** Any
- **Source:** LAN30 net
- **Destination:**
    - Click the dropdown
    - Choose **“RFC 1918 networks”**

VLAN4 Admins can reach all but guest

| Rule | Source | Destination | Port | Action | Purpose |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN40 net | any | any | Pass | Full admin access |
| 2 | LAN40 net | LAN30 net | any | Block | Block guest VLAN |

Inter-VLAN the blue print of how VLAN communication rules

| From / To | VLAN10 Staff | VLAN20 Servers | VLAN30 Guest | VLAN40 Mgmt |
| --- | --- | --- | --- | --- |
| **VLAN10 Staff** | — | Allow | Block | Block |
| **VLAN20 Servers** | Block | — | Block | Block |
| **VLAN30 Guest** | Block | Block | — | Block |
| **VLAN40 Mgmt** | Allow | Allow | Block | — |

reboot the router

DHCP:

Kea DHCP4

Interfaces apply all the vlans used, make sure enabled and firewall rules checked

Subnets - “+” 

- VLAN10 - 192.168.10.0/24
    
    Pool: 192.168.10.50 - 192.168.10.200
    
    GW: 192.168.10.1 this is becuase for vlans default gateway will be the first number available in the subnet range.
    
    DNS: 192.168.1.10
    
- VLAN20 - 192.168.20.0/24 lab broke at this point.

Had to restore original Static ip and default gateway.
