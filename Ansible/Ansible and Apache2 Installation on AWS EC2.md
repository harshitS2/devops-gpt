# 🚀 Ansible + Apache2 on AWS EC2 via Terraform

This project provisions an AWS EC2 instance using **Terraform** and configures it with **Apache2** using **Ansible**. It's designed for easy reproducibility and automation for dev/test environments.

---

## 📦 What This Project Does

- Provisions an Ubuntu EC2 instance on AWS using Terraform.
- Configures security groups to allow SSH (port 22) and HTTP (port 80).
- Installs Apache2 web server on the instance using Ansible.
- Verifies setup via browser access to Apache2 default page.

---

## 📋 Prerequisites

Before you begin, ensure the following are installed on your machine:

- [Terraform](https://www.terraform.io/downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and `aws configure` has been run
- SSH key pair for EC2 access (see below for how to generate one)

---

## 🔑 Generate EC2 SSH Key Pair

Generate a key pair named `my-ec2-key`:

```bash
ssh-keygen -t rsa -b 4096 -f my-ec2-key
```

This will produce two files:
- `my-ec2-key` — private key (keep secure!)
- `my-ec2-key.pub` — public key (used by Terraform)

Move the key files into your Terraform working directory:

```bash
mv my-ec2-key* /path/to/terraform/
chmod 400 my-ec2-key  # Ensure proper permissions
```

---

## 🏗️ Infrastructure Setup with Terraform

### 📁 [`main.tf`](Terraform/Ubuntu%20EC2%20Instance%20create/main.tf)

This Terraform config:
- Creates a security group
- Adds SSH and HTTP ingress rules
- Provisions an EC2 instance using Ubuntu 22.04 LTS
- Outputs the public IP of the instance

> Make sure your `my-ec2-key.pub` is in the same directory as `main.tf`.

### ✅ Initialize & Apply

```bash
terraform init
terraform apply
```

When prompted, confirm with `yes`.

### 🌐 Output

The output will include the public IP of the instance, which you'll use in the next steps.

---

## 📁 Ansible Inventory Configuration

Create a file named `inventory.ini` in your project directory:

```ini
[web]
ec2 ansible_host=<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=./my-ec2-key
```

Replace `<EC2_PUBLIC_IP>` with the output from Terraform.

---

## 📜 Ansible Playbook to Install Apache2

Create a file named `apache-install.yml`:

```yaml
---
- name: Install Apache on EC2 Ubuntu
  hosts: web
  become: yes

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install Apache2
      apt:
        name: apache2
        state: present

    - name: Start and enable Apache2 service
      service:
        name: apache2
        state: started
        enabled: yes
```

---

## 🚀 Run the Ansible Playbook

Execute the playbook with:

```bash
ansible-playbook -i inventory.ini apache-install.yml
```

---

## 🔍 Verify Apache2 Installation

Open your browser and visit:

```
http://<EC2_PUBLIC_IP>
```

You should see the **Apache2 Ubuntu Default Page**.

---

## 🧠 Summary

| Step | Description |
|------|-------------|
| 1. | Generate SSH key for EC2 |
| 2. | Write Terraform configuration |
| 3. | Launch EC2 using `terraform apply` |
| 4. | Use Ansible to install Apache2 |
| 5. | Access Apache2 via public IP in browser |

---

## 🔐 Security Best Practices

- 🔒 **NEVER** expose SSH to `0.0.0.0/0` in production; use your own IP.
- 📁 Use `chmod 400` on private keys.
- 🔁 Rotate keys and update your `aws_key_pair` resources as needed.
- 🔐 Add `.gitignore` for `.pem` or key files if pushing to GitHub.

---

## 📂 File Structure

```
terraform-apache-ansible/
├── apache-install.yml
├── inventory.ini
├── main.tf
├── my-ec2-key
├── my-ec2-key.pub
└── README.md
```

---

## 📌 Notes

- Default user for Ubuntu AMI is `ubuntu`
- You can customize the playbook to install more software (e.g., Jenkins, Nginx, Docker)
- If Ansible fails to connect, ensure port 22 is open and `my-ec2-key` has `chmod 400` permission

---

Happy Automating! 🎯
