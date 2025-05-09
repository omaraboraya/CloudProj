
# Event-Driven Order Notification System (AWS)

This project implements a basic event-driven architecture on AWS to simulate the backend of an e-commerce platform. The system receives order events, stores them in a database, and sends out notifications, while also handling failures using a dead-letter queue.

---

## 🔧 Technologies Used

- **Amazon SNS** – Broadcasts order notifications
- **Amazon SQS** – Buffers messages for processing
- **AWS Lambda** – Processes messages and inserts into DynamoDB
- **Amazon DynamoDB** – Stores the order data
- **IAM** – Handles permission control

---

## 📐 Architecture Diagram

```
Client → SNS Topic → SQS Queue → Lambda Function → DynamoDB
                                     ↓
                                  DLQ (on failure)
```
SNS (OrderTopic)
     ↓
SQS (OrderQueue)
     ↓
Lambda (ProcessOrderLambda)
     ↓
DynamoDB (Orders Table)

---

## 📦 Setup Instructions

### Step 1 – DynamoDB

- Table name: `Orders`
- Partition key: `orderId` (String)
- Additional attributes: `userId`, `itemName`, `quantity`, `status`, `timestamp`
- Add data manually or via Lambda

---

### Step 2 – SNS Topic

- Name: `OrderTopic`
- Type: Standard
- Add subscription: SQS → `OrderQueue`

---

### Step 3 – SQS Queue

- Name: `OrderQueue` (Standard)
- Create DLQ first: `OrderDLQ` (Standard)
- Set DLQ in OrderQueue with maxReceiveCount = 3

---

### Step 4 – Lambda Function

- Name: `ProcessOrderLambda`
- Runtime: Python 3.12
- Trigger: SQS (`OrderQueue`)
- Permissions: Add `AmazonDynamoDBFullAccess` and `AmazonSQSFullAccess`

```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Orders')

def lambda_handler(event, context):
    for record in event['Records']:
        message = json.loads(record['body'])
        table.put_item(Item={
            'orderId': message['orderId'],
            'userId': message['userId'],
            'itemName': message['itemName'],
            'quantity': message['quantity'],
            'status': message['status'],
            'timestamp': message['timestamp']
        })
        print(f"✅ Order {message['orderId']} saved successfully.")
```

---

### Step 5 – Testing

1. Go to `OrderTopic` → Publish message
2. Use JSON:

```json
{
  "orderId": "O1234",
  "userId": "U123",
  "itemName": "Laptop",
  "quantity": 1,
  "status": "new",
  "timestamp": "2025-05-03T12:00:00Z"
}
```

3. Verify:
   - SNS receives
   - SQS delivers
   - Lambda logs output
   - DynamoDB stores item

---

## ✅ Deliverables

- Lambda Code File
- Screenshots: SNS, SQS, DynamoDB item
- This README

- 1-Page Write-up: How DLQ + Visibility Timeout improve reliability
