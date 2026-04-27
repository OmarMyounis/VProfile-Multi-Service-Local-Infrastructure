# Vagrant Project — Multi-VM Development Environment

This repository provides a multi-VM Vagrant environment used to run a small services stack (database, cache, message broker, application server, and web server) for development and testing.

The environment is defined in the provided `Vagrantfile` and provisioned using shell scripts in the repository:

- `mariadb.sh` — provision the MariaDB / database VM (`db01`)
- `memcached.sh` — provision the Memcached VM (`mc01`)
- `rabbitmq.sh` — provision the RabbitMQ VM (`rmq01`)
- `tomcat.sh` — provision the application VM running Tomcat (`app01`)
- `nginx.sh` — provision the web/frontend VM (`web01`)

Summary of Vagrant machines (as defined in `Vagrantfile`):

- `db01` (eurolinux-vagrant/centos-stream-9) — private IP 192.168.56.15, memory: 2048 MB
- `mc01` (eurolinux-vagrant/centos-stream-9) — private IP 192.168.56.14, memory: 900 MB
- `rmq01` (eurolinux-vagrant/centos-stream-9) — private IP 192.168.56.13, memory: 600 MB
- `app01` (eurolinux-vagrant/centos-stream-9) — private IP 192.168.56.12, memory: 4200 MB
- `web01` (ubuntu/jammy64) — private IP 192.168.56.11, memory: 800 MB (GUI enabled in Vagrantfile)

The Vagrant configuration uses the `virtualbox` provider for all VMs and enables the `vagrant-hostmanager` plugin to manage host entries.

## Prerequisites (host machine)

Before running this environment on your workstation (Linux, macOS, or Windows), ensure the host machine meets these prerequisites:

- Virtualization support enabled in BIOS/UEFI (VT-x/AMD-V).
- Vagrant installed (recommended >= 2.2.x). Install from https://www.vagrantup.com/
- VirtualBox installed (recommended latest stable release). If you prefer another provider, update the `Vagrantfile` accordingly.
- Vagrant plugin `vagrant-hostmanager` (used by this Vagrantfile to update /etc/hosts). Install with:

	vagrant plugin install vagrant-hostmanager

- Sufficient RAM and disk space. The configured VMs request approximately 8.5 GB of RAM total (2048 + 900 + 600 + 4200 + 800). To run them all concurrently, a host with at least 12 GB RAM is recommended. Allow ~20 GB free disk space for boxes and provisioning artifacts.
- A working internet connection for the first `vagrant up` (to download boxes and packages during provisioning).
- On Linux hosts: you may need to install additional kernel modules for VirtualBox (e.g., `virtualbox-dkms`) and ensure you can run VirtualBox as your user.

Notes:
- If you cannot run all VMs concurrently due to host resource limits, start only the required VM(s) with `vagrant up <vm-name>` (examples below).
- The `vagrant-hostmanager` plugin will attempt to update your system hosts file. It may prompt for a password or require manual action on some systems. If it fails, you can add the private IPs to `/etc/hosts` manually.

## How to bring up the environment

1. Clone or open this repository and cd into the project directory.

2. Install prerequisites listed above (Vagrant, VirtualBox, `vagrant-hostmanager`).

3. Start the full environment:

	vagrant up

	- To force a specific provider (if multiple are installed), use `vagrant up --provider=virtualbox`.

	- To bring up a single machine only, specify its name (example):

		vagrant up db01

4. SSH into a machine:

	vagrant ssh db01

5. Common lifecycle commands:

	vagrant halt           # stop all VMs
	vagrant reload db01    # restart and re-provision db01
	vagrant destroy        # remove all VMs and their storage

## Verification and where to look

- After `vagrant up`, provisioning shell scripts run automatically. You can inspect each script in the repository (e.g., `mariadb.sh`, `nginx.sh`) to see installed packages, configuration files, and service ports.
- VMs use a private network with stable IPs (192.168.56.11 - 192.168.56.15). Access services from your host at those IPs, for example:
	- Nginx (web01): http://192.168.56.11/ (or access via forwarded ports if added later)
	- Tomcat (app01): check `/opt/tomcat` or configured service ports in `tomcat.sh`
	- MariaDB (db01): connect at 192.168.56.15:3306 (if the provisioning opens the port)

## Troubleshooting

- If `vagrant up` fails with VirtualBox kernel module errors on Linux, run your distribution's VirtualBox kernel module installer (for example `sudo apt install virtualbox-dkms` on Debian/Ubuntu) and reboot or reload kernel modules.
- If `vagrant-hostmanager` fails to write `/etc/hosts`, run the command with elevated privilege when prompted or add the IP-to-hostname mappings manually to `/etc/hosts`:

	192.168.56.15 db01
	192.168.56.14 mc01
	192.168.56.13 rmq01
	192.168.56.12 app01
	192.168.56.11 web01

- If provisioning scripts fail, inspect the machine logs via `vagrant ssh <vm>` and check `/var/log/` or the relevant service logs. You can also re-run provisioning with:

	vagrant provision <vm-name>

## Security and notes

- The VM provisioning scripts run with elevated privileges (they use `privileged: true`). Review the scripts before running in untrusted environments.
- This setup is intended for local development and testing only — do not expose production data or sensitive services without proper hardening.

## Next steps / improvements

- Add explicit port forwarding for services you want accessible on localhost.
- Add a simple `hosts` file sample or automated check to ensure required host prerequisites are met.
- Convert provisioning to Ansible for idempotent provisioning and easier maintenance.

If you want, I can:

- add port forwards to the `Vagrantfile` for specific services (e.g., Tomcat on 8080),
- create a small script to check host prerequisites and print guidance before `vagrant up`, or
- generate minimal usage examples for each service (how to connect to MariaDB, RabbitMQ, etc.).

---
Generated: updated README with Vagrant usage and host prerequisites.
