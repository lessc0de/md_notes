# [Virtual Private Cloud (VPC) Best Configuration Practices](http://blog.flux7.com/blogs/aws/vpc-best-configuration-practices)

## 1. Create VPC

CIDR
  * When you create a VPC, specify the __CIDR__ (Classless Inter-Domain Routing) block, for example (`10.0.0.0/16`).

Tenancy
  * Dedicated tenancy runs instances on single-tenant hardware. This costs $2/hr.


## 2. Create Subnet

Each subnet should reside in a different availability zones (__AZ__). This ensures geographic fault tolerance.

Create subnets for each __tier__ (e.g. ELB, EC2, RDS). Each tier should have at least 2 subnets across AZs. Choose the same AZs for all tiers b/c network traffic within the same AZ is free.
  * Example:
	| Tier | Subnet 1 | AZ 1 | Subnet 2 | AZ 2 |
	| ---- | -------- | ---- | -------- | ---- |
	| App | `10.0.1.0/24` | `us-east1b` | `10.0.2.0/24` | `us-east-1d` |
	| ELB | `10.0.51.0/24` | `us-east1b` | `10.0.52.0/24` | `us-east-1d` |
	| RDS | `10.0.11.0/24` | `us-east1b` | `10.0.12.0/24` | `us-east-1d` |

Subnets are __private__. Instances in the subnet cannot be accessed from the Internet. These instances don't have a public IP unless you assign an EIP.


## 3. Create Internet Gateway

Just as instances in subnets cannot be accessed via a public IP (unless you assign an EIP), they also cannot communicate with the internet. You need to first attach an internet gateway (`igw`) to the VPC first.


## 4. Create Route Tables

A __route table__ contains a set of rules, called __routes__, that determine where network traffic is directed.

Each subnet must be associated with a route table. However, you can associate multiple subnets with a single route table.
  * Example:
  
    App tier/subnet
      * Associate `10.0.1.0/24` and `10.0.2.0/24` with a route table
    
    ELB tier/subnet
      * Associate `10.0.51.0/24` and `10.0.52.0/24` with a route table
    
    RDS tier/subnet
      * Associate `10.0.11.0/24` and `10.0.12.0/24` with a route table

Creating a VPC automatically also creates a __main route table__ which, by default, enables the instances in your VPC to communicate with one another. Add a route here to allow your VPC to reach the internet:
  * Destination: `0.0.0.0/0`
  * Target: `<id of your igw>`


## 5. Create AWS Security Groups

Create separate security groups (__sg__) for your tiers and NATs (NATs will be explained later)

Example:
1. APP_SG01
2. NAT_SG01
3. ELB_SG01
4. DB_SG01

Allow inbound rules for your tiers to suit your needs. We'll address NAT security group rules later.


## 6. Create a NAT instance

Previously, we created an internet gateway and added a `0.0.0.0/0` destination to it, but this still isn't enough to allow the VPC to reach the internet. We need to use a Network Address Translation (__NAT__) instance, which enables instances in the private subnet to initiate outbound traffic.

1. Create a subnet with a mask of `10.0.0.0/24` (which is similar to the VPC's `10.0.0.0/16` CIDR block, but  smaller). We'll call this subnet __public__ (the other subnets, of course, are private).

2. Associate this subnet to the __main route table__.

3. Search for `ami-vpc-nat` community AMI. Launch after choosing
    * the previously created VPC
    * the public subnet
    * an optional EIP
    * the security group previously created for the NAT

4. Refer to [Selecting a NAT Instance Size on EC2](https://www.azavea.com/blog/2015/01/05/selecting-a-nat-instance-size-on-ec2/) for choosing an instance type.

5. Once launched, disable "Source/Destination" checking. Refer to [these instructions](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck).

6. Add inbound rules for the NAT sg:
    * Example
      * HTTP for your App subnet
      * HTTPS for your App subnet
      * _RDS/ELB remain private_

7. Go back to the route table of the __App subnet__ and add the following route:
    * Destination: `0.0.0.0/0`
    * Target: `<instance id of your NAT>`

---
Notes

* Technically there is no difference between a public or private subnet. For clarity, we call publicly accessible instances "public subnets."

* You can also create separate route tables for the public subnet to associate to. If you do, remember to add a route that will allow internet traffic into this subnet (step 4).

* You could also create a NAT for each AZ so the NAT won't be the single point of failure.

* NAT gives outbound internet access. EIP gives inbound (i.e. public address) and outbound (like NAT) internet access. The advantage of NAT is that not all machines need an EIP. EIP opens your private machien to the public, so it's more work to properly secure it.


## 7. Create App Servers

Create app instances across your app server subnets.

If your app server needs to be reached by the internet (of course), give it a public IP address (or EIP). See [What's the difference between a public IP and an elastic IP in AWS EC2?](https://www.quora.com/Whats-the-difference-between-a-public-IP-and-an-elastic-IP-in-AWS-EC2)

How do you access a private instance from outside the VPC? You can't ordinarily, but to do so, you'll need a __bastion box__ in the public subnet. You can use a NAT instance as the bastion server (aka a __jump box__). Just log into the bastion first, then log into any instance in the private subnet. Refer to the detail instructions in [Securely Connect to Linux Instances Running in a Private Amazon VPC](https://aws.amazon.com/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/).


## 8. Create RDS

1. Navigate to Services -> RDS.
2. Go to Subnet Groups in the navigation pane and click "__Create DB Subnet Group__."
3. Select the VPC ID from the drop down menu.
4. Select "__Availability Zone__" and choose the correct Subnet IDs (e.g. `10.0.11.0/24` and `10.0.12.0/24`). Then click "__Add__."
5. Click "__Yes, Create__" to create the subnet group.
6. Launch an RDS instance within the subnet group created above.
7. In the Additional Config window, select the VPC and DB Subnet Groups created previously.
8. To make sure that your RDS instance is launched in the correct subnets, select the `mydb-subgroup01` subnet group.
9. All other steps for creating an RDS are as usual.


## 9. Create ELB

The ELB will be the frontend and will be accessible from the internet, which means that the ELB should be launched in the public subnets (e.g. `10.0.51.0/24` and `10.0.52.0/24` will be public).

As before, select the route table associated with the ELB tier and add the following route:
* Destination: `0.0.0.0/0`
* Target: `<id of your igw>`

Create a load balancer inside your VPC. The subnets to be balanced should be the __app subnets__.

Remember to enable the __App sg__ inbound ports to the ELB sg so that the ELB can route traffic to backend app servers. Also make sure that the __ELB sg__ HTTP and HTTPS ports are publicly accessible (`0.0.0.0/0`)
