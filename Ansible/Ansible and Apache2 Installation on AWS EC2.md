# Ansible and Apache2 Installation on AWS EC2

This guide walks you through using **Terraform** to create an EC2 instance on AWS and **Ansible** to install **Apache2** on the EC2 Ubuntu instance.

## Prerequisites

1. **Terraform** installed on your local machine.
2. **Ansible** installed on your local machine.
3. **AWS credentials** set up in your environment (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`).
4. **A key pair** for SSH access to the EC2 instance.

## Steps

### 1. **Set up Terraform to Create EC2 Instance on AWS**

First, make sure you have the following files:

#### `main.tf` (Terraform Configuration)

This file will:

- Set up a security group for SSH (port 22), HTTP (port 80), and Jenkins (port 8080).
- Create a key pair for SSH access.
- Launch an EC2 instance (Ubuntu 22.04 LTS) with `user_data` to install Jenkins (optional step for this guide).

Here's the content for `main.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-2"
}

# 🔐 Security group to allow SSH and HTTP (Port 80)
resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow SSH and HTTP access"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # HTTP access (port 80)
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow_ssh"
  }
}

# 🔑 Key pair
resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("${path.module}/my-ec2-key.pub")
}

# 💻 EC2 instance
resource "aws_instance" "app_server" {
  ami                    = "ami-04f167a56786e4b09" # Ubuntu 22.04 LTS
  instance_type          = "t2.micro"
  key_name               = aws_key_pair.my_key.key_name
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  tags = {
    Name = "ApacheServer"
  }
}

# 🌐 Output the instance public IP
output "public_ip" {
  value = aws_instance.app_server.public_ip
}
```

### 2. **Apply Terraform to Launch EC2 Instance**

1. Run the following commands to initialize Terraform and create the EC2 instance:
   ```bash
   terraform init
   terraform apply
   ```

2. Terraform will output the **public IP** of the EC2 instance. Make note of it, as you will use it in the next steps.

### 3. **Set Up Ansible to Install Apache2**

Create an **inventory file** (`inventory.ini`) to define the target EC2 instance for Ansible.

#### `inventory.ini`

```ini
[web]
ec2 ansible_host=<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=./my-ec2-key
```

Replace `<EC2_PUBLIC_IP>` with the public IP output by Terraform.

### 4. **Create the Ansible Playbook to Install Apache2**

Create a playbook file (`apache-install.yml`) that installs Apache2 on the EC2 instance:

#### `apache-install.yml`

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

This playbook does the following:

- Updates the APT cache.
- Installs Apache2.
- Starts and enables Apache2 to run at boot.

### 5. **Run the Ansible Playbook**

Run the following command to execute the Ansible playbook and install Apache2 on the EC2 instance:

```bash
ansible-playbook -i inventory.ini apache-install.yml
```

Ansible will connect to the EC2 instance and install Apache2.

### 6. **Access Apache2 Web Server**

Once the playbook finishes, you can access the Apache2 web server by visiting the **public IP** of your EC2 instance in a web browser:

```
http://<EC2_PUBLIC_IP>
```

You should see the **Apache2 Ubuntu Default Page**.

## Conclusion

You have successfully:

- Created an EC2 instance on AWS using Terraform.
- Used Ansible to install Apache2 on the EC2 instance.
- Opened the required port (80) in the security group to allow HTTP traffic.

---

### Notes:
- This guide assumes you are using **Ubuntu** on the EC2 instance.
- Make sure to replace `<EC2_PUBLIC_IP>` in the inventory file with the actual public IP address output by Terraform.
- This example installs Apache2, but you can modify the playbook to install other software if needed.
```

---

### Explanation:

- **Terraform setup**: Describes how to create the EC2 instance and the necessary security group to allow HTTP access (port 80).
- **Ansible setup**: Walks you through creating an inventory file and an Ansible playbook to install Apache2 on the EC2 instance.
- **Testing**: Describes how to verify the Apache installation by visiting the EC2 instance's public IP.
