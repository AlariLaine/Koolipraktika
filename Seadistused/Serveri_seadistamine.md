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

<img width="512" height="384" alt="Raid2" src="https://github.com/user-attachments/assets/690807ed-981c-4f42-83ff-3f0462f72e92" /><br>
<img width="512" height="384" alt="Raid" src="https://github.com/user-attachments/assets/a4a27824-9395-40b8-91b5-f57773fe0f77" /><br>
<img width="512" height="384" alt="Raid1" src="https://github.com/user-attachments/assets/dbccfc70-f898-4036-a7db-c00d4bdf4436" /><br>

## 2.2 Proxmoxi pealelaadimine
Kõigepealt tuleb laadida Proxmoxi iso kõige uuem versioon, või kindluse mõttes natuke vanem versioon, mis käitub nagu sillana, et hiljem uuendada kõige uuemale versioonile.
Mina leidsin omale vajaliku versiooni siit [Proxmox](https://www.proxmox.com/en/), peale seda kasutasin Rufust, et formaatida pulk. Valisin oma mälupulga ja Boot selection oli siis Proxmox iso, peale seda vajutasin start ja see tegi mulle pulga valmis. Panin mälupulga serverisse ja hakkasin laadima Proxmox VE installer.

## 2.3 Proxmoxi sätted
Peale laadimist peab ka endale vastavalt konfigureerima. Esimesel lehel vajuta "I agree", kui nõustud tingimustega, muidu edasi ei saa.

<img width="512" height="384" alt="Prox" src="https://github.com/user-attachments/assets/bcb32251-f847-4d0a-8536-6e76805256e4" /> 

Teisel paneelil enne kui next vajutada tuleb valida ZFS(RAID1), ja nüüd võib next vajutada.

<img width="512" height="384" alt="Prox1" src="https://github.com/user-attachments/assets/3dd09485-f6e0-4957-8215-97a08c1cb0dc" /><br>
<img width="512" height="384" alt="Prox2" src="https://github.com/user-attachments/assets/605fd4bd-dbb4-4d05-ad95-f0f7b0862d2f" />

Järgmisel lehel tuleb valida, kus elad, mis ajavöönd ja klaviatuuri paigutus.

<img width="512" height="384" alt="Prox3" src="https://github.com/user-attachments/assets/37f9b75d-f490-48d3-8287-e36ba39fc229" />

Järgmisel lehel loo endale parool, seda ära kellegagi jaga ning kirjuta email kuhu sa tahad, et teated tuleksid, vajuta next.

<img width="512" height="384" alt="Prox4" src="https://github.com/user-attachments/assets/4aeec079-d45c-404c-9362-e24fac461681" /><br>
Eeliviimasel lehel panin ma järgnevalt: <br>
Hostname: alari.hkhk.edu.ee<br>
IP address: 192.168.30.213 /24<br>
Gateway: 192.168.30.1<br>
DNS Server: 8.8.8.8<br>
✅Pin network interface names.<br>
<img width="512" height="384" alt="Prox5" src="https://github.com/user-attachments/assets/3f82b6e8-71de-4553-9384-5a07148a61d2" />

Võib vajutada next. Viimasel lehel kontrolli veel viimast korda üle, kas kõik tundub õige ja võid vajutada install.

<img width="512" height="384" alt="Prox6" src="https://github.com/user-attachments/assets/3d2ef627-1265-4cc8-80c7-8193f3ed35e9" />
