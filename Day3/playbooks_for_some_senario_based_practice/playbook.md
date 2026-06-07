# Real-Time Ansible Practice Scenarios on Ubuntu Servers

This repository contains real-time Ansible practice scenarios using one Ansible Master EC2 instance and multiple Ubuntu Worker EC2 instances.

The goal of this lab is to create common Linux/DevOps problems on worker nodes and fix them using Ansible playbooks.

---

## Architecture

```text
Local Laptop
   |
   | SSH
   v
Ansible Master EC2
   |
   | Ansible over SSH
   v
Ubuntu Worker Node 1
Ubuntu Worker Node 2
Ubuntu Worker Node 3
```

---

## Lab Environment

```text
Operating System : Ubuntu
Control Node     : Ansible Master EC2
Managed Nodes    : Ubuntu Worker EC2 instances
Connection       : SSH
Authentication   : SSH key-based passwordless authentication
```

---

## Repository Structure

```text
ansible-realtime-scenarios/
├── inventory.ini
├── install-tools.yml
├── fix-nginx-service.yml
├── fix-website-content.yml
├── cleanup-disk.yml
├── create-users.yml
├── harden-ssh.yml
├── fix-port-conflict.yml
└── health-check.yml
```

---

## Inventory File

Create an inventory file named `inventory.ini`.

Use private IPs if the Ansible master and worker nodes are in the same AWS VPC.

```ini
[workers]
worker1 ansible_host=<WORKER-1-PRIVATE-IP>
worker2 ansible_host=<WORKER-2-PRIVATE-IP>
worker3 ansible_host=<WORKER-3-PRIVATE-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

Example:

```ini
[workers]
worker1 ansible_host=172.31.41.142
worker2 ansible_host=172.31.32.217

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

---

## Test Ansible Connectivity

Before running any playbook, test connectivity.

```bash
ansible -i inventory.ini workers -m ping
```

Expected output:

```text
worker1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Recommended Learning Flow

Follow this order:

```text
1. Install basic tools
2. Fix service down issue
3. Fix wrong website content
4. Disk cleanup issue
5. User and group creation
6. SSH hardening
7. Port conflict troubleshooting
8. Server health check report
```

---

# Scenario 1: Install Basic Troubleshooting Tools

## Use Case

In real-time environments, DevOps/SRE teams usually install common troubleshooting tools on servers.

Tools include:

```text
htop
net-tools
curl
wget
unzip
tree
telnet
dnsutils
```

## Create Playbook

Create a file:

```bash
vi install-tools.yml
```

Add:

```yaml
---
- name: Install basic troubleshooting tools
  hosts: workers
  become: yes

  tasks:
    - name: Install common tools
      apt:
        name:
          - htop
          - net-tools
          - curl
          - wget
          - unzip
          - tree
          - telnet
          - dnsutils
        state: present
        update_cache: yes

    - name: Verify installed tools
      command: which htop curl wget unzip
      register: tools_output

    - name: Show tool paths
      debug:
        var: tools_output.stdout_lines
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini install-tools.yml
```

---

# Scenario 2: Fix Nginx Service Down Issue

## Use Case

In production, a web service may go down. Ansible can be used to bring the service back online.

## Create the Problem

Stop Nginx on all worker nodes:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=stopped" --become
```

Check service status:

```bash
ansible -i inventory.ini workers -m command -a "systemctl is-active nginx" --become
```

## Create Fix Playbook

Create a file:

```bash
vi fix-nginx-service.yml
```

Add:

```yaml
---
- name: Fix Nginx service down issue
  hosts: workers
  become: yes

  tasks:
    - name: Install nginx if missing
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Verify nginx status
      command: systemctl is-active nginx
      register: nginx_status

    - name: Show nginx status
      debug:
        msg: "Nginx service status is {{ nginx_status.stdout }}"
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini fix-nginx-service.yml
```

---

# Scenario 3: Fix Wrong Website Content

## Use Case

Sometimes website files are modified incorrectly. Ansible can restore the correct content.

## Create the Problem

```bash
ansible -i inventory.ini workers -m copy -a "dest=/var/www/html/index.html content='Wrong Website Content' owner=www-data group=www-data mode=0644" --become
```

Check current content:

