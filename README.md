# vProfile Lift-and-Shift (AWS)

## 📌 Project Overview

vProfile is a **multi-tier web application** migrated from a local/on-premise environment to AWS using a **Lift-and-Shift** migration strategy. The goal of this project was to deploy the existing application stack to AWS without changing its architecture, while leveraging AWS services for scalability, security, and global accessibility.

The application stack includes:

- **Spring MVC** & **Spring Security**
- **MySQL** (hosted on EC2)
- **RabbitMQ** (messaging broker)
- **Memcached** (caching)
- **Apache Tomcat** (application server)

For **global hosting**, the project uses a **custom domain** (`projectops.xyz`) registered via GoDaddy, configured with **AWS ACM** for HTTPS and integrated via **Route 53**.

---

## 🏗 Architecture Diagram

![AWS Lift and Shift Architecture](Vprofile%20Lift%20and%20Shift%20project.png)

---

## 🚀 AWS Services Used

- **Amazon EC2** – Hosts Tomcat, MySQL, RabbitMQ, and Memcached
- **Amazon S3** – Stores application artifacts (WAR files)
- **Elastic Load Balancer (ALB)** – Distributes traffic to app instances
- **AWS Auto Scaling** – Automatically scales app instances based on load
- **Amazon Route 53** – Provides DNS resolution (private for internal services, public for domain)
- **AWS Certificate Manager (ACM)** – Issues SSL/TLS certificate for HTTPS
- **Amazon EBS** – Block storage for EC2 instances
- **IAM** – Access control for AWS resources

---

## ⚙️ Deployment Flow

### 1️⃣ **Preparation**

- Registered a domain on GoDaddy (`projectops.xyz`)
- Requested an **ACM public certificate** for `vprofile.projectops.xyz`
- Created **key pairs** for SSH access
- Created **three Security Groups**:
  - **ELB SG** – Allows HTTP/HTTPS from the internet
  - **App SG** – Allows port 8080 from ELB SG, SSH from admin IP
  - **Backend SG** – Allows MySQL (3306), Memcached (11211), RabbitMQ (5672) from App SG, SSH from admin IP

---

### 2️⃣ **Backend Setup**

- Launched EC2 instances for:
  - **MySQL**
  - **RabbitMQ**
  - **Memcached**
- Configured **Route 53 private hosted zone** for backend DNS names (`db01`, `mc01`, `rmq01`)

---

### 3️⃣ **Application Setup**

- Launched **Tomcat EC2 instance** with user-data script to install & configure app
- Built **WAR artifact** locally with Maven:
  ```bash
  mvn clean install
  ```
- Uploaded WAR to **S3 bucket**
- Downloaded WAR from S3 to Tomcat EC2 instance and deployed

---

## 4️⃣ Load Balancer & HTTPS

- Created **Target Group** for Tomcat instances on port **8080**
- Created **Application Load Balancer**:
  - Listener on port **80** (HTTP) → Redirect to HTTPS
  - Listener on port **443** (HTTPS) with ACM certificate
- Updated **GoDaddy DNS** to create **CNAME** for `vprofile.projectops.xyz` pointing to ALB DNS name
- Verified secure access:

```arduino
https://vprofile.projectops.xyz
```

## 5️⃣ Auto Scaling

- Created **AMI** from the Tomcat EC2 instance
- Created **Launch Template** using AMI, key pair, App SG
- Created **Auto Scaling Group**:
  - Min: 1 instance, Max: 4 instances
  - Scaling policy: CPU > 50% → Scale out, CPU < 50% → Scale in
  - Attached to ALB Target Group
  - Enabled **stickiness** for session persistence

---

## ✅ Validation

- Verified:
  - Login works (`admin_vp` / `admin_vp`)
  - RabbitMQ queues generated
  - Memcached caching (first request from DB, subsequent from cache)
- Checked HTTPS certificate validity from ACM

## 📚 Key Learnings

- Migrating applications to AWS using a **Lift-and-Shift** approach
- Setting up **multi-tier architecture** with AWS services
- Configuring **HTTPS** using ACM & integrating with custom domain
- Implementing **Auto Scaling** with load-based scaling policies
- Using **Route 53** for both private and public DNS

---

## 🛠 Tech Stack

- **Java JDK 11**
- **Maven 3**
- **Spring MVC & Spring Security**
- **MySQL 8.0**
- **RabbitMQ**
- **Memcached**
- **Apache Tomcat**
- **AWS EC2, S3, ALB, Auto Scaling, Route 53, ACM**

## 🔧 DevOps Implementation & AWS Deployment By

**Maaz Khan** – DevOps Engineer | Cloud & Automation Enthusiast
