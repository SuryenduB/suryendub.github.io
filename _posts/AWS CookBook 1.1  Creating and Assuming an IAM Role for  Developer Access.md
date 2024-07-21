# AWS CookBook 1.1 : Creating and Assuming an IAM Role for  Developer Access
 #aws/cookbook/security 
## Problem
As an IAM Engineer, it is always our responsibility to ensure high privileged permissions are not always in use.

## Solution
1. Create a role using an IAM Policy that will allow the role to be assumed later.

   ### Steps
   1. Create a file named assume-policy-template.json
   ```{
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
   2. Retrieve the principal ARN
   ```PRINCIPAL_ARN=$(aws sts get-caller-identity --query Arn --output text)
   ```
   3. Replace the **Principal_ARN** in the `assume-role-policy-template.json` and generate the `assume-role-policy.json` .
   4. Create a role policy and specify the assume role policy.
   ```
   ROLE_ARN=$(aws iam create-role --role-name AWSCookBookSuryenduRole101 \
    --assume-role-policy-document file://assume-role-policy.json \ 
    --output text --query Role.Arn )
   ```
   5. Attach the PowerUserAccess policy to the role.
   > PowerUserAccess policy  is designed to grant full access to AWS services and resources for users without giving them permissions to manage IAM users, groups, and roles.
   ```
   aws iam attach-role-policy --role-name AWSCookBookSuryenduRole101 \ 
   --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
   ```
   Power User Access Policy JSON
   ```
   {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "NotAction": [
                "iam:*",
                "organizations:*",
                "account:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceLinkedRole",
                "iam:DeleteServiceLinkedRole",
                "iam:ListRoles",
                "organizations:DescribeOrganization",
                "account:ListRegions",
                "account:GetAccountInformation"
            ],
            "Resource": "*"
        }
    ]
   }
   ```
   6. Assume the role created in previous steps.
   
   ```aws sts assume-role --role-arn $ROLE_ARN \                            --role-session-name AWSSuryendu101
   ```
   7. We will get output similar to this.
   ![](AWS%20CookBook%201.1%20%20Creating%20and%20Assuming%20an%20IAM%20Role%20for%20%20Developer%20Access/image.png)