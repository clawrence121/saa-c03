# Fundamentals

## Public vs Private Services

Public vs private is solely a networking construct, around how you access the resource.

Private services run within a VPC, and can only be accessed within that VPC, public services can be accessed directly from the internet (eg S3 urls).

VPC = Virtual Private Cloud. Isolated from eachother and from the internet. Internet access is only available if you configure it.

AWS has a Public Zone that runs between the internet and VCPs. It's a network that is connected to the public internet. Services with a public endpoint, like S3, live here.

On-prem networks can be configured to connect to a VPC via a VPN or Direct Connect.

You can also add an Internet Gateway to a VPC to allow a private VPC to access the internet. Traffic between a private VPC and an aws public resource like S3 through an internet gateway does NOT go through the public internet, but rather just into the AWS Public Zone.

Adding a public IP address to an EC2 instance in a VPC is like moving some of it into the Public Zone.

## AWS Global Infrastructure

AWS has 2 types of global deployments, Regions and Edge Locations.

A region is an area where they have a full deployments of AWS services and infra, for instance `us-east-1`. As regions are globally distributed, we can build applications that are resiliant to area outages.

An edge location is a smaller distribution point, usually just a Content Distribution Netowkr (CDN), and some types of edge compute. Useful for companies like Neetflix to store movies close to customers for high speed, low latency loading. Further location == slower. https://aws.amazon.com/about-aws/global-infrastructure/

Some services, like EC2, require you to select a region. Other services like IAM, which are global, will show Global and not allow you to choose a region.

Geographic Separation = Isolated Fault Domain.

Geopolitical Separation = Different governance. Data stays to a given region, unless you explicitly tell AWS to move it.

Location Control = Performance.

Region Code = 'ap-southeast-2'. Region Name = 'Asia Pacific (Sydney)'.

Availability Zones are isolated infra within a given region, eg for `ap-southeast-2` there are the zones a, b, c. They are fully isolated for power, data, compute, everything. EG you can place some of your severs in each AZ to get fallback and failover coverage if one AZ fails.

AZ are connected with high speed, redundant networking, and services and be placed across multiple AZ to make them resiliant.

Service Resilience is defined in 3 ways:
1. Globally Resilient - A service operates globally, and data is replicated globally across multiple regions, eg IAm and Route53.
2. Region Resilient - Operate in a single region, one set of data per region, typically will rpelicate data to multiple AZs within a region.
3. AZ Resilient - Services run within a single AZ, with no failover if the AZ fails.

As SAs, we need to keep this in mind when designing solutions.

## Virtual Private Cloud (VPC) Basics

A VPC = a virtual network inside AWS.

A VPC is within 1 account and 1 region, they are regionally resiliant services, they operate from multiple AZs within a region.

By default, they are private and isolated. By default, services within a VPC can communicate, but are isoalted from the public internet and the AWS Public Zone (unless you configure otherwise).

However, you can have 1 and ONLY 1 default VPCs per region, then you can have unlimited Custom VPCs. Custom VPCs are fully configurable end to end, and are private by default. Default VPCs are created by default from AWS, and are configured in a very specific way by AWS. efault region _can_ be removed, but ideally not.

This section focuses on the Default VPC.

- Created within an Account
- Created with a Region
- VPCs cannot communicate out, unless you specifically allow it

Every VPC is defined a VPC CIDR, this is the range of IP address that the VPC can use. Comms into the VPC must go to the CIDR, and comms out of the VPC will come from that CIDR.

Custom VPCs can have multiple CIDRs, but the Default VPC can only have one: 172.31.0.0/16

Default VPC is ALWAYS configured to have one /20 Subnet in each AZ by default. Each subnet will have a different range within the VPC, for instance:
- us-east-2a: 172.31.0.0/20. Start: 172.31.0.0. End: 172.31.15.255.
- us-east-2b: 172.31.16.0/20. Start: 172.31.16.0. End: 172.31.31.255.
- us-east-2c: 172.31.32.0/20. Start: 172.31.32.0. End: 172.31.47.255.

A VPC is resiliant by working across multi AZs with multiple zones.

Default VPC comes with:
- Internet Gateway (IGW)
- Security Group (SG)
- Network ACLs (NACL)

By default, any resource places in the Default VPC is assigned a Public IPv4 address.

## Elastic Compute Cloud (EC2)

EC2 provides compute instances, and should be the default starting point.

IAAS - Provides virtual machines (Instance As A Service). The Instance is an operating system comfigured in a specific way

EC2 is a Private service, uses VPC networking by default. EC2 instances must be created within a VPC and single subnet (by default). You can allow public access only if the VPC supports internet access.

EC2 is AZ resilient, the instance will fail if the AZ fails.

Different sizes and capabilities, like compute, storage, gpu, networking etc.

On-demand billing, with per second or per hour depending on the software on the instance. Charges for: compute, storage, software

Can use either on-host storage, of Elastic Block Store (EBS) which is network storage made available to the instance.

Instance Lifecycle => Running, Stopped, Terminated
- After provisioning, an instance will be in Running
- From Running, an instance can move to Stopped if shut down
- From Stopped to Running, eg turning off when not running
- Can be Terminated from Running or Stopped, but this is a one-way action, CANNOT be undone

