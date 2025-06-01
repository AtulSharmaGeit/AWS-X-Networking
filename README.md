# AWS X Networking

## Project Overview
In this project, you’ll build an isolated network on AWS using a VPC. Key tasks include configuring subnets and attaching an Internet Gateway.

---

##  Table Of Content
-  [Build a Virtual Private Cloud](#build-a-virtual-private-cloud)
-  [VPC Traffic Flow and Security](#vpc-traffic-flow-and-security)
-  [Creating a Private Subnet ](#creating-a-private-subnet)
-  [Launching VPC Resources](#launching-vpc-resources)
-  [Testing VPC Connectivity](#testing-vpc-connectivity)
-  [VPC Peering](#vpc-peering)
-  [VPC Monitoring with Flow Logs](#vpc-monitoring-with-flow-logs)
-  [Access S3 from a VPC](#access-s3-from-a-vpc)
-  [VPC Endpoints](#vpc-endpoints)
---

## Build a Virtual Private Cloud

We'll dive into the core of AWS networking by creating our very own Virtual Private Cloud (VPC). Setting up and managing a VPC is a vital skill for anyone looking to master cloud infrastructure. We'll learn how to set up and configure a VPC from scratch.

**Step - 1 : Create a VPC**

If we imagine our AWS Region as a country, a Virtual Private Cloud (VPC) is like our own private city inside that country. We can design neighborhoods (known as subnets), traffic rules, and security measures to control how resources, like EC2 instances and databases, connect and work together.

- Log into **AWS Management Console**.
- In the **AWS Management Console** search field, type `VPC`.
- Select **VPC** from the drop down menu.
- In the left navigation panel, choose **Your VPCs**.

**What's the point of having VPCs, why do they matter?** To put it simply - without VPCs, every AWS resource would exist in one giant, open space in the cloud, like a country without cities or districts. Resources would be randomly scattered with no privacy or personal space, so everyone could see and access everyone else's resources.

VPCs are the reason why resources can be made private to us. We also get control over resources in a VPC, so we can organize how they communicate and integrate with each other without the public internet.

- Make sure we're on the **Region** that's closest to us. Use the dropdown on the top right hand corner to switch Regions.

![image alt](Networking-1)

- We'll notice that there is already a VPC in your account!

![image alt](Networking-2)

When we created our AWS account, AWS automatically sets up a **default VPC** for us! This default VPC is why we could launch resources (e.g. EC2 instances) and connect services together from Day 1 of using AWS. If it didn't exist, we would've had to learn how to create a VPC before we can use some of the services that need VPCs to function. This default VPC is a handy starting point, especially for beginners, but we can always create custom VPCs to fit specific requirements e.g. strict security measures.

**Can we create anything in our AWS account without a VPC?** We can use some AWS services like Amazon S3 or AWS Lambda without setting up a VPC. These services are designed to work on the internet without needing a private network setup. However, other services like Amazon EC2 or certain databases need a secure, isolated network to connect with each other and run securely. We would need a VPC in these cases.

- Choose **Create VPC**.
- Choose **VPC Only**.
- **Name** : `NextWork VPC`
- **IPv4 CIDR** : `10.0.0.0/16`

![image alt](Networking-3)

- Select **Create VPC** to finish setting up your VPC.

**Step - 2 : Create Subnets**

We've created our VPC, which is like setting up a brand new city in our AWS Region. Our new city is just a big open space until we organize it into different neighborhoods or areas. Our next step is to divide this large space into subdivisions called **subnets**, so we can start planning where different resources will live and operate.

- In the **VPC Dashboard**, under **Virtual Private Cloud**, choose **Subnets**.
- Ooo, there are already subnets in here!

The default VPC in your account comes with predefined subnets in each **Availability Zone** of a Region, which means we'll see 3 subnets on our page if our Region has 3 Availability Zones.

An AWS Region is made of clusters of data centers dotted around the Region. These clusters are what we call Availability Zones (AZs). When we create a subnet within our VPC, that subnet has to fall under a specific AZ instead of the entire Region! By spreading our resources across multiple AZ in the same region, we're essentially creating backup options. If one AZ faces issues, others can step up to keep our application running without any issues for our users.

- Choose **Create subnet**.
- Configure our subnet settings:
   - **VPC ID**: `NextWork VPC`
   - **Subnet name**: `Public 1`
   - **Availability Zone**: Select the first Availability Zone in the list.
   - **IPv4 VPC CIDR block**: `10.0.0.0/16`
   - **IPv4 subnet CIDR block**: `10.0.0.0/24`
 
![image alt](Networking-4)

- Choose **Create subnet**.
- Select the checkbox next to **Public 1**.
- In the **Actions** menu, select **Edit subnet settings**.

![image alt](Networking-5)

- Check the box next to **Enable auto-assign public IPv4 address**.
- Choose **Save**.

By default, our resources already have private IP addresss, but this only allows internal communication within our VPC.To access the internet or be accessible from the internet, the instance would need a public IP address. When we **enable auto-assign public IPv4 address** for a subnet, any EC2 instance launched in that subnet will instantly get a public IP address so we won't have to create one manually.

**Step - 3 : Create an internet gateway**

Let's attach our VPC with an internet gateway. This is like building a bridge (internet gateway) that links our private city (VPC) to the outside world (the internet), so our resources can communicate beyond our private space.

- In the left navigation pane, choose **Internet gateways**.

Internet gateways are key to making applications available on the internet. By attaching an internet gateway, our instances can access the internet and be accessible to external users.

-   Aha! An existing internet gateway.

An existing internet gateway in our AWS account comes with the default VPC that AWS created for our account. This **default internet gateway** is the reason why we could launch instances with a connection to the internet from the day we've created our AWS account.

- Choose **Create internet gateway**.
- Configure your internet gateway settings:
   - **Name tag** : `NextWork IG`
- Choose **Create internet gateway**.

![image alt](Networking-6)

- Select your newly created internet gateway and choose **Actions**, then **Attach to VPC**.

![image alt](Networking-7)

- Select **NextWork VPC**.
- Select **Attach internet gateway**.

Attaching an internet gateway means resources in our VPC can now access the internet. The EC2 instances with public IP addresses also become accessible to users, so our applications hosted on those servers become public too.

---
##   VPC Traffic Flow and Security

Let's keep diving into the core essentials of building an Amazon Virtual Private Cloud (VPC).

**Step - 1 : Create a Route Table**  

Even though we've created an internet gateway and attached it to our VPC, we still have to tell the resource in our public subnet how to get to the internet. We'll have to set up route tables to direct traffic from your resource to your internet gateway!
   
- In the left navigation pane, choose **Route tables**.

Every subnet in our VPC needs to be linked to a route table, because the table tells our subnet's traffic where to travel to send and receive data. For example, if we have a web server (i.e. an EC2 instance) hosting a website, the EC2 instance's subnet needs a route table that knows how to direct incoming traffic to the website.

When a subnet's route table has a route that directs internet-bound traffic to the internet gateway, the subnet becomes a public subnet. This means our subnet can communicate with the internet.

- Refresh our page.
- Ooo, two route tables! Why are there two?
- Let's investigate. Select one of the two route tables and select the **Routes** tab.
- Uncheck that route table, and switch to the other route table.
- Select the **Routes** tab again.
- Aha, the two tables have different routes!

One of our route tables was created with our AWS account's default VPC! This is a route table with two routes inside:

![image alt](Networking-8)

Let's decode the routes inside this table:
   - **Route 0.0.0.0/0 | igw**- directs traffic to the default internet gateway.
   - **Route 172.31.0.0/16 | local** manages internal traffic within the VPC.
      -   Note that the CIDR block might not be **172.31.0.0/16** exactly for our unique default route table, and that's totally okay!
      -   The important part is that the CIDR block in the route e.g. **172.31.0.0/16** is the entire IP address space allocated to our default VPC. This means this route applies to all traffic that is bound for another resource within the VPC.
      -   The target being **local** means the traffic should be routed to the correct resource inside the VPC.
 
AWS also created the other route table automatically when we set up **NextWork VPC**.
-   This route table has a single route that allows traffic within the **10.0.0.0/16 CIDR block** to flow within the network.
-   There is no route with an internet gateway as the target! This means there is no route for traffic to leave our VPC.
-   **Destination**: The IP address range that traffic wants to reach.
-   **Target**: The road or path that the traffic will have to take to get to its destination.
      -   **igw-xxxxxx**: Means the traffic is routed to the internet via the Internet Gateway.
      -   **local**: Means the traffic stays within the VPC, allowing internal communication between resources.
 
- Let's rename our NextWork VPC route table so it's easier to recognise.
- Make sure we have our NextWork VPC route table selected - this is the route table with a single route to 10.0.0.0/16.
- Select the pencil icon in the **Name** column of our route table.
- Enter the name `NextWork route table`.
- Select **Save**.

![image alt](Networking-9)

- Select the **Routes** tab.
- Choose **Edit routes**.
- Choose **Add route** near the bottom of the page.

![image alt](Networking-10)

- Destination: `0.0.0.0/0`

`0.0.0.0/0` means all IPv4 addresses! When we set 0.0.0.0/0 as the destination in a route table, we are creating a default route that sends any traffic that doesn't match more specific routes on our route table. In our case, since the the only other route has a destination of 10.0.0.0/16, this means all traffic that is not bound for another resource within our VPC is bound for the internet gateway! The internet gateway then forwards this traffic to the internet, allowing our resources to communicate with external networks and users.

Routing rules are evaluated from the most restrictive (i.e. destinations with the bigger number after the slash) through to the least restrictive (which is 0.0.0.0/0 since it refers to all IPv4 addresses). This means our route table will first try to send traffic within the VPC if the destination falls within the VPC's CIDR block, otherwise it is send to the Internet.

- Target: **Internet Gateway**.
- Select **NextWork IG**.

![image alt](Networking-11)

- Choose **Save changes**.
- Choose the **Subnet associations** tab.
- Under the **Explicit subnet associations** tab, choose **Edit subnet associations**.
- Select **Public 1**.
- Choose **Save associations**.

![image alt](Networking-12)

Our subnet is now public because it is connected to the Internet via the internet gateway!

**Step - 2 : Create a security group**

Let's add a security group so that users can access resources in our VPC. Every resource must be associated with a security group. This means security groups don't attach to a VPC or a subnet, they attach to a specific resource within that VPC/subnet. If we don't specify a security group when we launch a resource, it will use the default security group that AWS creates whenever we set up a VPC.

Security groups are responsible for checking who comes in and out. They have strict rules about what kind of traffic can enter or leave the resource based on its IP address, protocols and port numbers.
1.   **Protocols**: Protocols are special rules that help data move across the internet, each designed to send data for a specific kind of task. Here are some protocols we might come across:
      -   **HTTP (Hypertext Transfer Protocol)**: This is the standard protocol for sending web pages over the internet. HTTP carries web page data to our browser. Most website links start with HTTP, for example, http://www.nextwork.org/.
      -   **SMTP (Simple Mail Transfer Protocol)**: SMTP is the standard protocol used for sending emails across the Internet! It acts as a mailman, ensuring that our email reaches the correct inbox without errors.
      -   **FTP (File Transfer Protocol)**: FTP is like a delivery truck that is used for transporting large loads of files between computers on a network. It's a popular tool for uploading and downloading bulky items to and from servers, and developers often use FTP to upload website files directly to a hosting server from their local computers.
      -   **SSH (Secure Shell Protocol)**: SSH is like a secure, private car that allows encrypted communication directly with a server for private tasks such as managing software or configuring settings. In AWS and other cloud environments, SSH is widely used for securely accessing and managing virtual servers like EC2 instances! We'll eventually come across SSH in more advanced projects.
2.   **Port numbers**: Think of port numbers as specific doors on a building where data will enter or exit. Each door is designed for a specific protocol, helping servers (i.e. EC2 instances) direct incoming and outgoing traffic to the right application/process to process that protocol. Without port numbers, an EC2 instance would struggle to manage incoming data correctly. It wouldn't know which application or service each piece of data should be directed to. Some common port numbers are:
      -   **Port 80**: for HTTP protocols delivering web pages.
      -   **Port 25**: for SMTP protocols delivering email.
      -   **Port 21**: for FTP protocols delivering files.

- In the left navigation pane, choose **Security groups**. Note that this is further down the navigation panel than our other pages so far!

**Why do we already have existing security groups?** - AWS automatically creates a default security group for each new VPC, which allows all traffic between resources within the same VPC. This default rule enables secure communication between resources without exposing them to external threats! AWS does not charge for creating and maintaining security groups.

![image alt](Networking-13)

- Choose **Create security group**.

![image alt](Networking-14)

- Security group name:  `NextWork Security Group`
- Description:`A Security Group for the NextWork VPC.`
- VPC: **NextWork VPC**.
- Under the **Inbound rules** panel, choose **Add rule**.

**Inbound rules** control the data that can enter the resources in our security group, while **outbound rules** control that data that our resources can send out. In this scenario, setting up inbound rules is important for allowing users to access our public website, while outbound rules help manage how our server interacts with other parts of the internet.

- Type: `HTTP`
- Source: `Anywhere-IPv4`

![image alt](Networking-15)

The yellow popup in your image is a warning from AWS. AWS is concerned that the security rule we've just set, i.e. setting the source to `0.0.0.0/0`, allows any IP address to access our resource. This wide-open access can be risky, exposing our server to potential threats from any location. AWS suggests tightening our security by restricting access to only known IP addresses, which would limit who can reach our server and help keep it safe from unwanted or malicious traffic.

But, setting the inbound rule to allow HTTP traffic from `0.0.0.0/0` (meaning any IP address) is typical and necessary for public subnets, since this setting makes sure that anyone on the internet can access our public resources.

- At the bottom of the screen, choose **Create security group**.

**What about outbound rules?** **We only created an inbound rule**. By default, AWS security groups already allow all outbound traffic. So unless we specify otherwise, any resource associated with the security group can access and send data to any IP address - whether it's in our VPC, other VPCs (if we have the right permissions) and on the public internet!

![image alt](Networking-16)

**Step - 3 : Create a Network ACL**

Nice, that's our traffic flow (route table) and basic security (security groups) sorted for our VPC! To level up our VPC's security, let's add a network ACL i.e. network access control list.

- In the left navigation pane, choose **Network ACLs**.

**Network ACLs** are used to set broad traffic rules that apply to an entire **subnet**. For example, blocking incoming traffic from a particular range of IP addresses or denying all outbound traffic to certain ports. We can set broad restrictions at the subnet level with ACLs, and more specific limits at the resource level through security groups. This dual layer takes security to the next level as traffic must pass through multiple checks, which reduces the chances of unwanted access.


- Oooo existing ACLs!

![image alt](Networking-17)

AWS sets up a **default network ACL** for every VPC in our account. This default is designed to allow all traffic to move freely until we decide to customize the rules to fit our needs.

- Choose the network ACL that's associated with your **Public 1** subnet, and check out the tabs for **Inbound rules** and **Outbound rules**.

![image alt](Networking-18)

![image alt](Networking-18')

Network ACLs use inbound and outbound rules to decide which data packets are allowed to enter or leave subnets:
   - **Rule 100 Inbound** allows all inbound traffic into the Public Subnet.
   - **Rule 100 Outbound** allows all traffic out of the Public Subnet.
   - The second line in each ruleset shows an asterisk (*) that acts as a catch-all rule in case traffic does not match any of the earlier rules. In our case, since Rule 100 already allows all traffic, the asterisk rule won't actually come into play.

This means default network ACLs allow **all** inbound and outbound traffic, unless customized.

To solidify our learnings, let's recreate this set up ourselves in the console! Our default ACL has everything we need, but it's great practice to set up everything from scratch.
- Select **Create new network ACL**.
- Name: `NextWork Network ACL`
- VPC: **NextWork VPC**
- Select **Create network ACL**.

![image alt](Networking-19)

- Uncheck the default network ACL we've selected.
- Select the checkbox next to **NextWork Network ACL**.
- Select the **Inbound rules** tab.

![image alt](Networking-20)

**There is only one rule that denies all traffic!** The **default** network ACLs that AWS creates allow all inbound and outbound traffic. But for custom network ACLs that we create, all inbound and outbound traffic are denied until we add rules about the kind of traffic we'll allow.

- Select **Edit inbound rules**.
- Select **Add new rule**.
- Rule number: `100`

In network ACLs, rule numbers decide the order that rules are checked—lower numbers go first. Starting at `100` gives us room to add new rules before it if we need to. For example, if we later want to block a specific protocol, we can slip in a rule with a lower number to make sure it gets checked first. Otherwise, it's standard to start from 100 and add rules with higher numbers.

- Type: **All traffic**.

**Why did the Protocol and Port range fields grey out?** When we selected "All traffic" for the traffic type, this choice implies that our rule will apply to all protocols and port ranges, so there's no need to specify them anymore.

- Source: `0.0.0.0/0`

![image alt](Networking-21)

- Click **Save changes**.

- Select the **Outbound rules** tab.
- Select **Edit outbound rules**.
- Select **Add new rule**.
- Rule number: `100`
- Type: **All traffic**.
- Destination: `0.0.0.0/0`

- Select the **Subnet associations** tab, which should be right next to the Outbound rules tab.
- Aha, we're not finished, The subnet associations tab is empty!

Just like attaching internet gateways to VPCs, we need to associate network ACLs with subnets. If our network ACL isn't associated with any subnets, all the rules we define won't affect our VPC's traffic. Our network ACL isn't actually securing our network!

- Under the **Subnet associations** tab, select **Edit subnet associations**.

![image alt](Networking-22)

- Select your **Public 1** subnet.
- Select **Save changes**.

Only one ACL can be associated with a subnet at a time. Once we associate our Public 1 subnet with a new ACL, the default ACL that AWS created for us gets replaced.

---
##  Creating a Private Subnet 

Let's set up our very first private subnet and learn its differences from a public subnet along the way.

**Step - 1 : Set up a private subnet**
-   Still in our VPC console, select the **Subnets** tab again.
-   Select **Create subnet**.
-   For the **VPC ID**, select **NextWork VPC**.
-   Set the **Subnet name** as `NextWork Private Subnet`
-   For the subnet's **Availability Zone**, use the second AZ on the dropdown (not the first!)
-   The IPv4 VPC CIDR block is already pre-set to 10.0.0.0/16
-   For the IPv4 subnet CIDR block, what will happen if this new subnet has the exact same CIDR block as our Public subnet?
-   Let's investigate by entering `10.0.0.0/24` to start with.
-   Select **Create subnet**.
-   Aha! An error message.

![image alt](Networking-23)

The error message says: **CIDR Address overlaps with existing Subnet CIDR: 10.0.0.0/24**. This error pops up when the CIDR block we're trying to assign to a new subnet is already being used by another subnet in the same VPC. Every subnet in a VPC must have a unique CIDR block so traffic is routed correctly and there are no conflicts.

-   How would we fix this error?
-   Let's assign our new subnet a CIDR block that doesn't overlap with **Public 1**.
-   Replace the IPv4 subnet CIDR block with `10.0.1.0/24`.

By setting the CIDR block of our new private subnet to `10.0.1.0/24`, we allocate a specific range of IP addresses from `10.0.1.0` to `10.0.1.255`.

The Public 1 subnet is defined with the CIDR block `10.0.0.0/24`, which is a different range of IP addresses from `10.0.0.0` to `10.0.0.255`.

![image alt](Networking-24)

-   Select **Create subnet**.
-   Success!
-   To tidy up our subnets' naming conventions, let's retitle our **Public 1** subnet to `NextWork Public Subnet`.

![image alt](Networking-25)

**Step - 2 : Set up a private route table**

Like our public subnet, a private subnet also needs to be associated with a route table.
  
-   Head to the Route tables page in our console.

Back when we set up **NextWork route table**, we renamed the default route table that AWS automatically created with our VPC. This means **NextWork route table** is the default route table for our entire VPC. Subnets that aren't associated with another route table automatically use **NextWork route table**. But **NextWork route table** has a route to an internet gateway, and having that route would make our subnet public! It's time to set up a new route table for our private subnet.

-   Select **Create route table**.
-   Name our new route table `NextWork Private Route Table`.
-   Under **VPC**, select **NextWork VPC**.
-   Select **Create route table**.

![image alt](Networking-26)

-   Nice! With our private route table set up, let's make sure it can only direct traffic to another internal resource (instead of the public internet).
-   Select **NextWork Private Route Table**.
-   Check the **Routes** tab - does it only have one default route with a **local** target?

![image alt](Networking-27)

-   Switch tabs to **Subnet associations**.
-   Select **Edit subnet associations** under the **Explicit subnet associations** tab.
-   Select the checkbox next to **NextWork Private Subnet**.
-   Select **Save associations**.

![image alt](Networking-28)

-   To tidy up your Route tables' naming conventions, let's also retitle **NextWork route table** to `NextWork Public Route Table`.

![image alt](Networking-29)

**Step - 3 : Set up a private network ACL**
-   Select **Network ACLs** from the left hand navigation panel.
-   Select the checkbox next to the default ACL for our VPC.
      -   Note that this is NOT **NextWork Network ACL** - our default ACL isn't named!
-   Select the **Inbound rules** and **Outbound rules** tabs.

![image alt](Networking-30)

We might recall that there is a default network ACL set up for every VPC. This default network ACL is associated with our private subnet, since we haven't set up an explicit association between our private subnet and another network ACL. A VPC's default network ACL allows **all** traffic, which exposes our private subnet to unrestricted access from the internet or other untrusted networks. Let's set up a new network ACL that restricts traffic and protects our private subnet!

Removing a route to the internet gateway does prevent direct internet access to and from the subnet. However, relying only on route table configurations might not be great for security. If any part of our VPC (e.g. our public subnet) gets compromised, the attacker could take advantage of our permissive ACL setup to access or attack the resources in our private subnet!

-   So while the default ACL is very convenient in allowing all traffic types, we'll create a new custom network ACL to keep our private subnet safe.
-   Select **Create network ACL** on the top right.
-   For the name, enter `NextWork Private NACL`.
-   Select **NextWork VPC**.
-   Select **Create network ACL**.
-   Done!
-   Observe the **Inbound rules** and **Outbound rules** tabs for our private network ACL.

Remember that **custom network ACLs** start with denying all inbound and outbound traffic!

![image alt](Networking-31)

![image alt](Networking-32)

-   Switch tabs to **Subnet associations**.
-   Select **Edit subnet associations**.
-   Select your private subnet.
-   Select **Save changes**.

![image alt](Networking-33)

-   To tidy up your Network ACLs' naming conventions, let's also rename **NextWork Network ACL** to `NextWork Public NACL`.

![image alt](Networking-34)

All done! Your private subnet is set up and ready to go.

---
##   Launching VPC Resources

We're going to level up by launching resources into our VPC - we'll also learn some handy EC2 concepts along the way!

**Step - 1 : Launch a Public EC2 Instance**

Let's kick things off by launching an EC2 instance in our public subnet.
-   Head to the **EC2 console** - search for `EC2` in the search bar at the top of screen.
-   Select **Instances** at the left hand navigation bar.
-   Select **Launch instances**.
-   Since our first EC2 instance will be launched in the public subnet, let's name it `NextWork Public Server`
-   For the **Amazon Machine Image**, select **Amazon Linux 2023 AMI**.
-   For the **Instance type**, select **t2.micro**.

![image alt](Networking-35)

An **AMI** is a template or blueprint used to create EC2 instances and contains the operating system along with the applications needed to launch the instance. **Instance types** cover the 'hardware' components - CPU power, memory size, storage space and more!

-   For the **Key pair (login)** panel, select **Create new key pair**.

**Key pairs** help engineers directly access their virtual machines, like EC2 instances. It consist of two cryptographic keys: one private and one public. The public key is installed on the virtual machine, and the private key remains with the user. When we attempt to connect, the machine uses the public key to create an encrypted challenge that can only be decrypted with the private key. Key pairs make sure that access to our EC2 instances is secure and authenticated.

-   For the **Key pair name**, use `NextWork key pair`.
-   Keep the **Key pair type** as **RSA**, and the **Private key file format** as **.pem**

![image alt](Networking-36)

**Key pair type** determines the algorithm used for generating the key pair's cryptographic keys.

We use **RSA (Rivest-Shamir-Adleman)**, which is one of the most common cryptographic algorithms used due to its strength and security. RSA is widely supported and trusted for creating digital signatures and encrypting data.

**.pem** format, which stands for Privacy Enhanced Mail, started off as a way to secure emails but has since become the go-to format for managing cryptographic keys because it is supported by many different types of servers e.g. EC2 instances!

-   Select **Create key pair**.

The file that started downloading is the private key (.pem) i.e. our half of the new key pair! It's usually very important to save it securely. Losing this file means losing the ability to securely access our instances, and it cannot be downloaded again from AWS.

-   At the **Network settings** panel, select **Edit** at the right hand corner.

By default, all resources are launched into the default VPC that AWS has set up for our account. We need to tell AWS that we actually want to launch this instance in NextWork VPC and the NextWork Public Subnet!

-   Select **NextWork VPC** from the drop-down in the VPC list.
-   Select our public subnet.
-   For the **Firewall (security groups)**, we've already created the security group for our public subnet's resources. Choose **Select existing security group**.
-   Select **NextWork Public Security Group**.

![image alt](Networking-37)

-   Select **Launch instance**.
-   Click into our instance once it's successfully launched.

-   Head back to the **Instances** page.
-   Select the checkbox next to your instance, and a **Details** panel pops up!
-   Switch the tab to **Networking**.
-   Notice how our public server has a Public IPv4 address, a subnet it's associated with, an Availability zone it's launched in, and a VPC ID that links it with NextWork VPC too.

![image alt](Networking-38)

**Availability Zone** is the specific area within our AWS Region that our instance is hosted.

**VPC ID** identifies that within the AWS Region we're using, the Public Server belongs in our NextWork VPC.

**NextWork Public Subnet** determines the range of IP addresses within NextWork VPC that can be assigned to our EC2 instance. Because this subnet has a route to an internet gateway, our VPC has opened up communication between all resources in the subnet and the internet.

**Public IPv4 address** is the external IP address assigned to our EC2 instance. This address is globally unique, so no other server has the same public IPv4 address on the internet! Having a public IPv4 address means our instance can communicate with the internet and be accessible from outside our private AWS network.

**Step - 2 : Launch a Private EC2 Instance**

Let's follow similar steps to launch our **private** server!

-   Select **Launch instances** again.
-   Name: `NextWork Private Server`
-   Amazon Machine Image (AMI): **Amazon Linux 2023 AMI**
-   Instance type: **t2.micro**
-   Key pair: **NextWork key pair**

**Can we use the same key pair between multiple instances?** Yup, we can use the same key pair for more than one EC2 instance! This means we can use the same private key (i.e. the .pem file) to log into any of our instances using this key pair, making it easier to manage. From a security point of view, anyone with that key can access all the instances it's connected to - making it even more important to keep our private key safe.

-   At the **Network settings** panel, select **Edit** at the right hand corner.
-   Network: **NextWork VPC**
-   Subnet: **NextWork Private Subnet**
-   **Firewall (security groups)**: we said we'd use an alternative way to set up security groups for our private subnet's resources, and here we are!
      -   Select **Create security group**.
      -   For **Security group name**, let's use `NextWork Private Security Group`
      -   For Description, we'll replace the default value with `Security group for NextWork Private Subnet`.
      -   Notice the default Inbound Security Groups, the **Type** is set to **ssh**.
 
SSH, or Secure Shell, is the protocol we use for this secure access to a remote machine. When we connect to the instance, SSH verifies we possess the correct private key corresponding to the public key on the server, ensuring only authorized users can access the instance.

In terms of network communication, SSH is also as a type of network traffic. Once SSH has established a secure connection between us and the EC2 instance, all data transmitted (including our commands and the responses from the instance) is encrypted. This encryption makes SSH an ideal method for securely exchanging confidential data e.g. login credentials!

![image alt](Networking-39)

This popup says "Rules with source of `0.0.0.0/0` allow all IP addresses to access our instance. We recommend setting security group rules to allow access from known IP addresses only."

AWS is concerned that the default security rule, i.e. with the source being `0.0.0.0/0`, allows any IP address to access our resource using SSH. We were okay with allowing HTTP traffic from `0.0.0.0/0` for our public subnet, but the private subnet is a different story!

-   Change the **Source type** from **Anywhere** to **Custom**.
-   In the **Source** drop down, scroll down and select **NextWork Public Security Group**.

Choosing the **NextWork Public Security Group** as the source means only resources that are part of the NextWork Public Security Group can communicate with our instance. This restricts access to a much smaller group of trusted resources, rather than allowing potentially any IP address on the internet (0.0.0.0/0) to access our instance. A great move for securing a private subnet!

![image alt](Networking-40)

-   Select **Launch instance**.

**Step - 3 : Delete Our Resources**

Keeping track of our resources, and deleting them at the end, is absolutely a skill that will help us reduce waste in our account.

-   In your **VPC console**, select the checkbox next to **NextWork VPC**.
-   Select the **Actions** dropdown.
-   Select **Delete VPC**.
-   If stopped from deleting our VPC because **network interfaces** are still attached to our VPC - delete all the attached network interfaces first!

**Network interfaces** get created automatically when we launch an EC2 instance. Think of them as a component that attaches to an EC2 instance on one end and our VPC on another - so that our EC2 instance is connected to our network and can send and receive data! Network interfaces are usually deleted automatically with our EC2 instance, but on some occassions it'd be faster to delete them manually.

-   Type `delete` at the bottom of the pop up panel.
-   Select **Delete**.

![image alt](Networking-40')

Now visit each of the pages below! **Refresh** our the page before checking if the resource we created is still in our account. They should be automatically deleted with our VPC, but it's always a good idea to check anyway:
      1.   Subnets
      2.   Route tables
      3.   Internet gateways
      4.   Network ACLs
      5.   Security groups

-   In our **EC2 console**, select the checkboxes next to both instances.
-   Select the **Instance state** dropdown, and select **Terminate instance**.
-   Select **Terminate**.

---
##   Testing VPC Connectivity

**Step - 1 : Set up our VPC basics**

Let's try a new way to create our entire VPC setup (it's a time saver)
-   [Log in to our AWS Account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26state%3DhashArgsFromTB_ap-southeast-2_fffdf5be4bb1a27e&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=m-aiqeB2UZeXTGXNyugMP8L64zd_AGUxJl4HLnA-X1o&code_challenge_method=SHA-256).
-   Head back to your **VPC** console.
-   From the left hand navigation bar, select **Your VPCs**.
-   Select **Create VPC**.
-   We previously stuck to creating a VPC only, but this time select **VPC and more**.
-   Woah! A visual flow diagram pops up that shows us other VPC resources. This is called a **VPC resource map**!

![image alt](Networking-41)

With VPC resource map, we can quickly understand the architectural layout of a VPC, like the number of subnets, which subnets are associated with which route table, and which route tables have routes to an internet gateway.

-   Now with this handy VPC resource map, we get to see that selecting the **VPC and more** option will also help us create VPC resources in the exact same page. No more jumping between pages in our VPC console!


-   Scroll to view the entire VPC resource map, and take note of the resources listed.
-   See if we can answer these questions:
      1.   How many subnets are being created in our VPC?
      2.   How many of those subnets are public, how many are private?
      3.   How many route tables are being created in our VPC? Why?
      4.   Is there an internet gateway being created?
 
![image alt](Networking-42)

Here are the answers!
-   There are **4 subnets** being created.
-   2 of those subnets are public, 2 are private.
-   There are **3 route tables** being created - 1 for both public subnets to share, and 1 for each private subnet.
-   There is an internet gateway being created - we can spot it under the fourth panel, **Network connections**!

**Resource maps** help us understand how different components in our setup are connected and interact with each other. This makes it easier to design, manage, and troubleshoot our architecture because we can see everything at a glance, rather than sifting through lists and configurations. Our resource map looks straightforward since we have a nice and simple VPC architecture, but imagine how useful this tool would be for complex VPC setups with many subnets!

-   Scroll back to the left hand side of the screen to see the VPC's set up.
-   Under **Name tag auto-generation**, enter `Nextwork`

**Name tag auto-generation** is a nifty feature that tags all our VPC resources with a name based on what we enter. If we type in `nextwork`, all our resources will have that in their name tags, making it super easy to track and manage everything linked to our VPC. We'll see this in action soon!

-   The VPC's **IPv4 CIDR block** is already pre-filled to `10.0.0.0/16`.

We can have multiple VPCs with the same IPv4 CIDR block in the same AWS region and account. AWS VPCs are isolated from each other by default, so there won't be any IP conflicts unless we explicitly connect them using VPC peering. Bottom line, it's possible for our new VPC to share the same CIDR block as an existing one, but this set up will mean our overlapping VPCs can't talk to each other directly. That's why it'd be best practice to have completely unique CIDR blocks for each VPC in our account!

-   For **IPv6 CIDR block**, we'll leave in the default option of **No IPv6 CIDR block**.

IPv6 is the latest version of IP addresses with a lot more IP addresses than IPv4. If we’re mostly working with IPv4, we can skip IPv6 for now to keep things simple. We usually don't need IPv6 unless our applications or users specifically need it.

-   For **Tenancy**, we'll keep the selection of **Default**.

Tenancy in AWS refers to the type of hardware our instances run on. We have two main options:
      1.   **Default**: Our instances share hardware with other AWS customers. This is the standard option and is cost-effective because we’re sharing resources.
      2.   **Dedicated**: Our instances run on hardware that's dedicated to us only. For example, imagine a healthcare company that needs to ensure the highest level of security for patient data. They might choose dedicated tenancy to make sure their servers are completely isolated from other customers, helping them meet compliance standards and keep sensitive information secure. Dedicated does come at a higher cost!

-   For **Number of Availability Zones (AZs)**, we'll leave the default value of `2` for now and come back to this soon.
-   Expand the **Customize AZs** arrow. Ooo we can even configure which two Availability Zones we'd like to set up for this VPC!

![image alt](Networking-43)

-   Next, notice that **Number of public subnets** only gives us two options - `0` or `2`.

This is **AWS' best practice** advice at work! When we pick 2 Availability Zones, the wizard makes sure we have a public subnet in each one. This way, our public resources stay up even if one of the two Availaiblity Zones goes down. This setup is called **redundancy** (having backups in different places) and **high availability** (making sure our resources are always accessible). Just one public subnet wouldn’t offer this kind of reliability, so AWS doesn't let us create just one!

**VPC wizard** limits us to two public subnets to keep things straightforward. If we need more, we can always add them manually later - we can have up to 200 subnets total in a VPC! After all, this VPC set up page’s aim is to get us up and running quickly without overwhelming us with options.

-   Similar to this, **Number of private subnets** only gives you three options - `0`, `2` or `4`.

**How is it that we can create up to 4 private subnets, but only 2 public subnets?** AWS's best practice advice is at work again! Having more private subnets can help with organizing our resources and isolating them for security purposes, whereas public subnets are limited to ensure manageable exposure to the internet.

-   Try selecting the option for 4 private subnets, and watch our resource map update itself!

![image alt](Networking-44)

**Why do we have 5 route tables?** In a VPC, **public subnets** usually share a single route table because they all need to route traffic to the internet through the same internet gateway. This simplifies management since all public subnets follow the same rules for internet access. For **private subnets**, each one often has its own route table to control and customize traffic routing more precisely. This allows for different routing rules and security controls for each private subnet. 

![image alt](Networking-45)

-   PAUSE - what will happen if we change the number of Availability Zones from `2` to `1`?
-   Make our prediction on how this resource map will look differently...
-   Scroll back to the Availability Zone field, and change the selection from `2` to `1`!

![image alt](Networking-46)

Changing the number of availability zones updates the number of subnets and route tables to keep things balanced and reliable. Fewer zones mean fewer resources are needed to maintain that balance and reliability.

-   Change the **Number of private subnets** from `2` to `1`. Now we have just two subnets total - one public and one private subnet!
-   Update our public and private subnets' CIDR blocks:
      -   Update our public subnet CIDR block to `10.0.0.0/24`
      -   Update our private subnet CIDR block to `10.0.1.0/24`
 
**Why do the subnets' default CIDR blocks finish in /20 by default?** The default /20 subnet provides 4,096 IP addresses, which is a good middle ground for most use cases. Typically the norm is to use **/8 /16 /24 /32**. The /20 size offers a balance between too few and too many IP addresses, making it useful for most network setups without overwhelming us with an excessive number of IPs.

![image alt](Networking-47)

-   Next, for the **NAT gateways ($)** option, make sure we've selected **None**. As the dollar sign suggests, NAT gateways cost money!

![image alt](Networking-48)

**NAT gateways** let instances in private subnets access the internet for updates and patches, while blocking inbound traffic. For example, our private server in our private subnet might need to download security updates. By using a NAT gateway, the server can access these updates securely while remaining protected from external threats!

On the other hand, **internet gateways** let instances in public subnets communicate with the internet both ways i.e. both inbound and outbound traffic.

**Why would we use a NAT gateway? Couldn't we just set up an internet gateway with no public inbound traffic but allow outbound traffic?** Instances in public subnets using an internet gateway still need public IP addresses to communicate with the internet. Assigning public IP addresses to our instances makes them accessible from the internet, increasing the attack surface. Even with strict security group rules, there's always a risk of misconfiguration or vulnerabilities being exploited.

Private subnets are meant to keep our instances isolated from the public internet, so using public IP addresses for instances in private subnets would not be ideal. That's where NAT gateways come in! Instances in private subnets using a NAT gateway do not need public IP addresses. The NAT gateway handles a translation to a public IP, keeping our instances' private IPs hidden.

-   Next, for the **VPC endpoints** option, select **None**.

![image alt](Networking-49)

Normally, to access some AWS services like S3 from our VPC, our traffic would go out to the public internet. But, VPC endpoints let us connect our VPC privately to AWS services without using the public internet. This means our data stays within the AWS network, which can improve security and reduce data transfer costs.

There are many types of VPC endpoints, and the **S3 Gateway endpoint** is the most common and useful one - many applications need to access S3 for storing or retrieving data after all! The endpoints for other AWS services can be added later, but this setup tool simplifies the initial setup by focusing on S3.

-   We can leave the **DNS options** checked.

When we enable **DNS hostnames**, our EC2 instances can have human-readable names, like denzelnextwork.compute-1.amazonaws.com, instead of just numeric IP addresses. This makes it simpler to identify and connect to our instances.

When we enable **DNS resolution**, AWS takes care of translating these hostnames to their corresponding IP addresses so that network requests can find the correct instance. This is particularly useful in environments where IP addresses might change - hostnames can stay consistent, so references to our resource would still point to the right thing.

-   Select **Create VPC**.
-   Super satisfying to see this loading bar of our VPC and its resources getting created!

![image alt](Networking-50)

-   Select **View VPC**.
-   Select the **Resource map** tab.

![image alt](Networking-51)

-   Note how name tag auto-generation, which we enabled in the set up page, is at work now - all of our VPC's resources have `nextwork` at the start of the name!
-   Within our resource map, click on our **public subnet**.
-   Oooo, now we get to see how our public subnet is connected to a public route table and our internet gateway.

![image alt](Networking-52)

**Validate and rename your resources**

Let's tidy up our resources' names! Take 3 minutes to rename our:
-  **VPC**
      -   Select **Your VPCs** from the left hand navigation panel.
      -   Hover over **NextWork-vpc** and select the pencil icon.
      -   Rename our VPC to `NextWork VPC`
      -   Select **Save**.

-   **Subnets**
      -   Select **Subnets** from the left hand navigation panel.
      -   Rename our public subnet i.e. **NextWork-subnet-public1-xxx** to `NextWork Public Subnet`.
      -   Rename our private subnet i.e. **NextWork-subnet-private1-xxx** to `NextWork Private Subnet`.

-   **Route tables**
      -   Select **Route tables** from the left hand navigation panel.
      -   Rename our public route table i.e. **NextWork-rtb-public** to `NextWork Public Route Table`.
      -   Rename our private route table i.e. **NextWork-rtb-private1-xxx** to `NextWork Private Route Table`.
 
-   **Internet gateway**
      -   Select **Internet gateways** from the left hand navigation panel.
      -   Rename our internet gateway i.e. **NextWork-igw** to `NextWork IG`.
 
-   **Network ACLs**
      -   Select **Network ACLs** from the left hand navigation panel.
      -   Select **Create network ACL** on the top right.
      -   For the name, enter `NextWork Private NACL`.
      -   Select **NextWork VPC**.
      -   Select **Create network ACL**.
      -   Select the checkbox next to **NextWork Private NACL**.
      -   Switch tabs to **Subnet associations**.
      -   Select **Edit subnet associations**.
      -   Select our **private** subnet.
      -   Select **Save changes**.
      -   To tidy up our network ACLs' naming conventions, let's also rename our VPC's default NACL to `NextWork Public NACL`
🧠 Hint: the other network ACL with just 1 subnet associated is our VPC's default network ACLs!
      -   Observe the **Inbound rules** and **Outbound rules** tabs for our private network ACL.
 
**Why are they both denying all traffic?**  Remember that custom network ACLs start with denying all inbound and outbound traffic! We'll leave these settings for now.

-   **Security groups**
      -   Select **Security groups** from the left hand navigation panel.
      -   Yay default security groups have already been set up for us! Their names are **default** and... **default**.
      -   Select either one of the two and observe its inbound rules.
      -   Hmmm these **default** security groups don't quite have the same inbound rules as the ones we used to set up ourselves.
      -   Let's create new security groups instead of using these default ones.
      -   Choose **Create security group**.
      -   Security group name:  `NextWork Public Security Group`.
      -   Description: `A Security Group for the NextWork VPC Public Server`.
      -   VPC: **NextWork VPC**.
      -   Under the **Inbound** rules panel, choose **Add rule**.
      -   Type: `HTTP`
      -   Source: `Anywhere-IPv4`
      -   At the bottom of the screen, choose **Create security group**.
 
**Launch a new EC2 instance in NextWork Public Subnet**
-   Head to the **EC2 console** - search for `EC2` in the search bar at the top of screen.
-   Select **Instances** at the left hand navigation bar.
-   Select **Launch instances**.
-   Since our first EC2 instance will be launched in the public subnet, let's name it `NextWork Public Server`
-   For the **Amazon Machine Image**, select **Amazon Linux 2023 AMI**.
-   For the **Instance type**, select **t2.micro**.
-   For the **Key pair (login)** panel, select **NextWork key pair**.
      -   NOTE: If we've never created **NextWork key pair**, or we've deleted it, all we have to do is:
            -   Select **Create new key pair**.
            -   For the **Key pair name**, use `NextWork key pair`.
            -   Keep the **Key pair type** as **RSA**.
            -   Keep the **Private key file format** as **.pem**.
            -   Select **Create key pair**.
       
-   At the **Network settings** panel, select **Edit** at the right hand corner.
-   Select **NextWork VPC** from the drop-down in the VPC list.
-   Select our public subnet.
-   Update the **Auto-assign public IP** setting to **Enable**.
-   For the **Firewall (security groups)** setting, we've already created a security group - let's use that!
-   Choose **Select existing security group**.
-   Select **NextWork Public Security Group**.
-   Select **Launch instance**.
-   Click into our instance once it's successfully launched.
-   Select the checkbox next to our instance, and a **Details** panel pops up!
-   Switch the tab to **Networking**.
-   Notice how NextWork Public Server has a Public IPv4 address, a subnet, an Availability zone, and a VPC ID.

![image alt](Networking-53)

**Launch a new EC2 instance in NextWork Private Subnet**
-   Select **Launch instances** again.
-   Name: `NextWork Private Server`.
-   Amazon Machine Image (AMI): **Amazon Linux 2023 AMI**
-   Instance type: **t2.micro**
-   Key pair: **NextWork key pair**
-   At the **Network settings** panel, select **Edit**.
-   Network: **NextWork VPC**
-   Subnet: **NextWork Private Subnet**
-   **Firewall (security groups)**: Select **Create security group**.
      -   For **Security group name**, let's use `NextWork Private Security Group`.
      -   For **Description**, we'll replace the default value with `Security group for NextWork Private Server`.
      -   Notice that under the **Inbound Security Group Rules** section, there is a rule with **Type** set to **ssh**.
-   Change the **Source type** from **Anywhere** to **Custom**.
-   In the **Source** drop down, scroll down and select **NextWork Public Security Group**.

Choosing the **NextWork Public Security Group** as the source means only resources that are part of the NextWork Public Security Group can communicate with your instance.

![image alt](Networking-54)

-   Select **Launch instance**.

**Step - 2 : Connect to NextWork Public Server**

Now that we've set up a public server and a private server, let's test whether we can get our EC2 instances to talk to each other despite being in different subnets.
-   Still in your **EC2** console, select **Instances** from the left hand navigation panel.
-   Select the checkbox next to **NextWork Public Server**.
-   Select **Connect**.

Connectivity testing often involves fine-tuning our security group settings and network ACLs to make sure the right traffic can flow in and out of our instances as we'd expect!

-   Keep all of the default settings.

![image alt](Networking-55)

**EC2 Instance Connect** is an alternative way to use SSH - Instance Connect lets us securely connect to our EC2 instances directly using the AWS Management Console. We're still using SSH, but with all the key management handling it for us. This takes away a lot of the complexity of setting up SSH.

-   Select Connect.
-   Oh no! We've failed to connect to our instance?

![image alt](Networking-56)

Let's investigate what happened by reviewing our security settings.
-   Head back to our **VPC console**.
-   Select **Subnets** from the left hand navigation panel.
-   Select the checkbox next to **NextWork Public Subnet**.
-   Hmm let's take a look! Investigate the **Route table** and **Network ACL** tabs - what do we see?

![image alt](Networking-57)

![image alt](Networking-58)

Everything looks good! Our Network ACL allows all traffic in and out, and our route table is correctly setting up a route to the Internet Gateway.

-   Hmmm that leaves one more thing to investigate...
-   Head into the **Security groups** page from the left hand navigation bar.
-   Select the checkbox next to **NextWork Public Security Group**.
-   Select the **Inbound rules** tab.
-   Aha! Mystery solved.

![image alt](Networking-59)

Security group associated with NextWork Public Server lets in all inbound HTTP traffic, but this is not how we're trying to access our Public Server! We're trying to access NextWork Public Server using SSH through EC2 Instance Connect, which is a different traffic type.

-   In the **Inbound rules** tab, select **Edit inbound rules**.
-   Select **Add rule**.
-   For our new rule, configure the **Type** as **SSH**.
-   Then, under **Source type**, select **Anywhere-IPv4**.

In this step, we are updating NextWork Public Server's security group so it can let in SSH traffic. Choosing **Anywhere-IPv4** as the source lets in SSH connections from **any** IPv4 address. It's generally not best practice to set SSH access to "Anywhere-IPv4" due to security risks - this setup exposes our server to potential unauthorized access from any location.

Setting the source to Anywhere-IPv4 makes sure that EC2 Instance Connect will be successful, no matter which IP address it's using. If this was a long-term project, we'd search for the specific CIDR block of IP addresses that EC2 Instance Connect uses and restrict inbound SSH traffic to that range.

![image alt](Networking-60)

-   Select **Save rules**.
-   With that modified, refresh our EC2 console's **Instances** page.
-   Select our **Public Server** and select **Connect** again.
-   Phew! Success.

![image alt](Networking-61)

**Step - 3 : Test connectivity between our EC2 instances**

Let's see if we can connect with our Private Server from here.

**What does it mean to connect to our Private Server from our Public Server?** When our servers "talk" or connect, it usually means the public server is sending or requesting data from the private server. This could be for various reasons, such as fetching private information  or updating databases.

-   Leave open the **EC2 Instance Connect** tab, but head back to your **EC2 console** in a new tab.
-   Select **NextWork Private Server**.
-   Copy our private server's **Private IPv4 address**.

![image alt](Networking-62)

-   Switch back to the **EC2 Instance Connect** tab.
-   Run `ping [the Private IPv4 address you just copied]` in terminal.
      -   Our final result should look similar to something like `ping 10.0.1.227`.
      -   Don't know where to enter this prompt? Look for the `$` sign at the bottom line of the black window, and type in our command after the $ sign.
 
**Ping** is a common computer network tool used to check whether our computer can communicate with another computer or device on a network. When we "ping" a specific IP address address (i.e. another server's address), our server (in this case, NextWork Public Server) sends a small packet of data to that address (NextWork Private Server), asking for a response. Ping will tells us whether we get a response back and how long it took to get a response.

If we receive a response quickly, it means the connection between our computer and the other computer is good. If it takes a long time or we get no response, there might be a problem with the connection!

-   We should see a response similar to this:

![image alt](Networking-63)

This single line indicates that our Public Server has sent out a ping message... and that's about it. Usually, when we ping another computer successfully, we should see several replies back instantly. Each reply tells us how long it took for the message to go to the Private Server and come back.

If we don't get any replies (that's our situation right now), or if the replies stop suddenly, it's usually a sign that there's a problem with the connection.

**Why is there a problem with the connection between our servers?** One common reason for these issues is that the target machine (i.e. NextWork Private Server) or its network might be blocking the type of messages used in ping, which are known as **ICMP (Internet Control Message Protocol) traffic**. Blocking ICMP traffic is often done to prevent network attacks, like attackers can overwhelming a server with ping messages so it can't respond to real users wanting to use our application. Fair enough that ICMP traffic is blocked by default!

-   To resolve this connectivity error, let's investigate whether NextWork Private Server is allowing in ICMP traffic.
-   Leave open the **EC2 Instance Connect** tab, but head back to our **VPC console** in a new tab.
-   In the VPC console, select the **Subnets** page.
-   Select **NextWork Private Subnet**.
-   Let's investigate the **Route tables** and **Network ACL** tabs for our private subnet.
-   Aha! Mystery solved.

![image alt](Networking-64)

**How are the Route tables and Network ACLs the cause of our connectivity issue?** Our route table is set up perfectly (zero issues there!), but the Network ACL tab shows us that all traffic inbound and outbound are denied. This means even if our route table correctly directs the ping to NextWork Private Server, the network ACL is checking the ping traffic at the entrance of our private subnet. If it finds that ICMP traffic is not allowed, it stops the ping there. This blockage could be the reason why we observed only one line in the ping response - the single line records NextWork Public Server's first attempt to communicate (i.e. a ping message is sent), but there are no replies back from NextWork Private Server.

-   Let's resolve that by clicking on the link to our **NextWork Private NACL**.
-   Select the checkbox next to **NextWork Private NACL**.
-   Select the **Inbound rules** tab.
-   Select **Edit inbound rules**.
-   Let's add a new rule to let NextWork Public Server ping NextWork Private Server.
-   Select **Add new rule**.
-   Assign `100` as the rule number.
-   Change the **Type** to **All ICMP - IPv4**.

When we set a rule for **All ICMP - IPv4**, we're allowing all types of ICMP messages for IPv4 addresses. This covers a wide range of operational messages that are essential for diagnosing network connectivity issues, ping requests and responses are just one type of ICMP messages. We might notice that ICMP is not limited to IPv4; there is also ICMP for IPv6 , which functions similarly but is tailored specifically for IPv6 networks.

-   Set the **Source** to traffic coming from our public subnet - `10.0.0.0/24`.
-   Select **Save changes**.

![image alt](Networking-65)

-   Let's apply the same to Outbound rules.
      -   Rule number: `100`.
      -   Type: **All ICMP - IPv4**.
      -   Source: `10.0.0.0/24`.
-   Before we finish, let's check the security groups! Select **Security groups** from the left hand navigation panel.
-   Select **NextWork Private Security Group**.
-   Check your **Inbound rules** tab - does this security group allow ICMP traffic? (Nope!)

![image alt](Networking-66)

Now that our private network ACLs allow ICMP traffic, ICMP messages can enter our public subnet. However, these messages still need to be let in by our **NextWork Private Security Group** to reach our private server. So if our security group isn't allowing in ICMP traffic too, the ping message wouldn't reach our private server!

-   Select **Edit inbound rules**.
-   Select **Add rule**.
-   For **Type**, select **All ICMP - IPv4**.
-   For **Source**, select **NextWork Public Security Group**.

![image alt](Networking-67)

**We didn't pick NextWork Public Security Group as the source for the network ACL... why is the source different here?** Notice how we can select traffic from the **NextWork Public Security Group** as a source here, which is much more granular and exclusive than the private NACL, which allows in all traffic from our public subnet. NACLs typically have a broader scope since its settings would affect all resources in our subnet.

-   Select **Save rules**.
-   Revisit the **EC2 Instance Connect** tab that's connected to NextWork Public Server.
-   Woah! Lots of new lines coming through in the terminal.

![image alt](Networking-68)

The multiple new lines appearing in our terminal are a sign of successful communication between the two EC2 instances! Each line represents a reply from the ping command we sent. This means that the ICMP (Internet Control Message Protocol) traffic is now successfully reaching the private server, thanks to the adjustments we made in the network ACLs and Security Groups.

**Step - 4 : Test VPC connectivity with the internet**

Since our NextWork Public Route Table has a route from the NextWork Public Subnet to an internet gateway, we can validate that resources in NextWork Public Subnet can access the internet!

-   Quit the ping command by pressing `Control + C` on our keyboard.
-   Let's enter a new command!
-   Type in `curl example.com` in the prompt, i.e. right after the $ sign at the bottom line of the black window.
-   We should see a response similar to this!

![image alt](Networking-69)

Just like ping, **curl** is a tool to test connectivity in a network. Where ping checks if one computer can contact another (and how long messages take to travel back and forwth), curl is used to transfer data to or from a server. That means on top of checking connectivity, we can use curl to grab data from, or upload data into other servers on the internet!

When we use the `curl` command followed by a website address, e.g. `example.com`, the command sends an HTTP request to the server that hosts the website. This request tells the server that we want to retrieve the HTML content of the website. The website's server then processes this request and sends back the data as a response. The curl command outputs this data to our terminal, so what we're seeing is the raw HTML from the server!

-   This output confirms that our **Public Sever** instance can talk with the internet.
-   This wouldn't be possible if NextWork Public Subnet, our internet gateway and our security settings weren't set up properly... nice work!
-   Now let's run `curl nextwork.org`

![image alt](Networking-70)

The output **Found** typically means that the website at the URL we entererd has moved to a new URL! This is true - `nextwork.org` currently redirects requests to the first project on our web app, which has a different URL!

-   Now let's try running curl with the URL that our terminal returned. Run `curl https://learn.nextwork.org/projects/aws-host-a-website-on-s3`

![image alt](Networking-71)

Just like the previous example using `curl example.com`, curl has fetched the complete HTML content of NextWork's web app (specifically, the first project on the web app), which is why we now see a large amount of HTML data. We've just used our Public Server to fetch all the HTML code necessary to render a webpage!

**Step - 5 : Delete Our Resources**

**Public and Private Servers:**
-   In our **EC2 console**, select the checkboxes next to both instances.
-   Select the **Instance state** dropdown, and select **Terminate instance**.
-   Select **Terminate**.

**VPC:**
-   In our **VPC console**, select the checkbox next to **NextWork VPC**.
-   Select the **Actions** dropdown.
-   Select **Delete VPC**.
-   If stopped from deleting our VPC because network interfaces are still attached to our VPC - delete all the attached network interfaces first!

**Network interfaces** get created automatically when we launch an EC2 instance. Think of them as a component that attaches to an EC2 instance on one end and our VPC on another - so that our EC2 instance is connected to our network and can send and receive data! Network interfaces are usually deleted automatically with our EC2 instance, but on some occassions it'd be faster to delete them manually.

-   Type `delete` at the bottom of the pop up panel.
-   Select **Delete**.

![image alt](Networking-72)

Now visit each of the pages below!

**Refresh** our page before checking if the resource we created is still in our account. They should be automatically deleted with our VPC, but it's always a good idea to check anyway:
1.   Subnets
2.   Route tables
3.   Internet gateways
4.   Network ACLs
5.   Security groups

---
## VPC Peering

We're going to level up by setting up VPC Peering - we're going to play with TWO VPCs instead of one!

**Step - 1 :  Set up our VPCs in Minutes**

Now that we've learnt about the VPC wizard, let's use it to set up TWO VPCs in minutes.
-   [Log in to our AWS Account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26state%3DhashArgsFromTB_ap-southeast-2_fffdf5be4bb1a27e&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=m-aiqeB2UZeXTGXNyugMP8L64zd_AGUxJl4HLnA-X1o&code_challenge_method=SHA-256).
-   Head to our **VPC** console - search for `VPC` at the search bar at top of our page.
-   From the left hand navigation bar, select **Your VPCs**.
-   Select **Create VPC**.
-   We previously stuck to creating a VPC only, but this time let's select **VPC and more**.

![image alt](Networking-73)

**Create VPC 1**
-   Under **Name tag auto-generation**, enter `NextWork-1`.
-   The VPC's **IPv4 CIDR block** is already pre-filled to `10.0.0.0/16` - change that to `10.1.0.0/16`

Since we're setting up 3 VPCs, it's easier to remember their CIDR blocks if their ranges match their names. For example, 10.**1**.0.0/16 for **VPC 1**, then 10.**2**.0.0/16 for **VPC 2**, and so on...

-   For **IPv6 CIDR block**, we'll leave in the default option of **No IPv6 CIDR block**.
-   For **Tenancy**, we'll keep the selection of **Default**.
-   For **Number of Availability Zones (AZs)**, we'll use just **1** Availability Zone.
-   Make sure the **Number of public subnets** chosen is `1`.
-   For **Number of private subnets**, we'll keep thing simple today and go with `0` private subnets.
-   We updated our subnets' CIDR blocks in previous projects, but we don't need defined subnets for this one. Let's skip the Customize subnets CIDR for this project.

![image alt](Networking-74)

-   Next, for the **NAT gateways ($)** option, make sure you've selected **None**. As the dollar sign suggests, NAT gateways cost money!

NAT (Network Address Translation) gateways let instances in private subnets access the internet for updates and patches, while blocking inbound traffic.

-   Next, for the **VPC endpoints** option, select **None**.

VPC endpoints let us connect our VPC privately to AWS services without using the public internet. This means our data stays within the AWS network, which can improve security and reduce data transfer costs.

-  We can leave the **DNS options** checked.
-  Select **Create VPC**.

![image alt](Networking-75)

-   Select **View VPC**.
-   Select the **Resource map** tab - nice, all of these resources have been set up for us in a flash!

**Set up VPC 2**

We're going to set up another VPC with slightly different settings - pay close attention!
-   Select **Create VPC**.
-   Select **VPC and more**.
-   Under **Name tag auto-generation**, enter  `NextWork-2`
-   The VPC's **IPv4 CIDR block** should be unique! Make sure the CIDR block is NOT `10.1.0.0/16` - it should be `10.2.0.0/16`.

Each VPC must have a unique IPv4 CIDR block so the IP addresses of their resources don't overlap. Having overlapping IP addresses could cause routing conflicts and connectivity issues!

-   For **IPv6 CIDR block**, we'll leave in the default option of **No IPv6 CIDR block**.
-   For **Tenancy**, we'll keep the selection of **Default**.
-   For **Number of Availability Zones (AZs)**, we'll use just `1` Availability Zone.
-   Make sure the **Number of public subnets** chosen is `1`.
-   For **Number of private subnets**, we'll go with `0`. Let's keep it simple with just a single subnet!
-   For the **NAT gateways ($)** option, select **None**. 
-   For the VPC endpoints option, select None.
-   You can leave the **DNS options** checked.
-   Select **Create VPC**.

**Step - 2 : Create a Peering Connection**

Now that we have two VPCs ready to go, let's bridge them together with a peering connection.

-   Still in the VPC console, click on **Peering connections** on the left hand navigation panel.

A peering connection lets VPCs and their resources route traffic between them using their private IP addresses. This means data can now be transferred between VPCs without going through the public internet. Without a peering connection, data transfers between VPCs would use resources' public address - meaning VPCs have to communicate over the public internet.

![image alt](Networking-76)

-   Click on **Create peering connection** in the right hand corner.
-   Name our **Peering connection** name as `VPC 1 <> VPC 2`
-   Select **NextWork-1-VPC** for our **VPC ID (Requester)**.

In VPC peering, the Requester is the VPC that initiates a peering connection. As the requester, they will be sending the other VPC an invitation to connect!

![image alt](Networking-77)

-   Under **Select another VPC to peer with**, make sure **My Account** is selected.

VPC peering can occur between VPCs in **different AWS accounts**! This flexibility lets businesses collaborate securely by sharing resources across accounts without exposing them to the public internet.

-   For **Region**, select **This Region**.
-   For **VPC ID (Accepter)**, select **NextWork-2-VPC**

In VPC peering, the **Accepter** is the VPC that receives a peering connection request! The Accepter can either accept or decline the invitation. This means the peering connection isn't actually made until the other VPC also agrees to it!

![image alt](Networking-78)

-   Click on **Create peering connection**.
-   Our newly created peering connection isn't finished yet! The green success bar says the peering connection has been requested.

![image alt](Networking-79)

-   On the next screen, select **Actions** and then select **Accept request**... Get ready to take a screenshot!

We set up the VPC peering connection as VPC 1 (the Requestor), but don't forget that the Accepter needs to approve of it too! By clicking Accept request, we were accepting the connection as VPC 2.

![image alt](Networking-80)

-   Click on **Accept request** again on the pop up panel.
-   Click on **Modify my route tables now** on the top right corner.

![image alt](Networking-81)

**Step - 3 : Update Route Tables**

With a peering connection all set up, now it's time for traffic in our VPCs to learn how to use it.

**Update VPC 1's route table**
-   Select the checkbox next to VPC 1's route table i.e. called **NextWork-1-rtb-public**.
-   Scroll down and click on the **Routes** tab.
-   Click **Edit routes**.
-   Let's add a new route!

Even if our peering connection has been accepted, traffic in VPC 1 won't know how to get to resources in VPC 2 without a route in our route table! We need to set up a route that directs traffic bound for VPC 2 to the peering connection we've set up.

-   Add a new route to **VPC 2** by entering the CIDR block `10.1.0.0/16` as our Destination.
-   Under Target, select **Peering Connection**.
-   Select **VPC 1 <> VPC 2**.

![image alt](Networking-82)

-   Click **Save changes**.
-   Oops! We've hit an error.

![image alt](Networking-83)

The message says the destination we've put in has the same CIDR block as VPC 1's own CIDR block! Whoops that was a typo - VPC 2's CIDR block is actually 10.**2**.0.0/16. Route tables cannot differentiate between local and remote destinations if they share the same CIDR block. Imagine if we didn't set up VPC 2 with its own unique CIDR block in the previous step! We'd get into trouble here.

-   Let's fix that typo now! Select **Back**.
-   Update our new route's **Destination** to `10.2.0.0/16`.
-   Click **Save changes** again.
-   Confirm that the new route appears in VPC 1's **Routes** tab now!

![image alt](Networking-84)

**Update VPC 2's route table**
-   The route table we're updating is **NextWork-2-rtb-public**.
-   The **Destination** is the CIDR block `10.1.0.0/16`.

![image alt](Networking-85)

**Step - 4 : Launch EC2 Instances**

Launch an EC2 instance in each VPC, so we can use them to test our VPC peering connection later.

**Launch an instance in VPC 1**
-   Head to the **EC2 console** - search for `EC2` in the search bar at the top of screen.
-   Select **Instances** at the left hand navigation bar.
-   Select **Launch instances**.
-   Since our first EC2 instance will be launched in our first VPC, let's name it `Instance - NextWork VPC 1`
-   For the **Amazon Machine Image**, select **Amazon Linux 2023 AMI**.
-   For the **Instance type**, select **t2.micro**.
-   For the **Key pair (login)** panel, select  **Proceed without a key pair (not recommended)**.
-   At the **Network settings** panel, select **Edit** at the right hand corner.
      -   Under **VPC**, select **NextWork-vpc-1**.
      -   Under **Subnet**, select your VPC's public subnet.
      -   Keep the **Auto-assign public IP** setting to **Disable**.
      -   For the **Firewall (security groups)** setting, Amazon VPC already created a security group for our VPC - let's use that!
      -   Choose **Select existing security group**.
      -   Select the **default** security group for our VPC.
-   Select **Launch instance**.

**Launch an instance in VPC 2**
-   The **Name** is `Instance - NextWork VPC 2`.
-   The **VPC** is **NextWork-vpc-2**.

![image alt](Networking-86)

**Step - 5 : Connect to Instance 1**

To test our VPC peering connection, we'll need to get one of our EC2 instances to try talk to the other.

-   Still in our **EC2 console**, select the checkbox next to **Instance - NextWork VPC 1**.
-   Select **Connect**.

![image alt](Networking-87)

Check out the error message: **"No public IPv4 address assigned. With no public IPv4 address, you can't use EC2 Instance Connect."**

Keeping **Disable** for the **Auto-assign IP address** option in our EC2 instance's network settings caused this error! If we want to connect to our instance over EC2 Instance Connect, then our instance must have a public IP address and be in a public subnet. This is because using EC2 Instance Connect connects to our server **over the internet** by default.

-   Verify this by heading back to the **Instances** page in our EC2 console, and checking the Public IPv4 address field... it's empty!

![image alt](Networking-88)

-   On our EC2 console's left hand navigation panel, select **Elastic IPs**.

![image alt](Networking-90)

**Elastic IPs** are static IPv4 addresses that get allocated to our AWS account, and is ours to delegate to an EC2 instance. Elastic IPs are also a great way to assign an instance a public IPv4 address after launching it!

-   Select **Allocate Elastic IP** addresses.
-   Leave all default options.

![image alt](Networking-89)

This setting is asking us to choose the source of the Elastic IP address. Since IPv4 addresses are unique on the internet, the IPv4 address we are allocated has to come from somewhere and can't overlap with other existing IPv4 addresses! The default setting is **Amazon's pool of IPv4 addresses**. Think of this as a public pool of all IPv4 addresses that Amazon has saved to allocate to AWS accounts, so we won't need to find one yourself.

If we have our own pool of IP addresses, we can also set those up in our AWS account and allocate an IPv4 address from there instead (this is called Bring Your Own IP, or BYOIP).

-   Select **Allocate**.
-   Refresh our page, then select the new IP address we've set up.
-   Select the **Actions** dropdown, then select **Associate Elastic IP address**.
-   Under **Instance**, select **Instance - NextWork VPC 1**.

![image alt](Networking-91)

-   Click **Associate**.

![image alt](Networking-92)

This setting isn't quite relevant to us since our EC2 Instance only has one private IP address, but it becomes very important if an EC2 instance has multiple private IPs.

-   Awesome! Our EC2 Instance should have a public IP address now.
-   To check this, select **Instances** from the left hand navigation panel.
-   Select the checkbox next to **Instance - NextWork VPC 1**.
-   Do you see a **Public IPv4 address** for our EC2 instance now?

![image alt](Networking-93)

Looks like the IP address should be all resolved now... let's try connecting to our EC2 instance again!
-   Select **Connect**.
-   In the EC2 Instance Connect set up page, select **Connect** again.
-   Oh no! We've failed to connect to our instance?!

![image alt](Networking-94)

-   Head back to our **VPC console**.
-   Select **Subnets** from the left hand navigation panel.
-   Select the checkbox next to **NextWork-1-subnet-public1**...
-   Hmm let's take a look! Investigate the **Route table** and **Network ACL** tabs - what do we see?

![image alt](Networking-95)

![image alt](Networking-96)

Everything looks good! Our Network ACL allows all traffic in and out, and our route table is correctly setting up a route to the Internet Gateway. That leaves one more thing to investigate...

-   Copy the **VPC ID** of **NextWork-1-vpc**.

![image alt](Networking-97)

-   Head into the **Security groups** page from the left hand navigation panel.
-   Woah it's a bunch of nameless security groups!
-   To find the VPC 1's default security group, let's use the handy search bar.
-   Click into the search bar, and paste the ID we copied into the search bar. Make sure there aren't any empty spaces before our text.
-   Select the filter!

![image alt](Networking-98)

-   Nice, that narrowed it nicely for us. Select the checkbox next to VPC 1's **default** security group.
-   Select the **Inbound rules** tab.

![image alt](Networking-99)

-   Aha! Mystery solved.

The answer lies in the **Source** of our security group's inbound rules. We're trying to access **Instance - NextWork VPC 1** using SSH through EC2 Instance Connect, which is trying to connect to our instance over the internet. Our default security group only allows inbound traffic from within the VPC, so traffic from the internet is being cut off! The default security group for a new VPC does not allow incoming traffic from outside of the VPC. We have to allow inbound SSH traffic on port 22 ourself!

-   In the **Inbound rules** tab, select **Edit inbound rules**.
-   Select **Add rule**.
-   For our new rule, configure the **Type** as **SSH**.
-   Then, under **Source type**, select **Anywhere-IPv4**.
-   Select **Save rules**.

![image alt](Networking-100)

-   With that modified, refresh our EC2 console's **Instances** page.
-   Select our **Instance-NextWork VPC 1** and select **Connect** again.
-   Select **Connect** in the EC2 Instance Connect setup page.
-   Phew! Success.

![image alt](Networking-101)

**Step - 6 : Test VPC Peering**

We've figured out how to connect with VPC 1's Instance! Let's see if we can connect with VPC 2's instance from here.

-   Leave open the **EC2 Instance Connect** tab, but head back to your **EC2 console** in a new tab.
-   Select **Instance - NextWork VPC 2**.
-   Copy Instance - NextWork VPC 2's **Private IPv4 address**.

![image alt](Networking-102)

-   Switch back to the **EC2 Instance Connect** tab.
-   Run `ping [the Private IPv4 address we just copied]` in the terminal.
      -   Our final result should look similar to something like `ping 10.0.1.227`
      -   Don't know where to enter this prompt? Look for the `$` sign at the bottom line of the black window, and type in our command after the `$` sign.
 
**Ping** is a common computer network tool used to check whether our computer can communicate with another computer or device on a network. When we "ping" a specific IP address, our server (in this case, Instance - NextWork VPC 1) sends a small packet of data to the target server (Instance - NextWork VPC 2), asking for a response. Ping will tell us whether we get a response back and how long it took to get a response. If we receive a response quickly, it means the connection between our computer and the other computer is good. If it takes a long time or we get no response, there might be a problem with the connection!

-   We should see a response similar to this:

![image alt](Networking-103)

This single line indicates that our Instance - NextWork VPC 1 has sent out a ping message... and that's about it. If we don't get any replies (that's our situation right now), or if the replies stop suddenly, it's usually a sign that there's a problem with the connection.

One common reason for these issues is that the target server (Instance - NextWork VPC 2) or its network might be blocking the type of messages used in ping, which are known as **ICMP (Internet Control Message Protocol) traffic**. Blocking ICMP traffic is often done to prevent network attacks, like attackers can overwhelming a server with ping messages so it can't respond to real users wanting to use our application. Fair enough that ICMP traffic is blocked by default!

-   To resolve this connectivity error, let's investigate whether **Instance - NextWork VPC 2** is allowing inbound ICMP traffic.
-   Leave open the **EC2 Instance Connect** tab, but head back to our **VPC console** in a new tab.
-   In the VPC console, select the **Subnets** page.
-   Select VPC 2's subnet i.e. **NextWork-2-subnet-public1-...**
-   Let's investigate the **Route tables** and **Network ACL** tabs for our public subnet.
-   The network ACL allows all types of inbound traffic from anywhere! So this looks perfectly fine.
-   Before we finish, let's check the security groups!
-   Copy the **VPC ID** of VPC 2.

![image alt](Networking-104)

-   Select **Security groups** from the left hand navigation panel.
-   Paste the **VPC ID** in the search bar, and select the suggested filter.
-   Check our security group's **Inbound rules** tab - does this security group allow ICMP traffic from sources outside of VPC 2? (Nope!)
-   Aha! Mystery solved.

**Let's fix this by letting inbound ICMP traffic from VPC 1.**
-   Select **Edit inbound rules**.
-   Select **Add new rule**.
-   Change the **Type** to **All ICMP - IPv4**.

When we set a rule for **All ICMP - IPv4**, we're allowing all types of ICMP messages for IPv4 addresses. This covers a wide range of operational messages that are essential for diagnosing network connectivity issues, ping requests and responses are just one type of ICMP messages.

-   Set the **Source** to traffic coming from **VPC 1** - `10.1.0.0/16`
-   Select **Save rules**.

![image alt](Networking-105)

-   Revisit the **EC2 Instance Connect** tab that's connected to **Instance - NextWork VPC 1**.
-   Woah! Lots of new lines coming through in the terminal.

![image alt](Networking-106)

The multiple new lines appearing in your terminal are a sign of successful communication between the two EC2 instances. 

**Congratulations!** We've set up a peering architecture that connects **VPC 1** to **VPC 2** AND validated it with ping.

**Step - 7 : Delete Our Resources**

Keeping track of our resources, and deleting them at the end, is absolutely a skill that will help us reduce waste in our account.

**Delete our EC2 Instances**
-   Head back to the **Instances** page of our EC2 console.
-   Select the checkboxes next to **Instance - NextWork VPC 1** and **Instance - NextWork VPC 2**.
-   Select **Instance state**, then select **Terminate Instance**.
-   Select **Terminate**.

**Delete our Elastic IP address**
-   Select **Elastic IPs** from the left hand navigation panel.
-   Select the IP address we've created.
-   Select **Actions**, then **Release Elastic IP addresses**.

When we release an Elastic IP address, that address returns to Amazon's pool of IPv4 address and will eventually get reallocated to another AWS user!

-   Select **Release**.

**Delete VPC Peering Connections**
-   Head back to our **VPC** console.
-   Select **Peering connections** from your left hand navigation panel.
-   Select the `VPC 1 <> VPC 2` peering connection.
-   Select **Actions**, then **Delete peering connection**.
-   Select the checkbox to **Delete related route table entries**.
-   Type `delete` in the text box and click **Delete**.

**Delete our VPCs**
-   Select **Your VPCs** from your left hand navigation panel.
-   Select **NextWork-1-vpc**, then **Actions**, and **Delete VPC**.
-   Type `delete` in the text box and click **Delete**.
-   Note: if stopped from deleting our VPC because network interfaces are still attached to our VPC - delete all the attached network interfaces first!
-   Select **NextWork-2-vpc**, then **Actions**, and **Delete VPC**.
-   Type `delete` in the text box and click **Delete**.

---
##   VPC Monitoring with Flow Logs

Network monitoring is how engineers keep track of their network's traffic and analyze it too.

For example, network monitoring includes:
1.   Checking the source of traffic coming into our VPC,
2.   Analysing how much data is being transferred in our network, and
3.   Identifying traffic that's been blocked by our security settings.

Once we get to a more advanced stage, network monitoring also includes optimizing our network setup to speed up data transfers, automating responses to security threats, and so much more!

**Step - 1 : Set up our VPCs in Minutes**

Let's use VPC wizard to set up TWO VPCs in minutes. We can absolutely set up monitoring for architectures with a single VPC, but we're settings up two VPCs today so we can revise some learnings from the VPC peering project too!

-   [Log in to our AWS Account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26state%3DhashArgsFromTB_ap-southeast-2_fffdf5be4bb1a27e&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=m-aiqeB2UZeXTGXNyugMP8L64zd_AGUxJl4HLnA-X1o&code_challenge_method=SHA-256).
-   Head to our **VPC console** - search for `VPC` at the search bar at top of our page.
-   From the left hand navigation bar, select **Your VPCs**.

**Create VPC 1**
-   Select **Create VPC**.
-   Select **VPC and more**.
-   Under **Name tag auto-generation**, enter `NextWork-1`.
-   The VPC's **IPv4 CIDR block** is already pre-filled to `10.0.0.0/16` - change that to `10.1.0.0/16`.
-   For **IPv6 CIDR block**, we'll leave in the default option of **No IPv6 CIDR block**.
-   For **Tenancy**, we'll keep the selection of **Default**.
-   For **Number of Availability Zones (AZs)**, we'll use just `1` Availability Zone.
-   Make sure the **Number of public subnets** chosen is `1`.
-   For **Number of private subnets**, we'll keep thing simple and go with `0` private subnets.
-   Next, for the **NAT gateways ($)** option, make sure we've selected **None**. As the dollar sign suggests, NAT gateways cost money!
-   Next, for the **VPC endpoints** option, select **None**.
-   We can leave the **DNS options** checked.
-   Select **Create VPC**.
-   Select **View VPC**.
-   Select the **Resource map** tab - nice, all of these resources have been set up for us in a flash!

**Set up VPC 2**
-   Our second VPC has the exact same settings, except:
      -   Under **Name tag auto-generation**, enter `NextWork-2`.
      -   The VPC's **IPv4 CIDR block** should be unique! Make sure the CIDR block is `10.2.0.0/16` - NOT `10.1.0.0/16`.

**Step - 2 : Launch EC2 Instances**

We need to create these EC2 instances so that they can send data to each other later in the project, which gives us network activity to monitor.

**Launch an instance in VPC 1**
-   Head to the **EC2 console** - search for `EC2` in the search bar at the top of screen.
-   Select **Instances** at the left hand navigation bar.
-   Select **Launch instances**.
-   Since our first EC2 instance will be launched in our first VPC, let's name it `Instance - NextWork VPC 1`.
-   For the **Amazon Machine Image**, select **Amazon Linux 2023 AMI**.
-   For the **Instance type**, select **t2.micro**.
-   For the **Key pair (login)** panel, select **Proceed without a key pair (not recommended)**.
-   At the **Network settings** panel, select **Edit** at the right hand corner.
-   Under **VPC**, select **NextWork-vpc-1**.
-   Under **Subnet**, select our VPC's public subnet.
-   Keep the **Auto-assign public IP** setting to... do we remember whether our EC2 instance needs a public IP address?
-   Select **Enable**.
-   For the **Firewall (security groups)** setting, choose **Create security group**.
-   Name our security group `NextWork-1-SG`.
-   Choose **Add security group rule**.
-   For the new rule's **Type**, select **All ICMP - IPv4**.
-   For the new rule's **Source**, select `0.0.0.0/0`.

![image alt](Networking-107)

-   Select **Launch instance**.

**Launch an instance in VPC 2**

-   We use the same instructions above but make sure:
      -   The **Name** is `Instance - NextWork VPC 2`.
      -   The **VPC** is **NextWork-vpc-2**.
      -   Make sure we select **Enable** for **Auto-assign public IP** here too
      -   Name our security group `NextWork-2-SG`.
      -   Allow ICMP traffic from ALL IP addresses.
 
**Step - 3 : Set Up Flow Logs**

Next up, we are ready to start monitoring VPC traffic! We're using a tool called **VPC flow logs** to do this... let's set them up in this step.

-   Navigate to the **CloudWatch console** - search for `CloudWatch` in our Management Console's search bar.

**Amazon CloudWatch** is a powerful monitoring service that watches over everything happening in our AWS environment in real time. AWS services would report to CloudWatch with their metrics, which are stats or measurements of the service's performance. We can then use CloudWatch to take these metrics and transform them into clear, easy-to-read graphs and dashboards to help us understand our AWS environment's performance and health health. For example, if we were hosting an application on AWS, our dashboard could track how quickly our app is loading for our users.

Another great use case of CloudWatch is that we can set it to automatically do actions based on key metrics! For example, we could set CloudWatch to automatically shut down a VPC if it detects that it's letting in illegitimate traffic.

-   Check the **Region** we're on - is this the same Region as the one we've used to launch our VPCs?
      -   Double check our Region by looking at the top right hand corner of our console. Our Region is right next to our account name!
 
CloudWatch data is region-specific! That means the data collected on our VPC in one region is not automatically available in other regions. If we're using a region outside of the one we used to launch our VPCs, we won't get access to CloudWatch's metrics on our VPC!

-   At the left hand navigation panel, click **Log groups** under **Logs**.

![image alt](Networking-108)

-   Click **Create log group** at the top right.

Think of a **log group** as a big folder in AWS where we keep related logs together. Usually, logs from the same source or application will go into the same log group, BUT logs are also region-specific. This means log data gets created and saved in the region it was created, although we can use CloudWatch dashboards to bring together logs from different regions.

Logs are like a diary for our computer systems. They record everything that happens, from users logging in to errors popping up. It's the go-to place to understand what's going on with our systems, troubleshoot problems, and keep an eye on who’s doing what.

-   Enter `NextWorkVPCFlowLogsGroup` as the **Log group name**.

![image alt](Networking-109)

-   That's it! Click **Create**.

There were some interesting settings we skipped!
1. **Retention setting** is **Never expire** by default, which means our logs won’t be deleted over time. They’ll stick around as long as we need them, unless we decide to clear them out ourself.

![image alt](Networking-110)

2. **Log class** is **Standard** by default, which means the logs that get created will get accessed or analyzed regularly.
If we chose Infrequent Access instead, our logs will be stored for long-term archiving - we are charged less for storage, but higher for each time we need to access the, for analysis. This setting isn't quite important since our usage will fall under the Free Tier!

![image alt](Networking-111)

-   Head back to our **VPC console**.
-   Select the **Your VPCs** page.
-   Select **NextWork-1-vpc**.
-   Scroll down to the **Flow Logs** tab, and click on **Create flow log**.

Now that we know that logs are digital diaries that services keep, think of **flow logs** as a specific type of diary for VPCs. Flow logs capture traffic going to and from the network - noting down who's visiting our VPC and the specific network interface it going to!

**Network interfaces** exist to connect our resources to our VPC! For example, whenever we launch an EC2 instance, AWS automatically creates a primary network interface with a private IP address from our subnet's IPv4 CIDR block. That's how our EC2 instance gets a private IP address. 
They're not too important in simple networks like ours, but network interfaces become very powerful when we have complex network situations, like wanting an EC2 instance to belong to two subnets at once!

**Flow logs** are associated with specific network interfaces because each interface represents a distinct point where traffic enters or exits an EC2 instance or other AWS resources. No two resources can share the same network interface!

-   Welcome to the **Flow log settings** page! Nice, we've just unlocked a new section of VPC set up.
-   Enter `NextWorkVPCFlowLog` in the **Name** field.
-   Set **Filter** to **All**.

![image alt](Networking-112)

**Filter** setting in the context of VPC flow logs determines the type of traffic that is logged.
1. Setting the filter to **All** means that it captures all the traffic flowing in and out of the VPC.
2. The other option, **Accept**, means only traffic that was successfully allowed through our security groups and network ACLs get logged - great for understanding what kind of traffic is being let through.
3. There's also a **Reject** filter that only captures the traffic blocked by our network settings - great for detecting illegitimate traffic trying to access our resources, or any network setting issues!

-   Set the **Maximum aggregation interval** to **1 minute**.

**Maximum aggregation interval** means how often traffic flow data will get lumped into a single flow log record. If we set the interval to '1 minute', flow log records are generated every minute. This gives us a more granular view of network traffic compared to '10 minutes', which would give we less frequent and potentially less detailed records.

-   Leave **Destination** as **Send to CloudWatch Logs**.

![image alt](Networking-113)

Besides sending our flow logs straight to to CloudWatch, we can also send them to **Amazon S3**. This option is useful if we need to archive logs for cost-effective, long-term storage or use them with other analytics and processing tools in the future. **Amazon Data Firehose** is another destination! Data Firehose is a handy option that lets we process and analyze data in real-time (whereas S3 is better for storing large volumes of data that don't need real-time analysis).

-   Set **Destination log group** as **NextWorkVPCFlowLogsGroup**.
-   Under the **IAM role**, we might notice that there isn't an IAM role that's designed for Flow Logs!

![image alt](Networking-114)

**VPC Flow Logs doesn't have the permission to write logs and send them to CloudWatch... yet. Let's give Flow Logs the permission to do both, using the power of IAM roles and policies!**
-   Select that handy **Set up permissions** link under **IAM role**.

![image alt](Networking-115)

-   In the navigation pane, choose **Policies**.
-   Choose **Create policy**.
-   Choose **JSON**.
-   Delete everything in the **Policy editor**.
-   Add this JSON policy to the empty Policy editor:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}
```

![image alt](Networking-116)

-   Choose **Next**.
-   For our policy's name, let's call it `NextWorkVPCFlowLogsPolicy`.
-   Choose **Create policy**.
-   In the left hand navigation pane, choose **Roles**.
-   Choose **Create role**.
-   For **Trusted entity type**, choose **Custom trust policy**.

Custom trust policy is specific type of policy! They're different from IAM policies. While IAM policies help us define the actions a user/service can or cannot do, custom trust policies are used to very narrowly define who can use a role. By choosing a custom trust policy, we're making sure that only VPC Flow Logs can use this role.

We could assign permissions using the "AWS service" option too but we can't actually find an option for VPC Flow Logs! VPC Flow Logs is technically just a feature within the wider Amazon VPC service (instead of being its own standalone AWS service), so we'd have to use a custom trust policy to assign permissions specifically to VPC Flow Logs.

-   A custom trust policy panel pops up!
-   Replace `"Principal": {}`, with the following:

```json
"Principal": {
   "Service": "vpc-flow-logs.amazonaws.com"
},
```

-   To avoid any errors, make sure the statement is well formatted and the spacing of our lines look exactly like this!

![image alt](Networking-117)

The statement `"Service": " vpc-flow-logs.amazonaws.com "` in a trust policy specifically points to VPC Flow Logs as the only service that can use this role! Even if we try to give this role to other AWS services, they can't use it because the permissions are locked down to just VPC Flow Logs. This is so good for security, in case this role gets accidentally assigned to another service/user.

`"Principal"` defines the entity that is given the permissions in this policy. In this case, the entity is a service (VPC Flow Logs), but other entity types are IAM Users and IAM roles!

-   Choose **Next**.
-   On the **Add permissions** page, search for the policy we've created - `NextWorkVPCFlowLogsPolicy`.
-   Select our policy.

![image alt](Networking-118)

-   Choose **Next**.
-   Enter a name for our role - `NextWorkVPCFlowLogsRole`.
-   Choose **Create role**.

-   Head back to our VP console's **Create flow log** page.

![image alt](Networking-119)

-   Select the **refresh** button next to the IAM role field.
-   Select our IAM role - **NextWorkVPCFlowLogsRole**.

![image alt](Networking-120)

-   Click on **Create flow log**.

![image alt](Networking-121)

Nice - the flow log is all set up! This means network traffic going into and out of our VPC is now getting tracked.

**Step - 4 : Test VPC Peering**

Let's generate some network traffic and see whether our flow logs can pick up on them. We're going to generate network traffic by trying to get our instance in VPC 1 to send a message to our instance in VPC 2.

Since we're trying to get our instances to talk to each other, this means we're also testing our VPC peering setup at the same time!
-   Head to our **EC2** console and the **Instances** page.
-   Select the checkbox next to **Instance - NextWork VPC 1**.
-   Select **Connect**.
-   In the EC2 Instance Connect set up page, select **Connect** again.
-   Phew! Success.

![image alt](Networking-122)

-   Leave open the **EC2 Instance Connect** tab, but head back to our **EC2** console in a new tab.
-   Select **Instance - NextWork VPC 2**.
-   Copy Instance - NextWork VPC 2's **Private IPv4 address**.

This step is all about testing the peering connection between our EC2 instances. If the peering connection is succesful, our instances can communicate using each other's private IPv4 addresses. If they can't communicate using their private IPv4 addresses, then... we have an error! That's why it's a good idea to test with the private IPv4 addresses - using public IPv4 addresses wouldn't give us much information about whether the peering connection was successful!

![image alt](Networking-123)

-   Switch back to the **EC2 Instance Connect** tab.
-   Run `ping [the Private IPv4 address you just copied]` in the terminal.
      -   Our final result should look similar to something like `ping 10.0.1.227`.
      -   Don't know where to enter this prompt? Look for the `$` sign at the bottom line of the black window, and type in our command after the `$` sign.
 
**Ping** is a network tool used to check whether our computer can communicate with another computer or device on a network. When we "ping" a specific IP address, our server (in this case, Instance - NextWork VPC 1) sends a small packet of data to the target server (Instance - NextWork VPC 2), asking for a response. Ping will tell us whether we get a response back and how long it took to get a response.

-   We should see a response similar to this:

![image alt](Networking-124)

If we don't get any replies (that's our situation right now), or if the replies stop suddenly, it's usually a sign that there's a problem with the connection. But we've made sure our security group allows inbound ICMP traffic, so what's happened here?

-   Let's do some investigating... press **Ctrl + C** on our keyboard to end this ping test.
-   See if we can test the connection from VPC 1 to VPC 2's **public** IP address.
-   Head back to our **EC2** console, and copy the **Public IPv4 address of Instance - NextWork VPC 2**.
-   Head back to our **EC2 Instance Connect** tab and run a ping test with this public IPv4 address.
-   Our final result should look similar to something like `ping [public IPv4 address]`.
-   Do we get ping replies back?

![image alt](Networking-125)

Receiving ping replies from the public IPv4 address means Instance 2 is correctly configured to respond to ping requests, and Instance 1 can actually communicate with Instance 2 if it traffic goes across the public internet!

-   In our **EC2 Instance Connect** page, press **Ctrl + C** on our keyboard again to end this ping test.
-   Ping our EC2 Instance 2's **private** address again.

![image alt](Networking-126)

-   Still no replies with the **private** IP address.

We receive ping replies when we use Instance 2's **public** IP address, which confirms that VPC 2's security groups and NACL's are letting in ICMP traffic. But using Instance 2's **private** IP address doesn't give us any ping replies.

-   Leave open the **EC2 Instance Connect** tab, but head back to our **VPC** console in a new tab.
-   In the VPC console, select the **Subnets** page.
-   Select VPC 1's subnet i.e. **NextWork-1-subnet-public1-...**
-   Let's investigate the **Route tables** and **Network ACL** tabs for our subnet.
-   The network ACL allows all types of inbound traffic from anywhere! So this looks perfectly fine.
-   But let's take a closer look at the route tables...
-   Aha! Mystery solved.

![image alt](Networking-127)

The missing ingredient in our architecture is the **VPC peering connection** that directly connects VPCs 1 and 2. The purpose of a peering connection is to create a **direct** link between two resources so they can communicate with their private IP addresses. We'd be correct to say that Instance 1 and Instance 2 are currently connected through the route with a destination of 0.0.0.0/0, but that traffic is through the internet gateway i.e. traffic will travel through and be exposed to the public internet. To make sure communication between Instances 1 and 2 is direct, we need to set up a new route that directs traffic to our peering connection (instead of the public internet).

-   Head to the VPC console, click on **Peering connections** on the left hand navigation panel.

A VPC peering connection is a direct connection between two VPCs! A peering connection lets VPCs and their resources route traffic between them using their **private** IP addresses. This means data can now be transferred between VPCs without going through the public internet. Without a peering connection, data transfers between VPCs would use resources' public address - meaning VPCs have to communicate over the public internet!

![image alt](Networking-128)

-   Click on **Create peering connection** in the right hand corner.
-   Name our **Peering connection name** as `VPC 1 <> VPC 2`.
-   Select **NextWork-1-VPC** for our **VPC ID (Requester)**.

In VPC peering, the Requester is the VPC that initiates a peering connection. As the requester, they will be sending the other VPC an invitation to connect!

![image alt](Networking-129)

-   Under **Select another VPC to peer with**, make sure **My Account** is selected.
-   For **Region**, select **This Region**.
-   For **VPC ID (Accepter)**, select **NextWork-2-VPC**.

In VPC peering, the Accepter is the VPC that receives a peering connection request! The Accepter can either accept or decline the invitation. This means the peering connection isn't actually made until the other VPC also agrees to it!

![image alt](Networking-130)

-   Click on **Create peering connection**.
-   Our newly created peering connection isn't finished yet! The green success bar says the peering connection has been requested.

![image alt](Networking-131)

-   On the next screen, select **Actions** and then select **Accept request**...!

We set up the VPC peering connection as VPC 1 (the Requestor), but don't forget that the Accepter needs to approve of it too! By clicking **Accept request**, we were accepting the connection as VPC 2.

![image alt](Networking-132)

-   Click on **Accept request** again on the pop up panel.
-   Click on **Modify my route tables now** on the top right corner.

**Update VPC 1's route table**
-   Select the checkbox next to VPC 1's route table i.e. called **NextWork-1-rtb-public**.
-   Scroll down and click on the **Routes** tab.
-   Click **Edit routes**.
-   Let's add a new route!

Even if our peering connection has been accepted, traffic in VPC 1 won't know how to get to resources in VPC 2 without a route in our route table! We need to set up a route that directs traffic bound for VPC 2 to the peering connection we've set up.

-   Add a new route to **VPC 2** by entering the CIDR block `10.2.0.0/16` as our Destination.
-   Under Target, select **Peering Connection**.
-   Select **VPC 1 <> VPC 2**.

![image alt](Networking-133)

-   Click **Save changes**.
-   Confirm that the new route appears in VPC 1's **Routes** tab!

![image alt](Networking-134)

**Update VPC 2's route table**
-   We use the same instructions above but make sure:
      -   The route table we're updating is **NextWork-2-rtb-public**.
      -   The **Destination** is the CIDR block `10.1.0.0/16`
      -   Save our changes!
 
![image alt](Networking-135)

-   Revisit the **EC2 Instance Connect** tab that's connected to NextWork Public Server.
-   Woah! Lots of new lines coming through in the terminal.

![image alt](Networking-136)

Congratulations!!! We've successfully resolved the connectivity issue by setting up a peering architecture between VPC 1 and VPC 2!

**Step - 5 : Analyze Flow Logs**

Let's check out what VPC Flow Logs has recorded about our network's activity!
-   Head to our **CloudWatch** console.
-   Select **Log groups** from the left hand navigation panel.
-   Click into **NextWorkVPCFlowLogsGroup**.
-   Click into our log stream to see flow logs from EC2 Instance 1!

![image alt](Networking-137)

Log streams in CloudWatch are often named after the network interface ID (`eni-xxx`) when they're associated with VPC flow logs. This helps us organise which streams are tracking traffic to which resources in our VPC.

`eni` = Elastic Network Interface (ENI)! If ever wonder how an EC2 instance can have a public/private IP address, security group rules and the option connect to other services in our AWS environment, the ENI is the answer. Think of ENI as a cloud networking component that attaches to an EC2 instance and gives the EC2 instance networking capabilities. Every EC2 instance must have an ENI to exist and be in a VPC.

-   Oooo check out these **log events!**

![image alt](Networking-138)

-   Scroll to the very top and try expanding a log at the top - woahhh, welcome to this flow log.

![image alt](Networking-139)

This flow log shows that **344 bytes** of data were sent successfully from the IP address **18.237.140.165** to **10.1.5.112** using TCP protocol on **port 22**, with **4 packets** transferred and the traffic was allowed (**"ACCEPT"**). EC2 Instance Connect's IP address range in the Oregon region is 18.237.140.160/29, which means the source of traffic mentioned in this log is actually EC2 Instance Connect's direct access to Instance 1!

-   Now scroll to the very bottom and select one of the newer logs, are there any differences?
-   For example, we might find one that says **REJECT OK** instead of **ACCEPT OK** at the end. These would represent the ping messages that failed to reach Instance 2!

![image alt](Networking-140)

-   In the left hand navigation panel, click on **Logs Insights**.

![image alt](Networking-141)

**Logs Insights** is a CloudWatch feature that analyzes our logs. In Log Insights, we use queries to filter, process and combine data to helps us troubleshoot problems or better understand our network traffic!

-   Select **NextWorkVPCFlowLogsGroup** from the **Select log group(s)** dropdown.
-   Select the **Queries** folder on the right hand side.

Queries are like commands we run to analyze our logs! We can use queries to filter logs, group data, and perform calculations like sums and averages. We use queries to extract specific information from large volumes of log data, making it easier to see trends, identify outliers, and troubleshoot issues. For example, if we're managing a network and traffic suddenly slows down, we could use queries to extract logs on the longest wait times in our network.

![image alt](Networking-142)

-   Under **Flow Logs**, select **Top 10 byte transfers by source and destination IP addresses**.

The query **"Top 10 byte transfers by source and destination IP addresses"** is all about discovering the top 10 biggest data transfers between IP addresses in our network! We'll find out which resources are moving the most data, which uncovers the busiest traffic routes in our network.

![image alt](Networking-143)

-   Click **Apply**, and then **Run query**.

![image alt](Networking-144)

-   Review the query results... wow!

![image alt](Networking-145)

Out of all the logs that Flow Logs has captured, here are the ten pairs of source and destination IP addresses that transferred the most data between them. So as we might imagine, this is a great query for investigating any heavy traffic flows or unusual data transfers! The bar chart at the top is just a little visualization to show us how many logs were captured at specific times of the day. The table below are the actual results of our query.

-   Can we map the logs with IP addresses we've come across during this project?

![image alt](Networking-146)

**Step - 6 : Delete Our Resources**

Keeping track of our resources, and deleting them at the end, is absolutely a skill that will help us reduce waste in our account.

**Delete our CloudWatch Log Group**
-   In our **CloudWatch** console, select **Log groups** from the left hand navigation panel.
-   Select our **NextWorkVPCFlowLogsGroup**.
-   Click the **Actions** button, and select **Delete log group(s)**.
-   Confirm by pressing the **Delete** button.

![image alt](Networking-147)

**Delete our EC2 Instances**
-   Head back to the **Instances** page of our **EC2** console.
-   Select the checkboxes next to **Instance - NextWork VPC 1** and **Instance - NextWork VPC 2**.
-   Select **Instance state**, then select **Terminate Instance**.
-   Select **Terminate**.

**Delete VPC Peering Connections**
-   Head back to our **VPC** console.
-   Select **Peering connections** from our left hand navigation panel.
-   Select the `VPC 1 <> VPC 2` peering connection.
-   Select **Actions**, then **Delete peering connection**.
-   Select the checkbox to **Delete related route table entries**.
-   Type `delete` in the text box and click **Delete**.

![image alt](Networking-148)

**Delete our VPCs**
-   Select **Your VPCs** from our left hand navigation panel.
-   Select **NextWork-1-vpc**, then **Actions**, and **Delete VPC**.
-   Type `delete` in the text box and click **Delete**.
-   Note: if you get stopped from deleting our VPC because **network interfaces** are still attached to our VPC - delete all the attached network interfaces first!
-   Select **NextWork-2-vpc**, then **Actions**, and **Delete VPC**.
-   Type `delete` in the text box and click **Delete**.

**Delete our CloudWatch IAM Role and Policy**
-   Head to our **IAM** console.
-   Select **Policies** from the left hand navigation panel.
-   Search for `FlowLogs`, and select **NextWorkVPCFlowLogsPolicy**.
-   Select **Delete**, then enter `NextWorkVPCFlowLogsPolicy` to confirm our deletion.
-   Select **Roles** from the left hand navigation panel.
-   Search for `FlowLogs`, and select **NextWorkVPCFlowLogsRole**.
-   Select **Delete**, then enter `NextWorkVPCFlowLogsRole` to confirm our deletion.

![image alt](Networking-149)

---
##   Access S3 from a VPC

**Amazon S3** is a classic AWS service that doesn't live inside a VPC! **S3** is, by default, accessible from the internet, so we can manage its access and security without having to change network settings or internet gateways. And S3 is just the beginning. Services like AWS IAM, Amazon Route 53, and even databases like DynamoDB also live outside of our VPC.

Engineers and companies combine lots of AWS services when they build applications, so resources inside our VPC need to know how to work with the other AWS services outside. Let’s kick off by accessing Amazon S3 from our VPC in this introductory project!

**Step - 1 : Set up our VPC and EC2 Instance**
-   [Log in to your AWS Account](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26state%3DhashArgsFromTB_ap-southeast-2_fffdf5be4bb1a27e&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=m-aiqeB2UZeXTGXNyugMP8L64zd_AGUxJl4HLnA-X1o&code_challenge_method=SHA-256).

**Create Your VPC**
-   Head to our **VPC** console - search for **VPC** at the search bar at top of our page.
-   From the left hand navigation bar, select **Your VPCs**.
-   Select **Create VPC**.
-   Select **VPC and more**.
-   Under **Name tag auto-generation**, enter `NextWork`.
-   The VPC's **IPv4 CIDR block** is already pre-filled to `10.0.0.0/16` - we'll use this default CIDR block!
-   For **IPv6 CIDR block**, we'll leave in the default option of **No IPv6 CIDR block**.
-   For **Tenancy**, we'll keep the selection of **Default**.
-   For **Number of Availability Zones (AZs)**, we'll use just `1` Availability Zone.
-   Make sure the **Number of public subnets** chosen is `1`.
-   For **Number of private subnets**, we'll keep thing simple today and go with `0` private subnets.
-   For the **NAT gateways ($)** option, make sure we've selected **None**. As the dollar sign suggests, NAT gateways cost money!
-   For the **VPC endpoints** option, select **None**.
-   We can leave the **DNS options** checked.
-   Select **Create VPC**.

**Launch an instance in our VPC**
-   Head to the **EC2 console** - search for `EC2` in the search bar at the top of screen.
-   Select **Instances** at the left hand navigation bar.
-   Select **Launch instances**.
-   Since our first EC2 instance will be launched in our first VPC, let's name it `Instance - NextWork VPC Project`.
-   For the **Amazon Machine Image**, select **Amazon Linux 2023 AMI**.
-   For the **Instance type**, select **t2.micro**.
-   For the **Key pair (login)** panel, select **Proceed without a key pair (not recommended)**.
-   At the **Network settings** panel, select **Edit** at the right hand corner.
-   Under **VPC**, select **NextWork-vpc**.
-   Under **Subnet**, select our VPC's public subnet.
-   Keep the **Auto-assign public IP** setting to **Enable**.
-   For the **Firewall (security groups)** setting, choose **Create security group**.
      -   Name your security group `SG - NextWork VPC Project`.
      -   That's it! No security group rules to add.

By default, this new security group allows all inbound SSH traffic. That's all we need to use EC2 Instance Connect later. We used to add an inbound rule to allow ICMP traffic - but we won't need it if we aren't running any ping tests!

-   Select **Launch instance**.

**Step - 2 : Connect to Our EC2 Instance**

In this step, we're gonna connect to our EC2 instance and try access an AWS service!
-   Head to our **EC2** console and the **Instances** page.
-   Select the checkbox next to **Instance - NextWork VPC Project**.
-   Select **Connect**.
-   In the EC2 Instance Connect set up page, select **Connect** again.
-   Yay! Successfully connected.

![image alt](Networking-150)

-   For our first command, try running `aws s3 ls`, which is a command used to list the S3 buckets in our account (yup, **ls** stands for list!).

![image alt](Networking-151)

-   Hmmm, doesn't look like a success. We need to provide credentials.

Credentials in AWS are like keys that let us access and manage AWS services securely. Without credentials, we won't have the permission to do things like viewing our S3 bucket list.

When we log into our AWS Management Console, we're already authenticated using our AWS user's details. Running commands from an EC2 instance is a different story! We can think of our EC2 instance as another user that needs its own username and password (i.e. credentials) to get access to our AWS account. Our account by default doesn't have credentials set up for running commands from an EC2 instance. We'll need to manually set these up to securely access and manage AWS services from our instance.

-   Run `aws configure`
-   The terminal is now asking us for an Access Key ID!

![image alt](Networking-152)

An access key ID is a part of a credential! Our credentials are made up of a username and password; think of the access key ID as the username. We don't automatically have one, but we can create access keys IDs through AWS IAM.

Our EC2 instance needs credentials to access our AWS services, so let's set up access keys right away!
-   Search for `Access keys` in the search bar.
-   Aha! So it's the IAM console that helps us set this up. Select **IAM**.
-   Search for `Access keys` again in the IAM console's left hand navigation panel.

![image alt](Networking-153)

-   Choose the search result for managing an access key for our IAM Admin account.

![image alt](Networking-154)

-   Select **Create access key**.
-   On the first set up page, select **Command Line Interface (CLI)**.
-   Select the checkbox that says **I understand the above recommendation and want to proceed to create an access key.**

![image alt](Networking-155)

In this project we're creating access keys and manually applying them in our EC2 instance, but typically the recommended way is to create an IAM role with the necessary permissions and then attaching that role to our EC2 instance. Our EC2 instance would inherit the permissions from the role, and this is best practice as we can easily attach and detach EC2 instances from roles to give and take away their credentials. We aren't using this method so that we can learn about access keys, but roles are usually a better alternative for security.

-   Select **Next**.
-   For the **Description tag value**, we'll write `Access key created to access an S3 bucket from an EC2 Instance. NextWork VPC project.`

![image alt](Networking-156)

-   Select **Create access key**.
-   OOoo this is important! **STAY ON THIS PAGE**

![image alt](Networking-157)

This is the only time that our secret access key can be viewed or downloaded. We cannot recover it later. However, we can create a new access key any time.

-   Select **Download .csv file** near the bottom of the page.

**Step - 3 : Create an S3 bucket with 2 files**

Before we jump back into our EC2 instance, let's create a bucket in Amazon S3. After creating this bucket, we'll learn how to access it from our EC2 instance and do things like checking what objects are in the bucket.

-   Search for **S3** at the search bar at the top of our console.
-   Make sure we're in the same Region as our NextWork VPC!
-   Select **Create bucket**.
-   Let's set up our bucket! Keep the **Bucket type** as **General purpose**.
-   Our **bucket name** should be `nextwork-vpc-project-yourname`
    -   Don't forget to replace **yourname** with our first name! **S3 bucket names need to be globally unique**.
-   We'll leave all other default settings, go straight to **Create bucket**.

Next, let's upload two files into the bucket. This can be any two files in our local computer!
-   Click into our bucket.
-   Select **Upload**.

![image alt](Networking-159)

-   Select **Add files** in the **Files and folders panel**.
-   Select two files in our local computer to upload.

Not sure what to upload? You can download and use these images instead! **Right click** on each link, select **Save link as**... and select **Save**.
-   [NextWork - Denzel is awesome.png](https://storage.googleapis.com/nextwork_course_resources/courses/aws/AWS%20Project%20People%20projects/Project%3A%20VPC%20Endpoints/NextWork%20-%20Denzel%20is%20awesome.png)
-   [NextWork - Lelo is awesome.png](https://storage.googleapis.com/nextwork_course_resources/courses/aws/AWS%20Project%20People%20projects/Project%3A%20VPC%20Endpoints/NextWork%20-%20Lelo%20is%20awesome.png)

Our files should show up in our **Files and folders** panel once they're uploaded!

![image alt](Networking-160)

-   Select **Upload** at the bottom of the page.

**Step - 4 : Connect to our S3 bucket**

-   Now back to our **EC2 Instance Connect** tab!
-   Are we ready to enter our **Access keys details now?**
-   Open the **AccessKeys.csv** file we've downloaded.
-   Copy the **Access key ID**.
-   Paste this into our EC2 instance's terminal, and press **Enter** on our keyboard.

![image alt](Networking-161)

-   Let's do the same for the **AWS Secret Access Key!**
-   Copy the key from our .csv file, and paste it in the terminal. Press **Enter**.
-   Next, for our **Default region name**, open our **Region** dropdown on the top right hand corner of our AWS console.
-   Copy our **Region's code name**. For example, us-west-2.
-   Paste that in our EC2 instance's terminal. Press **Enter** on our keyboard.
-   We don't have a default output format, so we can leave that empty. Press **Enter**.

![image alt](Networking-162)

-   Nice! Set up is done.
-   Let's try running the `aws s3 ls` command again to see a list of our account's S3 buckets.

Do we see a list of our buckets? We might have a shorter list than the one in this screenshot, but the only line to look out for is **vpc-project-yourname**.

![image alt](Networking-163)

-   Next, let's run the command `aws s3 ls s3://nextwork-vpc-project-yourname`. Make sure to replace `nextwork-vpc-project-yourname` with our actual bucket name.
-   Based on the command, and what we've learnt about **aws s3 ls**, can we guess what this command does?
-   Nice - we can even see the objects that are inside our bucket!

![image alt](Networking-164)

We can also upload files using AWS CLI too?!
-   Run `sudo touch /tmp/test.txt` to create a blank .txt file in your EC2 instance.

Let's break it down!
      1.   **sudo** stands for "superuser do" and it's used when we're running a command that typicaly needs elevated user permissions, like installing software or creating new files.
      2.   **touch** is a standard command used to create an empty file if it doesn't exist. If it does exist, this command will update the file's timestamp (i.e. the last time it was accessed) instead.
      3.   **/tmp/** is a common directory path (we can think of a directory path as layers and layers of folders) typically used for temporary files.
      4.   **test.txt** is the name of the empty file we're creating!

-   Next, run `aws s3 cp /tmp/test.txt s3://nextwork-vpc-project-yourname` to upload that file into our bucket.

Let's break it down!
      1.   **aws s3 cp**: is the command to **copy** files.
      2.   **/tmp/test.txt** is the **source file path**. It's saying that the file to copy is test.txt, which exists inside a folder called tmp.
      3.   **s3://nextwork-vpc-project-yourname** is the **destination path**. It's saying that the file should be copied to the S3 bucket vpc-project-yourname!

-   Finally, run `aws s3 ls s3://nextwork-vpc-project-yourname` again - what objects are listed in our bucket now?

![image alt](Networking-165)

The numbers next to each file name is the size of that file! Since test.txt is an empty file with no data, it has 0 bytes.

-   Switch tabs back to our **S3** console.
-   Refresh the **Objects** tab for our S3 bucket.
-   Do we see our new file there?

![image alt](Networking-166)

A file we've created in our EC2 instance now lives in our S3 bucket too!

---
##   VPC Endpoints

Previously we learnt how to use our VPC to **access other AWS services**, specifically S3. With direct access to our EC2 instance, we could use AWS CLI commands to see what objects are in our S3 bucket and even upload files! But there's just one thing... with this setup, our EC2 instance is accessing our S3 bucket through the **public internet**.

When traffic is going through the internet, there is a higher risk of external threats and attacks looking at our data! For example, attackers can expose our EC2 instance's keys to our AWS account if they manage to break into the traffic between our instance and S3 bucket.

**VPC endpoints** gives our VPC **private, direct access to other AWS services like S3**, so traffic doesn't need to go through the internet. Just like how internet gateways are like our VPC's door to the internet, we can think of VPC endpoints as private doors to specific AWS services.

**Step - 1 : Create an endpoint**

Our EC2 instance definitely has access to our bucket - our access key worked But our instance is connected to our bucket through the public internet. It's time we bring in VPC endpoints!
-   In a new tab, head back to your **VPC** console.
-   Select **Endpoints** from the left hand navigation panel.

![image alt](Networking-167)

-   Select **Create endpoint**.

Not all AWS resources get launched inside our VPC! While our EC2 instances live within our VPC, S3 buckets and some other AWS services exist outside our VPC because they're designed to be highly available and accessible from anywhere. For example, the commands we're putting through AWS CLI now will be communicated to our S3 bucket through the public internet.

That's why VPC endpoints exist to create a private connection between our VPC and AWS services. Having a VPC endpoint means our instances can now access services like S3 directly without routing through the public internet, which makes sure our data stays within the AWS network for security.

-   For the **Name tag**, let's use `NextWork VPC Endpoint`.
-   Keep the **Service category** as **AWS services**.

![image alt](Networking-168)

-   In the **Services** panel, search for `S3`.
-   Select the filter result that just ends with **s3**.

![image alt](Networking-169)

-   Select the row with the **Type** set to **Gateway**.

![image alt](Networking-170)

A **Gateway** is a type of endpoint used specifically for Amazon S3 and DynamoDB (DynamoDB is an AWS database service). Gateways work by simply adding a route to our VPC route table that directs traffic bound for S3 or DynamoDB to head straight for the Gateway instead of the internet.

As AWS evolved and more companies started using it for diverse and complex tasks, there was a need for more detailed control over how data moves within AWS networks. Gateway endpoints they could manage traffic but didn’t offer detailed settings - this led to AWS creating **Interface endpoints**, which offer more security settings.

-   Next, at the **VPC** panel, select **NextWork-vpc**.
-   We'll leave the remaining default values and select **Create endpoint**.

![image alt](Networking-171)

-   Select the checkbox next to our endpoint's name.

![image alt](Networking-172)

**Step - 2 : Create a super secure bucket policy**

We're going to block off our S3 bucket from ALL traffic... except traffic coming from the endpoint. It's the ultimate test - if our endpoint was truly set up properly, then our EC2 instance should still be able to access S3. Otherwise, our instance's access is blocked!

In our S3 bucket, we're going to kick things up a notch and make access super secure. Let's say we want to make this bucket completely private to ALL access... except access through the VPC endpoint we've set up.
-   In the **S3** console, select **Buckets** from the left hand navigation panel.
-   Click into our bucket **vpc-endpoints-yourname**.
-   Select the **Permissions** tab.

![image alt](Networking-173)

-   Scroll to the **Bucket policy** panel, and select **Edit**.

A bucket policy is a type of IAM policy designed for setting access permissions to an S3 bucket. Using bucket policies, we get to decide who can access the bucket and what actions they can perform with it.

-   Add the below bucket policy to the policy editor.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-xxxxxxx"
        }
      }
    }
  ]
}
```

This policy **denies** all actions (`s3:*`) on our S3 bucket and its objects to everyone (`Principal: "*"`)... unless the access is from the VPC endpoint with the ID defined in `aws:sourceVpce`. In other words, only traffic coming from our VPC endpoint can get any access to our S3 bucket!

Don't forget to replace:
1.   BOTH instances of **arn:aws:s3:::your-bucket-name** with our actual bucket ARN.
      -   Handy tip: we can find our Bucket ARN right above the Policy window
  
![image alt](Networking-174)

1.   **vpce-xxxxxxx** with our VPC endpoint's ID.
      -   To find this ID, we'll have to switch back to our **Endpoints** tab and copy our endpoint's ID. Make sure it starts with `vpce-`.
  
![image alt](Networking-175)

After replacing the default values with our resources' ARN or ID, double check that we haven't deleted the quote marks around any of our statements. Also check that we still have `/*` at the second Resource line!

-   Select **Save changes**.

Woah! Once we've saved our changes, panels have turned red all across the screen.

![image alt](Networking-176)

Our policy denies all actions unless they come from our VPC endpoint. This means any attempt to access our bucket from other sources, including the AWS Management Console, is blocked!

**Step - 3 : Access Our S3 bucket again!**

Now that our S3 bucket's access is only limited to our endpoint, can our EC2 instance still access our bucket?

If it can, we've set up our endpoint connection perfectly.

If it can't, something's gone wrong with our VPC endpoint set up...

-   Head back to **EC2 Instance Connect**.
-   Try running `aws s3 ls s3://nextwork-vpc-endpoints-yourname` again.
-   Ah! Access denied. It looks like the bucket policy had stopped us.

![image alt](Networking-177)

-   Troubleshoot this error by heading to our VPC console.
-   Select **Subnets** from the left hand navigation panel.
-   Select our public subnet, which starts with **NextWork-subnet-public1**.
-   Select the **Route table** tab.
-   Aha! Mystery solved.

![image alt](Networking-178)

If our route table doesn't have a route that directs traffic bound for S3 to our VPC endpoint, traffic from our EC2 instance is actually trying to get to our S3 bucket through the public internet instead.

-   Let's make sure our public subnet has a route to our endpoint.
-   Select **Endpoints** from the left hand navigation panel.
-   Select the checkbox next to our endpoint, and select **Route tables**.
-   Select **Manage route tables**.

![image alt](Networking-179)

-   Select the checkbox next to our public route table.
-   Select **Modify route tables**.

![image alt](Networking-180)

-   Head back to the **Subnets** page.
-   Select the refresh button next to the **Actions** dropdown.
-   Check the **Route table** tab for our public subnet again.

![image alt](Networking-181)

Nice! there's a route to the VPC endpoint now.

To validate our work, let's get our EC2 instance to interact with our S3 bucket one last time.

-   Back in our **EC2 Instance Connect** tab, run `aws s3 ls s3://nextwork-vpc-endpoints-yourname` again.
-   WOOOHOOOO! We're back.

![image alt](Networking-182)

-   Congrats on making a successful VPC endpoint connection!

**VPC endpoints can be configured using policies too!**
-   We'll do a simple configuration - switch to our **Endpoints** tab.
-   Select the checkbox next to our policy.
-   Select the **Policy** tab.
-   Ooo! By default, our VPC endpoint allows access to all AWS services.
-   Select **Edit policy**.
-   Change the line **"Effect": "Allow"** to `"Effect": "Deny"`!

![image alt](Networking-183)

-   Click **Save**.
-   Switch back to our EC2 Instance Connect tab.
-   Run `aws s3 ls s3://nextwork-vpc-endpoints-yourname` - what do we see now?
-   Wow! The endpoint policy now stops us from accessing our S3 bucket.

![image alt](Networking-184)

This is a great tool to quickly block off access if we suspect attackers are using our endpoint to access our resources! Instantly switch the effect to Deny, and our Gateway is closed. We could also use VPC endpoint policies to be even more granular with the way we give away access to our AWS services. For example, we can write a policy that gives our endpoint read access only to S3 buckets, so the permission to upload objects is denied.

-   Switch tabs back to our **Endpoints** page.
-   Select **Edit Policy**.
-   Change the policy back to `"Effect": "Allow"` - this will be important when it comes to deleting our resources later!
-   Select **Save changes**.

![image alt](Networking-185)

**Step - 4 : Delete Our Resources**

Keeping track of our resources, and deleting them at the end, is absolutely a skill that will help us reduce waste in our account.

**Delete our S3 Buckets**
-   Head to our **S3** console.
-   Select the **Buckets** page.
-   Select our **nextwork-vpc-endpoints-yourname**, and select **Delete**.
-   Enter our bucket name, and select **Delete bucket**.
-   Oh no!

![image alt](Networking-186)

-   Deleting our bucket from the console is not possible - don't forget, our S3 bucket policy literally denies everyone else (**except** traffic from the VPC endpoint) access to the bucket.
-   Our only option is to delete our bucket through the EC2 instance!
-   Switch tabs to our instance's terminal.
-   Run the command `aws s3 rb s3://nextwork-vpc-endpoints-yourname`.

`rb` is a command that deletes our bucket! This means `rb s3://nextwork-vpc-endpoints-yourname` is used to delete our S3 bucket nextwork-vpc-endpoints-yourname.

![image alt](Networking-187)

-   Nope! The delete failed - we need to make sure our bucket is empty first.
-   To delete everything in our bucket, run the command `aws s3 rm s3://nextwork-vpc-endpoints-yourname --recursive`

`rm` stands for "remove" and is used to remove things inside our bucket. `--recursive` means the remove command should include all objects and subdirectories inside our bucket. This is a super helpful command to run before delete a non-empty bucket.

![image alt](Networking-188)

-   Run the command `aws s3 rb s3://nextwork-vpc-endpoints-yourname` again.

![image alt](Networking-189)

-   That's our S3 bucket successfully removed.
-   Let's check it's gone by running `aws s3 ls` one last time - does our bucket still show in the list of results?

![image alt](Networking-190)

**Delete our EC2 Instance**
-   Head back to the **Instances** page of our EC2 console.
-   Select the checkboxes next to **Instance - NextWork VPC Endpoint**.
-   Select **Instance state**, then select **Terminate Instance**.
-   Select **Terminate**.

**Delete our VPC Endpoint**
-   Head back to the **Endpoints** page of our VPC console.
-   Select the checkbox next to our endpoint.
-   Select the **Actions** dropdown.
-   Select **Delete VPC endpoints**.

**Delete our VPCs**
-   Select **Your VPCs** from our left hand navigation panel.
-   Select **NextWork-vpc**, then **Actions**, and **Delete VPC**.
-   Type `delete` in the text box and click **Delete**.
-   Note: if we get stopped from deleting our VPC because **network interfaces** are still attached to our VPC - delete all the attached network interfaces first!

Other network components should be automatically deleted with our VPC, but it's always a good idea to check anyway.

**Delete our IAM Access Keys**
Head to our **IAM** console.
Select **Users**.
Select our **Security credentials** tab.
Scroll down to our **Access keys** panel and select the **Actions** drop down.
Select **Delete**.
At the popup panel, select **Deactivate**, and enter our **access key ID** into the text field.
Select **Delete**.
Last but definitely not least - don't forget to delete the local access key **.csv file** saved on our local computer!

![image alt](Networking-191)
