## Event Driven Sales Data Analysis

### Project Overview

This project demonstrates an event-driven architecture for processing sales data using AWS services. The architecture leverages Amazon S3, Amazon EventBridge, AWS Step Functions, AWS Lambda, Amazon DynamoDB, and Amazon SQS to create a robust pipeline for handling and validating sales orders data.

### Architecture Components

1. **Amazon S3**: Used for storing JSON files containing sales order data.
2. **Amazon EventBridge**: Detects the upload of new JSON files in the S3 bucket and triggers the Step Functions workflow.
3. **AWS Step Functions**: Orchestrates the workflow for processing each order in the JSON file.
4. **AWS Lambda**: Performs validation on the orders data.
5. **Amazon DynamoDB**: Stores validated order data.
6. **Amazon SQS**: Handles errors and stores failed validation messages in a dead-letter queue.

### Workflow Description

1. **JSON File Upload**: Sales order data is uploaded to an Amazon S3 bucket in JSON format.
2. **EventBridge Trigger**: The file upload triggers an EventBridge rule that initiates an AWS Step Functions state machine.
3. **Step Functions Execution**:
    - **GetObject**: Fetches the JSON file from S3 and parses its content.
    - **Map State**: Iterates over each order in the JSON file and processes them in parallel.
        - **Lambda Invocation**: Validates each order's data via a Lambda function.
        - **DynamoDB PutItem**: Stores validated orders into a DynamoDB table.
        - **Error Handling**: If validation fails, the order data is sent to an SQS dead-letter queue for further inspection.

### Detailed Step Functions Workflow

- **GetObject State**:
  - Retrieves the uploaded JSON file from S3.
  - Parses the content of the file into JSON format.
  
- **Map State**:
  - Iterates through each order in the JSON payload.
  - Processes orders concurrently with a maximum concurrency of 5.

- **validate_orders_data (Lambda Function)**:
  - Validates the contact and order information in the payload.
  - On success, returns the validated data.
  - On failure, triggers an error handling flow.

- **DynamoDB PutItem**:
  - Inserts validated orders into the `ecom-orders-kcd` DynamoDB table.
  - Retries the operation up to 3 times in case of failure with exponential backoff.

- **SQS SendMessage**:
  - If the validation fails, the order data is sent to an SQS dead-letter queue for later analysis.

### Error Handling

The state machine includes a catch block to handle task failures during validation or DynamoDB insertion. Failed tasks are redirected to the SQS dead-letter queue to ensure no data is lost and can be reprocessed or inspected manually.

### Lambda Function

The Lambda function `validate_orders_data` validates the order's contact information and order details. If the validation is successful, it returns the order data; otherwise, it raises an exception.

### Conclusion

This project showcases a scalable and reliable architecture for processing sales order data using AWS serverless services. The use of Step Functions for orchestration, Lambda for validation, DynamoDB for storage, and SQS for error handling provides a comprehensive solution for event-driven data processing.