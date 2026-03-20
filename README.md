🖥️ Homelab — Intel R2312IP4LHPC Proxmox Server
> Vista College — Niveau 4 System Engineer  
> Project: Homelab opzetten met enterprise hardware
---
📋 Projectoverzicht
Dit project beschrijft het opzetten van een volledig functioneel homelab op enterprise server hardware. Van bare metal tot werkende Proxmox VE installatie met RAID storage, netwerk segmentatie via VLANs en VM-ready infrastructuur.
---
🔧 Hardware
Component	Specificatie
Server chassis	Intel R2312IP4LHPC (Nemko R2000)
Moederbord	Intel S2600IP dual-socket (1x CPU geplaatst)
RAM	DDR3 ECC Registered (blauwe slots)
RAID controller	Broadcom/LSI MegaRAID SAS 2208
Switch	Cisco Catalyst 3560 48-port FastEthernet
Router	Netgear (gateway: 192.168.1.254)
Schijfconfiguratie (achter RAID controller)
Slot	Grootte	RAID	Status
0	2.728 TB	RAID-6 (VD1)	Online
1	2.728 TB	RAID-6 (VD1)	Online
2	2.728 TB	RAID-6 (VD1)	Online
3	931 GB	RAID-1 (VD0)	Online
4	931 GB	RAID-1 (VD0)	Online
5	2.728 TB	—	Unconfigured
6	1.819 TB	RAID-6 (VD2)	Online
7	1.819 TB	RAID-6 (VD2)	Online
8	1.819 TB	RAID-6 (VD2)	Online
9	1.819 TB	RAID-6 (VD2)	Online
10	1.819 TB	RAID-6 (VD2)	Online
11	1.819 TB	RAID-6 (VD2)	Online
---
🌐 Netwerktopologie
```
Internet
   │
Netgear Router (192.168.1.254)
   │
Cisco Catalyst 3560 (192.168.1.2)
   ├── Fa0/1 → Proxmox Server (trunk: VLAN 1,10,20,30)
   ├── Fa0/2 → Laptop beheer
   └── Fa0/3 → Netgear Router (trunk: VLAN 1,10,20,30)
```
IP-schema
VLAN	Naam	Subnet	Gebruik
1	default	192.168.1.0/24	Beheer / bestaand netwerk
10	management	192.168.10.0/24	Proxmox beheer
20	servers	192.168.20.0/24	VMs
30	storage	192.168.30.0/24	NAS / backup
---
🚀 Installatiestappen
1. Proxmox VE installeren
Proxmox VE 9.1.1 geïnstalleerd op `/dev/sda` (1TB, RAID-1 VD0)
Gateway geconfigureerd via `/etc/network/interfaces`
```bash
# /etc/network/interfaces
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.100/24
    gateway 192.168.1.254
    bridge-ports nic0
    bridge-stp off
    bridge-fd 0
```
Enterprise repository uitgeschakeld
No-subscription repository toegevoegd
```bash
# /etc/apt/sources.list.d/pve-no-subscription.list
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
```
---
2. MegaCLI installeren (RAID controller beheer)
MegaCLI64 is de beheertool voor de LSI MegaRAID SAS 2208 controller.  
Op Proxmox VE 9 (Debian Trixie) ontbreekt `libncurses5` — dit is opgelost door de `.deb` pakketten handmatig te installeren.
```bash
# Installeer benodigde libraries
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libtinfo5_6.4-4_amd64.deb
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libncurses5_6.4-4_amd64.deb
dpkg -i libtinfo5_6.4-4_amd64.deb
dpkg -i libncurses5_6.4-4_amd64.deb
```
```bash
# Alle schijven weergeven
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL | grep -E "Slot Number|Raw Size|Firmware state"

# RAID arrays weergeven
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL
```
---
3. Storage toevoegen aan Proxmox
De RAID controller presenteert de virtual drives als `/dev/sdb` (2.7TB) en `/dev/sdc` (7.3TB).
```bash
# Formatteren
mkfs.ext4 /dev/sdb
mkfs.ext4 /dev/sdc

# Mountpoints aanmaken
mkdir -p /mnt/storage-2tb
mkdir -p /mnt/storage-7tb

# UUIDs ophalen
blkid /dev/sdb
blkid /dev/sdc

# Toevoegen aan /etc/fstab voor automatisch mounten
echo "UUID=<uuid-sdb>  /mnt/storage-2tb  ext4  defaults  0  2" >> /etc/fstab
echo "UUID=<uuid-sdc>  /mnt/storage-7tb  ext4  defaults  0  2" >> /etc/fstab

# Mounten
systemctl daemon-reload
mount -a
```
Daarna toegevoegd aan Proxmox via:  
Datacenter → Storage → Add → Directory
ID	Path	Beschikbaar
storage-2tb	/mnt/storage-2tb	2.6 TB
storage-7tb	/mnt/storage-7tb	6.9 TB
---
4. VLANs configureren op Cisco Catalyst 3560
```
! VLANs aanmaken
conf t
vlan 10
 name management
vlan 20
 name servers
vlan 30
 name storage

! Trunk poorten instellen
interface Fa0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30

interface Fa0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30
end
write memory
```
SSH toegang ingesteld op de switch:
```
ip domain-name homelab.local
crypto key generate rsa modulus 1024
username admin privilege 15 secret <wachtwoord>
line vty 0 15
 transport input ssh
 login local
ip ssh version 2
```
Verbinden met oude IOS via moderne SSH client:
```bash
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oCiphers=+aes256-cbc \
    -oMACs=+hmac-sha1 \
    admin@192.168.1.2
```
---
5. VLAN interfaces in Proxmox
```bash
# /etc/network/interfaces — VLAN subinterfaces toevoegen
auto vmbr0.10
iface vmbr0.10 inet static
        address 192.168.10.1/24

auto vmbr0.20
iface vmbr0.20 inet static
        address 192.168.20.1/24

auto vmbr0.30
iface vmbr0.30 inet static
        address 192.168.30.1/24
```
```bash
systemctl restart networking
ip addr show
```
---
✅ Huidige status
[x] Proxmox VE 9.1.1 geïnstalleerd
[x] MegaCLI werkend — alle 12 schijven zichtbaar
[x] RAID arrays geïdentificeerd (RAID-1 + 2x RAID-6)
[x] 2.7 TB + 7.3 TB storage gemount in Proxmox
[x] Cisco 3560 geconfigureerd met VLAN 10/20/30
[x] Trunk poorten actief op Fa0/1 en Fa0/3
[x] SSH toegang op Cisco switch
[x] Proxmox VLAN interfaces actief
[ ] Eerste VM aanmaken
[ ] Slot 5 (2.728 TB) toevoegen als extra storage
[ ] DHCP per VLAN instellen
---
🔜 Volgende stappen
Eerste VM aanmaken op storage-7tb (VLAN 20)
Slot 5 configureren als extra RAID volume
DHCP server instellen per VLAN
---
📚 Gebruikte bronnen
Proxmox VE documentatie
MegaCLI referentie
Cisco IOS 12.2 configuratiehandleiding
---
Vista College — Niveau 4 System Engineer | 2026
