# Creating Networking Resources in an Amazon VPC

## Scenario

Today, we’re going to build a fully functional Virtual Private Cloud (VPC) in AWS.
The idea is to create a public subnet, connect it to an internet gateway and then EC2 instance that can actually reach the internet.
I’ve seen  some setups where everything looks right on the surface, but something small like a missing route or security group rule breaks connectivity. 
So, let's get hands-on and really understand how the networking pieces fit together — not just to make it work, but to know why it works when it does.

---

## Objectives

- Understanding architecture and issues at hand. 
- Building and configuring a complete, routable VPC Internet Gateway, Route Table, Security Group, Network Access List, and EC2 instance to create a routable network within the VPC  
- Validate our setup by pinging `google.com` from the instance

---

## Tools & Services Used

- Amazon VPC  
- EC2 (Amazon Linux 2023)  
- Subnetting   
- Internet Gateway  
- Route Tables  
- Network ACLs  
- Security Groups  
- SSH (Putty / Terminal)

---

## Network Design

| Component          | Configuration             |
|--------------------|---------------------------|
| VPC CIDR Block     | `192.168.0.0/18`          |
| Public Subnet      | `192.168.1.0/26`          |
| Internet Gateway   | `IGW test VPC`            |
| Route Table        | Public, 0.0.0.0/0 → IGW   |
| NACL               | All traffic allowed (in+out) |
| Security Group     | SSH (22), HTTP (80), HTTPS (443) |
| EC2 Public IP      | Enabled                   |

---

![alt text](<Screenshots/1 Brock vpc architecture.png>)
This our VPC Architecture


## In the scenario, Brock, the customer requesting assistance, has requested help in creating resources for his VPC to be routable to the internet. Keep the VPC CIDR at 192.168.0.0/18 and public subnet CIDR of 192.168.1.0/26.


# Task 1: Creating Necessary Tools Needed

Once in the AWS console:

## Creating the VPC :`192.168.0.0/18`

In the services menu Navigate to the Network and Content Delivery  select the VPC.

On the VPC page select Create VPC.

![alt text](<Screenshots/2 create vpc.png>)

Name the VPC: (You can name it anything) mine will be ' Test VPC'

IPv4 CIDR block: 192.168.0.0/18

Leave everything else as default, and select Create VPC.

![alt text](<Screenshots/3 TEST VPC.png>)

## Created a public subnet : `192.168.1.0/26`

On the left navigation pane select Subnets. In the top right corner, select Create subnet.

![alt text](<Screenshots/4 create a subnet.png>)

Create subnets for the VPC you created- Test VPC

![alt text](<Screenshots/5 create a subnet.png>)

Name the subnet- Public Subnet(Optional)
IPv4 subnet CIDR block - 192.168.0.0/28

choose Create Subnet

A green message will pop up displaying 'you have successfully created 1 subnet: subnet details...'


## Create Route Table

Now navigate to the left navigation pane, and select Route Tables. In the top right corner select Create route table.

![alt text](<Screenshots/6 create route tables.png>)
  
  Name your Route Table.
  Assign it to a VPC (Test VPC)
  Simple like that no drama.


## Create Internet Gateway and attach Internet Gateway

From the left click Internet gateway. Selecting Create internet gateway at the top right corner. 

![alt text](<Screenshots/7 Internet gateway setting.png>)

Simple right? We aren't finished yet. Our Internet gateway is not attached to our VPC. Let's do it a quick one. Select Actions at the top right corner and clicking Attach to VPC.

![alt text](<Screenshots/9 attached.png>)


## Add route to route table and associate subnet to route table

Navigate to the Route Table section on the left navigation pane. 
Select Public Route Table, and the scroll to the bottom and select the Routes tab. 
Select the Edit routes button located in the routes box.

On the Edit routes page, the first IP address is the local route and cannot be changed.

Select Add route.

    In the Destination section, type 0.0.0.0/0 in the search box. This is the route to the IGW. You are telling the route table that any traffic that needs internet connection will use 0.0.0.0/0 to reach the IGW so that it can reach the internet.
    Also Click in the Target section and select Internet Gateway since you are targeting any traffic that needs to go to the internet to the IGW. Once you select the IGW, you will see our  IGW Test VPC appear. Select that IGW, navigate to the bottom right.
    
    Select Save changes.

