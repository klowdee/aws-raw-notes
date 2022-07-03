# SAA - Solutions Architect Associate Notes

## Kinesis

- Send streaming data to Kinesis and do stuff with it
- Ingests and analayzes data in real time
- Extreme scalability
- An AWS managed service

---

## Storage

### EC2

- *Emphemeral* storage - is destroyed once an instance is stopped

### S3

- Object stroage
- Highly fault tolerant
    - Copies of data stored across multiple AZ’s
- 99.99% availability
- **11 9’s** durability (99.999999999%)

### EBS

- 99.99% availability
- 99.999% durability with **io2** volumes
    - Other volumes are rated 99.8-99.9% durable
    - Snapshosts can be used to provide backup points (to increase durability).  Snapshots stored in S3.
- Highly

### EFS

- Cloud native NFS
- Can provide parallel shared access to thousands of EC2 instances
- Stored across multiple AZ’s and accessed concurrently in all of a Region’s AZ’s
- 99.99% availability

---

## IPs

### IPv4

- Most common format used online
- 3.7 billion different addresses in public space
- Ex: `1.160.10.240`

### IPv6

- AWS supports IPv6
- Common for IoT
- Supports 340 trillion trillion trillion unqiue IP addresses
- Ex: `3ffe:1900:4545:3:200:f8ff:fe21:67cf`

### Public IP

- Can be identified on the internet
- Must be unique across the entire internet
- Can be geo-located

### Private IP

- Can only be identifed on a private network
- Must be unique across private network
- Two different private networks can have the same IPs
- Conntects to the internet via a NAT device + IGW
- Only specifed range of IPs can be used as private IPs

### Elastic IPs

- Fixed public IPv4 IP address for an EC2 instance
- Can be attached to one istance at a time
- You own the IP as lon gas you don’t delete it
- Can be used to mask instance/software failure by remapping the address to another instance
- You can only have 5 Elastic IP in your AWS account (but can ask AWS to increase the amount)
- Using Elastic IP often reflects bad architectual designs
    - Preferable to use random public IP with a registered DNS name

### Misc

- By default, EC2 instances come with a private IP for the internal AWS network, and a public IP for the internet
- When you SSH to an EC2 instance you must use the public IP (because a private IP wouldn’t work because it’s not the same network)
- If an EC2 instance stops and is restarted, the public IP can change (if it doesn’t have an associated Elastic IP)

---

## Placemnet Groups

- Strategies for EC2 instance placement

### Cluster

- All EC2 instance are on the same rack in the same AZ
- Good for applications that need very low latency and high network throughput (Big Data jobs)
- Pros
    - Low latency
    - 10Gbps bandwidth between instances
- Cons
    - If rack fails, all instances will fail at the same time

### Spread

- Opposite of Cluster
- Minimizes failure risks
- All EC2 instances are on different hardware racks
- Good for critical apps that need to maximize high availability
- Pros
    - Can span across AZs
- Cons
    - Limited to 7 instances per AZ per Placement Group

### Partition

- Instances in a partition do not share racks with instances in other partitions
- Can span across AZs
- EC2 instances get access to the partition info as metadata
- Good for Big Data - HDFS, HBase, Cassandra, Kafka

---

## Elastic Network Interfaces (ENI)

- Represents a **virtual network card**
- Bound to a specific AZ
- Can create an ENI independantly and attach/move them on EC2 instances for failover
- An ENI can have:
    - Primary private IPv4, one more secondary IPv4
    - One Elastic IP (IPv4) per private IPv4
    - One public IPv4
    - One or more Security Groups
    - A MAC address

---

## EC2 Hibernate

- In-memory (RAM) state is preserved
- Instance boot is very fast
- Under the hood the RAM state is written to a file in the root EBS volume (the EBS volume ***must*** be encrypted)
- Good for services that take a long time to boot/initialize
- Supports many instance families
- Supports large RAM size (up to 150 GB)
- Does not support bare metal instances
- Supports many AMI
- Available for On-Demand, Reserved, Spot instances
- ***Cannot*** have an instance in hibernation more than 60 days
- 

---

## Advanced EC2 Concepts

### EC2 Nitro

- New virtualization technology
- Allows better performance
- Better networking capabilities (HPC, IPv6)
- Higher speed EBS (Nitro required for 64,000 EBS IOPS - max 32,000 on non-Nitro)
- Better security

### vCPU

- CPU cores time threads per CPU.
    - Ex: 4 CPU, 2 threads per CPU = **8 vCPU**
- With multithreading, each thread is represented as a virtual CPU (vCPU)
- You can tweak/optomize
    - \# of CPU cores (you can decrease to lower **licensing costs**)
    - \# of threads per core (to disable multithreading for high performance computing <HPC> workloads)

### EC2 Capacity Reservations

- Ensures you have capacity when needed
- No need for 1 or 3 year commitment
- Manual or planned end-date for the reservation
- Capacity access is immediate
- Billing begins immediately
- Each Capacity Reservation is for one AZ only (need to do multiple reservations for other AZs)
- Can combine with Reserved Instances and Savings Plans for cost savings

