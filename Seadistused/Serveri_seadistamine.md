# 1. Spetsifikatsioon:

Minu server on Primergy RX 200 S7, minu serveril on 2 Samsung 1TB SSD, 64GB ram (8x8GB), XEON E5-2630 2,3GHz protsessor ja d3032 mainboard.

# 2. Seadistamine:

F2 et saada BIOS-i, F12 et valida boot device, C et minna konfiguratsiooni, CTRL+H et saada webBIOS

## 2.1 Bios update:
Mälupulga formaatisin FAT32 ja laadisin sinna Fujitsu support lehelt [Fujitsu](https://support.ts.fujitsu.com/IndexDownload.asp?SoftwareGuid=AF69ECB5-28B4-4C13-9522-0F652093C86D) BIOS admin packi d3032 mainboardi jaoks, "Direct download", kus sees oli siis kõige uuem BIOS.

Täpselt nii. Kui sa oled BIOS-i juba ära uuendanud, siis on sul vundament olemas. Järgmiseks mine otse WebBIOS-i (Ctrl + H), sest seal määrad sa ära, kuidas server kettaid näeb.
Kuna õpetaja tahab JBOD-i (et Proxmox saaks ketastele otse ligi), siis WebBIOS-is on sul kaks valikut:
1. Kontrolli, kas JBOD on toetatud
Mõnes D2607 püsivara versioonis saab JBOD-i lihtsalt sisse lülitada:
Mine Controller Properties -> Next -> Next.
Otsi rida "JBOD Mode". Kui seal on "Disabled", proovi see panna "Enabled".
Kui see õnnestub, siis Proxmox näeb kettaid täpselt nii nagu vaja.
2. Kui JBOD valikut pole (tavaline D2607 puhul)
Kui sa JBOD-i sisse lülitada ei saa, siis tee "Single Drive RAID 0":
Vali Configuration Wizard -> New Configuration.
Vali Manual Configuration.
Vali ainult esimene ketas ja tee sellest RAID 0.
Vali ainult teine ketas ja tee sellest teine RAID 0.
Nii on sul kaks "Virtual Drive'i", kus kummaski on üks füüsiline ketas. Proxmoxi jaoks on see sisuliselt sama mis JBOD ja sa saad kasutada ZFS-i.
Miks see on praegu parim plaan?
Sa ei raiska aega keeruliste kontrolleri uuenduste peale (mis võivad ebaõnnestuda).
Sa saad Proxmoxi installiga kohe edasi minna.
Õpetaja eesmärk (et Proxmox haldaks kettaid) saab täidetud.
Tee nii:
Võta mälupulk välja.
Tee restart ja vajuta Ctrl + H.
Vaata, kas saad kuskilt JBOD-i sisse lülitada.
Kas sul õnnestus WebBIOS-i menüüdesse sisse pääseda?
