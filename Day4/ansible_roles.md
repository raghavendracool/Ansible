# Ansible Roles and Ansible Galaxy - 2048 Game Deployment

This project explains how to deploy a **2048 Game web application** using **Ansible Roles** and **Ansible Galaxy**.

The target servers are Ubuntu worker nodes, and the application is deployed using Nginx.

---

## Architecture

```text
Local Laptop
   |
   | SSH
   v
Ansible Master EC2
   |
   | Ansible over SSH using private IP
   v
Ubuntu Worker Nodes
   |
   | Nginx serves 2048 game
   v
Browser access using public IP
```

---

## What is Ansible Role?

An **Ansible Role** is a standard folder structure used to organize Ansible automation.

Instead of writing all tasks in one big playbook, we split the automation into multiple folders such as:

```text
tasks
handlers
files
templates
defaults
vars
meta
```

### Simple Explanation

```text
Normal Playbook = One big YAML file
Role            = Organized reusable automation folder
```

Roles are used in real-time projects because they are easy to reuse, maintain, and share with teams.

---

## Why Do We Use Roles?

Roles help us to:

```text
Organize playbooks properly
Reuse automation code
Separate tasks, variables, files, and handlers
Make playbooks clean and small
Follow real-time DevOps project structure
Share automation with other teams
```

---

## What is Ansible Galaxy?

**Ansible Galaxy** is used to create, download, and share Ansible roles and collections.

### Create a role structure

```bash
ansible-galaxy init roles/nginx_2048
```

### Download a public role

```bash
ansible-galaxy role install geerlingguy.nginx
```

For this project, we use Ansible Galaxy to create the role folder structure.

---

# Project Structure

```text
2048-ansible-role-project/
├── inventory.ini
├── site.yml
└── roles/
    └── nginx_2048/
        ├── defaults/
        │   └── main.yml
        ├── files/
        │   └── index.html
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        ├── templates/
        ├── vars/
        │   └── main.yml
        ├── meta/
        │   └── main.yml
        ├── README.md
        └── tests/
```

---

# Role Folder Explanation

| Folder/File         | Purpose                                  |
| ------------------- | ---------------------------------------- |
| `tasks/main.yml`    | Main automation steps                    |
| `handlers/main.yml` | Restart/reload tasks triggered by notify |
| `defaults/main.yml` | Default variables                        |
| `vars/main.yml`     | Higher-priority variables                |
| `files/`            | Static files copied as-is                |
| `templates/`        | Dynamic files using Jinja2 variables     |
| `meta/main.yml`     | Role metadata and dependencies           |
| `README.md`         | Documentation for the role               |
| `tests/`            | Test files created by Ansible Galaxy     |

---

# Step 1: Create Project Directory

Run this on the Ansible master EC2:

```bash
mkdir -p ~/2048-ansible-role-project
cd ~/2048-ansible-role-project
```

---

# Step 2: Create Inventory File

Create `inventory.ini`.

```bash
cat > inventory.ini <<'EOF'
[workers]
server1 ansible_host=<WORKER-1-PRIVATE-IP>
server2 ansible_host=<WORKER-2-PRIVATE-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
EOF
```

Example:

```ini
[workers]
server1 ansible_host=172.31.41.142
server2 ansible_host=172.31.32.217

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

Test connectivity:

```bash
ansible -i inventory.ini workers -m ping
```

Expected output:

```text
server1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Step 3: Create Role Using Ansible Galaxy

Create `roles` directory:

```bash
mkdir roles
```

Create role structure:

```bash
ansible-galaxy init roles/nginx_2048
```

Check role structure:

```bash
tree roles/nginx_2048
```

If `tree` is not installed:

```bash
sudo apt install tree -y
tree roles/nginx_2048
```

---

# Step 4: Create Default Variables

File:

```text
roles/nginx_2048/defaults/main.yml
```

Create content:

```bash
cat > roles/nginx_2048/defaults/main.yml <<'EOF'
---
web_package: nginx
web_service: nginx
web_root: /var/www/html
app_file: index.html
EOF
```

## Explanation

```yaml
web_package: nginx
```

Package to install.

```yaml
web_service: nginx
```

Service to start/restart.

```yaml
web_root: /var/www/html
```

Default Nginx web root path on Ubuntu.

```yaml
app_file: index.html
```

Main application file.

---

# Step 5: Add 2048 Game File

File:

```text
roles/nginx_2048/files/index.html
```

Create a simple 2048 game page:

