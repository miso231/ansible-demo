# Ansible demo

## Introduction
* This repo demonstrates how ansible works and contains simple tasks to show various use cases

## Setup your environment
* [Install ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (version <= 2.9.x is currently supported)
* Deploy test hosts in your infrastructure
  * Add your SSH key to the the hosts
  * Ensure you have SSH connectivity to them from your laptop
* Alter inventory file `inventory.yml` with your hosts configuration

# Tasks

## Task 1
Setup your environment and run playbook `simple-playbook.yml`

```
ansible-playbook -i inventory.yml simple-playbook.yml
```

## Task 2
Alter the `simple-playbook.yml` to install docker

* Set up docker apt repository
  ```
  - name: Add Docker APT GPG key
    ansible.builtin.apt_key:
      url: https://download.docker.com/linux/ubuntu/gpg
      state: present
  
  - name: Add Docker APT repository
    ansible.builtin.apt_repository:
      repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release | lower }} stable"
      state: present
      filename: docker
  ```

* Install `docker-ce` packages
  ```
  - name: Install Docker CE packages
    ansible.builtin.apt:
      name:
        - docker-ce
        - docker-ce-cli
        - containerd.io
      state: present
      update_cache: yes
  ```

## Task 3
Try to run `role-playbook.yml` to apply common role to all servers
```
ansible-playbook -i inventory.yml -e servers=all role-playbook.yml
```

## Task 4
Using `role-playbook.yml`, install nginx on webservers and postgres on database hosts

* Use role [geerlingguy.nginx](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/nginx/install/) for installing nginx
  * Alter groupvars for `webserver` group
    ```
    server_roles:
    - common
    - geerlingguy.nginx
    ```
* Use role [geerlingguy.postgresql](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/postgresql/install/) for installing postgres and create database `demo-database`
  * Alter groupvars for `database` group
    ```
    server_roles:
    - common
    - geerlingguy.postgresql

    postgresql_databases:
    - name: demo-database
    ```
* Install roles to your laptop
  ```
  ansible-galaxy role install -r requirements.yml
  ```
* Run the `role-playbook.yml` playbook

# Useful stuff

## Commands
```
# Show inverntory with all variables
ansible-inventory -i inventory.yml --list

# Show inverntory hosts organized into groups
ansible-inventory -i inventory.yml --graph

# Run the playbook
ansible-playbook -i inventory.yml simple-playbook.yml

# List server's fact
ansible -i inventory.yml web-test-1 -m setup

# Run single command on tergeted hosts
ansible all -i inventory.yml -m shell -a "df -h"

```

## Parameters of ansible-playbook
* `--list-hosts`
  * Only display selected hosts
  * Does not run the playbook
* `--check`
  * Run the playbook in check mode - no changes are made
* `--diff`
  * Show changes of the files
* `--limit`
  * Regex expression to limit playbook run to certain hosts

## Links
* [Ansible docs](https://docs.ansible.com/ansible/latest/index.html)
* [Ansible inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)
* [Ansible Galaxy](https://galaxy.ansible.com/ui/)
