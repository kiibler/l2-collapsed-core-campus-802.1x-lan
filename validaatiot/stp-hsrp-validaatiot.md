# HSRP-gatewayredundanssi ja STP Juurikytkimet

### 1. DSW1 HSRP Active VLAN 10/20

```text
DSW1# show standby brief
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   254 P Active  local           172.16.1.61     172.16.1.1
Vl20        20   254 P Active  local           172.16.1.125    172.16.1.65
Vl30        30   100   Standby 172.16.1.189    local           172.16.1.129
Vl40        40   100   Standby 172.16.1.253    local           172.16.1.193
```

### 2. DSW2 HSRP Active VLAN 30/40

```text
DSW2# show standby brief
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   100   Standby 172.16.1.62     local           172.16.1.1
Vl20        20   100   Standby 172.16.1.126    local           172.16.1.65
Vl30        30   254 P Active  local           172.16.1.190    172.16.1.129
Vl40        40   254 P Active  local           172.16.1.254    172.16.1.193
```

### 3. STP Root Bridge ja HSRP Active -roolien täsmäävyys

```text
DSW1# show spanning-tree vlan 10 active 
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    24586
             Address     006b.e645.3c10
             This bridge is the root

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   Shr 
Po2                 Desg FWD 3         128.66   Shr 
Po3                 Desg FWD 3         128.67   Shr
```

```text
DSW1#show spanning-tree vlan 20 active
VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    24596
             Address     006b.e645.3c10
             This bridge is the root

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   Shr 
Po2                 Desg FWD 3         128.66   Shr 
Po3                 Desg FWD 3         128.67   Shr 
```

```text
DSW1#show spanning-tree vlan 30 active
VLAN0030
  Root ID    Priority    24606
             Address     0059.bc45.6d10
             Cost        6
             Port        66 (Port-channel2)

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   Shr 
Po2                 Root FWD 3         128.66   Shr 
Po3                 Altn BLK 3         128.67   Shr 
```

```text
DSW1#show spanning-tree vlan 40 active
VLAN0040
  Root ID    Priority    24616
             Address     0059.bc45.6d10
             Cost        6
             Port        66 (Port-channel2)

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   Shr 
Po2                 Root FWD 3         128.66   Shr 
Po3                 Altn BLK 3         128.67   Shr
```
