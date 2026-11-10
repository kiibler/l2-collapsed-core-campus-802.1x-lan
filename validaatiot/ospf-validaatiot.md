# OSPFv2

### 1. OSPF-naapuruudet

```text
DSW1# show ip ospf neighbor 
Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           0   FULL/  -        00:00:32    4.4.4.4         GigabitEthernet2/1
3.3.3.3           0   FULL/  -        00:00:36    3.3.3.3         GigabitEthernet2/0
2.2.2.2           0   FULL/  -        00:00:34    2.2.2.2         GigabitEthernet1/2
2.2.2.2           0   FULL/  -        00:00:38    2.2.2.2         GigabitEthernet1/3
```

### 2. Equal-Cost Multi-Path (ECMP) -reititystaulukko

```text
DSW1# show ip route ospf
      2.0.0.0/32 is subnetted, 1 subnets
O        2.2.2.2 [110/101] via 2.2.2.2, 1d04h, GigabitEthernet1/2
                 [110/101] via 2.2.2.2, 1d04h, GigabitEthernet1/3
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/101] via 3.3.3.3, 1d04h, GigabitEthernet2/0
      4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/101] via 4.4.4.4, 1d04h, GigabitEthernet2/1
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
O        172.16.2.0/26 [110/200] via 4.4.4.4, 00:00:47, GigabitEthernet2/1
                       [110/200] via 3.3.3.3, 00:00:57, GigabitEthernet2/0
O        172.16.2.64/26 [110/200] via 4.4.4.4, 00:00:47, GigabitEthernet2/1
                        [110/200] via 3.3.3.3, 00:00:57, GigabitEthernet2/0
O        172.16.2.128/26 [110/200] via 4.4.4.4, 00:00:47, GigabitEthernet2/1
                         [110/200] via 3.3.3.3, 00:00:57, GigabitEthernet2/0
O        172.16.2.192/26 [110/200] via 4.4.4.4, 1d04h, GigabitEthernet2/1
                         [110/200] via 3.3.3.3, 00:00:47, GigabitEthernet2/0
```
