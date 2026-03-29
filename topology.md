
---

# Topology

# Network Design

## Overview
This network simulates a segmented enterprise environment with multiple departments, centralized services, and controlled VLAN propagation.

## Design Decisions

### VLAN Segmentation
Each department is isolated into its own VLAN to:
- Reduce broadcast domains
- Improve security
- Simplify policy enforcement

### DHCP Architecture
A centralized DHCP server was used in VLAN 5.

All other VLANs use:
- `ip helper-address` on router subinterfaces
- DHCP relay to forward broadcast requests

### Routing Strategy
Router-on-a-Stick (ROAS) was implemented:
- One physical interface
- Multiple subinterfaces
- 802.1Q tagging

### Trunk Design
Trunks were configured to:
- Allow only required VLANs
- Limit unnecessary broadcast traffic
- Enforce segmentation

Example:
- Southbound trunks only allow VLANs 20, 30, 60
- Server VLANs restricted to required paths

## Addressing Scheme
- /24 networks for user VLANs
- /28 networks for server VLANs

This conserves IP space while maintaining scalability.

## ACL Usage
Current ACL implementations (extended and named)
Extended IP access list 100
- This ACL is applied on the router's inbound g0/0.10 interface to deny HR access to both Telnet and Server Cluster 2 (SC2), while allowing everything else.
    ```10 deny tcp 10.0.10.0 0.0.0.255 any eq telnet
    20 deny ip 10.0.10.0 0.0.0.255 10.0.55.0 0.0.0.15
    30 permit ip 10.0.10.0 0.0.0.255 any 
Extended IP access list IT_POLICY
- This ACL denies Telnet out of the IT VLAN, permits SSH and ICMP into SC2, denies all other IT > SC2 traffic, and allows IT to reach all other VLANs with non Telnet traffic.
    ```10 deny tcp 10.0.60.0 0.0.0.255 any eq telnet
    20 permit tcp 10.0.60.0 0.0.0.255 10.0.55.0 0.0.0.15 eq 22
    30 permit icmp 10.0.60.0 0.0.0.255 10.0.55.0 0.0.0.15
    40 deny ip 10.0.60.0 0.0.0.255 10.0.55.0 0.0.0.15
    50 permit ip 10.0.60.0 0.0.0.255 any
ip access-list standard HR_FILTER
- This ACL is a standard one, blocking all traffic into HR that is not IT or HR. This will be applied on g0/0.10 outbound interface, close to the source.
 ```deny 10.0.20.0 0.0.0.255
    deny 10.0.30.0 0.0.0.255
    deny 10.0.40.0 0.0.0.255
    deny 10.0.50.0 0.0.0.255
    deny 10.0.55.0 0.0.0.15
    permit 10.0.60.0 0.0.0.255
    permit 10.0.10.0 0.0.0.255
