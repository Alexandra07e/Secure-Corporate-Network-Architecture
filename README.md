# Secure Corporate Network Architecture

A complete corporate network built and simulated in **Cisco Packet Tracer** as the final project for the
**SCRC** laboratory (Faculty of Automation and Computer Engineering, Politehnica University of Timișoara).

The network is segmented with VLANs, routed with Router-on-a-Stick, configured automatically through
DHCP, reachable by name through DNS, connected to the Internet through NAT/PAT and secured with ACLs.

## Scenario

A company needs an internal network for several departments. Each department must receive IP addresses
automatically, reach only the resources it is allowed to, and inter-VLAN traffic must be controlled by
access lists. The company has the following zones:

- IT department
- HR department
- Guest network for visitors
- Internal server zone
- Internet connection through an ISP

## Topology

| Device                     | Quantity |
|----------------------------|----------|
| Company router (CE)        | 1        |
| ISP router                 | 1        |
| Switches                   | 2        |
| PCs (IT, HR, Guest)        | 3        |
| Internal server            | 1        |
| External server / Internet | 1        |

Connections:
- IT, HR and Guest PCs connected to the switches
- Internal server connected in VLAN 40
- Trunk link between the two switches
- Trunk link between a switch and the company router
- WAN link between the company router and the ISP router
- External server connected to the ISP router

## VLANs and IP Addressing

| VLAN | Name    | Role          | Network          | Gateway      |
|------|---------|---------------|------------------|--------------|
| 10   | IT      | IT department | 192.168.10.0/24  | 192.168.10.1 |
| 20   | HR      | HR department | 192.168.20.0/24  | 192.168.20.1 |
| 30   | GUEST   | Visitors      | 192.168.30.0/24  | 192.168.30.1 |
| 40   | SERVERS | Servers       | 192.168.40.0/24  | 192.168.40.1 |

| Link                   | Network         |
|------------------------|-----------------|
| Company – ISP (WAN)    | 220.110.0.0/30  |
| Simulated Internet     | 10.0.0.0/8      |

The internal server uses the static address **192.168.40.10**.

## Implemented Features

### Inter-VLAN Routing (Router-on-a-Stick)
The company router has one subinterface per VLAN (10, 20, 30, 40), each configured with that VLAN's
gateway address and 802.1Q encapsulation.

### DHCP + DHCP Relay
The internal server provides a DHCP pool for VLANs 10, 20 and 30 (pool name, default gateway, DNS server,
start IP, subnet mask, maximum number of users). Because the server sits in VLAN 40, the router relays
DHCP requests with `ip helper-address 192.168.40.10` on the VLAN 10, 20 and 30 subinterfaces.

### DNS
The internal server runs DNS with the record:

    www.firma.local  ->  192.168.40.10

### Internal Web Server
HTTP is enabled on the internal server and can be tested from any PC's browser at
`http://www.firma.local`.

### OSPF
OSPF runs between the company router and the ISP router:
- the routers form an OSPF adjacency
- the WAN network is advertised
- the ISP router advertises the external server's network
- the company router learns a route to the external server

### NAT / PAT
The company router provides Internet access through NAT overload:
- internal subinterfaces set as `ip nat inside`
- the WAN interface set as `ip nat outside`
- an ACL matching the private networks 192.168.10.0/24, 192.168.20.0/24 and 192.168.30.0/24
- PAT on the WAN interface

### ACL Security Policy

| # | Rule                                                               |
|---|--------------------------------------------------------------------|
| 1 | GUEST (VLAN 30) cannot access IT (VLAN 10)                         |
| 2 | GUEST (VLAN 30) cannot access HR (VLAN 20)                         |
| 3 | GUEST can access the internal server **only over HTTP**            |
| 4 | GUEST cannot ping the internal server                              |
| 5 | HR (VLAN 20) cannot access IT (VLAN 10)                            |
| 6 | IT (VLAN 10) has full access to all VLANs and servers              |
| 7 | GUEST can access the external server / Internet                    |

## Repository Contents

| File                              | Description                                   |
|-----------------------------------|-----------------------------------------------|
| `Final_Project.pkt`               | Cisco Packet Tracer simulation of the network |
| `Final_Project_Requirements.pdf`  | Project requirements (in Romanian)            |

## How to Run

1. Open `Final_Project.pkt` in Cisco Packet Tracer.
2. On a PC, go to *Desktop → IP Configuration* and select **DHCP** to receive an address.
3. Open *Desktop → Web Browser* and go to `http://www.firma.local`.
4. Use *Command Prompt* (`ping`, `tracert`) to check the ACL rules and Internet access.
5. On the routers, check the configuration with `show ip route`, `show ip ospf neighbor`,
   `show ip nat translations` and `show access-lists`.

## Author

**Tănase Alexandra** – [@Alexandra07e](https://github.com/Alexandra07e)