---

## EBS Encryption

- With an encrypted EBS volume you get:
    - Data at rest encrypted inside the volume
    - All data moving between instance and volume is encrypted
    - All snapshots encrypted
    - All volumes created from (encrypted) snapshots
- Encryption (and decryption) happen transparently with no needed actions (other than enabling)
- Has a very minimal impact on latency
- EBS Encryption **uses keys from KMS (AES-256)**

### To encrypt an unemcrypted EBS volume

- Create an EBS snapshot of the volume
- Encrypt the snapshot (using copy)
- Create new EBS volume from the snapshot (this will be encrypted too)
- Attach the encrypted volume to the original EC2 instance
- Note: you can also take any unencrypted snapshot and encrypt it 

---

## Auto Scaling Groups (ASG)

### ASG Default Termination Policy:

- Find AZ with most # of instances
- If there are multiple AZ to choose from, delete the one with the oldest launch configuration
- ASG tries to balance the number of instances across AZ’s by default

### Lifecycle Hooks

- You have ability to perform actions before an instance goes in service (while in *Pending* state)
    - Will move to a *Pending:Wait* sate and can do things like configurations, etc, and then move to *Pending:Proceed*
- You can also perform actions before an instance is terminated (*Terminating* state)
    - Moves to *Terminating:Wait* where you can perform actions
        - Good for extracting info, like logs
    - Then once done goes to *Terminating:Proceed* state to complete the termination

### Launch Template (newer) vs Launch Configuration (legacy)

- Both:
    - Used to define id of the AMI, the instance type, a key pair, Security Groups, and other parametres used when launching EC2 instances (tags, user-data, etc)
- Launch Configuration
    - Must be re-created every time
- Launch Template
    - Recommended by AWS
    - Can have multiple versions
    - Create parameter subsets
        - partial configuration for re-use and inheritance
    - Provision using both On-Demand and Spot instances (or a mix)
    - Can use T2 unlimited burst feature

---

## Ports

### Important Ports:

- FTP: `21`
- SSH: `22`
- SFTP: `22` (same as SSH)
- HTTP: `80`
- HTTPS: `443`

### RDS ports:

- PostgreSQL: `5432`
- MySQL: `3306`
- Oracle RDS: `1521`
- MSSQL Server: `1433`
- MariaDB: `3306` (same as MySQL)
- Aurora:
    - `5432` (if PostgreSQL compatible)
    - `3306` (if MySQL compatible)

---

## API Gateway

- Serverless API - AWS Lambda + API Gateway
- Support WebSocket Protocol
- Handles API versioning
- Handles different environments (QA, prod, etc)
- Handles security
- Swagger/Open API to quickly define APIs
- Transform & validate requests & responses
- Generate SDK/API specifications
- Cache API responses

- Integrates with
    - Lambda
    - HTTP
    - AWS Service
- Endpoint Types (deployments)
    - Edge-optomized (Default)
        - Routed through CloudFront Edge locations
        - Still lives in 1 region
    - Regional
        - For clients in same region
    - Private
        - Only accessed from your VPC
        - Use resource policies to access

### Security

- IAM
    - IAM Permissions to attach User/Role
    - API  Gateway verifies IAM permissions passed by the calling application
    - Good for accessing within your own infra backend
    - Leverages SigV4 capability where IAM creds are in headers
- Lambda (Custom) Authorizer
    - Uses Lambda to validate the token in header
    - Helps w/ OAuth, SAML, 3rd party auths
    - Lambda must return an IAM policy for user
- Cognito User Pools
    - Fully manages user lifecycle
    - Verifies identify automatically
    - No custom implementation/code required
    - Only helps with authentication, not authorization

## Cognito

### Cognito User Pools (CUP)

- Sign in functionality for app users
- Integrates w/ API Gateway for auth
- simple login: Username/email, password combo
- Enable Federated Identifies (Facebook, Google, SAML, etc)

### Cognito Federated Identity Pools (FIP)

- Provide AWS credentials to users so that they can access AWS directly
- Example - provide temporary access to write to S3 bucket using Facebook Login

### Cognito Sync

- Synchronizes data from device to Cognito
- Store preferences, config, state of app
- Cross device sync
- Requires FIP (not CUP)
- Deprecate and replaced by AppSync

## SAM - Serverless Application Model

- Framework for developing and deploying serverless apps
- All config is YAML code
    - Lambda functions
    - DynamoDB Tables
    - API Gateway
    - Cognito User Pools
- Can help you run Lambda, API Gateway, Dynamo locally
- 


# Architecting Accelerator

Managed services - resiliency of infrastructure 

WAF - Well-Architected Framework


## Well-Architected Framework

CloudTrail - logs every single API request


You need to adopt a consumption model.  

Managed services let you focus on your product more 




*Sustainability* is now the 6th pillar

## Global Infrastructure


Edge location can also be called PoPs (point of presence)




## Operating an AWS Service


## RECAP DAY 1

Recap Day 1:

