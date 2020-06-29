Lambda + EMR

Basic Spark Job
```python
import boto3

def delegate_emr(event, context):
    print('EMR')
    
    emr = boto3.client('emr', region_name='us-east-2')
    
    response = emr.run_job_flow(
        Name='boto_test',
        Applications = [{
            'Name': 'Spark'
        }],
        LogUri='s3://xxx/temp/emr/logs',
        ReleaseLabel='emr-5.30.1',
        Instances={
            'MasterInstanceType': 'm5.xlarge',
            'SlaveInstanceType': 'm5.xlarge',
            'InstanceCount': 3
        },
        Steps=[
        {
            'Name': 'Load Candidates',
            'ActionOnFailure': 'TERMINATE_CLUSTER',
            'HadoopJarStep': {
                'Jar': 'command-runner.jar',
                'Args': [
                    'spark-submit', 
                    '--deploy-mode',
                    'cluster',
                    '--class',
                    'io.xxx.candidate.CandidateLoader',
                    '--jars',
                    's3://io.xxx.deployment.us-east-2/jars/elasticsearch-hadoop/elasticsearch-hadoop-7.1.1.jar'
                    's3://io.xxx.deployment.us-east-2/jars/data-processor/data-processor_2.11-0.1.0.jar', 
                    'arg1',
                    'arg2',
                    'arg3'
                ]
            }
        },
    ],
    VisibleToAllUsers=True,
    ServiceRole='EMR_DefaultRole',
    JobFlowRole='EMR_EC2_DefaultRole')
    
    print(response)

    return 1
```
