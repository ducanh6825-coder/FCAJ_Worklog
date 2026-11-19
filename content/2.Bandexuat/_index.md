---
title: "Proposal"
weight: 2
chapter: false
pre: "<b>2. </b>"
---

## Design and Deployment of a Secure Web Server on AWS Using VPC, EC2, S3, and CloudWatch.

### 2.1 Summary

This project proposes a basic Web Server system on AWS, focusing on **firewall-based network access control, data security, and operational monitoring**, while optimizing costs for a hands-on learning environment.

The system uses four core services: **Amazon VPC, Amazon EC2, Amazon S3, and Amazon CloudWatch**. An EC2 instance running Nginx is deployed in a Public Subnet to serve the website over HTTPS. Security Groups and Network ACLs control network traffic; a private S3 bucket stores website assets and backups; and CloudWatch collects metrics and logs and triggers alerts.

The team consists of **four members**, responsible for networking and firewalls, the Web Server, storage and data security, and monitoring and costs, respectively. All members deploy their components within one shared system and develop their corresponding workshop sections in Markdown.

The planned project duration is **four weeks**, with a target of approximately **48 total EC2 running hours**. This is a hands-on model supported by testing and evidence, rather than a system intended for continuous operation or production-level high availability.

### 2.2 Problems to Be Addressed

| Problem | Requirement |
|---|---|
| The Web Server is directly exposed to the Internet and may be subject to unauthorized access | Open only necessary ports, restrict SSH source addresses, and test both permitted and denied access |
| Data may be exposed through public S3 access or overly broad permissions | Keep the bucket private, apply least-privilege permissions, use IAM Roles, and encrypt data |
| Data may be accidentally overwritten or deleted | Enable Versioning, maintain backups, and practice data recovery |
| Insufficient information is available to detect and investigate incidents | Collect metrics and logs, build a Dashboard, and test alerts |
| Resources left running outside practice sessions may generate additional costs | Limit running hours, storage volume, and log retention; monitor costs and clean up resources |
| Inconsistent configurations across team members make the system difficult to reproduce | Agree on shared configuration values, assign task-based permissions, develop workshops, and retain test evidence |

The project's central objective is to demonstrate that the security controls work through practical testing: legitimate access succeeds, access outside the permitted scope is blocked, data is protected and recoverable, and notifications are delivered when configured alert conditions are met.

### 2.3 Proposed Solution

**Overall Architecture**

Deploy the system in the **Singapore Region (`ap-southeast-1`)**, using one VPC, one Public Subnet, and one shared EC2 instance in a single Availability Zone.

Users access the website through an Internet Gateway to EC2, with traffic controlled by a Network ACL and a Security Group. EC2 accesses the private S3 bucket through an S3 Gateway VPC Endpoint using an IAM Role. Metrics and logs are sent to CloudWatch for monitoring, while SNS supports email alert notifications.

![Sơ đồ kiến trúc](/FCAJ_Worklog/images/5/so_do_kien_truc_aws.drawio.png?featherlight=false&width=90pc)

**Networking and Firewall Configuration — Amazon VPC**

- Create VPC `vpc-secure-web` with CIDR `10.0.0.0/16` and Public Subnet `subnet-web-public` with CIDR `10.0.1.0/24`.
- Configure an Internet Gateway and a Route Table to provide Internet connectivity for EC2.
- Use a Security Group as the primary access control layer: allow port 22 only from the administrator's IP address in `/32` format; use port 443 for HTTPS; and use port 80 for HTTPS redirection and certificate validation when needed.
- Configure a Network ACL as an additional subnet-level filtering layer, ensuring that rules cover both traffic directions and return traffic.
- Create an S3 Gateway Endpoint and associate it with the subnet's Route Table.
- Test authorized source addresses, unauthorized sources, and an IP-blocking scenario using the Network ACL.

Security Groups provide stateful traffic filtering, while Network ACLs provide stateless filtering. These components control network traffic; they do not replace HTTP content inspection mechanisms for preventing SQL injection or XSS.

**Web Server Deployment — Amazon EC2**

