# 🚀 AWS Multi-Tier Application Deployment (Multi-Region, EKS, RDS, Route 53, CI/CD)

## 📌 Table of Contents

* [1. Introduction](#1-introduction)
* [2. Application Overview](#2-application-overview)
* [3. Infrastructure Design Principles](#3-infrastructure-design-principles)
* [4. Overall Architecture](#4-overall-architecture)
* [5. CloudFormation Deployment - Region 1](#5-cloudformation-deployment---region-1)
* [6. Terraform Deployment - Region 2](#6-terraform-deployment---region-2)
* [7. Global Traffic Management (Route 53)](#7-global-traffic-management-route-53)
* [8. CI/CD Pipeline Setup](#8-cicd-pipeline-setup)
* [9. Monitoring & Alerting](#9-monitoring--alerting)
* [10. Security Best Practices](#10-security-best-practices)
* [11. Future Enhancements](#11-future-enhancements)
* [12. Author](#12-author)

---

## 1. Introduction

This project provisions a **highly available**, **fault-tolerant**, **multi-tier web application** infrastructure in two AWS regions using **CloudFormation** and **Terraform**. It uses **Amazon EKS** for container orchestration, **Amazon RDS MySQL** for the database, and **Route 53** for global failover routing.

---

## 2. Application Overview

* **GitHub Repo**: [3-tier-app](https://github.com/arjunkoppineni/3-tier-app.git)
* **Tech Stack**: Spring Boot + Thymeleaf + RDS (MySQL)
* **Infra Tools**: CloudFormation (Region 1), Terraform (Region 2)

---

## 3. Infrastructure Design Principles

| Goal              | Strategy                                              |
| ----------------- | ----------------------------------------------------- |
| High Availability | Multi-AZ subnets, Multi-region failover with Route 53 |
| Fault Tolerance   | Redundant EKS node groups, Route 53 failover policy   |
| Scalability       | EKS Node Group Auto Scaling                           |
| DR Readiness      | Active/passive regional setup with ALB + Route 53     |

---

## 4. Overall Architecture

```
                         🌐 Internet
                              |
                         [ Route 53 ]
                      /                  \
         📍 Region 1 (us-east-1)       📍 Region 2 (us-west-2)
         ------------------------       ------------------------
         [ ALB ]                       [ ALB ]
            |                              |
         [ EKS ]                        [ EKS ]
            |                              |
   [ Spring Boot App ]           [ Spring Boot App ]
            |                              |
      [ RDS MySQL DB ]            [ RDS MySQL DB ]
```

---

## 5. CloudFormation Deployment - Region 1

### 💡 Architecture

See: [`vpc-eks-rds.yaml`](./region-1-cloudformation/vpc-eks-rds.yaml)

### 🔍 Key Features

| Layer      | Resources                                                     |
| ---------- | ------------------------------------------------------------- |
| Network    | VPC, Public & Private Subnets, IGW, NAT Gateway, Route Tables |
| Compute    | EKS Cluster, EKS Node Groups with IAM Roles                   |
| Database   | RDS MySQL with Multi-AZ + Security Groups                     |
| Monitoring | CloudWatch Alarms for RDS & EKS, SNS Topic                    |
| IAM        | Roles for EKS, EC2, CloudWatch, CodeBuild                     |

### 🛠 Deployment Steps
Create a Stack in CloudFormation and use the yaml file where resources are Specifed.

                (or)
Create Infrastructure using CloudFormation and CodePipeline. Create a Git Repo and Place the Yaml file in that repo. Next Create a pipeline and select Source as Github and select Deploy stage (CF). Deploy to Cloudformation. It will create a stack.
                

---

## 6. Terraform Deployment - Region 2

### 💡 Terraform Structure

```hcl
provider "aws" {
  region = "us-west-2"
}

module "vpc" { ... }
module "eks" { ... }
resource "aws_db_instance" "mysql" { ... }
```

### 🛠 Steps

```bash
cd region-2-terraform
terraform init
terraform apply
```

---

## 7. Global Traffic Management (Route 53)

### 🧠 Failover Logic

| Record Type | Region    | Failover Role | Health Check |
| ----------- | --------- | ------------- | ------------ |
| A (Alias)   | us-east-1 | PRIMARY       | Enabled      |
| A (Alias)   | us-west-2 | SECONDARY     | N/A          |

> If primary ALB is unhealthy, traffic fails over to secondary.

---

## 8. CI/CD Pipeline Setup

### 📦 Tools Used

| Service      | Purpose                                  |
| ------------ | ---------------------------------------- |
| CodePipeline | Pipeline orchestration                   |
| CodeBuild    | Builds Spring Boot app & deploys to EKS  |
| CodeDeploy   | (Optional) EC2/ECS deployment management |
| ECR          | (Optional) Container image storage       |
| S3           | (Optional) Artifact storage              |

### 📊 Architecture Diagram

```
┌────────────┐       ┌────────────┐       ┌──────────────┐       ┌────────────┐
│  GitHub    │ ───▶  │ CodeBuild  │ ───▶  │ EKS/Kubernetes│ ───▶ │ CodeDeploy │
└────────────┘       └────────────┘       └──────────────┘       └────────────┘
     ▲                        │                   │                       │
     └─────────────[Triggered on push]────────────┴────[Rolling updates / hooks]
```

### 🧰 Sample `buildspec.yml`

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      java: corretto11
  build:
    commands:
      - mvn clean package
      - kubectl apply -f deployment.yaml
artifacts:
  files: ["**/*"]
```


### 🛠 CI/CD Setup Steps

1. Create CodePipeline with stages: Source (GitHub), Build (CodeBuild), Deploy (CodeBuild or CodeDeploy)
2. Store Docker image in ECR (if containerized)
3. Use IAM roles with EKS/CodeBuild/CodeDeploy permissions
4. Monitor via CloudWatch Logs

---

## 9. Monitoring & Alerting

### 🔔 Alarms

| Metric                 | Threshold | Action    |
| ---------------------- | --------- | --------- |
| RDS CPUUtilization     | >70%      | SNS Email |
| EKS Pod CPUUtilization | >80%      | SNS Email |

> Alerts delivered via SNS Topic (email notification)

---

## 10. Security Best Practices

* **Private RDS**: Not publicly accessible
* **IAM Roles**: Least privilege principle
* **Secrets**: Use SSM Parameter Store or Secrets Manager
* **Ingress Rules**: Restrict traffic using SGs and NACLs

---

## 11. Future Enhancements

* Add **WAF + Shield** for ALB
* Integrate **Prometheus + Grafana** for observability
* Extend CI/CD with canary/blue-green deployments

---

## 12. Author

**Arjun Koppineni**
GitHub: [@arjunkoppineni](https://github.com/arjunkoppineni)

---

## 📝 Useful Commands

```bash
aws eks update-kubeconfig --region <region> --name <cluster-name>
kubectl get nodes
kubectl get svc
kubectl logs -l app=myapp
```

