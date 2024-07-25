---
layout: post
title: AWS CookBook 1.3 Enforcing IAM User Password Policies in Your AWS Account
cover-img: assets/img/AWSIAMPolicy.png
thumbnail-img: assets/img/AWSIAMPolicy.png
share-img: assets/img/AWSIAMPolicy.png
tags: [ AWS , IAM , Security , Password  ]

---

# AWS CookBook 1.3: Enforcing IAM User Password Policies in Your AWS Account
#aws/cookbook/security

### Problem

Your Security Policy requires that you must enforce password policy for all the users in your AWS Account.

| Requirements      | Values        |
|-------------------|---------------|
| Password Validity | 90 Days       |
| PasswordLength    | 32 Characters |
| Lower Case        | Atleast 1     |
| Upper Case        | Atleast 1     |
| Number            | Atleast 1     |
| Symbols           | Atleast 1     |

## Solution

- [X] Set a Password policy for IAM users.
- [X] Create user and add a login policy.

![AWSIAMPolicy](assets/img/AWSIAMPolicy.png)

## Steps

1. **Set an IAM password policy** : Create a password policy based on previous requirement. 
 
 
```
aws iam update-account-password-policy \
--minimum-password-length 32 \
--require-symbols \
--require-numbers \
--require-uppercase-characters \
--require-lowercase-characters \
--allow-users-to-change-password \
--max-password-age 90 \
--password-reuse-prevention 10 

```

2. **Create an IAM group** : `aws iam create-group --group-name AWSCookBookSuryenduGroup`
   **Output**: 
   
   We  get the following output.

```
   {
    "Group": {
        "Path": "/",
        "GroupName": "AWSCookBookSuryenduGroup",
        "GroupId": "AGPAW3MD7SYJPBXI3YVIT",
        "Arn": "arn:aws:iam::471112586770:group/AWSCookBookSuryenduGroup",
        "CreateDate": "2024-07-24T21:48:05+00:00"
    }
}
```

3. **Attach the ReadOnlyAccess policy to the group**

```
aws iam attach-group-policy \
--group-name AWSCookBookSuryenduGroup \
--policy-arn arn:aws:iam::aws:policy/AWSBillingReadOnlyAccess

```

4. **Create an IAM User** : `aws iam create-user --user-name AWSCookBookSuryenduUser`
   **Output**: And the output we get is as follows

```
{
    "User": {
        "Path": "/",
        "UserName": "AWSCookBookSuryenduUser",
        "UserId": "AIDAW3MD7SYJO3XT7DC5P",
        "Arn": "arn:aws:iam::471112586770:user/AWSCookBookSuryenduUser",
        "CreateDate": "2024-07-24T21:57:08+00:00"
    }
}
```

5. **Add the user to the group with attached policy**

```
   aws iam add-user-to-group \
--group-name AWSCookBookSuryenduGroup \
--user-name AWSCookBookSuryenduUser

```

6. **Use Secrets Manager**  to generate a password that confirms to your password policy and Create a login profile for the user with the generated password.

```
RANDOM_STRING=$(aws secretsmanager get-random-password \
--password-length 32 \
--require-each-included-type \
--output text \
--query RandomPassword )


aws iam create-login-profile --user-name AWSCookBookSuryenduUser \
--password $RANDOM_STRING \


```
7. **Validation Checks** Verify that password policy is now set. `aws iam get-account-password-policy`. We will get an output similar to this.

```
	{
    "PasswordPolicy": {
        "MinimumPasswordLength": 32,
        "RequireSymbols": true,
        "RequireNumbers": true,
        "RequireUppercaseCharacters": true,
        "RequireLowercaseCharacters": true,
        "AllowUsersToChangePassword": true,
        "ExpirePasswords": true,
        "MaxPasswordAge": 90,
        "PasswordReusePrevention": 10
    }
}

```
8. **Negative Validation Checks** : Create an user  and add a login policy that does not adhere to the password policy.

```
aws iam create-user --user-name AWSCookBookSuryenduUser2

RANDOM_STRING=$(aws secretsmanager get-random-password \
--password-length 16 \
--require-each-included-type \
--output text \
--query RandomPassword \
--profile root)

aws iam create-login-profile --user-name AWSCookBookSuryenduUser2 \
--password $RANDOM_STRING \
--profile root


```

You will receive an error similar to this.

![image](assets/img/Screenshot 2024-07-25 at 00.26.28.png)