```bash
cat > roles/nginx_2048/files/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>2048 Game - Ansible Role Deployment</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #faf8ef;
            text-align: center;
            margin: 0;
            padding: 20px;
        }

        h1 {
            font-size: 60px;
            color: #776e65;
            margin-bottom: 10px;
        }

        .subtitle {
            color: #776e65;
            font-size: 18px;
            margin-bottom: 20px;
        }

        .score-board {
            font-size: 24px;
            color: white;
            background: #bbada0;
            display: inline-block;
            padding: 12px 25px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .game-container {
            width: 420px;
            height: 420px;
            background: #bbada0;
            margin: 20px auto;
            border-radius: 10px;
            padding: 10px;
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
        }

        .tile {
            width: 95px;
            height: 95px;
            background: #cdc1b4;
            border-radius: 6px;
            font-size: 35px;
            font-weight: bold;
            color: #776e65;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .tile-2 { background: #eee4da; }
        .tile-4 { background: #ede0c8; }
        .tile-8 { background: #f2b179; color: white; }
        .tile-16 { background: #f59563; color: white; }
        .tile-32 { background: #f67c5f; color: white; }
        .tile-64 { background: #f65e3b; color: white; }
        .tile-128 { background: #edcf72; color: white; font-size: 30px; }
        .tile-256 { background: #edcc61; color: white; font-size: 30px; }
        .tile-512 { background: #edc850; color: white; font-size: 30px; }
        .tile-1024 { background: #edc53f; color: white; font-size: 25px; }
        .tile-2048 { background: #edc22e; color: white; font-size: 25px; }

        button {
            background: #8f7a66;
            color: white;
            border: none;
            padding: 12px 25px;
            font-size: 18px;
            border-radius: 6px;
            cursor: pointer;
        }

        button:hover {
            background: #776e65;
        }

        .footer {
            margin-top: 20px;
            color: #776e65;
            font-size: 16px;
        }
    </style>
</head>
<body>

    <h1>2048</h1>
    <div class="subtitle">Deployed using Ansible Roles - SDHub Class</div>

    <div class="score-board">
        Score: <span id="score">0</span>
    </div>

    <div class="game-container" id="game"></div>

    <button onclick="restartGame()">Restart Game</button>

    <div class="footer">
        Use Arrow Keys to Play
    </div>

    <script>
        let board;
        let score = 0;

        function startGame() {
            board = [
                [0, 0, 0, 0],
                [0, 0, 0, 0],
                [0, 0, 0, 0],
                [0, 0, 0, 0]
            ];
            score = 0;
            addNumber();
            addNumber();
            drawBoard();
        }

        function drawBoard() {
            const game = document.getElementById("game");
            game.innerHTML = "";
            document.getElementById("score").innerText = score;

            for (let r = 0; r < 4; r++) {
                for (let c = 0; c < 4; c++) {
                    const tile = document.createElement("div");
                    tile.classList.add("tile");

                    if (board[r][c] !== 0) {
                        tile.innerText = board[r][c];
                        tile.classList.add("tile-" + board[r][c]);
                    }

                    game.appendChild(tile);
                }
            }
        }

        function addNumber() {
            let empty = [];

            for (let r = 0; r < 4; r++) {
                for (let c = 0; c < 4; c++) {
                    if (board[r][c] === 0) {
                        empty.push({ r, c });
                    }
                }
            }

            if (empty.length === 0) return;

            let random = empty[Math.floor(Math.random() * empty.length)];
            board[random.r][random.c] = Math.random() < 0.9 ? 2 : 4;
        }

        function slide(row) {
            row = row.filter(num => num !== 0);

            for (let i = 0; i < row.length - 1; i++) {
                if (row[i] === row[i + 1]) {
                    row[i] *= 2;
                    score += row[i];
                    row[i + 1] = 0;
                }
            }

            row = row.filter(num => num !== 0);

            while (row.length < 4) {
                row.push(0);
            }

            return row;
        }

        function moveLeft() {
            let oldBoard = JSON.stringify(board);

            for (let r = 0; r < 4; r++) {
                board[r] = slide(board[r]);
            }

            if (JSON.stringify(board) !== oldBoard) {
                addNumber();
                drawBoard();
            }
        }

        function moveRight() {
            let oldBoard = JSON.stringify(board);

            for (let r = 0; r < 4; r++) {
                board[r].reverse();
                board[r] = slide(board[r]);
                board[r].reverse();
            }

            if (JSON.stringify(board) !== oldBoard) {
                addNumber();
                drawBoard();
            }
        }

        function moveUp() {
            let oldBoard = JSON.stringify(board);

            for (let c = 0; c < 4; c++) {
                let row = [board[0][c], board[1][c], board[2][c], board[3][c]];
                row = slide(row);

                for (let r = 0; r < 4; r++) {
                    board[r][c] = row[r];
                }
            }

            if (JSON.stringify(board) !== oldBoard) {
                addNumber();
                drawBoard();
            }
        }

        function moveDown() {
            let oldBoard = JSON.stringify(board);

            for (let c = 0; c < 4; c++) {
                let row = [board[0][c], board[1][c], board[2][c], board[3][c]];
                row.reverse();
                row = slide(row);
                row.reverse();

                for (let r = 0; r < 4; r++) {
                    board[r][c] = row[r];
                }
            }

            if (JSON.stringify(board) !== oldBoard) {
                addNumber();
                drawBoard();
            }
        }

        document.addEventListener("keydown", function(event) {
            if (event.key === "ArrowLeft") moveLeft();
            else if (event.key === "ArrowRight") moveRight();
            else if (event.key === "ArrowUp") moveUp();
            else if (event.key === "ArrowDown") moveDown();
        });

        function restartGame() {
            startGame();
        }

        startGame();
    </script>

</body>
</html>
EOF
```

