# AWS X Networking

## Project Overview
In this project, you’ll build an isolated network on AWS using a VPC. Key tasks include configuring subnets and attaching an Internet Gateway.

---

##  Table Of Content
-  [Build a Virtual Private Cloud](#build-a-virtual-private-cloud)
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
