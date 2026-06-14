# Fully Automated AWS EC2 + Ansible + Ansible Galaxy Project

This project creates a complete Ansible lab environment on AWS and deploys a **2048 web application** using **Ansible Roles** created through **Ansible Galaxy**.

The complete setup is automated using Ansible.

---

## Project Objective

The goal of this project is to automate the following tasks:

```text
1. Create AWS infrastructure using Ansible
2. Use the default VPC
3. Create 1 Ansible master EC2 instance
4. Create 3 Ubuntu worker EC2 instances
5. Use existing AWS key pair named ansible
6. Install Ansible automatically on the master node
7. Create inventory.ini automatically using worker private IPs
8. Copy ansible.pem to the master node automatically
9. Configure master-to-worker SSH access
10. Create Ansible role using ansible-galaxy
11. Deploy 2048 application on all worker nodes
12. Access the application from browser using worker public IPs
```

---

## Architecture

```text
Controller EC2
    |
    | Runs Ansible AWS automation
    |
    v
Creates AWS Infrastructure
    |
    |-------------------------
    |                        |
    v                        v
Ansible Master EC2       Worker EC2 Nodes
                         worker1
                         worker2
                         worker3

Ansible Master connects to worker nodes using private IPs.

Users access the application using worker public IPs.
```

---

## Servers Used

| Server             | Purpose                           |
| ------------------ | --------------------------------- |
| Controller EC2     | Runs the main automation playbook |
| Ansible Master EC2 | Runs Ansible against worker nodes |
| Worker EC2 1       | Hosts 2048 app using Nginx        |
| Worker EC2 2       | Hosts 2048 app using Nginx        |
| Worker EC2 3       | Hosts 2048 app using Nginx        |

---

## What is Ansible Collection?

An **Ansible Collection** is a packaged set of Ansible content.

A collection can include:

```text
Modules
Roles
Plugins
Playbooks
Documentation
```

Earlier, many Ansible modules were built directly into Ansible. Now, many cloud-specific and third-party modules are delivered through collections.

---

## Why Are We Using Collections?

In this project, we are working with AWS resources like:

```text
EC2 instances
Security groups
Default VPC
Subnets
Tags
```

For this, we need AWS-specific Ansible modules.

Those AWS modules are available inside the `amazon.aws` collection.

Without this collection, Ansible will not understand modules like:

```yaml
amazon.aws.ec2_instance
amazon.aws.ec2_security_group
amazon.aws.ec2_vpc_net_info
amazon.aws.ec2_vpc_subnet_info
```

---

## Collections Used in This Project

```yaml
collections:
  - name: amazon.aws
  - name: community.general
  - name: community.crypto
```

### 1. amazon.aws

Used for AWS automation.

We use it for:

```text
Creating EC2 instances
Creating security groups
Reading default VPC details
Reading subnet details
Managing AWS infrastructure
```

Example modules:

```yaml
amazon.aws.ec2_instance
amazon.aws.ec2_security_group
amazon.aws.ec2_vpc_net_info
amazon.aws.ec2_vpc_subnet_info
```

### 2. community.general

Used for additional community-supported modules.

It is commonly used in real-time Ansible projects.

### 3. community.crypto

Used when we want to create or manage SSH keys, certificates, and crypto-related resources.

In the final version of this project, we are using the existing AWS key pair `ansible`, but this collection is still useful for future improvements.

---

## What is Ansible Galaxy?

**Ansible Galaxy** is used to install collections and roles.

We use it in two ways:

### 1. Install collections

```bash
ansible-galaxy collection install -r requirements.yml
```

### 2. Create role structure

```bash
ansible-galaxy init roles/nginx_2048
```

This command creates the standard role folder structure.

---

## What is Ansible Role?

An **Ansible Role** is a standard way to organize playbooks.

Instead of writing everything inside one big playbook, we split the automation into folders.

Example role structure:

```text
roles/nginx_2048/
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
├── meta/
└── README.md
```

---

## Why Use Roles?

Roles help us to:

```text
Organize automation properly
Reuse code
Keep playbooks clean
Separate variables, files, handlers, and tasks
Follow real-time DevOps project structure
Share automation with teams
```