---

# Step 6: Create Role Tasks

File:

```text
roles/nginx_2048/tasks/main.yml
```

Create content:

```bash
cat > roles/nginx_2048/tasks/main.yml <<'EOF'
---
- name: Update apt package cache
  apt:
    update_cache: yes

- name: Stop Apache if running
  shell: |
    systemctl stop apache2 || true
    systemctl disable apache2 || true

- name: Install Nginx package
  apt:
    name: "{{ web_package }}"
    state: present

- name: Remove default Nginx index page
  file:
    path: "{{ web_root }}/index.nginx-debian.html"
    state: absent

- name: Remove old index.html if present
  file:
    path: "{{ web_root }}/{{ app_file }}"
    state: absent

- name: Deploy 2048 game file
  copy:
    src: "{{ app_file }}"
    dest: "{{ web_root }}/{{ app_file }}"
    owner: www-data
    group: www-data
    mode: '0644'
  notify: Restart nginx

- name: Ensure Nginx service is started and enabled
  service:
    name: "{{ web_service }}"
    state: started
    enabled: yes
EOF
```

---

# Step 7: Create Handler

File:

```text
roles/nginx_2048/handlers/main.yml
```

Create content:

```bash
cat > roles/nginx_2048/handlers/main.yml <<'EOF'
---
- name: Restart nginx
  service:
    name: "{{ web_service }}"
    state: restarted
EOF
```

## What is a Handler?

A handler is a special task that runs only when notified.

In our task:

```yaml
notify: Restart nginx
```

This means:

```text
If index.html changes, restart nginx.
If index.html does not change, do not restart nginx.
```

This is useful in real-time because we restart services only when needed.

---

# Step 8: Create Main Playbook

File:

```text
site.yml
```

Create content:

```bash
cat > site.yml <<'EOF'
---
- name: Deploy 2048 Game using Ansible Role
  hosts: workers
  become: yes

  roles:
    - nginx_2048
EOF
```

## Explanation

This playbook is very small because the logic is inside the role.

```yaml
roles:
  - nginx_2048
```

This tells Ansible to run the role named `nginx_2048`.

---

# Step 9: Syntax Check

```bash
ansible-playbook -i inventory.ini site.yml --syntax-check
```

Expected output:

```text
playbook: site.yml
```

---

# Step 10: Run Deployment

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# Step 11: Verify Deployment

Check Nginx:

```bash
ansible -i inventory.ini workers -m shell -a "systemctl is-active nginx || true" --become
```

Check port 80:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

Test from Ansible master using worker private IP:

```bash
curl http://<WORKER-PRIVATE-IP>
```

Open from laptop browser using worker public IP:

```text
http://<WORKER-PUBLIC-IP>
```

---

# Difference Between Normal Playbook and Role

## Normal Playbook

```yaml
---
- name: Deploy app
  hosts: workers
  become: yes

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

## Role-Based Playbook

```yaml
---
- name: Deploy app
  hosts: workers
  become: yes

  roles:
    - nginx_2048
```

In role-based structure, tasks are stored inside:

```text
roles/nginx_2048/tasks/main.yml
```

---

