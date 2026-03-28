# Gitea — Git Server + CI/CD

## Installatie via Docker
```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    ports:
      - "3001:3000"
      - "222:22"
    volumes:
      - gitea_data:/data
    environment:
      - USER_UID=1000
      - USER_GID=1000

volumes:
  gitea_data:
```

## Webinterface
- URL: http://192.168.1.103:3001

## CI/CD Runner installeren
```bash
docker run -d \
  --name gitea-runner \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/gitea-runner:/data \
  -e GITEA_INSTANCE_URL=http://192.168.1.103:3001 \
  -e GITEA_RUNNER_REGISTRATION_TOKEN=<token> \
  -e GITEA_RUNNER_NAME=homelab-runner \
  gitea/act_runner:latest
```

## Voorbeeld workflow .gitea/workflows/ci.yml
```yaml
name: Homelab CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run test
        run: |
          echo "CI Pipeline draait!"
          echo "Branch: ${{ github.ref }}"
          echo "Commit: ${{ github.sha }}"
```