![alt text](<Screenshots/11 editing routes.png>)

From the Public route table dashboard, select the Subnet associations tab. Select the Edit subnet associations button.
![alt text](<Screenshots/12 EDIT SUB NET.png>)

Note: Every route table needs to be associated to a subnet. You are now associating this route table to this subnet.

![alt text](<Screenshots/13 EDITTED SUBNET ASSOCIATION.png>)


## Creating a Network ACL

What are Network Access Control list?

So, NACLs are like a set of traffic rules that work at the subnet level — kind of like a security guard for a whole neighborhood, not just one house. They're similar to security groups, but with some key differences.
Here’s the thing:
- Every NACL must be linked to a subnet — otherwise, it won’t do anything.
- NACLs are stateless, which means if you allow traffic in, you also have to allow it out manually (it doesn’t remember anything).
- Rules are checked in order, starting from the lowest number. For example, rule #10 is checked before #100. First match wins.
- You define what type of traffic it is (like HTTP or SSH).
- Then the protocol — usually TCP, UDP, or just "all".
- Set a port range — like port 22 for SSH, or 80 for HTTP.
- For outbound rules, you also pick a destination (like 0.0.0.0/0 for internet).
- Finally, you choose if that traffic should be allowed or denied.
Honestly, they’re a bit more strict than security groups, and easier to mess up if you forget to allow something both ways. But once you  get the logic, it's not too bad.


From the left navigation pane, select Network ACLs. Navigate to the top right corner and select Create network ACL to create a Network Access Control Lists (NACLs)
![alt text](<14 create network ACL.png>)

On the Network ACLs option, from the list of ACLs select Public Subnet ACL
From the tabs below, select Inbound rules and then choose Edit inbound rules
On the Edit inbound rules, choose Add new rule and configure:
Rule number: Enter 100
Type: Choose All traffic from dropdown
Choose Save changes
Back on the  Network ACLs option, ensure that Public Subnet ACL is selected
Choose Outbound rules and then choose *Edit outbound rules
On the Edit outbound rules, choose Add new rule and configure:
Rule number: Enter 100
Type: Choose All traffic from dropdown
Choose Save changes
​ 
Inbound After creating the NACL, it will should look like the following. This indicates there is only one rule number, and is 100, that states that all traffic, all protocols, all port ranges, from any source (0.0.0.0/0) are allowed to enter (inbound) the subnet. The asterisk * indicates that anything else that does not match this rule is denied.

![alt text](<15 inbound rules.png>)

Figure: Default inbound rule configuration for NACL. This will allow all traffic from anywhere and deny anything else that does not match this rule at the subnet level.

Outbound What do you think this rule says?

![alt text](<16 Outbound rules.png>)

Figure: Default outbound rule configuration for NACL. This will allow all traffic from anywhere and deny anything else that does not match this rule at the subnet level.


## Creating a Security Group

What’s a security group again?
Think of a security group like a personal bodyguard for your EC2 instance. It decides what kind of traffic can come in and what can go out of your server.
Unlike NACLs, security groups don’t do blocking. They’re nice — they only allow stuff. If something isn’t on the allowed list, it’s just silently dropped (ignored). Also, they’re stateful, which means if something comes in, the return traffic is automatically allowed out — you don’t need to write rules for both directions like with NACLs.
By default, everything is blocked, until you tell it what to allow. You must attach it to an EC2 instance — otherwise, it does nothing.
Here’s what you can configure in a security group:
Inbound Source: Who's allowed to connect to you? Could be a specific IP, or "everyone" (0.0.0.0/0), or maybe another AWS service.
Outbound Destination: Where your server is allowed to send stuff — usually also 0.0.0.0/0 if you want it to reach the internet.
Protocol: The type of traffic — like TCP or UDP (or all).
Port Range: Which doors (ports) are open. Like 22 for SSH, 80 for HTTP, etc.
Description: Just notes for you — to remember what a rule does.

