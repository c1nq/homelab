# Nmap Netwerk Scan — Homelab

**Tool:** Nmap 7.98  
**Scope:** 192.168.1.0/24 (intern homelab netwerk)  
**Doel:** Network discovery en service detectie  
**Datum:** Maart 2026

---

## Gevonden hosts (19 actief)

| Host | Type | Open poorten | OS |
|------|------|-------------|-----|
| switch | Cisco Catalyst 3560 | 22, 80, 443 | Cisco IOS |
| access-point-1 | TP-Link WAP | 80, 443 | OpenWrt Linux |
| access-point-2 | Ubiquiti | 22, 8080 | Linux |
| camera | IP camera | 53, 554 (RTSP) | Linux |
| smart-device-1 | Apple device | 49152, 62078 | — |
| chromecast | Google Chromecast | 8008, 8009, 8443 | — |
| smart-device-2 | Resideo | — | Linux |
| smart-device-3 | Apple device | 49152, 62078 | — |
| smart-device-4 | eQ-3 | — | — |
| proxmox | Proxmox VE | 22, 111, 3128 | Debian Linux |
| pfsense | pfSense firewall | gefilterd | FreeBSD |
| windows-server | Windows Server 2022 | 22, 53, 88, 135, 389, 445 | Windows |
| ubuntu-server | Ubuntu 24.04 | 22, 443, 3000, 9000, 9090, 9200 | Linux |
| truenas | TrueNAS SCALE | 80, 111, 139, 443, 445 | Linux |
| router | Netgear router | 80, 443, 8099 | Linux |
| kali | Kali Linux | 22 | Debian Linux |

---

## Interessante bevindingen

**Windows Server (AD/DNS/DHCP):**
- Kerberos (88), LDAP (389), SMB (445) — Active Directory volledig operationeel
- Domein: homelab.local zichtbaar via LDAP

**Ubuntu Server (monitoring stack):**
- Grafana (3000), Prometheus (9090), Portainer (9000), Wazuh (443) — volledig operationeel

**Netwerk apparaten:**
- Cisco switch bereikbaar via SSH en HTTP management interface
- IP camera met RTSP stream gedetecteerd
- Meerdere IoT apparaten (Chromecast, Apple devices, smart home)

---

## Gebruikte commando's
```bash
# Volledige netwerk scan met service detectie
nmap -sV 192.168.1.0/24 -oN scan-results.txt

# Specifieke host scan
nmap -sV -p- 192.168.1.x

# OS detectie
nmap -O 192.168.1.x
```

---

*Uitgevoerd op eigen homelab netwerk voor educatieve doeleinden.*