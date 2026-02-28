# Role: common

Baseline configuration applied to every managed host. Lightweight by design —
does only what every host needs, nothing service-specific.

## What it does

Currently: configures passwordless sudo for the Ansible user so that `become`
works without a password prompt during playbook runs.

Extend this role with additional baseline tasks as your fleet grows:
- SSH hardening (`sshd_config`)
- MOTD / login banner
- UFW base rules (allow SSH, deny all inbound)
- NTP / chrony configuration
- Common packages (`vim`, `curl`, `htop`, `jq`)

## Requirements

- Target host must have `sudo` installed
- `ansible_user` must exist on the target host

## Variables

No role-specific variables. Uses `ansible_user_id` (auto-detected by Ansible).

## Example playbook

```yaml
- hosts: all
  roles:
    - role: common
```

## What it writes

Creates `/etc/sudoers.d/<ansible_user>` with:
```
<ansible_user> ALL=(ALL) NOPASSWD:ALL
```

This file is managed by Ansible. Don't edit it manually — it will be overwritten
on the next playbook run.

## Notes

- The sudoers file uses `mode: '0440'` (required by sudo)
- The file is validated by Ansible before writing (prevents sudo lockout)
- Safe to re-run (idempotent)
