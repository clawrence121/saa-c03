# Simple Storage Service

## S3 Security (Resource Policies & ACLs)

Bucket policies are a type of resource policy

S3 buckets are private by default

Resource pov for the policies

Identity policies allow same account only, resource policies allow access from other accounts

Can allow or deny anon principle (people who are not authed)

Resource policies have a Principal attribute to define the WHO it applies to

A * policy gives access to literally everyone

You can even do ip address validation

Only one policy per bucket, for identities merged with identity policies

Access Control Lists are now legacy, but we need to know they exist, very simple permissions

ACLs can be READ, WRITE, READ_ACP, WRITE_ACP, FULL_CONTROL, not very granular

Block Public Access is a default setting that stops public access even if you allow all principals in the policy, it's a failsafe

POWER UP

Think about who you are doing policies for

Identity: Controlling different resources

Identity: You have a preference for IAM

Identity: Same account

Bucket: Just controlling s3

Bucket: Anaon or cross account

ACLS: don't use

## S3 Static Hosting

Normal access is via aws apis

This feature allows for access via HTTP

Index and Error documents are set

It creates a static Website hosting endpoint

If you want to map to a domain, the domain must match the R53 domain exactly

Grate for offloading static assets, like images, from a compute service like ec2

Can be used for "out of bound" pages, eg hosting an error page to redirect to if your main service is down

Pricing is per gb per month, plus a data transfer fee (free in, paid out), plus a fee per (1000) requests

## Object Versioning & MFA Delete

Starts in a disabled state, if enabled, cannot be disabled, and then only mived to suspended

Cannot be disabled again

Object versioning is based on the key

When versioning is enabled an id is given to the object, then the object is updated it gets a new id

If you don't specify the id you get the latest, else you can get a specific version if id

A delete marker is a special version of the object that hides all versions

If you delete a delete marker, the latest version is restored

To delete all versions of an object in a versioned bucket you have to delete all ids

You are paying for space for all the versions of all the objects

MFA delete is something you can enable, so that it requires mfa to change versioning states or to delete versioning

## S3 Performance Optimization

Single Put Upload:
- Single data stream using put object api call
- Simple, but if the stream fails then the whole request needs to be restarted
- Speed and reliability issues if limited to 1 stream
- A single stream is a lot slower than a multistream
- Limited to 5gb of data

Multipart upload
- Data is split into smaller parts
- Min of 100mb
- Split into a max of 10000 partsm min of 5mb max of 5gb, last part of any size
- Parts can fail and be restarted
- Risk for the upload is reduced
- Transfer rate is improved as the parts run in parallel

S3 Accelerated Transfer (OFF)
- Connecting to an s3 bucket over a large distance will force your data to go over the public internet before it reaches the s3 public zone for the region the bucket is in
- This usually doesn't match the best performance route
- Transfer Acceleration will allow you to use edge locations to then route the data using the aws global network
- This is faster and more direct and more resiliant

## Key Management Service (KMS)

Used for encryption

Regional and public service

Create store and manage keys, can be sym and asym keys

Can do cryptographic operations

Keys NEVER leave KMS, and is a FIPS 140-2 L2 service

KMS keys contain a ID, date, policy, desc, state

Key material can be generate or imported

KMS keys can be used on data up to 4kb

Always stored on disc encrypted

Users always need the exact permissions for each action, eg manage keys, encrypt, decrypt

Data Encryption Keys can be used to work on data over 4kb in size

KMS does not store the DEK itself, and provides it to you to use, the cipher key is encrypted with the KMS key for later

