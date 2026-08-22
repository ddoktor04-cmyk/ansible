# Ansible AWS Inventory

Ansible inventory file for managing AWS EC2 instances.

## Hosts

| Name           | IP             | OS            | User     |
|----------------|----------------|---------------|----------|
| amazon_linux   | 13.61.145.29   | Amazon Linux  | ec2-user |
| ubuntu         | 56.228.15.146  | Ubuntu 26.06  | ubuntu   |

## Commands

### Connection Test
```bash
ansible aws_servers -m ping
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

### Ad-hoc Commands
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

### Playbook
```bash
# Run playbook
ansible-playbook playbook.yml

# Dry run
ansible-playbook playbook.yml --check --diff

# Limit to specific host
ansible-playbook playbook.yml --limit ubuntu

# Use tags
ansible-playbook playbook.yml --tags "config"
```

### Verbose
```bash
ansible aws_servers -m ping -vvv
```
