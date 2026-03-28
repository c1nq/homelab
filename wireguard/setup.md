# WireGuard VPN

## Installatie op Ubuntu Server
```bash
sudo apt install wireguard -y
sudo bash -c "wg genkey | tee /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key"
sudo chmod 600 /etc/wireguard/private.key
```

## Server configuratie /etc/wireguard/wg0.conf
```ini
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
```

## Client configuratie
```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.0.0.2/24
DNS = 192.168.1.102

[Peer]
PublicKey = <server-public-key>
Endpoint = 192.168.1.103:51820
AllowedIPs = 192.168.1.0/24, 10.0.0.0/24
PersistentKeepalive = 25
```

## Starten
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
sudo wg show
```
