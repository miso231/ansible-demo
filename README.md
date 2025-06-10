# Ansible demo

## Introduction
* This repo demonstrates how ansible works and contains simple tasks to show various use cases

## Setup your environment
* [Install ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (2.9.x or newer)
* Deploy test hosts in your infrastructure
  * Ubuntu Server 22.04 is used in this demo
  * Add your SSH key to the the hosts (use user `ansible`)
  * Ensure you have SSH connectivity to them from your laptop
* Alter inventory file `inventory.yml` with your hosts configuration

# Tasks

## Task 1

### Assignment
Setup your environment and run playbook `simple-playbook.yml`

### Implementation
```
ansible-playbook -i inventory.yml simple-playbook.yml
```

### Test
The ansible run should work without any error

## Task 2

### Assignment
Alter the `simple-playbook.yml` to install docker

* Set up docker apt repository
* Install `docker-ce` packages

### Implementation
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

- name: Install Docker CE packages
  ansible.builtin.apt:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    state: present
    update_cache: yes
```

### Test
Run e.g. `docker pull alpine` on one of the hosts to verify docker is installed

## Task 3

### Assignment
* Try to run `role-playbook.yml` to apply common role to all servers
* You have to specify `servers` variable (there would be error otherwise)

### Implementation
```
ansible-playbook -i inventory.yml -e servers=all role-playbook.yml
```

### Test
* The ansible run should work without any error
* There will be a lot of `changed` tasks
* The next run will be all tasks in `ok` state

## Task 4

### Assignment
* Using `role-playbook.yml`, install nginx on webservers and postgres on database hosts
* Use role [geerlingguy.nginx](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/nginx/install/) for installing nginx
* Use role [geerlingguy.postgresql](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/postgresql/install/) for installing postgres and create database `demo-database`

### Implementation
* Alter groupvars for `webserver` group
  ```
  server_roles:
  - common
  - geerlingguy.nginx
  ```
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

### Test
* Login to database server as `postgres` and run `psql -c "\l"` to list databases
* verify that `demo-database` is there
  ```
  sudo -iu postgres
  psql -c "\l"
  ```
* Check URL of both web servers (http://<WEB_SERVER_IP>) and verify that Nginx welcome page is present

## Task 5

### Assignment
* Create a new role to install [2048 game](https://github.com/alexwhen/docker-2048) on the webservers in docker container
* Configure nginx as reverse proxy to the game

### Implementation
* Create a new file in path `roles/game-2048/tasks/main.yml`
  ```
  - name: Run 2048 game container
    community.docker.docker_container:
      name: 2048
      image: alexwhen/docker-2048
      state: started
      restart_policy: always
      ports:
        - "127.0.0.1:8080:80"
  ```
* Alter groupvars for `webserver` group
  ```
  server_roles:
  - common
  - geerlingguy.nginx
  - game-2048

  nginx_remove_default_vhost: true
  nginx_vhosts:
    - listen: "80"
      server_name: "_"
      extra_parameters: |
        location / {
            proxy_pass http://localhost:8080;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
  ```
* Run the `role-playbook.yml` playbook

### Test
Play the game

# Useful stuff

## Commands
* Show inverntory with all variables
  ```
  ansible-inventory -i inventory.yml --list
  ```
* Show inverntory hosts organized into groups
  ```
  ansible-inventory -i inventory.yml --graph
  ```
* Run the playbook
  ```
  ansible-playbook -i inventory.yml simple-playbook.yml
  ```
* List server's fact
  ```
  ansible -i inventory.yml web-test-1 -m setup
  ```
* Run single command on targeted hosts
  ```
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
