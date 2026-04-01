# RDS in Private Subnets

## Overview

This section shows the deployment of the database layer using Amazon RDS. In this project the database will be placed in private subnets, and it is never exposed to the internet and can only be accessed by the application servers. This is the AWS best practices for secure, production‑grade architectures.

## 1. RDS Subnet Group

In the Aurora and RDS Dashboard select Subnet group and create a new DB subnet group with the following configuration

- Name: `rds-subnet-group`

- Subnets: `private-db-subnet-1` and `private-db-subnet-2`

If we were in a production environment, RDS needs a DB subnet group that covers at least two availability zones for multi-AZ deployments. This setup would guarantee high availability, automatic failover, and keep the databases safe from public networks.

However, in a non-production (test) environment, the project will use a single-AZ RDS instance to save costs.

In a live production setting, we would switch to Multi-AZ to ensure high availability and seamless failover.

## 2. RDS DB Configuration

In the Aurora & RDS Dashboard select Databases and create a new Database with the following configuration:

- Engine: `MySQL / PostgreSQL`
- Deployment: `Single‑AZ RDS`
- Instance Class: db.t3.micro
- Public Access: Disabled
- VPC: `secure-vpc`
- Subnet Group: `rds-subnet-group`

Why this is imprtant:

- No public access ensures the DB is never reachable from the internet

- Private subnets enforce strict isolation

###  RDS Security Group

![alt text](<img/rds security group-1.png>)

## 3. Connectivity Flow

Allowed
Private EC2 → RDS
(through `app-servers-sg` security group → `rds-sg` security group)
Not Allowed

- Internet → RDS
- ALB → RDS
- Bastion Host → RDS
- NAT Gateway → RDS


## 4. Connectivity Validation

All connectivity tests were performed from the private app server and i had to make sure the database is reachable only through the intended path:

`
[ec2-user@<private-ec2-host> ~]$ nc -vz database-1.cbqqmc40c6mq.eu-west-2.rds.amazonaws.com 3306
Ncat: Version 7.93 ( https://nmap.org/ncat )
Ncat: Connected to <private-ip>:3306.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.`

### Installation of a Lightweight MySQL Client (PyMySQL)
Amazon Linux 2023 does not include a native MySQL CLI by default, so I had to use a Python client intead:


`sudo dnf install python3 python3-pip -y
python3 -m pip install --user pymysql`

### Set up a Dedicated Application User

Using the same Python client, I will create a least‑privilege user for the application:

![alt text](<img/rds user created.png>)

### Validate connection

![alt text](<img/rds validate connection.png>)
