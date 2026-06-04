# Ansible Ad-hoc Commands - Common Modules

This document contains a list of commonly used Ansible modules for ad-hoc commands with examples.

Ansible ad-hoc commands are useful for quick one-time tasks without writing a full playbook.

---

## Basic Syntax

```bash
ansible -i inventory.ini <host-group> -m <module-name> -a "<arguments>"
```

Example:

```bash
ansible -i inventory.ini workers -m ping
```

---

## Inventory Example

```ini
[workers]
worker1 ansible_host=<EC2-WORKER-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

---

# 1. ping Module

Used to test connectivity between Ansible master and worker nodes.

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

# 2. command Module

Used to run simple Linux commands.

```bash
ansible -i inventory.ini workers -m command -a "hostname"
```

Examples:

```bash
ansible -i inventory.ini workers -m command -a "uptime"

ansible -i inventory.ini workers -m command -a "df -h"

ansible -i inventory.ini workers -m command -a "free -m"

ansible -i inventory.ini workers -m command -a "whoami"
```

Note:

```text
command module does not support pipe |, redirect >, or shell variables.
```

---

# 3. shell Module

Used to run shell commands with pipes, redirects, and variables.

```bash
ansible -i inventory.ini workers -m shell -a "ps -ef | grep nginx"
```

Examples:

```bash
ansible -i inventory.ini workers -m shell -a "echo $HOME"

ansible -i inventory.ini workers -m shell -a "df -h | grep /dev/root"

ansible -i inventory.ini workers -m shell -a "cat /etc/os-release | grep VERSION"
```

Use `shell` when your command contains:

```text
Pipe      |
Redirect  >
Variable  $HOME
Multiple commands using &&
```

---

# 4. apt Module

Used to install, update, or remove packages on Ubuntu/Debian servers.

Install nginx:

```bash
ansible -i inventory.ini workers -m apt -a "name=nginx state=present update_cache=yes" --become
```

Remove nginx:

```bash
ansible -i inventory.ini workers -m apt -a "name=nginx state=absent" --become
```

Update apt cache:

```bash
ansible -i inventory.ini workers -m apt -a "update_cache=yes" --become
```

---

# 5. yum Module

Used to install packages on older RHEL, CentOS, or Amazon Linux systems.

Install httpd:

```bash
ansible -i inventory.ini workers -m yum -a "name=httpd state=present" --become
```

Remove httpd:

```bash
ansible -i inventory.ini workers -m yum -a "name=httpd state=absent" --become
```

---

# 6. dnf Module

Used to install packages on newer RHEL, Fedora, Rocky Linux, AlmaLinux, or Amazon Linux 2023 systems.

Install httpd:

```bash
ansible -i inventory.ini workers -m dnf -a "name=httpd state=present" --become
```

Remove httpd:

```bash
ansible -i inventory.ini workers -m dnf -a "name=httpd state=absent" --become
```

---

# 7. package Module

Generic package module. It works across different Linux package managers.

```bash
ansible -i inventory.ini workers -m package -a "name=nginx state=present" --become
```

This is useful when you do not want to specify `apt`, `yum`, or `dnf`.

---

# 8. service Module

Used to start, stop, restart, or enable services.

Start nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=started" --become
```

Stop nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=stopped" --become
```

Restart nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted" --become
```

Enable nginx on boot:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx enabled=yes" --become
```

Start and enable nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=started enabled=yes" --become
```

---

# 9. systemd Module

Used to manage systemd services.

```bash
ansible -i inventory.ini workers -m systemd -a "name=nginx state=started enabled=yes" --become
```

Reload systemd daemon:

```bash
ansible -i inventory.ini workers -m systemd -a "daemon_reload=yes" --become
```

Restart service using systemd:

```bash
ansible -i inventory.ini workers -m systemd -a "name=nginx state=restarted" --become
```

---

# 10. file Module

Used to create/delete files, directories, and manage permissions.

Create file:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/test.txt state=touch"
```

Create directory:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/demo state=directory mode=0755"
```

Delete file:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/test.txt state=absent"
```

Delete directory:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/demo state=absent"
```

