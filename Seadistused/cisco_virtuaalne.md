# Cisco Packet Tracer
Enne kui päris ühenduste kallale läksin, tegin kogu ühenduse simulatsioonis läbi.

### Komponendid: <br>
Switch 2960 <br>
Ruuter 1941<br>
2x Server-PT<br>
PC<br>
Laptop<br>
Cloud<br>

## Topoloogia <br>

| Teenus | Sise-IP | Siseport | Väline Port | Ligipääsu aadress |
| :--- | :--- | :--- | :--- | :--- |
| **Proxmox Web** | 172.21.100.2 | 8006 | 8006 | https://192.168.30.213:8006 |
| **Proxmox SSH** | 172.21.100.2 | 22 | 2200 | ssh -p 2200 root@192.168.30.212 |
| **Windows SRV 1** | 172.21.30.3 | 3389 | 3389 | RDP: 192.168.30.212:3389 |
| **Windows SRV 2** | 172.21.30.4 | 3389 | 3390 | RDP: 192.168.30.212:3390 |
| **Linux SRV 1** | 172.21.30.5 | 22 | 2201 | ssh -p 2201 root@192.168.30.212 |
| **Switch SSH** | 172.21.1.2 | 22 | 2203 | ssh -p 2203 root@192.168.30.212 |
| **Välisveeb** | 172.21.30.10 | 80 | 80 | http://192.168.30.212 |


### Füüsilised ühendused (Kaabeldus)


| Seade ja Port | Ühendub seadmega ja pordiga | Kaabli tüüp | Märkused |
| :--- | :--- | :--- | :--- |
| **Ruuter (R5) G0/0** | **Cloud (WAN)** | Copper Straight | Välisühendus (Koolivõrk) |
| **Ruuter (R5) G0/1** | **Switch (SW51) G0/1** | Copper Straight | **Trunk liin** (VLAN 1, 10, 30, 100, 200) |
| **Switch (SW51) Fa0/1** | **Server1 (Proxmox)** | Copper Straight | Access port (VLAN 100) |
| **Switch (SW51) Fa0/2** | **Server2 (VMid)** | Copper Straight | Access port (VLAN 30) |
| **Switch (SW51) Fa0/10** | **PC-Kontor** | Copper Straight | Access port (VLAN 10) |
| **Switch (SW51) Fa0/20** | **Laptop/AP** | Copper Straight | Access port (VLAN 200) |

### Loogiline ülesehitus (VLAN ja IP)


| VLAN | Nimi | Alamvõrk (CIDR) | Default Gateway | Seadmed selles võrgus |
| :---: | :--- | :--- | :--- | :--- |
| **1** | HALDUS | `172.21.1.0/28` | `172.21.1.1` | Ruuter, Switch, Wifi AP |
| **10** | KONTOR | `172.21.10.0/28` | `172.21.10.1` | Töötajate arvutid (DHCP) |
| **30** | SERVERID | `172.21.30.0/28` | `172.21.30.1` | VM-id, DHCP server, Veeb |
| **100** | HOST | `172.21.100.0/28` | `172.21.100.1` | Füüsiline Proxmox server |
| **200** | WIFI | `172.21.200.0/28` | `172.21.200.1` | Külaliste seadmed |


<img width="641" height="506" alt="image" src="https://github.com/user-attachments/assets/de25ae5a-7589-4dd7-be87-d85c5964916c" />

### Simulatsiooni ja reaalse võrgu erinevused

Kuigi Cisco Packet Tracer simulatsioon töötas probleemideta, ilmnes reaalses keskkonnas viga.

Peamised erinevused:

* Packet Tracer ei simuleeri realistlikult stateful firewalli käitumist
* NAT töötab lihtsustatud kujul (puudub ühenduste jälgimine)
* Puuduvad vahepealsed võrguseadmed (nt tulemüürid, turvapoliitikad)
* Tagasitee (return traffic) filtreerimist ei modelleerita

Seetõttu:

* simulatsioonis töötas ühendus (SYN → SYN-ACK)
* reaalses võrgus blokeeris ruuteri firewall tagasiliikluse

### Järeldus

Simulatsioon sobib loogika testimiseks, kuid ei pruugi paljastada:

* firewalli probleeme
* NAT + firewall koosmõju
* turvapoliitikate mõju liiklusele
### Lisaks topoloogia kohta

