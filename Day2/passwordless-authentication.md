# How to setup Passwordless Authentication

## EC2 Instances

### Using Public Key

```
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

- ssh-copy-id: This is the command used to copy your public key to a remote machine.
- -f: This flag forces the copying of keys, which can be useful if you have keys already set up and want to overwrite them.
- "-o IdentityFile <PATH TO PEM FILE>": This option specifies the identity file (private key) to use for the connection. The -o flag passes this option to the underlying ssh command.
- ubuntu@<INSTANCE-IP>: This is the username (ubuntu) and the IP address of the remote server you want to access.

### Using Password 

- Go to the file `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf`
- Update `PasswordAuthentication yes`
- Restart SSH -> `sudo systemctl restart ssh`

# How to Setup Passwordless Authentication

This document explains how to configure SSH access from an Ansible Master EC2 instance to a Worker EC2 instance.

## EC2 Instances

```text
Local Laptop
   |
   | SSH using PEM key
   v
Ansible Master EC2
   |
   | Passwordless SSH
   v
Worker EC2
```

## Placeholders Used

```text
<PEM-FILE>                 = ansible.pem
<EC2-MASTER-PUBLIC-DNS>    = Master EC2 Public DNS
<EC2-WORKER-IP>            = Worker EC2 Public or Private IP
```

Example:

```text
<PEM-FILE>              = ansible.pem
<EC2-MASTER-PUBLIC-DNS> = ec2-100-54-231-64.compute-1.amazonaws.com
<EC2-WORKER-IP>         = 100.55.86.231
```

---

# 1. Login to Master EC2 from Local Laptop

Run this command from your local laptop where the PEM file is available.

```bash
ssh -i "<PEM-FILE>" ubuntu@<EC2-MASTER-PUBLIC-DNS>
```

Example:

```bash
ssh -i "ansible.pem" ubuntu@ec2-100-54-231-64.compute-1.amazonaws.com
```

---

# 2. Copy PEM Key from Local Laptop to Master EC2

Run this command from your local laptop.

```bash
chmod 400 <PEM-FILE>
scp -i <PEM-FILE> <PEM-FILE> ubuntu@<EC2-MASTER-PUBLIC-DNS>:/home/ubuntu/
```

Example:

```bash
chmod 400 ansible.pem
scp -i ansible.pem ansible.pem ubuntu@ec2-100-54-231-64.compute-1.amazonaws.com:/home/ubuntu/
```

---

# 3. Login to Master EC2 Again

```bash
ssh -i "<PEM-FILE>" ubuntu@<EC2-MASTER-PUBLIC-DNS>
```

Example:

```bash
ssh -i "ansible.pem" ubuntu@ec2-100-54-231-64.compute-1.amazonaws.com
```

---

# 4. Verify PEM Key on Master EC2

Run this on the Master EC2.

```bash
ls -l /home/ubuntu/<PEM-FILE>
```

Set correct permission:

```bash
chmod 400 /home/ubuntu/<PEM-FILE>
```

Verify again:

```bash
ls -l /home/ubuntu/<PEM-FILE>
```

Expected permission:

```text
-r-------- 1 ubuntu ubuntu 1674 Jun  3 18:13 /home/ubuntu/ansible.pem
```

---

# 5. Check Whether SSH Key Already Exists on Master EC2

Run this on the Master EC2.

```bash
ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```

If the key does not exist, you may see:

```text
ls: cannot access '/home/ubuntu/.ssh/id_rsa': No such file or directory
ls: cannot access '/home/ubuntu/.ssh/id_rsa.pub': No such file or directory
```

---

# 6. Generate SSH Key on Master EC2

Run this command on the Master EC2.

```bash
ssh-keygen -t rsa -b 4096
```

Press `Enter` for all prompts:

```text
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): Press Enter
Enter passphrase: Press Enter
Enter same passphrase again: Press Enter
```

This will create:

```text
/home/ubuntu/.ssh/id_rsa
/home/ubuntu/.ssh/id_rsa.pub
```

---

# 7. Setup Passwordless Authentication Using Public Key

Run this command from the Master EC2.

```bash
ssh-copy-id -f -i ~/.ssh/id_rsa.pub -o IdentityFile=/home/ubuntu/<PEM-FILE> ubuntu@<EC2-WORKER-IP>
```

Example:

```bash
ssh-copy-id -f -i ~/.ssh/id_rsa.pub -o IdentityFile=/home/ubuntu/ansible.pem ubuntu@100.55.86.231
```

## Explanation

```text
ssh-copy-id
```

This command copies your public key to the remote server.

```text
-f
```

Forces the key copy. Useful when keys already exist and you want to overwrite or re-add them.

```text
-i ~/.ssh/id_rsa.pub
```

Specifies the public key from the Master EC2 that needs to be copied to the Worker EC2.

```text
-o IdentityFile=/home/ubuntu/<PEM-FILE>
```

Specifies the PEM private key used only for initial authentication to the Worker EC2.

```text
ubuntu@<EC2-WORKER-IP>
```

Specifies the remote username and Worker EC2 IP address.

---

# 8. Test SSH Login to Worker Using New Private Key

Run this command from the Master EC2.

```bash
ssh -i ~/.ssh/id_rsa ubuntu@<EC2-WORKER-IP>
```

Example:

```bash
ssh -i ~/.ssh/id_rsa ubuntu@100.55.86.231
```

If login is successful, exit from Worker EC2:

```bash
exit
```

---

# 9. Test Passwordless SSH Login Without PEM Key

Now run this command from the Master EC2.

```bash
ssh ubuntu@<EC2-WORKER-IP>
```

Example:

```bash
ssh ubuntu@100.55.86.231
```

If it logs in without asking for PEM key or password, passwordless SSH authentication is working.

Exit from Worker EC2:

```bash
exit
```

---

# Using Password Authentication

Password authentication is different from passwordless SSH key authentication.

Use this only for lab/testing. For production, SSH key-based authentication is recommended.

## 1. Login to Worker EC2

From Master EC2:

```bash
ssh -i /home/ubuntu/<PEM-FILE> ubuntu@<EC2-WORKER-IP>
```

Example:

```bash
ssh -i /home/ubuntu/ansible.pem ubuntu@100.55.86.231
```

---

## 2. Set Password for Ubuntu User

Run this on Worker EC2:

```bash
sudo passwd ubuntu
```

Enter and confirm the password.

---

## 3. Enable Password Authentication

Open the SSH cloud image config file:

```bash
sudo vi /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
```

Update or add this line:

```text
PasswordAuthentication yes
```

Also verify the main SSH config:

```bash
sudo vi /etc/ssh/sshd_config
```

Make sure these values are present:

```text
PasswordAuthentication yes
PubkeyAuthentication yes
PermitRootLogin no
```

---

## 4. Restart SSH Service

Run this on Worker EC2:

```bash
sudo systemctl restart ssh
```

Check status:

```bash
sudo systemctl status ssh
```

---

## 5. Test Password Login from Master EC2

Exit from Worker EC2 and go back to Master EC2.

Then run:

```bash
ssh ubuntu@<EC2-WORKER-IP>
```

Example:

```bash
ssh ubuntu@100.55.86.231
```

It should ask for password.

---

# Final Command Summary

## From Local Laptop

```bash
cd ~/Downloads

