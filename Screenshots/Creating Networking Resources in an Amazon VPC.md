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








6. **Created a Network ACL** allowing all traffic (rule #100)
7. **Created a Security Group** allowing SSH, HTTP, HTTPS
8. **Launched an EC2 instance** in the public subnet with public IP
9. **SSH’d into the instance** using PEM key
10. **Ran `ping google.com`** – successful internet access confirmed

---


---

## 💡 What I Learned

- How to troubleshoot and enable internet access in a custom VPC  
- Importance of properly associating route tables and IGWs  
- How to use NACLs and security groups for layered security  
- That public IP + IGW + correct route = connectivity

---

- A Virtual Private Cloud (VPC) is like a data center but in the cloud. Its logically isolated from other virtual networks from which you can spin up and launch your AWS resources within minutes.
- Private Internet Protocol (IP) addresses are how resources within the VPC communicate with each other. An instance needs a public IP address for it to communicate outside the VPC. The VPC will need networking resources such as an Internet Gateway (IGW) and a route table in order for the instance to reach the internet.
- An Internet Gateway (IGW) is what makes it possible for the VPC to have internet connectivity. It has two jobs: perform network address translation (NAT) and be the target to route traffic to the internet for the VPC. An IGW's route on a route table is always 0.0.0.0/0.
- A subnet is a range of IP addresses within your VPC.
- A route table contains routes for your subnet and directs traffic using the rules defined within the route table. You associate the route table to a subnet. If an IGW was on a route table, the destination would be 0.0.0.0/0 and the target would be IGW.
- Security groups and Network Access Control Lists (NACLs) work as the firewall within your VPC. Security groups work at the instance level and are stateful, which means they block everything by default. NACLs work at the subnet level and are stateless, which means they do not block everything by default.

---


## 🔗 References

- [AWS VPC Docs](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)  
- [EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)  
- [Network ACLs vs Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

---

## 👤 Author

Kelvin Mwangi  
GitHub: [@MwangiKelvin1](https://github.com/MwangiKelvin1)  
LinkedIn: [linkedin.com/in/mwangikabuchu](https://linkedin.com/in/mwangikabuchu)