---

## Project Repository Structure

```text
full-aws-ansible-galaxy-2048/
├── README.md
├── requirements.yml
├── create-full-lab.yml
└── group_vars/
    └── all.yml
```

After automation runs, the Ansible master will contain:

```text
/home/ubuntu/2048-role-project/
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
        └── tasks/
            └── main.yml
```

---

# Fresh Setup Steps

Use these steps if you delete everything and want to start fresh.

---

## Step 1: Create One Controller EC2

Create one Ubuntu EC2 instance manually.

This server is called:

```text
controller EC2
```

This controller will run the main Ansible automation playbook.

Recommended instance type:

```text
t3.micro
```

Operating system:

```text
Ubuntu 24.04
```

---

## Step 2: Attach IAM Permission to Controller

The controller needs permission to create AWS resources.

For lab practice, attach this policy to the IAM user or IAM role:

```text
AmazonEC2FullAccess
```

Better production approach:

```text
Use IAM Role attached to the Controller EC2.
Avoid long-term access keys.
Use least privilege permissions.
```

---

## Step 3: Configure AWS Access

If using IAM role attached to EC2, no need to run:

```bash
aws configure
```

If using IAM user access key for lab, configure AWS CLI:

```bash
aws configure
```

Provide:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name: us-east-1
Default output format: json
```

Verify access:

```bash
aws sts get-caller-identity
```

Expected output:

```json
{
  "UserId": "xxxx",
  "Account": "xxxxxxxxxxxx",
  "Arn": "arn:aws:iam::xxxxxxxxxxxx:user/testinguser"
}
```

---

## Step 4: Copy Existing AWS Key Pair File

This project uses an existing AWS key pair:

```text
Key pair name: ansible
Private key file: ansible.pem
```

Copy `ansible.pem` to the controller EC2.

Expected path:

```text
/home/ubuntu/ansible.pem
```

Set permission:

```bash
chmod 400 /home/ubuntu/ansible.pem
```

Verify:

```bash
ls -l /home/ubuntu/ansible.pem
```

Expected:

```text
-r-------- 1 ubuntu ubuntu ansible.pem
```

---

## Step 5: Install Required Tools on Controller

Run on controller EC2:

```bash
sudo apt update
sudo apt install ansible python3-pip awscli -y
```

Install boto libraries:

```bash
sudo apt install python3-boto3 python3-botocore -y
```

Check versions:

```bash
ansible --version
aws --version
python3 --version
```

---

## Step 6: Create Project Directory

```bash
mkdir -p ~/full-aws-ansible-galaxy-2048/group_vars
cd ~/full-aws-ansible-galaxy-2048
```

---

## Step 7: Create requirements.yml

Create the Ansible Galaxy requirements file:

```bash
cat > requirements.yml <<'EOF'
---
collections:
  - name: amazon.aws
  - name: community.general
  - name: community.crypto
EOF
```

Install collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Verify:

```bash
ansible-galaxy collection list | grep -E "amazon.aws|community.general|community.crypto"
```

---

## Step 8: Create Variables File

Create:

```bash
cat > group_vars/all.yml <<'EOF'
---
aws_region: us-east-1
aws_key_name: ansible
pem_private_key_path: /home/ubuntu/ansible.pem

instance_type: t3.micro
ubuntu_ami_id: ami-020cba7c55df1f615

security_group_name: ansible-full-auto-sg

master_name: ansible-master-auto

worker_names:
  - ansible-worker-auto-1
  - ansible-worker-auto-2
  - ansible-worker-auto-3

