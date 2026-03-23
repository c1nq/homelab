# pfSense Firewall Configuratie

## VM Specificaties
| Instelling | Waarde |
|-----------|--------|
| VM ID | 100 |
| CPU | 2 cores |
| RAM | 2048 MB |
| Disk | 20 GB (storage-7tb) |
| WAN IP | 192.168.1.101 |
| LAN IP | 192.168.20.1 |
| Versie | pfSense CE 2.6.0 |
| Webinterface | http://192.168.1.101 |

## Interfaces
| Interface | Adapter | IP | Functie |
|-----------|---------|-----|---------|
| WAN | em0 | 192.168.1.101/24 | Uplink naar netwerk |
| LAN | em1 | 192.168.20.1/24 | Gateway VLAN 20 |

## WAN Firewall Regels
| Action | Protocol | Source | Destination | Port | Beschrijving |
|--------|----------|--------|-------------|------|--------------|
| Pass | TCP | any | WAN address | 80 | Allow HTTP webGUI |
| Pass | TCP | any | WAN address | 443 | Allow HTTPS webGUI |

## LAN DHCP
- Range: 192.168.20.100 — 192.168.20.200
- Gateway: 192.168.20.1
- DNS: 8.8.8.8, 8.8.4.4

## Noot
pfctl -d uitvoeren via console om tijdelijk firewall uit te schakelen bij lockout.
