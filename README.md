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
In this project, you’ll build an isolated network on AWS using a VPC. Key tasks include configuring subnets and attaching an Internet Gateway.

**Objectives:**  
- Define a custom VPC with a `10.0.0.0/16` CIDR block.  
- Create public subnets.  
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

1. **Create a VPC**
   - Log into **AWS Management Console**.
   - In the **AWS Management Console** search field, type VPC.
   - Select **VPC** from the drop down menu.
   - In the left navigation panel, choose **Your VPCs**.
   - Make sure you're on the **Region** that's closest to you. Use the dropdown on the top right hand corner to switch Regions.
   - You'll notice that there is already a VPC in your account! When you created your AWS account, AWS automatically sets up a default VPC for you! This default VPC is why you could launch resources (e.g. EC2 instances) and connect services together from Day 1 of using AWS.
   - Choose **Create VPC**.
   - Choose **VPC Only**.
   - Name: `NextWorkVPC`  
   - IPv4 CIDR: `10.0.0.0/16`
   - Select **Create VPC** to finish setting up your VPC.

![image alt]([VPC setup](https://github.com/AtulSharmaGeit/Networking-Build_A_Virtual_Private_Cloud/blob/6d3e189d4acaa417352ea80a0d71fed40d646454/Images/VPC%20SetUp.png))

2. **Create Subnets**
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

![image alt](subnet panel)

3. **Add an Internet Gateway**  
   - In the left navigation pane, choose **Internet gateways**.
   - Aha! An existing internet gateway. This **default internet gateway** is the reason why you could launch instances with a connection to the internet from the day you've created your AWS account.
   - Choose **Create internet gateway**.
   - **Name tag**: NextWork IG
   - Choose **Create internet gateway**.
   - Select your newly created internet gateway and choose **Actions**, then **Attach to VPC**.
   - Select **NextWork VPC**.
   - Select **Attach internet gateway**.

  ![image alt](https://github.com/AtulSharmaGeit/AWS-X-Networking-Project/blob/571d887aa16a1edb456467ec605068c38509225d/Images/Internet%20Gateway.png)

---

## Architecture
The VPC includes:  
- **VPC**: `10.0.0.0/16`  
- **Public Subnet**: `10.0.0.0/24`   
- **Internet Gateway**: Links public subnet to the internet.

---

## Key Learnings
- **CIDR**: Allocate IP ranges effectively.  

---

## Troubleshooting
- **No Internet?** Check route table and subnet association.  

---

## Next Steps
- Create a route table.
- Create a security group.
- Create a Network ACL (Network Access Control List).

