## 📁 Project 1: AWS VPC Network Setup

In this project, I designed and deployed a fully secure Virtual Private Cloud (VPC) architecture across multiple Availability Zones. This foundational networking setup includes:

* **Multi-AZ VPC** configuration for high availability
* **Public and Private Subnets** split across two Availability Zones
* **Internet Gateway** for controlled ingress/egress
* **NAT Gateway** to allow private instances secure outbound access
* **Bastion Host (Jump Box)** for secure administrative access to private EC2 instances
* **Route Tables & Security Groups** designed with the principle of least privilege

This architecture provides a production-ready network environment suitable for secure workloads and scalable applications.

## Step by Step Build

**Step 1) VPC & Subnets**
*  I Create  VPC using the **CIDR block 10.0.0.0/16** - This was done to provide a large and flexible IP range to support future scalability.
*  Then Implement a **multi-AZ** architecture with **two Availability Zones**, each containing one public and one private subnet to ensure high availability and fault tolerance.
*  Enable auto-assign public IPs on the public subnets to allow the **Bastion host** and **NAT Gateway** to access the internet.

  This design follows the principle of least privilege by clearly separating public-facing resources from private workloads, aligning with best practices used in production environments.
  <img width="815" height="498" alt="Screenshot 2025-12-02 at 09 44 32" src="https://github.com/user-attachments/assets/053b583c-245e-4665-88c5-2b7b311c4f59" />

**Step 2) Internet & NAT Gateways**
* Attached an Internet Gateway (IGW) to the VPC to enable inbound and outbound internet traffic for resources in the public subnets.
* Deployed a NAT Gateway in a Public Subnet and associated it with an **Elastic IP** to provide secure outbound internet access for private subnets.
  
Reasons:
  * Public subnets require direct internet access for components such as the Bastion host and the NAT Gateway.
  * Private subnets should remain isolated from direct inbound internet traffic but still require outbound access for updates and package installations.
  * The NAT Gateway fulfills this requirement by allowing instances in private subnets to initiate outbound connections while remaining non-addressable from the internet.

    <img width="1198" height="356" alt="Screenshot 2025-12-02 at 10 12 45" src="https://github.com/user-attachments/assets/023e3210-431f-4408-8a40-6d0ef4407288" />
<img width="1193" height="338" alt="Screenshot 2025-12-02 at 10 17 17" src="https://github.com/user-attachments/assets/71ca6f4f-6ac3-4587-b842-17eea4912a96" />

**Step 3) Route Tables**
* **Public RT: 0.0.0.0/0 → IGW** (associated with public subnets).
* **Private RT: 0.0.0.0/0 → NAT Gateway** (associated with private subnets).

Reasoning: Route tables dictate traffic flow. This setup ensures:

* Public subnets talk to the internet directly.
* Private subnets reach the internet only via NAT.
  
  <img width="1197" height="334" alt="Screenshot 2025-12-02 at 10 26 15" src="https://github.com/user-attachments/assets/a3130594-0dc5-43d2-8623-e014a327766e" />
<img width="1194" height="309" alt="Screenshot 2025-12-02 at 10 28 20" src="https://github.com/user-attachments/assets/862ce59a-ec5b-449d-92a4-242eb67c2adc" />

**Step 3) Security Groups**
* Bastion-SG:
  Inbound SSH (22) only from my IP.
* Private-SG:
Inbound SSH (22) only from Bastion-SG.

Reasoning:

* Restricting SSH at the source prevents brute force attacks.
* Private instances are not exposed to the internet at all — only reachable via Bastion.

  <img width="1188" height="295" alt="Screenshot 2025-12-02 at 10 56 53" src="https://github.com/user-attachments/assets/53ff8b19-cc3c-484a-9b8d-c26d64dbb81b" />
  <img width="1197" height="328" alt="Screenshot 2025-12-02 at 10 57 50" src="https://github.com/user-attachments/assets/aed9b6a1-c7ee-44a5-b41c-e0628cf9c858" />

**Step 3) EC2 Instances**
* Deployed a Bastion host (Ubuntu) in a Public Subnet with a public IP address.
* Deployed a Private Instance in a Private Subnet (no public IP).

**Validation:**

* Confirmed SSH access to the Bastion host from my trusted IP address.
* Verified that SSH connectivity from the Bastion host to both private instances functions correctly.
* Ensured private instances can access the internet through the NAT Gateway for updates and package installations.

This architecture reflects a secure administrative access pattern: the Bastion host serves as the single controlled entry point, while private instances remain protected from direct external exposure.

<img width="1181" height="590" alt="Screenshot 2025-12-02 at 11 16 16" src="https://github.com/user-attachments/assets/2f5a42f0-9c6b-4241-bee0-88d30d0ef845" />
<img width="1187" height="565" alt="Screenshot 2025-12-02 at 11 17 24" src="https://github.com/user-attachments/assets/928a0e42-a5fb-4113-984d-ba1c007538bb" />
