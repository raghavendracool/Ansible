# Deploy 2048 Game Using Ansible

This project demonstrates how to deploy a simple **2048 Game web application** on multiple AWS EC2 worker nodes using **Ansible**.

The game is deployed using:

```text
Nginx Web Server
Static HTML + CSS + JavaScript
Ansible Playbook
AWS EC2 Worker Nodes
```

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
Worker EC2 Nodes
   |
   | Nginx serves 2048 game
   v
Browser access using public IP
```

---

## Project Structure

```text
2048-ansible-app/
├── inventory.ini
├── deploy-2048-game.yml
└── app/
    └── index.html
```

---

## Prerequisites

Before running this project, make sure:

```text
1. Ansible is installed on the master EC2 instance
2. Passwordless SSH is configured from master to worker nodes
3. Worker EC2 private IPs are added in inventory.ini
4. Worker security group allows HTTP port 80
5. Worker nodes are Ubuntu-based servers
```

---

## 1. Create Project Directory

Run this on the **Ansible master EC2**:

```bash
mkdir -p ~/2048-ansible-app/app
cd ~/2048-ansible-app
```

---

## 2. Create 2048 Game HTML File

Create the application file:

```bash
cat > app/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>2048 Game - Ansible Deployment</title>
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
    <div class="subtitle">Deployed using Ansible - SDHub Class</div>

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

## 3. Create Inventory File

Use worker **private IPs** in the inventory file because Ansible master connects to worker nodes inside the AWS VPC.

```bash
cat > inventory.ini <<'EOF'
[workers]
worker1 ansible_host=<WORKER-1-PRIVATE-IP>
worker2 ansible_host=<WORKER-2-PRIVATE-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
EOF
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

## 4. Create Ansible Playbook

Create the deployment playbook:

```bash
cat > deploy-2048-game.yml <<'EOF'
---
- name: Deploy 2048 Game Application using Nginx
  hosts: workers
  become: yes

  tasks:
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Stop Apache if running on port 80
      shell: |
        systemctl stop apache2 || true
        systemctl disable apache2 || true

    - name: Install Nginx web server
      apt:
        name: nginx
        state: present

    - name: Remove default Nginx index page
      file:
        path: /var/www/html/index.nginx-debian.html
        state: absent

    - name: Remove default index.html if present
      file:
        path: /var/www/html/index.html
        state: absent

    - name: Deploy 2048 game index.html
      copy:
        src: app/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Start and enable Nginx service
      service:
        name: nginx
        state: restarted
        enabled: yes
EOF
```

---

## 5. Check Playbook Syntax

```bash
ansible-playbook -i inventory.ini deploy-2048-game.yml --syntax-check
```

Expected output:

```text
playbook: deploy-2048-game.yml
```

---

## 6. Deploy the 2048 Game

```bash
ansible-playbook -i inventory.ini deploy-2048-game.yml
```

---

## 7. Verify Nginx Service

```bash
ansible -i inventory.ini workers -m command -a "systemctl is-active nginx" --become
```

Expected output:

```text
active
```

---

## 8. Test from Ansible Master

Use worker private IPs:

```bash
curl http://<WORKER-1-PRIVATE-IP>
curl http://<WORKER-2-PRIVATE-IP>
```

You should see the HTML content of the 2048 game.

---

## 9. Open Game from Laptop Browser

Use worker **public IPs** in your browser:

```text
http://<WORKER-1-PUBLIC-IP>
http://<WORKER-2-PUBLIC-IP>
```

Example:

```text
http://35.173.202.190
http://54.92.191.146
```

---

## 10. AWS Security Group Rule

To access the game from a browser, the worker EC2 security group should allow HTTP port `80`.

| Type | Protocol | Port | Source         |
| ---- | -------- | ---: | -------------- |
| HTTP | TCP      |   80 | Your Public IP |

For lab only:

```text
HTTP  TCP  80  0.0.0.0/0
```

For safer access:

```text
HTTP  TCP  80  Your-Public-IP/32
```

---

## 11. Useful Verification Commands

Check worker connectivity:

```bash
ansible -i inventory.ini workers -m ping
```

Check Nginx status:

```bash
ansible -i inventory.ini workers -m command -a "systemctl status nginx --no-pager" --become
```

Check if port 80 is listening:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

Check deployed file:

```bash
ansible -i inventory.ini workers -m command -a "ls -l /var/www/html/index.html" --become
```

View deployed HTML:

```bash
ansible -i inventory.ini workers -m command -a "head -20 /var/www/html/index.html" --become
```

Restart Nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted enabled=yes" --become
```

---

## 12. Troubleshooting

### Issue: Browser is not opening the game

Check if Nginx is running:

```bash
ansible -i inventory.ini workers -m command -a "systemctl is-active nginx" --become
```

Check if Security Group allows HTTP port `80`.

---

### Issue: Apache page is still showing

Apache may still be running on port `80`.

Stop Apache:

```bash
ansible -i inventory.ini workers -m shell -a "sudo systemctl stop apache2 || true && sudo systemctl disable apache2 || true"
```

Restart Nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted enabled=yes" --become
```

---

### Issue: Port 80 already in use

Check which process is using port `80`:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

Stop Apache if needed:

```bash
ansible -i inventory.ini workers -m service -a "name=apache2 state=stopped enabled=no" --become
```

Restart Nginx:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=restarted enabled=yes" --become
```

---

## 13. Important Notes

```text
Ansible uses private IPs to connect to worker nodes.
Browser uses public IPs to access the website.
```

Example:

```text
Inventory IP  : 172.31.x.x
Browser URL   : http://<WORKER-PUBLIC-IP>
```

---

## 14. Final Command Summary

```bash
mkdir -p ~/2048-ansible-app/app
cd ~/2048-ansible-app

# Create app/index.html
# Create inventory.ini
# Create deploy-2048-game.yml

ansible -i inventory.ini workers -m ping

ansible-playbook -i inventory.ini deploy-2048-game.yml --syntax-check

ansible-playbook -i inventory.ini deploy-2048-game.yml

ansible -i inventory.ini workers -m command -a "systemctl is-active nginx" --become

curl http://<WORKER-PRIVATE-IP>
```

Open in browser:

```text
http://<WORKER-PUBLIC-IP>
```

---

## 15. Learning Outcome

After completing this project, students will understand:

```text
1. How to create a static web application
2. How to write an Ansible inventory file
3. How to write an Ansible playbook
4. How to install and start Nginx using Ansible
5. How to deploy website files using Ansible copy module
6. How to access the application from a browser
7. Difference between private IP and public IP usage
```
