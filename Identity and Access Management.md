IAM is a global service and does not require region selection. User Accounts,  groups and roles do not incur charges. Users and roles are also called identities or principals. Groups help manage a set of users. Groups within groups are not allowed. Every IAM user can generate access keys which allow programmatic access through AWS CLI. A user can be a part of multiple groups. 

Roles can be used by services to access resources (rather than using hard-coded access keys). Inter-account resource access. Externally authenticated users (identity federation). Services can assume roles to perform actions on your behalf. Only one role can be assigned to a user/instance at a time.
IAM also allows integration with other IDPs.

**For root accounts, it is recommended to delete access keys and set MFA. IAM Users with administrator access can not enable MFA delete on S3 bucket or set account level budgets. These things can only be done with a root account.**
 
Policies can be assigned to: 
* Users,groups,roles (Identity based policies)
* S3 bucket, SQS queue, a key in KMS (resource based policies)
 
 [Identity-Based Policies and Resource-Based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html)

Policy evaluation logic:
**Explicit Deny > Explicit Allow > Default Deny**

IAM policies can be assigned to any number of identities. Maximum 10 policies can be assigned to a given identity. The following types of policies are possible: 
* **AWS managed** : Created and administered by AWS (Standalone policies i.e. have their own ARN)
* **Customer managed**: Standalone policies created by customers. 
* **Inline**: Inline policy is a policy that's embedded in an IAM identity (a user, group, or role).

Inline policies maintain strict one to one relationship. Managed policies allow reusability, centralized changed management and delegation of management. IAM is eventually consistent because caching is used.

**IAM policy example:**

Allows managing Amazon EC2 security groups associated with a specific virtual private cloud (VPC). This policy also grants the necessary permissions to complete this action on the console. Note that conditions are optional. 
Acronym for elements: EAR.

        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:AuthorizeSecurityGroupEgress",
                        "ec2:AuthorizeSecurityGroupIngress",
                        "ec2:DeleteSecurityGroup",
                        "ec2:RevokeSecurityGroupEgress",
                        "ec2:RevokeSecurityGroupIngress"
                    ],
                    "Resource": "arn:aws:ec2:*:*:security-group/*",
                    "Condition": {
                        "ArnEquals": {
                            "ec2:Vpc": "arn:aws:ec2:*:*:vpc/vpc-vpc-id"
                        }
                    }
                },
                {

        "Effect": "Allow",
        "Action": [
                        "ec2:DescribeSecurityGroups",
                        "ec2:DescribeSecurityGroupReferences",
                        "ec2:DescribeStaleSecurityGroups",
                        "ec2:DescribeVpcs"
                    ],
                    "Resource": "*"
                }
            ]
        }

**ARN syntax:** 

**arn:aws:[service]:[region]:[account]:resourceType/path**

Bucket policy example: This makes the S3 bucket public with an exception. Apart from test.txt all objects have been made public. If we don't want the exception and want all objects to be public. We can say:

> "Resource": "arn:aws:s3:::shurmajee/*" 

        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "openS3Bucket",
                    "Effect": "Allow",
                    "Principal": "*",
                    "Action": "s3:GetObject",
                    "NotResource": "arn:aws:s3:::shurmajee/test.txt"
                }
            ]
        }

**sid(statementid)** is optional
The **Principal** element specifies the user, account, service, or other entity that is allowed or denied access to a resource. Cant only be used in IAM policy.
**Not** prefix creates an exception. I recommend going through policy examples in AWS docs to develop a basic understanding of the policy language.

**Key Rotation:** It is recommended to retire older access keys for users. For roles this process is automated. 
* AWS Secrets manager can be used to manage third party API keys
* Accesskeys allow programmatic access. A user can have max 2 at a time. They allow access through CLI and SDK. They are shown only once when generated.
* IAM stores up to five versions of your customer managed policies. You can use policy versions to revert a policy to an earlier version if you need to.

**Amazon Cognito**

Decentralized way to manage authentication. It provides authentication, authorization, and user management for your web and mobile apps. Your users can sign in directly with a user name and password, or through a third party such as Facebook, Amazon, Google or Apple.

**User Pools** provide account management services and sessions are persisted with JWT. Cognito sync can be used for data synchronization across devices. 
Sends notification to users using SNS.

**Identity pools** would give temporary access to AWS resources to your users without needing to give them IAM credentials. e.g. A photo librry app allowing users to store images to cloud (S3). 
