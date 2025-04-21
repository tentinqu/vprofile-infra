# VProfile AWS Deployment Guide

## Architecture Overview

![Architecture Diagram](INSERT-YOUR-DIAGRAM-HERE)

## Prerequisites
- **Java JDK**: 17 or 21
- **Maven**: 3.9
- **MySQL**: 8+

## Tech Stack
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- Tomcat
- MySQL
- Memcached
- RabbitMQ

---

## Database Setup

Import the SQL dump to your MySQL DB:
```bash
mysql -u <user_name> -p accounts < /src/main/resources/db_backup.sql
```

---

## 1. Setting Up Security Groups

### ELB Security Group
- Allow HTTP/HTTPS from anywhere (IPv4 + IPv6).

### App Server SG
- Allow traffic from ELB SG.
- Allow SSH from your IP.

### Backend Services SG (MySQL, Memcached, RabbitMQ)
- Port 3306 (MySQL) from App SG
- Port 11211 (Memcached) from App SG
- Port 5672 (RabbitMQ) from App SG
- Allow traffic from **itself** to enable internal service communication.
- SSH from your IP

---

## 2. Creating Key Pair
- Format: `.pem` (for terminal/Git Bash), `.ppk` if using PuTTY.

---

## 3. Launching EC2 Instances

Create 4 instances:
- memcached
- rabbitmq
- tomcat
- mysql

All should be attached to the **Backend SG**. SSH and check services using `systemctl`.

> Use DNS over IPs to avoid config issues if instances are replaced.

---

## 4. Setting Up DNS (Route 53)
- Create hosted zone: `vprofile.in`
- Record type: A
- Example record: `db01.vprofile.in → private IP of MySQL instance`

### Check DNS resolution:
```bash
ping -c 4 db01.vprofile.in
```
(Allow ICMP in SG to test ping)

---

## 5. Building & Deploying Artifact

### Maven Build
```bash
mvn install
```
Artifact path: `target/vprofile-v2.war`

### AWS CLI Setup
```bash
aws configure
```
- Stores credentials in `~/.aws/credentials`
- Stores config in `~/.aws/config`

### S3 Upload
```bash
aws s3 cp target/vprofile-v2.war s3://<your-bucket-name>/
```

### On App Server
```bash
snap install aws-cli --classic
aws s3 cp s3://<your-bucket-name>/vprofile-v2.war /tmp/
sudo cp /tmp/vprofile-v2.war /var/lib/tomcat10/webapps/
```

---

## 6. Load Balancer Setup

### Target Group
- Protocol: HTTP
- Port: 8080 (Tomcat default)
- Health check: override port to 8080

### Load Balancer
- Use ELB SG
- Attach listener for **HTTP (80)** and **HTTPS (443)**
- Use **ACM Certificate** for HTTPS

### Domain Integration
- Go to GoDaddy or registrar
- Create a **CNAME record**
  - Name: `vprofileapp`
  - Value: ELB DNS
- Now accessible via: `https://vprofileapp.harshitch.xyz`

---

## 7. Auto Scaling Group (ASG)

### Requirements:
- AMI from existing app01
- Launch Template with AMI, Key Pair, App SG
- ASG with:
  - Min capacity
  - Max capacity
  - Desired capacity
  - All AZs selected
  - Health checks enabled to replace failed instances

### Tip:
- To distribute load across multiple instances:
  - Go to Target Groups > Attributes > Edit > Enable sticky sessions or round-robin settings as needed

---

## Summary Flow
```
Internet → ELB → App Server (Tomcat) → Backend Services (MySQL, RabbitMQ, Memcached)
```

All communication secured via specific SG rules and DNS records for IP independence.

## PSA 
This repository focuses on the infrastructure of the vprofile project. For the original code, features, and full details of the website, please check out the official repository here: [vprofile-project](https://github.com/hkhcoder/vprofile-project/tree).

To-do: 
upload screenshots

