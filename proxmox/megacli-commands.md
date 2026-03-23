# MegaCLI Commando's

## Probleem
Proxmox VE 9 draait op Debian Trixie — libncurses5 is niet meer beschikbaar in de repos.

## Oplossing
`ash
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libtinfo5_6.4-4_amd64.deb
wget http://ftp.debian.org/debian/pool/main/n/ncurses/libncurses5_6.4-4_amd64.deb
dpkg -i libtinfo5_6.4-4_amd64.deb
dpkg -i libncurses5_6.4-4_amd64.deb
`

## Nuttige commando's
`ash
# Alle physical drives weergeven
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aALL | grep -E "Slot Number|Raw Size|Firmware state"

# RAID arrays weergeven
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL

# Aantal schijven
/opt/MegaRAID/MegaCli/MegaCli64 -PDGetNum -aALL
`

## RAID configuratie
| Virtual Drive | RAID Level | Grootte | Schijven | Status |
|---------------|-----------|---------|----------|--------|
| VD0 | RAID-1 | 930 GB | 2x 931 GB (slot 3,4) | Optimal |
| VD1 | RAID-6 | 2.727 TB | 3x 2.728 TB (slot 0,1,2) | Optimal |
| VD2 | RAID-6 | 7.271 TB | 6x 1.819 TB (slot 6-11) | Optimal |
