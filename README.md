# SPAN-OSPF-SLA-on-Multilayer-Switch


**Configuring VLAN Routing, OSPF, and SPAN on a Multilayer Switch**

In this lab, I configured three VLANs on a multilayer switch, enabled routing, established an OSPF adjacency, and configured traffic mirroring for monitoring.

**VLANs and gateway interfaces**

I assigned the switch ports and configured the following switched virtual interfaces (SVIs):

| VLAN | IP address | Access port |
|---|---|---|
| 10 | 10.10.1.1/24 | Gi0/1 |
| 12 | 10.12.1.1/24 | Fa0/1 |
| 14 | 10.14.1.1/24 | Fa0/2 |

The SVIs initially showed **administratively down**. I enabled each using `no shutdown`. For an SVI to become operational, its VLAN must also exist and have an active forwarding port.

I then enabled Layer 3 forwarding:

```cisco
ip routing
```

Before this command, `show ip route` displayed host-mode information rather than the normal routing table. Configuring SVI addresses alone does not enable inter-VLAN routing.

**OSPF configuration**

I enabled OSPF process 1 and included all three networks in Area 0:

```cisco
router ospf 1
 network 10.10.1.0 0.0.0.255 area 0
 network 10.12.1.0 0.0.0.255 area 0
 network 10.14.1.0 0.0.0.255 area 0
```

On VLAN 10, I configured the OSPF network type as point-to-point:

```cisco
interface Vlan10
 ip ospf network point-to-point
```

This network type avoids a DR/BDR election on that link. The switch formed a **FULL adjacency** with neighbor `1.1.1.1`, and a ping to `10.10.1.2` succeeded.

An early OSPF startup message also highlighted an important prerequisite: without a manually configured router ID, OSPF needs an eligible operational IP interface from which to select one.

**Traffic monitoring with SPAN**

I configured local SPAN session 11 to copy traffic received and transmitted on VLAN 12 to Gi0/2:

```cisco
monitor session 11 source vlan 12 both
monitor session 11 destination interface GigabitEthernet0/2
```

A packet analyzer connected to Gi0/2 can inspect the mirrored traffic. This port serves as the monitoring destination rather than a normal user-facing switch port.

**Verification**

```cisco
show vlan brief
show ip interface brief
show ip ospf neighbor
show ip route
show monitor session 11
```

This lab reinforced the importance of enabling SVIs, activating IP routing, verifying OSPF neighbors, and selecting the correct SPAN source and destination. The configuration and local connectivity were verified; packet capture on the monitoring device would confirm the mirrored traffic.
