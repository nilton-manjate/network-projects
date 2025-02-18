# Network Infrastructure Documentation
**Author:** Nilton Manjate

## Overview
This documentation details the network infrastructure designed with VLAN segmentation, inter-VLAN routing using HSRP, and an optimized spanning-tree protocol for redundancy and performance. The network has been configured to support future implementations, including essential servers such as DHCP, SNMP, and NTP.

## Topology
The network is organized into three layers:
1. **Core** - Responsible for interconnecting the entire network and providing connectivity to the Internet and servers.
2. **Distribution** - Responsible for inter-VLAN routing and redundancy.
3. **Access** - Connects end-host devices and distributes traffic to the distribution layer.
![network topology](/images/topology.png)

## Subnets and VLANs
The network was segmented using VLSM (Variable Length Subnet Mask) to optimize IPv4 addressing space.

| VLAN  | Department     | Network Range        | Subnet Mask        |
|-------|---------------|----------------------|--------------------|
| 10    | Engineering   | 172.16.2.0/25        | 255.255.255.128   |
| 20    | Sales         | 172.16.0.0/23        | 255.255.254.0     |
| 30    | Finance       | 172.16.2.128/27      | 255.255.255.224   |
| 40    | HR            | 172.16.2.160/27      | 255.255.255.224   |
| 50    | Head Office   | 172.16.2.192/29      | 255.255.255.248   |

### IPv4 Addressing Configuration
Currently, end devices (PCs) are configured with static IPv4 addressing. In the future, a DHCP server will be implemented for automatic address distribution.

## Spanning Tree Protocol (STP)
To prevent network loops and ensure fast convergence, all switches are configured with **Rapid PVST+** mode.

### Root Bridge
- **DSW1** is the root bridge for VLANs 10, 20, and 30.
- **DSW2** is the root bridge for VLANs 40 and 50.

## Inter-VLAN Routing and High Availability
Inter-VLAN routing is performed by the distribution layer switches (**DSW1 and DSW2**) using the **HSRP (Hot Standby Router Protocol)** for redundancy and load balancing.

## Device Configuration
### Access Switch Configuration (ASW1)
```console
! VLAN Configuration
ASW1(config)#vlan 10
ASW1(config-vlan)#name Engineering
ASW1(config-vlan)#vlan 20
ASW1(config-vlan)#name Sales
ASW1(config-vlan)#vlan 30
ASW1(config-vlan)#name Finance
ASW1(config-vlan)#vlan 40
ASW1(config-vlan)#name HR
ASW1(config-vlan)#vlan 50
ASW1(config-vlan)#name Head-Office

! Access ports configuration
ASW1(config)#interface range f0/1-2
ASW1(config-if-range)#switchport mode access
ASW1(config-if-range)#switchport access vlan 10
ASW1(config-if-range)#interface range f0/3-4
ASW1(config-if-range)#switchport mode access
ASW1(config-if-range)#switchport access vlan 20
ASW1(config-if-range)#interface range f0/5-6
ASW1(config-if-range)#switchport mode access
ASW1(config-if-range)#switchport access vlan 30

! Trunk ports configuration
ASW1(config)#interface range g0/1-2
ASW1(config-if-range)#switchport mode trunk
ASW1(config-if-range)#switchport trunk native vlan 97
ASW1(config-if-range)#switchport trunk allowed vlan 10,20,30,40,50

! Rapid PVST+ STP Configuration
spanning-tree mode rapid-pvst
```

### Access Switch Configuration (ASW2)
```console
! VLAN Configuration
ASW2(config)#vlan 10
ASW2(config-vlan)#name Engineering
ASW2(config-vlan)#vlan 20
ASW2(config-vlan)#name Sales
ASW2(config-vlan)#vlan 30
ASW2(config-vlan)#name Finance
ASW2(config-vlan)#vlan 40
ASW2(config-vlan)#name HR
ASW2(config-vlan)#vlan 50
ASW2(config-vlan)#name Head-Office

! Access ports configuration
ASW2(config)#interface range f0/1-2
ASW2(config-if-range)#switchport mode access
ASW2(config-if-range)#switchport access vlan 40
ASW2(config-if-range)#interface range f0/3-4
ASW2(config-if-range)#switchport mode access
ASW2(config-if-range)#switchport access vlan 50
ASW2(config-if-range)#interface range f0/5-6
ASW2(config-if-range)#switchport mode access
ASW2(config-if-range)#switchport access vlan 20

! Trunk ports configuration
ASW2(config)#interface range g0/1-2
ASW2(config-if-range)#switchport mode trunk
ASW2(config-if-range)#switchport trunk native vlan 97
ASW2(config-if-range)#switchport trunk allowed vlan 10,20,30,40,50

! Rapid PVST+ STP Configuration
spanning-tree mode rapid-pvst
```


