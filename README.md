# 🖥️ Homelab — Proxmox VE op Enterprise Hardware

![Proxmox](https://img.shields.io/badge/Proxmox-VE%209.1.1-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Cisco](https://img.shields.io/badge/Cisco-Catalyst%203560-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![pfSense](https://img.shields.io/badge/pfSense-2.6.0-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-Trixie-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)

> **Vista College — Niveau 4 System Engineer**  
> Homelab opgebouwd op een Intel enterprise rack server met RAID storage, VLAN segmentatie, pfSense firewall en Proxmox VE als hypervisor.

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

Proxmox VE 9 draait op Debian Trixie waar `libncurses5` niet meer beschikbaar is. Oplossing: handmatig installeren van Debian Bookworm.
```bash
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libtinfo5_6.4-4_amd64.deb
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libncurses5_6.4-4_amd64.deb
dpkg -i libtinfo5_6.4-4_amd64.deb
dpkg -i libncurses5_6.4-4_amd64.deb

# Alle physical drives weergeven
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL | grep -E "Slot Number|Raw Size|Firmware state"

# RAID arrays weergeven
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
| LAN | 192.168.20.1 (vmbr0, VLAN 20) |
| Webinterface | http://192.168.1.101 |
| Versie | pfSense CE 2.6.0 |

**Firewall regel voor WAN toegang webinterface:**
- Action: Pass
- Interface: WAN
- Protocol: TCP
- Destination: WAN address, poort 80 + 443

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
- [ ] TrueNAS VM aanmaken
- [ ] Windows Server 2022 VM
- [ ] Active Directory + DNS + DHCP
- [ ] Ubuntu Server met Ansible + Docker
- [ ] Grafana + Prometheus monitoring
- [ ] Wazuh SIEM
- [ ] Kali Linux VM
- [ ] WireGuard VPN
- [ ] Gitea + CI/CD pipeline

---

*Vista College — Niveau 4 System Engineer | 2026*
