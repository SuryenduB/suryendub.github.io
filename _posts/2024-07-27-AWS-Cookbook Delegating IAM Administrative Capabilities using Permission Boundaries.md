---
layout: post
title: 1.5 Delegating IAM Administrative Capabilities using Permission Boundaries
cover-img: assets/img/AWSRole.png)
thumbnail-img: assets/img/AWSRole.png)
share-img: assets/img/AWSRole.png)
tags: [ AWS , IAM , Security , Permissions  ]

---

# AWS CookBook 1.5 Delegating IAM Administrative Capabilities using Permission Boundaries

#aws/cookbook/security

1. [AWS CookBook 1.5 Delegating IAM Administrative Capabilities using Permission Boundaries](#aws-cookbook-15-delegating-iam-administrative-capabilities-using-permission-boundaries)
   1. [**Problem** ](#problem)
   1. [**Discussions**](#discussions)
   1. [**Solution**](#solution)
   1. [**Validation Checks**](#validation-checks)

## **Problem**

We need to seggregate permision boundaries between teams. For Example , let us assume we need to deploy **Lambda Functions** abd create IAM roles for them. We need to limit the effective permissions of the role created so that they allow only actions needed by the function.

## **Discussions**

Permission Boundaries act as a giardrail and limit provilege escalation. In order to limit the maximum permission of a delegated administrator.

1. Allow the creation of IAM customer managed policies.
2. Allow IAM role creation with a condition that a permission boundary must be attached.
3. Allow attachment of policy but only to roles that have a permission boundary.
4. Allow `iam:Passrole` to defined list of  AWS services that

## **Solution**

1. Create a file named

   `assume-role-policy-template.json` 

```json

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": { 
                "AWS": "PRINCIPAL_ARN"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

```

2. Replace the ARN for the user in the template file and use it to create a role.

```json

sed -e "s|PRINCIPAL_ARN|${PRINCIPAL_ARN}|g" \
assume-role-policy-template.json > assume-role-policy.json


ROLE_ARN=$(aws iam create-role --role-name AWSCookBookSuryendu105Role \
    --assume-role-policy-document file://assume-role-policy.json \
    --output text --query Role.Arn  )


```

3. Create a Permission boundary JSON file name boundary-template.json that allows specifice permissions for DynamoDB, S3, CloudWatch Logs action.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CreateLogGroup",
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:*:471112586770:*"
        },
        {
            "Sid": "CreateLogStreamandEvents",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:471112586770:*"
        },
        {
            "Sid": "DynamoDBPermissions",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem"
            ],
            "Resource": "arn:aws:dynamodb:*:471112586770:table/AWSCookbook*"
        },
        {
            "Sid": "S3Permissions",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::AWSCookbook*/*"
        },
        {
            "Sid": "CreateIAMRole",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:DeleteRole"
            ],
            "Resource": "*"
        }
    ]
}


```
4. Replace the ~*AWS_ACCOUNT_ID*~ and create a boundary policy.
```
sed -e "s|AWS_ACCOUNT_ID|${AWS_ACCOUNT_ID}|g" \
boundary-policy-template.json > boundary-policy.json

aws iam create-policy --policy-name AWSCookbookSuryendu105PB \
--policy-document file://boundary-policy.json 


```

We got an output as follows.

```json
{
    "Policy": {
        "PolicyName": "AWSCookbookSuryendu105PB",
        "PolicyId": "ANPAW3MD7SYJOPK2XCL5P",
        "Arn": "arn:aws:iam::471112586770:policy/AWSCookbookSuryendu105PB",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-07-25T11:42:42+00:00",
        "UpdateDate": "2024-07-25T11:42:42+00:00"
    }
```

5. Create a policy file template policy-template.json.
    **DenyPBDelete** : Deny the ability to delete permission boundaries.

```json
   {
            "Sid": "DenyPBDelete", 
            "Effect": "Deny",
            "Action": "iam:DeleteRolePermissionsBoundary",
            "Resource": "*"
        }

```
* **IAMRead** : Allow read-only IAM Access.
```
  {
            "Sid": "IAMRead",
            "Effect": "Allow",
            "Action": [
                "iam:Get*",
                "iam:List*"
            ],
            "Resource": "*"
        }

```

**IAMPolicies** : Allow the creation of IAM policies enforced naming convention.

```json
{
            "Sid": "IAMPolicies",
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:DeletePolicy",
                "iam:CreatePolicyVersion",
                "iam:DeletePolicyVersion",
                "iam:SetDefaultPolicyVersion"
            ],
            "Resource": "arn:aws:iam::AWS_ACCOUNT_ID:policy/AWSCookbook*"
        }

```

**IAMRolesWithBoundary** : Allow the creation and deletion of IAM Roles only with the permission boundary.

```json
{
            "Sid": "IAMRolesWithBoundary",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:PutRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::AWS_ACCOUNT_ID:role/AWSCookbook*"
            ],
             "Condition": {
                "StringEquals": {
                    "iam:PermissionsBoundary": "arn:aws:iam::471112586770:policy/AWSCookbookSuryendu105PB"
                }
            }
        }

```

**ServerlessFullAccess** : Allow full access to server less.

```json

            "Sid": "ServerlessFullAccess",
            "Effect": "Allow",
            "Action": [
                "lambda:*",
                "logs:*",
                "dynamodb:*",
                "s3:*"
            ],
            "Resource": "*"
        }

```

**PassRole** : Allow to pass IAM roles to Lambda Functions.

```json
{
            "Sid": "PassRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::AWS_ACCOUNT_ID:role/AWSCookbook*",
            "Condition": {
                "StringLikeIfExists": {
                    "iam:PassedToService": "lambda.amazonaws.com"
                }
            }
        }

```

**ProtectPB** : Explicitly deny the ability to modify the permissions boundary that bound the roles they create.

```json
{
            "Sid": "ProtectPB",
            "Effect": "Deny",
            "Action": [
                "iam:CreatePolicyVersion",
                "iam:DeletePolicy",
                "iam:DeletePolicyVersion",
                "iam:SetDefaultPolicyVersion"
            ],
            "Resource": [
                "arn:aws:iam::AWS_ACCOUNT_ID:policy/AWSCookbook105PB",
                "arn:aws:iam::AWS_ACCOUNT_ID:policy/AWSCookbook105Policy"
            ]
        }

```

6. Create the policy by replacing  `AWS_ACCOUNT_ID` and attach the policy to the role.

```json
aws iam create-policy --policy-name AWSCookbookSuryendu105PB \
--policy-document file://boundary-policy.json

aws iam create-policy --policy-name AWSCookbookSuryendu105Policy \
--policy-document file://policy.json 

```

We receive output as follows.

```json
{
    "Policy": {
        "PolicyName": "AWSCookbookSuryendu105Policy",
        "PolicyId": "ANPAW3MD7SYJCXSSI34WB",
        "Arn": "arn:aws:iam::471112586770:policy/AWSCookbookSuryendu105Policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2024-07-25T12:01:12+00:00",
        "UpdateDate": "2024-07-25T12:01:12+00:00"
    }
}

```

```shell
aws iam attach-role-policy --policy-arn arn:aws:iam::471112586770:policy/AWSCookbookSuryendu105Policy \
--role-name AWSCookBookSuryenduRole105


```

## **Validation Checks**

Assume the role we have created and set the output to local variables.

```bash
creds=$(aws --output text sts assume-role --role-arn $ROLE_ARN \
--role-session-name "AWSCookBookSuryendu105" | \
grep CREDENTIALS | CUT -d " " -f2,4,5)

export AWS_ACCESS_KEY_ID=$(echo "$creds" | cut -d$'\t' -f2)
export AWS_SECRET_ACCESS_KEY=$(echo "$creds" | cut -d$'\t' -f4)
export AWS_SESSION_TOKEN=$(echo "$creds" | cut -d$'\t' -f5)


echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
echo "AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY"
echo "AWS_SESSION_TOKEN: $AWS_SESSION_TOKEN"



```

Create the role, specifying the permissions boundary, which conforms to the role naming standard specified in the policy.

```bash
TEST_ROLE_1=$(aws iam create-role --role-name AWSCookbookSuryendu105test1 \
--assume-role-policy-document file://lambda-assume-role-policy.json \
--permissions-boundary arn:aws:iam::471112586770:policy/AWSCookbookSuryendu105PB \
--output text --query Role.Arn)

```

We can verify that the role is created in the console.

![](assets/img/AWSRole.png)