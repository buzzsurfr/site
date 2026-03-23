---
title: "Rotate IAM Access Keys"
date: 2018-08-16T12:00:00-05:00
draft: false
categories: ["Deep Dive"]
tags: ["aws", "github", "iam"]
description: "How often do you change your password? Within AWS is a service called Trusted Advisor. Trusted Advisor runs checks in an AWS account looking for best practic..."
image: ""
---

How often do you change your password?

Within AWS is a service called Trusted Advisor. Trusted Advisor runs checks in an AWS account looking for best practices around Cost Optimization, Fault Tolerance, Performance, and Security.

In the Security section, there's a check (Business and Enterprise Support only) for the age of an Access Key attached to an IAM user. The Trusted Advisor check that will warn for any key older than 90 days and alert for any key older than 2 years. AWS recommends rotating the access keys for each IAM user in the account.

From [Trusted Advisor Best Practices (Checks)](https://aws.amazon.com/premiumsupport/trustedadvisor/best-practices/):

> 

Checks for active IAM access keys that have not been rotated in the last 90 days. When you rotate your access keys regularly, you reduce the chance that a compromised key could be used without your knowledge to access resources. For the purposes of this check, the last rotation date and time is when the access key was created or most recently activated. The access key number and date come from the access_key_1_last_rotated and access_key_2_last_rotated information in the most recent IAM credential report.

The reason for these times is the mean time to crack an access key. Using today's standard processing unit, and AWS Access Key could take xxx to crack, and users should rotate their Access Key before that time.

Yet in my experience, this often goes unchecked. I've come across an Access Key that was 4.5 years old! I asked why not change it, and the answer is mostly the same--the AWS Administrators and Security teams *do not* own and manage the credential, and the user doesn't want to change the credential for fear it will break their process.

Rotating an AWS Access Key is not difficult. It's a few simple commands to the AWS CLI (which you presumably have installed if you have an Access Key).

1. Create a new access key (CreateAccessKey API)
2. Configure AWS CLI to use the new access key (`aws configure`)
3. Disable the old access key (UpdateAccessKey API)
4. Delete the old access key (DeleteAccessKey API)

Instead of requiring each user to remember the correct API calls and parameters to each, I've created a script in [buzzsurfr/aws-utils](https://github.com/buzzsurfr/aws-utils) called `rotate_access_key.py` that orchestrates the process. Written in Python (a dependency of AWS CLI, so again should be present), the script minimizes the number of parameters and removes the undifferentiated heavy lifting associated with selecting the correct key. The user's access is confirmed to be stable by using the new access key to remove the old access key. The script can be scheduled using crown or Scheduled Tasks and supports CLI profiles.

```
usage: rotate_access_key.py [-h] --user-name USER_NAME
                            [--access-key-id ACCESS_KEY_ID]
                            [--profile PROFILE] [--delete] [--no-delete]
                            [--verbose]

optional arguments:
  -h, --help            show this help message and exit
  --user-name USER_NAME
                        UserName of the AWS user
  --access-key-id ACCESS_KEY_ID
                        Specific Access Key to replace
  --profile PROFILE     Local profile
  --delete              Delete old access key after inactivating (Default)
  --no-delete           Do not delete old access key after inactivating
  --verbose             Verbose
```

In order to use the script, the user must have the right set of permissions for their IAM user. This template is an example and only grants the IAM user permissions to change their *own* access Key.

From [IAM: Allows IAM Users to Rotate Their Own Credentials Programmatically and in the Console](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_iam_credentials_console.html):

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:ListUsers",
                "iam:GetAccountPasswordPolicy"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:*AccessKey*",
                "iam:ChangePassword",
                "iam:GetUser",
                "iam:*ServiceSpecificCredential*",
                "iam:*SigningCertificate*"
            ],
            "Resource": ["arn:aws:iam::*:user/${aws:username}"]
        }
    ]
}
```

This script is designed for users to rotate their credentials. This does not apply for "service accounts" (where the credential is configured on a server or unattended machine). If the machine is an EC2 Instance or ECS Task, then attaching an IAM Role to the instance or task will automatically handle rotating the credential. If the machine is on-premise or hosted elsewhere, then adapt the script to work unattended (I've thought about coding it as well).

As an AWS Administrator, you cant simply pass out the script and expect all users to rotate their access keys on time. Remember to build the system around it. Periodically query the TA check looking for access keys older than 90 days (warned), and send that user a reminder to rotate their access key. Take it a step further by automatically disabling access keys older than 120 days (warn them in the reminder). Help create good security posture and a good experience for your users, and make your account more secure at the same time!