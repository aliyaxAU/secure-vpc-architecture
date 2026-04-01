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

- [VPC + Subnets](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/01-vpc-and-subnets.md)
- [Routing + Gateways](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/02-routing-and-gateway.md)
- [Bastion + EC2](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/03-bastion-and-ec2.md)
- [Load Balancer](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/04-load-balancer.md)
- [RDS](https://github.com/aliyaxAU/secure-vpc-architecture/blob/release-5/RDS-private-subnets/docs/05-rds-private-subnets.md)
- [Security Groups](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/06-security-groups.md)
- [Monitoring + issues replications](https://github.com/aliyaxAU/secure-vpc-architecture/blob/main/docs/07-monitoring-and-flowlogs.md)