S3 objects are stored in buckets.  It’s private by default. Need to make config changes to make it public.  
S3 - multiple classes to store data in (standard, one zone, standard IA, glacier, etc.).  Can store unlimited amount of data (max file size is 5TB)
Intelligent tiering - moving based on ML patterns
Compute: EC2 (instances - don’t call them VM on AWS).  Choose AMI, choose instance type (from family - choose CPU/RAM), networking, storage (EBS).  EBS charges you cased on the amount of storage you consume.
Security Groups - mechanism/firewall that controls what kind of traffic can come into and out of EC2 instances.
DBs: NoSQL and SQL (non-relational / relational).  RDS is managed solution for relational databases. Can save a lot of time with RDS since you don’t have to manage much.  

## Networking Part 1

### VPC


Logically isolated section of the AWS cloud.  Can think of it as your own data center. You have complete control of security. 


VPC gives you the networking layer in AWS.  It’s the virtual network that can contain instances and resources.  By default, every VPC is isolated from other networks. 


VPCs are a regional resource.  Can have multiple in a single region.  Max number of VPCs you can create is 5 by default (soft quota).  When you setup AWS all accounts get a default VPC (in each region). 

VPC dashboard is a great tool in the AWS console (something you may want to visit daily, etc.).

VPCs job is to put the resources across different AZs.  Extend VPC to other AZs with subnets. 


Having only one VPC is good for high-performance computing needs.  For most cases though you want to use Multi-VPC or Multi-Account


Good for separate stages (qa, stage, prod, etc.). 


Multi-account - can have account segregation at the account level (not VPC).  Separate by products. Requires a lot of management. 


Deploy up to 5 VPCs by default (per region).  It’s a soft quota, so if you need more you have request more from AWS. 


IP addressing is one of the trickiest part of VPCs. 

AWS needs to understand CIDR -

CIDR Block - IP range

Primary CIDR when you have a VPC


CIDR notation (slash notation): 172.16.0.0/16 = 65536 addresses.  The slash 16 is the prefix length (length of the subnet mask).  The range is /16 to /28.  Inverse relationship exists. Smaller the prefix means the larger amount of IP addresses. /28 is only 16 IP addresses. 


IPV4 - valid length /0 to /32. They are 32bits. 2^32 is the max number if IP addresses in the world.  They are almost full, so that’s why we have IPV6.
 If you don’t choose CIDR block carefully you could be in trouble in the future. 


