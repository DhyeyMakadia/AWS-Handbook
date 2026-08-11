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

</details>

---

<details open>

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

Create:

```powershell
lstk aws ec2 create-key-pair --key-name ec2-learning-key
```

List:

```powershell
lstk aws ec2 describe-key-pairs
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

Create:

```powershell
lstk aws ec2 create-security-group --group-name ec2-learning-sg --description "Security group for EC2 learning"
```

List:

```powershell
lstk aws ec2 describe-security-groups
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

List:

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

List:

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

### Step 2 — Check LocalStack

```powershell
lstk status
```

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

### Step 3 — Verify AWS CLI

```powershell
lstk aws sts get-caller-identity
```

### Step 4 — Check Regions

```powershell
lstk aws ec2 describe-regions
```

### Step 5 — Check AMIs

```powershell
lstk aws ec2 describe-images --output table
```

Find an AMI ID from your LocalStack environment.

### Step 6 — Create Key Pair

```powershell
lstk aws ec2 create-key-pair --key-name ec2-learning-key
```

No key file is required for this learning exercise.

### Step 7 — Create Security Group

```powershell
lstk aws ec2 create-security-group --group-name ec2-learning-sg --description "Security group for EC2 learning"
```

Note the returned Security Group ID.

### Step 8 — Launch Instance

```powershell
lstk aws ec2 run-instances --image-id <AMI_ID> --instance-type t2.micro --key-name ec2-learning-key --security-group-ids <SECURITY_GROUP_ID>
```

Note the returned Instance ID.

### Step 9 — Check Instance

```powershell
lstk aws ec2 describe-instances --output table
```

### Step 10 — Stop Instance

```powershell
lstk aws ec2 stop-instances --instance-ids <INSTANCE_ID>
```

### Step 11 — Start Instance

```powershell
lstk aws ec2 start-instances --instance-ids <INSTANCE_ID>
```

### Step 12 — Reboot Instance

```powershell
lstk aws ec2 reboot-instances --instance-ids <INSTANCE_ID>
```

### Step 13 — Terminate Instance

```powershell
lstk aws ec2 terminate-instances --instance-ids <INSTANCE_ID>
```

### Step 14 — Clean Up

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

### LocalStack Is Not Running

#### Cheatsheet

```powershell
lstk status
```

If needed:

```powershell
lstk start
```

---

### Check LocalStack Health

#### Cheatsheet

```powershell
curl.exe http://localhost.localstack.cloud:4566/_localstack/health
```

---

### EC2 Command Is Not Working

#### Cheatsheet

```powershell
lstk aws ec2 describe-instances
```

Then check LocalStack health:

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

# Check LocalStack status
lstk status

# Check LocalStack service health
curl.exe http://localhost.localstack.cloud:4566/_localstack/health

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
- [ ] create-key-pair
- [ ] describe-key-pairs
- [ ] create-security-group
- [ ] describe-security-groups
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

<!--
  For each service, keep the same pattern:

  ```text
  1. What is it?
  2. Core concepts
  3. Resource relationship
  4. Cheatsheet
  5. Create
  6. List / Read
  7. Update
  8. Delete
  9. Common workflow
  10. Troubleshooting
  11. Cleanup
  12. Learning checklist
  ```
-->
</details>

---

## 📃 References

- [LocalStack `lstk` CLI](https://docs.localstack.cloud/aws/developer-tools/running-localstack/lstk/)
- [LocalStack EC2](https://docs.localstack.cloud/aws/services/ec2/)
- [LocalStack AWS CLI](https://docs.localstack.cloud/aws/connecting/aws-cli/)
- [AWS EC2](https://docs.aws.amazon.com/ec2/)
