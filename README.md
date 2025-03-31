# AWS X Networking Project.

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
- Building a Virtual Private Cloud.
- VPC Traffic Flow and Security
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
   - **Name**: `NextWork VPC`
   - **IPv4 CIDR**: `10.0.0.0/16`
   - Select **Create VPC** to finish setting up your VPC.

![image alt](https://github.com/AtulSharmaGeit/Networking-Build_A_Virtual_Private_Cloud/blob/6d3e189d4acaa417352ea80a0d71fed40d646454/Images/VPC%20SetUp.png)

2. **Create Subnets**
   - In the **VPC Dashboard**, under **Virtual Private Cloud**, choose **Subnets**.
   - Ooo, there are already subnets in here! The default VPC in your account comes with predefined subnets in each **Availability Zone** of a Region, which means you'll see 3 subnets on your page if your Region has 3 Availability Zones.
   - Choose **Create subnet**.
   - **VPC ID**: `NextWork VPC`
   - **Subnet name**: `Public 1`
   - **Availability Zone**: Select the first Availability Zone in the list.
   - **IPv4 VPC CIDR block**: `10.0.0.0/16`
   - **IPv4 subnet CIDR block**: `10.0.0.0/24`
   - Choose **Create subnet**.
   - Select the checkbox next to **Public 1**.
   - In the **Actions** menu, select **Edit subnet settings**.
   - Check the box next to **Enable auto-assign public IPv4 address**.
   - Choose **Save**.

![image alt](https://github.com/AtulSharmaGeit/Networking-Build_A_Virtual_Private_Cloud/blob/6d3e189d4acaa417352ea80a0d71fed40d646454/Images/Subnet.png)

3. **Add an Internet Gateway**  
   - In the left navigation pane, choose **Internet gateways**. Aha! An existing internet gateway. This **default internet gateway** is the reason why you could launch instances with a connection to the internet from the day you've created your AWS account.
   - Choose **Create internet gateway**.
   - **Name tag**: `NextWork IG`
   - Choose **Create internet gateway**.
   - Select your newly created internet gateway and choose **Actions**, then **Attach to VPC**.
   - Select **NextWork VPC**.
   - Select **Attach internet gateway**.

Attaching an internet gateway means resources in your VPC can now access the internet. The EC2 instances with public IP addresses become accessible to users, so any application you host on those instances become public too.



  ![image alt](https://github.com/AtulSharmaGeit/AWS-X-Networking-Project/blob/571d887aa16a1edb456467ec605068c38509225d/Images/Internet%20Gateway.png)


4. **Create a Route Table**  
   - Even though you've created an internet gateway and attached it to your VPC, you still have to tell the resource in your public subnet how to get to the internet.
   - In the left navigation pane, choose **Route tables**.
   - Refresh your page. Ooo, two route tables! Why are there two?
   - Let's investigate. Select one of the two route tables and select the **Routes** tab.
   - Uncheck that route table, and switch to the other route table. Select the **Routes** tab again.
   - Aha, the two tables have different routes! As you might guess, one of your route tables was created with your AWS account's default VPC! This is a route table with two routes inside.
     - **Route 0.0.0.0/0 | igw**- directs traffic to the default internet gateway.
     - **Route 172.31.0.0/16 | local** manages internal traffic within the VPC.
   - AWS also created the other route table automatically when you set up **NextWork VPC**. This route table has a single route that allows traffic within the **10.0.0.0/16 CIDR block** to flow within the network. There is no route with an internet gateway as the target! This means there is no route for traffic to leave your VPC.
   - Let's rename your NextWork VPC route table so it's easier to recognise. Make sure you have your NextWork VPC route table selected - this is the route table with a single route to 10.0.0.0/16.
   - Select the pencil icon in the **Name** column of your route table. Enter the name `NextWork route table`.
   - Select **Save**.
   - Select the **Routes** tab.
   - Choose **Edit routes**.
   - Choose **Add route** near the bottom of the page.
   - Destination: `0.0.0.0/0` - 0.0.0.0/0 means all IPv4 addresses! When you set 0.0.0.0/0 as the destination in a route table, you are creating a default route that sends any traffic that doesn't match more specific routes on your route table.
   - Target: **Internet Gateway**.
   - Select **NextWork IG**. 
   - Choose **Save changes**.
   - Choose the **Subnet associations** tab.
   - Under the **Explicit subnet associations** tab, choose **Edit subnet associations**.
   - Select **Public 1**.
   - Choose **Save associations**.
  
![image alt](43)
   
Ayyy nice! Your subnet is now public because it is connected to the Internet via the internet gateway!


 

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

