<h1 align="center">ansible-server-bootstrap</h1>

<p align="center">
  Automate Ubuntu server setup from zero to production-ready — one command, fully idempotent.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-EE0000?style=flat-square&logo=ansible&logoColor=white" />
  <img src="https://img.shields.io/badge/Ubuntu-22.04-E95420?style=flat-square&logo=ubuntu&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?style=flat-square&logo=nginx&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Prometheus-Node%20Exporter-E6522C?style=flat-square&logo=prometheus&logoColor=white" />
  <img src="https://img.shields.io/badge/Vagrant-VirtualBox-1868F2?style=flat-square&logo=vagrant&logoColor=white" />
</p>

---

## What This Does

Run one command and get a hardened, monitored, production-ready Ubuntu server with [go-uptime-monitor](https://github.com/egayurcel990/go-uptime-monitor) deployed on top.

```bash
ansible-playbook -i inventory/hosts.yml site.yml
```

---

## Automation Coverage

| Role | What it does |
|---|---|
| `common` | Update packages, set timezone, configure hostname |
| `ssh-hardening` | Disable root login, change default port, enforce key-based auth, configure fail2ban |
| `ufw` | Enable firewall, whitelist SSH, HTTP, HTTPS, Node Exporter |
| `nginx` | Install Nginx, configure as reverse proxy for the app |
| `docker` | Install Docker + Compose, add deploy user to docker group |
| `node-exporter` | Install Prometheus Node Exporter as systemd service |
| `app-deploy` | Pull go-uptime-monitor image, run via Docker Compose, wire Nginx upstream |

---

## Architecture

```
Ansible Control Node (your machine)
            │
            │ SSH
            ▼
┌──────────────────────────────────┐
│        Ubuntu 22.04 Server       │
│                                  │
│  UFW Firewall                    │
│  ├─ Port 2222  (SSH hardened)    │
│  ├─ Port 80    (HTTP → Nginx)    │
│  └─ Port 9100  (Node Exporter)   │
│                                  │
│  Nginx (reverse proxy)           │
│  └─ :80 → go-uptime-monitor:8080 │
│                                  │
│  Docker                          │
│  └─ go-uptime-monitor container  │
│                                  │
│  Prometheus Node Exporter        │
│  └─ exposes /metrics :9100       │
└──────────────────────────────────┘
```

---

## Quick Start

**Prerequisites:** Ansible, VirtualBox, Vagrant

```bash
git clone https://github.com/egayurcel990/ansible-server-bootstrap
cd ansible-server-bootstrap

# Spin up test VM
vagrant up

# Run full bootstrap
ansible-playbook -i inventory/hosts.yml site.yml

# Run specific role only
ansible-playbook -i inventory/hosts.yml site.yml --tags ssh
ansible-playbook -i inventory/hosts.yml site.yml --tags nginx
ansible-playbook -i inventory/hosts.yml site.yml --tags app
```

---

## Project Structure

```
ansible-server-bootstrap/
├── inventory/
│   ├── hosts.yml                   # Target hosts
│   └── group_vars/
│       └── all.yml                 # Shared variables
├── roles/
│   ├── common/
│   │   ├── tasks/main.yml
│   │   └── defaults/main.yml
│   ├── ssh-hardening/
│   │   ├── tasks/main.yml
│   │   └── templates/sshd_config.j2
│   ├── ufw/
│   │   └── tasks/main.yml
│   ├── nginx/
│   │   ├── tasks/main.yml
│   │   └── templates/app.conf.j2
│   ├── docker/
│   │   └── tasks/main.yml
│   ├── node-exporter/
│   │   ├── tasks/main.yml
│   │   └── templates/node-exporter.service.j2
│   └── app-deploy/
│       ├── tasks/main.yml
│       └── templates/docker-compose.yml.j2
├── site.yml                        # Master playbook
├── Vagrantfile                     # Local test VM (Ubuntu 22.04)
└── README.md
```

---

## Configuration

Edit `inventory/group_vars/all.yml`:

```yaml
# SSH
ssh_port: 2222
ssh_user: deploy

# App
app_image: ghcr.io/egayurcel990/go-uptime-monitor:latest
app_port: 8080
app_domain: localhost

# Monitoring
node_exporter_port: 9100
```

---

## Testing Locally with Vagrant

```bash
vagrant up                          # Start Ubuntu 22.04 VM
ansible-playbook -i inventory/hosts.yml site.yml   # Bootstrap
vagrant ssh                         # Verify manually
vagrant destroy -f && vagrant up    # Full clean rebuild
```

---

## Extending to Real Servers

Swap inventory to target a real VPS:

```yaml
# inventory/hosts.yml
all:
  hosts:
    myserver:
      ansible_host: YOUR_SERVER_IP
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

Works with AWS EC2, Oracle Cloud, or any Ubuntu 22.04 VPS — same playbook, zero changes.

---

<p align="center">
  <i>Deploys <a href="https://github.com/egayurcel990/go-uptime-monitor">go-uptime-monitor</a> · Universitas Brawijaya · 2025</i>
</p>
