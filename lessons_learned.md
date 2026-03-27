# Troubleshooting & Lessons Learned

## Overview
Briefly describe what this troubleshooting section covers.

---

## Issues Encountered

### Issue #1: IPs not being handed out to end devices via DHCP.

**Symptoms:**
- End devices showing blank IPs and gateways after configuring ROAS and DHCP IP pools.
- In some cases the end device could ping gateway, most of the time not.

**Root Cause:**
- Not all components of the VLAN path were configured (access/trunk ports, VLAN creation itself on the switch).

**Troubleshooting Process:**
- First, ping gateway. Failure = pathing problem. Success = DHCP problem.
- Second, analyze each switch with the commands below. Identify that the target VLAN exists and that the appropriate trunk/access interfaces recognize it.

```cisco
# show interfaces trunk
# show vlan brief
```

**Lessons Learned**
This initial problem was really important for me to really understand VLANs from a top down perspective. For my topology, it was not necessary for each switch to know about every VLAN in the topology (5,10,20,30,40,50,55,60). From endpoint to router, each switch along the way must carry VLAN traffic. Because of the ROAS configuration, as long as a VLAN reaches it's gateway the router will then retag the traffic with the appropriate destination network. This saves some hassle initially, but I imagine that it will take strong documentation to avoid headaches if this network were to grow.

One example of this is the sales VLAN 30. Switch3 Fa0/23 northbound only knows about one VLAN - 30. If traffic from HR needed to get to sales, it would travel from an HR endpoint to the gateway, then be re-encapsulated with a VLAN 30 tag. Because of this, Switch3 Fa0/23 will recognize it and transport it to the appropriate sales endpoint. It is crucial, however, that VLAN 30 be present on Switch0, Switch1, and Switch2 as well as the appropriate interfaces on these switches for this process to succeed.
