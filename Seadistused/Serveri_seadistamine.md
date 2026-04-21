# 1. Spetsifikatsioon:

Minu server on Primergy RX 200 S7, minu serveril on 2 Samsung 1TB SSD, 64GB ram (8x8GB), XEON E5-2630 2,3GHz protsessor ja d3032 mainboard.

# 2. Seadistamine:

F2 et saada BIOS-i, F12 et valida boot device, C et minna konfiguratsiooni, CTRL+H et saada webBIOS, CTRL+Y läheb webBIOS käsureale.

## 2.1 Bios update:
Mälupulga formaatisin FAT32 ja laadisin sinna Fujitsu support lehelt [Fujitsu](https://support.ts.fujitsu.com/IndexDownload.asp?SoftwareGuid=AF69ECB5-28B4-4C13-9522-0F652093C86D) BIOS admin packi d3032 mainboardi jaoks, "Direct download", kus sees oli siis kõige uuem BIOS.

Kui BIOS-i sain uuendanud, kulutasin liiga kaua aega JBOD kävitamisele, mille käigus selgus, et isegi kui nupp on olemas, siis kõigega see ei tööta. Minu probleemiks oli liiga vana server ja mainboard.
1. Kontrolli, kas JBOD on toetatud
Mõnes püsivara versioonis saab JBOD-i lihtsalt sisse lülitada:
Mine Controller Properties -> Next -> Next.
Otsi rida "JBOD Mode". Kui seal on "Disabled", pane "Enabled".
2. Kui JBOD valikut pole
Kui sa JBOD-i sisse lülitada ei saa, siis tee "Single Drive RAID 0":
Vali Configuration Wizard -> New Configuration.
Vali Manual Configuration.
Vali ainult esimene ketas ja tee sellest RAID 0.
Vali ainult teine ketas ja tee sellest teine RAID 0.
Nii on sul kaks "Virtual Drive'i", kus kummaski on üks füüsiline ketas. Süsteemi jaoks on see sisuliselt sama, mis JBOD ja sa saad kasutada ZFS-i.
<img width="4096" height="3072" alt="Raid2" src="https://github.com/user-attachments/assets/690807ed-981c-4f42-83ff-3f0462f72e92" />
<img width="4096" height="3072" alt="Raid" src="https://github.com/user-attachments/assets/a4a27824-9395-40b8-91b5-f57773fe0f77" />
<img width="4096" height="3072" alt="Raid1" src="https://github.com/user-attachments/assets/dbccfc70-f898-4036-a7db-c00d4bdf4436" />

## 2.2 Proxmoxi pealelaadimine
Kõigepealt tuleb laadida proxmoxi kõige uuem versioon, või kindluse mõttes natuke vanem versioon, mis käitub nagu sillana, et hiljem uuendada kõige uuemale versioonile.
Mina leidsin omale vajaliku versiooni siit [Proxmox](https://www.proxmox.com/en/)
