# Highly-Available-Two-Tier-Architecture-on-AWS
A production-style deployment of the EpicReads book platform using a two-tier architecture on AWS. This project demonstrates high availability, fault tolerance, and scalability by distributing infrastructure across multiple Availability Zones and implementing load balancing, auto scaling, and a resilient database layer.

## 1. Project Overview

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/Architecture%20Diagram.png" width="800"><br><br>
  <em>Architecture Diagram</em>
</p>

EpicReads is deployed using a classic two-tier architecture:

### Web Tier (Presentation + Application Layer)

- Runs on EC2 instances with Apache and application code
- Handles all incoming user requests

### Database Tier (Data Layer)

- Uses MySQL on Amazon RDS
- Stores all persistent application data
- Fully isolated within private subnets

Initial setup started as a single EC2 instance and database in one Availability Zone. This was later upgraded into a highly available architecture spanning multiple Availability Zones with redundancy at every layer.

---

## 2. Live Access

- **Application URL:** `http://<your-load-balancer-dns>`

### Testing Tips

- Refresh multiple times to observe load balancing
- Stop one EC2 instance and verify application availability
- Observe automatic traffic redirection by the load balancer

---

## 3. Two-Tier Architecture Explained

In this model:

- The web server handles both presentation and application logic
- The database runs on a separate, isolated server

### Key Benefits

- Independent scaling of web and database layers
- Improved security by isolating the database
- Easier maintenance and updates
---

## 4. Architecture Evolution

### Initial Setup (Non-HA)

- Single EC2 instance
- Single RDS database
- One Availability Zone
- No redundancy or failover support

### Final High-Availability Setup

- Multiple EC2 instances across two Availability Zones
- Application Load Balancer distributing traffic
- Auto Scaling Group maintaining healthy instances
- Multi-AZ RDS deployment with failover capability
- NAT Gateways for secure outbound access from private subnets

**Result:** No single point of failure in the system.

---
## 5. High Availability Design

### Web Tier

- Auto Scaling Group deployed across two Availability Zones
- Minimum 1 instance, maximum 4 instances
- Health checks ensure only healthy instances receive traffic

### Load Balancer

- Internet-facing Application Load Balancer
- Distributes traffic across multiple AZs
- Automatically routes around failed instances

### Database Tier

- Amazon RDS MySQL with Multi-AZ deployment
- Primary database in one AZ
- Standby/replica in another AZ
- Automatic failover support

### Networking

- Public and private subnets in each Availability Zone
- NAT Gateway per AZ for high availability
---
## 6. AWS Services Used

- Amazon EC2 – Web servers
- Auto Scaling – Maintains instance availability
- Application Load Balancer – Traffic distribution
- Amazon RDS (MySQL) – Managed relational database
- VPC – Isolated network
- Subnets – Network segmentation
- Internet Gateway – Public internet access
- NAT Gateway – Private subnet internet access
- S3 – Storage for logs and assets
- AWS Identity and Access Management (IAM) – Access control
- AWS Budgets – Cost monitoring
---
## 7. Network Architecture

- **Region:** `ap-south-1` (Mumbai)
- **VPC CIDR:** `10.0.0.0/16`

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-3.png" width="800"><br><br>
  <em>Network Architecture</em>
</p>

### Subnets

- **Public Subnet 1 (AZ-a):** `10.0.0.0/24` – Web Tier
- **Public Subnet 2 (AZ-b):** `10.0.1.0/24` – Web Tier
- **Private Subnet 1 (AZ-a):** `10.0.2.0/24` – Database Tier
- **Private Subnet 2 (AZ-b):** `10.0.3.0/24` – Database Tier

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-4.png" width="800"><br><br>
  <em>Subnets</em>
</p>

### Routing

#### Public Route Table

- `0.0.0.0/0 → Internet Gateway`

#### Private Route Table

