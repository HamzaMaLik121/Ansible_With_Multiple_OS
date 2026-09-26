# Ansible Multi-OS Nginx Deployment

Ansible playbook that installs and configures **Nginx** across three different Linux distributions in a single run — RHEL, Amazon Linux, and Ubuntu — each using its own OS-appropriate package manager and web root path.

## What this does

- Detects each target host's OS using Ansible facts (`ansible_distribution`)
- Installs Nginx using the correct package manager per OS:
  - **RHEL** → `dnf`
  - **Amazon Linux** → `dnf` (with a `yum` fallback for Amazon Linux 1)
  - **Ubuntu** → `apt`
- Enables and starts the Nginx service on all hosts
- Deploys a custom `index.html` to the correct web root per OS:
  - **Ubuntu** → `/var/www/html/`
  - **RHEL / Amazon Linux** → `/usr/share/nginx/html/`

## Project structure

```
playbooks/
├── install_nginx.yml
├── install_docker.yml
└── roles/
    └── nginx/
        ├── tasks/
        │   ├── main.yml        # OS detection + routing + HTML deploy
        │   ├── redhat.yml      # RHEL install tasks
        │   ├── amazon.yml      # Amazon Linux install tasks
        │   └── ubuntu.yml      # Ubuntu/Debian install tasks
        ├── handlers/
        │   └── main.yml
        ├── defaults/
        │   └── main.yml
        └── files/
            └── index.html      # Custom page served on all hosts
hosts.ini                       # Inventory (not committed — see below)
```

## Inventory

This project expects an inventory file (`hosts.ini`) with groups like:

```ini
[ubuntu_workers]
worker-ubuntu ansible_host=<PRIVATE_IP> ansible_user=ubuntu

[redhat]
worker-redhat ansible_host=<PRIVATE_IP> ansible_user=ec2-user

[amazon]
worker-amazon ansible_host=<PRIVATE_IP> ansible_user=ec2-user

[workers:children]
ubuntu_workers
redhat
amazon
```

> **Note:** `hosts.ini` and any private key files are intentionally excluded from this repo via `.gitignore`, since they contain infrastructure details and credentials. Copy `hosts.ini.example` (if provided) and fill in your own values, or create your own `hosts.ini` following the structure above.

## Usage

Run the playbook against your inventory:

```bash
ansible-playbook -i hosts.ini playbooks/install_nginx.yml
```

After it completes, each host should be serving the custom `index.html` on port 80.

## Requirements

- Ansible core 2.14+
- SSH access to all target hosts
- `become: true` privileges on all target hosts (sudo/root)

## Notes on OS detection

RHEL and Amazon Linux both fall under the `RedHat` OS family in Ansible, but they differ enough in package availability (e.g. EPEL) and default web root that this project checks the more specific `ansible_distribution` fact rather than `ansible_os_family` to route each host to the correct task file.

## License

MIT
