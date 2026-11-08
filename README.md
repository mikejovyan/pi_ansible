# pi_ansible

Ansible home server configuration for debian-based (including Raspberry Pi) hosts, only somewhat secure.

## Features

* **config.txt management**: optional, per-host, for Raspberry Pi hosts
* **fail2ban**: on the SSH jail
* **SSH hardening**: key-only auth, restricted AllowUsers, own sshd_config.d drop-in
* **firewalld**: zones scoped to a trusted LAN and a public web-facing zone
* **unattended-upgrades**: automatic security updates, including Raspberry Pi OS's own repo
* **podman 6**: a pinned, custom-built stack, working around Debian stable's incompatible version
* **cockpit**: web admin UI, including the Podman plugin

## Requirements

* **Host**: Debian or Raspberry Pi OS, with SSH access (key authorised) and a sudo-capable user

  ```bash
  ssh-copy-id -i ~/.ssh/raspberrypi.pub michael@raspberrypi5
  ```

* **Control node**: [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) itself, plus this project's required collections

  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```

## Deploying

Copy the inventory template for the host and fill in real values:

```bash
cp inventory.yml.example raspberrypi5.yml
```

Dry run first:

```bash
ansible-playbook deploy.yml -i raspberrypi5.yml --check --diff
```

If everything looks right, run it for real:

```bash
ansible-playbook deploy.yml -i raspberrypi5.yml
```

A single role can be run on its own:

```bash
ansible-playbook deploy.yml -i raspberrypi5.yml --tags ssh
```
