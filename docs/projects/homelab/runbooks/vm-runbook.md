# VM Initial Set Up

## Overview

This is my recipe for setting up new instances. I'd like this to evolve over time as I learn tooling in the DevOps space. I am starting from manual VM configuration and working up.

## Clone Proxmox VM Template

- Create [full clone of my 24.04 Ubuntu VM Template](https://pve.proxmox.com/wiki/VM_Templates_and_Clones)
- SSH into the new VM (keys should be copied in based on Cloud Init settings)

```shell
ssh ubuntu@{{ip}}
```

- Get MAC Address

```shell
ip link show
```

- Set Static DHCP lease for the IP in router admin and reboot the service
- Resize VM disk to need (see: [resize VM runbook](./resize-vm-runbook.md))
- [Install docker](https://docs.docker.com/engine/install/ubuntu/)
- Set up repo in home dir with README and docker compose

## TODO

- Learn the proper way to set a static IP
- Include docker in the template VM
- Learn Ansible / Terraform to automate provisioning/configuration
