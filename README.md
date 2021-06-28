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

- Run the TFE installer

```bash
curl https://install.terraform.io/ptfe/stable | sudo bash
```

Sample output

```bash
$ curl https://install.terraform.io/ptfe/stable | sudo bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  145k  100  145k    0     0  15427      0  0:00:09  0:00:09 --:--:-- 36091
Determining local address
The installer was unable to automatically detect the private IP address of this machine.
Please choose one of the following network interfaces:
[0] enp0s3	10.0.2.15
[1] enp0s8	192.168.56.33
Enter desired number (0-1): 

```

- Enter number which corresponds IP address 192.168.56.33. It's 1 in the current example.

```bash
1
```

Sample output

```bash
Enter desired number (0-1): 1
The installer will use network interface 'enp0s8' (with IP address '192.168.56.33').
Determining service address
The installer was unable to automatically detect the service IP address of this machine.
Please enter the address or leave blank for unspecified.
Service IP address: 
```

- Push "Enter" key

Sample output

```bash
The installer was unable to automatically detect the service IP address of this machine.
Please enter the address or leave blank for unspecified.
Service IP address: 
Does this machine require a proxy to access the Internet? (y/N) 
```

- Push "N" key

Sample output

```bash
Does this machine require a proxy to access the Internet? (y/N) n
Installing docker version 19.03.8 from https://get.replicated.com/docker-install.sh
# Executing docker install script, commit: b50fac9
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | apt-key add -qq - >/dev/null
Warning: apt-key output should not be parsed (stdout is not a terminal)
+ sh -c echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
INFO: Searching repository for VERSION '19.03.8'
INFO: apt-cache madison 'docker-ce' | grep '19.03.8.*-0~ubuntu' | head -1 | awk '{$1=$1};1' | cut -d' ' -f 3
+ [ -n 5:19.03.8~3-0~ubuntu-bionic ]
+ sh -c apt-get install -y -qq --no-install-recommends docker-ce-cli=5:19.03.8~3-0~ubuntu-bionic >/dev/null
+ sh -c apt-get install -y -qq --no-install-recommends docker-ce=5:19.03.8~3-0~ubuntu-bionic >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b7f0
 Built:             Wed Mar 11 01:25:46 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.17
  Git commit:       afacb8b7f0
  Built:            Wed Mar 11 01:24:19 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.4
  GitCommit:        05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc:
  Version:          1.0.0-rc93
  GitCommit:        12644e614e25b05da6fd08a38ffa0cfe1903fdec
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
External script is finished
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker

Running preflight checks...
[INFO] / disk usage is at 14%
[INFO] /var/lib/docker disk usage is at 1%
[INFO] Docker http proxy not set
[INFO] Docker using default seccomp profile
[INFO] Docker using standard root directory
[INFO] Docker icc (inter-container communication) enabled
[INFO] Docker open files (nofile) ulimit not set
[INFO] Docker userland proxy enabled
[INFO] Firewalld is not active
[INFO] Iptables chain INPUT default policy ACCEPT
Pulling replicated and replicated-ui images
stable-2.52.0: Pulling from replicated/replicated
Image docker.io/replicated/replicated:stable-2.52.0 uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
69692152171a: Pull complete 
4cec2ee532ca: Pull complete 
5e5336345ecc: Pull complete 
8c9af647b5fd: Pull complete 
f29b1c17ff51: Pull complete 
fce3e81d9123: Pull complete 
d94558f8f63a: Pull complete 
657e54daabf4: Pull complete 
dfa55743770a: Pull complete 
82a73fe0e9b9: Pull complete 
9c9b8421b4c8: Pull complete 
Digest: sha256:d38833cb625b0e226471f59f8a894469e76c44035288d9a57560bb8620b114b4
Status: Downloaded newer image for replicated/replicated:stable-2.52.0
docker.io/replicated/replicated:stable-2.52.0
stable-2.52.0: Pulling from replicated/replicated-ui
Image docker.io/replicated/replicated-ui:stable-2.52.0 uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
69692152171a: Already exists 
d26ac9469008: Pull complete 
92d885dd2446: Pull complete 
0a0630b58335: Pull complete 
8db0096a6f1e: Pull complete 
95d2036e9098: Pull complete 
Digest: sha256:7c5a916b9e265fa23677b797d278c44ac9bab3ba6f402927cc1fa6a43e5d6348
Status: Downloaded newer image for replicated/replicated-ui:stable-2.52.0
docker.io/replicated/replicated-ui:stable-2.52.0
Tagging replicated and replicated-ui images
Stopping replicated and replicated-ui service
Installing replicated and replicated-ui service
Starting replicated and replicated-ui service
Created symlink /etc/systemd/system/docker.service.wants/replicated.service → /etc/systemd/system/replicated.service.
Created symlink /etc/systemd/system/docker.service.wants/replicated-ui.service → /etc/systemd/system/replicated-ui.service.
Installing replicated command alias
Installing local operator
Installing local operator with command:
curl -sSL https://get.replicated.com/operator?replicated_operator_tag=2.52.0
Pulling latest replicated-operator image
stable-2.52.0: Pulling from replicated/replicated-operator
Image docker.io/replicated/replicated-operator:stable-2.52.0 uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
69692152171a: Already exists 
b8d1591cc3e8: Pull complete 
99408039ae2c: Pull complete 
f9266063add8: Pull complete 
3669a4e3d61c: Pull complete 
69bf04578910: Pull complete 
bc03da138eb1: Pull complete 
Digest: sha256:bd80d36e0a6dfe0ef8a50e858db9680418a072cd53f084caf4b488a294d8fcb9
Status: Downloaded newer image for replicated/replicated-operator:stable-2.52.0
docker.io/replicated/replicated-operator:stable-2.52.0
Tagging replicated-operator image
Stopping replicated-operator service
Installing replicated-operator service
Starting replicated-operator service
Created symlink /etc/systemd/system/docker.service.wants/replicated-operator.service → /etc/systemd/system/replicated-operator.service.

Operator installation successful

To continue the installation, visit the following URL in your browser:

  http://<this_server_address>:8800
```

- In your browser open URL http://<this_server_address>:8800/

- Click Advanced - Proceed to 192.168.56.33 (unsafe)

- Enter `192.168.56.33.nip.io` to Hostname field and click `Use Self-Signed Cert`

- In your browser open URL https://192.168.56.33.nip.io:8800/

- Click `Choose license` and select your TFE license file. Then click `Open`

- Click `Online`

- Click `Continue`

- Enter password `Password1#` to Password and Confirm password fields. Click `Continue`

![Password](https://github.com/antonakv/tfe-vagrant-online-pmd/raw/main/images/tfe-i-vagrant-9.png)

- On the next screen scroll down and click `Continue`

![Proceed anyway](https://github.com/antonakv/tfe-vagrant-online-pmd/raw/main/images/tfe-i-vagrant-10.png)

- On the Settings page set `Encryption password` as `Password1#` and select installation type `Production`. 

![Encryption password](https://github.com/antonakv/tfe-vagrant-online-pmd/raw/main/images/tfe-i-vagrant-12.png)

- Select Production type `External services` 

- In `PostgreSQL Configuration` fill values to corresponding fields

- Put `postgres` to `username` field

- Put `Password1#` to `password` field

- Put `aakulov-tfe.cu1fxtceaf9d.eu-central-1.rds.amazonaws.com` to `hostname and optional port` field

- Put `hashicorp` to `database name` field

- Put `sslmode=disable` to `optional extra parameters`


 