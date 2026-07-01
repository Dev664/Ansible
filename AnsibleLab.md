# Ansible Lab – Complete Hands-on Guide

## Lab Objective

In this lab, you will learn how to:

* Install Ansible on the Control Node
* Configure passwordless SSH authentication
* Create an Ansible inventory
* Execute ad-hoc Ansible commands
* Configure `ansible.cfg`
* Create and run your first Ansible Playbook

---

# Lab Architecture

```
                AWS VPC
+--------------------------------------------------+

        Ansible Control Node
            (EC2 Instance)
           172.31.10.10
                 |
                 | SSH (Port 22)
                 |
        -------------------------
        |                       |
        |                       |
  Target Node 1           Target Node 2
 172.31.10.11            172.31.10.12

+--------------------------------------------------+
```

---

# Prerequisites

Launch the following EC2 instances:

| Server           | Purpose                | OS                           |
| ---------------- | ---------------------- | ---------------------------- |
| EC2-1            | Ansible Control Node   | Amazon Linux / RHEL / Ubuntu |
| EC2-2            | Target Node            | Amazon Linux / RHEL / Ubuntu |
| EC2-3 (Optional) | Additional Target Node | Amazon Linux / RHEL / Ubuntu |

---

## Network Requirements

Ensure all EC2 instances have:

* Same VPC
* Same Subnet (recommended for lab)
* Same Security Group (recommended)

### Security Group Rules

| Protocol | Port | Source                      | Purpose         |
| -------- | ---- | --------------------------- | --------------- |
| TCP      | 22   | Control Node Security Group | SSH Access      |
| ICMP     | All  | Same Security Group         | Ping (Optional) |

---

# Step 1: Install Ansible

## Login to the Control Node

### Amazon Linux / RHEL

```bash
sudo yum update -y
sudo yum install ansible -y
```
---

## Verify Installation

```bash
ansible --version
```

Example Output:

```text
ansible [core 2.x.x]

python version

config file
```

---

# Step 2: Configure Passwordless SSH Authentication

Passwordless SSH allows Ansible to connect to remote servers without entering a password each time.

## Generate SSH Key (Control Node)

```bash
ssh-keygen -t rsa -b 2048
```

Press **Enter** for all prompts.

---

## Display Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the complete output.

---

## Configure the Target Node

On each target server:

```bash
mkdir -p ~/.ssh
vi ~/.ssh/authorized_keys
```

Paste the copied public key.

Set permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Restart SSH service:

### RHEL / Amazon Linux

```bash
sudo systemctl restart sshd
```

### Ubuntu

```bash
sudo systemctl restart ssh
```

---

## Test Passwordless SSH

```bash
ssh ec2-user@172.31.10.11
```

The login should complete without asking for a password.

Repeat for all target nodes.

---

# Step 3: Create an Ansible Project

```bash
mkdir ansible-project
cd ansible-project
```

---

# Step 4: Create the Inventory File

Create the inventory file:

```bash
vi hosts
```

Example:

```ini
[web]
172.31.10.11
172.31.10.12

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/my-key.pem
```

> **Note:** If you're using passwordless SSH with `id_rsa`, you can omit `ansible_ssh_private_key_file`.

---

# Step 5: Test Connectivity

Ping all managed nodes:

```bash
ansible -i hosts all -m ping
```

Expected Output:

```text
172.31.10.11 | SUCCESS

172.31.10.12 | SUCCESS
```

---

# Step 6: Run Your First Ad-hoc Command

Display system uptime:

```bash
ansible -i hosts all -m shell -a "uptime"
```

Check hostname:

```bash
ansible -i hosts all -m command -a "hostname"
```

Check disk usage:

```bash
ansible -i hosts all -m shell -a "df -h"
```

Check memory usage:

```bash
ansible -i hosts all -m shell -a "free -h"
```

---

# Step 7: Configure ansible.cfg

Instead of specifying the inventory every time, create a local Ansible configuration file.

Create:

```bash
vi ansible.cfg
```

Add:

```ini
[defaults]
inventory = hosts
remote_user = ec2-user
host_key_checking = False
private_key_file = ~/.ssh/my-key.pem
```

---

## Configuration File Priority

Ansible searches for configuration files in the following order:

1. `./ansible.cfg` (Project Directory)
2. `~/.ansible.cfg` (User Directory)
3. `/etc/ansible/ansible.cfg` (Global Configuration)

---

## Test Again

Now you no longer need to specify the inventory file:

```bash
ansible all -m ping
```

---

# Recommended Project Structure

```text
ansible-project/
│
├── ansible.cfg
├── hosts
├── playbooks/
├── roles/
└── group_vars/
```

---

# Step 8: Create Your First Playbook

Create the playbook:

```bash
vi install-nginx.yml
```

Add the following content:

```yaml
---
- name: Install Nginx on Web Servers
  hosts: web
  become: yes

  tasks:

    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Enable Nginx
      service:
        name: nginx
        enabled: yes

    - name: Start Nginx
      service:
        name: nginx
        state: started
```

---

# Step 9: Run the Playbook

```bash
ansible-playbook install-nginx.yml
```

Or, if not using `ansible.cfg`:

```bash
ansible-playbook -i hosts install-nginx.yml
```

---

# Step 10: Verify Installation

Verify Nginx is running:

```bash
ansible web -m shell -a "systemctl status nginx"
```

Verify the package:

```bash
ansible web -m shell -a "rpm -qa | grep nginx"
```



---

# Common Ansible Ad-hoc Commands

| Task            | Command                                                |
| --------------- | ------------------------------------------------------ |
| Ping Hosts      | `ansible all -m ping`                                  |
| Hostname        | `ansible all -m command -a "hostname"`                 |
| Uptime          | `ansible all -m shell -a "uptime"`                     |
| Disk Usage      | `ansible all -m shell -a "df -h"`                      |
| Memory Usage    | `ansible all -m shell -a "free -h"`                    |
| Install Package | `ansible all -m yum -a "name=httpd state=present"`     |
| Start Service   | `ansible all -m service -a "name=nginx state=started"` |
| Reboot Servers  | `ansible all -m reboot`                                |

---

# Lab Summary

By completing this lab, you have learned how to:

* Deploy Ansible on a Control Node
* Configure passwordless SSH authentication
* Create an inventory file
* Execute ad-hoc Ansible commands
* Configure `ansible.cfg`
* Organize an Ansible project
* Write your first playbook
* Install and manage software across multiple servers using Ansible

---

**End of Lab**

