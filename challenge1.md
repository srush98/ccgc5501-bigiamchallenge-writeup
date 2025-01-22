# Writeup for Big IAM Challenge: Challenge 1

# Challenge number

    1

## Challenge Statement

The goal of this challenge was to capture the flag by analyzing a publicly accessible bucket, understanding IAM (Identity and Access Management) policies, and using the AWS CLI to explore and interact with AWS resources.

## IAM Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-storage-9979f4b",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                }
            }
        }
    ]
}
```

### Short analysis about the IAM policy

```
* What do I have access to?
  - This policy allows public (`Principal: "*"`) read access (`s3:GetObject`) to objects in the bucket `thebigiamchallenge-storage-9979f4b`.
  - It also allows listing objects in the bucket (`s3:ListBucket`) if they are in the `files/` prefix.

* What don't I have access to?
  - You cannot list the entire bucket contents unless the objects are specifically under the `files/` prefix due to the condition in the `ListBucket` statement.

* What do I find interesting?
  - The policy provides broad public access, which might expose sensitive data if improperly configured.
  - The `Condition` restricting `ListBucket` access to the `files/` prefix adds a layer of control, but it still allows public enumeration of specific files.
```

## Solution

Step 1: Identified Accessible Files:

First, I listed the contents of the files/ prefix:

    aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/

    Access Bucket Contents: The directory contained two files:

            flag1.txt
            logo.png

Step 2: Downloaded Relevant Files:

Next, I identified files of interest (i.e., flag.txt) and downloaded them. To proceed with downloading the file containing the flag, I executed the following command:

    aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag1.txt /tmp/flag1.txt

Step 3: Viewing the Contents of the Flag File

After downloading flag1.txt, I viewed its contents using the following command:

    cat /tmp/flag1.txt

    This revealed the flag content.

    Flag Content: {wiz:exposed-storage-risky-as-usual}

Step 4: Inserting the flag

Finally, I inserted the flag found in the "Insert flag here" textbox and I was redirected to success screen.

## Reflection

- What was your approach?
  My approach involved carefully analyzing the IAM policy to understand access permissions, then using the AWS CLI to explore and download accessible files.

- What was the biggest challenge?
  Identifying the accessible files/ prefix and ensuring proper use of AWS CLI commands without credentials.

- How did you overcome the challenges?
  By testing AWS CLI commands iteratively and confirming access.

- What led to the break through?
  The realization that the s3:ListBucket permission is restricted to the files/ prefix guided me to focus on that specific directory.

- On the blue side, how can the learning be used to properly defend the important assets?

  - Avoid using overly permissive policies (Principal: "\*").

  - Apply stricter conditions and encryption to secure sensitive data.

  - Regularly audit and monitor bucket permissions and access logs to detect potential misuse or unauthorized access.