chmod 400 ansible.pem

ssh -i "ansible.pem" ubuntu@<EC2-MASTER-PUBLIC-DNS>

scp -i ansible.pem ansible.pem ubuntu@<EC2-MASTER-PUBLIC-DNS>:/home/ubuntu/
```

---

## From Master EC2

```bash
chmod 400 /home/ubuntu/ansible.pem

ls -l /home/ubuntu/ansible.pem

ls -l ~/.ssh/id_rsa ~/.ssh/id_rsa.pub

ssh-keygen -t rsa -b 4096

ssh-copy-id -f -i ~/.ssh/id_rsa.pub -o IdentityFile=/home/ubuntu/ansible.pem ubuntu@<EC2-WORKER-IP>

ssh -i ~/.ssh/id_rsa ubuntu@<EC2-WORKER-IP>

ssh ubuntu@<EC2-WORKER-IP>
```

---

# Common Errors

## Error: Permission denied while copying PEM

Incorrect command:

```bash
scp -i ansible.pem ansible.pem <EC2-MASTER-PUBLIC-DNS>:/home/ubuntu/
```

Correct command:

```bash
scp -i ansible.pem ansible.pem ubuntu@<EC2-MASTER-PUBLIC-DNS>:/home/ubuntu/
```

Reason: username `ubuntu@` was missing.

---

## Error: ssh-copy-id No identities found

Incorrect command:

```bash
ssh-copy-id -f "-o IdentityFile /home/ubuntu/ansible.pem" ubuntu@<EC2-WORKER-IP>
```

Correct command:

```bash
ssh-copy-id -f -i ~/.ssh/id_rsa.pub -o IdentityFile=/home/ubuntu/ansible.pem ubuntu@<EC2-WORKER-IP>
```

Reason: public key file was not specified with `-i`.

---

## Error: Wrong SSH Test Command

Incorrect command:

```bash
ssh -i -o 'IdentityFile=/home/ubuntu/ansible.pem' 'ubuntu@<EC2-WORKER-IP>'
```

Correct command:

```bash
ssh -i ~/.ssh/id_rsa ubuntu@<EC2-WORKER-IP>
```

Or:

```bash
ssh ubuntu@<EC2-WORKER-IP>
```

---

# Security Note

Copying PEM files to EC2 instances is okay only for lab or practice environments.

For production environments:

```text
Avoid copying PEM keys to servers
Use AWS Systems Manager Session Manager
Use restricted SSH keys
Disable password authentication
Restrict SSH access using Security Groups
Use Ansible Vault for sensitive values
```
