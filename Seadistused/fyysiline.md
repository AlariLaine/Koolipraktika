# Füüsiline võrgulahendus

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
| Ruuter (R5) G0/0     | Koolivõrk / Internet        | Ethernet    | WAN ühendus                      |
| Ruuter (R5) G0/1     | Switch (SW51) G0/1          | Ethernet    | **Trunk (VLAN 1,10,30,100,200)** |
| Switch (SW51) Fa0/1  | Proxmox server              | Ethernet    | VLAN 100                         |
| Switch (SW51) Fa0/2  | VM server                   | Ethernet    | VLAN 30                          |
| Switch (SW51) Fa0/10 | Kontori PC                  | Ethernet    | VLAN 10                          |
| Switch (SW51) Fa0/20 | WiFi Access Point           | Ethernet    | VLAN 200                         |

---

## Loogiline ülesehitus (VLAN ja IP)

| VLAN | Nimi     | Alamvõrk        | Gateway      | Kirjeldus            |
| ---- | -------- | --------------- | ------------ | -------------------- |
| 1    | HALDUS   | 172.21.1.0/28   | 172.21.1.1   | Võrguseadmete haldus |
| 10   | KONTOR   | 172.21.10.0/28  | 172.21.10.1  | Töötajate arvutid    |
| 30   | SERVERID | 172.21.30.0/28  | 172.21.30.1  | Virtuaalmasinad      |
| 100  | HOST     | 172.21.100.0/28 | 172.21.100.1 | Proxmox host         |
| 200  | WIFI     | 172.21.200.0/28 | 172.21.200.1 | Külaliste võrk       |

---

## NAT ja teenuste avalikustamine

Kõik teenused on väljast kättesaadavad läbi ruuteri WAN IP aadressi:

**192.168.30.212**

| Teenus        | Sise-IP      | Siseport | Väline port | Ligipääs                        |
| ------------- | ------------ | -------- | ----------- | ------------------------------- |
| Proxmox Web   | 172.21.100.2 | 8006     | 8006        | https://192.168.30.212:8006     |
| Proxmox SSH   | 172.21.100.2 | 22       | 2200        | ssh -p 2200 root@192.168.30.212 |
| Windows SRV 1 | 172.21.30.3  | 3389     | 3389        | RDP                             |
| Windows SRV 2 | 172.21.30.4  | 3389     | 3390        | RDP                             |
| Linux SRV     | 172.21.30.5  | 22       | 2201        | SSH                             |
| Switch        | 172.21.1.2   | 22       | 2203        | SSH                             |
| Veebiserver   | 172.21.30.10 | 80       | 80          | HTTP                            |

---

## Tulemüür (Firewall)

Ruuteril on aktiveeritud stateful tulemüür (inspect).

Testimise käigus ilmnes probleem:

* TCP SYN paketid jõudsid serverini
* server vastas
* kuid vastus ei jõudnud kliendini

Põhjus:
👉 tulemüür blokeeris tagasiliikluse

Ajutine lahendus:

* tulemüür keelati → ühendus töötas

Lõplik lahendus:

* tulemüüri reegleid tuleb kohandada, et lubada NAT-itud ühenduste tagasiliiklus

---

## Olulised erinevused simulatsiooniga

* Packet Tracer ei simuleeri realistlikult tulemüüri käitumist
* NAT töötab lihtsustatud kujul
* puuduvad reaalsed võrgupiirangud (ACL, inspect, security policies)

Seetõttu töötas simulatsioon probleemideta, kuid füüsilises võrgus tuli lahendada tulemüüri konfiguratsiooniga seotud probleem.

---

## Kokkuvõte

Füüsiline lahendus järgib sama loogikat nagu simulatsioon:

* VLAN segmentatsioon
* Router-on-a-stick lahendus
* NAT port forwarding

Lisaks sisaldab:

* reaalselt toimivat tulemüüri
* korrektset liikluse kontrolli

Peamine probleem tekkis tulemüüri konfiguratsioonist, mitte võrgu loogikast.
