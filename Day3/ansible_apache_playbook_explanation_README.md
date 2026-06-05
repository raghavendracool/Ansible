# Ansible Playbook Explanation - Install Apache and Deploy Custom Web Page

This README explains an Ansible playbook that installs Apache on worker nodes, stops Nginx if it is using port 80, creates a custom `index.html` page, and starts/enables the Apache service.

---

## Playbook Name

```bash
install-apache.yml
```

---

## Full Playbook

```yaml
---
- name: Install Apache and deploy custom index page
  hosts: workers
  become: yes

  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Stop nginx if running
      shell: |
        systemctl stop nginx || true
        systemctl disable nginx || true

    - name: Install Apache web server
      apt:
        name: apache2
        state: present

    - name: Remove default Apache index page
      file:
        path: /var/www/html/index.html
        state: absent

    - name: Create custom index.html page
      copy:
        dest: /var/www/html/index.html
        content: |
          <html>
          <head>
              <title>Ansible Class</title>
          </head>
          <body>
              <h1>Welcome to Ansible Class from SDHub</h1>
          </body>
          </html>
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Start and enable Apache service
      service:
        name: apache2
        state: restarted
        enabled: yes
```

---

# Line-by-Line Explanation

## 1. YAML Start

```yaml
---
```

This tells Ansible that this is a YAML file.

YAML is the format used to write Ansible playbooks.

---

## 2. Play Name

```yaml
- name: Install Apache and deploy custom index page
```

This is the name of the play.

It is mainly used for readability. When the playbook runs, Ansible displays this name in the output.

Example output:

```text
PLAY [Install Apache and deploy custom index page]
```

---

## 3. Target Hosts

```yaml
hosts: workers
```

This tells Ansible where to run the playbook.

Here, `workers` refers to the group name in the `inventory.ini` file.

Example inventory:

```ini
[workers]
worker1 ansible_host=172.31.41.142
worker2 ansible_host=172.31.32.217
```

So this playbook will run on both `worker1` and `worker2`.

---

## 4. Become Root User

```yaml
become: yes
```

This tells Ansible to run the tasks with sudo/root privileges.

We need root permission because the playbook is doing system-level tasks like installing packages, modifying `/var/www/html/index.html`, and starting services.

Without `become: yes`, these tasks may fail due to permission issues.

---

## 5. Tasks Section

```yaml
tasks:
```

This is where we define the actual work Ansible needs to perform.

Each item under `tasks` is one step.

---

# Task 1: Update apt Package Cache

```yaml
- name: Update apt package cache
  apt:
    update_cache: yes
```

## What This Does

This updates the Ubuntu package list.

Equivalent Linux command:

```bash
sudo apt update
```

## Why This Is Needed

Before installing a package, the server should know the latest available package versions.

## Module Used

```yaml
apt:
```

The `apt` module is used for package management on Ubuntu/Debian systems.

---

# Task 2: Stop Nginx If Running

```yaml
- name: Stop nginx if running
  shell: |
    systemctl stop nginx || true
    systemctl disable nginx || true
```

## What This Does

This stops and disables Nginx if it is running.

Equivalent Linux commands:

```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
```

## Why This Is Needed

Apache uses port `80`.

Nginx also uses port `80`.

Only one service can use port `80` at a time. If Nginx is already running, Apache cannot start and you may get this error:

```text
Address already in use: could not bind to address 0.0.0.0:80
```

So we stop Nginx first before starting Apache.

## What `shell: |` Means

```yaml
shell: |
```

The `shell` module is used to run Linux shell commands.

The `|` symbol allows us to write multiple lines of shell commands.

## What `|| true` Means

```bash
systemctl stop nginx || true
```

This means:

```text
Try to stop nginx.
If nginx is not installed or already stopped, do not fail the playbook.
```

Without `|| true`, the playbook may fail if Nginx is not available on the server.

---

# Task 3: Install Apache Web Server

```yaml
- name: Install Apache web server
  apt:
    name: apache2
    state: present
```

## What This Does

This installs Apache on Ubuntu.

