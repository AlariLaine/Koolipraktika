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
* Märkus: Otsest SSH sessiooni kliendi arvutist piirab koolivõrgu turvapoliitika, mis katkestab sessiooni pärast TCP kätlemist (sarnane pveproxy käitumisega).
* Testimise käigus selgus, et ühendus läbi ruuteri välisliidese (.212) on piiratud kliendi arvuti turvaseadete tõttu.

### Miks .212 vajab tulemüüri väljalülitamist?

*   **NAT ja asümmeetria:** Kuna päring liigub läbi Cisco ruuteri NAT-i, siis vastuspakett jõuab kliendi arvutini teistsuguse teekonnaga kui tavaline sisevõrgu liiklus.
*   **Windows Defender:** Windowsi tulemüür peab sellist NAT-itud tagasiliiklust tundmatuks ja blokeerib selle vaikimisi ära (asümmeetriline marsruutimine).
*   **Lahendus:** Praktikas tõestati ühenduse toimivust Wiresharki logidega (3-way handshake kinnitus) ja kliendi tulemüüri ajutise keelamisega.
<img width="1901" height="1071" alt="Screenshot 2026-04-23 132814" src="https://github.com/user-attachments/assets/abfd2969-c55e-4e48-ac5d-14e67b6b4dfe" />

---

## Testimine

Testimise käigus ilmnes probleem:

* TCP SYN paketid jõudsid serverini
* server vastas
* kuid vastus ei jõudnud kliendini

Põhjus:
* tulemüür blokeeris tagasiliikluse

Ajutine lahendus:

* tulemüür keelati → ühendus töötas

Lõplik lahendus:

* tulemüüri reegleid tuleb kohandada, et lubada NAT-itud ühenduste tagasiliiklus

---
