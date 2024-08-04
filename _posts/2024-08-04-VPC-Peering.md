---
layout: post
title: AWS Cookbook 2.11 Peering Two VPCs Together
cover-img: assets/img/VPCPeering2.png
thumbnail-img: assets/img/VPCPeering2.png
share-img: assets/img/VPCPeering2.png
tags: [ AWS , IAM , Networking , Permissions, Peering ]

---


## Table of Contents

- [Introduction](#introduction)
- [Design](#design)
- [Steps](#steps)
- [Validation Checks](#validation-checks)

## Introduction

We need to enable communication between two instances in separate VPCs.
Here is the existing architecture

- Two VPCs each with isolated subnets in two AZs and associated route tables.

- In each VPC exists one EC2 instance.

![](/assets/img/VPCPeering1.png)

## Design

We need to request a peering connection between two VPCs, accept the peering connection, update the route table for each vac subnet.

![](/assets/img/VPCPeering2.png)

## Steps

1. Create a VPC Peering connection to connect VPC1 and VPC2.

```shell
VPC_PEERING_CONNECTION_ID=$(aws ec2 create-vpc-peering-connection \
--vpc-id $VPC_ID_1 --peer-vpc-id $VPC_ID_2 --output text \
--query VpcPeeringConnection.VpcPeeringConnectionId)
```

2. Accept the peering connection

```shell
aws ec2 accept-vpc-peering-connection \
--vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID
```

![](/assets/img/VPCPeeringAcceptOutput.png)

3. Add a route in each subnet to direct traffic destined for the peered Net to the VPC_PEERING_CONNECTION_ID

```shell
aws ec2 create-route --route-table-id $VPC_SUBNET_RT_ID_1 \
--destination-cidr-block $VPC_CIDR_2 \
--vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID

aws ec2 create-route --route-table-id $VPC_SUBNET_RT_ID_2 \
--destination-cidr-block $VPC_CIDR_1 \
--vpc-peering-connection-id $VPC_PEERING_CONNECTION_ID

```

4. Add an Ingress rule to instance 2’s security group to allow ping from instance 1.

```
aws ec2 authorize-security-group-ingress \
--protocol icmp --port -1 \
--source-group $INSTANCE_SG_1 \
--group-id $INSTANCE_SG_2

```

## Validation Checks

We connect to the EC2 instance by using SSM Session Manager:

```shell
aws ssm start-session --target $INSTANCE_ID_1
```

![](/assets/img/VPCPeeringValidation.png)