Equivalent Linux command:

```bash
sudo apt install apache2 -y
```

## Explanation

```yaml
name: apache2
```

`apache2` is the Apache package name on Ubuntu.

```yaml
state: present
```

This means:

```text
Install apache2 if it is not already installed.
If it is already installed, do nothing.
```

This behavior is called idempotency.

## What Is Idempotency?

Idempotency means the playbook can be run multiple times safely.

If Apache is already installed, Ansible will not reinstall it unnecessarily.

---

# Task 4: Remove Default Apache Index Page

```yaml
- name: Remove default Apache index page
  file:
    path: /var/www/html/index.html
    state: absent
```

## What This Does

This removes the default Apache welcome page.

Equivalent Linux command:

```bash
sudo rm -f /var/www/html/index.html
```

## Why This Is Needed

After Apache installation, Ubuntu creates a default page that says:

```text
Apache2 Default Page
It works!
```

We want to replace that page with our own custom page:

```text
Welcome to Ansible Class from SDHub
```

## Module Used

```yaml
file:
```

The `file` module is used to manage files, directories, ownership, and permissions.

## Explanation

```yaml
path: /var/www/html/index.html
```

This is the default Apache web page location.

```yaml
state: absent
```

This means the file should be removed.

---

# Task 5: Create Custom index.html Page

```yaml
- name: Create custom index.html page
  copy:
    dest: /var/www/html/index.html
    content: |
      <html>
      <head>
          <title>Ansible Class</title>
      </head>
      <body>
          <h1>Welcome to Ansible Class from SDHub</h1>
      </body>
      </html>
    owner: www-data
    group: www-data
    mode: '0644'
```

## What This Does

This creates a new custom `index.html` file on the worker nodes.

Instead of manually editing the file using `vi`, Ansible creates the file automatically.

## Module Used

```yaml
copy:
```

The `copy` module can copy files from the Ansible master to worker nodes.

It can also create a file directly using the `content` option.

## Destination File

```yaml
dest: /var/www/html/index.html
```

This is where the custom HTML file will be created.

Apache serves web pages from:

```text
/var/www/html/
```

So when you open the worker public IP in a browser, Apache reads this file.

## File Content

```yaml
content: |
```

This means we are writing the file content directly inside the playbook.

The `|` symbol allows multi-line content.

This content becomes the actual website page:

```html
<html>
<head>
    <title>Ansible Class</title>
</head>
<body>
    <h1>Welcome to Ansible Class from SDHub</h1>
</body>
</html>
```

## File Owner

```yaml
owner: www-data
```

This sets the file owner as `www-data`.

`www-data` is the default Apache user on Ubuntu.

## File Group

```yaml
group: www-data
```

This sets the file group as `www-data`.

## File Permission

```yaml
mode: '0644'
```

This sets the file permission.

Meaning:

```text
Owner  = read and write
Group  = read only
Others = read only
```

Linux permission format:

```text
-rw-r--r--
```

This permission is enough for Apache to read and serve the file.

---

# Task 6: Start and Enable Apache Service

```yaml
- name: Start and enable Apache service
  service:
    name: apache2
    state: restarted
    enabled: yes
```

## What This Does

This restarts Apache and enables it to start automatically after reboot.

Equivalent Linux commands:

```bash
sudo systemctl restart apache2
sudo systemctl enable apache2
```

## Module Used

```yaml
service:
```

The `service` module is used to manage Linux services.

## Service Name

```yaml
name: apache2
```

On Ubuntu, the Apache service name is:

```text
apache2
```

On Red Hat/CentOS/Amazon Linux, Apache is usually called:

```text
httpd
```

## Restart Apache

```yaml
state: restarted
```

This means Apache will be restarted.

We restart Apache to make sure it is running properly after the page is updated.

## Enable Apache on Boot

```yaml
enabled: yes
```

This means Apache will automatically start after a server reboot.

Equivalent command:

```bash
sudo systemctl enable apache2
```

---

# Simple Flow of This Playbook