Website you can use to do the math for you: [https://www.ipaddressguide.com/cidr](https://www.ipaddressguide.com/cidr)

### Subnets

Subnets are a zonal resource (not regional).  Subnet placement is very important. A subnet is a logical container with a VPC that holds VPC resources. Lets you isolate resources from each other. Ex. Can create one subnet for public access, and other for web resources to access.  Kind of similar to LANs/VLANs. Every instance has to exist within a subnet. Once created in a subnet, cannot move it out.  Can’t move instances from one VPC to another. 
Carve out a small CIDR block for each subnet.

You don’t get all IP addresses from AWS.  They reserve 5.

- .0 and .255 are broadcast addresses


VPCs are effectively limited to one AZ.  If VPC has a primary and secondary subnet block, CIDR can be derived from either of them.  Subnet can exist only within an AZ (small geographic location).  Can create resilience by creating subnets and spreading across zones.  


Route tables: lets subnet resources know the path to take.  For allowing Web ec2 instance to talk to the DB instance for example. 
Think of them like a traditional router. The prefix for `arn` is **rtb-xxxxxx**


IP routing is destination based.  Based on the destination, not the source IP. When a subnet can reach the internet we call it **public**.  Otherwise, it’s called **private**. 


### Gateways


Associated with subnet.  Required to make a subnet public. Lets instances get a public IP address. 
arn resource ID prefix is **igw**-xxxxx


NAT gateway: connects private subnets to internet.  Resides in public subnet. 


Always choose large subnets over smaller ones.  Public IPs should be smaller/less.  Most architecture has more private resources.



ENI:
Virtual network interface.  Can be moved across AZ. Think of it like a network card that’s in a computer. 



Elastic IP addresses - allocated by AWS.  Once it’s allocated you have exclusive access to it.  Can have a fixed EIP for each ENI. 


### Security


Security groups: firewall that creates stateful rules (ingress is coming inside).  Security groups deny everything by default. 



NACLs: contains inbound and outbound rules.  Acts like a firewall. 


Must configure both inbound and outbound.  Allows everything by default.  You define what to **DENY**.

Review:


## Networking Part 2

### Connecting networks

VPG Virtual private gateway (VGW) - connects on-premise network to Amazon VPC




With Direct Connect you bypass the internet all together.  Good for transferring large data, regulatory requirements, etc


Direct connect is a dedicated private line to AWS resources


1Gbps and 10Gbps (even 100 Gbps) service from AWS
Service below 1Gbps can be purchased by an AWS partner

Connects using a virtual interface (VIF)


LAG - link aggregation protocol



Bigger companies use another direct connection as a backup cause VPN speeds could be slower

VPCs are completely isolated, but sometimes need to talk to each other


To enable VPC peering you need to set up connection between the 2 VPCs.  It’s a point to point between 2 and only 2 VPCs.  Peered VPCs cannot have any overlapping CIDR blocks. If you have more than more 2 VPCs that you need to connect you need to connect a peering connection between each pair.  VPC does not support transitive peering. 
Route table comes in to play with peering. You get a peering ID and put that in the route tables after establishing peering connection



If you have a lot of VPCS, then peering isn’t an efficient solution.  It’s like a mesh design and will get super complicated when you have a lot of VPCs. 


A centralized VPC would be better. Shared VPC concept.    



### VPC endpoints

Allows you to privately connect to VPC without internet.  Supported by Private Link service. 


Endpoint states - 2 types.  Interface endpoint & Gateway endpoint.


Transit Gateway: solution which is highly available way to connect multiple VPCs. Supports up to 5000 VPC attachment.  One Transit gateway could replace thousands of VPC peering connection.

### Load balancing

Very important term in AWS.  Balances/distributes your traffic.   


Service that sits in front of infrastructure and directs traffic.  Automates the process

ELB: has several options


Application LB - functions at Layer 7 (HTTP, HTTPS)

Network LB functions at Layer 4 (TCP, TLS, UDP)


ELB provides a lot of features: HA, health checks, security (control traffic), TLS termination(can install a SSL on the ELB - saves a lot of compute resources) 

EC2 instance needs to be registered with an ELB.  You set rules about how the ELB behaves.  Draining - ELB is smart enough to know to finish requests before terminating instances. 


Can do path-based routing with ELB


### High Availability

Your app should be able to recover from a failure and rollover to secondary source within an acceptable amount of time




You should distribute resources across AZs to achieve HA (at least 2).


For multi-region HA then Route 53 helps

### DNS

Route 53 is built to manage domain services. It’s HA and scalable. 




Weighted routing: can assign by %.  Half of all requests can go to one server, 25% to another, etc. 

## Identify & Access Management

### IAM users, groups, role

The one identity that comes with every account is the root user. They have access to everything.  This makes it an attractive target for hackers. AWS recommends delegating access.



Least privilege principle - allow access only based on their current need. 


IAM is a service that allows you to assign access and privilege to users. Can be used for auditing as well.  


Authentication and authorization.  Can give a new user programmatic access (access key) or as password.  You can also add users to a specific Group.  Gives all users in group specific permissions. 
AWS has many managed policies.  

When you log in with an IAM user you see the username at the top right of the console. 

Least privilege: be careful how and when you share resources.  Share only what is needed. 


IAM principals: anything that can manipulate a resource. Have no access by default - need to define everything.  IAM user ≠ account. 


You grant access with policies. Policy’s job is to control behavior of IAM identities.  Defines one or more action.  There are hundreds or preset policies. You can also create your own (only once you are well versed with the statement model)


Any action that is not explicitly allowed will be denied. If there is a conflict with two policies (allow/deny), then the denied policy will take effect.  Deny always wins.




How IAM determines permissions:





Identify based = attached to user 

Resource based = attached to service



You can group users and create just one policy for the group.  


### Federated identity management

Lets you grant short-term access (because IAM policies are permanent)


Roles are not applied, they are assumed.  Roles are similar to a user, but instead of being uniquely associated to one person it’s anybody who assumes role. 



### Amazon Cognito

Managed solution to handle authentication and auth for web/mobile apps.




## High Availability

### High Availability


Inelasticity: 
You keep paying for resources you don’t use to sustain high user spikes, etc.   Could be a big waste depending on spikes.  You have to guess the compute for how much resources you need. 


Elasticity - your infra can scale in and scale out depending on need


Time-based elasticity - you can shut things off on a schedule


### Monitoring

CloudWatch is used for monitoring.  Lets you know how your resources are performing.  Can analyze over time. 


You need to be aware of all the activities/events going on.  All metrics get sent to CloudWatch.  Can view/search logs.   



Can get statistics from metrics.  Show you current CPU usage, etc. 

CloudWatch organizes metrics, stores them and classifies.  Metrics is the data about performance.  Many metrics monitored by default.  Can enable detailed metrics too.


Has Log data


Can have alarms set to automatically initiate an action on your behalf.  Like send an email if a certain % of resources used.  Must be defined.


CloudWatch Events delivers a real-time stream of events


Rules 


Targets


With CloudTrail you can see who is doing what exactly.  It logs every API event. 



### EC2 Auto Scaling

ASG - provisions and starts on your behalf a certain number of instances.  Can automatically replace failures.  


When you create a ASG you need to specify the # of instances (min/max, desired, etc.)

There are 3 ways to scale - by time/schedule, dynamically based on resource utilization, predictive - uses ML instead of manually adjusting


Need to define Min, Max, Desired capacity for ASG

You should never set min to 0

If you don’t specify desired, then it will use min.


### Scaling DBs

RDS uses read replicas

Aurora can have 15 replicas!

Aurora also has serverless version


## RECAP DAY 2

Recap day 2:
Discussed networking.  VPCs have subnets/CIDR blocks. CIDR needs to be in VPC range.  Divide subnet based on the block. VPCs are private network in the cloud.  Logical isolation for workload.  Allows custom access controls, security settings.  Can have up to 5 VPCs in a region (default). When you create a VPC there can be a single environment, multi account, etc. There are two categories of subnets, private and public. The difference is public can access the internet, private ones can not. Public has entry of internet gateway in route table to give access.  Security groups are firewalls for EC2.  Controls inbound and outbound traffic.  Security Groups have rules that are stateful (info is tracked).  By default, requests are denied - you have explicitly create rules to allow.  What is allowed for inbound is allowed for outbound.  If you want to extend on-premise network you can leverage VPN, Direct Connect (dedicated 1/10/100 connectivity). DC designed for biz with long term need.  VPCs can talk to each other through VPC peering. Must connect each VPC to each other, VPC peering does not support transitive relationship.  It’s 1 to 1 only.  VPC endpoints can be used to connect across AWS resources without going over the internet.  Load balancers (ELB = managed service) distributes network traffic across VPC. Application, network and classic balancers exists.  App supports the traffic from layer 7.  Network load balancers support traffic from layer 4 (EC2 instances, containers, etc.). LB ensure HA, health checks, security features, etc. High Availability(HA) ensures the entire app is up in running in case of failures.  Route 53 can be used to provide routing options to make sure traffic is distributed based on policies.  IAM - principal users gets a policy.  Authorization and authentication. You grant user certain policy giving access to what they can do.  Can group IAM users to make management easier.  Federation can be used to extend authorization.  Elasticity/HA used to scale DBs, instances, etc. 

## Automation

### Why automate?

Reduce human interaction with the IT systems.  Automation can control all elements.  Goal is to improve efficiency of IT operations and staff.  The goal is to go “hands-off”.  Speeds workload deployments.  Help reduces cost. 


 Key component to drive efficiency. Updates/OS patches/Ordering/Configuration can become unmanageable burden to IT org over time. Automation helps increase agility.

If you don’t have automation you’re going to have a bad time:


Manual takes time, but is also prone to human error.  Humans can make catastrophic errors.  


When you do manual you put yourself at risk.  There are Risks without getting any Reward.


### Automate Infrastructure

Infra as a Code (IaC).  Reduces provisioning time from weeks/months to a few hours/min.  


Automation can help identify anomalies. 
AWS has a service, CloudFormation, to help automate the infrastructure completely (from scratch).  

IaC - treat infra like a computer program.  Makes calls to API service to do everything, such as creating resources. Does whatever is specified in the template. 


If you make a change to the stack, you just make an update to the template.  CF makes deployment a set of AWS resources really simple.  
A template is basically a txt file (JSON/YML) that describes and defines the resources needed in a env.  Stack is the collection of resources deployed together in a group.  Everything you define in template ends up in your infrastructure. 


Treat your templates like source code.  Put in repo, do code reviews, etc. 
Benefits: repeatability, usability, etc. Ex. If you have a dev env, you can easily deploy that to production env.  And you can repeat that same infra multiple times. Helps to ensure proper consistency.  Build really complex environments over and over again. 




Templates are the building block. You can consider as declarative instructions.  Consists of different elements in JSON/YAML.  All templates start with a template format version (which is about the structure of the template).  Description section is optional, and explains what the template is what the template will create. “Parameters” (optional) makes the template reusable.  Think of them like variables. CIDR/subnets/instance types/etc could be parameters.  Every time you use Parameter you should set a Default value (helps ensure parameters always have a value). 


Mapping element is similar to parameters, but provided in a dictionary format. 


Resources section is the only section that is required.  Defines resources you want to create.  


Not all services are in CF.  Here is the complete list for reference: [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

Output can retrieve the name and ids of resources after creation.


To update a stack you can edit existing templates.  You can also use a Change set.  Which lets you review/verify what to expect with those changes.



AWS QuickStarts are built-in solutions you can use.  




### AWS Elastic Beanstalk


No charge to use Beanstalk (you are just charged for resoucres used). Used to quickly deplyo an app wtiihout having to worry about managing the underlying resources at all.   It handles all the decisions based on the code you want to deploy.  




Two tiers - Web Server for frontend apps, or Infrastructure for backend.  

## Multiple Accounts

This can be a very tedious process to manage. Hard to keep track of passwords/creds.  Not efficient to manual manage.  You want multiple accounts for logical isolation. 


Create a role, have someone assume role, keep switching around roles is a good way to handle this.  Mult accounts is used for security/governance. Adds enhanced security. 


To manage multiple accounts using a service you can use AWS Organizations.  Great service/feature to centrally manage accounts.  Lots of redundant work can be cut down. 



Group accounts by OUs (logical containers).  Can attached multiple policies to OUs. 


A filter to control the behavior of accounts is called SCP (Service Control Policy)


You get Consolidated Billing is a big benefit of having AWS Organizaiton.  You grow out from the Root account. 


SCPs look like IAM policies.   They are linked to OUs (or root, which would impact everybody).  Can even add a SCP directly to an individual account.  Best practice is to link to OUs. 

SCPs do not give permissions.  They are like a filter.  Permissions are granted via IAM.  You get the intersection of IAM and SCP. 




With Organizations, you get one bill.  There is one parent account that pays for charges for all other accounts. 





## Deployment methods




### Blue-green deployment

Deployment strategy in which you create two separate by identical environments.  One runs old, one runs new version of your app.  


Helps reduce downtime in case of failure. Copies one env to another.  Then shifts traffic to the new one. 


With Blue-Green you have deal with the caching of the old DNS resolver.  Also can add a lot of extra costs by having duplicate envs. 


Global Accelerator acts like a permanent hub between architicture and DNS. 

### Canary deployment

Purpose of Canary is to reduce the risk of deploying a new version.  Shows new version to users in slower fashion (send to a small % of users and then check your KPI metrics, etc.  Then increase load gradually until you are at 100%).  



Migration phase lasts until all users have been around to never version. 

You can also use both Blue-green and Canary together. 


### Tools for deploying

CloudFormation StackSets - extends stacks by allowing you to create stacks across multiple different regions/accounts. (stacks are resources you create with CF)


To route traffic use Route 53’s weighted routing


Systems Manager - Helps you automatically collect software inventory.  Can do patches, configuration, etc.  Helps maintain compliance. Makes it easier to bridge existing infra w/ cloud.  


CI/CD - can make teams more productive. CI requires writing automated tests to validate software. CI stops at the Build stage.  After that it’s CD, where you want to deploy to the end target.   


Core AWS devOps services: 


CodeBuild is like Jenkins

## Caching

### Overview

Process of storing data in a cache. Temporary storage area (relatively small) but has a faster access time.  When your app needs to read data, it should look in cache, and only if the data isn’t there it goes to the data store.  

There are many different levels of caching.  Client side (os, browser, etc).  CDN caching, such as CloudFront.  Webserver caching can cache requests/responses.  DB caching is very common, usually some caching is enabled by default for all DBs.


Data that is expensive to query should be cached, such as some DB queries (that have joins on multiple tables).  Doesn’t make sense to change data that is rapidly changing.  Data should be relatively static. 


Provides high throughput, low-latency. Can Improves speed of app.  Response time reduces. Processing time reduction.  Write intensive apps might not be a good candiate. 



  

### Edge caching



Content is cached at edge locations. 


CloundFront is the low-latency CDN


Securely transfers content to the client with high speed.  It is simple to use.  Has geo-targeting features. CloudFront responds to requests whether not not data exists there.



To setup you create an origin servers.  That stores the original definite version of the object. 


A problem when dealing with caching is the expiration.  TTL is time to live.  TTL is set up with HTTP headers usually.    



### Web-tier caching

Keeping track of user state info is very important.  For a gaming server, maybe current score is something important.  


Start caching when you’re concern about response time. 



Service to help with caching DB is DAX (DynamoDB Accelerator).  Microsecond response time. Highly scalable. 


For RDS, AWS has ElastiCache. Was rolled out in 2011.  One of the most prominent features.  Awarded by StackOverflow as one of the most favorite solutions by devs. It’s famous for HIGH COST.  It is a Cache as a Service.  Comes as a fully managed solutions, to deploy, scale, distribute, etc. Allows you to create and operate in-memory data stores. Eliminates complexity. Boosts performance of webapps. 



Two options: Redis/Memcached


Lazy loading - load only when required.  Data is not there upfront.  


Any time data is in DB, it goes to cache. Write through model:


## Security of Data

### Classification

Shared Responsibility Model 


Security/compliance shared between customer and AWS.  AWS handles infra, physical security, etc.  AWS handles security OF the cloud, customer handle security IN the cloud.  The more managed solution, the less responsibly on the user. 
Data needs to be secure for many reasons.  Could cause financial or regulatory problems if not secure. 



Best thing to do is to CLASSIFY data



Once data is classified, then do risk assessment


### Encryption

Process of taking data and put so it’s not human readable. Use keys to encrypt data, then store it in encrypted storage.  The key is also encrypted.  They key is the most important, critical factor.  you need a key management solution.   You could do this on your own in your datacenter. 



AWS’s key management service is KMS. Secure and resilient service.  



You have complete control of the keys


KMS is a shared service on shared hardware.  If you need a more dedicated space/infrastructure you can use AWS CLoudHSM.




In transit: 


Automation and ML solution to protect sensitive data - Macie. AI powered security.  Helps classify info. 


Guard Duty - preventive 


## RECAP DAY 3

Automation - CloudFormation.  We can use CF to automate deployments.  Stacks are a bunch of resources created by a template.  Templates help automate the deployment stacks.  Automating is necessary these days.  
Caching is important.  Edge caching, CDN -content delivery network.  Use Edge Locations w/ AWS Cloud Front.  Ensure users are getting all the content they need from Edge locations closest to them.  DB Caching uses DAX - dynamodb accellerator.  Elasticache = redis/memcached.  Both engines help get the job done.  
Security - Encryption.  KMS key management service helps encrypt data (shared service).  If you want a custom/dedicated service use CloudHSM.  
Deployment - best way to deploy resources. Blue-green, Canary strategies help ensure better deployment.  B-G, you always have a deployment running.  No downtime. CloudFormation stack set, you can deploy multi region. 

## Decoupled Architectures





Large monolithic codebases are very hard to undedrstand.  Especially for new developers.  Difficult to change language/framework because everything is tightly coupled.  You may have to do big deployments even if making one change

Decoupling is breaking down your big monolithic app into smaller services.  Decopuling also includes all the communications between, such as ELB.  Qeues are important part of this as well. 


### SQS - Simple Queue Service

Managed service from AWS. 


There is Synchronous and Async communication.  In Sync communication between application can be problematic if increase in traffic.  SQS is managed service, pay per use, for storing messages in transit between computers.   A message queue - method in which data is processed.  Apps don’t have to invoke another app directly.  


Used a lot by devs because it reduces complexity of managing.  You can create an intermediate queue to handle functions, etc.  
FIFO - first in first out.  If order is important.  There is also only one processing.  Duplication is not allowed. 


Standard queue can have unlimited throughput (transactions per second). Has “at least once” delivery.  It’s possibly that duplciation could exist.   


SQS gives a lot of benefits.  DLQ - deal letter queue is a queue of messages that cannot be processed.  After a number of retries the messages go to the DLQ so you won’t loose them.  Visibility timeout - SQS locks messages so that they are hidden from other components (ex ec2q instances).  Long polling - way to retrieve messages from queue that makes things less expensive. 



SQS is a distributed queue system. 


Think of a queue as a temporary repository for messages.  1-14 days max (default ~4 days).  Really helps with decoupling apps. Messages support up to 256kb per message. SQS can also be combined with many AWS services (ec2, lambda, etc). 

SQS doesn’t automatically delete messages (cause it can’t guarantee delivery). It’s the consumers responsibility to delete it. 


### SNS - Simple Notification Service

Infrastructure process8ing.. Pub/Sub model.  Publishes data to an SNS topic, and the topic has subscribers that goes and gets the information. 


Logical access point that acts as a communication channel.  Like an intermediary brokerage channel. Subscribers subscribe to Topic regardless of who publishes the message(sends).  SNS is fully managed pub/sub system. Easy integration and inexepsenive. (Payas you go).

Topic = named groups of events or access points.  Has a unique ID. 
Subscribers = clients, end users, servers, etc that want to recieve topics
Publishers = who sends message to the topics. 



SNS helps with a fan-out scenario. 


## Optimizations

Questions to ask yourself.  Are you using the best resources? 


Is your architecture resilient?


Can you decouple anything?


Can you scale effectively?


Always think about Re-architecting

Example app:


if there is any deadlock or race condition you would have failures.  SQS would really help in this case:



Instead of using all On-demand instsances, we can use reserved instanes to save $$$ 



We can have multiple AZs to make it more resiliency. 


EC2 instances don’t need to be public


Add a new region to make extra resilient


Choose infrastructure based on the type of processing you’re doing




Don’t get too attached to any resource.  Think of them all as being disposable. 



We’re here on the cloud to make a product.  Focus on the service, not the server.


 Polyglot - choose based on what you need.  Be flexible and dynamic. 




Always consider the $$$.Just pay for what you need. 


Caching provides best experience for end users. 


Security is about ensuring your indviducla env and components are secure too.



## Microservices



Microservices are very resilient.  Single Responsibility Principle. Each service responsible for discrete task.    


Microservices offer a lot of fault tolerance.  Makes easier to choose tech stacks that is best suited for the functionality of the service. Offers unique advantages over traditional architecture.  


You need to clearly define business capabilities / requirements when designing microservices. 

Each MS should be completely self contained. 


Many microservices used in one app or on one page:


Good resource - 12 Factor App: [https://12factor.net/](https://12factor.net/)


Principles: 
#1 rely only on public API


Evolue the API over time.  Chnge the API contract


#2 Use the right tool - be polyglot


#3 Security

Think of security at each step of the App


API Gateway service comes in handy at the part

#4 Be a good citizen



Have clear SLAs

#5 Have small teams


#6 Automate everything - adopt Devops


### Containers


Ship from dev to prod in consistant way.  Container becomes a snapshot of the entire app.
Containers are very fast.  Give you granular control over resources (improves efficienty).  Pre-made container solutions of AWS Marketplace, but they are super customizable.  Lots of things to manage - compute, Docker, configuration, etc.


If you have lots and lotrs of containers AWS has a container service, ECS.

Highly scalable.  Handles all the infra part.  


Handles scaling, configuring, etc:


Can scale easily


Works very well with microservices



ECS has two types.  EC2 or Serverless.
Fargate is serverless container compute technology.  You only have to manage the containers, not the ec2, clusters, etc.  


## Serverless

Execution model where the cloud provider handles all the server managment. Abstracts the server layer away from you.  There are still servers under the hood, you just don’t have to worry about it. 



FaaS - functions as a service

Lamda is an AWS service that does FaaS.  Can run code w/o provisioning servers.  Scales your app by running code as needed. You are responsible with cost of instance runtime (much cheaper than having an ec2 server constantly running). 



Supports many different event sources:


Lambda handles almost everything except the function itself


Remember, if function isn’t running ther is no charge at all. 


API Gateway - easy API creation/management 




Orchestration of Lambda function becomes an important element:


Step Functions (Service) - function orchestrator

Handles decisions, retries, error handling, etc



State is just a JSON doc defining the flow:


You can see the workflow visually too


Lab exercise:


Resource for serverless architectures: [https://3factor.app/](https://3factor.app/)

## Resiliency

### Avoiding Failure


Go serverless - don’t have to manage or be responsible for as much




Cloud Map - a cloud discovery service.. You can register and manage 




### DDoS Attacks


DoS - Denial of Service - highjacking webserver, highjacking requests, etc.  Can be done from a single machine. Easier to execute, also easier to figure out the source


DDoS - Distributed DoS - intstead of one request, you get 100’s and flood your system. They look legit.  Come from multiple sources so hard to track. 





Sometimes HTTP errors can be genuine.  A user submits multiple forms, etc


Protect from DDoS/etc with WAF = Web application Firewall



## RECAP DAY 4

Resilience - build infra that can withstand failure and is HA.  Ensure that happens at all points of time.  Resistant to attacks.  Serveless architecure can help you avoid problems.  CloudMap to keep track of all endpoints.  DDoDs attacks - there are AWS services to help with this (WAF).  Lambda - max execution time is 15min per function.  Lambda runs functions as a service.    Source can be s3, dynamodb, SNS, etc, and go through a Lambda. You give AWS the function, AWS does whatever is necessary to run that function and you don’t have to worry about it.  API Gateway - service that helps you host your APIs.  You can control in a lot of ways when interaction wtiht he outside world. API gateway gives you control of API management (plus you get DDoS protection, etc).  Api gateway acts as a front door to other AWS services behind it. It’s so great because it’s Serverless. Step Functions -  a workflow orchestrrator. It helps when you have multiple lambda functions doing myultipel jobs.  Some lambda job might be based on the output of another lambda, and that’s where step functions come in to play. It’s a state machine.  Uses a visual workflwo.  You can launch/track each function.  Uses a states language document (JSON) to control.  ECS - Elastic Container Service - container management solution.  (Similar to EKS - which is more orchestration).  ECS takes away the pain of managing compute, the image repositopryu, etc.  Youc an just focus on your container runtime.  ECS - two launch types, ec2 and Fargate.  Fargate is serverless. Instruct fargate the number of containers to run, but how everything is deployed (resources/etc) is handled by Fargate.  

## Networking Part 3

### EC2 Performance


Single route IO Virtualization is like an interface/extension to the virtual network card.  Provides higher performance and lower CPU utilization.  No additional charge for enhanced networking ( you have to pay for the machine type that supports that model)

t2 family does not support enhanced netowrked.  ENA - Elastic Network adapter. 


MTU - Maximum transmission unit - tern in networking that defines largeest size of the pakcet that can be transferred in a single entity. Larger MTU results in more data transmission.  Smaller MTU can be faster.  1500 bytes is the ethernet standard.  With enhanced networking you can used jumbo frames (up to 9000 bytes)


### DNS Caching

Global Accelerator - managed traffic manager.  Easy to setup service.  Leverages edge locations to avoid any dns caching.  Allows for region-levle load balancing. Continuously monitros endpoint. 








### VPN Connections

A way to connect to AWS resources






Can make more resliant by adding another GW on customer side:



### Large-scale VPC Peering

Transit Gateway - connect thousands of VPCs and on-prem environments.  Don’t have to do individual peering.  Can create a transitive network archecture with a few clicks. 


Need to add transite gateway to route table



Acts as a cloud router.  Makes things very easy.  Fully managed service. 

## Cost efficiency

You should always want to reduce costs.

Can use pricing calculator to estimate


Trusted Advisor - service which can recommend based on best practices.  Scans your infra and makes recommendations




Note: you should always choose the newest generation when you have the choice














## Migration

5 phase process






need to do detrailed evaluation and analysis before deciding to migrate












## Disaster Planning & Recovery




RTO - time after a disaster that resources are can be used again (focuses on downtime) — looks forward

RPO - how much data loss is acceptable (focuses on amount of data) — looks backward
















[https://aws.amazon.com/serverless-workshops/](https://aws.amazon.com/serverless-workshops/)

[https://catalog.us-east-1.prod.workshops.aws/workshops/ef1c179d-8097-4f34-8dc3-0e9eb381b6eb/](https://catalog.us-east-1.prod.workshops.aws/workshops/ef1c179d-8097-4f34-8dc3-0e9eb381b6eb/)

[https://ecsworkshop.com/](https://ecsworkshop.com/)
