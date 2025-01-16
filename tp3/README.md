# I Setup réseau

## Grand A

### Config `Routeur1`

**IP dynamique `Routeur1`**
````
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  administratively down down
FastEthernet0/1            unassigned      YES unset  administratively down down
FastEthernet1/0            unassigned      YES unset  administratively down down
FastEthernet2/0            unassigned      YES unset  administratively down down

R1#conf t
R1(config)#int FastEthernet0/0
R1(config-if)#ip address dhcp
R1(config-if)#no shut


R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.122.37  YES DHCP   up                    up  
FastEthernet0/1            unassigned      YES unset  administratively down down
FastEthernet1/0            unassigned      YES unset  administratively down down
FastEthernet2/0            unassigned      YES unset  administratively down down
````

**IP statique `Routeur1`**
````
R1#conf t
R1(config)#int FastEthernet0/1
R1(config-if)#ip address 10.3.1.254 255.255.255.0
R1(config-if)#no shut

R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.122.37  YES DHCP   up                    up  
FastEthernet0/1            10.3.1.254      YES NVRAM  up                    up  

R1#conf t
R1(config)#int FastEthernet1/0
R1(config-if)#ip address 10.3.12.1 255.255.255.252
R1(config-if)#no shut

R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.122.37  YES DHCP   up                    up  
FastEthernet0/1            10.3.1.254      YES NVRAM  up                    up  
FastEthernet1/0            10.3.12.1       YES NVRAM  up                    up  
````

**IP statique `Routeur2`**
````

R1#conf t
R1(config)#int FastEthernet0/0
R1(config-if)#ip address 10.3.12.2 255.255.255.252
R1(config-if)#no shut


R2#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.12.2       YES NVRAM  up                    up 

R1#conf t
R1(config)#int FastEthernet0/1
R1(config-if)#ip address 10.3.2.254 255.255.255.0
R1(config-if)#no shut

R2#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.3.12.2       YES NVRAM  up                    up  
FastEthernet0/1            10.3.2.254      YES NVRAM  up                    up 
````

**Save config `Routeur1/2`**
````
R2#copy running-config startup-config
Destination filename [startup-config]? (Press Enter)
Building configuration...
[OK]
````

**Vérif PING `Routeur1 à Routeur2`**
````
R1#ping 10.3.12.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.12.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 24/54/68 ms
````

**Vérif PING `Routeur2 à Routeur1`**
````
R2#ping 10.3.12.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.12.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 28/42/64 ms
````

## Grand B

### Config route R1 & R2

**Conf route `R1 vers R2`**

````
R1#conf t
R1(config)#ip route 10.3.2.0 255.255.255.0 10.3.12.2
R1#sh ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 192.168.122.1 to network 0.0.0.0

C    192.168.122.0/24 is directly connected, FastEthernet0/0
     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C       10.3.12.0/30 is directly connected, FastEthernet1/0
C       10.3.1.0/24 is directly connected, FastEthernet0/1
S       10.3.2.0/24 [1/0] via 10.3.12.2
S*   0.0.0.0/0 [254/0] via 192.168.122.1
````

**Vérif ping `R1 vers R2`**
````
R1#ping 10.3.2.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 88/91/96 ms
````

**Conf route `R2 vers R1`**
````
R2#conf t
R2(config)#ip route 10.3.1.0 255.255.255.0 10.3.12.1

R2#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C       10.3.12.0/30 is directly connected, FastEthernet0/0
S       10.3.1.0/24 [1/0] via 10.3.12.1
C       10.3.2.0/24 is directly connected, FastEthernet0/1
````

**Vérif ping `R2 vers R1`**
````
R2#ping 10.3.1.254

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.3.1.254, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 28/43/92 ms
````

**WIRESHARK**

[ping PC1 lan1 vers PC3 lan2](pingpc1lan1verspc3lan2.pcapng)


**Address MAC `R1`**
````

R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.3.12.1               -   c401.053e.0010  ARPA   FastEthernet1/0
Internet  10.3.12.2              19   c402.0561.0000  ARPA   FastEthernet1/0
Internet  10.3.1.1                4   0050.7966.6800  ARPA   FastEthernet0/1
Internet  192.168.122.1          11   5254.0023.04c2  ARPA   FastEthernet0/0
Internet  192.168.122.37          -   c401.053e.0000  ARPA   FastEthernet0/0
Internet  10.3.1.254              -   c401.053e.0001  ARPA   FastEthernet0/1
````

**Address MAC `R2`**
````
R2#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.3.12.1              20   c401.053e.0010  ARPA   FastEthernet0/0
Internet  10.3.12.2               -   c402.0561.0000  ARPA   FastEthernet0/0
Internet  10.3.2.1                6   0050.7966.6802  ARPA   FastEthernet0/1
Internet  10.3.2.254              -   c402.0561.0001  ARPA   FastEthernet0/1
````

## Grand C

**Accès internet `R1`**
````
R1#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 60/78/96 ms
````

**Accès internet `PC1 LAN1`**
````
PC1> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=50 time=116.038 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=50 time=87.411 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=50 time=95.348 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=50 time=78.596 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=50 time=58.500 ms
````

**Accès internet `R2`**
````
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#ip route 0.0.0.0 0.0.0.0 10.3.12.1
````

**Vérif internet `R2`**
````
R2#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 120/140/188 ms
````

**Vérif internet `PC3 LAN2`**
````
PC3> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=49 time=182.403 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=49 time=121.645 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=49 time=121.076 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=49 time=106.215 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=49 time=126.829 ms
````

# II Router-on-a-stick

