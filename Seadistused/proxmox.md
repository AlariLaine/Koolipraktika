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
