# Automate Image Analysis and Categorization with AWS Serverless Architecture

## Overview

This project implements an image processing service using AWS serverless architecture. The service automatically analyzes and categorizes images uploaded to an Amazon S3 bucket using AWS Lambda and Amazon Rekognition.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setup and Configuration](#setup-and-configuration)
   - [Step 1: Create an S3 Bucket](#step-1-create-an-s3-bucket)
   - [Step 2: Set Up IAM Role for Lambda](#step-2-set-up-iam-role-for-lambda)
   - [Step 3: Create a Lambda Function](#step-3-create-a-lambda-function)
   - [Step 4: Configure S3 Bucket Trigger](#step-4-configure-s3-bucket-trigger)
   - [Step 5: Implement Lambda Function Code](#step-5-implement-lambda-function-code)
   - [Step 6: Test the Setup](#step-6-test-the-setup)
3. [Monitoring and Logging](#monitoring-and-logging)
4. [Conclusion](#conclusion)

## Prerequisites

- **AWS Account**: Ensure you have an AWS account. You can create one [here](https://aws.amazon.com/).
- **AWS CLI**: Install and configure the [AWS CLI](https://aws.amazon.com/cli/) for command-line access.
- **AWS SDK**: Familiarity with AWS SDK for the programming language of your choice (e.g., Python, Node.js).

## Setup and Configuration

### Step 1: Create an S3 Bucket

1. Go to the [Amazon S3 Console](https://s3.console.aws.amazon.com/s3/).
2. Click on **Create bucket**.
3. Enter a unique bucket name (e.g., `my-image-upload-bucket`) and select a region.
4. Click **Create**.

### Step 2: Set Up IAM Role for Lambda

1. Go to the [IAM Console](https://console.aws.amazon.com/iam/).
2. Click on **Roles** in the left sidebar and then **Create role**.
3. Select **AWS service** and choose **Lambda**.
4. Click **Next: Permissions**.
5. Attach the following policies:
   - **AmazonS3ReadOnlyAccess** (to allow access to S3)
   - **AmazonRekognitionReadOnlyAccess** (to allow access to Rekognition)
   - **AWSLambdaBasicExecutionRole** (to allow logging to CloudWatch)
6. Click **Next: Tags**, and then **Next: Review**.
7. Name the role (e.g., `LambdaImageProcessingRole`) and click **Create role**.

### Step 3: Create a Lambda Function

1. Go to the [AWS Lambda Console](https://console.aws.amazon.com/lambda/).
2. Click on **Create function**.
3. Select **Author from scratch**.
4. Enter the function name (e.g., `ImageProcessingFunction`).
5. Choose the runtime (e.g., Python 3.x).
6. Under **Permissions**, choose the role you created earlier (e.g., `LambdaImageProcessingRole`).
7. Click **Create function**.

### Step 4: Configure S3 Bucket Trigger

1. In the Lambda function configuration page, scroll down to the **Function overview** section.
2. Click on **+ Add trigger**.
3. Select **S3**.
4. Choose the bucket you created earlier.
5. For **Event type**, select **PUT** (this triggers the function when an object is uploaded).
6. Click **Add**.

### Step 5: Implement Lambda Function Code

1. In the Lambda console, scroll down to the **Function code** section.
2. Replace the existing code with the following Python example:

```python
import json
import boto3
import os

def lambda_handler(event, context):
    rekognition_client = boto3.client('rekognition')
    s3 = boto3.client('s3')

    # Loop through each record in the event
    for record in event['Records']:
        bucket_name = record['s3']['bucket']['name']
        object_key = record['s3']['object']['key']

        # Call Rekognition to detect labels in the image
        response = rekognition_client.detect_labels(
            Image={'S3Object': {'Bucket': bucket_name, 'Name': object_key}},
            MaxLabels=10
        )

        # Log detected labels
        labels = response['Labels']
        detected_labels = [label['Name'] for label in labels]
        print(f"Image: {object_key}, Detected Labels: {detected_labels}")

        # Optionally: Store labels in a database, send notifications, etc.

    return {
        'statusCode': 200,
        'body': json.dumps('Image processed successfully!')
    }
```
3. **Click Deploy** to save your changes.

### Step 6: Test the Setup

1. **Upload an Image**: Go to the S3 Console and upload a sample image to your bucket.
2. **Check CloudWatch Logs**:
   - Go to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/home#/logs).
   - Navigate to **Logs** > **Log groups**.
   - Find the log group for your Lambda function (e.g., `/aws/lambda/ImageProcessingFunction`).
   - Click on the latest log stream to view the logs and confirm that labels/categories were detected for the uploaded image.

## Monitoring and Logging

- Ensure that logging is set up correctly to track the function's execution.
- Monitor invocation metrics and check for errors in the CloudWatch Logs.
- You can also configure alarms for error rates or throttling in CloudWatch.

## Conclusion

You have successfully set up a serverless image processing service using AWS Lambda, Amazon S3, and Amazon Rekognition. This architecture automatically analyzes and categorizes images uploaded to S3, enabling efficient image management and analysis.

