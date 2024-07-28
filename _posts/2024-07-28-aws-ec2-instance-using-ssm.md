---
layout: post
title: AWS CookBook 1.6 Connecting to EC2 Instances using AWS SSM Session Manager
cover-img: /assets/img/ConnectionEC2SSM.png
thumbnail-img: /assets/img/ConnectionEC2SSM.png
share-img: /assets/img/ConnectionEC2SSM.png
tags: [ AWS , IAM , Security , Permissions  ]

---

# AWS CookBook 1.6 Connecting to EC2 Instances using AWS SSM Session Manager

#aws/cookbook/security

1. [AWS CookBook 1.6 Connecting to EC2 Instances using AWS SSM Session Manager](#aws-cookbook-16-connecting-to-ec2-instances-using-aws-ssm-session-manager)
   1. [Introduction](#introduction)
   2. [Problem](#problem)
   3. [Solutions](#solutions)

## Introduction

Connecting to EC2 Instances using AWS Session Manager eliminates the need to use Secure Shell(SSH) over internet for command-line access to your instances. Session Manager works by communicating with the AWS Systems Manager(SSM) API endpoint over HTTPS. 

Here are some of the security posture increase 

* No internet-facing TCP ports need to be allowed in security groups associated with instances.
* We can run instances in private subnets without directly exposimg them to the internet.
* There is no need to manage SSH keys.
* There is no need to manage user accounts and passwords on instances.
* You can delegate access to manage EC2 instances using IAM roles.

## Problem

We have an EC2 instance in a private subnet and need to connect to the instance without using SSH over internet.

## Solutions

1. Create an IAM role.
2. Attach the **AmazonSSMManagedInstanceCore** policy to the role.
3. Create an EC2 instance profile.
4. Attach the role created in step 1 to the instance profile.
5. Associate the instance profile to an EC2 instance.

The EC2 instance profile contains a role that you create. The instance profile association with an instance allows it to communicate with other AWS services using the IAM service.

![diagram](/assets/img/ConnectionEC2SSM.png)