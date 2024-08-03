---
layout: post
title: AWS Cookbook 2.3 Connecting VPC to the Internet
cover-img: /assets/img/AWSVPCInternet.png
thumbnail-img: /assets/img/AWSVPCInternet.png
share-img: /assets/img/AWSVPCInternet.png
tags: [ AWS , IAM , Networking , Permissions, Gateway ]

---
## Table of Contents

- [Introduction](#introduction)
- [Design](#design)
- [Steps](#steps)
  - [Set variables for the deployment](#set-variables-for-the-deployment)
  - [Create and Tag VPC](#create-and-tag-vpc)
  - [Create Subnets, Route Tables and Associate the Route Tables with respective subnets](#create-subnets-route-tables-and-associate-the-route-tables-with-respective-subnets)
  - [Create IAM Role for EC2 instance with SSM permissions](#create-iam-role-for-ec2-instance-with-ssm-permissions)
  - [Create an EC2 Instance with latest Amazon Linux Image](#create-an-ec2-instance-with-latest-amazon-linux-image)
  - [Create an Internet Gateway(IGW)](#create-an-internet-gatewayigw)
  - [Attach the internet gateway to the VPC](#attach-the-internet-gateway-to-the-vpc)
  - [Add route to the Internet Gateway](#add-route-to-the-internet-gateway)
  - [Create an EIP and Associate with EC2 Instance](#create-an-eip-and-associate-with-ec2-instance)
- [Validation Checks](#validation-checks)
- [Cleanup](#cleanup)

## Introduction

A subnet that has a route of **0.0.0.0/0** associated with an IGW is considered as a public subnet.End user facing load balancers are commonly placed in public subnets.

## Design

![image](/assets/img/AWSVPCInternet.png)

## Steps

### Set variables for the deployment

```shellshell
VPC_CIDR="10.10.0.0/23"
SUBNET_CIDR_1="10.10.0.0/24"
SUBNET_CIDR_2="10.10.1.0/24"
REGION="eu-central-1"  
INSTANCE_TYPE="t2.micro"

```

### Create and Tag VPC

```shell
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --query 'Vpc.VpcId' --output text)
aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=AWS-Cookbook-Suryendu-VPC

```

### Create Subnets, Route Tables and Associate the Route Tables with respective subnets

```shell

SUBNET_ID_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $SUBNET_CIDR_1 --availability-zone ${REGION}a --query 'Subnet.SubnetId' --output text)

SUBNET_ID_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block $SUBNET_CIDR_2 --availability-zone ${REGION}b --query 'Subnet.SubnetId' --output text)

aws ec2 create-tags --resources $SUBNET_ID_1 --tags Key=Name,Value=Tier1Subnet1

aws ec2 create-tags --resources $SUBNET_ID_2 --tags Key=Name,Value=Tier1Subnet2

ROUTE_TABLE_ID_1=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)

ROUTE_TABLE_ID_2=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)

aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID_1 --subnet-id $SUBNET_ID_1

aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID_2 --subnet-id $SUBNET_ID_2

aws ec2 create-tags --resources $ROUTE_TABLE_ID_1 --tags Key=Name,Value=AWSCookbookSuryenduSubnet1RouteTable1

aws ec2 create-tags --resources $ROUTE_TABLE_ID_2 --tags Key=Name,Value=AWSCookbookSuryenduSubnet2RouteTable2

```

![image](/assets/img/AWSVPCPortal.png)

### Create IAM Role for EC2 instance with SSM permissions

```shell

ROLE_NAME="AWSCookBookSuryenduInstanceSSMRole"

aws iam create-role --role-name $ROLE_NAME --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}'

aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

INSTANCE_PROFILE_NAME="AWSCookBookSuryenduInstanceProfile"

aws iam create-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME

aws iam add-role-to-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME --role-name $ROLE_NAME

```

### Create an EC2 Instance with latest Amazon Linux Image

```shell

AMI_ID=$(aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" --query 'Images[*].[ImageId,CreationDate]' --output text | sort -k2 -r | head -n1 | awk '{print $1}')

INSTANCE_ID=$(aws ec2 run-instances --image-id $AMI_ID --instance-type $INSTANCE_TYPE  --subnet-id $SUBNET_ID_1 --iam-instance-profile Name=$INSTANCE_PROFILE_NAME --query 'Instances[0].InstanceId' --output text)

aws ec2 create-tags --resources $INSTANCE_ID --tags Key=Name,Value=AWSCookBookSuryenduInstance

```

### Create an Internet Gateway(IGW)

```shell

INET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
--tag-specifications \
'ResourceType=internet-gateway,Tags=[{Key=Name,Value=AWSCookbookSuryendu202}]' \
--output text --query InternetGateway.InternetGatewayId)

```

### Attach the internet gateway to the VPC

```shell
aws ec2 attach-internet-gateway \
--internet-gateway-id $INET_GATEWAY_ID --vpc-id $VPC_ID

```

![image](/assets/img/Internet%20Gateway%20Portal.png)

### Add route to the Internet Gateway

In each previously created route table we add a route that sets the default route destination to the internet gateway.

```shell

aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_1 \
--destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID


aws ec2 create-route --route-table-id $ROUTE_TABLE_ID_2 \
--destination-cidr-block 0.0.0.0/0 --gateway-id $INET_GATEWAY_ID

```

![image](/assets/img/Route.png)

### Create an EIP and Associate with EC2 Instance

```shell

ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc \
--output text --query AllocationId)

aws ec2 associate-address \
--instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID

```shell

## Validation Checks

We can log into EC2 instance using SSM Session Manager and validate internet connectivity by pinging a host.

```shell

aws ec2 associate-address \
--instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID

```

![image](/assets/img/EC2Ping.png)

## Cleanup

```shell

#!/bin/bash

# Set variables (Ensure these values match those used during creation)
VPC_CIDR="10.10.0.0/23"
SUBNET_CIDR_1="10.10.0.0/24"
SUBNET_CIDR_2="10.10.1.0/24"
REGION="eu-central-1"  
INSTANCE_TYPE="t2.micro"

# Fetch the IDs of resources to be deleted

# Instance ID
INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=instance-type,Values=$INSTANCE_TYPE" "Name=tag:Name,Values=AWSCookBookSuryenduInstance" --query 'Reservations[*].Instances[*].InstanceId' --output text)

# Elastic IP Allocation ID
ALLOCATION_ID=$(aws ec2 describe-addresses --filters "Name=instance-id,Values=$INSTANCE_ID" --query 'Addresses[*].AllocationId' --output text)

# Internet Gateway ID
INET_GATEWAY_ID=$(aws ec2 describe-internet-gateways --filters "Name=attachment.vpc-id,Values=$VPC_ID" --query 'InternetGateways[*].InternetGatewayId' --output text)

# VPC ID
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=cidr,Values=$VPC_CIDR" --query 'Vpcs[*].VpcId' --output text)

# Subnet IDs
SUBNET_ID_1=$(aws ec2 describe-subnets --filters "Name=cidr-block,Values=$SUBNET_CIDR_1" --query 'Subnets[0].SubnetId' --output text)
SUBNET_ID_2=$(aws ec2 describe-subnets --filters "Name=cidr-block,Values=$SUBNET_CIDR_2" --query 'Subnets[0].SubnetId' --output text)

# Route Table IDs
ROUTE_TABLE_ID_1=$(aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=$SUBNET_ID_1" --query 'RouteTables[0].RouteTableId' --output text)
ROUTE_TABLE_ID_2=$(aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=$SUBNET_ID_2" --query 'RouteTables[0].RouteTableId' --output text)

# Endpoint IDs
SSM_ENDPOINT_ID=$(aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$VPC_ID" "Name=service-name,Values=com.amazonaws.$REGION.ssm" --query 'VpcEndpoints[0].VpcEndpointId' --output text)
EC2_MESSAGES_ENDPOINT_ID=$(aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$VPC_ID" "Name=service-name,Values=com.amazonaws.$REGION.ec2messages" --query 'VpcEndpoints[0].VpcEndpointId' --output text)
SSM_MESSAGES_ENDPOINT_ID=$(aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$VPC_ID" "Name=service-name,Values=com.amazonaws.$REGION.ssmmessages" --query 'VpcEndpoints[0].VpcEndpointId' --output text)

# IAM Role and Instance Profile
ROLE_NAME="AWSCookBookSuryenduInstanceSSMRole"
INSTANCE_PROFILE_NAME="AWSCookBookSuryenduInstanceProfile"

# Delete EC2 Instance
if [ -n "$INSTANCE_ID" ]; then
    aws ec2 terminate-instances --instance-ids $INSTANCE_ID
    aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID
fi

# Disassociate and Release Elastic IP
if [ -n "$ALLOCATION_ID" ]; then
    aws ec2 disassociate-address --allocation-id $ALLOCATION_ID
    aws ec2 release-address --allocation-id $ALLOCATION_ID
fi

# Delete Endpoints
if [ -n "$SSM_ENDPOINT_ID" ]; then
    aws ec2 delete-vpc-endpoint --vpc-endpoint-id $SSM_ENDPOINT_ID
fi

if [ -n "$EC2_MESSAGES_ENDPOINT_ID" ]; then
    aws ec2 delete-vpc-endpoint --vpc-endpoint-id $EC2_MESSAGES_ENDPOINT_ID
fi

if [ -n "$SSM_MESSAGES_ENDPOINT_ID" ]; then
    aws ec2 delete-vpc-endpoint --vpc-endpoint-id $SSM_MESSAGES_ENDPOINT_ID
fi

# Detach and Delete Internet Gateway
if [ -n "$INET_GATEWAY_ID" ]; then
    aws ec2 detach-internet-gateway --internet-gateway-id $INET_GATEWAY_ID --vpc-id $VPC_ID
    aws ec2 delete-internet-gateway --internet-gateway-id $INET_GATEWAY_ID
fi

# Delete Route Table Associations
ASSOCIATION_IDS=$(aws ec2 describe-route-tables --route-table-ids $ROUTE_TABLE_ID_1 $ROUTE_TABLE_ID_2 --query 'RouteTables[*].Associations[*].RouteTableAssociationId' --output text)

for ASSOCIATION_ID in $ASSOCIATION_IDS; do
    aws ec2 disassociate-route-table --association-id $ASSOCIATION_ID
done

# Delete Routes
if [ -n "$ROUTE_TABLE_ID_1" ]; then
    aws ec2 delete-route --route-table-id $ROUTE_TABLE_ID_1 --destination-cidr-block 0.0.0.0/0
fi

if [ -n "$ROUTE_TABLE_ID_2" ]; then
    aws ec2 delete-route --route-table-id $ROUTE_TABLE_ID_2 --destination-cidr-block 0.0.0.0/0
fi

# Delete Route Tables
if [ -n "$ROUTE_TABLE_ID_1" ]; then
    aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID_1
fi

if [ -n "$ROUTE_TABLE_ID_2" ]; then
    aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID_2
fi

# Delete Subnets
if [ -n "$SUBNET_ID_1" ]; then
    aws ec2 delete-subnet --subnet-id $SUBNET_ID_1
fi

if [ -n "$SUBNET_ID_2" ]; then
    aws ec2 delete-subnet --subnet-id $SUBNET_ID_2
fi

# Delete VPC
if [ -n "$VPC_ID" ]; then
    aws ec2 delete-vpc --vpc-id $VPC_ID
fi

# Delete IAM Role and Instance Profile
if [ -n "$INSTANCE_PROFILE_NAME" ]; then
    aws iam remove-role-from-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME --role-name $ROLE_NAME
    aws iam delete-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME
fi

if [ -n "$ROLE_NAME" ]; then
    aws iam detach-role-policy --role-name $ROLE_NAME --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
    aws iam delete-role --role-name $ROLE_NAME
fi

echo "Cleanup complete."

```
