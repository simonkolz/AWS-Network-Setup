## 📁 Project 2: Application Load Balancer Deployment

This project focuses on deploying an **Application Load Balancer (ALB)** to distribute traffic across EC2 instances in multiple Availability Zones.

Key components include:

* **ALB spanning multiple subnets** for fault tolerance
* **Target Groups** routing traffic to EC2 instances
* **Health Checks** for automatic detection of unhealthy instances
* **Security Groups** ensuring controlled, secure traffic flow
* **Auto Scaling compatibility** for future expansion if needed

The result is a highly available, resilient, and secure load-balanced web architecture.

---

## Architecture Overview

High-level architecture showing VPC, subnets, route tables, Internet Gateway, NAT Gateways, Application Load Balancer, EC2 instances, and Security Groups.

<img width="906" height="654" alt="Screenshot 2025-12-01 at 21 34 42" src="https://github.com/user-attachments/assets/9dd9bb9a-f89d-4055-a49c-55a6f14aacf8" />


Components deployed:

* Custom VPC with CIDR block 10.0.0.0/16

* Two public subnets for ALB + NAT Gateways

* Two private subnets for EC2 instances

* Internet Gateway attached to the VPC

* NAT Gateways in a AZ for outbound internet access from private subnets

* Application Load Balancer distributing traffic across two EC2s

* Auto Scaling Group maintaining redundancy and resilience

* Route 53 DNS + ACM SSL certificate providing secure HTTPS for kolz.link

* Bastion host (for testing only)

---

## Step-by-Step Build


**Step 1 – VPC and Subnets**

Created a custom VPC with CIDR block 10.0.0.0/16.

Provisioned:

* PublicSubnetA (10.0.1.0/24) in eu-west-2a

* PublicSubnetB (10.0.2.0/24) in eu-west-2b

* PrivateSubnetA (10.0.3.0/24) in eu-west-2a

* PrivateSubnetB (10.0.4.0/24) in eu-west-2b

<img width="1199" height="261" alt="Screenshot 2025-12-04 at 01 03 08" src="https://github.com/user-attachments/assets/f6e8de88-4358-4272-84bd-c2d13791b337" />

Reasons: Used non-overlapping CIDR ranges to enable clean subnet separation and seamless deployment across two Availability Zones, ensuring high availability and reliable failover capabilities.

---

## Step 2 – Internet Gateway and Route Tables

* Attached an Internet Gateway to the VPC.

Configured route tables:

* PublicRouteTable → 0.0.0.0/0 via IGW

* PrivateRouteTable → 0.0.0.0/0 via NAT-GW

<img width="1184" height="366" alt="Screenshot 2025-12-04 at 01 25 51" src="https://github.com/user-attachments/assets/a1046826-2830-421c-85d8-0a0b37ad4343" />
<img width="1197" height="409" alt="Screenshot 2025-12-04 at 01 26 19" src="https://github.com/user-attachments/assets/c34199d4-c5ce-425c-bbd2-83ddd639e3a2" />

 This will contols how traffic will be routed within the public and private subnet ensures public resources route through IGW while private EC2s use NAT for outbound internet.

---

## Step 3 – NAT Gateways

* Deployed NAT-GW in PublicSubnetA.
* Updated private route tables to send 0.0.0.0/0 via their local NAT.

<img width="1189" height="406" alt="Screenshot 2025-12-04 at 01 33 35" src="https://github.com/user-attachments/assets/baa71723-aed9-46b0-a6a4-25cb7f12bd06" />

This design keeps private-subnet resources isolated from inbound internet traffic while still enabling secure outbound access for updates, package installations, and data retrieval.

  ---

  ## Step 4 – Security Groups

  **ALB SG (ALB-SG):**

* Inbound: 80/443 from 0.0.0.0/0

* Outbound: all traffic

**Web SG (web-sg):**

* Inbound: 80 from ALB SG only

* Outbound: all traffic

<img width="1182" height="525" alt="Screenshot 2025-12-04 at 01 42 25" src="https://github.com/user-attachments/assets/784bb691-7499-4342-bfc6-222e40b4c6bd" />
<img width="1198" height="453" alt="Screenshot 2025-12-04 at 01 43 30" src="https://github.com/user-attachments/assets/60ebf0ba-3d20-4d40-b236-1f755c0c5b60" />

Reasoning: EC2s are never directly reachable from the internet — all traffic must flow through ALB.

---

## Step 5 – Launch Template & Auto Scaling Group

Created a Launch Template with:

* Ubuntu (AMI)

* Instance type: t3.micro

* Security group: web-sg

* User data script

  <img width="1180" height="586" alt="Screenshot 2025-12-04 at 02 02 34" src="https://github.com/user-attachments/assets/d121fd60-f08a-4f0b-b0fe-eb52cd7700f8" />
<img width="623" height="441" alt="Screenshot 2025-12-04 at 00 07 40" src="https://github.com/user-attachments/assets/3b1e66af-6f63-479d-acad-4b565a3b3704" />



**Configured an Auto Scaling Group:**

Desired capacity = 2

Subnets = PrivateSubnetA and PrivateSubnetB

Attached to ALB target group

Reasoning: ASG ensures at least 2 healthy instances across AZs.

