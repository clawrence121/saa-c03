# Virtual Private Cloud Basics

## VPC Sizing and Structure

IP CIDR is the IP address range for the VPC

This is VERY important to decide up front

The number of IPs define the size

Try to not clash with any other external ranges

VPC tier sctructre is used for different system considerations for security and resilience

Can only be a /28 to a /16 range (16 to 65536 IPs)

Reserve at least 2 ranges per region per account that you use

Micro, small, medium, large, extra large sizes, from 216 to 65k ip address

VPC services run within subnets within a vpc, not directly in a vpc

3 is a good default with 1 spare subnets per vpc

## Custom VPCs

VPCs are regionally isolated and resiliant

Totally isolated without explicit config

Flexible, can be simple or complex

Support Default or Dedicated Tenancy (dedicated hardware to you $$)

IPv4 Private CIDR Blocks and Public IPs

Requires 1 Primary Private CIDR block at least, with min /28 range

Can use a single IPv6 /56 CIDR block

DN provided by R53, can enableDnsHostnames and enableDnsSupport

## VPC Subnets

What services run from within VPCs

AZ resilient, a subnetwork of a VPC within a particular AZ

IPv4 CIDR and is a subscet of the VPC CIDR, can't overlap other subnets

Can have a IPv6 /64 CIDR

Subnets can communicate with other subnets in the VPC

Subnet reserved addresses (always 5):
- Network (default address for the subnet eg 10.16.16.0 for a 10.16.16.0/20 network)
- Network + 1 for the VPC router
- Network + 2 Reserved for DNS
- Network +3 reserved for future user
- Broadcast address, last IP in subnet

DHCP Options set is applied to a Subnet, one at a time for a subnet, on create and change no edit

DHCP Auto Assign IP needed to make a subnet public

## VPC Routing, Internet Gateway & Bastion Hosts

IG allows for data into/out of the private subnet to the public zone

VPC router is in every vpc, available in every AZ, always at network+1 address

Moves traffic between the subnets, each subnet has a "route table"

Created with a Main Route Table by default, can only have 1 at a time

Route table looks at destination IP of packets and sends it to the right place, the more specific the route the more prio the route has

Can point to either a gateway, or "local" for the VPC, which is created by default and can't be changed, and will always take priority

Internet Gateway (IGW) is Region Resilient attached to a VPC, 0 or 1 IGW per VPC, no more

IGW can have 0 or 1 VPC connected

Runs on the border of the VPC and the Public Zone/Internet

1. Create and attach the IGW to the VPC
2. Create a customer route table and associate with subnet
3. Create default routes with the target to IGW
4. Set subnet to allocate IPv4 public addresses

Public IPv4 addresses are actually just records for the IGW that map the private IP to the Public One, the public IP is not directly attached to the EC2 instance, and it never knows about it

When sending packets, the IGW will re-write the Source/Destination as needed to route packets in and out from internal services

For IPv6 the address is always publicly routable, and will be assigned directly to the resource

Bastian allows you to enter the VPC and access other resources

## Stateful vs Stateless Firewalls

This about perspective, is it INBOUND or OUTBOUND based on whose firewall it is

EG on a server, request is INBOUND and response is OUTBOUND

Stateless firewalls see every request and response as separate things

More overhead, but more control. Need to handle empherial ports from clients

Stateful firewalls knows the response for a given requests and can link them together

Generally only have to enable the Request and the Response is auto handled

## Network Access Control Lists (NACLs)

NACLs are like a VPC firewall, each subnet has an associated NACL

Only for data in and out of a subnet

NACLs are stateless

Explicit ALLOW or DENY, processed in order, lowest number rule runs first, numbers are unique for each INBOUND/OUTBOUND list

* used for a catchall implicit deny

Must remember the request and response 443/empherial rule pairs

By default NACLs allow all, and AWS recommends using security groups, custom ACLs set deny all by default

- Stateless
- Only affects data crossing subnet boundary
- Each NACL on multiple subnets needs the info
- Can explicitly ALLOW or DENY
- No resources referenced, just IPs/CIDRs, ports and protocols
- Use SGs to allow traffic, NACLs to deny like bad actors
- Each subnet can have only 1 NACL, but 1 NACL can associate multiple subnets

## Security Groups (SG)

State-full firewall, automatically detects response traffic and allows it

No way to do an EXPLICIT deny, only allow or IMPLICIT deny

Can't block bad actors, use NACLs

Allows referencing AWS resources and security groups and ITSELF for rules

Attached to ENIs not to instances directly, usually the primary network interface for a resource

Can allow traffic into a SG that has come out of another SG, scales really well as resources are added

Allows self referencing, so can ALLOW all traffic from other resources that use the same SG attached

## Network Address Translation (NAT) & NAT Gateway

Giving a resource outgoing access to the internet only

Network Address Translation - can adjust packet source or destination IPs

Static NAT is what IGWs use

IP masquerading is the process if hiding a whole CIDR block behind a single IP

This can be used to give a whole CIDR range outgoing ONLY access to the internet (like a home network)

NAT gateway is configured in a PUBLIC subnet, so it can use public IP addresses

In the private subnet, use a route table with a default IPv4 rule to send traffic to the NAT Gateway

NAT Gateway records a Translation Table to record all IPs and route handling

Packet flow: Private Subnet Ec2 --route table-> Public Subnet NAT Gateway --route table-> Internet Gateway

- Runs in public subnet (IGW, default ipv4 assignment)
- Uses an elastic IP
- AZ resilient service (deploy one per each AZ)
- Managed, up to 45Gbps, $ duration and data volume
- Nat Gateways don't work with IPv6

If you need an EC2 to work as a NAT instance, need to disable Source/Destination Checks