### Distribution Switch Configuration (DSW1)
```console
DSW1(config)#spanning-tree mode rapid-pvst
DSW1(config)#spanning-tree vlan 10 root primary
DSW1(config)#spanning-tree vlan 20 root primary
DSW1(config)#spanning-tree vlan 30 root primary
DSW1(config)#spanning-tree vlan 40 root secondary
DSW1(config)#spanning-tree vlan 50 root secondary

DSW1(config)#interface vlan 10
DSW1(config-if)#ip address 172.16.2.1 255.255.255.128
DSW1(config-if)#exit
DSW1(config)#standby 10 ip 172.16.2.3
DSW1(config)#standby 10 priority 200
DSW1(config)#standby 10 preempt

DSW1(config)#interface vlan 20
DSW1(config-if)#ip address 172.16.0.1 255.255.254.0
DSW1(config-if)#exit
DSW1(config)#standby 20 ip 172.16.0.3
DSW1(config)#standby 20 priority 200
DSW1(config)#standby 20 preempt

DSW1(config)#interface vlan 30
DSW1(config-if)#ip address 172.16.2.129 255.255.255.224
DSW1(config-if)#exit
DSW1(config)#standby 30 ip 172.16.2.131
DSW1(config)#standby 30 priority 200
DSW1(config)#standby 30 preempt

DSW1(config)#interface vlan 40
DSW1(config-if)#ip address 172.16.2.161 255.255.255.224
DSW1(config-if)#exit
DSW1(config)#standby 40 ip 172.16.2.163
DSW1(config)#standby 40 priority 50
DSW1(config)#standby 40 preempt

DSW1(config)#interface vlan 50
DSW1(config-if)#ip address 172.16.2.1 255.255.255.248
DSW1(config-if)#exit
DSW1(config)#standby 50 ip 172.16.2.3
DSW1(config)#standby 50 priority 50
DSW1(config)#standby 50 preempt
```

### Distribution Switch Configuration (DSW2)
```console
DSW2(config)#spanning-tree mode rapid-pvst
DSW2(config)#spanning-tree vlan 10 root primary
DSW2(config)#spanning-tree vlan 20 root primary
DSW2(config)#spanning-tree vlan 30 root primary
DSW2(config)#spanning-tree vlan 40 root secondary
DSW2(config)#spanning-tree vlan 50 root secondary

DSW2(config)#interface vlan 10
DSW2(config-if)#ip address 172.16.2.2 255.255.255.128
DSW2(config-if)#exit
DSW2(config)#standby 10 ip 172.16.2.3
DSW2(config)#standby 10 preempt

DSW2(config)#interface vlan 20
DSW2(config-if)#ip address 172.16.0.2 255.255.254.0
DSW2(config-if)#exit
DSW2(config)#standby 20 ip 172.16.0.3
DSW2(config)#standby 20 preempt

DSW2(config)#interface vlan 30
DSW2(config-if)#ip address 172.16.2.130 255.255.255.224
DSW2(config-if)#exit
DSW2(config)#standby 30 ip 172.16.2.131
DSW2(config)#standby 30 preempt

DSW2(config)#interface vlan 40
DSW2(config-if)#ip address 172.16.2.162 255.255.255.224
DSW2(config-if)#exit
DSW2(config)#standby 40 ip 172.16.2.163
DSW2(config)#standby 40 preempt

DSW2(config)#interface vlan 50
DSW2(config-if)#ip address 172.16.2.2 255.255.255.248
DSW2(config-if)#exit
DSW2(config)#standby 50 ip 172.16.2.3
DSW2(config)#standby 50 preempt
```

## Diagnostic Command Outputs
To validate the configuration, the following commands were used:

### Configured VLANs
```console
show vlan brief
```
![show vlan brief](/enterprise-network/images/show-vlan-brief.png)

### Spanning Tree Status
```console
show spanning-tree
```
![show spanning-tree](/enterprise-network/images/show-span-vlan-10-20.png)
![show spanning-tree](/enterprise-network/images/show-span-vlan-30-40.png)
![show spanning-tree](/enterprise-network/images/show-span-vlan-50.png)

### HSRP Status
```console
show standby brief
```
![show standby brief](/enterprise-network/images/show-standby-brief.png)

### Interfaces and Status
```console
show ip interface brief
```
![show ip interface brief](/enterprise-network/images/show-ip-int-br.png)

## Future Plans
- Configure the link between the distribution and core layers for Internet access.
- Implement a DHCP server for dynamic IP address assignment.
- Configure SNMP and NTP servers for network monitoring and time synchronization.

---
This documentation will be updated as the network evolves and new features are implemented.

