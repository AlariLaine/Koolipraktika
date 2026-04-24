# Füüsiline võrgulahendus

## Komponendid
Cisco 2960 switch

Cisco 1941 ruuter

Minu läpakas

Proxmox server

Kooliarvuti

## Ülevaade

Füüsiline võrgulahendus põhineb samal loogikal nagu Cisco Packet Tracer simulatsioon, kuid arvestab reaalse võrgu eripärasid nagu NAT, tulemüür ja seadmete tegelik käitumine.

Peamine erinevus simulatsiooniga võrreldes on see, et kogu välisliiklus läbib ruuterit, mis teostab nii NAT-i kui ka tulemüüri funktsiooni.

---

## Topoloogia

Füüsiline ühendus:

```
            [ Internet / Koolivõrk ]
                      |
                  (WAN)
                      |
                 [ Ruuter R5 ]
                      |
                trunk (VLANid)
                      |
                 [ Switch SW51 ]
          |        |        |        |
     VLAN100   VLAN30   VLAN10   VLAN200
      Proxmox    VM-id     PC       WiFi/AP
```

---

## Füüsilised ühendused (Kaabeldus)

| Seade ja Port        | Ühendub seadmega ja pordiga | Kaabli tüüp | Märkused                         |
| -------------------- | --------------------------- | ----------- | -------------------------------- |
| Ruuter (R5) G0/0     | Koolivõrk / Internet        | Ethernet    | Lan ühendus                      |
| Ruuter (R5) G0/1     | Switch (SW51) Gig0/2        | Ethernet    | **Trunk (VLAN 1,10,30,100,200)** |
| Switch (SW51) Gig1/0/1  | Proxmox server           | Ethernet    | VLAN 100                         |
| Switch (SW51)        | VM server                   | Ethernet    | VLAN 30                          |
| Switch (SW51)        | Kontori PC                  | Ethernet    | VLAN 10                          |
| Switch (SW51)        | WiFi Access Point           | Ethernet    | VLAN 200                         |

---

## Loogiline ülesehitus (VLAN ja IP)

| VLAN | Nimi     | Võrk            | Gateway      | Kirjeldus            |
| ---- | -------- | --------------- | ------------ | -------------------- |
| 1    | HALDUS   | 172.21.1.0/28   | 172.21.1.1   | Võrguseadmete haldus |
| 10   | KONTOR   | 172.21.10.0/28  | 172.21.10.1  | Töötajate arvutid    |
| 30   | SERVERID | 172.21.30.0/28  | 172.21.30.1  | Virtuaalmasinad      |
| 100  | HOST     | 172.21.100.0/28 | 172.21.100.1 | Proxmox host         |
| 200  | WIFI     | 172.21.200.0/28 | 172.21.200.1 | Külaliste võrk       |

---

## Aadressid ja ligipääsud


| Teenus | Sise-IP | Väline port | Ligipääsu aadress | Märkused |
| :--- | :--- | :--- | :--- | :--- |
| **Proxmox Web (Avalik)** | 172.21.100.2 | 8011 | [https://hkhk.edu.ee](https://hkhk.edu.ee) | Ligipääs kodust  |
| **Proxmox Web (NAT)** | 172.21.100.2 | 8011 | `https://192.168.30.212:8011` | Vajab kliendi tulemüüri väljalülitamist |
| **Proxmox Web (Otse)** | 172.21.100.2 | 8006 | `https://192.168.30.213:8006` | Töötab alati (ei läbi NAT-i) |
| **Proxmox SSH** | 172.21.100.2 | 2200 | `ssh -p 2200 root@192.168.30.212` | Testimiseks läbi Cisco NAT-i |
| **Windows SRV 1** | 172.21.30.3 | 3389 | `192.168.30.212:3389` | RDP (Vajab Windows VM-i) |
| **Linux SRV** | 172.21.30.5 | 2201 | `ssh -p 2201 kasutaja@192.168.30.212` | SSH suunamine läbi R5 |
| **Switch** | 172.21.1.2 | 2203 | `ssh -p 2203 root@192.168.30.212` | Haldusvõrgu ligipääs |

## Tehniline teostus ja märkused

## Switch SSH (Port 2203) testimine:
Sisevõrgu test: Ruuterist (R5) SSH-ühendus kommutaatorisse (172.21.1.2) õnnestus, mis kinnitab SSH-teenuse ja lüüsi (default-gateway) korrektset seadistust.
* NAT-i test: Ruuteri debug ip nat kinnitab, et välisvõrgust porti 2203 tulnud päringud suunatakse edukalt kommutaatorisse.
* Märkus: Otsest SSH sessiooni kliendi arvutist peab kasutama porti 2203.

---
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
</details>
```

---

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
