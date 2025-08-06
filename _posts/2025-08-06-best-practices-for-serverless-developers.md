---
title: 2024 - Best practices for serverless developers (SVS401)

date: 2025-08-06 00:00:00 +0800
categories: [AWS]
tags: [aws, re-event]
# TAG names should always be lowercase
---
## Service-full serverless (Compose, configure, then code)

| Orchestration                               | Choreography           |
| ------------------------------------------- | ---------------------- |
| A central coordination controlling services | Services collaboration |
| Workflows (AWS Step functions)              | Events (EventBridge)   |

### Step functions: standard vs. Express workflows

| Standard                 | Express                            |
| ------------------------ | ---------------------------------- |
| Max 1 year               | Max 5 min                          |
| Async                    | Sync & Async                       |
| Exactly once             | At least once                      |
|                          | High throughput                    |
|                          | Cost effective                     |
| Pay per state transition | Pay per number of executions + GBs |
| More expensive           | Cheaper                            |

Also, a nested workflow (using both) can take both advantages.

## Serverless compute

### AWS Lambda invocation model

3 types of Lambda integrations:

- Synchronous: including request streaming<br>
  e.g. `client <-> API gateway <-> Lambda`
- Asynchronous<br>
  e.g. `EventBridge <-> Lambda service queue <-> Lambda`
- Poll-based: event source mapping<br>
  e.g. `kinesis <-polls-> Lambda <-> Lambda`

### Lambda function memory power

- Lambda memory can be between 128 MB to 10 GB.
- Memory allocates CPU power & network bandwidth.
- If code is bounded by memory, CPU, or network, adding memory may improve performance and reduce cost.

### Lambda life cycle

| INIT       | INVOKE     | INVOKE | INVOKE | SHUTDOWN |
| ---------- | ---------- | ------ | ------ | -------- |
| Cold Start | Warm Start |        |        |          |

- `INIT`: (from 2025, it costs money)
    1. Extension init
        - create new execution environment
        - download code / image
    2. runtime init
    3. function init (start to be optimized by ourself)
- `INVOKE`
- `SHUTDOWN`:
    1. Runtime shutdown
    2. extension shutdown

### Function init optimization

- Don’t load it until you need
    - Optimize dependencies, SDKs and other libraries
    - Minify / Uglify production code
    - Reduce deployment package size
    - Avoid monolithic functions
- Lazy initialize shared libraries
- Establishing connections
    - Handle reconnections in handler
    - Keep-alive in SDKs
- State during environment reuse
    - Keep data for subsequent invocations
- Use provisioned concurrency or SnapStart

### Lambda Snapshot

- 在Cold start 與Warm start 中間，指定某版本（無法指定隨時為最新的）製作成snapshot
- 10x faster startup performance
- no additional cost

### Concurrency

多一個concurrency 需要再啟動一次cold start，也可設定concurrency requests limit（`throttled`）

### Provisioned concurrency

- 預訂數量個cold start，隨時可以進入warm start。
- Can be tracked through `ProvisionedConcurrencyUtilzation`
- 指定某版本（無法指定隨時為最新的）製作成snapshot

### Scaling quotas

Multiple quotas can be set:

- Account concurrency quota (一個帳號所有function的quota）
- Scale concurrency quota（一個function的quota）

### Synchronous invocations: Best practices

- Load test
    - Raise concurrency quota before burst
- Set appropriate timeouts
    - API gateway default is 29 seconds
- Retry with backoff
    - Client retry, use exponential backoff + jitter to not overload system
- Implement idempotency
    - Expect and account for retrying operations

### Poll-based (stream) invocations: Best practices

- Processing
    - Use filtering to avoid processing message
    - Evenly distribute records using partition key
    - Poison message can stop processing, gracefully handle errors
- Performance
    - Increase memory/optimize code/increase batch size
    - Add partitions/shards
    - Kinesis: increase parallelization factor/enhanced fan-out
- Monitoring
    - Enhanced monitoring: `IteratorAge`, `consumer_lag`

### AWS Fargate

A AWS managed ECS. Pay a constant rate per second for the task CPU and memory (not concurrency requests) → Lower requests per number is relatively cost.

### Consideration between Functions & Containers

| Condition     | Functions (Lambda)                                                         | Containers (ECS / Fargates)                                                       |
| ------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| When you want | Fewer decision and fast.<br>Faster prototype to production                 | Control of compute environment<br>Tooling consistency                             |
| Architecture  | Managed triggers, runtimes, scaling                                        | Custom code, services, management                                                 |
| Resources     | Memory of 128 MB to 10 GB + vCPU / Net.<br>Max 15 min / single concurrency | Memory of 512MB - 120 GB, 0.25 -16 vCPU<br>Unlimited run time / multi concurrency |

Also, we can use both inside step functions

## Production-ready serverless service

### Retries & timeouts in the record handler

Set up configuration when connecting to services

```python
from boto3 import client
from botocore.config import Config

config = Config(
    retries={
        'max_attempts': 5, 
        'mode': 'adaptive',
    },
    read_timeout=30,
    connect_timeout=10,
)

s3_client = client('s3', config=config)
```

### CloudWatch loggin tips

- Add correlation id/custom id to all logs
- Reduce cost:
    - set `INFO` log level as default log level
    - Log only what matters most, otherwise, log as `DEBUG` level
    - Set retention policies on log groups (14 days)

### Upgrade your runtime

Python 3.10 to 3.13 improves performance

---

Ref:

https://www.youtube.com/watch?v=5wokwEtddtc
