# ansible_role_setup_vm

Ansible role that provisions a baseline VM with a common set of tools and system configuration. Supports Fedora, Debian, Ubuntu, and Alpine Linux.

## What it does

- Upgrades all system packages
- Installs a curated set of CLI tools (see [Variables](#variables))
- Configures timezone
- Configures locales
- Deploys [Grml zshrc](https://grml.org/zsh/) to `/etc/skel/.zshrc`

## Requirements

- Ansible 2.12+
- Target host must be running Fedora, Debian, Ubuntu, or Alpine Linux

## Variables

All variables are defined in `defaults/main.yml` and can be overridden.

| Variable | Default | Description |
|---|---|---|
| `vm_timezone` | `America/Argentina/Buenos_Aires` | Timezone to set on the host |
| `vm_locales` | `[es_AR.UTF-8, en_US.UTF-8]` | Locales to generate/enable. First entry becomes the system `LANG`. |
| `fedora_packages` | see defaults | Packages installed via `dnf` on Fedora |
| `alpine_packages` | see defaults | Packages installed via `apk` on Alpine |
| `debian_packages` | see defaults | Packages installed via `apt` on Debian/Ubuntu |

## Usage

### Minimal playbook

```yaml
- hosts: vms
  roles:
    - ansible_role_setup_vm
```

### Overriding timezone and locales

```yaml
- hosts: vms
  roles:
    - role: ansible_role_setup_vm
      vars:
        vm_timezone: Europe/Berlin
        vm_locales:
          - de_DE.UTF-8
          - en_US.UTF-8
```

### Installing from git

```yaml
# requirements.yml
- name: setup_vm
  src: https://github.com/odradek/ansible_role_setup_vm
  scm: git
```

```
ansible-galaxy install -r requirements.yml
```

## Tags

Run only specific parts of the role with `--tags`:

| Tag | What it runs |
|---|---|
| `packages` | System upgrade + package installation + Grml zshrc |
| `locale` | Locale generation and system locale setting |
| `timezone` | Timezone configuration |

```
ansible-playbook site.yml --tags timezone
```

## Notes

- **Alpine**: locale support is provided by `musl-locales` / `musl-locales-lang`. Available locales are limited compared to glibc-based systems.
- **Fedora**: NTP is handled by `chrony` (the Fedora default). `ntp` is not installed.
- **Debian/Ubuntu**: NTP is handled by `systemd-timesyncd` (already present). No NTP package is installed.
