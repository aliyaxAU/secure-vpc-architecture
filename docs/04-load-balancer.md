# Application Load Balancer (ALB)

## Overview

In this section I will be configuring the public entry point into the Secure VPC Architecture, the Application Load Balancer (ALB). ALB distributes incoming `HTTP/HTTPS` traffic across private EC2 application servers while keeping those servers completely isolated from the internet.
The ALB lives in the public subnets, but forwards traffic only to private subnets, enforcing a clean separation between public access and internal workloads.

## 1. ALB Configuration steps

In the EC2 dashboard click on load balancer and create a new balancer with the following settings:

- Name: `secure-alb`

- Type: `Application Load Balancer`

- Scheme: `Internet-facing`

- IP Type: `IPv4`

With the subnets I will select both public subnets created in previous steps:

- `public-subnet-2a`

- `public-subnet-2b`

This ensures high availability across two Availability Zones.

## 2. ALB Security Group: alb-sg

Allow public web traffic into the ALB, and allow the ALB to forward traffic to private EC2 instances.

Inbound Rules
| Type | Port | Source | Purpose | 
-|-|-|-|
| HTTP | 80 | 0.0.0.0/0 | Public web traffic | 
| HTTPS | 443 | 0.0.0.0/0 | Secure web traffic | 

Outbound Rules
| Type | Port | Destination | Purpose | 
-|-|-|-|
| HTTP | 80 | app-servers-sg | Forward traffic to private EC2 | 

This ensures that the ALB can only talk to the application servers and nothing else.

## 3. Target Group Configuration

- ALB distributes traffic across AZs

- Health checks ensure only healthy instances receive traffic

- Private EC2 instances remain unreachable from the internet

Target Group

- Name: app-servers-tg
- Target Type: Instances
- Protocol: HTTP
- Port: 80
- Health Checks:
- Protocol: HTTP
- Path: /

Register Targets

Adding private EC2 instances:
- app-server-1 (private-app-subnet-1)
- app-server-2 (private-app-subnet-2)

## 4. Listener Configuration

HTTP Listener
- Port: 80
- Default Action: Forward to app-servers-tg

![alt text](<img/alb configuration.png>)

## 5. Traffic Flow Summary

### Inbound Path
Internet → ALB → Private EC2

### Outbound Path
Private EC2 → NAT Gateway → Internet (for updates)

### Admin Path
<MY_IP_ADDRESS> → Bastion Host → Private EC2

The following will ensure:

- Only the ALB is exposed publicly

- App servers stay private

- SSH access is tightly controlled

## 6. Why the ALB Setup Is important

Security

- No direct access to private EC2

- ALB acts as the only public entry point

- SG-to-SG rules enforce least privilege

High Availability

- ALB spans two public subnets

- App servers span two private subnets

- Traffic automatically shifts if an AZ fails

Scalability

- ALB automatically balances traffic

- Works seamlessly with Auto Scaling Groups