From the left navigation pane, select Security Groups. Navigate to the top right corner and select Create security group to create a security group.

![alt text](<Create security group.png>)

The completed security group is shown below. This indicates that for Inbound rules you are allowing SSH, HTTP, and HTTPS types of traffic, each of which has its own protocols and port range. The source from which this traffic reaches your instance can be originating from anywhere. For Outbound rules, you are allowing all traffic from outside your instance.

![alt text](<18 inbound and outbound.png>)

You now have a functional VPC. The next task is to launch an EC2 instance to ensure that everything works.

# Task 2: Launch EC2 instance and SSH into instance
We will be launching an EC2 instance within your Public subnet and test connectivity by running the command ping. This will validate that your infrastructure is correct, such as security groups and network ACLs, to ensure that they are not blocking any traffic from your instance to the internet and vice versa. This will validate that you have a route to the IGW via the route table and that the IGW is attached.

On the AWS Management Console, in the Search bar, enter and choose EC2 to go to the EC2 Management Console.
In the left navigation pane, choose Instances.
Choose Launch instances and configure the following options:
In the Name and tags section, leave the Name blank.
In the Application and OS Images (Amazon Machine Image) section, configure the following options:
Quick Start: Choose Amazon Linux.
Amazon Machine Image (AMI): Choose Amazon Linux 2023 AMI.
In the Instance type section, choose t3.micro.
In the Key pair (login) section, choose vockey.
In the Network settings section, choose Edit and configure the following options:
VPC - required: Choose Test VPC.
Subnet: Choose Public Subnet.
Auto-assign public IP: Choose Enable.
Firewall (security groups): Choose Select existing security group.
Choose public security group.
Choose Launch instance.

![alt text](<19 instances launched.png>)

## Use SSH to connect to an Amazon Linux EC2 instance
By Downloading PEM  file and saving it as the labsuser.pem 
Use it to connect to your instance from the CMD

# Task 3: Use ping to test internet connectivity

Ran `ping google.com`– successful internet access confirmed

![alt text](<21 ping.png>)

 The above results are saying you have replies from google.com and have 0% packet loss.

If you are getting replies back, that means that you have connectivity.

It means:
- Subnet is public 
- Route table is routing internet traffic 
- IGW is connected 
- Security group + NACL aren’t blocking anything

---

## What I Learned

- How to troubleshoot and enable internet access in a custom VPC  
- Importance of properly associating route tables and IGWs  
- How to use NACLs and security groups for layered security  
- That public IP + IGW + correct route = connectivity

---

## Explanation of Concept

- A Virtual Private Cloud (VPC) is like a data center but in the cloud. Its logically isolated from other virtual networks from which you can spin up and launch your AWS resources within minutes.
- Private Internet Protocol (IP) addresses are how resources within the VPC communicate with each other. An instance needs a public IP address for it to communicate outside the VPC. The VPC will need networking resources such as an Internet Gateway (IGW) and a route table in order for the instance to reach the internet.
- An Internet Gateway (IGW) is what makes it possible for the VPC to have internet connectivity. It has two jobs: perform network address translation (NAT) and be the target to route traffic to the internet for the VPC. An IGW's route on a route table is always 0.0.0.0/0.
- A subnet is a range of IP addresses within your VPC.
- A route table contains routes for your subnet and directs traffic using the rules defined within the route table. You associate the route table to a subnet. If an IGW was on a route table, the destination would be 0.0.0.0/0 and the target would be IGW.
- Security groups and Network Access Control Lists (NACLs) work as the firewall within your VPC. Security groups work at the instance level and are stateful, which means they block everything by default. NACLs work at the subnet level and are stateless, which means they do not block everything by default.

---

## References

- [AWS VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)  
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)  
- [Network ACLs vs Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

---


## Author

Kelvin Mwangi  
GitHub: [@MwangiKelvin1](https://github.com/MwangiKelvin1)  
LinkedIn: [linkedin.com/in/mwangikabuchu](https://linkedin.com/in/mwangi-gitimu)

