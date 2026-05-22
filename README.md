# TravelWish — Travel Bucket List App

A full-stack travel bucket list web application built on traditional server-based AWS infrastructure, designed from scratch. Built as a cloud engineering portfolio project.

## Architecture

```
                    Internet
                       │
                       ▼
              Internet Gateway
                       │
                       ▼
         ┌─────── Public Subnet ────────┐
         │  ┌──────────────────────┐    │
         │  │ Application Load     │    │
         │  │ Balancer             │    │
         │  └──────────┬───────────┘    │
         │             │                │
         │    ┌────────┴────────┐       │
         │    ▼                 ▼       │
         │  EC2 App-1        EC2 App-2  │
         │  Apache+Flask     Apache+Flask│
         │  (travelwish-     (travelwish-│
         │   app-sg)          app-sg)   │
         └─────────┬───────────┬────────┘
                   │           │
                   └─────┬─────┘
                         │ Port 3306 only
                         ▼
         ┌────── Private Subnet ───────┐
         │  ┌──────────────────────┐   │
         │  │  RDS MySQL           │   │
         │  │  (travelwish-db-sg)  │   │
         │  └──────────────────────┘   │
         └─────────────────────────────┘

         All inside: Custom VPC — us-west-2
         Two Availability Zones: us-west-2a, us-west-2b
         Auto Scaling Group: min 1 / desired 2 / max 4
```

##  AWS Services Used

| Service | Purpose |
|---------|---------|
| VPC | Custom private network containing all resources |
| Public Subnets | Two AZs — where EC2 app servers live |
| Private Subnets | Two AZs — where RDS database lives, no internet access |
| Internet Gateway | Connects public subnet to the internet |
| Route Tables | Directs public subnet traffic through the IGW |
| Security Groups | `travelwish-app-sg` for EC2, `travelwish-db-sg` for RDS |
| EC2 (x2) | App servers running Apache + Python Flask REST API |
| Application Load Balancer | Single entry point, distributes traffic across both EC2s |
| Auto Scaling Group | Adds/removes EC2 instances based on CPU utilization |
| RDS MySQL | Relational database in private subnet — destinations data |
| IAM | Roles and policies for EC2 access |

## Security Design

The database has **no public access**. The RDS security group (`travelwish-db-sg`) only accepts inbound connections on port 3306 from the app security group (`travelwish-app-sg`) — not from any IP range. This means even if someone had the RDS endpoint URL, they could not connect without being an EC2 instance inside the app security group.

```
Internet ──✗──► RDS (blocked)
EC2 (app-sg) ──✓──► RDS port 3306 (allowed)
```

##  Features

- Add travel destinations with country, category, priority, and notes
- Filter by category (Asia, Europe, Americas, etc.) and priority
- Mark destinations as visited
- Detail panel showing full destination info
- All data persists in RDS MySQL — no localStorage

##  REST API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/destinations` | Get all destinations |
| POST | `/destinations` | Add a new destination |
| PUT | `/destinations/<id>` | Update a destination |
| DELETE | `/destinations/<id>` | Delete a destination |

## Tech Stack

**Frontend:** HTML, CSS, JavaScript  
**Backend:** Python (Flask REST API)  
**Database:** Amazon RDS MySQL  
**Web Server:** Apache  
**Infrastructure:** AWS VPC, EC2, ALB, Auto Scaling

## Project Structure

```
travelwish/
├── travelwish.html    # Frontend + Flask API in one file
└── README.md
```

## What I Learned

- Designing a VPC from scratch — subnets, route tables, internet gateway
- Why RDS is preferred over DynamoDB for relational/structured data
- Security group chaining to protect database from public access
- Application Load Balancer target groups and health checks
- Auto Scaling launch templates and scaling policies
- The difference between public and private subnet architecture

## Note on Running This Project

This project runs on EC2 which incurs AWS costs when running. To avoid charges the instances are stopped when not in use. To run the project:

1. Start EC2 instances from the AWS console
2. Note the new public IPs (they change on restart)
3. Access via the ALB DNS name (stays the same)
4. Stop instances when done
