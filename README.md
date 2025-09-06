# HTML Website Hosting on AWS EC2

## 📌 Project Overview
This project demonstrates how to host a static HTML website on an **AWS EC2 instance** within the default VPC.  
The deployment was automated using **User Data scripts**, showcasing two approaches:  
1. Deploying from an **S3 bucket**  
2. Deploying from a **GitHub repository**  

---

## 🔧 Resources Utilized

### VPC Setup
- Used the **default VPC** for simplicity.  
- Created **two subnets** (public and private) within one availability zone.  
- Configured a **custom route table** for the private subnet to restrict direct internet access.  

### Security
- Implemented **security groups** for network security.  
- Deployed an **EC2 Instance Connect Endpoint** for secure communication with the private instance.  

### Infrastructure Components
- Public subnet hosted:
  - **NAT Gateway**
  - **Application Load Balancer (ALB)**  
- EC2 instance deployed in the **private subnet**.  

### File Hosting & Access Control
- Stored website files in an **S3 bucket** with restricted public access.  
- Configured an **IAM role** with S3 access and attached it to the EC2 instance.  
- Website files also maintained in **GitHub** for version control and alternative deployment.  

### Application Access
- Public access to the website was provided via an **ALB + Target Group** pointing to the EC2 instance.  

---

## 🚀 Deployment Scripts

Two deployment scripts were created and tested with EC2 **User Data**:  

### 1️⃣ S3 Deployment Script
- Downloads a ZIP file from S3.  
- Extracts contents into `/var/www/html`.  
- Automatically starts Apache.  

### 2️⃣ GitHub Deployment Script
- Clones a GitHub repository into `/var/www/html`.  
- Cleans up unnecessary files (e.g., `.git`).  
- Automatically starts Apache.  

---

## ⚠️ Challenges & Fixes

### Problem: S3 `.zip` File Extracting into a Subfolder
When the website files were stored in S3 as a **zipped project folder** (e.g., `mole/index.html` instead of `index.html`), unzipping on the EC2 instance resulted in an extra subfolder (`/var/www/html/mole`) instead of placing files directly in `/var/www/html`.  
This caused Apache to still show the default **“It works!”** page because the `index.html` was not located at the web root.  

### Fix:
A **post-extraction move command** was added to handle both scenarios:  

```bash
# Move files correctly, even if they are inside a folder
mv -f /tmp/site/*/* /tmp/site/* /var/www/html 2>/dev/null || mv -f /tmp/site/* /var/www/html
````

* If files are **nested inside a folder**, they are moved one level up into `/var/www/html`.
* If files are already at the root of the ZIP, they are copied directly.
* This makes the script **idempotent and flexible**, working with both zipped folders and zipped flat files.

---

## Deployment Scripts
Two deployment scripts were created to automate the hosting of the website during the EC2 instance launch using User Data.

# Task 1: S3 Deployment Script

```bash
#!/bin/bash
# ==============================
# Variables
# ==============================
S3_BUCKET=pros101
S3_REGION=us-east-1
S3_OBJECT=mole.zip
WEB_DIR=/var/www/html

# ==============================
# Install Apache, AWS CLI, unzip
# ==============================
yum update -y
yum install -y httpd awscli unzip

# ==============================
# Deploy Website
# ==============================

# Clear existing files
rm -rf $WEB_DIR/*

# Download zip from S3
aws s3 cp s3://$S3_BUCKET/$S3_OBJECT /tmp/$S3_OBJECT --region $S3_REGION

# Extract to temporary directory
mkdir -p /tmp/site
unzip -o /tmp/$S3_OBJECT -d /tmp/site

# Move extracted files to Apache directory (handles nested folders too)
mv -f /tmp/site/*/* /tmp/site/* $WEB_DIR 2>/dev/null || mv -f /tmp/site/* $WEB_DIR

# Set permissions
chown -R apache:apache $WEB_DIR
chmod -R 755 $WEB_DIR

# Restart and enable Apache
systemctl restart httpd
systemctl enable httpd

#Task 2: Github Deployment Script

#!/bin/bash

# Switch to the root user to gain full administrative privileges
sudo su

# Update all installed packages to their latest versions
yum update -y

# Install Apache HTTP Server
yum install -y httpd

# Change the current working directory to the Apache web root
cd /var/www/html

# Install Git
yum install git -y

# Clone the project GitHub repository to the current directory
git clone https://github.com/CaptainNii/host-an-html-website-on-aws.git

# Copy all files, including hidden ones, from the cloned repository to the Apache web root
cp -R host-an-html-website-on-aws/. /var/www/html/

# Remove the cloned repository directory to clean up unnecessary files
rm -rf host-an-html-website-on-aws

# Enable the Apache HTTP Server to start automatically at system boot
systemctl enable httpd 

# Start the Apache HTTP Server to serve web content
systemctl start httpd

## ✅ Acceptance Criteria

* Two independent scripts were created for **S3** and **GitHub** deployments.
* Both scripts can be added to **User Data** when launching an EC2 instance.
* The website is hosted automatically upon launch.

---

## 📖 Conclusion

This project successfully demonstrates how to host a static HTML website on an **AWS EC2 instance** using two different deployment methods (**S3** and **GitHub**).

Security best practices were implemented using **IAM roles, security groups, private subnets, and an ALB** to ensure a secure and reliable deployment.

---

## 📂 Repository Structure

```
.
├── s3-deploy.sh      # Script to deploy site from S3
├── github-deploy.sh  # Script to deploy site from GitHub
├── index.html        # Example website file
└── README.md         # Project documentation
```

---

## 👨‍💻 Author

* **CaptainNii** – Cloud & DevOps Enthusiast