When running, you pay for CPU, Memory, Disk, network. When stopped, you pay for only disk, as the storage is still allocated.

Amazon Machine Image (AMI) the the image for an EC2 instance. Can be used to create an EC2 instance, or can be created *from* an EC2 instance.

They contain:
- Attached permissions. Public = everyone allowed. Owner = implicit allow for the owner. Explicit = specific AWS accounts allowed.
- Root volume. The C drive in Windows or the `root` volume in Linux. It can contain other volumes.
- Block Device Mapping. Links the volumes that the AMI has to how they are pesented to the operating system. EG which volume is for boot, which is for data.

Connecting to EC2
- Windows => Through RDP on port 3389. Use SSH first to get the admin password to use in RDP.
- Linux => SSH on port 22

SSH connection uses an SSH Key Pair. AWS generates the pair when the instance is created, and you download and save the Private Key to connect to the instance. The Public key is placed on the instance to connect.

## S3

Public service, global platform

Data is stored in a single region at rest, all az in a region

Perfect for large dataset and files

Very economical, lots of ways to access files

Objects are the data, Bucket are the containers for objects

Objects are pretty much files, keys are like the name, value is the binary data for the object, can be between 0b to 5tb

Buckets are created in a single region, and the data is primary for that region, and the blast radius for that data is that region

bucket name has to be GLOBALLY unique, 3-63 chars, all lower casde, no underscore, can't be ip formatted

Unlimited number of objects, unlimited size

Flat structure, there are actually no folders, but they can be represented with prefixes

soft limit of 100 buckets, hard limit of 1000 buckets

Good for accessing whole files, can't be mounted as a filesystem (block storage)

Should be the default for INPUT and OUTPUT for most services

# Cloudformation

IaC templates in YAML or JSON for resources

All templates have a set of resources and is the only required part of the template

The description MUST directly follow the template format version

Metadata can control the UI for the template like ordering

Parameters are like variables for the user to add

Mappings allow you to create KV lookups

Conditions are like IF functions within a template

Outputs are a way to pass data back to users

A single resource is known as a Logical Resource, they are created within a Stack

For every Logical Resource within a Stack, a Physical Resource is created

When updating a Stack, the same actions are done to the Physical Resources

# Cloudwatch

Collects and manages operational data

Public service and can be oushed to from anywhere

Metrics, some resources send by default eg ec2 cpu, others need to be pushed from the cloudwatch agent

Logs, eg app logs

Events, aws services can generate events to listen to, or send events on a cron

Cloudwatch uses namespaces for data, eg AWS/EC2, aws namespace is reserved

Metrics are for data in a time order, the metric is the type eg CPU UTIL, then has a Dimension to identify it

A data point has a Timestamp and a Value

Dimensions are name value pairs for aws to separate the metrics

Alarms are linked to a specific metric, and can either ne OK or ALARM, where an action can be taken, eg send to SNS topic

# Shared Responsibility Model

AWS making it clear who manages what with security

AWS manages the security OF the cloud, software, compute, storage, hardware

YOU manae the security IN the cloud, data encryotion, networking and firewall, customer data, platform access management

# High-Availability vs Fault-Tolerance vs Disaster Recovery

HA aims to ensure a level of performance, usually uptime, for a higher than normal period

HA is not about user experience, it's about maximising the operational performance

Usually represented by 99.999% etc

With HA you can switch out compute with automation, user disrution is OK as it's overall uptime that is important

HA has costs to implement and manage

FT property of a system that enables it to continue operating in the event of a failure of one or more of its components

FT works through failure with no distruption to users

EG communicating to multiple servers at the same time to allow for the failure of a node

A plane is very FT as it needs to keep running until it can land and make repairs

DR is a set of policies tools or procedures to enable the recovery of continuation of vital tech following a natural or human induced disaster

EG having a spare backup location that is ready to go

Taking regular backups, and storing them offsite with logins, run testing

# Route53

Register domains and host zones, aka managed nameservers

Globall resilient and a global service

Route53 connects to all the domain registers

A zone file is a file for all the dns records for the domain, and shares that with nameservers via Nameserver Records (NS)

Hosted Zones are zone files hosted on aws managed nameservers

Can be public, or can be private and linked to a VPC

When you create a new hosted zone, it will be allocated 4 random aws nameservers, which need to be registered against the domain

# DNS Record Type

The .com zone (eg for amazon.com) contains NS records that point to the dns records for the amazon.com zone

A and AAAA records map hostnames to IP addresses, A is ipv4, AAAA is ipv6

CNAME records are similar to dns shortcuts, CNAMEs can app point to a single A record to then map to the IP address, CANME cannot point to an IP address

MX records are for email mostly, A to the ip for the mail server, MX points to the host within a zone, or a full domain outside the zone, can have a priority, lower is better

TXT records allow you to add arbitrary information to a domain, can be used to prove ownership of a zone and domain

DNS TTL is a value in seconds for dns cache, if you make changes you will likely need to wait for the ttl to expire for changes to roll out

Authoritative answers are from the hosted server, non-authoritative answers are the cached values
