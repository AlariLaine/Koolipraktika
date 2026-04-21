# Cisco Packet Tracer
Enne kui päris ühenduste kallale läksin, tegin kogu ühenduse simulatsioonis läbi.

### Komponendid: <br>
Switch 2960 <br>
Ruuter 1941<br>
2x Server-PT<br>
PC<br>
Laptop<br>
Cloud<br>

### Topoloogia <br>
Võrgutopoloogia
VLAN
Nimi
Võrguaadress
Ruuteri IP
Mask
Kasutusvaldkond
1
HALDUS
172.21.1.0/28
172.21.1.1
255.255.255.240
Ruuter, Switch, Wifi AP
10
KONTOR
172.21.10.0/28
172.21.10.1
255.255.255.240
Kontori tööjaamad (DHCP)
30
V-SERVERID
172.21.30.0/28
172.21.30.1
255.255.255.240
Windows, Linux, Veebiserverid
100
HOSTSERVER
172.21.100.0/28
172.21.100.1
255.255.255.240
Proxmox füüsiline host
200
GUEST-WIFI
172.21.200.0/28
172.21.200.1
255.255.255.240
Külaliste traadita võrk

Pordisuunamiste (NAT) tabel
.
Teenus
Sise-IP (Näide)
Siseport
Väline Port
Ligipääsu aadress
Proxmox Web
172.21.100.2
8006
8006
https://192.168.30.212:8006
Proxmox SSH
172.21.100.2
22
2200
ssh -p 2200 root@192.168.30.212
Windows SRV 1
172.21.30.3
3389
3389
RDP: 192.168.30.212:3389
Windows SRV 2
172.21.30.4
3389
3390
RDP: 192.168.30.212:3390
Linux SRV 1
172.21.30.5
22
2201
ssh -p 2201 root@192.168.30.212
Switch SSH
172.21.1.2
22
2203
ssh -p 2203 root@192.168.30.212
Välisveeb
172.21.30.10
80
80
http://192.168.30.212

