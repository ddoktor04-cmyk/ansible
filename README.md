# Ansible AWS Inventory

Ansible inventory file for managing AWS EC2 instances.

## Hosts

| Name           | IP             | OS            | User     |
|----------------|----------------|---------------|----------|
| amazon_linux   | 13.61.145.29   | Amazon Linux  | ec2-user |
| ubuntu         | 56.228.15.146  | Ubuntu 26.06  | ubuntu   |

## Usage

Test connection:
```bash
ansible aws_servers -m ping
```

Run command on all hosts:
```bash
ansible aws_servers -m shell -a "uptime"
```

Run playbook:
```bash
ansible-playbook playbook.yml
```
