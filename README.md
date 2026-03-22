# 🖥️ Homelab — Proxmox VE op Enterprise Hardware

![Proxmox](https://img.shields.io/badge/Proxmox-VE%209.1.1-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-Catalyst%203560-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![pfSense](https://img.shields.io/badge/pfSense-2.6.0-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![TrueNAS](https://img.shields.io/badge/TrueNAS-SCALE%2024.04-0095D5?style=for-the-badge&logo=truenas&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-Trixie-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)

> **Vista College — Niveau 4 System Engineer**  
> Homelab opgebouwd op een Intel enterprise rack server met RAID storage, VLAN segmentatie, pfSense firewall, TrueNAS NAS, Windows Server 2022 met Active Directory en Proxmox VE als hypervisor.

---

## 📦 Hardware

| Component | Model |
|-----------|-------|
| 🖥️ Server | Intel R2312IP4LHPC (Nemko R2000 chassis) |
| 🔌 Moederbord | Intel S2600IP dual-socket (1x CPU) |
| 🧠 RAM | DDR3 ECC Registered |
| 💾 RAID controller | Broadcom/LSI MegaRAID SAS 2208 |
| 🔀 Switch | Cisco Catalyst 3560 48-port FastEthernet |
| 🌐 Router | Netgear (192.168.1.254) |

---

## 💾 Schijfopstelling

| Slot | Grootte | RAID Array | Status |
|------|---------|------------|--------|
| 0 | 2.728 TB | RAID-6 — VD1 | ✅ Online |
| 1 | 2.728 TB | RAID-6 — VD1 | ✅ Online |
| 2 | 2.728 TB | RAID-6 — VD1 | ✅ Online |
| 3 | 931 GB | RAID-1 — VD0 | ✅ Online |
| 4 | 931 GB | RAID-1 — VD0 | ✅ Online |
| 5 | 2.728 TB | — | ⏳ Unconfigured |
| 6 | 1.819 TB | RAID-6 — VD2 | ✅ Online |
| 7 | 1.819 TB | RAID-6 — VD2 | ✅ Online |
| 8 | 1.819 TB | RAID-6 — VD2 | ✅ Online |
| 9 | 1.819 TB | RAID-6 — VD2 | ✅ Online |
| 10 | 1.819 TB | RAID-6 — VD2 | ✅ Online |
| 11 | 1.819 TB | RAID-6 — VD2 | ✅ Online |

---

## 🌐 Netwerk
```
Internet
    │
    ▼
Netgear Router ── 192.168.1.254
    │
    ▼
Cisco Catalyst 3560 ── 192.168.1.2
    ├── Fa0/1 ──► Proxmox Server   (trunk: VLAN 1, 10, 20, 30)
    ├── Fa0/2 ──► Laptop
    └── Fa0/3 ──► Netgear Router   (trunk: VLAN 1, 10, 20, 30)
```

### VLAN Schema

| VLAN | Naam | Subnet | Doel |
|------|------|--------|------|
| 1 | default | 192.168.1.0/24 | Beheer / bestaand netwerk |
| 10 | management | 192.168.10.0/24 | Proxmox beheer |
| 20 | servers | 192.168.20.0/24 | Virtuele machines |
| 30 | storage | 192.168.30.0/24 | NAS / backup |

### VM / Service IP-schema

| Host | IP | Functie |
|------|----|---------|
| Proxmox | 192.168.1.100 | Hypervisor |
| pfSense WAN | 192.168.1.101 | Firewall beheer |
| pfSense LAN | 192.168.20.1 | Gateway VLAN 20 |
| TrueNAS | 192.168.1.110 | NAS / SMB share |
| Windows Server | 192.168.1.102 | AD, DNS, DHCP |
| Cisco switch | 192.168.1.2 | Netwerk beheer |

---

## ⚙️ Configuratie

### Proxmox — `/etc/network/interfaces`
```bash
auto vmbr0
iface vmbr0 inet static
        address 192.168.1.100/24
        gateway 192.168.1.254
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0

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

---

### MegaCLI — Schijven zichtbaar maken
```bash
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libtinfo5_6.4-4_amd64.deb
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libncurses5_6.4-4_amd64.deb
dpkg -i libtinfo5_6.4-4_amd64.deb
dpkg -i libncurses5_6.4-4_amd64.deb

/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL | grep -E "Slot Number|Raw Size|Firmware state"
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL
```

---

### Storage — Mounten in Proxmox
```bash
mkfs.ext4 /dev/sdb
mkfs.ext4 /dev/sdc
mkdir -p /mnt/storage-2tb
mkdir -p /mnt/storage-7tb
echo "UUID=$(blkid -s UUID -o value /dev/sdb)  /mnt/storage-2tb  ext4  defaults  0  2" >> /etc/fstab
echo "UUID=$(blkid -s UUID -o value /dev/sdc)  /mnt/storage-7tb  ext4  defaults  0  2" >> /etc/fstab
systemctl daemon-reload && mount -a
```

| Storage ID | Pad | Capaciteit |
|------------|-----|------------|
| storage-2tb | /mnt/storage-2tb | 2.6 TB |
| storage-7tb | /mnt/storage-7tb | 6.9 TB |

---

### Cisco Catalyst 3560 — VLANs & Trunks
```
conf t
vlan 10
 name management
vlan 20
 name servers
vlan 30
 name storage
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

SSH verbinden vanaf Windows met IOS 12.2:
```bash
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 \
    -oHostKeyAlgorithms=+ssh-rsa \
    -oCiphers=+aes256-cbc \
    -oMACs=+hmac-sha1 \
    admin@192.168.1.2
```

---

### pfSense — Firewall VM

| Instelling | Waarde |
|-----------|--------|
| VM ID | 100 |
| CPU | 2 cores |
| RAM | 2048 MB |
| Disk | 20 GB (storage-7tb) |
| WAN | 192.168.1.101 (vmbr0) |
| LAN | 192.168.20.1 (vmbr0) |
| Webinterface | http://192.168.1.101 |
| Versie | pfSense CE 2.6.0 |

---

### TrueNAS SCALE — NAS VM

| Instelling | Waarde |
|-----------|--------|
| VM ID | 101 |
| CPU | 4 cores |
| RAM | 8192 MB |
| Boot disk | 50 GB (storage-7tb) |
| Data disks | 2x 500 GB Mirror (ZFS) |
| IP | 192.168.1.110 |
| Webinterface | http://192.168.1.110 |
| Versie | TrueNAS SCALE 24.04 |
| SMB share | \\192.168.1.110\data |
| Pool | datapool — ZFS Mirror, 480 GB beschikbaar |

---

### Windows Server 2022 — AD/DNS/DHCP VM

| Instelling | Waarde |
|-----------|--------|
| VM ID | 102 |
| CPU | 4 cores |
| RAM | 4096 MB |
| Disk | 60 GB (storage-7tb) |
| IP | 192.168.1.102 |
| Versie | Windows Server 2022 Standard Evaluation |
| Domein | homelab.local |

**Active Directory configuratie:**
- Forest/Domain: `homelab.local`
- Functional level: Windows Server 2016
- OUs: `Medewerkers`, `Werkstations`, `IT-Beheer`
- Gebruiker: `s.enkelmans@homelab.local`

**DHCP scope:**
- Range: `192.168.1.150 — 192.168.1.200`
- Gateway: `192.168.1.254`
- DNS: `192.168.1.102`

---

## ✅ Voortgang

- [x] Proxmox VE 9.1.1 geïnstalleerd
- [x] Repository geconfigureerd (no-subscription)
- [x] MegaCLI64 werkend — alle 12 schijven zichtbaar
- [x] RAID arrays geïdentificeerd (RAID-1 + 2x RAID-6)
- [x] 2.7 TB + 7.3 TB storage gemount en toegevoegd aan Proxmox
- [x] Cisco 3560 geconfigureerd met VLAN 10 / 20 / 30
- [x] Trunk poorten actief op Fa0/1 en Fa0/3
- [x] SSH toegang op Cisco switch
- [x] Proxmox VLAN interfaces actief (vmbr0.10 / .20 / .30)
- [x] pfSense VM geïnstalleerd en geconfigureerd
- [x] pfSense webinterface bereikbaar via WAN
- [x] TrueNAS SCALE VM geïnstalleerd
- [x] ZFS Mirror pool aangemaakt (2x 500GB)
- [x] SMB share actief en bereikbaar vanaf Windows
- [x] Windows Server 2022 VM geïnstalleerd
- [x] Active Directory domein `homelab.local` opgezet
- [x] DNS server geconfigureerd
- [x] DHCP scope aangemaakt en actief
- [x] Gebruiker `s.enkelmans@homelab.local` aangemaakt
- [ ] Ubuntu Server met Ansible + Docker
- [ ] Grafana + Prometheus monitoring
- [ ] Wazuh SIEM
- [ ] Kali Linux VM
- [ ] WireGuard VPN
- [ ] Gitea + CI/CD pipeline

---

*Vista College — Niveau 4 System Engineer | 2026*
