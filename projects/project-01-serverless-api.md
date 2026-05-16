# Project 1: Serverless REST API — API Gateway + Lambda + DynamoDB

## Architecture
```
Client (curl / Postman / React)
          │ HTTPS
          ▼
   AWS API Gateway
(REST API, routes, auth)
          │
          ▼
   AWS Lambda (Python)
(Business logic: validate, process)
          │
          ▼
  Amazon DynamoDB
(Serverless NoSQL database)
```

## What You'll Build
A fully serverless CRUD REST API for a **ToDo application** — no servers to manage, auto-scales to thousands of requests, pay-per-request billing.

## Services Used
- **API Gateway**: HTTP endpoints + routing
- **Lambda**: Backend logic (Python)
- **DynamoDB**: Database (serverless)
- **IAM**: Lambda execution role with DynamoDB permissions

---

## Step 1: Create DynamoDB Table
```bash
aws dynamodb create-table \
  --table-name Todos \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

## Step 2: Create Lambda Function (Python 3.12)
Create a Lambda with the following code. Attach a role with `AmazonDynamoDBFullAccess`.
```python
import json, boto3, uuid
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Todos')

def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    
    if method == 'POST' and path == '/todos':
        return create_todo(json.loads(event['body']))
    elif method == 'GET' and path == '/todos':
        return list_todos()
    elif method == 'GET' and '/todos/' in path:
        return get_todo(path.split('/')[-1])
    elif method == 'DELETE' and '/todos/' in path:
        return delete_todo(path.split('/')[-1])
    return {'statusCode': 404, 'body': json.dumps({'error': 'Not found'})}

def create_todo(body):
    item = {'id': str(uuid.uuid4()), 'title': body['title'], 
            'done': False, 'created': datetime.utcnow().isoformat()}
    table.put_item(Item=item)
    return {'statusCode': 201, 'body': json.dumps(item)}

def list_todos():
    result = table.scan()
    return {'statusCode': 200, 'body': json.dumps(result['Items'])}

def get_todo(todo_id):
    result = table.get_item(Key={'id': todo_id})
    item = result.get('Item')
    if not item:
        return {'statusCode': 404, 'body': json.dumps({'error': 'Not found'})}
    return {'statusCode': 200, 'body': json.dumps(item)}

def delete_todo(todo_id):
    table.delete_item(Key={'id': todo_id})
    return {'statusCode': 204, 'body': ''}
```

## Step 3: Create API Gateway REST API
1. API Gateway → Create API → REST API (public)
2. Name: `TodoAPI`
3. **Create resources and methods:**
   - Resource `/todos` → Method `GET` → Lambda integration → `TodoLambda`
   - Resource `/todos` → Method `POST` → Lambda integration → `TodoLambda`
   - Resource `/todos/{id}` → Method `GET` → Lambda integration
   - Resource `/todos/{id}` → Method `DELETE` → Lambda integration
4. Deploy API → Stage: `prod`
5. Copy the Invoke URL: `https://xxx.execute-api.us-east-1.amazonaws.com/prod`

## Step 4: Test with curl
```bash
BASE_URL="https://xxx.execute-api.us-east-1.amazonaws.com/prod"

# Create a todo
curl -X POST $BASE_URL/todos \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn AWS"}'

# List all todos
curl $BASE_URL/todos

# Get specific todo (use id from create response)
curl $BASE_URL/todos/<id>

# Delete a todo
curl -X DELETE $BASE_URL/todos/<id>
```

## Cleanup
Delete API Gateway → Delete Lambda → Delete DynamoDB table.
