# Basic Ansible Ad-hoc Commands

This README contains basic Ansible ad-hoc commands with examples for practicing from an Ansible Master EC2 instance.

---

## Assumption

Inventory file:

```bash
~/ansible-lab/inventory.ini
```

Example inventory:

```ini
[workers]
worker1 ansible_host=<EC2-WORKER-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

Move to your Ansible project directory:

```bash
cd ~/ansible-lab
```

---

# Ansible Ad-hoc Command Format

```bash
ansible -i <inventory-file> <host-or-group> -m <module-name> -a "<arguments>"
```

Example:

```bash
ansible -i inventory.ini workers -m command -a "hostname"
```

Meaning:

```text
-i inventory.ini     = inventory file
workers              = target group
-m command           = Ansible module
-a "hostname"        = command/module argument
```

For root/admin tasks, use:

```bash
--become
```

Example:

```bash
ansible -i inventory.ini workers -m command -a "whoami" --become
```

---

# 1. Test Ansible Connectivity

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

# 2. Check Hostname

```bash
ansible -i inventory.ini workers -m command -a "hostname"
```

---

# 3. Check Uptime

```bash
ansible -i inventory.ini workers -m command -a "uptime"
```

---

# 4. Check Disk Space

```bash
ansible -i inventory.ini workers -m command -a "df -h"
```

---

# 5. Check Memory Usage

```bash
ansible -i inventory.ini workers -m command -a "free -m"
```

---

# 6. Check Current Logged-in User

```bash
ansible -i inventory.ini workers -m command -a "whoami"
```

Expected output:

```text
ubuntu
```

To run as root:

```bash
ansible -i inventory.ini workers -m command -a "whoami" --become
```

Expected output:

```text
root
```

---

# 7. Check OS Details

```bash
ansible -i inventory.ini workers -m command -a "cat /etc/os-release"
```

---

# 8. Install Nginx

```bash
ansible -i inventory.ini workers -m apt -a "name=nginx state=present update_cache=yes" --become
```

Explanation:

```text
-m apt                  = Uses apt module
name=nginx              = Package name
state=present           = Install if not already installed
update_cache=yes        = Runs apt update before install
--become                = Runs with sudo/root privileges
```

---

# 9. Start Nginx Service

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=started" --become
```

---

# 10. Enable Nginx on Boot

```bash
ansible -i inventory.ini workers -m service -a "name=nginx enabled=yes" --become
```

---

# 11. Start and Enable Nginx Together

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=started enabled=yes" --become
```

---

# 12. Check Nginx Status

```bash
ansible -i inventory.ini workers -m command -a "systemctl status nginx --no-pager" --become
```

---

# 13. Stop Nginx Service

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=stopped" --become
```

---

# 14. Restart Nginx Service

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted" --become
```

---

# 15. Create a File

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/testfile.txt state=touch"
```

---

# 16. Create a Directory

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/ansible-demo state=directory mode=0755"
```

---

# 17. Copy File from Master to Worker

Create a file on Ansible Master:

```bash
echo "Hello from Ansible Master" > /tmp/demo.txt
```

Copy it to worker:

```bash
ansible -i inventory.ini workers -m copy -a "src=/tmp/demo.txt dest=/tmp/demo.txt"
```

Verify:

```bash
ansible -i inventory.ini workers -m command -a "cat /tmp/demo.txt"
```

---

# 18. Add Line to a File

```bash
ansible -i inventory.ini workers -m lineinfile -a "path=/tmp/demo.txt line='This line was added by Ansible'"
```

Verify:

```bash
ansible -i inventory.ini workers -m command -a "cat /tmp/demo.txt"
```

---

# 19. Delete a File

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/demo.txt state=absent"
```

---

# 20. Delete a Directory

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/ansible-demo state=absent"
```

---

# 21. Create a User

```bash
ansible -i inventory.ini workers -m user -a "name=devuser state=present shell=/bin/bash" --become
```

---

# 22. Delete a User

```bash
ansible -i inventory.ini workers -m user -a "name=devuser state=absent remove=yes" --become
```

---

# 23. Gather All Facts

```bash
ansible -i inventory.ini workers -m setup
```

---

# 24. Gather Memory Facts Only

```bash
ansible -i inventory.ini workers -m setup -a "filter=ansible_memtotal_mb"
```

---

# 25. Gather IP Address Facts

```bash
ansible -i inventory.ini workers -m setup -a "filter=ansible_default_ipv4"
```

---

# 26. Reboot Worker Node

```bash
ansible -i inventory.ini workers -m reboot --become
```

---

# 27. Run Shell Command with Pipe

Use `shell` module when command contains pipes, redirects, or variables.

```bash
ansible -i inventory.ini workers -m shell -a "ps -ef | grep nginx"
```

---

# 28. Command Module vs Shell Module

## command module

Use for simple commands.

```bash
ansible -i inventory.ini workers -m command -a "hostname"
```

## shell module

Use when you need shell features like pipe `|`, redirect `>`, variables `$HOME`.

```bash
ansible -i inventory.ini workers -m shell -a "echo $HOME && uptime"
```

---

# Most Used Daily Commands

```bash
ansible -i inventory.ini workers -m ping

ansible -i inventory.ini workers -m command -a "hostname"

ansible -i inventory.ini workers -m command -a "uptime"

ansible -i inventory.ini workers -m command -a "df -h"

ansible -i inventory.ini workers -m command -a "free -m"

ansible -i inventory.ini workers -m command -a "whoami"

ansible -i inventory.ini workers -m command -a "whoami" --become

ansible -i inventory.ini workers -m apt -a "name=nginx state=present update_cache=yes" --become

ansible -i inventory.ini workers -m service -a "name=nginx state=started enabled=yes" --become

ansible -i inventory.ini workers -m command -a "systemctl status nginx --no-pager" --become
```

---

# Notes

- Ansible ad-hoc commands are useful for quick one-time tasks.
- For repeated tasks, use Ansible playbooks.
- Use `--become` when the command needs sudo/root permission.
- Use `command` module for simple commands.
- Use `shell` module when using pipes, redirects, or shell variables.
