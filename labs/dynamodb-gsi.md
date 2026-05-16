# Lab 08: DynamoDB — CRUD with Global Secondary Index (GSI)

## Objective
Create a DynamoDB table with a Global Secondary Index (GSI) for querying by a non-primary attribute, and run CRUD operations via AWS CLI.

## Prerequisites
- AWS CLI configured: `aws configure`

## Steps

### Part 1: Create Table with GSI
```bash
aws dynamodb create-table \
  --table-name Orders \
  --attribute-definitions \
    AttributeName=orderId,AttributeType=S \
    AttributeName=customerId,AttributeType=S \
    AttributeName=status,AttributeType=S \
  --key-schema \
    AttributeName=orderId,KeyType=HASH \
  --global-secondary-indexes \
    '[{
      "IndexName": "CustomerIndex",
      "KeySchema": [
        {"AttributeName":"customerId","KeyType":"HASH"},
        {"AttributeName":"status","KeyType":"RANGE"}
      ],
      "Projection": {"ProjectionType":"ALL"}
    }]' \
  --billing-mode PAY_PER_REQUEST
```

### Part 2: Insert Items (PUT)
```bash
# Insert orders
aws dynamodb put-item --table-name Orders --item \
  '{"orderId":{"S":"ORD-001"},"customerId":{"S":"CUST-A"},"status":{"S":"PENDING"},"amount":{"N":"250"}}'

aws dynamodb put-item --table-name Orders --item \
  '{"orderId":{"S":"ORD-002"},"customerId":{"S":"CUST-A"},"status":{"S":"SHIPPED"},"amount":{"N":"175"}}'

aws dynamodb put-item --table-name Orders --item \
  '{"orderId":{"S":"ORD-003"},"customerId":{"S":"CUST-B"},"status":{"S":"PENDING"},"amount":{"N":"400"}}'
```

### Part 3: Read (GET)
```bash
# Get by primary key
aws dynamodb get-item --table-name Orders \
  --key '{"orderId":{"S":"ORD-001"}}'
```

### Part 4: Query via GSI (all orders for customer A)
```bash
aws dynamodb query \
  --table-name Orders \
  --index-name CustomerIndex \
  --key-condition-expression "customerId = :cid" \
  --expression-attribute-values '{":cid":{"S":"CUST-A"}}'
# Returns both ORD-001 and ORD-002!
```

### Part 5: Update (UPDATE)
```bash
aws dynamodb update-item --table-name Orders \
  --key '{"orderId":{"S":"ORD-001"}}' \
  --update-expression "SET #s = :newstatus" \
  --expression-attribute-names '{"#s":"status"}' \
  --expression-attribute-values '{":newstatus":{"S":"DELIVERED"}}'
```

### Part 6: Delete (DELETE)
```bash
aws dynamodb delete-item --table-name Orders \
  --key '{"orderId":{"S":"ORD-003"}}'
```

## Cleanup
```bash
aws dynamodb delete-table --table-name Orders
```