```bash
ansible -i inventory.ini workers -m shell -a "curl -s http://localhost"
```

## Create Fix Playbook

Create a file:

```bash
vi fix-website-content.yml
```

Add:

```yaml
---
- name: Restore correct website content
  hosts: workers
  become: yes

  tasks:
    - name: Ensure nginx is installed
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Restore correct homepage
      copy:
        dest: /var/www/html/index.html
        content: |
          <html>
          <head>
              <title>SDHub Ansible Class</title>
          </head>
          <body>
              <h1>Welcome to SDHub Ansible Class</h1>
              <h2>This page was restored using Ansible</h2>
          </body>
          </html>
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
        enabled: yes
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini fix-website-content.yml
```

## Verify

```bash
ansible -i inventory.ini workers -m shell -a "curl -s http://localhost"
```

---

# Scenario 4: Disk Cleanup Issue

## Use Case

In real-time, servers may generate large temporary files or logs, causing disk usage alerts.

## Create the Problem

Create a 500 MB dummy file on all worker nodes:

```bash
ansible -i inventory.ini workers -m shell -a "sudo mkdir -p /tmp/demo-logs && sudo fallocate -l 500M /tmp/demo-logs/bigfile.log" --become
```

Check disk usage:

```bash
ansible -i inventory.ini workers -m command -a "df -h"
```

## Create Cleanup Playbook

Create a file:

```bash
vi cleanup-disk.yml
```

Add:

```yaml
---
- name: Cleanup temporary logs and old files
  hosts: workers
  become: yes

  tasks:
    - name: Check disk usage before cleanup
      command: df -h
      register: disk_before

    - name: Show disk usage before cleanup
      debug:
        var: disk_before.stdout_lines

    - name: Remove demo log directory
      file:
        path: /tmp/demo-logs
        state: absent

    - name: Clean apt cache
      apt:
        autoclean: yes

    - name: Remove unused packages
      apt:
        autoremove: yes

    - name: Check disk usage after cleanup
      command: df -h
      register: disk_after

    - name: Show disk usage after cleanup
      debug:
        var: disk_after.stdout_lines
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini cleanup-disk.yml
```

---

# Scenario 5: User and Group Creation

## Use Case

DevOps teams often need to create Linux users and groups across multiple servers.

## Create Playbook

Create a file:

```bash
vi create-users.yml
```

Add:

```yaml
---
- name: Create users and groups on Ubuntu servers
  hosts: workers
  become: yes

  tasks:
    - name: Create devops group
      group:
        name: devops
        state: present

    - name: Create support group
      group:
        name: support
        state: present

    - name: Create user raghav
      user:
        name: raghav
        group: devops
        shell: /bin/bash
        create_home: yes
        state: present

    - name: Create user student
      user:
        name: student
        group: support
        shell: /bin/bash
        create_home: yes
        state: present

    - name: Allow devops group passwordless sudo
      copy:
        dest: /etc/sudoers.d/devops
        content: "%devops ALL=(ALL) NOPASSWD:ALL\n"
        owner: root
        group: root
        mode: '0440'
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini create-users.yml
```

## Verify Users

```bash
ansible -i inventory.ini workers -m command -a "id raghav" --become
ansible -i inventory.ini workers -m command -a "id student" --become
```

---

# Scenario 6: SSH Hardening

## Use Case

In production, SSH password authentication and root login are usually disabled for better security.

## Create Playbook

Create a file:

```bash
vi harden-ssh.yml
```

Add:

```yaml
---
- name: Harden SSH configuration
  hosts: workers
  become: yes

  tasks:
    - name: Disable SSH password authentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
        backup: yes

    - name: Disable root SSH login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        backup: yes

    - name: Ensure public key authentication is enabled
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PubkeyAuthentication'
        line: 'PubkeyAuthentication yes'
        backup: yes

    - name: Restart SSH service
      service:
        name: ssh
        state: restarted
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini harden-ssh.yml
```

## Verify SSH Configuration

```bash
ansible -i inventory.ini workers -m shell -a "grep -E 'PasswordAuthentication|PermitRootLogin|PubkeyAuthentication' /etc/ssh/sshd_config" --become
```

---

# Scenario 7: Fix Port 80 Conflict

