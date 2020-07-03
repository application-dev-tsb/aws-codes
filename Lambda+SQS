# Using SQS inside lambda

```python
import boto3
import uuid
import json

from commons import env

def delegate_command(event, context):
    
    job = event['job']
    
    if job == 'all':
        __load_all()
    else:
        return {
            'Status': 400,
            'Message': 'Unknown Command'
        }
    
    return {
        'Status': 200,
        'Message': 'Command Executed',
        'Job': job
    }
    
def __send(message):
    sqs = boto3.client('sqs')
    sqs.send_message(
        QueueUrl=env('COMMAND_QUEUE_URL'),
        MessageBody=json.dumps(message),
        MessageDeduplicationId=str(uuid.uuid1()),
        MessageGroupId='commands'
    )
    
```
