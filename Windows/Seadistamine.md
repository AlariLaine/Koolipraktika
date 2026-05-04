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

## 5. Group Policy (GPO)

## Windows Serveri haldus: Kasutajate import, GPO ja võrguseadistus
Käesolev dokumentatsioon kirjeldab minu poolt teostatud Windows Serveri keskkonna seadistamist. Projekti eesmärk oli automatiseerida kasutajate loomine, seadistada tsentraalne tarkvara paigaldus läbi grupipoliitikate (GPO) ja tagada turvaline võrgupääs.
## 1. Kasutajate ja struktuuri (OU) automatiseeritud import
Esmalt lõin PowerShell skripti, mis loob vajaliku organisatsiooniüksuste (OU) struktuuri ja impordib kasutajad CSV-failist.
## Teostatud sammud:

   1. OU struktuuri loomine: Lõin hierarhia ARVUTID ja KASUTAJAD koos vajalike alamüksustega (IT, Staff, Töötajad, Kontor, Raamatupidamine).
   2. Kasutajate import: Kirjutasin skripti, mis loeb kasutajad.csv faili ja loob kasutajad kujul eesnimi.perenimi.
   3. Gruppide haldus: Skript kontrollib automaatselt OU olemasolu enne kasutaja loomist ja lisab kasutaja vastavasse üksusesse.

# Kasutatud loogika näide
$samName = "$($firstName).$($lastName)".ToLower()
New-ADUser -Name $u.Name -SamAccountName $samName -Path $targetOU -Enabled $true -ChangePasswordAtLogon $true


## 2. Tarkvara tsentraalne paigaldus (GPO)
Tarkvara haldamiseks seadistasin serveris võrgujagamise ja lõin arvutipõhised GPO-d.
## Ettevalmistus:

* Lõin kausta C:\Deploy ja jagasin selle võrgus (\\DC1\Deploy).
* Andsin Domain Computers grupile Read õigused nii jagamise kui ka NTFS tasemel.

## GPO seadistused:

| GPO Nimi | Tegevus | Siht-OU |
|---|---|---|
| ID-kaardi tarkvara | Paigaldab DigiDoc4 (.msi) | OU=Töötajad |
| Libreoffice | Paigaldab LibreOffice (.msi) | OU=Staff |
| Google Chrome | Paigaldab Chrome Enterprise ja 7-Zip | OU=ARVUTID |

Märkus: LibreOffice MSI-faili muutmiseks kasutasin tarkvara Orca, et eemaldada keelevalikutest tulenevad paigaldusvead.

## 3. Kasutajate töökeskkonna reeglid
Lõin GPO-d, mis määravad kasutajate seaded sõltumata arvutist, kuhu nad sisse logivad.
## 3.1 Google Chrome'i avaleht

   1. Lisasin Google Chrome'i ADMX/ADML haldusmallid serveri PolicyDefinitions kausta.
   2. Lõin GPO "Veebilehe avaleht" ja linkisin selle üksusele KASUTAJAD.
   3. Seadistasin: User Configuration -> Administrative Templates -> Google Chrome -> Configure the home page URL väärtuseks https://hkhk.edu.ee.

## 3.2 Ettevõtte taustapilt

   1. Lõin GPO "Taustapilt" ja linkisin selle üksusele KASUTAJAD.
   2. Seadistasin: User Configuration -> Desktop -> Desktop -> Desktop Wallpaper.
   3. Määrasin pildi asukohaks võrgutee \\DC1\Deploy\taustapilt.jpg ja stiiliks Fill.


## 4. Turvaseadistused
Rakendasin kriitilised turvameetmed, et kaitsta domeeni kasutajaandmeid.
## 4.1 Viimase sisselogitud kasutaja peitmine

   1. Lõin GPO "Peida kasutaja" ja linkisin selle üksusele ARVUTID.
   2. Seadistasin: Computer Configuration -> Security Settings -> Local Policies -> Security Options -> Interactive logon: Don't display last signed-in: Enabled.

## 4.2 Turvaline paroolipoliitika
Kasutasin Active Directory Administrative Center (ADAC) abi, et luua Fine-Grained Password Policy, mis rakendub otse Domain Users grupile.

* Ajalugu: 5 viimast parooli hoitakse meeles.
* Pikkus: Minimaalselt 4 sümbolit.
* Keerukus: Nõutud on suurtäht, number ja erisümbol.

## Kokkuvõte ja kontroll
Kõik seadistused on testitud Windows 11 kliendi masinas.

* Käsk gpupdate /force rakendab reeglid koheselt.
* Arvuti taaskäivitus kinnitab tarkvara paigalduse ja sisselogimise muudatused.