ssh_user: ubuntu
project_dir: /home/ubuntu/2048-role-project
EOF
```

---

## Variable Explanation

| Variable               | Meaning                                |
| ---------------------- | -------------------------------------- |
| `aws_region`           | AWS region where infra will be created |
| `aws_key_name`         | Existing AWS key pair name             |
| `pem_private_key_path` | Path of private key file on controller |
| `instance_type`        | EC2 instance size                      |
| `ubuntu_ami_id`        | Ubuntu AMI ID                          |
| `security_group_name`  | Security group name                    |
| `master_name`          | Name tag for Ansible master            |
| `worker_names`         | Name tags for worker nodes             |
| `ssh_user`             | SSH username for Ubuntu                |
| `project_dir`          | Project path created on Ansible master |

---

# Step 9: Create Main Automation Playbook

Create:

```bash
cat > create-full-lab.yml <<'EOF'
---
- name: Fully automate AWS Ansible Galaxy 2048 lab using existing ansible key pair
  hosts: localhost
  connection: local
  gather_facts: no

  vars_files:
    - group_vars/all.yml

  tasks:
    - name: Check ansible.pem exists on controller
      stat:
        path: "{{ pem_private_key_path }}"
      register: pem_file

    - name: Fail if ansible.pem is missing
      fail:
        msg: "Private key file {{ pem_private_key_path }} not found."
      when: not pem_file.stat.exists

    - name: Read ansible.pem private key
      slurp:
        src: "{{ pem_private_key_path }}"
      register: ansible_pem_content

    - name: Get default VPC
      amazon.aws.ec2_vpc_net_info:
        region: "{{ aws_region }}"
        filters:
          isDefault: "true"
      register: default_vpc

    - name: Get default subnets
      amazon.aws.ec2_vpc_subnet_info:
        region: "{{ aws_region }}"
        filters:
          vpc-id: "{{ default_vpc.vpcs[0].vpc_id }}"
      register: default_subnets

    - name: Create security group
      amazon.aws.ec2_security_group:
        name: "{{ security_group_name }}"
        description: Fully automated Ansible lab security group
        region: "{{ aws_region }}"
        vpc_id: "{{ default_vpc.vpcs[0].vpc_id }}"
        rules:
          - proto: tcp
            ports:
              - 22
            cidr_ip: 0.0.0.0/0
            rule_desc: SSH lab access
          - proto: tcp
            ports:
              - 80
            cidr_ip: 0.0.0.0/0
            rule_desc: HTTP access for app
        rules_egress:
          - proto: -1
            cidr_ip: 0.0.0.0/0

    - name: Launch worker instances
      amazon.aws.ec2_instance:
        name: "{{ item }}"
        region: "{{ aws_region }}"
        key_name: "{{ aws_key_name }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ubuntu_ami_id }}"
        wait: yes
        count: 1
        vpc_subnet_id: "{{ default_subnets.subnets[0].subnet_id }}"
        security_group: "{{ security_group_name }}"
        network:
          assign_public_ip: true
        tags:
          Project: full-aws-ansible-galaxy-2048
          Role: worker
      loop: "{{ worker_names }}"
      register: worker_result

    - name: Collect worker private IPs
      set_fact:
        worker_private_ips: "{{ worker_result.results | map(attribute='instances') | flatten | map(attribute='private_ip_address') | list }}"

    - name: Create inventory content for master
      set_fact:
        generated_inventory: |
          [workers]
          {% for ip in worker_private_ips %}
          worker{{ loop.index }} ansible_host={{ ip }}
          {% endfor %}

          [workers:vars]
          ansible_user=ubuntu
          ansible_ssh_private_key_file=/home/ubuntu/.ssh/ansible.pem
          ansible_python_interpreter=/usr/bin/python3

    - name: Build master user data
      set_fact:
        master_user_data: |
          #!/bin/bash
          apt update -y
          apt install software-properties-common tree curl -y
          add-apt-repository --yes --update ppa:ansible/ansible
          apt install ansible -y

          mkdir -p /home/ubuntu/.ssh

          cat > /home/ubuntu/.ssh/ansible.pem <<'KEYEOF'
          {{ ansible_pem_content.content | b64decode }}
          KEYEOF

          chmod 400 /home/ubuntu/.ssh/ansible.pem
          chown -R ubuntu:ubuntu /home/ubuntu/.ssh

          mkdir -p {{ project_dir }}/roles

          cat > {{ project_dir }}/inventory.ini <<'INVEOF'
          {{ generated_inventory }}
          INVEOF

          cd {{ project_dir }}

          ansible-galaxy init roles/nginx_2048

          cat > roles/nginx_2048/defaults/main.yml <<'DEFEOF'
          ---
          web_package: nginx
          web_service: nginx
          web_root: /var/www/html
          app_file: index.html
          DEFEOF

          cat > roles/nginx_2048/files/index.html <<'HTMLEOF'
          <!DOCTYPE html>
          <html>
          <head>
            <title>2048 App - Ansible Galaxy</title>
            <style>
              body {
                font-family: Arial;
                background: #faf8ef;
                text-align: center;
                padding: 40px;
              }
              .box {
                background: white;
                padding: 30px;
                margin: auto;
                width: 600px;
                border-radius: 12px;
                box-shadow: 0 2px 10px rgba(0,0,0,0.2);
              }
              h1 {
                font-size: 70px;
                color: #776e65;
              }
              button {
                padding: 12px 25px;
                background: #8f7a66;
                color: white;
                border: none;
                border-radius: 6px;
                font-size: 18px;
                cursor: pointer;
              }
            </style>
          </head>
          <body>
            <div class="box">
              <h1>2048</h1>
              <h2>Fully Automated Deployment</h2>
              <h3>AWS EC2 + Ansible + Ansible Galaxy Role</h3>
              <p>This application is deployed on worker nodes using Ansible roles.</p>
              <button onclick="alert('Automation Successful!')">Test App</button>
            </div>
          </body>
          </html>
          HTMLEOF

          cat > roles/nginx_2048/tasks/main.yml <<'TASKEOF'
          {% raw %}
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

          - name: Deploy 2048 application file
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
          {% endraw %}
          TASKEOF

          cat > roles/nginx_2048/handlers/main.yml <<'HANDLEREOF'
          {% raw %}
          ---
          - name: Restart nginx
            service:
              name: "{{ web_service }}"
              state: restarted
          {% endraw %}
          HANDLEREOF

          cat > site.yml <<'SITEEOF'
          ---
          - name: Deploy 2048 Application using Ansible Galaxy Role
            hosts: workers
            become: yes

            roles:
              - nginx_2048
          SITEEOF

          chown -R ubuntu:ubuntu {{ project_dir }}

          sleep 90

          sudo -u ubuntu ansible -i {{ project_dir }}/inventory.ini workers -m ping
          sudo -u ubuntu ansible-playbook -i {{ project_dir }}/inventory.ini {{ project_dir }}/site.yml

    - name: Launch Ansible master instance
      amazon.aws.ec2_instance:
        name: "{{ master_name }}"
        region: "{{ aws_region }}"
        key_name: "{{ aws_key_name }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ubuntu_ami_id }}"
        wait: yes
        count: 1
        vpc_subnet_id: "{{ default_subnets.subnets[0].subnet_id }}"
        security_group: "{{ security_group_name }}"
        network:
          assign_public_ip: true
        user_data: "{{ master_user_data }}"
        tags:
          Project: full-aws-ansible-galaxy-2048
          Role: master
      register: master_result

    - name: Show master public IP
      debug:
        msg: "Master Public IP: {{ master_result.instances[0].public_ip_address }}"

    - name: Show worker public IPs
      debug:
        msg: "{{ item.tags.Name }} Public IP: {{ item.public_ip_address }}"
      loop: "{{ worker_result.results | map(attribute='instances') | flatten | list }}"

    - name: Show worker private IPs
      debug:
        msg: "{{ item.tags.Name }} Private IP: {{ item.private_ip_address }}"
      loop: "{{ worker_result.results | map(attribute='instances') | flatten | list }}"