## Use Case

Apache and Nginx cannot run on port `80` at the same time. This is a common web server issue.

## Create the Problem

Install and start Apache:

```bash
ansible -i inventory.ini workers -m apt -a "name=apache2 state=present update_cache=yes" --become
ansible -i inventory.ini workers -m service -a "name=apache2 state=started enabled=yes" --become
```

Install and try to start Nginx:

```bash
ansible -i inventory.ini workers -m apt -a "name=nginx state=present" --become
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted" --become
```

## Diagnose Port Conflict

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

## Create Fix Playbook

Create a file:

```bash
vi fix-port-conflict.yml
```

Add:

```yaml
---
- name: Fix port 80 conflict and keep only nginx running
  hosts: workers
  become: yes

  tasks:
    - name: Stop apache2
      service:
        name: apache2
        state: stopped
        enabled: no
      ignore_errors: yes

    - name: Start nginx
      service:
        name: nginx
        state: restarted
        enabled: yes

    - name: Check port 80 listener
      shell: ss -tulpn | grep ':80'
      register: port_output

    - name: Show port 80 process
      debug:
        var: port_output.stdout_lines
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini fix-port-conflict.yml
```

---

# Scenario 8: Server Health Check Report

## Use Case

DevOps/SRE teams regularly collect server health details like uptime, disk, memory, and top processes.

## Create Playbook

Create a file:

```bash
vi health-check.yml
```

Add:

```yaml
---
- name: Server health check report
  hosts: workers
  become: yes

  tasks:
    - name: Check hostname
      command: hostname
      register: hostname_output

    - name: Check uptime
      command: uptime
      register: uptime_output

    - name: Check disk usage
      command: df -h
      register: disk_output

    - name: Check memory usage
      command: free -m
      register: memory_output

    - name: Check top 5 memory processes
      shell: ps aux --sort=-%mem | head -6
      register: process_output

    - name: Show health report
      debug:
        msg:
          - "Hostname: {{ hostname_output.stdout }}"
          - "Uptime: {{ uptime_output.stdout }}"
          - "Disk:"
          - "{{ disk_output.stdout_lines }}"
          - "Memory:"
          - "{{ memory_output.stdout_lines }}"
          - "Top Memory Processes:"
          - "{{ process_output.stdout_lines }}"
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini health-check.yml
```

---

# Useful Ansible Commands

## Ping Workers

```bash
ansible -i inventory.ini workers -m ping
```

## Run Uptime

```bash
ansible -i inventory.ini workers -m command -a "uptime"
```

## Check Disk

```bash
ansible -i inventory.ini workers -m command -a "df -h"
```

## Check Memory

```bash
ansible -i inventory.ini workers -m command -a "free -m"
```

## Run Playbook Syntax Check

```bash
ansible-playbook -i inventory.ini <playbook-name>.yml --syntax-check
```

Example:

```bash
ansible-playbook -i inventory.ini health-check.yml --syntax-check
```

## Run Playbook

```bash
ansible-playbook -i inventory.ini <playbook-name>.yml
```

Example:

```bash
ansible-playbook -i inventory.ini health-check.yml
```

---

# Notes for Students

```text
Ad-hoc commands are used for quick one-time actions.
Playbooks are used for repeatable automation.
Inventory file contains server details.
Modules are reusable Ansible functions.
YAML indentation is very important.
Use private IPs in inventory when master and workers are in the same VPC.
Use public IPs only for browser or laptop access.
```

---

# Important Security Notes

For lab practice, some rules may be open for testing.

For production:

```text
Do not open SSH to 0.0.0.0/0
Allow SSH only from trusted IPs or bastion/security group
Use private IPs for Ansible communication
Disable password-based SSH login
Use key-based authentication
Use Ansible Vault for secrets
Rotate SSH keys regularly
Use AWS Systems Manager Session Manager where possible
```

---

# Final Practice Order

Run the scenarios in this order:

```text
1. install-tools.yml
2. fix-nginx-service.yml
3. fix-website-content.yml
4. cleanup-disk.yml
5. create-users.yml
6. harden-ssh.yml
7. fix-port-conflict.yml
8. health-check.yml
```

This gives a strong real-time understanding of Ansible for Ubuntu server administration.
