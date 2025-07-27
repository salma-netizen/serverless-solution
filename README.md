# serverless-solution
# 🚀 Architecting Solutions: Building a Proof of Concept for a Serverless Solution

This project provides you with instructions for how to build a proof of concept for a serverless solution in the AWS Cloud.

Suppose you have a customer that needs a serverless web backend hosted on AWS. The customer sells cleaning supplies and often sees spikes in demand for their website, which means that they need an architecture that can easily scale in and out as demand changes. The customer also wants to ensure that the application has decoupled application components.


---
**Default Architecture**

![project structure](/project%20structure.png)

In this architecture, you will use a REST API to place a database entry in the Amazon SQS queue. Amazon SQS will then invoke the first Lambda function, which inserts the entry into a DynamoDB table. After that, DynamoDB Streams will capture a record of the new entry in a database and invoke a second Lambda function. The function will pass the database entry to Amazon SNS. After Amazon SNS processes the new record, it will send you a notification through a specified email address.

In this project, you will learn how to do the following:

-Create IAM policies and roles to follow best practices of working in the AWS Cloud.
-Create a DynamoDB table to store data.
-Create an Amazon SQS queue to receive, store, and send messages between software components.
-Create Lambda functions and set up triggers to invoke actions in different AWS services.
-Enable DynamoDB Streams to capture modifications in the database table.
-Configure Amazon SNS to receive email or text notifications.
-Create a REST API to insert data into a database.


Notes:

To complete the instructions in this exercise, choose the US East (N. Virginia) us-east-1 Region in the navigation pane of the AWS Management Console.

The instructions might prompt you to enter your account ID. Your account ID is a 12-digit account number that appears under your account alias in the top-right corner of the AWS Management Console. When you enter your account number (ID), make sure that you remove hyphens (-).

---

Task 1. Setup: Creating IAM policies and roles
When you first create an account on AWS, you become a root user, or an account owner. We don’t recommend that you use the account root user for daily operations and tasks. Instead, you should use an IAM user or IAM roles to access specific services and features. IAM policies, users, and roles are offered at no additional charge.

In this task, you create custom IAM policies and roles to grant limited permissions to specific AWS services.

Step 1.1: Creating custom IAM policies
1-Sign in to the AWS Management Console.

2-In the search box, enter IAM.

3-From the results list, choose IAM.

4-In the navigation pane, choose Policies.

5-Choose Create policy.

The Create policy page appears. You can create and edit a policy in the visual editor or use JSON. In this exercise, we provide JSON scripts to create policies. In total, you must create four policies.

6-In the JSON tab, paste the following code:

```bash

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:DescribeTable"
            ],
            "Resource": "*"
        }
    ]
}

```


-Choose Next: Tags and then choose Next: Review.

For the policy name, enter Lambda-Write-DynamoDB.

-Choose Create policy.

-After you create the Lambda-Write-DynamoDB policy, repeat the previous steps to create the following policies:

-A policy for Amazon SNS to get, list, and publish topics that are received by Lambda:

-Name: Lambda-SNS-Publish
-JSON:

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sns:Publish",
                "sns:GetTopicAttributes",
                    "sns:ListTopics"
            ],
                "Resource": "*"
        }
    ]
 }


```

 ***A policy for Lambda to get records from DynamoDB Streams:

Name: Lambda-DynamoDBStreams-Read
JSON:

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "sqs:DeleteMessage",
                "sqs:ReceiveMessage",
                "sqs:GetQueueAttributes",
                "sqs:ChangeMessageVisibility"
            ],
            "Resource": "*"
        }
    ]
}

```



Step 1.2: Creating IAM roles and attaching policies to the roles
Because AWS follows the principle of least privilege, we recommend that you provide role-based access to only the AWS resources that are required to perform a task. In this step, you create IAM roles and attach policies to the roles.

1-In the navigation pane of the IAM dashboard, choose Roles.

2-Choose Create role and in the Select trusted entity page, configure the following settings:
       .Trusted entity type: AWS service
       .Common use cases: Lambda
3-Choose Next.

4-On the Add permissions page, select Lambda-Write-DynamoDB and Lambda-Read-SQS.

5-Choose Next

6-For Role name, enter Lambda-SQS-DynamoDB.

7-Choose Create role.

8-Follow the previous steps to create two more IAM roles:

     1- An IAM role for AWS Lambda: This role grants permissions to obtain records from the DynamoDB streams and send the records to Amazon SNS. Use the following information to create the role.
                -IAM role name: Lambda-DynamoDBStreams-SNS
                -Trusted entity type: AWS service
                -Common use cases: Lambda
                -Attach policies: Lambda-SNS-Publish and Lambda-DynamoDBStreams-Read
      2-An IAM role for Amazon API Gateway: This role grants permissions to send data to the SQS queue and push logs to Amazon CloudWatch for troubleshooting. Use the following information to create the role.
                -IAM role name: APIGateway-SQS
                -Trusted entity type: AWS service
                -Common use cases: API Gateway
                -Attach policies: AmazonAPIGatewayPushToCloudWatchLogs

---

###Task 2: Creating a DynamoDB table
In this task, you create a DynamoDB table that ingests data that’s passed on through API Gateway.

     1-In the search box of the AWS Management Console, enter DynamoDB.

     2-From the list, choose the DynamoDB service.

     3-On the Get started card, choose Create table and configure the following settings:
            -Table: orders
            -Partition key: orderID
            -Data type: Keep String
Keep the remaining settings at their default values, and choose Create table.

---

###Task 3: Creating an SQS queue
 In this task, you create an SQS queue. In the architecture for this exercise, the Amazon SQS receives data records from API Gateway, stores them, and then sends them to a database.

  1-In the AWS Management Console search box, enter SQS and from the list, choose Simple Queue Service.

  2-On the Get started card, choose Create queue.

  The Create queue page appears.

  3-Configure the following settings:
      - Name: POC-Queue
      - Access Policy: Basic
      - Define who can send messages to the queue:
              -Select Only the specified AWS accounts, IAM users and roles
              -In the box for this option, paste the Amazon Resource Name (ARN) for the APIGateway-SQS IAM role
              -Note: For example, your IAM role might look similar to the following: arn:aws:iam::<account ID>:role/APIGateway-SQS.
      -Define who can receive messages from the queue:
              -Select Only the specified AWS accounts, IAM users and roles.
              -In the box for this option, paste the ARN for the Lambda-SQS-DynamoDB IAM role.
              -Note: For example, your IAM role might look similar to the following: arn:aws:iam::<account_ID>:role/Lambda-SQS-DynamoDB
   4-Choose Create queue

---

Task 4: Creating a Lambda function and setting up triggers
In this task, you create a Lambda function that reads messages from the SQS queue and writes an order record to the DynamoDB table

Step 4.1: Creating a Lambda function for the Lambda-SQS-DynamoDB role
1-In the AWS Management Console search box, enter Lambda and from the list, choose Lambda.
2-Choose Create function and configure the following settings:
     Function option: Author from scratch
     Function name: POC-Lambda-1
     Runtime: Python 3.9
     Change default execution role: Use an existing role
     Existing role: Lambda-SQS-DynamoDB
3-Choose Create function.