Change file permissions:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/test.txt mode=0644"
```

Change owner and group:

```bash
ansible -i inventory.ini workers -m file -a "path=/tmp/test.txt owner=ubuntu group=ubuntu mode=0644" --become
```

---

# 11. copy Module

Used to copy files from Ansible master to worker nodes.

Create a file on master:

```bash
echo "Hello from Ansible Master" > /tmp/demo.txt
```

Copy file to worker:

```bash
ansible -i inventory.ini workers -m copy -a "src=/tmp/demo.txt dest=/tmp/demo.txt"
```

Copy with owner and permission:

```bash
ansible -i inventory.ini workers -m copy -a "src=/tmp/demo.txt dest=/tmp/demo.txt owner=ubuntu group=ubuntu mode=0644" --become
```

---

# 12. fetch Module

Used to copy files from worker nodes to Ansible master.

```bash
ansible -i inventory.ini workers -m fetch -a "src=/var/log/syslog dest=/tmp/logs/ flat=yes" --become
```

Example for Ubuntu auth log:

```bash
ansible -i inventory.ini workers -m fetch -a "src=/var/log/auth.log dest=/tmp/auth.log flat=yes" --become
```

---

# 13. lineinfile Module

Used to add, update, or remove a single line in a file.

Add line:

```bash
ansible -i inventory.ini workers -m lineinfile -a "path=/tmp/demo.txt line='Added by Ansible'"
```

Update SSH config:

```bash
ansible -i inventory.ini workers -m lineinfile -a "path=/etc/ssh/sshd_config regexp='^PasswordAuthentication' line='PasswordAuthentication yes'" --become
```

Add line if not present:

```bash
ansible -i inventory.ini workers -m lineinfile -a "path=/tmp/demo.txt line='This line should exist'"
```

---

# 14. replace Module

Used to replace text inside a file.

```bash
ansible -i inventory.ini workers -m replace -a "path=/tmp/demo.txt regexp='oldtext' replace='newtext'"
```

Example:

```bash
ansible -i inventory.ini workers -m replace -a "path=/tmp/demo.txt regexp='dev' replace='prod'"
```

---

# 15. user Module

Used to create, modify, or delete users.

Create user:

```bash
ansible -i inventory.ini workers -m user -a "name=devuser state=present shell=/bin/bash" --become
```

Create user with home directory:

```bash
ansible -i inventory.ini workers -m user -a "name=devuser state=present create_home=yes shell=/bin/bash" --become
```

Delete user:

```bash
ansible -i inventory.ini workers -m user -a "name=devuser state=absent remove=yes" --become
```

---

# 16. group Module

Used to create or delete groups.

Create group:

```bash
ansible -i inventory.ini workers -m group -a "name=dev state=present" --become
```

Delete group:

```bash
ansible -i inventory.ini workers -m group -a "name=dev state=absent" --become
```

---

# 17. authorized_key Module

Used to add SSH public keys to a user.

```bash
ansible -i inventory.ini workers -m authorized_key -a "user=ubuntu key='{{ lookup('file', '/home/ubuntu/.ssh/id_rsa.pub') }}'" --become
```

For simple ad-hoc setup, `ssh-copy-id` is usually easier:

```bash
ssh-copy-id ubuntu@<EC2-WORKER-IP>
```

---

# 18. setup Module

Used to gather facts about remote servers.

Gather all facts:

```bash
ansible -i inventory.ini workers -m setup
```

Filter memory:

```bash
ansible -i inventory.ini workers -m setup -a "filter=ansible_memtotal_mb"
```

Filter IP address:

```bash
ansible -i inventory.ini workers -m setup -a "filter=ansible_default_ipv4"
```

Filter OS family:

```bash
ansible -i inventory.ini workers -m setup -a "filter=ansible_os_family"
```

---

# 19. reboot Module

Used to reboot servers.

```bash
ansible -i inventory.ini workers -m reboot --become
```

Reboot with timeout:

```bash
ansible -i inventory.ini workers -m reboot -a "reboot_timeout=300" --become
```

---

# 20. stat Module

Used to check whether a file or directory exists.

```bash
ansible -i inventory.ini workers -m stat -a "path=/etc/passwd"
```

Check nginx config file:

```bash
ansible -i inventory.ini workers -m stat -a "path=/etc/nginx/nginx.conf"
```

---

# 21. cron Module

Used to create or remove cron jobs.

Create cron:

```bash
ansible -i inventory.ini workers -m cron -a "name='daily uptime check' minute='0' hour='9' job='uptime >> /tmp/uptime.log'"
```

Remove cron:

```bash
ansible -i inventory.ini workers -m cron -a "name='daily uptime check' state=absent"
```

---

# 22. hostname Module

Used to set hostname.

```bash
ansible -i inventory.ini workers -m hostname -a "name=worker1" --become
```

Check hostname:

```bash
ansible -i inventory.ini workers -m command -a "hostname"
```

---

# 23. get_url Module

Used to download a file from internet to worker node.

```bash
ansible -i inventory.ini workers -m get_url -a "url=https://example.com/file.txt dest=/tmp/file.txt"
```

Download with permission:

```bash
ansible -i inventory.ini workers -m get_url -a "url=https://example.com/file.txt dest=/tmp/file.txt mode=0644"
```

---

# 24. uri Module

Used to call HTTP/HTTPS URLs or APIs.

Check local nginx URL:

```bash
ansible -i inventory.ini workers -m uri -a "url=http://localhost status_code=200"
```

Call external URL:

```bash
ansible -i inventory.ini workers -m uri -a "url=https://example.com method=GET"
```

---

# 25. unarchive Module

Used to extract `.zip`, `.tar`, `.tar.gz` files.

If archive already exists on remote server:

```bash
ansible -i inventory.ini workers -m unarchive -a "src=/tmp/app.tar.gz dest=/opt/ remote_src=yes" --become
```

If archive is on Ansible master:

```bash
ansible -i inventory.ini workers -m unarchive -a "src=/tmp/app.tar.gz dest=/opt/" --become
```

---

# 26. archive Module

Used to create compressed archive files.

```bash
ansible -i inventory.ini workers -m archive -a "path=/var/log dest=/tmp/logs.tar.gz format=gz" --become
```

---

# 27. mount Module

Used to mount disks or filesystems.

```bash
ansible -i inventory.ini workers -m mount -a "path=/mnt/data src=/dev/xvdf1 fstype=ext4 state=mounted" --become
```

Unmount:

```bash
ansible -i inventory.ini workers -m mount -a "path=/mnt/data state=unmounted" --become
```

---

# 28. debug Module

Mostly used in playbooks, but can also print messages.

```bash
ansible -i inventory.ini workers -m debug -a "msg='Hello from Ansible'"
```

---

# 29. wait_for Module

Used to wait for port, file, or service availability.

Wait for SSH port:

```bash
ansible -i inventory.ini workers -m wait_for -a "port=22 timeout=30"
```

Wait for HTTP port:

```bash
ansible -i inventory.ini workers -m wait_for -a "port=80 timeout=30"
```

Wait for file:

```bash
ansible -i inventory.ini workers -m wait_for -a "path=/tmp/demo.txt timeout=30"
```

---

# 30. raw Module

Used when Python is not installed on the remote server.

```bash
ansible -i inventory.ini workers -m raw -a "sudo apt update"
```

Install Python using raw:

```bash
ansible -i inventory.ini workers -m raw -a "sudo apt install -y python3"
```

---

# Most Common Modules for Daily Use

| Module           | Purpose                               |
| ---------------- | ------------------------------------- |
| `ping`           | Test Ansible connectivity             |
| `command`        | Run simple Linux commands             |
| `shell`          | Run shell commands with pipe/redirect |
| `apt`            | Install packages on Ubuntu/Debian     |
| `yum`            | Install packages on RHEL/CentOS       |
| `dnf`            | Install packages on newer RHEL/Fedora |
| `package`        | Generic package management            |
| `service`        | Manage services                       |
| `systemd`        | Manage systemd services               |
| `file`           | Manage files/directories/permissions  |
| `copy`           | Copy files from master to worker      |
| `fetch`          | Copy files from worker to master      |
| `lineinfile`     | Add/update one line in a file         |
| `replace`        | Replace text in files                 |
| `user`           | Manage users                          |
| `group`          | Manage groups                         |
| `authorized_key` | Manage SSH authorized keys            |
| `setup`          | Gather server facts                   |
| `reboot`         | Reboot server                         |
| `stat`           | Check file status                     |
| `cron`           | Manage cron jobs                      |
| `hostname`       | Set hostname                          |
| `get_url`        | Download files                        |
| `uri`            | Call API/URL                          |
| `unarchive`      | Extract archive files                 |
| `archive`        | Create archive files                  |
| `mount`          | Mount filesystem                      |
| `debug`          | Print debug message                   |
| `wait_for`       | Wait for port/file/service            |
| `raw`            | Run raw commands without Python       |

---

# Commands to List Ansible Modules

List all available modules:

```bash
ansible-doc -l
```

Search for a specific module:

```bash
ansible-doc -l | grep apt
```

Check module documentation:

```bash
ansible-doc apt
```

Examples:

```bash
ansible-doc service

