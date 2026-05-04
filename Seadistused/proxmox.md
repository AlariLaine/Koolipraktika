# Proxmox VE Virtuaalmasinate Standard (Template)

Kõik labori virtuaalmasinad on loodud järgides enam vähem ühtset seadistust.

## 1. Riistvara konfiguratsiooni standardid


| Komponent | Seadistus | Selgitus |
| :--- | :--- | :--- |
| **CPU Type** | `host` | Võimaldab VM-il kasutada füüsilise protsessori kõiki käsustikke (AES, AVX jne). |
| **Machine** | `q35` | Kaasaegne chipset, mis on vajalik Windows 11 ja PCIe passthrough toe jaoks. |
| **BIOS** | `OVMF (UEFI)` | Nõutud Windows 11 ja Windows Server 2022 turvaliseks alglaadimiseks. |
| **SCSI Controller** | `VirtIO SCSI single` | Pakub parimat IOPS jõudlust ja madalat latentsust. |
| **Network Model** | `VirtIO (paravirtualized)` | Kõige kiirem virtuaalne võrgukaart (nõuab Windowsis eraldi draivereid). |

## 2. Salvestusruumi optimeerimine
Kuna kasutusel on **local-zfs** salvestusruum, on kõigil ketastel rakendatud järgmised seaded:
- **Discard:** Sisse lülitatud (TRIM tugi), et vabastada kettapinda ZFS-is pärast failide kustutamist VM-is.
- **SSD Emulation:** Sisse lülitatud, et operatsioonisüsteem teaks, et asub kiirel meedial.

## 3. Windowsi-spetsiifilised seaded
Windows 11 ja Windows Server 2022 ühilduvuse tagamiseks on lisatud:
- **TPM State:** Versioon v2.0 (salvestatud `local-zfs` peale).
- **QEMU Guest Agent:** Sisse lülitatud, et võimaldada korrektset *shutdown* käsku ja näha VM-i IP-aadresse Proxmoxi liideses.

## 4. Võrgu sildamine
Kõik masinad on ühendatud sillaga `vmbr1`, mis on füüsiliselt ühendatud Cisco switchiga (Trunk port).
- **VLAN Tagging:** Kasutatakse Proxmoxi tasemel VLAN märgistamist (nt Tag 10 klientidele ja Tag 30 serveritele).

<img width="304" height="533" alt="image" src="https://github.com/user-attachments/assets/e7d47140-f21c-4315-96ec-f6c5140a88c2" />


# Proxmox VirtioFS seadistus Windowsi VM-ile

See juhend kirjeldab, kuidas seadistada ühiskasutuskaust Proxmoxi hosti ja Windowsi VM vahel VirtioFS abil, et võimaldada failide lihtsat ülekandmist.

## Eeltingimused
- Proxmox VE 8.4 või uuem
- Windowsi VM (nt Windows Server 2022)
- `virtio-win.iso` monteeritud VM-le


## 1.Lisa kaustakaart Proxmoxi veebiliideses
  Mine Datacenter > Directory Mappings. Klõpsa Add. Määra:Name: VM-SharePath: /home/VMShare, Node: [Sõlme nimi], Comment: Ühiskasutuskaust Windowsi VM-ile, Klõpsa Create.
## 2. VM riistvaraseadistus
  Lülita Windowsi VM välja. Mine VM Hardware vahele. Klõpsa Add > Virtiofs. Vali Directory ID rippmenüüst VM-Share. Klõpsa Add.
## 3. Windowsi VM seadistus
  ### Paigalda VirtIO Guest Tools
  Lülita VM sisse. Ava virtio-win.iso failihalduris. Käivita virtio-win-guest-tools.exe. Veendu, et paigaldamise käigus on valitud viofs (Virtio File System) komponent. Paigalda WinFSP, lae alla viimane winfsp-x.x.xxxx.msi aadressilt https://winfsp.dev/rel/. Käivita paigaldaja ja järgi juhiseid. Käivita Virtio-FS teenusVajuta Win + R, sisesta services.msc ja vajuta Enter. Leia Virtio-FS Service, määra Startup type väärtuseks Automatic. Klõpsa Start.
## 4. Juurdepääs ühiskasutuskaustle
Ava "This PC" failihalduris. Ühiskasutuskaust kuvatakse uue võrgukaustana (tavaliselt Z:). Failid, mille kopeerid Proxmoxi hosti kausta /home/VMShare, on kohe saadaval VM-i Z: kettal. Kasutamine failide ülekandmiseks: Kasuta scp, WINSCP või Proxmoxi veebikäsku, et kopeerida faile Proxmoxi hosti kausta /home/VMShare. Pääse nendele failidele kohe ligi Windowsi VM-i (Z): kettal.
