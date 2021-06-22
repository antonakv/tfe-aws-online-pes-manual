# tfe-aws-online-pes
AWS interactive Prod Online External Services PES

## Intro

This manual is dedicated for Terraform Enterprise online interactive installation on Amazon AWS.

## Requirements

- Access to Amazon AWS console

## Preparation 

### Spin up EC2 instance

- Login to Amazon AWS console

- Click `Services -> VPC`

- Click `Your VPCs`

- Click `Create VPC`

- Fill fields Name with `myvpc` and IPc4 CIDR block with `10.3.0.0/16`

- Click `Create VPC`

- Click `Subnets`

- Click `Create subnet`

- In `VPC ID` field select `VPC` called `myvpc`

- In `Subnet name` put value `mysubnet1`

- In `IPv4 CIDR block` put value `10.3.1.0/24`

- Click `Create subnet`

- Click `VPC - Internet gateways`

- Click `Create Internet gateway`

- Put `myigw` to Name field

- Click `Create Internet gateway`

- Click `Attach to VPC`

- In list `Available VPCs` select `myvpc` and click `Attach internet gateway`

- Open `VPC - Security Groups`

- Create security group `my-tfe-online`

- Open `VPC - Route tables`

- Click `Create route table`

- Put `myroute` to Name field

- In `VPC ID` field select `VPC` called `myvpc`

- Click `Create route table`

- In just opened route table click `Edit routes`

- Click `Add route`

- In destination field put `0.0.0.0/0`

- In Target field select `Internet gateway`

- In Target field select `myigw`

- Click `Save changes`

- Open `EC2 - Instances`

- Click `Launch instances`

- Click Select on `Amazon Linux 2 AMI (HVM) - ami-0db9040eb3ab74509 (64-bit x86)`

- Select instance type `t2.large`

- Click `Next: Configure Instance Details`

- In field `Network` select `myvpc`

- In field `Auto-assign Public IP` select `Enable`

- Click `Next: Add Storage`

- In field `Size (GiB)` put `50`

- Click `Review and Launch`

- Click `Launch`

### Create Postgresql RDS database

- Open AWS console

- Create/reuse your own VPC with at least 2 subnets each one is linked to different Availability zone  

- Search for `RDS` and open it

- Click `Create database`

- Select type `PostgreSQL`

- Select Postgresql version supported in TFE installation requirements

- In section `Templates` select `Free tier`

- Call database with your name for example mydb-tfe

- Set master password as `Password1#`

- In Connectivity select your VPC prepared on the previous step

- In Connectivity public access select No

- Select Security group you already have for TFE

- Click Create database

- If you had error message “Your request to create DB instance mydb-tfe didn't work” - read the manual again

- Open AWS console - `VPC - Security groups`

- Click on security group used both for database instance and tfe instance

- Click `Edit inbound rules`

- Click `Add rule`

- Type: Postgresql Source: 0.0.0.0/0

- Click `Save rules `

- ssh to tfe aws instance

- install postgesql client. Run `sudo yum install postgresql`

- connect to database `psql -U postgres -h aakulov-tfe.cu1fxtceaf9d.eu-central-1.rds.amazonaws.com`

- create database for tfe `create database "aakulov-tfe" TEMPLATE template0 owner postgres;`

- exit from psql with `Ctrl + D`

### Setup S3 bucket

- Open AWS console `S3`

- Click `Create bucket`

- Fill field Bucket name

- Save bucket name

- Select region where you’re going to install TFE

- Click `Create bucket`

- Select `TFE S3 policy `

- Open `IAM` in AWS console

- Open `Roles`

- Click `Create role`

- Click `EC2`

- Click `Next: Permissions`

- Click `Create policy`

- Click `Next: tags`

- Click `Next: Review`

- In a field `Role name` put value `myuser-iam-role-tfe-s3`

- Click `Create role`

- Open `IAM` in AWS console

- Open `Policies`

- Click `Create policy`

- Click `JSON`

- Paste following

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:ListStorageLensConfigurations",
                "s3:ListAccessPointsForObjectLambda",
                "s3:GetAccessPoint",
                "s3:PutAccountPublicAccessBlock",
                "s3:GetAccountPublicAccessBlock",
                "s3:ListAllMyBuckets",
                "s3:ListAccessPoints",
                "s3:ListJobs",
                "s3:PutStorageLensConfiguration",
                "s3:CreateJob"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::aakulov"
        }
    ]
}
```

- Click `Next: Tags`

- Click `Next: Review`

- In the field Name put value `myuser-tfe-s3-profile`

- Click `Create policy`

- Open `EC2`

- Select your instance with tfe

- Click `Actions`

- Click `Security`

- Click `Modify IAM role`

- Select `myuser-iam-role-tfe-s3` from the list

- Click `Save`

### Create Route 53 domain record for TFE installation

- Open AWS console

- Open `Route53`

- Open `Hosted zones` https://console.aws.amazon.com/route53/v2/hostedzones#

- Click `Create`

- Create new zone `anton.hashicorp-success.com`

- Click `Create hosted zone`

- Grab contents of NS record for the new zone

- Back to the hosted zones

- Click on zone hashicorp-success.com

- Create record with type NS

- Call NS record as anton

- Fill NS servers to value field

- Set TTL to 1m

- Click `Create record`

- Create A record in anton.hashicorp-success.com called tfe1 

- Set ttl of A record as 1m

- Click `Create`

## Installation

- ssh to EC2 instance using public IP

