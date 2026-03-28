# Ubuntu Server 24.04 — Docker + Ansible + Monitoring

## VM Specificaties
| Instelling | Waarde |
|-----------|--------|
| VM ID | 103 |
| CPU | 4 cores |
| RAM | 4096 MB |
| Disk | 40 GB (storage-7tb) |
| IP | 192.168.1.103 (statisch) |
| OS | Ubuntu 24.04 LTS |

## Geïnstalleerde software
- Docker + Docker Compose v5.1.1
- Ansible 2.16.3
- QEMU Guest Agent

## Draaiende Docker containers

| Container | Poort | Functie |
|-----------|-------|---------|
| Portainer | 9000 | Docker beheer GUI |
| Prometheus | 9090 | Metrics collectie |
| Grafana | 3000 | Monitoring dashboard |
| Node Exporter | 9100 | Server metrics |
| Wazuh Manager | 1514/1515 | SIEM agent ontvanger |
| Wazuh Indexer | 9200 | Log opslag (OpenSearch) |
| Wazuh Dashboard | 443 | SIEM webinterface |

## Docker Compose — Monitoring stack
```yaml
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: always
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=<wachtwoord>
    volumes:
      - grafana_data:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    restart: always
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro

volumes:
  grafana_data:
```

## Statisch IP instellen
```bash
# /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens18:
      dhcp4: no
      addresses:
        - 192.168.1.103/24
      routes:
        - to: default
          via: 192.168.1.254
      nameservers:
        addresses:
          - 192.168.1.102
          - 8.8.8.8
  version: 2
```
