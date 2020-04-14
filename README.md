# Lab-routage-dynamique
Lab routage dynamique du 14.04.2020

# Etape 1 : Configuration globale des routeurs

```Router#config t
Router(config)#hostname R2
R2(config)#enable secret testtest
R2(config)#service password-encryption
R2(config)#username root secret testtest
R2(config)#ip domain-name lan2
R2(config)#crypto key generate rsa
The name for the keys will be: R2.lan2
How many bits in the modulus [512]: 2048
R2(config)#
R2(config)#ip ssh version 2
R2(config)#line vty 0 4
R2(config-line)#transport input ssh
R2(config-line)#login local
R2(config-line)#exit
R2(config)#exit
R2#wr
```


# Etape 2 : Verifier, activer et configurer les interfaces IPv4

```
R1#show ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES unset  administratively down down
GigabitEthernet0/1         unassigned      YES unset  administratively down down
GigabitEthernet0/2         unassigned      YES unset  administratively down down
GigabitEthernet0/3         unassigned      YES unset  administratively down down
GigabitEthernet0/4         unassigned      YES unset  administratively down down
GigabitEthernet0/5         unassigned      YES unset  administratively down down
GigabitEthernet0/6         unassigned      YES unset  administratively down down
GigabitEthernet0/7         unassigned      YES unset  administratively down down
```

```
R1(config)#int g0/0
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shut
R1(config-if)#int g0/2
R1(config-if)#ip address 192.168.225.1 255.255.255.0
R1(config-if)#no shut
R1(config-if)#int g0/3
R1(config-if)#ip address 192.168.226.1 255.255.255.0
R1(config-if)#no shut
R1(config-if)#int g0/1
R1(config-if)#ip address dhcp
R1(config-if)#access-list 1 permit 192.168.1.0 0.0.0.255
R1(config-if)#no shut
```

```
R2
int g0/0
ip address 192.168.33.2 255.255.255.0
no shut
int g0/1
ip address 192.168.225.2 255.255.255.0
no shut
int g0/3
ip address 192.168.227.2 255.255.255.0
no shut



R3
int g0/0
ip address 192.168.65.3 255.255.255.0
no shut
int g0/1
ip address 192.168.226.3 255.255.255.0
no shut
int g0/2
ip address 192.168.227.3 255.255.255.0
no shut
```
```
R1#show ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         192.168.1.1     YES manual up                    up  
GigabitEthernet0/1         192.168.122.193 YES DHCP   up                    up  
GigabitEthernet0/2         192.168.225.1   YES manual up                    up  
GigabitEthernet0/3         192.168.226.1   YES manual up                    up  
GigabitEthernet0/4         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/5         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/6         unassigned      YES NVRAM  administratively down down
GigabitEthernet0/7         unassigned      YES NVRAM  administratively down down
```

Idem sur R2 et R3 : les interfaces configurées sont up.

show ip int brief permet d'avoir les infos de L2 (colonne status : logique ; colonne protocol : physique) pour chacun des routeurs

```
R2#show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R3.lan3          Gig 0/3           145              R B             Gig 0/2
R1.lan1          Gig 0/1           151              R B             Gig 0/2


R3#show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R2.lan2          Gig 0/2           148              R B             Gig 0/3
R1.lan1          Gig 0/1           164              R B             Gig 0/3
```

R2 et R3 sont bien connectés physiquement entre eux et à R1

```
R1#show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R3.lan3          Gig 0/3           148              R B             Gig 0/1
R2.lan2          Gig 0/2           155              R B             Gig 0/1
R1.lan           Gig 0/1           168              R B             Gig 0/1
R1.lan           Gig 0/1           159              R B             Gig 0/1
R1.lan           Gig 0/1           135              R B             Gig 0/1
R1.LAB.tripod    Gig 0/1           144              R B             Gig 0/1
R1               Gig 0/1           176              R B             Gig 0/1
R1.R1.lan        Gig 0/1           160              R B             Gig 0/1
R1               Gig 0/1           178              R B             Gig 0/1
R1.mon.lan       Gig 0/1           122              R B             Gig 0/1
```

On observe que R2 et R3 sont reliés, puisque qu'on accède aux infos de leurs interfaces connectées. 
( Holdtme = la Valeur du délai de conservation en secondes, Capability = le Code de capacité du périphérique voisin)

On pourra faire 
```R1#show cdp neighbors detail``` 
pour obtenir les adresses IP des interfaces voisines.

`show ip route` permet de savoir si la connectivité de L3 est fonctionnelle entre les routeurs.
On pourra aussi faire un `ping` sur chaque routeur via chaque routeur.



