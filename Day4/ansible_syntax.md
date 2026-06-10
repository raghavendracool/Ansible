# How to Write Ansible Playbook in Correct Syntax

This section is useful for beginners.

---

## Basic Ansible Playbook Structure

```yaml
---
- name: Play name
  hosts: target_group
  become: yes

  tasks:
    - name: Task name
      module_name:
        key1: value1
        key2: value2
```

---

## Simple Example

```yaml
---
- name: Install Nginx on Ubuntu servers
  hosts: workers
  become: yes

  tasks:
    - name: Install nginx package
      apt:
        name: nginx
        state: present

    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## Explanation of Important Keywords

| Keyword     | Meaning                                      |
| ----------- | -------------------------------------------- |
| `---`       | Start of YAML file                           |
| `- name:`   | Name of play or task                         |
| `hosts:`    | Target group from inventory                  |
| `become:`   | Run with sudo/root permission                |
| `tasks:`    | List of tasks to execute                     |
| `apt:`      | Module to install packages on Ubuntu         |
| `service:`  | Module to manage services                    |
| `copy:`     | Module to copy/create files                  |
| `file:`     | Module to create/delete files or directories |
| `notify:`   | Calls a handler                              |
| `handlers:` | Special tasks that run when notified         |

---

## YAML Rules for Ansible

```text
1. YAML uses key: value format
2. Indentation is very important
3. Use spaces, not tabs
4. Use 2 spaces for indentation
5. Lists start with hyphen -
6. Strings can be written with or without quotes
7. Boolean values are yes/no or true/false
```

---

## Correct Indentation Example

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
```

Explanation:

```text
tasks:                starts task section
  - name:             first task
    apt:              module
      name: nginx     module argument
      state: present  module argument
```

---

## Wrong Indentation Example

```yaml
tasks:
- name: Install nginx
apt:
name: nginx
state: present
```

This is wrong because YAML cannot understand the hierarchy.

---

## How to Remember Playbook Syntax

Use this simple formula:

```text
PLAYBOOK
  hosts
  become
  tasks
    task name
    module
      module options
```

Example:

```yaml
---
- name: My first playbook
  hosts: workers
  become: yes

  tasks:
    - name: Install package
      apt:
        name: nginx
        state: present
```

---

## String, Integer, Boolean in YAML

### String

Text value.

```yaml
name: Raghavendra
job: DevOps Engineer
package: nginx
```

### Integer

Number value.

```yaml
port: 80
replicas: 2
```

### Boolean

True/false value.

```yaml
become: yes
enabled: yes
debug: false
```

---

## List in YAML

A list contains multiple values.

```yaml
packages:
  - nginx
  - curl
  - unzip
  - net-tools
```

Example in Ansible:

```yaml
- name: Install multiple packages
  apt:
    name:
      - nginx
      - curl
      - unzip
    state: present
```

---

## Variables in Playbook

Variables help avoid hardcoding.

```yaml
vars:
  web_package: nginx
  web_service: nginx
  web_root: /var/www/html
```

Use variables like this:

```yaml
name: "{{ web_package }}"
path: "{{ web_root }}/index.html"
```

Example:

```yaml
---
- name: Install package using variable
  hosts: workers
  become: yes

  vars:
    web_package: nginx

  tasks:
    - name: Install web package
      apt:
        name: "{{ web_package }}"
        state: present
```

---

## Handler Syntax

Handlers are used to restart services only when needed.

```yaml
tasks:
  - name: Copy website file
    copy:
      src: index.html
      dest: /var/www/html/index.html
    notify: Restart nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

Explanation:

```text
If copy task changes the file, handler runs.
If copy task does not change the file, handler does not run.
```

---

## Syntax Check Command

Before running any playbook, always check syntax:

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

---

## Dry Run Command

To see what will change without actually applying changes:

```bash
ansible-playbook -i inventory.ini site.yml --check
```

---

## Run Playbook

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# Common Syntax Mistakes

## Mistake 1: Using Tabs

Wrong:

```text
Using tab spaces
```

Correct:

```text
Use normal spaces only
```

---

## Mistake 2: Missing Colon

Wrong:

```yaml
hosts workers
```

Correct:

```yaml
hosts: workers
```

---

## Mistake 3: Wrong Indentation

Wrong:

```yaml
apt:
name: nginx
state: present
```

Correct:

```yaml
apt:
  name: nginx
  state: present
```

---

## Mistake 4: Forgetting Hyphen for Task

Wrong:

```yaml
tasks:
  name: Install nginx
  apt:
    name: nginx
```

Correct:

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
```

---

## Mistake 5: Wrong Variable Usage

Wrong:

```yaml
name: web_package
```

Correct:

```yaml
name: "{{ web_package }}"
```

---