```text
1. Connect to all worker nodes
2. Run tasks with sudo/root permission
3. Update apt package cache
4. Stop Nginx if it is using port 80
5. Install Apache
6. Delete the default Apache page
7. Create a custom index.html page
8. Restart Apache
9. Enable Apache on boot
10. Website is ready
```

---

# Command to Create the Playbook

Run this from your Ansible master EC2:

```bash
cat > install-apache.yml <<'EOF'
---
- name: Install Apache and deploy custom index page
  hosts: workers
  become: yes

  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Stop nginx if running
      shell: |
        systemctl stop nginx || true
        systemctl disable nginx || true

    - name: Install Apache web server
      apt:
        name: apache2
        state: present

    - name: Remove default Apache index page
      file:
        path: /var/www/html/index.html
        state: absent

    - name: Create custom index.html page
      copy:
        dest: /var/www/html/index.html
        content: |
          <html>
          <head>
              <title>Ansible Class</title>
          </head>
          <body>
              <h1>Welcome to Ansible Class from SDHub</h1>
          </body>
          </html>
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Start and enable Apache service
      service:
        name: apache2
        state: restarted
        enabled: yes
EOF
```

---

# Command to Check Playbook Syntax

```bash
ansible-playbook -i inventory.ini install-apache.yml --syntax-check
```

Expected output:

```text
playbook: install-apache.yml
```

---

# Command to Run the Playbook

```bash
ansible-playbook -i inventory.ini install-apache.yml
```

---

# Command to Verify Apache Status

```bash
ansible -i inventory.ini workers -m command -a "systemctl is-active apache2" --become
```

Expected output:

```text
active
```

---

# Command to Verify Web Page Content

```bash
ansible -i inventory.ini workers -m shell -a "curl -s http://localhost"
```

Expected output should contain:

```text
Welcome to Ansible Class from SDHub
```

---

# Open Website in Browser

Use worker public IP:

```text
http://<WORKER-PUBLIC-IP>
```

Example:

```text
http://35.173.202.190
```

You should see:

```text
Welcome to Ansible Class from SDHub
```

---

# Troubleshooting

## Issue: Apache Default Page Still Showing

Run:

```bash
ansible -i inventory.ini workers -m command -a "head -20 /var/www/html/index.html" --become
```

Then overwrite the file:

```bash
ansible -i inventory.ini workers -m copy -a "dest=/var/www/html/index.html content='<html><body><h1>Welcome to Ansible Class from SDHub</h1></body></html>' owner=www-data group=www-data mode=0644" --become
```

Restart Apache:

```bash
ansible -i inventory.ini workers -m service -a "name=apache2 state=restarted enabled=yes" --become
```

## Issue: Apache Fails to Start

Check Apache status:

```bash
ansible -i inventory.ini workers -m command -a "systemctl status apache2 --no-pager" --become
```

If you see:

```text
Address already in use: could not bind to address 0.0.0.0:80
```

It means another service, usually Nginx, is using port 80.

Check port 80:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

Stop Nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=stopped enabled=no" --become
```

Then restart Apache:

```bash
ansible -i inventory.ini workers -m service -a "name=apache2 state=restarted enabled=yes" --become
```

## Issue: Website Not Opening in Browser

If `curl http://localhost` works on workers but browser is not opening, check AWS Security Group.

Allow inbound HTTP:

| Type | Port | Source |
|---|---:|---|
| HTTP | 80 | Your IP or 0.0.0.0/0 for lab |

For lab only:

```text
HTTP  TCP  80  0.0.0.0/0
```

For safer access:

```text
HTTP  TCP  80  Your-Public-IP/32
```

---

# Notes for Students

- Ansible playbooks are written in YAML.
- Indentation is very important in YAML.
- Use spaces, not tabs.
- `hosts` decides where the playbook runs.
- `become: yes` means run with sudo/root permission.
- `tasks` contains the list of actions.
- `apt` installs packages on Ubuntu/Debian.
- `file` manages files and directories.
- `copy` copies or creates files.
- `service` starts, stops, restarts, or enables services.
- Playbooks are reusable and can be run multiple times safely.
