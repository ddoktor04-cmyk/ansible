---
name: ansible-best-practices
description: Best practices for Ansible playbooks, inventory, roles, and automation on AWS EC2 instances.
compatibility: Requires Ansible >= 2.12, Python >= 3.8
metadata:
  author: ddoktor04
  version: "1.0"
---

# Ansible Best Practices

## Project Structure

```
ansible-project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.ini
│   │   └── group_vars/
│   │       ├── all.yml
│   │       └── aws_servers.yml
│   └── staging/
│       ├── hosts.ini
│       └── group_vars/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── postgresql/
├── files/
├── templates/
└── README.md
```

## Inventory

### INI Format

```ini
[webservers]
web1 ansible_host=13.61.145.29 ansible_user=ec2-user
web2 ansible_host=56.228.15.146 ansible_user=ubuntu

[dbservers]
db1 ansible_host=10.0.1.50 ansible_user=ec2-user

[all:vars]
ansible_ssh_private_key_file=/home/user1/ansible/keys/Key1.pem
ansible_python_interpreter=/usr/bin/python3
ansible_become=yes
ansible_become_method=sudo
```

### YAML Format

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 13.61.145.29
          ansible_user: ec2-user
        web2:
          ansible_host: 56.228.15.146
          ansible_user: ubuntu
    dbservers:
      hosts:
        db1:
          ansible_host: 10.0.1.50
          ansible_user: ec2-user
  vars:
    ansible_ssh_private_key_file: /home/user1/ansible/keys/Key1.pem
    ansible_python_interpreter: /usr/bin/python3
    ansible_become: yes
    ansible_become_method: sudo
```

## Playbooks

### Basic Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes

  vars:
    http_port: 80
    max_clients: 200

  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes
```

### Running Playbooks

```bash
# Run all playbooks
ansible-playbook playbooks/site.yml

# Run with specific inventory
ansible-playbook -i inventory/production/hosts.ini playbooks/site.yml

# Run specific playbook
ansible-playbook playbooks/webservers.yml

# Limit to specific host
ansible-playbook playbooks/site.yml --limit web1

# Dry run (check mode)
ansible-playbook playbooks/site.yml --check

# Use tags
ansible-playbook playbooks/site.yml --tags "nginx,config"
```

## Roles

### Role Structure

```
roles/
└── nginx/
    ├── tasks/
    │   ├── main.yml
    │   └── install.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    ├── defaults/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    └── meta/
        └── main.yml
```

### Role Example: Nginx

**roles/nginx/tasks/main.yml:**
```yaml
---
- name: Install Nginx
  ansible.builtin.package:
    name: nginx
    state: present
  notify: Restart Nginx

- name: Deploy Nginx config
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Restart Nginx
```