EOF
```

---

# Step 10: Syntax Check

```bash
ansible-playbook create-full-lab.yml --syntax-check
```

Expected output:

```text
playbook: create-full-lab.yml
```

---

# Step 11: Execute Full Automation

```bash
ansible-playbook create-full-lab.yml
```

This playbook performs the following tasks:

```text
1. Checks ansible.pem exists on controller
2. Reads ansible.pem private key
3. Gets default VPC information
4. Gets default subnet information
5. Creates security group
6. Launches 3 worker EC2 instances
7. Collects worker private IPs
8. Generates inventory.ini content
9. Creates master user-data script
10. Launches Ansible master EC2 instance
11. Installs Ansible on master using user-data
12. Copies ansible.pem to master
13. Creates 2048-role-project on master
14. Creates nginx_2048 role using ansible-galaxy
15. Creates index.html application file
16. Creates role task file
17. Creates role handler file
18. Creates site.yml playbook
19. Tests worker connectivity
20. Deploys 2048 app on worker nodes
```

---

# Step 12: SSH to Ansible Master

After successful run, the playbook prints the master public IP.

SSH to master:

```bash
ssh -i /home/ubuntu/ansible.pem ubuntu@<MASTER_PUBLIC_IP>
```

Example:

```bash
ssh -i /home/ubuntu/ansible.pem ubuntu@3.230.119.208
```

---

# Step 13: Check cloud-init Status on Master

On master:

```bash
sudo cloud-init status --long
```

If it is running:

```bash
sudo tail -f /var/log/cloud-init-output.log
```

Once complete, verify the project folder:

```bash
ls -lrth
cd /home/ubuntu/2048-role-project
ls -lrth
```

---

# Step 14: Check Generated Inventory

On master:

```bash
cat inventory.ini
```

Expected:

```ini
[workers]
worker1 ansible_host=<WORKER-1-PRIVATE-IP>
worker2 ansible_host=<WORKER-2-PRIVATE-IP>
worker3 ansible_host=<WORKER-3-PRIVATE-IP>

