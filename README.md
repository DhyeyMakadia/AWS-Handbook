# AWS Handbook — LocalStack Edition

A hands-on reference for learning AWS using **LocalStack** (`lstk`) on **Windows 11** — no AWS account required, no charges incurred.

> **Verified environment**

| Tool    | Version   | Platform |
| ------- | --------- | -------- |
| Docker  | `29.5.3`  | Windows  |
| `lstk`  | `0.21.0`  | Windows  |
| AWS CLI | `2.36.18` | Windows  |

If a command behaves differently, run `lstk --version` / `aws --version` and cross-check the [lstk CLI reference](https://docs.localstack.cloud/aws/developer-tools/running-localstack/lstk/).

---

<details open>

<summary><strong>Setup</strong></summary>

## 🛠 Setup

### Start LocalStack

#### Documentation

LocalStack needs to be running before any AWS command can be executed locally — it's the emulator that stands in for AWS on your machine.

#### Cheatsheet

```powershell
lstk start
```

To keep LocalStack resources after restarting:

```powershell
lstk start --persist
```

---

### Check LocalStack Status

#### Documentation

Before troubleshooting anything else, confirm the LocalStack container and its services are actually up and healthy.

#### Cheatsheet

```powershell
lstk status
```

---

### Check LocalStack Health

#### Documentation

This hits LocalStack's health endpoint directly, which is useful when you want to see service status without going through `lstk`.

#### Cheatsheet

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

> On Windows, `curl.exe` is used explicitly to run the curl executable.

---

### Verify AWS CLI Connection

#### Documentation

This confirms the AWS CLI is actually talking to LocalStack and not real AWS — a good habit to run at the start of every session.

#### Cheatsheet

```powershell
lstk aws sts get-caller-identity
```

> While learning with LocalStack, prefer `lstk aws ...` instead of a plain `aws ...` command.

---

### Troubleshooting — LocalStack Itself

#### Documentation

This is the first place to look whenever **any** AWS service command misbehaves — before assuming the problem is service-specific, confirm LocalStack itself is running and healthy.

#### Cheatsheet

Check if LocalStack is running:

```powershell
lstk status
```

If it isn't running, start it:

```powershell
lstk start
```

Check LocalStack's service health directly:

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

Re-verify the AWS CLI is pointed at LocalStack:

```powershell
lstk aws sts get-caller-identity
```

> Only once these come back clean should you look at service-specific troubleshooting (see the **Troubleshooting** section inside S3, EC2 or similar).

</details>

---

<details>

<summary><strong>S3</strong></summary>

## 🪣 S3 — Simple Storage Service

### What is S3?

#### Documentation

S3 provides object storage — a place to store files (called **objects**) inside containers (called **buckets**).

Simple mental model:

```text
S3 = Online File Storage
```

An S3 bucket holds **objects**, each identified by a **key**, and can be governed by a **bucket policy**, **versioning** settings, and **public access block** settings.

---

### S3 Core Concepts

#### Documentation

Before touching any commands, it helps to know what each piece actually represents:

| Concept              | Meaning                            | Easy Mental Model         |
| --------------------- | ------------------------------------ | --------------------------- |
| S3                    | Object storage service               | Online file storage          |
| Bucket                | Container for objects                | A folder / drive              |
| Object                | A stored file                        | A file                          |
| Key                   | Full path/name of an object          | File name                        |
| Bucket Policy         | Resource-based access rules          | Bucket-level rulebook             |
| Versioning            | Keeps multiple copies of an object   | File history                       |
| Public Access Block   | Prevents accidental public access    | Bucket privacy switch               |
| Storage Class         | Cost/performance tier                | Shelf vs. cold storage                |
| Presigned URL         | Temporary access link                | Time-limited guest pass                |

---

### S3 Resource Relationship

#### Documentation

A bucket lives in a Region, and everything else hangs directly off the bucket:

```text
Region
  |
  +-- Bucket
        |
        +-- Object
        +-- Object
        +-- ...
```

Another useful way to look at it — from the object's own point of view, at upload time:

```text
Bucket
 |
 | put-object
 v
Object (Key)
 |
 +-- Storage Class
 +-- Version ID (if versioning enabled)
 +-- Metadata
```

---

## S3 — Buckets

### What is a Bucket?

#### Documentation

A bucket is a top-level container for objects. Bucket names must be globally unique (even under LocalStack's emulated naming rules, it's good practice to keep names unique).

```text
Bucket = Container
Object = File inside the container
```

---

### Bucket Commands

#### Documentation

Always start by listing what buckets already exist — you won't always know the exact names or how many buckets are already sitting in your LocalStack environment.

#### Cheatsheet

List:

```powershell
lstk aws s3 ls
```

List (low-level API form, includes creation date):

```powershell
lstk aws s3api list-buckets
```

Create:

```powershell
lstk aws s3 mb s3://s3-learning-bucket
```

Create (low-level API form):

```powershell
lstk aws s3api create-bucket --bucket s3-learning-bucket
```

List again to confirm the new bucket was created:

```powershell
lstk aws s3 ls
```

View details of one bucket's location:

```powershell
lstk aws s3api get-bucket-location --bucket s3-learning-bucket
```

Delete (bucket must be empty):

```powershell
lstk aws s3 rb s3://s3-learning-bucket
```

Delete (force — removes objects too):

```powershell
lstk aws s3 rb s3://s3-learning-bucket --force
```

> Deleting a bucket is destructive if `--force` is used — it removes every object inside it first.

---

## S3 — Objects

### What is an Object?

#### Documentation

An object is a file stored in a bucket, identified by its **key** (essentially its path/name within the bucket).

```text
s3://s3-learning-bucket/folder/file.txt
                        |______________|
                              key
```

---

### Object Commands

#### Documentation

As with buckets, list objects first if you're not sure what's already inside a bucket — especially before uploading, overwriting, or deleting anything.

#### Cheatsheet

List objects in a bucket:

```powershell
lstk aws s3 ls s3://s3-learning-bucket
```

List objects (low-level API form, includes metadata):

```powershell
lstk aws s3api list-objects-v2 --bucket s3-learning-bucket
```

Upload a file:

```powershell
lstk aws s3 cp .\file.txt s3://s3-learning-bucket/file.txt
```

Upload a file (low-level API form):

```powershell
lstk aws s3api put-object --bucket s3-learning-bucket --key file.txt --body .\file.txt
```

Upload an entire folder:

```powershell
lstk aws s3 cp .\my-folder s3://s3-learning-bucket/my-folder --recursive
```

List objects again to confirm the upload:

```powershell
lstk aws s3 ls s3://s3-learning-bucket
```

Download a file:

```powershell
lstk aws s3 cp s3://s3-learning-bucket/file.txt .\file.txt
```

Copy an object within/between buckets:

```powershell
lstk aws s3 cp s3://s3-learning-bucket/file.txt s3://s3-learning-bucket/backup/file.txt
```

Sync a local folder to a bucket (uploads changes only):

```powershell
lstk aws s3 sync .\my-folder s3://s3-learning-bucket/my-folder
```

Delete a single object:

```powershell
lstk aws s3 rm s3://s3-learning-bucket/file.txt
```

Delete all objects under a prefix:

```powershell
lstk aws s3 rm s3://s3-learning-bucket/my-folder --recursive
```

Important options:

| Option        | Meaning                                |
| ------------- | ---------------------------------------- |
| `--recursive` | Apply to all objects under a prefix      |
| `--exclude`   | Skip files matching a pattern            |
| `--include`   | Re-include files matching a pattern      |
| `--dryrun`    | Preview the action without executing     |

---

## S3 — Bucket Policy

### What is a Bucket Policy?

#### Documentation

A bucket policy is a resource-based JSON policy attached directly to a bucket that controls who can do what to it.

```text
Bucket
  |
  +-- Bucket Policy (JSON)
        |
        +-- Effect: Allow/Deny
        +-- Principal: who
        +-- Action: what
        +-- Resource: which bucket/objects
```

---

### Bucket Policy Commands

#### Cheatsheet

View (check if one already exists before overwriting it):

```powershell
lstk aws s3api get-bucket-policy --bucket s3-learning-bucket
```

Set (from a local JSON file):

```powershell
lstk aws s3api put-bucket-policy --bucket s3-learning-bucket --policy file://policy.json
```

Remove:

```powershell
lstk aws s3api delete-bucket-policy --bucket s3-learning-bucket
```

> In real AWS, avoid writing a policy with `"Principal": "*"` and broad `Action` unless the bucket is genuinely meant to be public. Scope principals and actions tightly.

---

## S3 — Versioning

### What is Versioning?

#### Documentation

Versioning keeps every version of an object each time it's overwritten, instead of replacing it.

```text
Versioning Off:
file.txt (overwritten each time)

Versioning On:
file.txt -> v1
file.txt -> v2
file.txt -> v3 (current)
```

---

### Versioning Commands

#### Cheatsheet

Check status (before assuming versioning is on or off):

```powershell
lstk aws s3api get-bucket-versioning --bucket s3-learning-bucket
```

Enable:

```powershell
lstk aws s3api put-bucket-versioning --bucket s3-learning-bucket --versioning-configuration Status=Enabled
```

List all versions of objects:

```powershell
lstk aws s3api list-object-versions --bucket s3-learning-bucket
```

Download a specific version:

```powershell
lstk aws s3api get-object --bucket s3-learning-bucket --key file.txt --version-id <VERSION_ID> file.txt
```

---

## S3 — Public Access Block

### What is Public Access Block?

#### Documentation

Public Access Block is a safety switch that prevents a bucket (or objects in it) from being made public by accident, regardless of what a bucket policy or object ACL says.

```text
Public Access Block
    |
    +-- Block public ACLs
    +-- Block public policy
    +-- Ignore public ACLs
    +-- Restrict public buckets
```

---

### Public Access Block Commands

#### Cheatsheet

View (check current settings before changing them):

```powershell
lstk aws s3api get-public-access-block --bucket s3-learning-bucket
```

Set (block everything public):

```powershell
lstk aws s3api put-public-access-block --bucket s3-learning-bucket --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

---

## S3 — Presigned URLs

### What is a Presigned URL?

#### Documentation

A presigned URL grants temporary, time-limited access to a private object without changing the bucket's permissions.

```text
Object (private)
    |
    | presign (with expiry)
    v
Temporary URL --> anyone with the link can access it until it expires
```

---

### Presigned URL Commands

#### Cheatsheet

Generate a link valid for 1 hour (3600 seconds, the default):

```powershell
lstk aws s3 presign s3://s3-learning-bucket/file.txt
```

Generate a link with a custom expiry (in seconds):

```powershell
lstk aws s3 presign s3://s3-learning-bucket/file.txt --expires-in 300
```

---

## S3 — AWS CLI Queries

### Why Use `--query`?

#### Documentation

`--query` lets you select only the information you need from a large AWS CLI response.

Use simple queries first while learning.

#### Cheatsheet

Get just the bucket names:

```powershell
lstk aws s3api list-buckets --query "Buckets[].Name"
```

Get just the object keys:

```powershell
lstk aws s3api list-objects-v2 --bucket s3-learning-bucket --query "Contents[].Key"
```

Get key and size for each object:

```powershell
lstk aws s3api list-objects-v2 --bucket s3-learning-bucket --query "Contents[].{Key:Key,Size:Size}" --output table
```

---

## S3 — Complete Learning Workflow

### Step 1 — Start LocalStack

```powershell
lstk start --persist
```

### Step 2 — Verify AWS CLI

```powershell
lstk aws sts get-caller-identity
```

> If either step fails, see **Setup → Troubleshooting — LocalStack Itself** first.

### Step 3 — List Existing Buckets

```powershell
lstk aws s3 ls
```

Do this before creating anything — it tells you what buckets already exist and their exact names, so you're never guessing.

### Step 4 — Create a Bucket

```powershell
lstk aws s3 mb s3://s3-learning-bucket
```

### Step 5 — Confirm the Bucket Exists

```powershell
lstk aws s3 ls
```

### Step 6 — List Objects in the Bucket (should be empty)

```powershell
lstk aws s3 ls s3://s3-learning-bucket
```

### Step 7 — Upload an Object

```powershell
lstk aws s3 cp .\file.txt s3://s3-learning-bucket/file.txt
```

### Step 8 — Confirm the Object Was Uploaded

```powershell
lstk aws s3 ls s3://s3-learning-bucket
```

### Step 9 — Check Versioning Status

```powershell
lstk aws s3api get-bucket-versioning --bucket s3-learning-bucket
```

### Step 10 — Enable Versioning

```powershell
lstk aws s3api put-bucket-versioning --bucket s3-learning-bucket --versioning-configuration Status=Enabled
```

### Step 11 — Upload a New Version

```powershell
lstk aws s3 cp .\file.txt s3://s3-learning-bucket/file.txt
```

### Step 12 — List Versions

```powershell
lstk aws s3api list-object-versions --bucket s3-learning-bucket
```

### Step 13 — Generate a Presigned URL

```powershell
lstk aws s3 presign s3://s3-learning-bucket/file.txt --expires-in 300
```

### Step 14 — Download the Object

```powershell
lstk aws s3 cp s3://s3-learning-bucket/file.txt .\downloaded-file.txt
```

### Step 15 — Clean Up Objects

```powershell
lstk aws s3 rm s3://s3-learning-bucket/file.txt --recursive
```

### Step 16 — Confirm the Bucket Is Empty

```powershell
lstk aws s3 ls s3://s3-learning-bucket
```

### Step 17 — Delete the Bucket

```powershell
lstk aws s3 rb s3://s3-learning-bucket --force
```

### Step 18 — Confirm the Bucket Is Gone

```powershell
lstk aws s3 ls
```

---

## S3 — Troubleshooting

### S3 Command Is Not Working

#### Documentation

If LocalStack itself is confirmed healthy (see **Setup → Troubleshooting — LocalStack Itself**) but an S3 command still isn't behaving as expected, start here.

#### Cheatsheet

```powershell
lstk aws s3 ls
```

If this fails or returns unexpected results, re-check LocalStack's health before assuming the S3 command syntax is wrong:

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

---

### Bucket or Object Is Unknown

#### Documentation

Use the matching `list`/`get` command instead of guessing a bucket name or object key.

#### Cheatsheet

| Resource            | Command                       |
| --------------------- | -------------------------------- |
| Bucket                 | `list-buckets` / `s3 ls`         |
| Object                 | `list-objects-v2` / `s3 ls`       |
| Object Versions        | `list-object-versions`             |
| Bucket Policy          | `get-bucket-policy`                 |
| Versioning Status      | `get-bucket-versioning`              |
| Public Access Settings | `get-public-access-block`             |

---

## S3 — Quick Refresh

### Mental Model

```text
S3 = Online File Storage
Bucket = Container
Object = File
Key = File Name/Path
Bucket Policy = Access Rulebook
Versioning = File History
Public Access Block = Privacy Switch
Presigned URL = Time-Limited Guest Pass
```

### Most Used Commands

#### Documentation

A single consolidated block of the commands you'll use in almost every session — handy for a fast refresh before an interview or after time away from the material.

#### Cheatsheet

```powershell
# Start LocalStack
lstk start

# Verify AWS CLI connection
lstk aws sts get-caller-identity

# List buckets (always check first — names may not be obvious)
lstk aws s3 ls

# List buckets (low-level API form)
lstk aws s3api list-buckets

# Create a bucket
lstk aws s3 mb s3://s3-learning-bucket

# List objects in a bucket
lstk aws s3 ls s3://s3-learning-bucket

# List objects (low-level API form)
lstk aws s3api list-objects-v2 --bucket s3-learning-bucket

# Upload a file
lstk aws s3 cp .\file.txt s3://s3-learning-bucket/file.txt

# Download a file
lstk aws s3 cp s3://s3-learning-bucket/file.txt .\file.txt

# Sync a folder
lstk aws s3 sync .\my-folder s3://s3-learning-bucket/my-folder

# Delete an object
lstk aws s3 rm s3://s3-learning-bucket/file.txt

# Delete a bucket and its contents
lstk aws s3 rb s3://s3-learning-bucket --force
```

---

## S3 — Learning Checklist

### Fundamentals

- [ ] Understand S3
- [ ] Understand bucket
- [ ] Understand object
- [ ] Understand key
- [ ] Understand bucket policy
- [ ] Understand versioning
- [ ] Understand public access block
- [ ] Understand storage class
- [ ] Understand presigned URL

### S3 Commands

- [ ] s3 ls (list buckets)
- [ ] list-buckets
- [ ] mb / create-bucket
- [ ] s3 ls (list objects in a bucket)
- [ ] list-objects-v2
- [ ] cp (upload/download)
- [ ] sync
- [ ] rm
- [ ] rb (delete bucket)
- [ ] put-bucket-policy
- [ ] get-bucket-policy
- [ ] delete-bucket-policy

### Versioning & Access

- [ ] get-bucket-versioning
- [ ] put-bucket-versioning
- [ ] list-object-versions
- [ ] get-public-access-block
- [ ] put-public-access-block
- [ ] presign

</details>

---

<details>

<summary><strong>EC2</strong></summary>

## 🖥️ EC2 — Elastic Compute Cloud

### What is EC2?

#### Documentation

EC2 provides virtual computers called **instances**.

Simple mental model:

```text
EC2 = Virtual Computer
```

An EC2 instance is launched using an **AMI**, given a **compute size**, connected to a **network**, protected by a **security group**, and can use **EBS storage**.

---

### EC2 Core Concepts

#### Documentation

Before touching any commands, it helps to know what each piece actually represents:

| Concept          | Meaning               | Easy Mental Model    |
| ---------------- | --------------------- | -------------------- |
| EC2              | Compute service       | Virtual computers    |
| Instance         | Virtual machine       | A computer           |
| AMI              | Machine image         | OS/template          |
| Instance Type    | Compute configuration | CPU/RAM size         |
| Key Pair         | Authentication        | Login key            |
| Security Group   | Network firewall      | Firewall             |
| EBS              | Block storage         | Hard disk            |
| VPC              | Virtual network       | Private network      |
| Subnet           | Part of a VPC         | Smaller network      |
| Route Table      | Network routing       | Traffic directions   |
| Internet Gateway | Internet connectivity | Door to the internet |
| ENI              | Network interface     | Virtual network card |

---

### EC2 Resource Relationship

#### Documentation

Everything in EC2 fits together hierarchically — a Region contains Availability Zones, which contain your networking, which contains your instance and everything attached to it:

```text
Region
  |
  +-- Availability Zone
        |
        +-- VPC
              |
              +-- Subnet
                    |
                    +-- EC2 Instance
                          |
                          +-- Security Group
                          +-- Key Pair
                          +-- EBS Volume
                          +-- Network Interface
```

Another useful way to look at it — from the instance's own point of view, at launch time:

```text
AMI
 |
 | launch
 v
EC2 Instance
 |
 +-- Instance Type
 +-- Key Pair
 +-- Security Group
 +-- EBS
 +-- Network Interface
```

---

## EC2 — Regions and Availability Zones

### List Regions

#### Documentation

A Region is a geographic AWS location.

#### Cheatsheet

```powershell
lstk aws ec2 describe-regions
```

---

### List Availability Zones

#### Documentation

An Availability Zone is an isolated location inside an AWS Region.

#### Cheatsheet

```powershell
lstk aws ec2 describe-availability-zones
```

Mental model:

```text
Region
  |
  +-- Availability Zone
  +-- Availability Zone
  +-- Availability Zone
```

For learning, use one region consistently, such as `us-east-1`.

---

## EC2 — AMI

### What is an AMI?

#### Documentation

**AMI (Amazon Machine Image)** is a template used to launch an EC2 instance.

```text
AMI = Operating System + Base Configuration
```

The relationship is:

```text
AMI
 |
 | launch
 v
EC2 Instance
```

---

### AMI Commands

#### Cheatsheet

List available images:

```powershell
lstk aws ec2 describe-images --output table
```

List images as JSON:

```powershell
lstk aws ec2 describe-images --output json
```

> Do not blindly copy an AMI ID from an online tutorial. Use an AMI ID available in your LocalStack environment.

---

## EC2 — Key Pair

### What is a Key Pair?

#### Documentation

A key pair is used for authentication to an EC2 instance.

```text
Key Pair
 |
 +-- Public Key
 |
 +-- Private Key
```

For this LocalStack learning setup, we do **not** need to save the key material to a `.pem` file.

---

### Key Pair Commands

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-key-pairs
```

Create:

```powershell
lstk aws ec2 create-key-pair --key-name ec2-learning-key
```

View one:

```powershell
lstk aws ec2 describe-key-pairs --key-names ec2-learning-key
```

Delete:

```powershell
lstk aws ec2 delete-key-pair --key-name ec2-learning-key
```

---

## EC2 — Security Group

### What is a Security Group?

#### Documentation

A Security Group is a stateful virtual firewall for EC2 resources.

```text
Internet
    |
    v
Security Group
    |
    +-- Allow TCP 22
    +-- Allow TCP 80
    +-- Allow TCP 443
    |
    v
EC2 Instance
```

---

### Security Group Commands

#### Cheatsheet

List (check what already exists before creating a new one):

```powershell
lstk aws ec2 describe-security-groups
```

Create:

```powershell
lstk aws ec2 create-security-group --group-name ec2-learning-sg --description "Security group for EC2 learning"
```

Add SSH rule:

```powershell
lstk aws ec2 authorize-security-group-ingress --group-name ec2-learning-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
```

Add HTTP rule:

```powershell
lstk aws ec2 authorize-security-group-ingress --group-name ec2-learning-sg --protocol tcp --port 80 --cidr 0.0.0.0/0
```

Add HTTPS rule:

```powershell
lstk aws ec2 authorize-security-group-ingress --group-name ec2-learning-sg --protocol tcp --port 443 --cidr 0.0.0.0/0
```

Delete:

```powershell
lstk aws ec2 delete-security-group --group-name ec2-learning-sg
```

> In real AWS, avoid opening SSH to `0.0.0.0/0` unless there is a specific reason. Restrict access to the required IP/CIDR.

---

## EC2 — Instance

### Launch an Instance

#### Documentation

The basic information needed to launch an EC2 instance is:

```text
AMI
Instance Type
Key Pair
Security Group
```

#### Cheatsheet

```powershell
lstk aws ec2 run-instances --image-id <AMI_ID> --instance-type t2.micro --key-name ec2-learning-key --security-group-ids <SECURITY_GROUP_ID>
```

Important options:

| Option                 | Meaning                      |
| ---------------------- | ---------------------------- |
| `--image-id`           | AMI to launch                |
| `--instance-type`      | CPU and memory configuration |
| `--key-name`           | Key pair                     |
| `--security-group-ids` | Security group               |
| `--count`              | Number of instances          |

---

### Instance Commands

#### Documentation

Once an instance exists, these are the commands you'll reach for most — to inspect it, and to move it through start, stop, reboot, and terminate. (These same lifecycle commands are referenced again in the next section, where the instance states are explained.)

#### Cheatsheet

List (check what already exists, and get instance IDs to use below):

```powershell
lstk aws ec2 describe-instances
```

List as a table:

```powershell
lstk aws ec2 describe-instances --output table
```

View one instance:

```powershell
lstk aws ec2 describe-instances --instance-ids <INSTANCE_ID>
```

Start:

```powershell
lstk aws ec2 start-instances --instance-ids <INSTANCE_ID>
```

Stop:

```powershell
lstk aws ec2 stop-instances --instance-ids <INSTANCE_ID>
```

Reboot:

```powershell
lstk aws ec2 reboot-instances --instance-ids <INSTANCE_ID>
```

Terminate:

```powershell
lstk aws ec2 terminate-instances --instance-ids <INSTANCE_ID>
```

> **Terminate is destructive.** It removes the EC2 instance.

---

## EC2 — Instance Lifecycle

### Instance States

#### Documentation

An EC2 instance moves through different states during its lifecycle:

```text
pending
   |
   v
running
   |
   +---- stopping ----> stopped
   |
   +---- rebooting ----> running
   |
   +---- shutting-down ----> terminated
```

Simple meaning:

```text
start     = stopped -> running
stop      = running -> stopped
reboot    = restart the instance
terminate = remove the instance
```

The commands that drive these transitions (`start-instances`, `stop-instances`, `reboot-instances`, `terminate-instances`, and `describe-instances` to check the current state) are listed under **EC2 — Instance → Instance Commands** above.

---

## EC2 — EBS

### What is EBS?

#### Documentation

**EBS (Elastic Block Store)** provides block storage for EC2 instances.

Simple mental model:

```text
EC2 = Computer
EBS = Disk
```

---

### EBS Commands

#### Cheatsheet

List (check what already exists, and get volume IDs to use below):

```powershell
lstk aws ec2 describe-volumes
```

Create:

```powershell
lstk aws ec2 create-volume --size 10 --availability-zone us-east-1a
```

Attach:

```powershell
lstk aws ec2 attach-volume --volume-id <VOLUME_ID> --instance-id <INSTANCE_ID> --device /dev/sdf
```

Detach:

```powershell
lstk aws ec2 detach-volume --volume-id <VOLUME_ID>
```

Delete:

```powershell
lstk aws ec2 delete-volume --volume-id <VOLUME_ID>
```

---

## EC2 — VPC and Networking

### What is a VPC?

#### Documentation

**VPC (Virtual Private Cloud)** is a logically isolated virtual network.

```text
VPC
 |
 +-- Subnet
 |
 +-- Route Table
 |
 +-- Internet Gateway
 |
 +-- Security Group
 |
 +-- EC2
```

---

### VPC Commands

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-vpcs
```

---

### Subnet Commands

#### Documentation

A subnet is a slice of a VPC's address space, and each subnet lives in exactly one Availability Zone.

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-subnets
```

Important information:

```text
Subnet ID
VPC ID
Availability Zone
CIDR Block
```

Mental model:

```text
VPC: 10.0.0.0/16
 |
 +-- Subnet A: 10.0.1.0/24
 +-- Subnet B: 10.0.2.0/24
```

---

### Route Table Commands

#### Documentation

A route table decides where network traffic should go.

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-route-tables
```

---

### Internet Gateway Commands

#### Documentation

An Internet Gateway is what allows resources inside a VPC to reach — and be reached from — the internet.

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-internet-gateways
```

Mental model:

```text
   Internet
      |
      v
Internet Gateway
      |
      v
     VPC
      |
      v
   Subnet
      |
      v
     EC2
```

---

### Network Interface Commands

#### Documentation

An ENI is a virtual network card for an EC2 instance.

#### Cheatsheet

List:

```powershell
lstk aws ec2 describe-network-interfaces
```

---

## EC2 — AWS CLI Queries

### Why Use `--query`?

#### Documentation

`--query` lets you select only the information you need from a large AWS CLI response.

Use simple queries first while learning.

#### Cheatsheet

Get instance IDs:

```powershell
lstk aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId"
```

Get instance states:

```powershell
lstk aws ec2 describe-instances --query "Reservations[].Instances[].State.Name"
```

Get instance ID and state:

```powershell
lstk aws ec2 describe-instances --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name}" --output table
```

---

## EC2 — Complete Learning Workflow

### Step 1 — Start LocalStack

```powershell
lstk start --persist
```

### Step 2 — Verify AWS CLI

```powershell
lstk aws sts get-caller-identity
```

> If either step fails, see **Setup → Troubleshooting — LocalStack Itself** first.

### Step 3 — Check Regions

```powershell
lstk aws ec2 describe-regions
```

### Step 4 — Check AMIs

```powershell
lstk aws ec2 describe-images --output table
```

Find an AMI ID from your LocalStack environment.

### Step 5 — List Existing Key Pairs

```powershell
lstk aws ec2 describe-key-pairs
```

### Step 6 — Create Key Pair

```powershell
lstk aws ec2 create-key-pair --key-name ec2-learning-key
```

No key file is required for this learning exercise.

### Step 7 — List Existing Security Groups

```powershell
lstk aws ec2 describe-security-groups
```

### Step 8 — Create Security Group

```powershell
lstk aws ec2 create-security-group --group-name ec2-learning-sg --description "Security group for EC2 learning"
```

Note the returned Security Group ID.

### Step 9 — Launch Instance

```powershell
lstk aws ec2 run-instances --image-id <AMI_ID> --instance-type t2.micro --key-name ec2-learning-key --security-group-ids <SECURITY_GROUP_ID>
```

Note the returned Instance ID.

### Step 10 — Check Instance

```powershell
lstk aws ec2 describe-instances --output table
```

### Step 11 — Stop Instance

```powershell
lstk aws ec2 stop-instances --instance-ids <INSTANCE_ID>
```

### Step 12 — Start Instance

```powershell
lstk aws ec2 start-instances --instance-ids <INSTANCE_ID>
```

### Step 13 — Reboot Instance

```powershell
lstk aws ec2 reboot-instances --instance-ids <INSTANCE_ID>
```

### Step 14 — Terminate Instance

```powershell
lstk aws ec2 terminate-instances --instance-ids <INSTANCE_ID>
```

### Step 15 — Clean Up

Delete the security group:

```powershell
lstk aws ec2 delete-security-group --group-name ec2-learning-sg
```

Delete the key pair:

```powershell
lstk aws ec2 delete-key-pair --key-name ec2-learning-key
```

Delete any unused EBS volumes:

```powershell
lstk aws ec2 delete-volume --volume-id <VOLUME_ID>
```

---

## EC2 — Troubleshooting

### EC2 Command Is Not Working

#### Documentation

If LocalStack itself is confirmed healthy (see **Setup → Troubleshooting — LocalStack Itself**) but an EC2 command still isn't behaving as expected, start here.

#### Cheatsheet

```powershell
lstk aws ec2 describe-instances
```

If this fails or returns unexpected results, re-check LocalStack's health before assuming the EC2 command syntax is wrong:

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

---

### Resource ID Is Unknown

#### Documentation

Use the matching `describe` command instead of guessing a resource ID.

#### Cheatsheet

| Resource          | Command                       |
| ----------------- | ----------------------------- |
| AMI               | `describe-images`             |
| Instance          | `describe-instances`          |
| Security Group    | `describe-security-groups`    |
| Key Pair          | `describe-key-pairs`          |
| EBS Volume        | `describe-volumes`            |
| VPC               | `describe-vpcs`               |
| Subnet            | `describe-subnets`            |
| Route Table       | `describe-route-tables`       |
| Internet Gateway  | `describe-internet-gateways`  |
| Network Interface | `describe-network-interfaces` |

---

## EC2 — Quick Refresh

### Mental Model

```text
EC2 = Computer
AMI = Machine Image
Instance Type = Computer Size
Key Pair = Authentication
Security Group = Firewall
EBS = Disk
VPC = Network
Subnet = Smaller Network
Route Table = Traffic Direction
Internet Gateway = Internet Connection
ENI = Network Card
```

### Most Used Commands

#### Documentation

A single consolidated block of the commands you'll use in almost every session — handy for a fast refresh before an interview or after time away from the material.

#### Cheatsheet

```powershell
# Start LocalStack
lstk start

# Verify AWS CLI connection
lstk aws sts get-caller-identity

# List AMIs
lstk aws ec2 describe-images --output table

# List key pairs
lstk aws ec2 describe-key-pairs

# List security groups
lstk aws ec2 describe-security-groups

# List instances
lstk aws ec2 describe-instances --output table

# Start instance
lstk aws ec2 start-instances --instance-ids <INSTANCE_ID>

# Stop instance
lstk aws ec2 stop-instances --instance-ids <INSTANCE_ID>

# Reboot instance
lstk aws ec2 reboot-instances --instance-ids <INSTANCE_ID>

# Terminate instance
lstk aws ec2 terminate-instances --instance-ids <INSTANCE_ID>

# List EBS volumes
lstk aws ec2 describe-volumes --output table

# List VPCs
lstk aws ec2 describe-vpcs --output table

# List subnets
lstk aws ec2 describe-subnets --output table
```

---

## EC2 — Learning Checklist

### Fundamentals

- [ ] Understand EC2
- [ ] Understand EC2 instance
- [ ] Understand AMI
- [ ] Understand instance type
- [ ] Understand key pair
- [ ] Understand security group
- [ ] Understand EBS
- [ ] Understand VPC
- [ ] Understand subnet

### EC2 Commands

- [ ] describe-images
- [ ] describe-key-pairs
- [ ] create-key-pair
- [ ] describe-security-groups
- [ ] create-security-group
- [ ] run-instances
- [ ] describe-instances
- [ ] start-instances
- [ ] stop-instances
- [ ] reboot-instances
- [ ] terminate-instances

### Storage

- [ ] describe-volumes
- [ ] create-volume
- [ ] attach-volume
- [ ] detach-volume
- [ ] delete-volume

### Networking

- [ ] VPC
- [ ] Subnet
- [ ] CIDR
- [ ] Route Table
- [ ] Internet Gateway
- [ ] ENI
- [ ] Private IP
- [ ] Public IP
- [ ] Security Group

</details>

<!--
  For each service, keep the same pattern:

  ```text
  1. What is it?
  2. Core concepts
  3. Resource relationship
  4. Cheatsheet
  5. List / Read
  6. Create
  7. Update
  8. Delete
  9. Common workflow
  10. Troubleshooting
  11. Cleanup
  12. Learning checklist
  ```
-->

---

## 📃 References

- [LocalStack `lstk` CLI](https://docs.localstack.cloud/aws/developer-tools/running-localstack/lstk/)
- [LocalStack AWS CLI](https://docs.localstack.cloud/aws/connecting/aws-cli/)
- [LocalStack S3](https://docs.localstack.cloud/aws/services/s3/)
- [LocalStack EC2](https://docs.localstack.cloud/aws/services/ec2/)
- [AWS S3](https://docs.aws.amazon.com/s3/)
- [AWS EC2](https://docs.aws.amazon.com/ec2/)
