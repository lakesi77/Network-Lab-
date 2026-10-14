STP Lab — Spanning Tree Protocol

## Objective

Build a 3-switch Layer 2 loop, observe STP electing a root bridge, blocking a redundant port, and maintaining connectivity without loops.

 SW1 Gi0/1 ↔ SW2 Gi0/1
- SW2 Gi0/2 ↔ SW3 Gi0/2
- SW3 Gi0/1 ↔ SW1 Gi0/2

**End devices (all VLAN 10):**
- PC1 on SW1 Fa0/1 — 192.168.10.1/24
- PC2 on SW2 Fa0/2 — 192.168.10.2/24
- PC3 on SW3 Fa0/3 — 192.168.10.3/24

**Switches:** Cisco 2960 (Layer 2 only)

## What I Configured

### 1. VLAN and trunks on all switches
vlan 10
name DATA
!
interface range gi0/1-2
switchport mode trunk
switchport trunk allowed vlan 10
no shut

nterface fa0/1 (SW1)
switchport mode access
switchport access vlan 10
no shut

Same for SW2 Fa0/2 and SW3 Fa0/3.

### 3. Forced root bridge
spanning-tree vlan 10 root primary (on SW1)
spanning-tree vlan 10 root secondary (on SW2)

## Verification
show spanning-tree vlan 10
show interfaces trunk
show vlan brief
show mac address-table vlan 10

# Results

- **Root bridge:** SW1 (lowest priority after `root primary`)
- **Root ports:** SW2 and SW3 each elected one Root Port toward SW1
- **Blocked port:** One uplink on SW3 went to **Altn (Alternate/Blocking)** — STP preventing the loop
- **Connectivity:** PC1 pinged PC2 and PC3 successfully
- **Failover:** Shutting the forwarding uplink caused the Altn port to transition to Forwarding within seconds

Problem I Hit and Fixed

**Issue:** PC1 could not ping PC3.

**Cause:** The access port Fa0/1 on SW1 was still in VLAN 1, not VLAN 10. The trunk carried VLAN 10, but the PC port wasn't assigned to it.

**Fix:**
interface fa0/1
switchport mode access
switchport access vlan 10


After fixing the access port on all three switches, ping worked end-to-end.

## What I Learned

- STP automatically blocks redundant ports to prevent Layer 2 loops
- Root bridge election is based on Bridge ID (priority + MAC)
- Path cost determines Root Port and Designated Port selection
- `show spanning-tree vlan 10` is the key troubleshooting command
- Access ports must be explicitly assigned to a VLAN — trunks alone aren't enough
- 2960 switches don't need `switchport trunk encapsulation dot1q` (Layer 2 only)