* Cisco Packet Tracer simulatsioonis on "Cloud" kasutatud lihtsustatud kujul ning ei kajasta reaalse võrgu keerukust
* Tegelikus lahenduses toimub kogu välisühendus läbi ruuteri (NAT + firewall)
* Switch ei ole otse ühendatud välisvõrguga
* Kõik välised teenused on kättesaadavad läbi ruuteri IP aadressi (192.168.30.212)
* Ruuter teostab nii NAT-i kui ka tulemüüri funktsiooni
* Firewall konfiguratsioon mõjutas ühenduste toimimist reaalses keskkonnas

## Ruuteri config
<details>
  <summary>Siin on kood</summary>

```
R5#show run
Building configuration...

Current configuration : 2209 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R5
!
!
!
!
!
!
!
!
no ip cef
no ipv6 cef
!
!
!
username root privilege 15 secret 5 $1$mERr$xCAe7t5qmMzw2xZrM/vyX.
!
!
license udi pid CISCO1941/K9 sn FTX1524792L-
!
!
!
!
!
!
!
!
!
ip domain-name alari.hkhk.edu.ee
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 description VALISVORK_WAN
 ip address 192.168.30.212 255.255.255.240
 ip nat outside
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/1.1
 description HALDUS
 encapsulation dot1Q 1 native
 ip address 172.21.1.1 255.255.255.240
 ip nat inside
!
interface GigabitEthernet0/1.10
 description KONTOR
 encapsulation dot1Q 10
 ip address 172.21.10.1 255.255.255.240
 ip helper-address 172.21.30.2
 ip nat inside
!
interface GigabitEthernet0/1.30
 description V-SERVERID
 encapsulation dot1Q 30
 ip address 172.21.30.1 255.255.255.240
 ip nat inside
!
interface GigabitEthernet0/1.100
 description HOSTSERVER
 encapsulation dot1Q 100
 ip address 172.21.100.1 255.255.255.240
 ip nat inside
!
interface GigabitEthernet0/1.200
 description GUEST-WIFI
 encapsulation dot1Q 200
 ip address 172.21.200.1 255.255.255.240
!
interface Vlan1
 no ip address
 shutdown
!
ip nat inside source static tcp 172.21.100.2 8006 192.168.30.212 8006 
ip nat inside source static tcp 172.21.100.2 22 192.168.30.212 2200 
ip nat inside source static tcp 172.21.30.3 3389 192.168.30.212 3389 
ip nat inside source static tcp 172.21.30.4 3389 192.168.30.212 3390 
ip nat inside source static tcp 172.21.30.5 22 192.168.30.212 2201 
ip nat inside source static tcp 172.21.30.6 22 192.168.30.212 2202 
ip nat inside source static tcp 172.21.1.2 22 192.168.30.212 2203 
ip nat inside source static tcp 172.21.30.10 80 192.168.30.212 80 
ip nat inside source static tcp 172.21.100.2 8006 192.168.30.212 8020 
ip classless
!
ip flow-export version 9
!
!
access-list 10 permit 192.168.30.208 0.0.0.15
!
!
!
!
snmp-server community public RO
!
line con 0
!
line aux 0
!
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
!
!
end
## Switchi config

```
---
</details>

## Switchi config
<details>
  <summary>Siin on kood</summary>

```
## Ruuteri config
<details>
  <summary>Siin on kood</summary>

```
R5#show run
Building configuration...

Current configuration : 3120 bytes

 Last configuration change at 08:15:44 UTC Fri Apr 24 2026 by root
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption

hostname R5

boot-start-marker
boot-end-marker

no aaa new-model
memory-size iomem 15

ip cef

ip domain name alari.hkhk.edu.ee
no ipv6 cef
multilink bundle-name authenticated

license udi pid CISCO1941/K9 sn FCZ17379124

username root privilege 15 secret 4 poRtFJyHWx67i5jN3K2.F42XsI9MrKrK8k0X0SwEVy6

ip ssh version 2

interface Embedded-Service-Engine0/0
 no ip address
 shutdown

interface GigabitEthernet0/0
 ip address 192.168.30.212 255.255.255.0
 ip nat outside
 ip virtual-reassembly in
 duplex auto
 speed auto

interface GigabitEthernet0/1
 no ip address
 ip nat inside
 ip virtual-reassembly in
 duplex auto
 speed auto

interface GigabitEthernet0/1.1
 encapsulation dot1Q 1 native
 ip address 172.21.1.1 255.255.255.240
 ip nat inside
 ip virtual-reassembly in

interface GigabitEthernet0/1.10
 description Kontor
 encapsulation dot1Q 10
 ip address 172.21.10.1 255.255.255.240
 ip helper-address 172.21.30.2

interface GigabitEthernet0/1.30
 description V-serverid
 encapsulation dot1Q 30
 ip address 172.21.30.1 255.255.255.240

