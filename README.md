# DevSecOps Test Task

Ansible deployment of Nginx + ModSecurity (OWASP CRS) with centralized logging.

## Hosts

| Host | IP | VPN | Role |
|---|---|---|---|
| ubu1 | 192.168.56.11 | 10.10.0.3 | Web + WAF, Grafana Alloy |
| ubu2 | 192.168.56.12 | 10.10.0.1 | WireGuard, Loki, Grafana, Ansible |

## Security

- UFW default deny; SSH key-only, no root
- SSH to ubu1 only via VPN
- Loki and Grafana reachable only via VPN

## Repository layout

```
site.yml, inventory.ini, ansible.cfg   Ansible entry point
roles/docker/                          docker from the official Docker repo
roles/web_waf/                         WAF, demo site, Alloy
  defaults/    images, paranoia level, Loki URL
  templates/   docker-compose.yml.j2, config.alloy.j2
  files/       static website
  handlers/    recreate containers on config change
monitoring/                            Loki + Grafana for ubu2
```

## Usage

Run ansible from ubu2 (ubu1 is reached via VPN at `10.10.0.3`):

```bash
ansible web -m ping
ansible-playbook site.yml -K
```

Monitoring stack on ubu2:

```bash
cd monitoring && cp .env.example .env   # set Grafana admin password
docker compose up -d
```

## Test

```bash
curl -i http://192.168.56.11/                                  # 200
curl -i "http://192.168.56.11/?id=1%27%20OR%20%271%27=%271"    # 403
```

Grafana `http://10.10.0.1:3000` → Explore:

```
{job="nginx", type="access"}
{job="modsecurity"} |= "ERROR_ID"
```

