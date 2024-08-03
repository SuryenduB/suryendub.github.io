---
layout: post
title: AWS Cookbook 2.4 Connecting VPC to the Internet Using NAT Gateway
cover-img: assets/img/VPCNATDesign.png
thumbnail-img: assets/img/VPCNATDesign.png
share-img: assets/img/VPCNATDesign.png
tags: [ AWS , IAM , Networking , Permissions, NAT ]

---



## Table of Contents

- [Introduction](#introduction)
- [Design](#design)
- [Steps](#steps)
- [Validation Checks](#validation-checks)

## Introduction

NAT Gateway   can be used to allow outbound access from private subnet tier. It allows outbound access but does not provide direct inbound internet access.
EIP associated with NAT gateway becomes the external IP address for all the communication.

## Design

![Design](/assets/img/VPCNATDesign.png)

## Steps

### Create an Elastic IP for the NAT gateway

```shell
ALLOCATION_ID=$(aws ec2 allocate-address --domain vpc \
--output text --query AllocationId)
```

### Create a NAT Gateway within the Public subnet of AZ1

```
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway \
 --subnet-id $VPC_PUBLIC_SUBNET_1 \
--allocation-id $ALLOCATION_ID \
--output text --query NatGateway.NatGatewayId)
```

### Check the status of the NAT Gateway

```shell
aws ec2 describe-nat-gateways \
--nat-gateway-ids $NAT_GATEWAY_ID \
--output text --query 'NatGateways[0].State'

```

### Add a default route for 0.0.0.0/0 with destination as NAT Gateway

```shell
aws ec2 create-route --route-table-id $PRIVATE_RT_ID_1 \
--destination-cidr-block 0.0.0.0/0 \
--nat-gateway-id $NAT_GATEWAY_ID



aws ec2 create-route --route-table-id $PRIVATE_RT_ID_2 \
--destination-cidr-block 0.0.0.0/0 \
--nat-gateway-id $NAT_GATEWAY_ID
```

## Validation Check

Connect to EC2 Instance 1 and test internet outbound connection.

```shell
macbookpro@MBP-von-medneo cdk-AWS-Cookbook-204 % aws ssm start-session --target $INSTANCE_ID_1

Starting session with SessionId: root-tvnwq2qs3wnppjvlnvjbzy3tcu
sh-4.2$ ping google.com
PING google.com (172.217.16.206) 56(84) bytes of data.
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=1 ttl=57 time=2.29 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=2 ttl=57 time=1.03 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=3 ttl=57 time=1.13 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=4 ttl=57 time=1.13 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=5 ttl=57 time=1.09 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=6 ttl=57 time=1.66 ms
64 bytes from fra16s08-in-f206.1e100.net (172.217.16.206): icmp_seq=7 ttl=57 time=1.05 ms
```
