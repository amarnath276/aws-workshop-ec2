# 🚀 Project: Deploy a Custom Apache Web Server on Amazon EC2

---

## 📘 Description

This project demonstrates how to launch an **Amazon EC2 instance** running **Amazon Linux 2023** and configure it as an **Apache web server**. The setup is fully automated using **User Data**, which installs Apache and deploys a custom, styled webpage on startup.

---

## ✅ Objectives

- Launch an EC2 instance using Amazon Linux 2023  
- Install and configure Apache (httpd) using a User Data script  
- Deploy a custom `index.html` web page  
- Access the hosted site via the instance's public IP  

---

## 🛠️ Steps to Complete the Project

### 🔸 Step 1: Launch an Amazon EC2 Instance

1. Sign in to your **AWS Management Console**
2. Go to **EC2 Dashboard**
3. Click **Launch Instance**
4. Configure the following:
   - **Name:** `Custom-Apache-Web-Server`
   - **AMI:** Amazon Linux 2023
   - **Instance Type:** `t2.micro` (Free Tier eligible)
   - **Key Pair:** Proceed without a key pair
   - **Network Settings:**
     - Enable **Auto-assign Public IP**
     - **Create a new security group** with:
       - ✅ **Allow HTTP (80)** from Anywhere `0.0.0.0/0`, `::/0`
       - ✅ **Allow SSH (22)** from My IP
   - **User Data:** Paste the script shown below

5. Click **Launch Instance**

---

### 🔸 Step 2: Configure User Data Script

Paste this script into the **User Data** section during instance configuration:

```bash
#!/bin/bash
# Update and install Apache
dnf update -y
dnf install -y httpd

# Start and enable Apache
systemctl start httpd
systemctl enable httpd

# Set permissions
chown -R ec2-user:ec2-user /var/www/html

# Create custom web page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Welcome to My Custom Apache Server</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      text-align: center;
      padding: 50px;
    }
    .container {
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
      display: inline-block;
    }
    h1 { color: #333; }
    p { color: #666; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Welcome to My Custom Apache Web Server!</h1>
    <p>Hosted on an Amazon Linux 2023 EC2 Instance.</p>
    <p>This page was deployed automatically using AWS User Data.</p>
  </div>
</body>
</html>
EOF

# Restart Apache
systemctl restart httpd
