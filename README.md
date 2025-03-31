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
  
![image alt](https://github.com/AtulSharmaGeit/AWS-X-Networking-Project./blob/f9d41f57a448da0891ed1f0fe0b78afd1ace083d/Images/Screenshot%20(43).png)
   
Ayyy nice! Your subnet is now public because it is connected to the Internet via the internet gateway!

5. **Create a Security Group**  
   - Let's add a security group so that users can access resources in your VPC.
   - In the left navigation pane, choose **Security groups**. Note that this is further down the navigation pane than our other pages so far!
   - **Woah! Why do we already have existing security groups?** - AWS automatically creates a default security group for each new VPC, which allows all traffic between resources within the same VPC. This default rule enables secure communication between resources without exposing them to external threats!
   - Choose **Create security group**.
   - Security group name:  `NextWork Security Group`
   - Description:`A Security Group for the NextWork VPC.`
   - VPC: **NextWork VPC**.
   - Under the **Inbound rules** panel, choose **Add rule** - Inbound rules control the data that can enter the resources in your security group.
   - Type: `HTTP`
   - Source: `Anywhere-IPv4`
   - At the bottom of the screen, choose **Create security group**.

**Wait, what about outbound rules? We only created an inbound rule.** By default, AWS security groups already allow all outbound traffic. So unless you specify otherwise, any resource associated with the security group can access and send data to any IP address - whether it's in your VPC, other VPCs (if you have the right permissions) and on the public internet!

![image alt](44)

6. **Create a Network ACL**  
   - To level up your VPC's security, let's add a network ACL i.e. network access control list.
   - In the left navigation pane, choose **Network ACLs**. Network ACLs are used to set broad traffic rules that apply to an entire subnet. For example, blocking incoming traffic from a particular range of IP addresses or denying all outbound traffic to certain ports.
   - Oooo existing ACLs! - AWS sets up a **default network ACL** for every VPC in your account. This default is designed to allow all traffic to move freely until you decide to customize the rules to fit your needs.
   - Choose the network ACL that's associated with your **Public 1** subnet, and check out the tabs for **Inbound rules** and **Outbound rules**. Just like security groups, network ACLs use inbound and outbound rules to decide which data packets are allowed to enter or leave subnets:
     - **Rule 100 Inbound** allows all inbound traffic into the Public Subnet.
     - **Rule 100 Outbound** allows all traffic out of the Public Subnet.
     - The second line in each ruleset shows an asterisk (*) that acts as a catch-all rule in case traffic does not match any of the earlier rules.
   - Let's recreate this set up ourselves in the console! Your default ACL has everything we need, but it's great practice to set up everything from scratch.
     - Select **Create new network ACL**.
     - Name: `NextWork Network ACL`
     - VPC: **NextWork VPC**
     - Select **Create network ACL**.
   - Uncheck the default network ACL you've selected. The **default network ACLs** that AWS creates allow all inbound and outbound traffic. But for **custom network ACLs** that we create, all inbound and outbound traffic are denied until you add rules about the kind of traffic you'll allow.
   - Select the checkbox next to **NextWork Network ACL**.
   - Select the **Inbound rules** tab.
   - Select **Edit inbound rules**.
   - Select **Add new rule**.
   - Rule number: `100` - In network ACLs, rule numbers decide the order that rules are checked—lower numbers go first. Starting at 100 gives you room to add new rules before it if you need to.
   - Type: **All traffic**.
   - Source: `0.0.0.0/0`
   - Click **Save changes**.
   - Select the **Outbound rules** tab.
   - Select **Edit outbound rules**.
   - Select **Add new rule**.
   - Rule number: `100`
   - Type: **All traffic**.
   - Destination: `0.0.0.0/0`
   - Select the **Subnet associations** tab, which should be right next to the Outbound rules tab. Aha, we're not finished, The subnet associations tab is empty! If your network ACL isn't associated with any subnets, all the rules you define won't affect your VPC's traffic.
   - Under the **Subnet associations** tab, select **Edit subnet associations**.
   - Select your **Public 1** subnet.
   - Select **Save changes**.

  Once you associate your Public 1 subnet with a new ACL, the default ACL that AWS created for you gets replaced.

  ![image alt](45)















  

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

