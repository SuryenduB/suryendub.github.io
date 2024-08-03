---
layout: post
title: AWS Cookbook 2.2 Creating a Network Tier with Subnets and a Route Table in a VPC
cover-img: /assets/img/AWSVPCDesign.png
thumbnail-img: /assets/img/AWSVPCDesign.png
share-img: /assets/img/AWSVPCDesign.png
tags: [ AWS , IAM , Networking , Permissions ]

---
# AWS Cookbook 2.2 Creating a Network Tier with Subnets and a Route Table in a VPC

Here's the Table of Contents (TOC) for the provided document:

## Table of Contents

1. [Introduction](#introduction)
2. [Design](#design)
3. [Steps](#steps)
   1. [Deploy a VPC](#deploy-a-vpc)
   2. [Create a Route Table](#create-a-route-table)
   3. [Create Subnets](#create-subnets)
   4. [Associate Route Table with Subnets](#associate-route-table-with-subnets)
4. [Validation Checks](#validation-checks)
5. [Cleanup](#cleanup)
   1. [Delete your subnets](#delete-your-subnets)
   2. [Delete your route table](#delete-your-route-table)
   3. [Delete your VPC](#delete-your-vpc)
   4. [Unset your manually created environment variables](#unset-your-manually-created-environment-variables)

#aws/cookbook/Networking

## Introduction

When creating a VPC in a Region, it is a best practice to spread subnet across AZs in the networking tier. The number of AZs different per region, but most have at least three.
A Subnet has one route table associated with it. Route Tables can be associated with one or more subnets and direct traffic  to a destination. Entries within route tables are called routes and are defined as pairs of Destination and Targets. When a route table is created a default local route is added for intra-VPC traffic.


## Design

![Design](/assets/img/AWS%20VPC%20Design.png)

## Steps

* Deploy a VPC.

```shell
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.10.0.0/23 \
--tag-specifications \
'ResourceType=vpc,Tags=[{Key=Name,Value=AWSCookbook202}]' \
--output text --query Vpc.VpcId)

```

* Create a route table to customise traffic routes for subnets.

```shell
ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID \
--tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=AWSCookbookSuryendu202}]' \
--output text --query RouteTable.RouteTableId )


```

* Create two Subnets in each Az to define address spaces for creating resources.

```shell
SUBNET_ID_1=$(aws ec2 create-subnet \
--vpc-id $VPC_ID \
--cidr-block 10.10.0.0/24 \
--availability-zone ${AWS_REGION}a \
--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbookSuryendu202a}]' \
--output text --query Subnet.SubnetId )


SUBNET_ID_2=$(aws ec2 create-subnet \
--vpc-id $VPC_ID \
--cidr-block 10.10.1.0/24 \
--availability-zone ${AWS_REGION}b \
--tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=AWSCookbookSuryendu202b}]' \
--output text --query Subnet.SubnetId )


```

* Associate the route_table created in step 2 with the Subnets.

```shell

aws ec2 associate-route-table \
--route-table $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_1

aws ec2 associate-route-table \
--route-table $ROUTE_TABLE_ID --subnet-id $SUBNET_ID_2

```

* This generates an output similar to this respectively for each subnet.

```json
{
    "AssociationId": "rtbassoc-0c58de60a9584de72",
    "AssociationState": {
        "State": "associated"
    }
}
```

```json
{
    "AssociationId": "rtbassoc-0a7e8c81bae6bd16c",
    "AssociationState": {
        "State": "associated"
    }
}
```

## Validation Checks

We can describe each resource to validate deployments.

1. Subnet 1
![Subnet 1](/assets/img/AWSSubnet1.png)
2. Subnet 2
![Subnet 2](/assets/img/AWS_Subnet2.png)
3. Route Table
![Route Table](/assets/img/Route%20Table.png)

## Cleanup

### Delete your subnets:

`aws ec2 delete-subnet --subnet-id $SUBNET_ID_1`

`aws ec2 delete-subnet --subnet-id $SUBNET_ID_2`

### Delete your route table:

`aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID`

### Delete your VPC:

`aws ec2 delete-vpc --vpc-id $VPC_ID`

### Unset your manually created environment variables

```shell
unset VPC_ID
unset ROUTE_TABLE_ID
unset SUBNET_ID_1
unset SUBNET_ID_2
```
