# AWS Networks: Building a Virtual Private Cloud (VPC)

Welcome to the **AWS Networks VPC Project**! This hands-on project is part of the [NextWork AWS Course](https://learn.nextwork.org/projects/aws-networks-vpc?track=high). You'll create a fully functional Amazon Virtual Private Cloud (VPC) with public and private subnets, routing, and internet connectivity.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Architecture](#architecture)
- [Key Learnings](#key-learnings)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)
- [Resources](#resources)

---

## Project Overview
In this project, you’ll build an isolated network on AWS using a VPC. Key tasks include configuring subnets, attaching an Internet Gateway, and setting up routing. Test your setup by deploying an EC2 instance!

**Objectives:**  
- Define a custom VPC with a `10.0.0.0/16` CIDR block.  
- Create public and private subnets.  
- Enable internet access for public resources.

---

## Prerequisites
- **AWS Account**: Sign up for the [AWS Free Tier](https://aws.amazon.com/free/).  
- **Knowledge**: Basic understanding of IP addressing and CIDR.  
- **Tools**:  
  - AWS Management Console (browser-based).  
  - AWS CLI (optional).

---

## Setup Instructions
Follow these steps to build your VPC:

1. **Log in to AWS Management Console**  
   - Go to **VPC** under "Networking & Content Delivery."

2. **Create a VPC**  
   - Name: `MyNextWorkVPC`  
   - IPv4 CIDR: `10.0.0.0/16`

3. **Create Subnets**  
   - **Public Subnet**: `10.0.1.0/24` (Name: `PublicSubnet1`, AZ: `us-east-1a`)  
   - **Private Subnet**: `10.0.2.0/24` (Name: `PrivateSubnet1`, AZ: `us-east-1a`)

4. **Add an Internet Gateway**  
   - Name: `MyIGW`  
   - Attach to `MyNextWorkVPC`.
  ![image alt](https://github.com/AtulSharmaGeit/AWS-X-Networking-Project/blob/571d887aa16a1edb456467ec605068c38509225d/Images/Internet%20Gateway.png)

5. **Configure Routing**  
   - Create `PublicRouteTable`.  
   - Add route: `0.0.0.0/0` → `MyIGW`.  
   - Associate with `PublicSubnet1`.

6. **Test (Optional)**  
   - Launch an EC2 instance in `PublicSubnet1` with a public IP.  
   - SSH to verify connectivity.

![VPC Creation Screenshot](https://github.com/username/repo/raw/main/vpc-creation.png)  
*Screenshot of the VPC creation page in AWS Console.*

---

## Architecture
The VPC includes:  
- **VPC**: `10.0.0.0/16`  
- **Public Subnet**: `10.0.1.0/24`  
- **Private Subnet**: `10.0.2.0/24`  
- **Internet Gateway**: Links public subnet to the internet.

![VPC Architecture Diagram](https://github.com/username/repo/raw/main/architecture-diagram.png)  
*Diagram of the VPC setup with subnets and routing.*

---

## Key Learnings
- **CIDR**: Allocate IP ranges effectively.  
- **Subnets**: Public vs. private differences.  
- **Routing**: Manage traffic flow in AWS.

---

## Troubleshooting
- **No Internet?** Check route table and subnet association.  
- **EC2 Unreachable?** Ensure public IP and security group allow SSH (port 22).

---

## Next Steps
- Add a NAT Gateway for private subnet internet access.  
- Explore VPC Peering.  
- Enable Flow Logs.

