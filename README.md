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
* *Attach a policy to the role that delegates access to Amazon S3. For example, this policy grants access to all actions to a specific bucket and the objects stored in the bucket:* *
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
   __Enter the following policy. Replace arn:aws:iam::111111111111:role/ROLENAME with the Amazon Resource Name (ARN) of the IAM role that you created in Account B.__
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
 After you connect to the instance, verify if the directory already has a folder named .aws _ _path of the file \home\ec2-user\.aws\config_ _
 
 Edit with your favarite editor
 
 vi \home\ec2-user\.aws\config

 In the file, enter the following text. Replace enterprofilename with the name of the role that you
attached to the instance. Then, replace arn:aws:iam::111111111111:role/ROLENAME with the
ARN of the role that you created in Account B.
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
