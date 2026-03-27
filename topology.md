
---

# (Topology / Technical Write-Up)

#Note - this file was created with the help of AI.

```markdown
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
