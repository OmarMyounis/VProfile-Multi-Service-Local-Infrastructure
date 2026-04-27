# VProfile — Multi-Tier Application Stack

> A fully automated, multi-VM local infrastructure project provisioned with Vagrant and VirtualBox. Simulates a production-grade service stack with five isolated VMs, static private networking, and automated shell-based provisioning.

[![Stack](https://img.shields.io/badge/IaC-Vagrant-1563FF?style=flat-square)](https://www.vagrantup.com/)
[![OS](https://img.shields.io/badge/OS-CentOS%20Stream%209%20%7C%20Ubuntu%2022.04-informational?style=flat-square)]()
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

---

## Architecture Overview

```
                        ┌─────────────────────────────────┐
                        │        Host Machine (Browser)    │
                        └────────────────┬────────────────┘
                                         │ HTTP :80
                              ┌──────────▼──────────┐
                              │   Nginx  (web01)     │
                              │   192.168.56.11      │
                              │   Ubuntu Jammy 64    │
                              └──────────┬──────────┘
                                         │ Reverse Proxy → :8080
                              ┌──────────▼──────────┐
                              │  Tomcat  (app01)     │
                              │  192.168.56.12       │
                              │  CentOS Stream 9     │
                              └──────┬───────┬───────┘
                                     │       │
                   ┌─────────────────┘       └──────────────────┐
                   │ :3306                                :5672  │
       ┌───────────▼──────────┐               ┌─────────────────▼──┐
       │  MariaDB  (db01)     │               │  RabbitMQ (rmq01)   │
       │  192.168.56.15       │               │  192.168.56.13      │
       │  CentOS Stream 9     │               │  CentOS Stream 9    │
       └──────────────────────┘               └────────────────────┘
                   │ :11211
       ┌───────────▼──────────┐
       │ Memcached (mc01)     │
       │ 192.168.56.14        │
       │ CentOS Stream 9      │
       └──────────────────────┘
```

---

## Tech Stack

| VM Name | Role | Service | OS | Private IP | RAM |
|---------|------|---------|-----|------------|-----|
| `web01` | Web / Reverse Proxy | Nginx | Ubuntu 22.04 (Jammy) | 192.168.56.11 | 800 MB |
| `app01` | Application Server | Apache Tomcat | CentOS Stream 9 | 192.168.56.12 | 4200 MB |
| `rmq01` | Message Broker | RabbitMQ | CentOS Stream 9 | 192.168.56.13 | 600 MB |
| `mc01` | Cache Layer | Memcached | CentOS Stream 9 | 192.168.56.14 | 900 MB |
| `db01` | Database | MariaDB | CentOS Stream 9 | 192.168.56.15 | 2048 MB |

**Total RAM required:** ~8.5 GB (12 GB+ recommended on host)

---

## Prerequisites

Ensure the following are installed and configured on your host machine before proceeding.

| Requirement | Notes |
|-------------|-------|
| [Vagrant](https://www.vagrantup.com/) >= 2.2.x | Infrastructure provisioning tool |
| [VirtualBox](https://www.virtualbox.org/) (latest stable) | Hypervisor provider |
| `vagrant-hostmanager` plugin | Manages `/etc/hosts` for VM hostnames |
| Virtualization enabled in BIOS | VT-x (Intel) or AMD-V required |
| ~12 GB free RAM | For running all 5 VMs concurrently |
| ~20 GB free disk space | For VM boxes and provisioning artifacts |

Install the required Vagrant plugin:

```bash
vagrant plugin install vagrant-hostmanager
```

---

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/vprofile-project.git
cd vprofile-project

# 2. Install the Vagrant plugin (first time only)
vagrant plugin install vagrant-hostmanager

# 3. Boot the full stack
vagrant up
```

> On first run, Vagrant downloads the base boxes and runs all provisioning scripts automatically. This may take 10–20 minutes depending on your internet speed.

To bring up a single VM only:

```bash
vagrant up db01
```

---

## Accessing Services

Once `vagrant up` completes successfully:

| Service | URL / Connection |
|---------|-----------------|
| Web Application | http://192.168.56.11 |
| Tomcat (direct) | http://192.168.56.12:8080 |
| RabbitMQ Management UI | http://192.168.56.13:15672 |
| MariaDB | `mysql -h 192.168.56.15 -u root -p` |
| Memcached | `telnet 192.168.56.14 11211` |

---

## How Provisioning Works

Each VM is automatically provisioned on first boot using a dedicated shell script:

```
Vagrantfile
├── db01  ──▶  mariadb.sh    (installs and configures MariaDB)
├── mc01  ──▶  memcached.sh  (installs and configures Memcached)
├── rmq01 ──▶  rabbitmq.sh   (installs and configures RabbitMQ)
├── app01 ──▶  tomcat.sh     (installs Java, Tomcat, deploys app)
└── web01 ──▶  nginx.sh      (installs Nginx, sets reverse proxy)
```

Provisioning scripts run with `privileged: true` (root). Hostname resolution between VMs is handled automatically by the `vagrant-hostmanager` plugin, which writes all VM hostnames to `/etc/hosts` on each machine.

---

## Common Commands

```bash
vagrant up              # Start all VMs
vagrant halt            # Stop all VMs gracefully
vagrant destroy         # Remove all VMs and their disks
vagrant ssh app01       # SSH into the Tomcat VM
vagrant reload db01     # Restart and re-provision db01
vagrant provision       # Re-run provisioning scripts on running VMs
vagrant status          # Show state of all VMs
```

---

## Troubleshooting

See [docs/troubleshooting.md](docs/troubleshooting.md) for detailed fixes.

| Problem | Quick Fix |
|---------|-----------|
| VirtualBox kernel module errors on Linux | `sudo apt install virtualbox-dkms && reboot` |
| `vagrant-hostmanager` sudo prompt | Enter host password when prompted, or add IPs to `/etc/hosts` manually |
| VM fails to boot | Run `vagrant destroy <vm> && vagrant up <vm>` |
| Provisioning script fails mid-run | SSH into the VM and check `/var/log/` |
| App unreachable after boot | Wait 60s and retry — Tomcat startup takes time |

---

## Project Structure

```
vprofile-project/
├── Vagrantfile          # Defines all 5 VMs, networking, and provisioning hooks
├── mariadb.sh           # Provisions db01 — MariaDB setup and configuration
├── memcached.sh         # Provisions mc01 — Memcached setup
├── rabbitmq.sh          # Provisions rmq01 — RabbitMQ setup
├── tomcat.sh            # Provisions app01 — Java + Tomcat + app deployment
├── nginx.sh             # Provisions web01 — Nginx reverse proxy config
├── docs/
│   ├── architecture.md  # Detailed architecture and design decisions
│   ├── mariadb.md       # MariaDB service reference
│   ├── tomcat.md        # Tomcat service reference
│   ├── nginx.md         # Nginx service reference
│   ├── rabbitmq.md      # RabbitMQ service reference
│   ├── memcached.md     # Memcached service reference
│   └── troubleshooting.md
└── CHANGELOG.md
```

---

## Roadmap

- [ ] Migrate shell provisioning to **Ansible playbooks**
- [ ] Add **Docker Compose** version for container-based deployment
- [ ] Integrate **GitHub Actions** CI pipeline with ShellCheck linting
- [ ] Add **Prometheus + Grafana** monitoring layer
- [ ] Add port-forwarding to Vagrantfile for localhost access

---
