# Windows Server 2022 ja Teenuste Seadistamine

Siin on seadistus Windows serveri, mis on läbitud DC1 (Domain Controller) seadistamisel.

## 1. Esmane seadistamine (DC1)
- **Hostinimi:** Arvuti nime muutsin `DC1`-ks enne rollide lisamist.
- **Võrk:** Määrasin staatiline IP-aadress `172.21.30.3/28` (VLAN 30).
- **Rollid:** Lisasin `Active Directory Domain Services` ja `DHCP Server`.

## 2. Domeeni ja Metsa (Forest) loomine
- **Domeeninimi:** Tegin uus meta nimega `alari.praktika`.
- **DSRM parool:** Seadistatud vastavalt labori turvanõuetele.
- **DNS:** AD paigaldamise käigus seadistati automaatselt DNS server, mis teenindab kogu domeeni nimesid.

## 3. DHCP Teenuse seadistamine
Seadistati DHCP skoop (Scope) VLAN 10 klientidele:
- **Skoobi nimi:** `Kliendid`
- **Vahemik:** `172.21.10.2` - `172.21.10.14`
- **Alamvõrgu mask:** `255.255.255.240` (/28)
- **Lüüs (Default Gateway):** `172.21.10.1`
- **DNS Server:** `172.21.30.3` (DC1 ise)
- **Autoriseerimine:** DHCP server on AD-s autoriseeritud ja teenus on aktiivne.

## 4. Klientmasina liitmine domeeni
Esimene klientmasin (**Klient1**) sai edukalt domeeni liidetud:
1. **Draiverid:** Installisin VirtIO võrgudraiverid, et Windows 11 tuvastaks virtuaalse võrgukaardi.
2. **Võrgu test:** Testisin ühendust DC1-ga käsuga `ping alari.praktika`.
3. **Domeeniga liitumine:** 
   - Arvuti seadetes määrasin domeeniks `alari.praktika`.
   - Kinnitasin `Administrator` kontoga.
   - Masin on nähtav AD "Computers" konteineris.

## 5. Group Policy (GPO), OU ...
*(Pooleli)*
- [ ] Tarkvara paigaldus (Chrome, LibreOffice, ID-kaart)
- [ ] Kasutajate piirangud ja turvapaketid
- [ ] OU skript
