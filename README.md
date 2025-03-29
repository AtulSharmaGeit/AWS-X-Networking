# AWS X Networking: Build a Virtual Private Cloud (VPC)

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Architecture](#architecture)
- [Key Learnings](#key-learnings)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Project Overview
In this project, you’ll build an isolated network on AWS using a VPC. Key tasks include configuring subnets, attaching an Internet Gateway, and setting up routing. Test your setup by deploying an EC2 instance!

**Objectives:**  
- Define a custom VPC with a `10.0.0.0/16` CIDR block.  
- Create public and private subnets.  
- Enable internet access for public resources in our public subnet.

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
   - Go to **VPC** by typing vpc in search bar.
   - In the left navigation panel, choose **Your VPCs**.
   - Make sure you're on the **Region** that's closest to you. Use the dropdown on the top right hand corner to switch Regions.
   - You'll notice that there is already a VPC in your account! When you created your AWS account, AWS automatically sets up a default VPC for you! This default VPC is why you could launch resources (e.g. EC2 instances) and connect services together from Day 1 of using AWS.

2. **Create a VPC**
   - Choose **Create VPC**.
   - Choose **VPC Only**.
   - Name: `NextWorkVPC`  
   - IPv4 CIDR: `10.0.0.0/16`
   - Select **Create VPC** to finish setting up your VPC.

![image alt](VPC setup)

4. **Create Subnets**
   - In the **VPC Dashboard**, under **Virtual Private Cloud**, choose **Subnets**.
   - Ooo, there are already subnets in here! The default VPC in your account comes with predefined subnets in each **Availability Zone** of a Region, which means you'll see 3 subnets on your page if your Region has 3 Availability Zones.
   - Choose **Create subnet**.
   - **VPC ID**: NextWork VPC
   - **Subnet name**: Public 1
   - **Availability Zone**: Select the first Availability Zone in the list.
   - **IPv4 VPC CIDR block**: 10.0.0.0/16
   - **IPv4 subnet CIDR block**: 10.0.0.0/24
   - Choose **Create subnet**.
   - Select the checkbox next to **Public 1**.
   - In the **Actions** menu, select **Edit subnet settings**.
   - Check the box next to **Enable auto-assign public IPv4 address**.
   - Choose **Save**.
   - 
6. **Add an Internet Gateway**  
   - Name: `MyIGW`  
   - Attach to `MyNextWorkVPC`.
  ![image alt](https://github.com/AtulSharmaGeit/AWS-X-Networking-Project/blob/571d887aa16a1edb456467ec605068c38509225d/Images/Internet%20Gateway.png)

7. **Configure Routing**  
   - Create `PublicRouteTable`.  
   - Add route: `0.0.0.0/0` → `MyIGW`.  
   - Associate with `PublicSubnet1`.

8. **Test (Optional)**  
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

