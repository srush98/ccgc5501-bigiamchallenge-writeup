# Writeup for Big IAM Challenge: Challenge 4

# Challenge number

    4

## Challenge Statement

The goal of this challenge was to exploit an overly permissive IAM policy for an Amazon S3 bucket. My objective was to bypass the restrictions on listing and accessing objects in the bucket, retrieve the flag, and demonstrate the risks associated with misconfigured IAM policies.

## IAM Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                },
                "ForAllValues:StringLike": {
                    "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
                }
            }
        }
    ]
}
```

### Short analysis about the IAM policy

```
* What do I have access to?
  - I have access to the s3:GetObject action for all objects within the S3 bucket (arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*). This means I can fetch any object as long as I know its name.

* What don't I have access to?
  - I don’t have direct access to the s3:ListBucket action unless my PrincipalArn matches arn:aws:iam::133713371337:user/admin. This restricts my ability to list the files in the bucket under normal circumstances.

* What do I find interesting?
  - The s3:GetObject permission is publicly available, which makes it exploitable despite the restriction on s3:ListBucket.
  - The IAM policy condition for s3:ListBucket applies only to authenticated users but does not restrict anonymous access to s3:GetObject.
  - The reliance on PrincipalArn for restricting s3:ListBucket highlights a common misconfiguration where individual permissions do not align with broader access controls.
  - The lack of anonymous access restrictions makes the bucket vulnerable, especially when sensitive files are stored without additional layers of protection.
```

## Solution

Step 1: Reviewing the IAM Policy
When analyzing the S3 bucket policy, I found two key points: - Public access was allowed for s3:GetObject, which meant anyone could fetch objects. - The s3:ListBucket action was restricted using a PrincipalArn condition, requiring the ARN to match arn:aws:iam::133713371337:user/admin.
Here's the policy snippet I analyzed:

```
{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:ListBucket",
    "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
    "Condition": {
        "ForAllValues:StringLike": {
            "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
        }
    }
}
```

Step 2: Attempting to List Files (Access Denied)
Initially, I tried listing the files in the bucket using the following command:

    aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/

    An error occurred (AccessDenied) when calling the ListObjectsV2 operation: Access Denied

    This resulted in an Access Denied error because my Principal ARN did not match the admin user required by the policy.

Step 3: Attempting to Download a File

I realized the policy restricted the s3:ListBucket action but didn’t enforce signed requests for anonymous access. By using the --no-sign-request option, I bypassed the condition:

    aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/ --no-sign-request

    This successfully listed the files:

        flag-as-admin.txt
        logo-admin.png

Steo 4: Fetching the Flag
Next, I directly fetched the flag using the same anonymous access method:

aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt - --no-sign-request

    Flag: {wiz:principal-arn-is-not-what-you-think}

Step 5: Submitting the Flag

Finally, I inserted the flag found in the "Insert flag here" textbox and I was redirected to success screen.

## Reflection

- What was your approach?
  I analyzed the IAM policy to identify misconfigurations and tested various access methods. By using anonymous access with the --no-sign-request option, I bypassed the s3:ListBucket restriction and accessed the objects directly.

- What was the biggest challenge?
  The biggest challenge was recognizing that the s3:GetObject permission for public access could be exploited without needing the s3:ListBucket action.

- How did you overcome the challenges?
  I carefully studied the IAM policy and experimented with different access methods, eventually realizing that anonymous access would work for s3:GetObject.

- What led to the breakthrough?
  Understanding that the s3:GetObject action was not conditioned by PrincipalArn or ListBucket restrictions allowed me to bypass the restrictions and directly access the files.

- On the blue side, how can the learning be used to properly defend important assets?
  - Restrict Public Access: Avoid using Principal: "\*" in IAM policies unless absolutely necessary.
  - Align Permissions: Ensure that all actions (e.g., s3:GetObject and s3:ListBucket) are conditioned appropriately to avoid bypass scenarios.
  - Enable Bucket Policies: Use S3 bucket policies to explicitly deny anonymous access or unauthorized actions.
  - Implement Least Privilege: Grant only the minimum permissions required for specific roles or users.
  - Enable Logging and Monitoring: Use AWS CloudTrail to monitor access and detect any unauthorized or suspicious activity.
  - Encrypt Sensitive Files: Protect sensitive data using S3 bucket encryption, ensuring it remains unreadable even if accessed improperly.
