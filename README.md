# AWS Real-Time Data Pipelin

## Overview
This project implements a real-time data pipeline using AWS services such as AWS VPC, AWS Managed Streaming for Apache Kafka (MSK), AWS Lambda, Amazon ECS, and Amazon S3. The pipeline collects, processes, and stores live API data while ensuring scalability, security, and cost optimization. AWS Glue Schema Registry is used for schema validation.

## Prerequisites
- Ensure that AWS roles and permissions are set up correctly when creating services.
- Required modules:
  - **kafka-python**: [Documentation](https://kafka-python.readthedocs.io/en/master/index.html)
  - **aws-glue-schema-registry**: [PyPI Link](https://pypi.org/project/aws-glue-schema-registry/)
- Add necessary Lambda layers for dependencies.

## Architecture Setup
### VPC & Networking
- **Custom VPC for enhanced security**
- **Subnets:**
  - 2 private subnets (for AWS MSK, Lambda, ECS) in different Availability Zones (ap-south-1a, ap-south-1b).
  - 2 public subnets (each in a separate Availability Zone).
- **Route Tables:**
  - One for public subnets and another for private subnets.
- **Internet Gateway** for external access.
- **NAT Gateway** (allocated an Elastic IP) for secure internet access from private subnets.

## Data Flow
### 1. Data Source
- Live data is retrieved from the **Mastodon API**, which continuously provides new post data.

### 2. API Gateway
- REST API is created using **AWS API Gateway**.
- The **POST method** routes data to an **SQS queue** to enable asynchronous processing.

### 3. SQS (Amazon Simple Queue Service)
- Acts as a buffer for incoming data to optimize Lambda invocations.
- Configured to trigger Lambda when either **50 messages are batched** or **60 seconds** have elapsed.

### 4. Lambda Producer
- Reads data from SQS and forwards it to **MSK (Kafka cluster)**.
- Uses **kafka-python** and **AWS Glue Schema Registry** for serialization (Avro format, compressed with Gzip).
- Logs are stored in **CloudWatch**.
- Execution role grants necessary permissions.

### 5. AWS MSK (Kafka Cluster)
- **Provisioned cluster** with two brokers (one in each Availability Zone).
- Default configurations were used for cost efficiency.
- Authentication:
  - **Plaintext (port 9092)** for this project.
  - **IAM with TLS encryption (port 9094)** can be used for enhanced security.
- Broker endpoints are used by both producer and consumer.

### 6. Consumer (ECS)
- **Python consumer script** using **kafka-python**.
- Consumer logic includes **group ID definition** and **deserialization** of Avro data.
- Packaged into a **Docker container**, uploaded to **AWS ECR**.
- ECS Cluster setup:
  - Task Definition references the container image and sets IAM permissions.
  - ECS Service automatically scales tasks based on Kafka partitions.
- **Consumed data is sent to Kinesis Firehose**.

### 7. Kinesis Firehose
- **Optimizes S3 writes** by batching data before storing.
- Transformation via Lambda can be enabled (skipped in this project for simplicity).
- Configured to use **AWS Glue Schema** for Parquet format storage in S3.
- Data compression options are available (e.g., `.json.gzip`).

## Summary
This project provides a cost-effective and scalable real-time data pipeline using AWS services. Many configurations can be modified based on specific use cases, such as increased security, transformations, or different storage formats. The implementation offers insights into AWS networking, serverless computing, streaming data processing, and cloud security.

Hope this helps!


