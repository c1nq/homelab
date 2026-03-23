# Windows Server 2022 — Active Directory, DNS & DHCP

## VM Specificaties
| Instelling | Waarde |
|-----------|--------|
| VM ID | 102 |
| CPU | 4 cores |
| RAM | 4096 MB |
| Disk | 60 GB (storage-7tb) |
| IP | 192.168.1.102 (statisch) |
| Versie | Windows Server 2022 Standard Evaluation |

## Geïnstalleerde Roles
- Active Directory Domain Services (AD DS)
- DNS Server
- DHCP Server

## Active Directory
| Instelling | Waarde |
|-----------|--------|
| Domein | homelab.local |
| Forest level | Windows Server 2016 |
| Domain level | Windows Server 2016 |
| Domain Controller | WIN-KRSGURJU6CJ |

## Organizational Units
| OU | Beschrijving |
|----|--------------|
| Medewerkers | Domeingebruikers |
| Werkstations | Domeincomputers |
| IT-Beheer | IT administrators |

## Gebruikers
| Naam | UPN | OU |
|------|-----|----|
| Sem Enkelmans | s.enkelmans@homelab.local | Medewerkers |

## DNS
- Primaire zone: homelab.local
- DNS server IP: 192.168.1.102

## DHCP Scope
| Instelling | Waarde |
|-----------|--------|
| Scope naam | homelab |
| Start IP | 192.168.1.150 |
| Eind IP | 192.168.1.200 |
| Subnet mask | 255.255.255.0 |
| Gateway | 192.168.1.254 |
| DNS server | 192.168.1.102 |
| Lease tijd | 8 dagen |

## VirtIO Drivers
Windows Server 2022 in Proxmox vereist VirtIO drivers voor schijf en netwerk.
`
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
`
Installeer via virtio-win-guest-tools.exe na installatie.