ansible-doc file

ansible-doc copy

ansible-doc user

ansible-doc lineinfile
```

---

# Quick Practice Commands

```bash
ansible -i inventory.ini workers -m ping

ansible -i inventory.ini workers -m command -a "hostname"

ansible -i inventory.ini workers -m command -a "uptime"

ansible -i inventory.ini workers -m command -a "df -h"

ansible -i inventory.ini workers -m command -a "free -m"

ansible -i inventory.ini workers -m apt -a "name=nginx state=present update_cache=yes" --become

ansible -i inventory.ini workers -m service -a "name=nginx state=started enabled=yes" --become

ansible -i inventory.ini workers -m command -a "systemctl status nginx --no-pager" --become

ansible -i inventory.ini workers -m file -a "path=/tmp/test.txt state=touch"

ansible -i inventory.ini workers -m file -a "path=/tmp/demo state=directory mode=0755"

ansible -i inventory.ini workers -m user -a "name=devuser state=present shell=/bin/bash" --become

ansible -i inventory.ini workers -m setup -a "filter=ansible_default_ipv4"
```

---

# Notes

* Use `--become` when root or sudo permission is needed.
* Use `command` for simple commands.
* Use `shell` when using pipes, redirects, variables, or multiple commands.
* Use `apt` for Ubuntu/Debian.
* Use `yum` for older RHEL/CentOS/Amazon Linux.
* Use `dnf` for newer RHEL/Fedora/Amazon Linux 2023.
* Use `package` when you want generic package installation.
* For repeated tasks, use Ansible playbooks instead of ad-hoc commands.
