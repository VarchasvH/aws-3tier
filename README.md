# Designing a Three-Tier Architecture in AWS

A Three-Tier Architecture is one that is segmented into three parts:
* **Presentation Layer (Front-End)** - Acts as the UI and handles the user interaction.
* **Application Layer (Back-End)** - Responsible for Data processing and business logic.
* **Data Layer (DataBase)** - Manages data storage and data retrival.

This Architecture promotes the seperation of concerns and facilitates easy updates or modification to specific layers without impacting others. 

It is a shift from the monolithic way of building an application where the frontend, the backend and the database are both sitting in one place.
## AWS Services used
* EC2 (Elastic Computer Cloud)
* Auto Scaling Groups
* VPC (Virtual Private Cloud)
* ELB (Elastic Load Balancer)
* Security Groups
* Internet Gateway
---
## Visual Representation of the Architecture
![Link to the structure](https://miro.medium.com/v2/resize:fit:828/format:webp/1*PDHya_zt_n657nYUAm1OeA.jpeg)

---
## What we are trying to acheive?
1. **Modularity:** It helps us manage each part of the application independently and it also helps us recover quickly from an unexpected disaster by foucsing solely on the faulty part.

2. **High Scalability:** Every tier of this architecture can scale automatically when the traffic and number of requests are increased. This can easily done by using Auto Scaling groups as they scale up or down according to the network traffic and demand.

3. **High Availability:** A infrastructure that is in multiple regions is highly available as if there is a earthquake in the location of one region, other regions can still be active and that results in better uptime of the application.

4. **Security:** We want an infrastructure that is secure from hackers and others with malicious intents. Therefore, our application will be inside a private subnet in the VPC and to access it we are gonna use a bastion host for remote SSH and a NAT gateway.

5. **Fault Tolerant:** We want our infrastructure to adapt to any unexpected changes to the traffic and fault. To acheive this, we are gonna use a redundant system.

---

# Step-by-Step Tutorial

### Step-1: Setup the VPC
VPC stands for Virtual Private Cloud (VPC). It is a virtual network where you create and manage your AWS resource in a more secure and scalable manner.
1. Click on **Create VPC** button.
2. Give your VPC a name and a CIDR block of 10.0.0.0/16.

### Step-2: Setup the Internet Gateway
The Internet Gateway allows communication between the EC2 instances in the VPC and the internet.
1. Click on **Create internet gateway** button and Give it a name.
2. Select the internet gateway and Click on the **Actions** button.
3. Select the **Attach to VPC** option and attach your VPC to it.

--- 


### Step-3: Create 4 Subnets
The subnet is a way for us to group our resources within the VPC with their IP range. A subnet can be public or private. EC2 instances within a public subnet have public IPs and can directly access the internet while those in the private subnet does not have public IPs and can only access the internet through a NAT gateway.

Search for **Subnets** and create 4 subnets as follow:

* demo-public-subnet-1 | CIDR (10.0.1.0/24) | Availability Zone (us-east-1a)
* demo-public-subnet-2 | CIDR (10.0.2.0/24) | Availability Zone (us-east-1b)
* demo-private-subnet-3 | CIDR (10.0.3.0/24) | Availability Zone (us-east-1a)
* demo-private-subnet-4 | CIDR(10.0.4.0/24) | Availability Zone (us-east-1b)

--- 


### Step-4: Create two Route tables
Route tables is a set of rule that determines how data moves within our network. We need two route tables; private route table and public route table. The public route table will define which subnets that will have direct access to the internet ( ie public subnets) while the private route table will define which subnet goes through the NAT gateway (ie private subnet).
1. Navigate over to **Route Tables** and **Click on route table** button.
2. Name these Route tables as public and private accordingly:
    * demo-public-RT
    * demo-private-RT
3. Select a RT and choose the **Subnet Association** tab.
4. Go to **Edit Subnet Associations** and add public subnets to the public RT and same for private subnets and RT.
5. Now, Select Public RT and choose the **Routes** tab and add the **Internet Gateway** rule.

--- 


### Step-5: Create the NAT Gateway
The NAT gateway enables the EC2 instances in the private subnet to access the internet. The NAT Gateway is an AWS managed service for the NAT instance.

Please ensure that you know the Subnet ID for the demo-public-subnet-2. This will be needed when creating the NAT gateway.
1. Go the **NAT Gateways** page and click on **Create NAT Gateway**.
2. Choose the subnet and click on **Create a NAT Gateway**.
3. Now select the private RT and click on **Edit Routes**.
4. Add the nat rule and Click on **save routes**.

--- 


### Step-6: Create Elastic Load Balancer
From our architecture, our frontend tier can only accept traffic from the elastic load balancer which connects directly with the internet gateway while our backend tier will receive traffic through the internal load balancer. The essence of the load balancer is to distribute load across the EC2 instances serving that application. If however, the application is using sessions, then the application needs to be rewritten such that sessions can be stored in either the Elastic Cache or the DynamoDB.
1. Search for **Load Balancer** page and click on **Create Load Balancer**.
2. Select the Application Load Balancer and Click on the Create button.
3. Configure the Load Balancer with a name. Select internet facing for the load balancer that we will use to communicate with the frontend and internal for the one we will use for our backend.
4. Under the Availability Zone, for the internet facing Load Balancer, we will select the two public subnets while for our internal Load Balancer, we will select the two private subnet.
5. Under the Security Group, we only need to allow ports that the application needs. For instance, we need to allow HTTP port 80 and/or HTTPS port 443 on our internet facing load balancer. For the internal load balancer, we only open the port that the backend runs on (eg: port 3000) and the make such port only open to the security group of the frontend. This will allow only the frontend to have access to that port within our architecture.
6. Under the Configure Routing, we need to configure our Target Group to have the Target type of instance. We will give the Target Group a name that will enable us to identify it. This is will be needed when we will create our Auto Scaling Group.

--- 


### Step-7: Create Launch Configuration
1. Click on **Create Launch Configuration**.
2. Choose a AMI or create your own AMI.
3. Choose an appropriate instance type (t2.micro for free tier).
4. Name the Launch configuration and copy the commands from Advanced Details -> User Data.
5. Again under the security group, we want to only allow the ports that are necessary for our application.
6. Click on Create Launch Configuration.

--- 


### Step-8: Create Auto Scaling Groups
1. Click on **Create Auto Scaling Group**
2. Make sure your Group looks like this:


![asg](https://github.com/VarchasvH/aws-3tier/assets/100064742/ddd45eda-7bd4-4cc6-b455-8f7510eecb2d)

3. Under the Configure scaling policies, we want to add one instance when the CPU is greater than or equal to 80% and to scale down when the CPU is less than or equal to 50%. Use the image as a template.


![igs](https://github.com/VarchasvH/aws-3tier/assets/100064742/14126640-9843-4205-b10c-848ab5864e5a)

![dsg](https://github.com/VarchasvH/aws-3tier/assets/100064742/5bb1e2f0-2d21-485e-9b94-bb5b6b95cd11)

4. Click on **Create Auto Scaling Group** button.


--- 

### Step-9: Bastion Host
The bastion host is just an EC2 instance that sits in the public subnet. The best practice is to only allow SSH to this instance from your trusted IP. 

To create a bastion host, navigate to the EC2 instance page and create an EC2 instance in the demo-public-subnet-1 subnet within our VPC. Also, ensure that it has public IP.


![bh1](https://github.com/VarchasvH/aws-3tier/assets/100064742/30b782d6-3579-4747-a130-3d9267e5d080)

![bh2](https://github.com/VarchasvH/aws-3tier/assets/100064742/c0d6fc27-3df8-483e-afcc-c23527b2bed5)

We also need to allow SSH from our private instances from the Bastion Host.

---

### Step-10: Deleting all the resources
you need to stop and delete all the resources such as the EC2 instances, Auto Scaling Group, Elastic Load Balancer etc you set up. Otherwise, you get charged for it when you keep them running for a long.
















































 





