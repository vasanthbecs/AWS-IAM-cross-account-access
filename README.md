# AWS-IAM-cross-account-access


## Grant Amazon EC2 instance access to an Amazon S3 bucket in another AWS account?
### Issue
**I want my Amazon Elastic Compute Cloud (Amazon EC2) instance to be able to access my Amazon Simple Storage Service (Amazon S3) bucket in another AWS account. How can I grant? this access without storing credentials on the instance?**
#### Solution
Follow these steps to grant an Amazon EC2 instance in one account (Account A) the permissions to access an Amazon S3 bucket in another account (Account B).
From Account B, create an IAM role
1.    Sign in to the AWS Management Console with Account B.
2.    Open the AWS Identity and Access Management (IAM) console.
3.    In the navigation pane, choose Roles.
4.    Choose Create role.
5.    For Select type of Trust Relationships, choose Another AWS account.
6.    For Account ID, enter the account ID of Account A.
7.    Choose Next: Permissions.
*Attach a policy to the role that delegates access to Amazon S3. For example, this policy grants access to all actions to a specific bucket and the objects stored in the bucket:*
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3::: BucketName /*",
                "arn:aws:s3::: BucketName "
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:HeadBucket"
            ],
            "Resource": [
                "arn:aws:s3::: BucketName /*",
                "arn:aws:s3::: BucketName "
            ]
        }
    ]
}

```

***From Account A, create another role (instance profile) and attach it to the instance***

1.    Sign in to the AWS Management Console with Account A.
2.    Open the IAM console.
3.    From the navigation pane, choose Roles.
4.    Choose Create role.
5.    For Select type of trusted entity, choose AWS service.
6.    For Choose the service that will use this role, choose EC2.
7.    Choose Next: Permissions.
8.    Choose Next: Tags.
9.    For Role name, enter a name for the role.
10.    Choose Create role.
11.    Choose Add inline policy, and then choose the JSON view.

**Enter the following policy mentioned below Replace _arn:aws:iam::111111111111:role/ROLENAME_ with the Amazon Resource Name (ARN) of the IAM role that you created in Account B**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": " arn:aws:iam::111111111111:role/ROLENAME "
        }
    ]
}
```

  Attach the IAM role (instance profile) to the Amazon EC2 instance that you&#39;ll use to access the
Amazon S3 bucket.
Install AWS CLI
From the Amazon EC2 instance, configure the role with your credentials
 After you connect to the instance, verify if the directory already has a folder named .aws **path of the file _\home\ec2-user\.aws\config_ where the .aws folder is located**
 
 Edit with your favarite editor

 >vi \home\ec2-user\.aws\config

 In the file, enter the following text. **Replace enterprofilename with the name of the role that you
attached to the instance. Then, replace _arn:aws:iam::111111111111:role/ROLENAME_ with the
ARN of the role that you created in Account B**.
```
[profile profilename]
role_arn = arn:aws:iam::111111111111:role/ROLENAME
credential_source = Ec2InstanceMetadata

````

Verify the instance profile
To verify that your instance&#39;s role (instance profile) can assume the role in Account B, run the
following command while connected to the instance. Replace profilename with the name of the
role that you attached to the instance.
```
aws sts get-caller-identity --profile profilename
```
To verify access to the Amazon S3 bucket, please follow the mentioned steps.
aws s3 ls BucketName






# Another Alternative method for cross account access
 
 ### Below example we have AWS account A where we have Route53 domain hosted and another AWS account B where EC2 instance launched we have to need to have account b instance to access Route53 hosted in account B.
 #### Here, in this example cross account access is granting using the temporary access key and screte key which will have some hours life time.
 
 How to do it, please see below.
 
 1. account A create a IAM role for cross account access
 

 
 
 ![edited1](https://user-images.githubusercontent.com/24250130/70145669-a40ef200-16c6-11ea-890d-9319b3f461ad.png)
 
2. Once IAM role created attached the below Inline policies to access the Route53.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "route53:GetHostedZone",
                "route53:ChangeResourceRecordSets",
                "route53:ListResourceRecordSets",
                "route53:GetHostedZoneLimit"
            ],
            "Resource": "arn:aws:route53:::hostedzone/xxxxxxx"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "route53:GetHostedZoneCount",
            "Resource": "*"
        }
    ]
}

 
 
```

3. Create below policy and attach to the same role which we have created erlier.

**STS assume policy**

```

{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "sts:AssumeRole",
        "Resource": "arn:aws:iam:: xxxxx:role/role name"  (ARM of role created in account A)
    }
}

```

4. Edit the below trust relationship policy for the same account A IAM role to get the cross account access to account B account ID

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam:: account B ID:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}

```

5. Login to account B and create IAM role for ec2 instance and add the below STS policy to assume the IAM role of account A

```

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole",
                "sts:GetFederationToken",
                "sts:AssumeRoleWithSAML",
                "sts:AssumeRoleWithWebIdentity"
            ],
            "Resource": "arn of account A role"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "sts:DecodeAuthorizationMessage",
                "sts:GetCallerIdentity"
            ],
            "Resource": "*"
        }
    ]
}

```
6. Attach the role to instance in account B.

7. login to the mentioned EC2 instances  and run the below command which will give you the temporary credentials to use the cross account role

**aws sts assume-role --role-arn "arn of account A role" --role-session-name "Name of your choice"**


This will give you the temporary credentials and use the same to create a CLI profile.
Example output will be like below.

 ![edited2](https://user-images.githubusercontent.com/24250130/70145671-a40ef200-16c6-11ea-8b9b-4c959f2aef7f.png)
 
 
 finaly test the connection with AWS CLI command
 
 
 
 
 