<img width="1172" height="659" alt="Screenshot 2025-12-04 at 02 10 31" src="https://github.com/user-attachments/assets/f34c10d4-6ce5-4eec-bf04-83902a98e2e3" />

---

## Step 6 – Application Load Balancer

**Deployed internet-facing ALB across both public subnets.**

Configured listeners:

* HTTP (80) → redirect to HTTPS (443)

* HTTPS (443) → forward to target group (ALB-TG)

* The target group (ALB-TG) was configured to perform health checks on "/" - the root of the web server.

<img width="1191" height="396" alt="Screenshot 2025-12-04 at 02 13 34" src="https://github.com/user-attachments/assets/1c9a7ed2-b9fc-4a11-ba30-29e3011eb3c0" />
<img width="1185" height="370" alt="Screenshot 2025-12-04 at 02 15 33" src="https://github.com/user-attachments/assets/92b6db59-33c7-4be1-a102-8ce2b72de05d" />
<img width="1195" height="451" alt="Screenshot 2025-12-04 at 02 16 42" src="https://github.com/user-attachments/assets/9e979624-0737-4413-aa45-734626489235" />

Reasoning: ALB provides fault tolerance + HTTPS termination.

---

## Step 7 – HTTPS with ACM and Route53

* Requested an ACM certificate for kolz.link

* Validated via DNS an **A record** with an **Alias** in **Route53**.

* Attached certificate to ALB HTTPS listener.

In Route53:

* Created A record with Alias for kolz.link → ALB DNS hostname.

<img width="1155" height="227" alt="Screenshot 2025-12-04 at 02 22 44" src="https://github.com/user-attachments/assets/6794a755-72a1-4fcb-a5a4-c54328865d6c" />

---

## Step 8 - Testing WebbApp/Certificate Validity

* Searched DNS in Browser
* Tested webbapp with nslookup

<img width="1177" height="289" alt="Screenshot 2025-12-04 at 02 50 21" src="https://github.com/user-attachments/assets/deddcc78-2ae2-4934-9165-8ca5d733c7e4" />
<img width="570" height="332" alt="Screenshot 2025-12-04 at 02 51 09" src="https://github.com/user-attachments/assets/b08bbb63-846f-46a5-9383-cf122922b706" />

---

## Testing NAT Gateway and Bastion Host

To test how everything worked, I deployed a Bastion host in PublicSubnetA, allowed SSH only from my IP, and permitted SSH from

bastion-sg → web-sg.

From my laptop I connected to the Bastion (public IP) and then into the private EC2s (private IPs).

Once inside the private instances, I verified outbound internet access via the NAT Gateways by running:

"ping google.com"


Both succeeded → confirming the private EC2s had outbound connectivity through the NAT Gateways.

Created Bastion Security Group(SG)

<img width="1202" height="469" alt="Screenshot 2025-12-04 at 03 04 21" src="https://github.com/user-attachments/assets/b560e067-67e3-4d6d-bcf2-caac0a71c89e" />


**Edited WebSG to allow SSH from Bastion:**
<img width="1197" height="521" alt="Screenshot 2025-12-04 at 03 06 22" src="https://github.com/user-attachments/assets/276376c9-33c7-4375-92ab-79bc654655d3" />

SSH into Bastion:

<img width="572" height="462" alt="Screenshot 2025-12-04 at 03 19 50" src="https://github.com/user-attachments/assets/9072f3eb-93c9-4f82-ac56-22b7938a08c3" />


**SSH(jumpbox) into private EC2 Instance in the Private subnet**

<img width="571" height="464" alt="Screenshot 2025-12-04 at 03 38 39" src="https://github.com/user-attachments/assets/cb02a773-997d-4bd8-9d04-f96dcca0904f" />

**Test to see if i get a response by entering "ping google.com" in private instance:**

* The private instance has outbound internet access
* The route table is pointing 0.0.0.0/0 to the NAT Gateway
* The NAT Gateway is working correctly

<img width="596" height="477" alt="Screenshot 2025-12-04 at 03 55 56" src="https://github.com/user-attachments/assets/17d33ab6-3583-47ae-9066-38b74b30d85b" />

---

## Testing Auto Scaling Group and Target group

Auto Scaling Test: terminated one EC2; ASG launched replacement → passed health checks and served traffic.


<img width="1177" height="368" alt="Screenshot 2025-12-04 at 04 33 08" src="https://github.com/user-attachments/assets/ae99749e-ea38-48fe-b3b5-eaa55c284b75" />
<img width="1188" height="417" alt="Screenshot 2025-12-04 at 04 30 33" src="https://github.com/user-attachments/assets/5bbf7987-e1d1-4214-9c1c-3d21a628bd42" />
<img width="1154" height="272" alt="Screenshot 2025-12-04 at 04 29 28" src="https://github.com/user-attachments/assets/99367062-7f31-4df4-949c-8cd79ce95198" />
<img width="1177" height="368" alt="Screenshot 2025-12-04 at 04 33 08" src="https://github.com/user-attachments/assets/1e3176a7-5fb4-41b5-9f55-ee02375840d2" />
<img width="1158" height="270" alt="Screenshot 2025-12-04 at 04 40 27" src="https://github.com/user-attachments/assets/743b8f96-c52b-46a7-96d7-49510adfc0ee" />
