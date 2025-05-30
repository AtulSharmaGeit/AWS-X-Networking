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
