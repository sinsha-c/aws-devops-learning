# 🚀 Nginx Server Setup on AWS EC2 (Amazon Linux, Ubuntu & Red Hat)

## 📌 Project Overview
This mini project demonstrates how to set up and configure an **Nginx web server** on different Linux-based EC2 instances using AWS:

- Amazon Linux 2
- Ubuntu
- Red Hat Enterprise Linux (RHEL)

The setup includes both:
- Manual installation steps
- Automated deployment using **EC2 User Data (Bash script)**

---

## 🎯 Objectives
- Launch EC2 instances using different Linux AMIs
- Install and configure Nginx web server
- Verify Nginx default page via browser
- Automate setup using user data scripts

---

## ✅ Expected Outcome
- Nginx is successfully installed and running on all instances
- Accessing `http://<Public-IP>` shows the **default Nginx web page**
- Setup is automated using user data scripts (no manual installation required)

---

## 🧰 Prerequisites

Before starting, ensure the following:

1. AWS account with access to EC2 service
2. Preferred AWS region (e.g., Mumbai)
3. Key Pair created in the selected region
4. Security Group configured with:
   - SSH (Port 22)
   - HTTP (Port 80)
5. Basic knowledge of Linux commands

---

## 🖥️ EC2 Instance Setup

1. Go to AWS Console → EC2 Service
2. Click **Launch Instance**
3. Select the following AMIs:
   - Amazon Linux 2
   - Ubuntu Server
   - Red Hat Enterprise Linux
4. Attach:
   - Key Pair
   - Security Group (SSH + HTTP enabled)
5. Launch the instance

---

# ⚙️ Manual Nginx Installation Steps

## 🟡 1. Amazon Linux 2

### Commands:
```bash
sudo su
dnf update -y
dnf install nginx -y
systemctl start nginx
systemctl status nginx
