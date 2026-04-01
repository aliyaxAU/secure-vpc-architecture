# Security group architecture

This section explains the security group design for the AWS environment. As we all know security groups act like virtual firewalls, they control traffic between the public tier, private tier, and database layer.

In this implementation we are going to follow AWS best practices: least privilege and no public access to private resources.

### Security Group Overview

| Security Group | Purpose | Attached To | 
-|-|-|
| alb-sg | Accepts public `HTTP/HTTPS` traffic | Application Load Balancer | 
| bastion-sg | Allows `SSH` from admin IP | Bastion Host (EC2) | 
| app-servers-sg | Accepts traffic from ALB and SSH from bastion | App Servers (EC2) | 
| ec2-rds-1 | Allows `MySQL` only from app servers | RDS MySQL Instance | 

## Inbound & Outbound Rules


### 1. Application Load Balancer SG  `alb-sg` (Public Tier)

**What this security group does:**

- Make the application accessible online

- Directs traffic exclusively to app-servers-sg

**Inbound**
- `TCP` `80` from `0.0.0.0/0`
- `TCP` `443` from `0.0.0.0/0`

**Outbound**
- All traffic (default)

### 2. Bastion SG `bastion-sg` (Admin Access)

**What this security group does:**

- Secure SSH jump host

- There is no direct SSH to private servers from the internet

**Inbound**

- `TCP` `22` from my IP only

**Outbound**

- `TCP` `22` to `app-servers-sg`

### 3. Private Application SG `app-servers-sg`

**What this security group does:**
- Receive traffic from ALB only
- Allow SSH only from bastion
- Possibility to reach RDS for database queries

**Inbound**
- `TCP` `80` from `alb-sg`
- `TCP` `22` from `bastion-sg`

**Outbound**
- `TCP` `3306` to `ec2-rds-1`
- All outbound (default)

### 4. Database SG `ec2-rds-1`

**What this security group does:**

- Ensure the DB is reachable from the application layer only
- There is no public access, SSH and direct admin access

**Inbound**
- `TCP` `3306` from `app-servers-sg`

**Outbound**
- All outbound (default)

## Traffic Flow Summary

Internet -> ALB (Application Load Balancer) (`alb-sg`) -> App Servers (`app-servers-sg`) -> RDS MySQL (`ec2-rds-1`)

**Admin access:**

My Device -> Bastion Host (`bastion-sg`) -> App Servers (`app-servers-sg`)


## Security Best Practices Implemented

- There is no public access to private EC2 or RDS

- Security group - to - Security group rules instead of CIDR ranges

- Bastion host restricted to a single IP

- RDS restricted to app servers only

- ALB is the only public entry point

- No SSH keys stored on app servers except via bastion

