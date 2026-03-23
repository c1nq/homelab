# TrueNAS SCALE Configuratie

## VM Specificaties
| Instelling | Waarde |
|-----------|--------|
| VM ID | 101 |
| CPU | 4 cores |
| RAM | 8192 MB |
| Boot disk | 50 GB (storage-7tb) |
| IP | 192.168.1.110 |
| Versie | TrueNAS SCALE Dragonfish 24.04.2 |
| Webinterface | http://192.168.1.110 |

## ZFS Pool
| Naam | Type | Schijven | Capaciteit | Status |
|------|------|----------|------------|--------|
| datapool | Mirror (RAID-1) | 2x 500 GB | 480 GiB | Online |

## Datasets
| Dataset | Path | Type |
|---------|------|------|
| data | datapool/data | Filesystem |

## SMB Share
| Naam | Path | Toegang |
|------|------|---------|
| data | /mnt/datapool/data | Alle gebruikers |

## Verbinden via Windows Verkenner
`
\\192.168.1.110\data
`
Login met lokale TrueNAS gebruiker.
