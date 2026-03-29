# Troubleshooting & Lessons Learned

## Overview
This file describes my lessons learned through issues or targeted practice.

---

### #1 - DHCP Problems: IPs not being handed out to end devices via DHCP.

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
- This initial problem was really important for me to really understand VLANs from a top down perspective. For my topology, it was not necessary for each switch to know about every VLAN in the topology (5,10,20,30,40,50,55,60). From endpoint to router, each switch along the way must carry VLAN traffic. Because of the ROAS configuration, as long as a VLAN reaches it's gateway the router will then retag the traffic with the appropriate destination network. This saves some hassle initially, but I imagine that it will take strong documentation to avoid headaches if this network were to grow.

- One example of this is the sales VLAN 30. Switch3 Fa0/23 northbound only knows about one VLAN - 30. If traffic from HR needed to get to sales, it would travel from an HR endpoint to the gateway, then be re-encapsulated with a VLAN 30 tag. Because of this, Switch3 Fa0/23 will recognize it and transport it to the appropriate sales endpoint. It is crucial, however, that VLAN 30 be present on Switch0, Switch1, and Switch2 as well as the appropriate interfaces on these switches for this process to succeed.


### #2 - Standard vs. Extended ACLs: When to use which.

**Tasks**
- Set up ACLs on my network to further secure it.
- Preplan requirements.
- Decide on what type of ACLs are needed.

**Requirements**
- HR can only be reachable by IT and HR.
- SC2 can only be reachable by IT using ICMP and SSH protocols.
- Telnet must be blocked on all VLANs.

**Lessons Learned**
- The topology page contains specific commands that I used to build these, but overall my largest issue here was deciding on the placement and type of ACL. Initially, I was going to place a standard ACL with a list of deny statements closest to the source (in this case, the outbound g0/0.X outbound) for both HR and SC2. This is actually okay for HR, but failed in the case of SC2 because standard ACLs can only filter by IP, not specific protocols. Therefore, the SC2 requirement as well as the Telnet requirement must be extended ACLs placed close to the source, for IT that would be g0/0.60 inbound. Why? Extended ACLs are focused traffic restriction, and should be caught as early as possible.
