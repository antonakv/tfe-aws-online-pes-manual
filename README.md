# tfe-aws-online-pes
AWS interactive Prod Online External Services PES

## Intro

This manual is dedicated for Terraform Enterprise online interactive installation on Amazon AWS.

## Requirements

- Access to Amazon AWS console

## Preparation 

- Login to Amazon AWS console

- Click `Services -> VPC`

- Click `Your VPCs`

- Click `Create VPC`

- Fill fields Name with `myvpc` and IPc4 CIDR block with `10.3.0.0/16`

- Click `Create VPC`

- Click `Subnets`

- Click `Create subnet`

- In `VPC ID` select `VPC` called `myvpc`

- In `Subnet name` put value `mysubnet1`

- In `IPv4 CIDR block` put value `10.3.1.0/24`

- Click `Create subnet`

- Click `VPC - Internet gateways`

- Click `Create Internet gateway`

- Put `myigw` to name field

- Click `Create Internet gateway`

- Click `Attach to VPC`

- In list `Available VPCs` select `myvpc` and click `Attach internet gateway`

- Open `VPC - Security Groups`

