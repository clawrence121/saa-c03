# IAM, Accounts and Organisations

## IAM Identity Policies

Set of security statements, Allows or Denies access to aws services and resources

JSON format, with one or more statements

Sid => Statement Id, human readable to describe the statement

Actions are `service:operation` eg `s3:ListObjects`, lists of actions to allow or deny

Resources are specific aws resource to allow or deny, use the ARN

Effect is Allow or Deny only

For overlapping statements, both of them are processed

First priority => Explicit Deny, overrides everything else

Second priority => Explicit Allow, the deny will always override

Third priority => Default Deny (implicit) by default everything is denied

"Deny, Allow, Deny"

If a users has multiple policies, policies from a group, and resource policies, AWS brings them all together than follows the above

Inline policy => giving a small JSON policy to every identity, good for special or exceptions

Managed policy => create the policy and attach to identities, reusable, low management overhead, should be the default

## IAM Users and ARNs

IAM users are an identity used for anything requiring long term aws access, eg humans, applications or service accounts

Principle => identity looking to access aws, has to authenticate to do anything

Username & Password or Access Keys

Then becomes an Authenticated Identity, this is what the Authorization will happen against by checking policies

ARNs are Amazon Resource Names, can identify resources uniquely globally

arn:partition:service:region:account-id:resource-{id|type{:|/}resource-id}

