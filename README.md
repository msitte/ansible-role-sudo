<!--
SPDX-FileCopyrightText: 2026 Matthias Sitte

SPDX-License-Identifier: MIT
-->

# sudo

OpenSCAP Security Guide aligned sudo configuration for recent Ubuntu LTS and Debian releases.

The role targets:

- Ubuntu 24.04 LTS and 26.04 LTS
- Debian 12 and 13

It manages sudo package installation, `/etc/sudoers` defaults, sudoers include rules, and validation through `visudo`.

## Requirements

- Ansible Core 2.18 or later
- Privilege escalation for package and sudoers management

## Reference

Relevant manual pages:

- `sudo(8)`
- `sudoers(5)`
- `visudo(8)`

## Role Variables

Primary control variables:

```yaml
sudo__manage_package: true
sudo__manage_config: true
sudo__package_state: present
sudo__validate_config: true
sudo__manage_ansible_user: true
```

Policy variables:

```yaml
sudo__enable_logging: true
sudo__logfile_path: /var/log/sudo.log
sudo__sudoer_config: []
sudo__timestamp_timeout: 15
```

Variables prefixed with `_sudo__` are private implementation details. They are loaded dynamically from OS-family
specific files such as `vars/Debian.yml` and should not be set from inventory.

Use `sudo__sudoer_config` for sudoers include entries:

```yaml
sudo__sudoer_config:
  - name: allow-demo-restricted-less
    user: demo
    commands:
      - /usr/bin/less
    noexec: true
    nopassword: false
    state: present
```

By default, the role also creates an unrestricted sudoers entry for the Ansible connection user so hardening unmanaged
include files does not lock out the automation account. Disable this with `sudo__manage_ansible_user: false` only when
another role or baseline guarantees equivalent access.

## Compliance Notes

The default baseline follows common OpenSCAP Security Guide sudo controls, including:

- enabling `Defaults use_pty`
- configuring a dedicated sudo logfile
- removing `!authenticate` and `NOPASSWD` from unmanaged sudoers include files
- centralizing `timestamp_timeout` in `/etc/sudoers`
- validating all sudoers changes with `visudo`

Review sudo access entries and timestamp behavior before applying to production.

## Example Playbook

```yaml
---
- name: Harden sudo
  hosts: servers
  become: true
  roles:
    - role: sudo
```

## Tags

- `sudo`
- `sudo_preflight`
- `sudo_package`
- `sudo_config`
- `sudo_validate`
- `sudo_uninstall`

## Task Structure

The role uses explicit task phases:

- `preflight.yml`
- `install.yml`
- `configure.yml`
- `validate.yml`
- `uninstall.yml`

Files named `tasks/_*.yml` are private implementation details for those phases and should not be called directly from
playbooks.

## Testing

Run role tests with:

```bash
uv run molecule test
```
