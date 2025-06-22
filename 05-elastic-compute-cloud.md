# Elastic Compute Cloud

## Virtualization 101

EC2 is virtualisation as a service

Process of running more than one operating system on a single system

Kernel is the only part of the operating system that can directly go to hardware

Emulated virtualisation uses software hypervisors to run multiple OS on one machine

Paravirtualisation are modified OS that call "hypercalls" to the hypervisor

SR-IOV allows add on cards to present themselves as many "smaller" cards at a hardware level, in ec2 this is called Enhanced Networking

## EC2 Architecture and Resilience

EC@ instances run on Shared or Dedicated hosts

When shared, the instances are fully isolated

Hosts run in a single AZ, so instances are AZ resiliant

EC2 instances can have multiple network interfaces for multiple subnets

EBS is network based storage and is AZ

Instances stay on a single host unless the host dies, or the instance is STOPPED and then STARTED

What's ec2 good for
- Traditional OS + application compute needed
- Long running compute needs, eg multiple years
- Server style applications
- Either burst or steady state loads
- Great for monolithic application stacks
- Migrated application workload or disaster recovery

## EC2 Instance Types

Chooses the raw resources you get, the amount and the ratio between them

Storage and data network bandwidth

System arch and the vendor, eg arm vs amd vs intel

Allitional features like GPUs and FPGAs

5 main categories
- General purpose, always start here
- Compute optimized, media processing, high end compute, machine learning
- Memory optimized, processing large in memory datasets, some databases like redis
- Accelerated Computing, like gpus or fpgas
- Storage Optimized, lots of local storage for high iops

Instance type eg R5dn.8xlarge

Instance family is the first letter eg R

Number is the generation eg 5

Size is after the . eg 8xlarge, this is how the compute is allocated

Additional capabilities are after the generation, and optional

ARM is good for efficiency

T is a burstable for workloads that have small peaks but a low base load, uses burst credits

M is the general steady state workload

C is compute optimized

## Storage Refresher

Direct (local) is storage on the ec2 host, fast, but can be lost

Network (EBS) is separate and resilient

Emphemeal storage is temp

Persistent storage is permanent and lives past the life of the instance

Categories:
- Block storage is a volume that can be mounted to the OS and is bootable, OS will add the filesystem
- File storage has a file system already, mountable, not bootable
- Object storage, not mountable not bootable, like S3

Performance:
- IOPS is like the horsepower, number of read/write operations per second
- IO block size is the size of the wheels, size of the blocks of data being written to disk
- Throughput is IOPS x Block size, rate of data can be stored

Max throughput requires matching the correct block size and IOPS

## Elastic Block Store (EBS) Service Architecture

Service that provides block storage through block ids

EC2 sees a raw device to add a file system on top of

Storage is provisioned on ONE AZ only

Generally attached to a single ec2 instance, but there are some settings for multiattach

Can be moved from one instance to another, not linked to the lifecycle of a specific instance

Snapshots can be backed up to S3 to have region resilience, or for migrating between AZs

Can be provisioned with different performance types and perf profiles

Data is replicated within an AZ

## EBS Volume Types - General Purpose

Also called GP2, General Purpose SSD

Can be from 1gb to 16tb

You create with IO credits, and that's how your usage is metered, leaky bucket algo with a refil baseline rate depending on the bucket size

100 min, plus 3 credits per second per gb of volume size

And can burst up to 3000 iops for up to a minute

All volumes get an initial 5.4m credits

For buckets over 1tb they get a baseline perf and can't drop to 0 credits

GP3 is the new type and uses a more simple rate limiting structure

3000 iops and 125 MiBs standard

20% cheaper that GP2

You pay for up to 16k iops or 1000MiBs

## EBS Volume Types - Provisioned IOPS

Called IO1 and IO2

IOPS config apart from the storage size

Up to 64k iops per volume, up to 1000mibs or 4000mibs on block express

io1 has mac 50iops per gn, io2 has 500iops per gb

There is a max performance between ec2 and ebs

EG for io1 max is 260k iops and 7.5k mibs, so you need 4 volumes to saturate that limit, io2 is a little less, block express more

## EBS Volume Types - HDD-Based

ST1 for throughput SC1 for cold storage

Good a sequential access

st1 From 125gb to 16tb, with 100mb blocks, up to 40mb/s/tb base, 250mb/s/tb burst

Good for data warehouses

sc1 is slower, but is the lowerest cost ebs available

## Instance Store Volumes - Architecture

Provide block storage devices

Physically connected to one ec2 host

Locally attached so very high performance

Included in the instance prices

Have to be attached at launch time

If an instance moves between hosts, then the data in the instance store is lost

Different instance types have difference access to instance stores