arn:aws:s3:::catgifs => just the bucket
arn:aws:s3:::catgifs/* => objects in the bucket

`:::` don't need to specify as s3 is globally unique, only use when doesn't NEED to be specifified

**5000 IAM users per account**

IAM user can be a member of max 10 groups

## IAM Groups

IAM groups are containers for Users, they do not have credentials and you cannot log into them, no limit to number of users per group

A IAM user can be in multiple IAM groups, eg teams or projects

Groups can have Inline and Management policies attached to them

No nesting, no groups within groups

Limit of 300 groups per account, can be increased

Resource policies can refer to specific Indentities (principals), eg Users or Roles, but NOT groups as they are not a true identity

## IAM Roles - The Tech

A type of identity within IAM, different to IAM Users

An IAM user should be users by a single principle, eg a single person

IAM Roles should be used by multiple users or services

Something becomes the role for a SHORT period of time and then stop, they "assume" them

Roles have two items:
1. Trust Policy, who can assume the Role, Identities in our Account, or from outside of our account
2. Permissions Policy, a standard IAM policy document

If the ident can assume the role, they get temporary credentials for the time period

Roles are used within AWS orgs to access multiple accounts easily

Secure Token Service generates the tokens for assumed access

## When to use IAM Roles

1) For AWS Services, eg AWS Lambda

Anything that is not an identity needs access keys, and can do that through Lambda Execution Role

The runtime environment assumes the role, gets the credentials, access other services with them

When the role is assumed it provides credentials for enough time to complete the task

2) Emergency situations

EG a helpdesk team has a readonly base level of access, but need a break glass ability to get more access

Team can assume an emergency role, where it will be logged, and can have higher level of access

3) Adding to existing network

EG if you have on-prem Active Directory, or you have more than 5k people

External accounts can't be used in AWS directly, but can assume as role for access

This is Identity Federation, where users stay in the external store, but each ID can assume a role to access AWS based on the trust policy

No creds will be stored in our app, and allows people to use their existing accounts, perfect for mobile accounts

4) Cross account

Partner account can expose a role that allows principles from your account, or the whole account

## Service-linked Roles & PassRole

An IAM role linked to a specific AWS service

A set of permissions predefined by a service to interact with other services

You can't delete a service linked role until it's no longer used

Good to separate responsibilities, some people have the create service linked role permissions, some have pass role permissions to actually assign it to a service

A method to implement "role separation"

## AWS Organizations

Large businesses need multiple accounts to function, this is hard to scale

A standard account is USED to create an organisation, it becomes the management (fka master) account

Can invite other, existing, standard accounts to join the organisation, and change to being Member accounts

The Organisation Root container contains multiple Organisational Units (OUs), that then contain accounts or more OUs

Organisations consolidate billing to the management account, the organisation can consolidate reservations and volume discounts

You can create new unique accounts in an organisation as well, no invites needed

Best practice becomes using AWS roles for access to each account, with a single account (usually the management account) to manage all the identities

## Service Control Policies (SCPs)

Used to restrict AWS accounts from the organisation level

SCP is a JSON policy doc that is attached to the organisation, or one or more organisational units (OUs), or individual accounts

They inherit down the organisational tree

Management account IGNORES SCPs!!

SCPs are "account permissions boundaries", including the account root user as the account is restricted even if the user isn't

SCPs don't grant permissions, they're just boundaries/limits

Deny list:

Like Policies, SCPs are Deny All (remember inplict deny from policies) if no statements are added

By default, a FullAWSAccess statement will be added to a new SCP which will allow everything in aws for that account

Then, add deny policies to restrict specific actions/resources we don't want that account to use.

Allow list:

By default, SCPs are Deny All (remember inplict deny from policies)

Explicitly add the policies we want the account to be able to grant

Effective permissions for an account is the overlap between SCPs and IAM policies

## CloudWatch Logs

Public Service, can be used from VPC, on prem or other systems, Regional service

Store, monitor and access logging data

Built in integrations with a bunch of AWS services, or can use the unified cloudwatch agent, or through the SDKs

Can generate metrics based on metrics using a Metric Filter

Log events have a timestamp and message

Log stream is a sequence of log events from the same source, a single resource and "thing"

A log group holds a bunch of streams from different sources

Retention settings and permissions are defined at the Group

Metric Filters also work at the Log Group level

## CloudTrail

CloudTrail is an aws service that logs api calls and activities as CloudTrail Events

By default stores the last 90 days of event history for no cost

For customisations, need a Trail

Management events and Data events

Logs "Control Plane Operations"

By default only management events are logged (invoking an lambda) vs data events (access an s3 object)

CloudTrail is a regional service, though a Trail can be created for All Regions, it'll create a trail in all regions and manage it from one place

Global Services always log to us-east-1, and you need to enable global event logging if you want to capture from a trail outside

You can choose an S3 bucket to store the Trail events, and can be stored in there forever (just pay storage costs)

Or you can send to CloudWatch Logs, can be used for search or metric filters and alarms etc

You can also create an Organisational Trail for org with logging

Essentials:
- Enabled by default for 90 days
- Trails are how you configure to S3 and cw logs
- Management events only by default
- Data events need to be add specifically and adds costs
- IAM, STS, CloudFront => log as global service events, must enable in Trail
- NOT REALTIME, there is a 15 minute delay and buffers into files

## AWS Control Tower 101

Allow the quick and easy setup of multi-account environments

Orchestrates other aws services to do this, including Organisations, IAM Ident, CloudFormation, etc

Landing Zone => multi-account environment, SSO/ID Fed, Central Logging, Auditing

Guard Rails => Detect or Mandate rules and standards

Account Factory => Automate and standardises new account creation

Dashboard => single page oversite

Created from an account, which becomes the management account (like the org management account)

Typically creates 2 OUs, Foundational and Custom

In Foundational OU creates Audit account (for external audit access) and the Log Archive account (readonly backstore for logs and config)

Account Factory can create accounts in a fully automated way as needed on the fly

Uses Config and SCPs to configure guard rails

Landing Zone
- Well arched multi account env, with a Home Region eg us-east-1
- Built with Organisations, Config and CloudFormation
- Security OU with Log Archive and Audit accounts
- Sandbox OU for testing with less regid secrutiy
- Uses IAM Ident Center with SSO for ID federation
- Monitoring an notifications with CloudWatch and SNS
- End users can provision accounts with service catalog

Guard Rails
- Rules for multi account governance
- Mandatory, Stringly Recommended, Elective
- Preventative: STOP you doing things, either enforce or not enabled, eg allow or deny regions
- Dective: Clear, in violation, not enabled, eg see if things are enabled like CloudTrail within an account

Account Factory
- Automated account provisioning
- Cloud admins and end users can use
- Guard Rails are automatally added
- Account admin given to a named user, therefore people can "give themselves" admin on an account
- Account standard config
- Accounts can be closed or repurposed
- Can be tightly integrated with business SDLC
