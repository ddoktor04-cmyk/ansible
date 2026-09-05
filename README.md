# Ansible AWS Inventory

Ansible inventory file for managing AWS EC2 instances and Windows machines.

## Hosts

| Name           | IP             | OS            | User     | Connection |
|----------------|----------------|---------------|----------|------------|
| amazon_linux   | 13.61.145.29   | Amazon Linux  | ec2-user | SSH        |
| ubuntu         | 56.228.15.146  | Ubuntu 26.06  | ubuntu   | SSH        |
| win10          | 172.14.50.80   | Windows 10    | user1    | WinRM      |

## Vault

Паролі зберігаються в зашифрованому файлі `group_vars/windows.yml` (Ansible Vault).
Пароль vault вводиться при запиті перед виконанням.

```bash
# Decrypt vault file
ansible-vault view group_vars/windows.yml --ask-vault-pass

# Edit vault file
ansible-vault edit group_vars/windows.yml --ask-vault-pass

# Run playbook with vault
ansible-playbook playbooks/win-config.yml --ask-vault-pass
```

## Commands

### Connection Test
```bash
# Linux hosts
ansible aws_servers -m ping

# Windows host
ansible windows -m ansible.windows.win_ping --ask-vault-pass
```

### Latency Check
```bash
time ansible aws_servers -m ping
ansible aws_servers -m ping -vvv
```

### Inventory
```bash
ansible-inventory --graph
ansible-inventory -i inventory.ini --list
```

### Ad-hoc Commands (Linux)
```bash
# Run command on all hosts
ansible aws_servers -m shell -a "uptime"

# Run on specific host
ansible amazon_linux -m shell -a "df -h"

# Check disk space
ansible aws_servers -m shell -a "free -m"

# Update packages (Amazon Linux)
ansible amazon_linux -m yum -a "name=* state=latest" --become

# Update packages (Ubuntu)
ansible ubuntu -m apt -a "update_cache=yes upgrade=yes" --become
```

### Ad-hoc Commands (Windows)
```bash
# Check Windows info
ansible windows -m ansible.windows.win_shell -a "hostname" --ask-vault-pass

# Check disk space
ansible windows -m ansible.windows.win_shell -a "Get-PSDrive -PSProvider FileSystem" --ask-vault-pass

# Run command on Windows
ansible windows -m ansible.windows.win_command -a "whoami" --ask-vault-pass
```

### Playbook
```bash
# Run playbook
ansible-playbook playbook.yml --ask-vault-pass

# Dry run
ansible-playbook playbook.yml --check --diff --ask-vault-pass

# Limit to specific host
ansible-playbook playbook.yml --limit ubuntu

# Use tags
ansible-playbook playbook.yml --tags "config"
```

### Verbose
```bash
ansible aws_servers -m ping -vvv
```

## Playbooks

### Install Apache (Linux)
```bash
ansible-playbook playbooks/apache.yml
```

### Bootstrap WinRM (Windows)
Спочатку запустіть PowerShell-скрипт на Windows для увімкнення WinRM:
```powershell
# On Windows machine (as Administrator)
powershell -ExecutionPolicy Bypass -File playbooks/winrm-setup.ps1
```
Потім запустіть bootstrap-плейбук:
```bash
ansible-playbook playbooks/win-bootstrap.yml --ask-vault-pass
```

### Basic Windows Configuration
```bash
# Show system info
ansible-playbook playbooks/win-config.yml --ask-vault-pass --tags "info"

# Apply configuration
ansible-playbook playbooks/win-config.yml --ask-vault-pass --tags "config"

# Install Windows updates
ansible-playbook playbooks/win-config.yml --ask-vault-pass --tags "update"

# Security hardening
ansible-playbook playbooks/win-config.yml --ask-vault-pass --tags "hardening"

# Full configuration
ansible-playbook playbooks/win-config.yml --ask-vault-pass
```

## Files

```
├── ansible.cfg              # Ansible configuration
├── inventory.ini            # Hosts inventory
├── group_vars/
│   └── windows.yml          # Encrypted vault file
├── playbooks/
│   ├── apache.yml           # Apache installation (Linux)
│   ├── win-bootstrap.yml    # WinRM bootstrap (Windows)
│   ├── win-config.yml       # Windows configuration
│   └── winrm-setup.ps1      # PowerShell WinRM setup script
├── ansible-skill/
│   └── SKILL.md             # Best practices
└── README.md
```
