# Wazuh SIEM — Security Information and Event Management

## Installatie via Docker
```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.0 --depth 1
cd wazuh-docker/single-node
docker compose -f generate-indexer-certs.yml run --rm generator
docker compose up -d
```

## Webinterface
- URL: https://192.168.1.103
- Username: admin
- Password: SecretPassword

## Actieve agents

| Agent ID | Naam | IP | OS | Status |
|----------|------|----|----|--------|
| 001 | ubuntu-server | 192.168.1.103 | Ubuntu 24.04 LTS | ✅ Active |
| 002 | WIN-KRSGURJU6CJ | 192.168.1.102 | Windows Server 2022 | ✅ Active |

## Agent installatie — Ubuntu
```bash
curl -o wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.0-1_amd64.deb
sudo WAZUH_MANAGER='192.168.1.103' dpkg -i wazuh-agent.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

## Agent installatie — Windows Server
```powershell
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi" -OutFile "C:\wazuh-agent.msi" -UseBasicParsing
msiexec /i C:\wazuh-agent.msi /q WAZUH_MANAGER="192.168.1.103"
NET START WazuhSvc
```

## Modules actief
- Security events
- Integrity monitoring (FIM)
- System auditing
- Policy monitoring (SCA)
- Vulnerability detection
- MITRE ATT&CK
- PCI DSS compliance
- NIST 800-53 compliance
