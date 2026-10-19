# EtherChannel Lab — LACP Link Aggregation

## Objective

Bundle two physical links between two switches into a single logical EtherChannel using LACP (802.3ad). Prove that STP sees one logical link instead of two, test load balancing, and verify that shutting one physical link does not break connectivity.

**Skills practiced:**
- EtherChannel concepts (bandwidth aggregation, redundancy)
- LACP (active/passive modes)
- Port-channel interface configuration
- STP interaction with EtherChannel
- Failover testing
- Troubleshooting bundle mismatches

---

## Topology





- SW1 Gi0/1 ↔ SW2 Gi0/1
- SW1 Gi0/2 ↔ SW2 Gi0/2
- PC1 on SW1 Fa0/1 — VLAN 10
- PC2 on SW2 Fa0/1 — VLAN 10

---

## Devices Used

| Device | Model | Role |
|--------|-------|------|
| SW1 | Cisco 2960 | Access switch |
| SW2 | Cisco 2960 | Access switch |
| PC1 | Generic PC | End device |
| PC2 | Generic PC | End device |

---

## IP Addressing Scheme

| Device | IP Address | Subnet Mask | VLAN |
|--------|------------|-------------|------|
| PC1 | 192.168.10.1 | 255.255.255.0 | 10 |
| PC2 | 192.168.10.2 | 255.255.255.0 | 10 |

**VLAN 10 — DATA**

---

## Port Assignment

| Device | Port | Connects To | Mode | Channel Group |
|--------|------|-------------|------|---------------|
| SW1 | Gi0/1 | SW2 Gi0/1 | Trunk (VLAN 10) | 1 (active) |
| SW1 | Gi0/2 | SW2 Gi0/2 | Trunk (VLAN 10) | 1 (active) |
| SW1 | Fa0/1 | PC1 | Access (VLAN 10) | — |
| SW2 | Gi0/1 | SW1 Gi0/1 | Trunk (VLAN 10) | 1 (active) |
| SW2 | Gi0/2 | SW1 Gi0/2 | Trunk (VLAN 10) | 1 (active) |
| SW2 | Fa0/1 | PC2 | Access (VLAN 10) | — |

---

## Configuration

### SW1

```cisco
enable
conf t
hostname SW1
no ip domain-lookup

vlan 10
 name DATA
exit

interface range gi0/1-2
 switchport mode trunk
 switchport trunk allowed vlan 10
 channel-group 1 mode active
 no shut

interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shut
