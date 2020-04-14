# Lab-routage-dynamique
Lab routage dynamique du 14.04.2020

## Etape 1 : Configuration globale des routeurs

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
Idem pour R1 et R3

## Etape 2 : Verifier, activer et configurer les interfaces IPv4

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

`show ip int brief` permet d'avoir les infos de L2 (colonne status : logique ; colonne protocol : physique) pour chacun des routeurs

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

```R1(config)#int g0/1
R1(config-if)#no cdp enable
```
Ici, on déactive cdp sur l'interface donnant sur l'Internet (car indiscret)

`cdp run` permet d'activer globalement sur toutes interfaces du routeurs

```
R1#show cdp neighbors
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R3.lan3          Gig 0/3           154              R B             Gig 0/1
R2.lan2          Gig 0/2           177              R B             Gig 0/1
R1.lan           Gig 0/1           2                R B             Gig 0/1
R1.lan           Gig 0/1           22               R B             Gig 0/1
R1               Gig 0/1           25               R B             Gig 0/1
```

On observe que R2 et R3 sont reliés, puisque qu'on accède aux infos de leurs interfaces connectées. 

(Holdtme = la Valeur du délai de conservation en secondes, Capability = le Code de capacité du périphérique voisin)

On pourra faire 
```R1#show cdp neighbors detail``` 
pour obtenir les adresses IP des interfaces voisines.

`show ip route` permet de savoir si la connectivité de L3 est fonctionnelle entre les routeurs.
On pourra aussi faire un `ping` sur chaque routeur via chaque routeur.

## Etape 3 : Activer DHCP pour Ipv4

```ip dhcp pool LANR1
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 1.1.1.1
exit
ip dhcp excluded-address 192.168.1.1 192.168.1.100
exit

ip dhcp pool LANR2
network 192.168.33.0 255.255.255.0
default-router 192.168.33.2
dns-server 1.1.1.1
exit
ip dhcp excluded-address 192.168.33.1 192.168.33.100
exit

ip dhcp pool LANR3
network 192.168.65.0 255.255.255.0
default-router 192.168.65.3
dns-server 1.1.1.1
exit
ip dhcp excluded-address 192.168.65.1 192.168.65.100
exit
```

Pour vérifier le pool DHCP :
`R1#show ip dhcp pool LANR1`

On active le DHCP relay sur R2 et R3 :
```R2(config)#int g0/0
R2(config-if)#ip helper-address 192.168.1.1

R3(config)#int g0/0
R3(config-if)#ip helper-address 192.168.1.1
```
Pour le moment, le routage n'est pas activé pour la destination 192.168.1.1 donc PC2 et PC3 ne peuvent pas échanger de messages DHCP.

## Etape 4 : Configurer et activer le routage OSPFv2

Bande passante à 1 Gbit/s

```R1#router ospf 1
R1(config-router)#router-id 1.1.1.1
R1(config-router)#passive-interface g0/0
R1(config-router)#network 192.168.1.1 0.0.0.0 area 0
R1(config-router)#network 192.168.225.1 0.0.0.0 area 0
R1(config-router)#network 192.168.226.1 0.0.0.0 area 0
R1(config-router)#auto-cost reference-bandwidth 1000 

R2
router ospf 1
router-id 2.2.2.2
passive-interface g0/0
network 192.168.33.2 0.0.0.0 area 0
network 192.168.225.2 0.0.0.0 area 0
network 192.168.227.2 0.0.0.0 area 0
auto-cost reference-bandwidth 1000 

R3
router ospf 1
router-id 3.3.3.3
passive-interface g0/0
network 192.168.65.3 0.0.0.0 area 0
network 192.168.226.3 0.0.0.0 area 0
network 192.168.227.3 0.0.0.0 area 0
auto-cost reference-bandwidth 1000 
```

Les interfaces dites "passives" sont celles qui n'envoient aucun message d'un protocole de routage comme OSPF.
Par contre, cette mesure n'empêche pas l'interface de prendre en compte des messages OSPF.

On verifiera la configuration PSPFv2 avec `show ip protocols | b ospf 1`.
On remarquera les ID des routeurs voisins.

`show id ospf neighbor` permet de voir sui est DR
`clear ip ospf process`
* R3 est DR (ID la plus elevée qui remporte l'élection)
* R2 est DR
* R1 est BDR

On pourra changer la priorité 
```(config)# interface G0/0
(config-if)# ip ospf priority 255
```
L'ensemble des destinations à joindre est :
* 192.168.1.0
* 192.168.33.0
* 192.168.65.0
* 192.168.225.0
* 192.168.226.0
* 192.168.227.0

On retrouvera l'ensemble des destinations joignable avec `sh ip route` sur chaque router.

On verifiera la connectivité avec `ping` de bout en bout PC2 > PC1 et PC3 > PC1 

## Etape 5 : Activer les interfaces et le routage IPv6

On configure link-local sur chaque interface :

```R1(config)#int g0/0
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#int g0/2
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#int g0/3
R1(config-if)#ipv6 address fe80::1 link-local

R2
int g0/0
ipv6 address fe80::2 link-local
int g0/1
ipv6 address fe80::2 link-local
int g0/3
ipv6 address fe80::2 link-local

R3
int g0/0
ipv6 address fe80::3 link-local
int g0/1
ipv6 address fe80::3 link-local
int g0/2
ipv6 address fe80::3 link-local
```
On configure les Unique Local Address:

```R1
int g0/0
ipv6 address fd00:192:168:1::1/64
int g0/2
ipv6 address fd00:192:168:225::1/64
int g0/3
ipv6 address fd00:192:168:226::1/64

R2
int g0/0
ipv6 address fd00:192:168:33::2/64
int g0/1
ipv6 address fd00:192:168:225::2/64
int g0/3
ipv6 address fd00:192:168:227::2/64

R3
int g0/0
ipv6 address fd00:192:168:65::3/64
int g0/1
ipv6 address fd00:192:168:226::3/64
int g0/2
ipv6 address fd00:192:168:227::3/64
```

On active sur chacun des routeurs :

`ipv6 unicast-routing`

## Etape 6

## Etape 7 : Activer et configurer de la connec&vité IPv4 publique

On définit les ACL:

```access-list 1 permit 192.168.1.0 0.0.0.255
access-list 1 permit 192.168.33.0 0.0.0.255
access-list 1 permit 192.168.65.0 0.0.0.255
```
On crée la règle de traduction nat :
```
R1(config)#ip nat inside source list 1 interface g0/1 overload
```
Overload est necessaire pour avoir plusieurs acces simultannés (plusieurs ip privées auront acces à internet)

On configure les interfaces sur R1 :
```
R1(config)#int g0/0
R1(config-if)#ip nat inside
R1(config-if)#int g0/2
R1(config-if)#ip nat inside
R1(config-if)#int g0/3
R1(config-if)#ip nat inside
R1(config-if)#int g0/1
R1(config-if)#ip nat outside
R1(config-if)#end
```


Toutefois, les pings à partir de R3 et R2 ne fonctionnent pas. Il leur faut une passerelle par défault.
`show ip` route sur R3 et R2 nous permet d'observer qu'il n'y a pas de route vers l'internet.

On active cette route manquante :

```R1(config)#router ospf 1
R1(config-router)#default-information originate
```

Alors en `show ip route` sur R2 et R3 on observera :
```OE2  0.0.0.0/0 [110/1] via 192.168.225.1, 00:17:16, GigabitEthernet0/1
OE2  0.0.0.0/0 [110/1] via 192.168.226.1, 00:17:43, GigabitEthernet0/1
```

Les ping fonctionnent désormais sur les LAN et les routeurs