KMS keys are isolated to a region and never leave (unless it's a multi region key)

AWS Owned and Customer Owned

Customer Owned can be aws managed or customer managed

KMS keys have backing keys so that older encrypted data can still be decrypted by newer versions of the key after rotation

Policies:
- Key Policies are a Resource policy
- Every key has a dedicated policy
- Each key doesn't trust any account by default
- Very granular
- Use iam policies to access specific key resources

## S3 Object Encryption CSE/SSE

Buckets are not encrypted, objects are!

Client Side encryption and server side encryption

This defines how data is encrypted at rest

Encryption is always encrypted in transit (with a couple of small exception)

Client side: encrypted before being sent to aws, stored as received

Server side: sent to s3 in plaintext, ecrypted when it lands in s3 by s3

You can choose based on how much trust you want to place on s3

SSE-C is server with customer keys, plaintext object plus key sent to aws, hash of key stored with object to allow for decryption, choose when you need to manage your keys but aws can do the processing

SSE-S3 is server with aws keys, plaintext object sent to s3, s3 has its own key to encrypt to save to disk, not visible to user, is a good default, you can't stop admins from accessing data, uses aes-256

SSE-KMS us server with keys stored in kms, plaintext object to s3, then kms handles the keys with kms policies to do the encryption, gives you a LOT more control, eg role separation

SSE is now mandantory, you just choose which one you want to use

## S3 Bucket Keys

By default, every object uses a unique Data Encryption Key, and a unique call to KMS, this becomes very expensive! And can be throttled

With bucket keys, kms will create a bucket key that is then used for a short period of time within the bucket to generate all the DEKs needed

Offloads the work from KMS to S3

Cloudtrail logs only show the bucket now not the object

Work with same and cross region replicated object, it will be encrypted even if not in source but is in destination, which can change the etag type

## S3 Object Storage Classes

S3 standard is the default, and is stored across at least 3 AZs for replication

11x9s of reliability, means 1 object lost every 10k years...

MD5 checksums for Cyclic Redundancy Checks

If PUT responded with HTTP/1.1 200 OK then we know our data has been stored durably

S3 pricing is data storage, transfer out and per 1000 requests

Data is available within milliseconds

Standard should be used for most frequently access data

S3 Standard IA is pretty much the same as Standard, BUT storage costs is much cheaper, but has a new retrieval fee per gb, and there is a minimum duration of 30 days and 128kb per object of fees. Should be used for important data that doesn't need to be accessed.

S3 One Zone IA is the same as standard IA, with only one AZ, used for long lived data which is non-critical and replaceable, where access is infrequent

S3 Glacier Instant, like Standard IA with lower storage costs, minimum length of 90 days, higher retrieval costs

S3 Glacier Flexible, same as instant but are "cold" and can't be made public, and you have to call a retrieval job with 3 different types, Expedidted (1-5m) Standard (3-5h) Bulk (5-12h), faster is more expensive, archival data which minute to hours

S3 Glacier Deep Archive, data is "frozen" min 180 duration, 12-48h restore time, for data that is rarely needed, good for legal retention period data

S3 Intelligent tiering, 5 tiers across frequent, IA, Archive IA, Archive Access, Deep Archive, S3 will automatically move between classes, you application needs to be able to support across tiers, has a per 1000 objects management cost

## S3 Lifecycle Configuration

Lifecycle rules transition or expire objects automatically

Set of rules of actions for a bucket or group of objects

Transition objects to a different storage tier

Expiration actions to remove objects or versions (can be complex with versions)

Transitions are like a waterfall to push items down, to pretty much all the levels below, can't happen in an upwards direction!

Be careful for moving costs and minimums

Standard or one zone ia has a minimum of 30 days before can be moved via lifecycle

If an object is moved from standard to ia, has to wait another 30 days before moving to glacier

## S3 Replication

Can replicate data between a source and destination bucket

Cross Region Replication or Same Region Replication

Can be in the same or in a different account

The config needs a destination bucket, and a role that trusts the source bucket and has write permissions to the destination

Replication is encrypted with SSL

For different accounts the role is not trusted by default by the destination, so the destination needs a bucket policy that trusts the role

Default all objects, or can filter by prefix or tags

Choose destination storage class, good for moving to an archive bucket, default is the same as the source

By default the objects will be owned by the source account, so the ownership can be overridden

Replication Time Control can be added for a 15 minute SLA if you need it

Considerations:
- Not retroactive, only new objects after it has been enabled, batch replication can be used for existing objects
- Both buckets need to have versioning on
- One way only by default, but bi-directional can be added
- Encryption can work with all SSE types and unencrypted
- Will not replicate system events, like lifecycle, and can't replicate glacier, nor delete markers (unless enabled)

Why?
- Log aggregation from multiple systems
- PROD or TEST sync
- Resilience with strict sovereignty, if the data can't be moved to another region
- Cross region for global resilience
- Region latency reduction

## S3 PreSigned URLs

Give an external person access to an object in a safe and secure way

By default, creds are checked at request time, and needed

Usually to give ext access you would need to provide an anon user with an identity, credentials or make the bucket public

S3 can generate a presigned url with who accessed, what is accesses, and when it expires

The person using it uses the url as the person who generated the url

Good for temp and one off GETs and PUTs

Presigned urls allow for applications to create short term urls for frontend users to temp access photos or videos in a media bucket, without having to make the bucket public

Exam:
- You can generate a url for an object that you don't have access to, the url won't either
- When using the url it has the same permissions as the person who created it RIGHT NOW, not when they created it
- Don't generate based on a Role, the creds and permissions of a Role will usually last for less time than the URL, so use a long term identity like an IAM user

## S3 Select and Glacier Select

Ways to retrieve part of objects rather than whole objects

S3 can store huge objects up to 5tb

You often want to retrieve the whole thing, but that takes time and data

Filtering on the client side doesn't reduce this

Select services allow you to select partial objects with SQL like statements

Faster speeds and cost reduction as only sends the data it needs

## S3 Events

Event notifications when a certain thing happens in a bucket

For event processes to happen as a result, can send to SNS, SQS, lambda

Useful for retrivals and replication notifications as well

Destination services need resource policies to receive the notifications

EventBridge is an alternative here, more recommended

## S3 Access Logs

Source bucket we watch, target where the logs go, can be done with console or api

It's a best efforts process, enabled within a few hours

Log files include newline delim log records with attributes space delim

Mostly used for access audits

## S3 Object Lock

Can only be enabled for new buckets, and required versioning

Write Once Read Many (WORM) no deletes or overrides of object versions

Individual object versions are locked

Can be 2 things, Retention period and Legal Hold, can be both, one or the other or none

Defined per version or at the bucket level default

Retention period: specify days and years, Compliance mode means no changes at all inc the root user, Governance mode can grant special permissions to bypass useful for accidental protection

Legal hold: no period just ON or OFF, needs a LegalHold permission to make changes, eg to mark an object as critical for a period like a legal case

## S3 Access Points

Simplify managing access to s3 buckets

Rather than 1 bucket with 1 massive resource policy, create many access points each with a different policy and different network access controls, have to give the access point permissions on the bucket policy

Each access point can restrict access to objects within the bucket as well

Can be created from the console or the cli

Access points can also be used to restrict access to a specific VPC