- Use one `t4g.micro` EC2 instance with Amazon Linux 2023 ARM64 and an encrypted **8 GiB** gp3 EBS volume; select **Standard** CPU credit mode.
- Install Nginx, deploy a demonstration website, and configure HTTPS.
- Use a valid certificate with a domain managed by the team. If no domain is available, use a self-signed certificate only for encryption testing and explicitly document this limitation.
- Administer the instance using an SSH key, restrict source IP addresses, require IMDSv2, and attach an IAM Role to EC2.
- Synchronize website assets from S3 to the Nginx serving directory, and back up the required content and configuration files to S3.
- Do not include access keys, passwords, SSH private keys, or TLS private keys in source code or supporting evidence.

**Storage and Data Security — Amazon S3**

- Create a private bucket in the same Region, enable all Block Public Access settings, and use SSE-S3 encryption.
- Enable Versioning and organize data under the `assets/` and `backups/` prefixes.
- Grant the EC2 IAM Role least-privilege access based on the required bucket, prefixes, and operations.
- Apply a bucket policy that denies data transfers without TLS, and restrict the endpoint policy to the designed access scope.
- Test authorized access, denied access, and version recovery after accidental overwriting or deletion.
- Configure Lifecycle rules according to the agreed retention periods for backups and older versions.

The bucket does not directly serve a public website. EC2 synchronizes the content approved for publication from S3 and serves it through Nginx.

**Monitoring and Alerts — Amazon CloudWatch**

- Monitor CPU utilization, network traffic, and EC2 status; use the CloudWatch Agent to collect additional memory and disk usage metrics and Nginx logs.
- Build **one Dashboard** summarizing the required metrics.
- Maintain no more than **two standard alarms** active at the same time, testing different alarm conditions in separate sessions.
- Use SNS to send notifications to an email address with a confirmed subscription.
- Enable VPC Flow Logs only during firewall testing sessions; retain ACCEPT/REJECT evidence and compare it with the configuration for analysis.
- Set CloudWatch Logs retention to **1–3 days** during the hands-on phase.

**Cost Optimization**

The team will use one shared system, stop EC2 after each practice session, target approximately **48 total running hours over four weeks**, and limit S3 data to approximately **200 MB**, including monitoring of stored versions and backups.

Retain the agreed design: do not deploy a NAT Gateway, Elastic IP, ALB, AWS WAF, AWS Network Firewall, or Interface Endpoint within the basic project scope. The **USD 3–5 budget for six weeks** is retained as a planning target from the previous proposal, not as a price quotation or a guarantee of actual costs. The team must verify current pricing and account benefits before deployment, monitor incurred charges, and review actual costs at project completion.

Stopping EC2 does not eliminate all storage charges. After completing the project and retaining the evidence, team members will clean up resources within their assigned permissions; the account owner will handle actions beyond those permissions.

**Team Responsibilities**

| Team Member | Responsibilities | Deliverables |
|---|---|---|
| Member 1 | VPC, subnet, Internet Gateway, Route Table, Security Group, Network ACL, and S3 Gateway Endpoint | Network configuration, firewall rule table, test results, and VPC workshop |
| Member 2 | EC2, Nginx, HTTPS, and server security | Demonstration website, Web Server configuration, and EC2 workshop |
| Member 3 | S3, data access permissions, encryption, Versioning, backup, and recovery | S3 configuration, permission policies, recovery evidence, and S3 workshop |
| Member 4 | CloudWatch, logs, Dashboard, alerts, and cost reporting | Dashboard, alert evidence, cost report, and CloudWatch workshop |

Each member receives least-privilege permissions for their assigned tasks. The account owner manages foundational permissions and budget configuration; the team does not share the root account or access keys.

### 2.4 Expected Outcomes

Upon completion, the project is expected to achieve the following outcomes:

| Outcome | Verification Criteria |
|---|---|
| The website operates on EC2 | The website is accessible over HTTPS while EC2 is running; HTTP redirects to HTTPS; the certificate type is documented |
| Firewall controls enforce the intended access rules | SSH succeeds from the allowed IP address; connections from unauthorized sources or to closed ports are blocked; NACL test evidence is available |
| S3 data is protected | The bucket is private, data is encrypted, EC2 uses an IAM Role for access, and operations outside the permitted scope are denied |
| The recovery mechanism is verified | At least one accidentally overwritten or deleted data item is successfully recovered, with the procedure and results documented |
| Monitoring and alerts function correctly | The Dashboard displays data, logs are collected, and at least one test alert is successfully delivered |
| Costs are controlled | Running hours are recorded, actual costs are compared with the target, and any differences are explained |

