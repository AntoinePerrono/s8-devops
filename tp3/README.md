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

Creation of the network on `docker/main.yml` (and put this in each `roles/main.yaml`):
```yaml
    networks:
      - name: app-network
```

Add our new container in `playbook.yml`:
```yaml
  roles: 
    - role: './roles/docker' #call task from this role
    - role: './roles/httpd' #call task from this role
    - role: './roles/webserver' #call task from this role
    - role: './roles/db' #call task from this role
```

Add this each roles `main.yaml`:
```yaml
    pull: true
    recreate: true
```

Port redirection for **httpd** roles:
```yaml
    ports: "80:80"
```

Use a volume for data persistance in **db** roles
```yaml
    volumes:
      - db:/var/lib/postgresql/data
```

## Environment variables

Oups, all my user, password and other are accessicle for all !

In **docker**:  
Update `docker-compose.yml`, for the database with 
```yaml
environment:
    - POSTGRES_DB=db
    - POSTGRES_USER=usr 
    - POSTGRES_PASSWORD=pwd
```
And for the backend:
```yaml
environment:
    - USERNAME=usr
    - PASSWORD=pwd
    - URL=db:5432
    - DB=db 
```

In **Ansible**:
In the **main.yaml** of each roles who need some env var, use this syntax:
```yaml
 env:
      POSTGRES_DB: "{{ POSTGRES_DB }}"
      POSTGRES_USER: "{{ POSTGRES_USER }}"
      POSTGRES_PASSWORD: "{{ POSTGRES_PASSWORD }}"
```

And in `inventories/setup.yml`, in the **vars** section, do :
```yaml
   POSTGRES_DB: "db"
   POSTGRES_USER: "usr"
   POSTGRES_PASSWORD: "pwd"
   POSTGRES_PORT: "5432"
   POSTGRES_HOST: "{{ POSTGRES_DB }}:{{POSTGRES_PORT}}"
```

## Front 

We need to create a new role, in its `tasks/main.yml`, we add:
```yaml
- name: Launch front
  docker_container:
    name: front
    image: skafee/tp-devops-front
    pull: true
    recreate: true
    networks:
      - name: app-network
```

We need to change some port exposition in others roles main.yaml

Add to **playbook.yml**:
```yaml
    - role: './roles/front'
```

After updating our github workflows, we can see our front on our server.

## Continuous deployment