- `0.0.0.0/0 → NAT Gateway`

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-5.png" width="800"><br><br>
  <em>Routing</em>
</p>

#### Gateways
- Internet Gateway IGW-EpicReads – attached to VPC, serves public subnets.
- NAT Gateways – one in each public subnet, enabling private subnets to pull updates without exposing the database to the internet.

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-6.png" width="800"><br><br>
  <em>Gateways</em>
</p>

---
## 8. Security Configuration

Security Groups enforce strict tier separation:

### Load Balancer Security Group

- Allows HTTP (port 80) from the internet

### Web Server Security Group

- Allows HTTP only from Load Balancer
- Allows SSH only from trusted IP

### Database Security Group

- Allows MySQL (port 3306) only from Web Servers

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-7.png" width="800"><br><br>
  <em>Security Configuration</em>
</p>

### Key Security Practices

- No public access to the database
- Principle of least privilege using IAM
- MFA enabled for root and IAM users
---
## 9. Compute and Auto Scaling

### Launch Template

- Preconfigured AMI with Apache, PHP, and application code

### Instance Type

- `t3.micro`

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-9.png" width="800"><br><br>
  <em>Launch Template</em>
</p>

### Auto Scaling Group

- Minimum: 1
- Maximum: 4
- Distributed across two Availability Zones

### Health Checks

- Integrated with Load Balancer

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-13.png" width="800"><br><br>
  <em>Auto Scaling Group</em>
</p>

---
## 10. Load Balancing

### Application Load Balancer

- Internet-facing
- Listens on HTTP (port 80)

### Target Group

- Routes traffic to EC2 instances
- Health check path: `/`
- Only healthy instances receive traffic

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-11.png" width="800"><br><br>
  <em>Load Balancing</em>
</p>

---
## 11. Database Layer

- **Engine:** MySQL (Amazon RDS)
- **Instance Type:** `db.t3.micro`

### Configuration

- Multi-AZ deployment
- Automatic failover enabled

### Security

- No public access
- Only accessible from web tier

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Highly-Available-Two-Tier-Architecture-on-AWS/blob/main/images/P1-14.png" width="800"><br><br>
  <em>Database Layer</em>
</p>

---
## 12. High Availability Testing (Important)

To validate the architecture:

- Stopped one EC2 instance → traffic continued via other instance
- Simulated failure scenario → load balancer rerouted traffic
- Verified system remained accessible during instance failure

This confirms fault tolerance at the web tier.

---
## 13. Monitoring and Cost Control

### Monitoring

- Basic monitoring via AWS console
- Health checks through Load Balancer

### Cost Control

- AWS Budgets configured for alerts

### Approximate Monthly Cost

- EC2 (2 instances): low cost (`t3.micro`, free tier eligible if applicable)
- RDS Multi-AZ: moderate cost
- NAT Gateway: highest cost component
- Load Balancer: moderate cost

**Estimated total:** medium cost range for a learning project.

---
## 14. Key Learnings

- Designed a highly available architecture with no single point of failure
- Implemented Auto Scaling for dynamic traffic handling
- Configured secure communication between application tiers
- Built a fault-tolerant system using Multi-AZ deployment
- Understood real-world cloud architecture patterns
  
---
## 15. Future Enhancements

- Add HTTPS using SSL (ACM or Let's Encrypt)
- Integrate CI/CD pipeline (e.g., Jenkins or GitHub Actions)
- Containerize application using Docker
- Deploy using Kubernetes (EKS)
- Add advanced monitoring (CloudWatch, Prometheus, Grafana)
- Use Infrastructure as Code tools for automation in future

## 16. Conclusion

This project demonstrates a practical implementation of a highly available two-tier architecture on AWS. By introducing redundancy, load balancing, and failover mechanisms, the system is resilient to failures and capable of handling real-world workloads.

It reflects a strong foundation in cloud architecture, networking, and DevOps practices, even when implemented manually.
