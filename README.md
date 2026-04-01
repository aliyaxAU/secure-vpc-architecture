# Secure VPC Architecture

This project implements a Secure VPC Architecture on AWS, following best practices for network isolation, least‑privilege access, and multi‑tier design. It demonstrates how to build an environment suitable for web applications, internal services, or regulated workloads.

## Purpose of This Project

The project demonstrates how to create a secure AWS network. It focuses on keeping workloads separate using subnets and routing, ensures that access is limited to what’s necessary through security groups, only makes public elements available when needed, protects databases and internal services and provides clear visibility into network operations.

## Architecture Components

### VPC
- CIDR: `10.0.0.0/16`
- Provides isolated network boundary

### Subnets
- Public: ALB + Bastion
- Private App: EC2 application servers
- Private DB: RDS instances

### Routing
- Public route table -> Internet Gateway
- Private route table -> NAT Gateway

### Security
- Bastion host allows SSH only from trusted IP
- App servers only reachable from ALB + Bastion
- RDS only reachable from app servers
- No public access to private resources

### Monitoring
- VPC Flow Logs
- CloudWatch metrics + alarms
- SNS notifications

## Documentation

Each component has its own documentation:

- VPC + Subnets 
- Routing + Gateways
- Bastion + EC2
- Load Balancer
- RDS
- Security Groups
- Monitoring + issues replications
