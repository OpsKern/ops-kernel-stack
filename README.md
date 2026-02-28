# Homelab Ansible Stack

A production-grade Ansible collection for self-hosted homelabs. Covers backup
automation, metrics/logging infrastructure, and automated remediation — all
battle-tested on a real homelab running 24/7.

## Features

- **Restic backup fleet** — encrypted, deduplicated backups via SFTP/NFS with
  systemd timers, prune policies, and per-host overrides
- **Prometheus node_exporter** — fleet-wide metrics agent with idempotent install
- **Loki logging stack** — deploy Grafana Loki with Docker Compose, provisioned
  datasource, and persistent storage
- **Promtail fleet** — ship logs from all hosts to Loki with a single playbook
- **Automated remediation bridge** — Alertmanager webhook → Python dispatcher →
  Ansible playbook execution with cooldown tracking. No equivalent exists in the
  public ecosystem.

## Prerequisites

- Ansible 2.14+
- Python 3.9+ on control node
- SSH key-based access to all managed hosts
- `ansible-vault` for secret management

## Quickstart

```bash
# 1. Clone and configure
git clone <this-repo> ansible-homelab && cd ansible-homelab
cp ansible.cfg.example ansible.cfg         # edit paths
cp inventory.yml.example inventory.yml     # fill in your IPs
cp group_vars/all/vault.yml.example group_vars/all/vault.yml
ansible-vault encrypt group_vars/all/vault.yml  # add real secrets

# 2. Edit group_vars/all/vars.yml
#    Set: homelab_domain, docker_host_ip, dns_server_ip, etc.

# 3. Validate connectivity
ansible all -m ping

# 4. Deploy (always --check first)
ansible-playbook playbooks/deploy-restic-fleet.yml --check
ansible-playbook playbooks/deploy-restic-fleet.yml
```

## Playbooks

| Playbook | What it does |
|----------|-------------|
| `playbooks/deploy-restic-fleet.yml` | Install + configure Restic on all backup_servers |
| `playbooks/deploy-node-exporter.yml` | Deploy Prometheus node_exporter fleet-wide |
| `playbooks/deploy-loki.yml` | Deploy Grafana Loki on docker-host via Docker Compose |
| `playbooks/deploy-promtail-fleet.yml` | Ship logs from all hosts to Loki |
| `playbooks/deploy-remediation-bridge.yml` | Deploy the Alertmanager → Ansible remediation bridge |

## Remediation Playbooks

Targeted one-shot playbooks for automated or manual incident response:

| Playbook | Trigger variable |
|----------|-----------------|
| `remediation/restart-container.yml` | `container_name` |
| `remediation/service-restart.yml` | `service_name`, `target_host` |
| `remediation/cleanup-disk.yml` | `target_host` |
| `remediation/caddy-reload.yml` | (targets go-host directly) |

## Roles

| Role | Purpose |
|------|---------|
| `roles/common` | Baseline hardening (passwordless sudo for ansible_user) |
| `roles/restic` | Restic binary install + repository init |
| `roles/restic_backup` | Systemd timer/service for scheduled backups |

## Variable Reference

All key variables are in `group_vars/all/vars.yml`. The most important ones:

| Variable | Description |
|----------|-------------|
| `homelab_domain` | Your domain (e.g. `example.com`) |
| `docker_host_ip` | IP of your Docker/container host |
| `dns_server_ip` | Primary DNS server IP |
| `loki_url` | Loki push URL (auto-derived from docker_host_ip) |
| `ntfy_url` | ntfy notification server URL |
| `restic_sftp_repo` | Restic SFTP repository path |
| `vault_pass_file` | Path to ansible-vault password file |

## Architecture

```
Control Node (ansible-host)
├── inventory.yml          ← all hosts + IPs
├── group_vars/all/vars.yml ← shared variables
└── group_vars/all/vault.yml ← encrypted secrets

Managed Fleet
├── backup_servers         ← restic + promtail on each
├── docker-host            ← Loki + remediation bridge
└── proxmox_hosts          ← hypervisors
```

## Security Notes

- All secrets live in `vault.yml` (ansible-vault encrypted)
- No plaintext passwords in playbooks or inventory
- SSH key-only access; `become` for privilege escalation
- Rotate vault password every 6 months: `ansible-vault rekey vault.yml`

## License

MIT