**roles/nginx/handlers/main.yml:**
```yaml
---
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

**roles/nginx/defaults/main.yml:**
```yaml
---
nginx_worker_processes: auto
nginx_worker_connections: 1024
nginx_listen_port: 80
```

**roles/nginx/templates/nginx.conf.j2:**
```nginx
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    server {
        listen {{ nginx_listen_port }};
        server_name {{ ansible_fqdn }};

        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

### Using Roles in Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes

  roles:
    - common
    - nginx
    - { role: app, tags: ['app', 'deploy'] }
```

## Variables

### group_vars

**group_vars/all.yml:**
```yaml
---
timezone: Europe/Kyiv
ntp_server: pool.ntp.org
```

**group_vars/webservers.yml:**
```yaml
---
http_port: 443
https_enabled: yes
```

### host_vars

**host_vars/web1.yml:**
```yaml
---
ansible_host: 13.61.145.29
custom_var: value
```

## Templates (Jinja2)

```jinja2
{# Config file template #}
server {
    listen {{ port }};
    server_name {{ server_name }};

{% if ssl_enabled %}
    ssl_certificate /etc/ssl/{{ cert_file }};
    ssl_certificate_key /etc/ssl/{{ key_file }};
{% endif %}

    location / {
        proxy_pass http://{{ backend_host }}:{{ backend_port }};
    }
}
```

## Idempotency

```yaml
# GOOD: Idempotent
- name: Ensure Nginx is installed
  ansible.builtin.package:
    name: nginx
    state: present

# GOOD: Idempotent
- name: Ensure config file exists
  ansible.builtin.copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
```

## Tags

```yaml
---
- name: Configure servers
  hosts: all
  become: yes

  tasks:
    - name: Install packages
      ansible.builtin.package:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
      tags:
        - install
        - packages

    - name: Deploy config
      ansible.builtin.template:
        src: app.conf.j2
        dest: /etc/app/config.conf
      tags:
        - config
```

```bash
# Run only tagged tasks
ansible-playbook site.yml --tags "config"
```

## Ansible Vault

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass

# Use vault password file
ansible-playbook site.yml --vault-password-file=.vault_pass

# Set vault_password_file in ansible.cfg (recommended)
# [defaults]
# vault_password_file = .vault_pass
```

### Store Secrets in group_vars

```bash
# Create vault-encrypted group_vars
ansible-vault create group_vars/windows.yml
# Content: win_password: my_secret_password

# Reference in inventory.ini
# ansible_winrm_password={{ win_password }}

# Run with vault
ansible-playbook site.yml
# (vault_password_file in ansible.cfg auto-decrypts)
```

### Encrypt Variables

```yaml
# In playbook
vars:
  db_password: !vault |
    $ANSIBLE_VAULT;1.1;AES256
    663864396532363364626265666530633361...
```

## Check Mode and Diff

```bash
# Dry run
ansible-playbook site.yml --check

# Show differences
ansible-playbook site.yml --diff

# Combine
ansible-playbook site.yml --check --diff
```

## AWS EC2 Examples

### Install and Configure Web Server

```yaml
---
- name: Configure AWS web server
  hosts: webservers
  become: yes
  gather_facts: yes

  vars:
    app_name: myapp
    app_port: 8080

  tasks:
    - name: Update system
      ansible.builtin.yum:
        name: '*'
        state: latest
      when: ansible_os_family == "RedHat"

    - name: Update system (Ubuntu)
      ansible.builtin.apt:
        update_cache: yes
        upgrade: dist
      when: ansible_os_family == "Debian"

    - name: Install required packages
      ansible.builtin.package:
        name:
          - nginx
          - python3
          - git
        state: present

    - name: Deploy application
      ansible.builtin.git:
        repo: https://github.com/user/repo.git
        dest: /opt/{{ app_name }}
        version: main

    - name: Ensure services are running
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - nginx
        - {{ app_name }}
```

## Windows Management (WinRM)

### Windows Inventory

```ini
[windows]
win10 ansible_host=172.14.50.80 ansible_user=user1

[windows:vars]
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
ansible_winrm_scheme=http
ansible_port=5985
ansible_winrm_password={{ win_password }}
```

### WinRM Bootstrap

**Step 1: Enable WinRM on Windows (run on target):**

```powershell
# PowerShell script: winrm-setup.ps1
Enable-PSRemoting -Force -SkipNetworkProfileCheck
winrm quickconfig -q
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
netsh advfirewall firewall add rule name="WinRM HTTP" dir=in action=allow protocol=TCP localport=5985
Set-Service -Name WinRM -StartupType Automatic
Start-Service -Name WinRM
```

**Step 2: Ansible bootstrap playbook:**

```yaml
---
- name: Bootstrap WinRM on Windows
  hosts: windows
  gather_facts: no
  tasks:
    - name: Enable WinRM Quick Config
      ansible.windows.win_shell: winrm quickconfig -q

    - name: Allow unencrypted connections
      ansible.windows.win_shell: >
        winrm set winrm/config/service '@{AllowUnencrypted="true"}'

    - name: Enable Basic authentication
      ansible.windows.win_shell: >
        winrm set winrm/config/service/auth '@{Basic="true"}'

    - name: Open firewall port 5985
      ansible.windows.win_shell: >
        netsh advfirewall firewall add rule name="WinRM HTTP"
        dir=in action=allow protocol=TCP localport=5985

    - name: Ensure WinRM service is running
      ansible.windows.win_service:
        name: WinRM
        state: started
        startup_type: automatic
```

### Windows Playbook Example

```yaml
---
- name: Configure Windows server
  hosts: windows
  gather_facts: yes

  tasks:
    - name: Display system info
      ansible.windows.win_shell: hostname
      register: host_info

    - name: Set timezone
      community.windows.win_timezone:
        timezone: Europe/Kyiv

    - name: Install Windows updates
      ansible.windows.win_updates:
        category_names:
          - CriticalUpdates
          - SecurityUpdates
        state: installed

    - name: Enable firewall
      community.windows.win_firewall:
        state: enabled
        profiles:
          - Domain
          - Private
          - Public

    - name: Disable unnecessary services
      ansible.windows.win_service:
        name: "{{ item }}"
        state: stopped
        startup_type: disabled
      loop:
        - SysMain
        - DiagTrack
```

### Windows Ad-hoc Commands

```bash
# Test connection
ansible windows -m ansible.windows.win_ping

# Run command
ansible windows -m ansible.windows.win_shell -a "whoami"

# Check disk
ansible windows -m ansible.windows.win_shell -a "Get-PSDrive -PSProvider FileSystem"

# Run executable
ansible windows -m ansible.windows.win_command -a "dir C:\\"
```

### Requirements

- `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`
- `community.windows` collection: `ansible-galaxy collection install community.windows`
- Python `pywinrm` package: `pip install pywinrm`

## Checklist

- [ ] Inventory файли структуровані за середовищами
- [ ] Використовуються roles для організації коду
- [ ] Змінні винесені в group_vars/host_vars
- [ ] Playbook'и ідемпотентні
- [ ] Використовуються теги для групування тасків
- [ ] Чутливі дані захищені через vault
- [ ] Vault password file в .gitignore
- [ ] WinRM bootstrap виконано на Windows хостах
- [ ] Тести перед застосуванням (--check --diff)
- [ ] Документація в README