[workers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/.ssh/ansible.pem
ansible_python_interpreter=/usr/bin/python3
```

---

# Step 15: Test Connectivity from Master to Workers

```bash
ansible -i inventory.ini workers -m ping
```

Expected:

```text
worker1 | SUCCESS
worker2 | SUCCESS
worker3 | SUCCESS
```

---

# Step 16: Deploy 2048 App Manually if Needed

The app deployment is already triggered from user-data.

But for verification, run:

```bash
ansible-playbook -i inventory.ini site.yml
```

---

# Step 17: Verify Nginx on Worker Nodes

```bash
ansible -i inventory.ini workers -m shell -a "systemctl is-active nginx || true" --become
```

Expected output:

```text
active
active
active
```

Check port 80:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'"
```

---

# Step 18: Get Worker Public IPs

From controller EC2, run:

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Project,Values=full-aws-ansible-galaxy-2048" "Name=tag:Role,Values=worker" "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[Tags[?Key=='Name'].Value|[0],PrivateIpAddress,PublicIpAddress]" \
  --output table
```

Open worker public IPs in browser:

```text
http://<WORKER-1-PUBLIC-IP>
http://<WORKER-2-PUBLIC-IP>
http://<WORKER-3-PUBLIC-IP>
```

---

# How Ansible Master Connects to Workers

The master uses worker private IPs.

Inventory example:

```ini
[workers]
worker1 ansible_host=172.31.x.x
worker2 ansible_host=172.31.x.x
worker3 ansible_host=172.31.x.x
```

The master uses this private key:

```text
/home/ubuntu/.ssh/ansible.pem
```

This key is copied to the master automatically using user-data.

---

# Security Group Rules

The playbook creates a security group with:

| Port | Purpose         | Source    |
| ---- | --------------- | --------- |
| 22   | SSH access      | 0.0.0.0/0 |
| 80   | HTTP app access | 0.0.0.0/0 |

For lab, this is okay.

For production, restrict:

```text
SSH: only your IP or bastion security group
HTTP: load balancer security group or trusted CIDR
```

---

# Important Security Notes

This project is for lab and training.

For production:

```text
Do not open SSH to 0.0.0.0/0
Do not copy private keys across servers
Use IAM roles instead of access keys
Use AWS Systems Manager Session Manager
Use private subnets for worker nodes
Use ALB instead of direct public IP access
Use least privilege IAM policies
Use Ansible Vault or AWS Secrets Manager for secrets
```

---

# Cleanup

To delete all lab EC2 instances:

```bash
aws ec2 terminate-instances \
  --region us-east-1 \
  --instance-ids $(aws ec2 describe-instances \
    --region us-east-1 \
    --filters "Name=tag:Project,Values=full-aws-ansible-galaxy-2048" "Name=instance-state-name,Values=pending,running,stopping,stopped" \
    --query "Reservations[*].Instances[*].InstanceId" \
    --output text)
```

To check remaining lab instances:

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Project,Values=full-aws-ansible-galaxy-2048" "Name=instance-state-name,Values=pending,running,stopped" \
  --query "Reservations[*].Instances[*].[InstanceId,Tags[?Key=='Name'].Value|[0],State.Name,PublicIpAddress]" \
  --output table
```

---

# Troubleshooting

## Issue 1: Permission denied publickey

Make sure you are using the correct key:

```bash
ssh -i /home/ubuntu/ansible.pem ubuntu@<MASTER_PUBLIC_IP>
```

Also check permission:

```bash
chmod 400 /home/ubuntu/ansible.pem
```

---

## Issue 2: Project folder not created on master

Check cloud-init:

```bash
sudo cloud-init status --long
sudo tail -n 200 /var/log/cloud-init-output.log
```

If cloud-init is still running, wait:

```bash
sudo tail -f /var/log/cloud-init-output.log
```

---

## Issue 3: Ansible cannot connect to workers

Check inventory:

```bash
cat /home/ubuntu/2048-role-project/inventory.ini
```

Test SSH manually:

```bash
ssh -i /home/ubuntu/.ssh/ansible.pem ubuntu@<WORKER_PRIVATE_IP>
```

Test Ansible:

```bash
ansible -i inventory.ini workers -m ping
```

---

## Issue 4: Nginx inactive

Start Nginx manually through Ansible:

```bash
ansible -i inventory.ini workers -m service -a "name=nginx state=started enabled=yes" --become
```

Check:

```bash
ansible -i inventory.ini workers -m shell -a "systemctl is-active nginx || true" --become
```

---

## Issue 5: Browser not opening app

Check:

```bash
ansible -i inventory.ini workers -m shell -a "sudo ss -tulpn | grep ':80'" --become
```

Also verify security group allows HTTP port 80.

---

# Learning Outcomes

After completing this project, students will understand:

```text
1. What Ansible collections are
2. Why amazon.aws collection is required
3. How to automate AWS EC2 creation using Ansible
4. How to use default VPC and subnet information
5. How to create security groups using Ansible
6. How to create EC2 master and worker nodes
7. How to generate dynamic inventory content
8. How to install Ansible automatically using user-data
9. How to use ansible-galaxy init for role creation
10. How to deploy an application using Ansible roles
11. How Ansible master connects to workers using private IPs
12. How to troubleshoot cloud-init, SSH, Nginx, and Ansible connectivity
```

---

# Real-Time Explanation for Students

In this project, the controller EC2 is responsible for creating the lab.

The controller uses the `amazon.aws` collection to communicate with AWS.

The automation creates one Ansible master and three worker nodes.

The master receives worker private IPs inside `inventory.ini`.

The master creates a role called `nginx_2048` using `ansible-galaxy init`.

The role installs Nginx and copies the 2048 application file to:

```text
/var/www/html/index.html
```

Nginx serves the application on port 80.

Users access the app using worker public IPs.

---

# Final Command Flow

```bash
# On controller EC2

cd ~/full-aws-ansible-galaxy-2048

chmod 400 /home/ubuntu/ansible.pem

ansible-galaxy collection install -r requirements.yml

ansible-playbook create-full-lab.yml --syntax-check

ansible-playbook create-full-lab.yml
```

After deployment:

```bash
ssh -i /home/ubuntu/ansible.pem ubuntu@<MASTER_PUBLIC_IP>
```

On master:

```bash
sudo cloud-init status --long
cd /home/ubuntu/2048-role-project
ansible -i inventory.ini workers -m ping
ansible-playbook -i inventory.ini site.yml
ansible -i inventory.ini workers -m shell -a "systemctl is-active nginx || true" --become
```

Open application:

```text
http://<WORKER_PUBLIC_IP>
```

---

# Summary

This project demonstrates a full end-to-end real-time DevOps automation flow:

```text
AWS Infrastructure Creation
Ansible Master Setup
Worker Node Provisioning
Ansible Galaxy Collection Usage
Ansible Role Creation
Nginx Installation
2048 Application Deployment
Browser Access
Troubleshooting
Cleanup
```

This is a complete hands-on project for learning AWS, Ansible, Ansible Galaxy, roles, collections, and application deployment.
