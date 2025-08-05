---
title: Gain expert-level knowledge about Powertools for AWS Lambda (OPN402)

date: 2025-08-04 00:00:00 +0800
categories: [AWS]
tags: [aws, re-event]
# TAG names should always be lowercase
---
## Powertools for AWS Lambda

AWS Lambda Powertools is an open-source library that helps you build serverless applications following best practices for observability, reliability, and maintainability. It provides utilities for logging, metrics, tracing, idempotency, input/output validation, and more—making it easier to write production-grade Lambda functions.

**Supported Languages:** Python (most features), Java, TypeScript, .NET

**Key Features Iintroduced:**
- Structured logging with correlation IDs
- Custom and wide logs for better traceability
- Metrics publishing to CloudWatch
- Distributed tracing with AWS X-Ray
- Input/output validation and OpenAPI documentation
- Idempotency for safe retries
- Feature flags, config management, secrets handling, caching, batch processing

## Structured logging

### Raw

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

logger.info("Hello world")
```
-> `[INFO] YYYY-MM-DDTHH:… [ID]…Hello world`

- Raw and hard to read

### Semi-structured

```python
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

logger.info("Hello world")
```

-> `[INFO] YYYY-MM-DDTHH:... ... {"message": "Hello world"}`

### Canonical

```python
import logging
import os
from logfmter import Logfmter

handler = logging.StreamHandler()
handler.setFormatter(Logfmter())
logging.basicConfig(handlers=[handler], level=os.getenv("LOG_LEVEL", "INFO"))

logger.info("Hello world", extra={"request_latency": 0.1})
```

-> `at=INFO; mesg=Hello world; requst_latency=0.1`

It has additional keys for customization. 

### Structured

```python
import logging
import os
from pythonjsonlogger import jsonlogger

logger = logging.getLogger()
structured_handler = logging.StreamHandler()
formatter = jsonlogger.JsonFormat(
    fmt='%(asctime)s %(levelname)s %(name)s %(message)'
)

structured_handler.setFormatter(formatter)
logger.addHandler(structured_handler)
logger.setLevel(loggingos.getenv("LOG_LEVEL", "INFO"))

logger.info("Hello world")
```

It will return a JSON-formatted log:

```json
{
  "asctime": "...",
  "levelname": "INFO",
  "name": "root",
  "message": "..."
}
```

Decorator of logger:

```python
from aws_lambda_powertools import logger
from aws_lambda_powertools.logging import correlation_paths

logger = Logger(service="payment")

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST)
def handler(event, context):
    logger.info("Hello")
```

It will also returns `"correlation_id"` in logger

### BYO formatter

A customized log

```python
from aws_lambda_powertools import logger
from aws_lambda_powertools.logging.formatter import LmabdaPowertoolsFormatter
from aws_lambda_powertools.logging.types import LogRecord

class CustomFormatter(LmabdaPowertoolsFormatter):
    def serialize(self, log: LogRecord) -> str:
          return self.json_serializer({
              "level": log["level"],
              "message": log["message"],
              ...
              "function_metadata": {
                    "name": log["lambda_function_name"], 
                    ...
              }
          })

logger = Logger(service="payment", logger_formatter=CustomFormatter)
logger.info("Hello")
```

### Wide logs

Make less log and less noisy. 

```python
from aws_lambda_powertools import logger

logger = Logger(service="payment")

@logger.inject_lambda_context()
def handler(event, context)
    customer = validate(event)
    logger.append_keys(custom_id=customer.id)
    
    try:
        subscription_id = update_subscription(customer_id=customer.id)
        logger.append_keys(customer_id=subscription_id)
    except Exception as exc:
          logger.append_keys(subscription_update_fault=exc)
      
      notification_id = send_email_notification(email=customer.email)
      logger.append_keys(notification_id=notification_id)
      logger.info("subscription proces")
```

This will add new keys into logs:

```json
{
  "level": "INFO",
  ...
  "custom_id": "jfdkal",
  "subscription_id": "fjkdafejwiqa",
  ...
}
```

## Event Handler

### A common pattern

routing, input/output validation, observability, serialization, idempotency, openAPI, Swagger UI, config management BYO middleware.

### API design & Trade-offs

How many functions needed for an API?

```json
Lambda monolith <-----> Micro-functions
```

### Lambda monolith / Micro-functions

| One route per Lambda function | per multiple Lambda functions |
| --- | --- |
| Simplicity | Complexity |
| Lower cold start chance (as all routes are going to same pool of execution env.) | Higher |
| Higher cold start time (large apps need more routes, needing more dependencies, starts independently) | Lower |
| Scaling & quotas | Independent scaling |
| Broad permissions | Granular permissions |
| Simpler CI/CD | Multiple deployments |

### From Event Handler to Routing

From event handler, we need to define if-else when event is in:

```python
def handler(event, context):
    try:
        http_method = event["httpMethod"]
        if http_method == "POST":
            ...
            return {
                "statusCode": 201,
                "body": json.dumps(...)
            }
        if http_method == "GET":
            ...
            return {
                "statusCode": 200,
                "body": json.dumps(...)
            }
        except:
            return {
                "statusCode": 500,
                "body": json.dumps(...)
            }
```

Routing, using decorator, can separate each `Method`:

```python
app = APIGatewayRestResolver()
def lambda_handler(event: dict, context: LambdaContext):
    return app.resolve(event, context)

@app.get("/todos/<todo_id>")
def get_todo_by_id(todo_id: int):
    ...
    return todo.json()
    
@app.route("/todo", methods=["POST", "PUT"])
def create_todo():
    ...
    return todo.json()
    
# Exception Handler
@app.exception_handler(ValueError)
def hanlder_invalid_limit_qs(ex: ValueError):
    meta_data = {"path": app.current_event.path, 
                 "query_strings": app.current_event.query_string_parameters}
    logger.error(...)
    return Response(
            status_code=400,
            content_type=...,
            body=...
    )

# Schema with Pydantic
app = APIGatewayRestResolver(enable_validation=True)
class Todo(BaseModel):
    userId: int
    ...

@app.get(...)
def get_todo_by_id(todo_id: int) -> Todo:
    ...
    
# OpenAPI spec
@app.get(
    "/todos/<todo_id>",
    summary="...",
    description="...",
    responses={
        200: ...
        400: ...
    },
    tags=["..."],
)
```

## Idempotency

Running multiple times with same results

Why we need idempotency: At-leat-once delivery, Lambda retries, transient failures, SDK retries, eventual consistency, auto-scaling operations

```python
from aws_lambda_powertools.utilities.idempotency import (
      DynamoDBPersistenceLayer,
      idempotent
)

@idempotent(persistent_store=ddbPersistenStore)
def lambda_handler(event: dict, context: LambdaContext):
    try:
        ...
        return {
                "statusCode": 200,
                ...
        }
     except Exception as e:
         raise ValueError(...)

# Context awareness
config = IdempotencyConfig(event_key_jmespath="order_id")
@idempotent_function(data_keyword_argument="order", config=config, persistentce_store=dynamodb)
def process_order(order, Order):
    ...
    
def lambda_handler(event, context):
    config.register_lambda_context(context)
    ...
```

---

Ref:

https://www.youtube.com/watch?v=kxJTq8FTkDA

https://docs.powertools.aws.dev/lambda/python/latest/utilities/jmespath_functions/#extracting-data