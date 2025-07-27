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

ask 1. Setup: Creating IAM policies and roles
When you first create an account on AWS, you become a root user, or an account owner. We don’t recommend that you use the account root user for daily operations and tasks. Instead, you should use an IAM user or IAM roles to access specific services and features. IAM policies, users, and roles are offered at no additional charge.

In this task, you create custom IAM policies and roles to grant limited permissions to specific AWS services.

Step 1.1: Creating custom IAM policies
Sign in to the AWS Management Console.

In the search box, enter IAM.

From the results list, choose IAM.

In the navigation pane, choose Policies.

Choose Create policy.

The Create policy page appears. You can create and edit a policy in the visual editor or use JSON. In this exercise, we provide JSON scripts to create policies. In total, you must create four policies.

In the JSON tab, paste the following code:

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

This JSON script grants permissions to put items into the DynamoDB table. The asterisk (*) indicates that the specified actions can apply to all available resources.

Choose Next: Tags and then choose Next: Review.

For the policy name, enter Lambda-Write-DynamoDB.

Choose Create policy.

After you create the Lambda-Write-DynamoDB policy, repeat the previous steps to create the following policies:

A policy for Amazon SNS to get, list, and publish topics that are received by Lambda:

Name: Lambda-SNS-Publish
JSON:
