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


# [Practical VPC Design](https://medium.com/aws-activate-startup-blog/practical-vpc-design-8412e1a18dcc#.xve35vrcp)

Let's create a standard n-tier app with web hosts that are addressable externally. We'll use `10.0.0.0/16` as our address space.

The easiest way to layout a VPC's address space is to forget about IP ranges and think in terms of subnet masks.

For example, take the `10.0.0.0/16` address space above. Let's assume you want to run across all three AZs available to you in `us-west-2` (e.g. so your Mongo cluster can achieve a reliable quorum). Doing this by address ranges would be obnoxious. Instead, you can simply say "I need 4 blocks - one for each of the three AZs and one spare." __Since subnet masks are binary, every bit you add to the mask divides your space by two. So if you need four blocks, you need two more bits. Your 16-bit becomes four 18-bits.__

```
10.0.0.0/16
    10.0.0.0/18     - AZ A
    10.0.64.0/18    - AZ B
    10.0.128.0/18   - AZ C
    10.0.192.0/18   - Spare
```

Now within each AZ, let's say you determine that you want a public subnet, a private subnet, and some spare capacity. Your publicly-accessible hosts will be far fewer in number than your internal-only ones, so you decide to give the public subnets half the space of the private ones. To create the separate address spaces, you just keep adding bits.

```
10.0.0.0/18           - AZ A
    10.0.0.0/19       - Private
    10.0.32.0/19
        10.0.32.0/20  - Public
        10.0.48.0/20  - Spare
```

Later on, if you want to add a "Protected" subnet with [NACL's](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html), you just subdivide your spare space:

```
10.0.0.0/18           - AZ A
    10.0.0.0/19       - Private
    10.0.32.0/19
        10.0.32.0/20  - Public
        10.0.48.0/21  - Protected
        10.0.56.0/21  - Spare
```

Just make sure whatever you do in one AZ, you duplicate in all the others:

```
10.0.0.0/16
    10.0.0.0/18           - AZ A
        10.0.0.0/19       - Private
        10.0.32.0/19
            10.0.32.0/20  - Public
            10.0.48.0/21  - Protected
            10.0.56.0/21  - Spare

    10.0.0.0/18           - AZ B
        10.0.0.0/19       - Private
        10.0.32.0/19
            10.0.32.0/20  - Public
            10.0.48.0/21  - Protected
            10.0.56.0/21  - Spare

    10.0.0.0/18           - AZ C
        10.0.0.0/19       - Private
        10.0.32.0/19
            10.0.32.0/20  - Public
            10.0.48.0/21  - Protected
            10.0.56.0/21  - Spare

    10.0.0.0/18           - Spare
```

Your routing tables would look like this:

```
"Public"
    10.0.0.0/16 - Local
    0.0.0.0/0   - Internet Gateway

"Internal-only" (i.e. Protected or Private)
    10.0.0.0/16 - Local
```

Create those two route-tables and then apply them to the correct subnets in each AZ. You're done.

And in case anyone on your team gets worried about running out of space, show them this table:

```
16-bit: 65534 addresses
18-bit: 16382 addresses
19-bit: 8190 addresses
20-bit: 4094 addresses
```

Obviously, you're not going to need 4,094 IP addresses for your web servers. That's not the point. The point is that this VPC has only those routing requirements. There's no reason to create new subnets in this VPC that don't need to route differently within the same AZ.

Check [this guy's VPC layout as well](http://alexschoof.com/posts/my-vpc-layout.html)
