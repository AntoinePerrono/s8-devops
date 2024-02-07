# TP3 DevOps - Ansible
Author: Antoine PERRONO - 4IRC

## Introduction
### Ping remote server with Ansible:
`ansible all -i inventories/setup.yml -m ping` or `ansible all -i <path_to_hosts_file> -m ping --private-key=<path_to_your_ssh_key> -u centos`

### Inventories
**Question 3.1:**  
See [setup.yml](./ansible/inventories/setup.yml)

### Fact

To see the remote distribution:
`ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"`

## Playbooks
**Question 3.2:**  
See [setup.yml](./ansible/inventories/setup.yml)

New `playbook.yml` with the docker role:
```yaml
- hosts: all
  gather_facts: false
  become: true
  roles: 
    - role: './roles/docker'
```

All the tasks have been moved to the `main.yml` file in `roles/docker/task`.

## Deploy your App

