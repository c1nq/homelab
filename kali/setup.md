# Kali Linux — Security Testing VM

## VM Specificaties
| Instelling | Waarde |
|-----------|--------|
| VM ID | 104 |
| CPU | 2 cores |
| RAM | 2048 MB |
| Disk | 40 GB (storage-7tb) |
| IP | 192.168.1.104 (statisch) |
| OS | Kali Linux Rolling 2025.4 |

## Geïnstalleerde tools
- Nmap 7.98
- Metasploit Framework 6.4.116
- Kali Linux Default toolset

## Statisch IP instellen
```bash
sudo ip link set eth0 up
sudo ip addr add 192.168.1.104/24 dev eth0
sudo ip route add default via 192.168.1.254
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Permanent via /etc/network/interfaces
auto eth0
iface eth0 inet static
    address 192.168.1.104
    netmask 255.255.255.0
    gateway 192.168.1.254
    dns-nameservers 8.8.8.8
```

## APT sources instellen
```bash
echo "deb http://http.kali.org/kali kali-rolling main non-free contrib" | sudo tee /etc/apt/sources.list
sudo apt update
sudo apt install -y kali-linux-default
```

## Netwerk scan uitgevoerd

Zie [nmap-scan.md](nmap-scan.md) voor de resultaten van de netwerk discovery scan.