interface GigabitEthernet0/1.100
 description Host
 encapsulation dot1Q 100
 ip address 172.21.100.1 255.255.255.240
 ip nat inside
 ip virtual-reassembly in

interface GigabitEthernet0/1.200
 description Guest
 encapsulation dot1Q 200
 ip address 172.21.200.1 255.255.255.240

interface Serial0/0/0
 no ip address
 shutdown
 clock rate 2000000

interface Serial0/0/1
 no ip address
 shutdown
 clock rate 2000000

ip default-gateway 192.168.30.1
ip forward-protocol nd

no ip http server
no ip http secure-server

ip nat inside source list 1 interface GigabitEthernet0/0 overload
ip nat inside source static tcp 172.21.30.10 80 192.168.30.212 80 extendable
ip nat inside source static tcp 172.21.100.2 22 192.168.30.212 2200 extendable
ip nat inside source static tcp 172.21.30.5 22 192.168.30.212 2201 extendable
ip nat inside source static tcp 172.21.30.6 22 192.168.30.212 2202 extendable
ip nat inside source static tcp 172.21.1.2 22 192.168.30.212 2203 extendable
ip nat inside source static tcp 172.21.30.3 3389 192.168.30.212 3389 extendable
ip nat inside source static tcp 172.21.30.4 3389 192.168.30.212 3390 extendable
ip nat inside source static tcp 172.21.100.2 8006 192.168.30.212 8011 extendable
ip route 0.0.0.0 0.0.0.0 192.168.30.1

ip access-list extended gig0/1
 permit tcp any host 192.168.30.212 eq 2203

access-list 1 permit 172.21.0.0 0.0.255.255
access-list 1 permit 172.21.100.0 0.0.0.15
access-list 10 permit 192.168.30.0 0.0.0.255

snmp-server community public RO
snmp-server enable traps entity-sensor threshold

control-plane

line con 0
line aux 0
line 2
 no activation-character
 no exec
 transport preferred none
 transport output pad telnet rlogin lapb-ta mop udptn v120 ssh
 stopbits 1
line vty 0 4
 access-class 10 in
 login local
 transport input ssh

scheduler allocate 20000 1000

end`

## Switchi config

```
---
</details>

## Switchi config
<details>
  <summary>Siin on kood</summary>

```
SW51#show run
Building configuration...

Current configuration : 2115 bytes

 Last configuration change at 05:14:24 UTC Thu Mar 31 2011 by root
 NVRAM config last updated at 06:36:52 UTC Thu Mar 31 2011

version 15.0
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption

hostname SW51

boot-start-marker
boot-end-marker

username root privilege 15 secret 5 $1$1X0O$WWaNEGsyTXJkzbGQl2VOs1
no aaa new-model
switch 1 provision ws-c2960s-24ts-l

ip dhcp snooping vlan 10,30,100,200
ip dhcp snooping
ip domain-name alari.hkhk.edu.ee

spanning-tree mode pvst
spanning-tree extend system-id

vlan internal allocation policy ascending

ip ssh version 2

interface FastEthernet0
 no ip address
 shutdown

interface GigabitEthernet1/0/1
 switchport access vlan 100
 switchport mode access

interface GigabitEthernet1/0/2
 switchport mode trunk
 ip dhcp snooping trust

interface GigabitEthernet1/0/3

interface GigabitEthernet1/0/4

interface GigabitEthernet1/0/5

interface GigabitEthernet1/0/6

interface GigabitEthernet1/0/7

interface GigabitEthernet1/0/8

interface GigabitEthernet1/0/9

interface GigabitEthernet1/0/10
 switchport access vlan 10
 switchport mode access

interface GigabitEthernet1/0/11

interface GigabitEthernet1/0/12

interface GigabitEthernet1/0/13

interface GigabitEthernet1/0/14

interface GigabitEthernet1/0/15

interface GigabitEthernet1/0/16

interface GigabitEthernet1/0/17

interface GigabitEthernet1/0/18

interface GigabitEthernet1/0/19

interface GigabitEthernet1/0/20

interface GigabitEthernet1/0/21

interface GigabitEthernet1/0/22

interface GigabitEthernet1/0/23

interface GigabitEthernet1/0/24

interface GigabitEthernet1/0/25

interface GigabitEthernet1/0/26

interface GigabitEthernet1/0/27

interface GigabitEthernet1/0/28

interface Vlan1
 ip address 172.21.1.2 255.255.255.240

ip default-gateway 172.21.1.1
ip http server
ip http secure-server


snmp-server community public RO


line con 0
line vty 0 4
 login local
 transport input ssh
line vty 5 15
 login local
 transport input ssh

end
```
</details>
```
</details>
