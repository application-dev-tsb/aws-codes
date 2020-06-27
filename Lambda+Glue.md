# Lambda + Glue

```python
import boto3
import datetime

def delegate_glue(event, context):
    print('Glue')
    
    glue = boto3.client('glue', region_name='us-west-2')
    
    response = glue.start_job_run(JobName='data-processor',Arguments={'var1': 'val1'}, WorkerType='Standard', NumberOfWorkers=4)
```
