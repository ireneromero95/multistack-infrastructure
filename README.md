#  MultiStack Voting App — AWS Infrastructure

A multi-service voting application fully deployed on **AWS** using **Terraform**, **Ansible**, and **Docker**. Built as a capstone project during the Ironhack Cloud & DevOps Bootcamp.

---

##  Architecture

```
 Internet
    │
 ┌──▼──────────┐
 │   AWS ALB   │  (Path-based routing)
 └──┬──────┬───┘
    │      │
 /vote    /result
    │      │
 ┌──▼──┐ ┌─▼────────┐
 │Vote │ │Result EC2│
 │EC2  │ │(Node.js) │
 │(Py) │ └──┬───────┘
 └──┬──┘    │
    │        │
 ┌──▼────────▼──────────┐
 │      Private Subnet  │
 │  ┌───────┐ ┌───────┐ │
 │  │ Redis │ │Postgre│ │
 │  └───────┘ └───────┘ │
 │     ┌────────┐        │
 │     │ Worker │        │
 │     │ (.NET) │        │
 │     └────────┘        │
 └──────────────────────┘
         │
 ┌───────▼───────┐
 │  Bastion Host │  (SSH access)
 └───────────────┘
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Infrastructure as Code | Terraform (modular) |
| Configuration Management | Ansible |
| Containerization | Docker + Docker Compose |
| Cloud Provider | AWS (EC2, VPC, ALB, Security Groups) |
| Vote Service | Python / Flask |
| Result Service | Node.js / Express |
| Worker | .NET 7.0 (C#) |
| Queue | Redis |
| Database | PostgreSQL |

---

## Repository Structure

```
MultiStack-Infrastructure/
├── terraform/              # Modular Terraform infrastructure
│   ├── modules/
│   │   ├── vpc/            # VPC, subnets, route tables, NAT Gateway
│   │   └── ec2/            # EC2 instances (vote, result, worker, bastion)
│   ├── main.tf             # ALB, security groups, listeners & target groups
│   ├── variables.tf
│   └── outputs.tf
├── ansible/                # Playbooks to configure EC2 & deploy containers
├── vote/                   # Python/Flask voting app
├── result/                 # Node.js results dashboard
├── worker/                 # .NET worker service
├── healthchecks/           # Health check scripts
└── docker-compose.yml      # Local development stack
```

---

## How to Deploy

### Prerequisites

- AWS CLI configured (`aws configure`)
- Terraform >= 1.3
- Ansible >= 2.12
- An existing EC2 key pair in AWS

### 1. Provision Infrastructure with Terraform

```bash
cd terraform
# Important: always create .gitignore before terraform init
terraform init
terraform plan
terraform apply
```

This will create:

- VPC with public and private subnets
- Application Load Balancer with path-based routing (`/vote` → Vote EC2, `/result` → Result EC2)
- EC2 instances for each service
- Bastion Host for secure SSH access
- Security Groups with least-privilege rules
- NAT Gateway for private subnet internet access

### 2. Configure & Deploy with Ansible

```bash
cd ansible
# Update inventory with Terraform outputs
ansible-playbook -i inventory playbook.yml
```

Ansible handles:

- Installing Docker on each EC2 instance
- Pulling the correct images from Docker Hub
- Running containers with the right environment variables
- Setting up inter-service networking

### 3. Access the Application

After deploy, get the ALB DNS from Terraform outputs:

```bash
terraform output alb_dns_name
```

- **Vote:** `http://<ALB_DNS>/vote`
- **Results:** `http://<ALB_DNS>/result`

---

##  Local Development

To run the full stack locally:

```bash
docker compose up
```

- Vote app: http://localhost:8080
- Results: http://localhost:8081

---

## Key Lessons Learned

- Always create `.gitignore` **before** running `terraform init` (avoid committing `.terraform/` state)
- Never edit `~/.ssh/config` in VS Code — use terminal heredoc to avoid encoding issues
- Environment variable names must match **exactly** between Ansible playbook and Docker image expectations
- Path-based routing on ALB requires careful Target Group and Listener Rule configuration

---

## Authors

| Author | Links |
|---|---|
| **João Ribeiro** | [GitHub](https://github.com/joaodmorgadoribeiro-del) · [LinkedIn](https://www.linkedin.com/in/joaoribeiro9595) |
| **Irene Romero** | [GitHub](https://github.com/ireneromero95) · [LinkedIn](http://linkedin.com/in/irene-romero-mart%C3%ADnez-0b6215119/) |

---

*Ironhack Cloud & DevOps Bootcamp — Capstone Project ��*
