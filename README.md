# Multi-Region Highly Available 3-Tier Application on AWS

## Overview

This project demonstrates the deployment of a **highly available and fault-tolerant 3-tier application** across two AWS regions. The application, sourced from [3-tier-app GitHub repo](https://github.com/arjunkoppineni/3-tier-app.git), combines the frontend and backend into a single Spring Boot application with an RDS MySQL database.

The infrastructure is created in two regions using two different Infrastructure-as-Code (IaC) tools:

* **Region 1**: Deployed using AWS CloudFormation.
* **Region 2**: Deployed using Terraform.

High availability is achieved through:

* Multi-AZ VPC and subnet setup
* EKS clusters with managed node groups in each region
* RDS Multi-AZ deployment
* Route 53 with **failover routing policy** across regions

---

## Architecture Diagram

```
                         Internet
                            |
                       [Route 53]
                    /               \
         [ALB - Region 1]       [ALB - Region 2]
              |                        |
           [EKS]                    [EKS]
              |                        |
        [Spring Boot App]       [Spring Boot App]
              |                        |
         [RDS MySQL]             [RDS MySQL]
```

---

## Components

### Application Layer

* **Source**: [3-tier-app](https://github.com/arjunkoppineni/3-tier-app.git)
* **Stack**: Spring Boot + Thymeleaf templates
* **Database**: AWS RDS (MySQL)

### Infrastructure Layer

#### CloudFormation Stack (Region 1)

* VPC with 2 public and 4 private subnets
* Internet Gateway and NAT Gateway
* Route Tables with route associations
* RDS MySQL with Multi-AZ deployment
* IAM Roles for EKS Cluster and Node Group
* EKS Cluster
* EKS Node Group
* CloudWatch Alarms
* SNS Topic and Email Subscription

#### Terraform Stack (Region 2)

* Equivalent setup to Region 1
* Uses same application and architecture design

---

## CloudFormation Template Features

### VPC and Subnet Setup

* 1 VPC with configurable CIDR (`10.0.0.0/16` or `20.0.0.0/18`)
* Public Subnets in AZ1 and AZ2
* Private Subnets in AZ1 and AZ2

### Routing and Internet Access

* NAT Gateway in Public Subnet AZ1
* IGW attached to VPC
* Public route table routes internet traffic via IGW
* Private route table routes via NAT Gateway

### Database Layer

* **RDS MySQL** with Multi-AZ deployment
* Accessible via security group from app subnets
* DBSubnetGroup used for RDS placement
* Monitored using CloudWatch alarm for high CPU utilization

### Compute Layer (EKS)

* EKS Cluster with IAM Role
* EKS Managed NodeGroup with autoscaling (min: 1, max: 3)
* Worker node IAM role with policies to manage EC2, EKS, ECR, and SSM

### Monitoring

* CloudWatch Alarms for:

  * High RDS CPU Utilization
  * High EKS Pod CPU Utilization
* SNS Topic for alert notifications (email based)

### Outputs

* RDS endpoint exported for application usage

---

## Route 53 Failover

* Two ALBs (one in each region)
* **Route 53 Failover Routing** is configured to monitor the health of the primary region's ALB
* If the primary ALB becomes unhealthy, traffic automatically fails over to the secondary region's ALB

---

## Prerequisites

* AWS CLI configured with proper access
* Verified email for SNS notifications
* Kubernetes tools: `kubectl`, `eksctl`
* Terraform installed (for Region 2 setup)
* Helm (optional, for managing deployments to EKS)

---

## Deployment Steps

### Region 1 - CloudFormation

1. Clone the repo

```bash
git clone https://github.com/arjunkoppineni/3-tier-app.git
cd 3-tier-app
```

2. Deploy the CloudFormation stack

```bash
aws cloudformation create-stack \
  --stack-name MultiRegionAppStack \
  --template-body file://your-template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters ParameterKey=MyVpcCidr,ParameterValue=10.0.0.0/16 \
               ParameterKey=MyDBRootUserPassword,ParameterValue=your-password \
               ParameterKey=MySNSEmailEndpoint,ParameterValue=your-email@example.com
```

3. Confirm SNS subscription by verifying email

4. Deploy the Spring Boot app to EKS (via `kubectl apply -f deployment.yaml`)

---

### Region 2 - Terraform

> Instructions would be similar but using Terraform configuration files.

```bash
cd terraform-region-2
terraform init
terraform apply
```

---

## Notes

* Replace placeholder values in parameters for passwords and email
* RDS MySQL credentials and endpoints should be securely referenced in your app's deployment manifest
* Use ALB Ingress Controller or AWS Load Balancer Controller for exposing services in EKS

---

## Security Best Practices

* Do not open MySQL to `0.0.0.0/0` in production
* Use Secrets Manager or SSM Parameter Store for DB credentials
* Ensure IAM roles follow least privilege principle

---

## Monitoring & Alerting

* CloudWatch Alarms for RDS and EKS
* SNS Topic with email notifications
* Can be extended with CloudWatch Dashboards and Container Insights

---

## Future Enhancements

* Add CI/CD pipeline with CodePipeline and CodeBuild
* Integrate WAF and Shield for ALB
* Add Redis or ElastiCache for caching layer
* Auto-scale EKS workloads using HPA

---

## License

This project is open-source and available under the MIT License.

---

## Author

**Arjun Koppineni**
GitHub: [arjunkoppineni](https://github.com/arjunkoppineni)
