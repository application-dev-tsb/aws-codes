Lambda + EMR

Basic Spark Job
```python
import boto3
import uuid

from commons import env

def delegate_emr(event, context):
    print('EMR')
    
    job = event['job']
    
    random_id = uuid.uuid1()
    s3_data = env('S3_BUCKET_DATA_PROCESSOR')
    es_host = env('ES_HOST')
    jar_data_proc = env('JAR_DATA_PROCESSOR')
    jar_es_hadoop = env('JAR_ES_HADOOP')
    worker_sg = env('WORKER_SG')
    worker_subnet = env('WORKER_SUBNET')
    
    emr = boto3.client('emr')
    lambda_client = boto3.client('lambda')
    
    step_name = job
    step_args = [
        'spark-submit', 
        '--deploy-mode',
        'cluster',
        '--class',
        'io.headhuntr.JobRunner',
        '--jars',
        jar_data_proc,
        jar_es_hadoop,
        '--job', job,
        '--esHost', 'https://{es}:443'.format(es=es_host),
        '--dmzDir', 's3://{s3}/temp/default'.format(s3=s3_data),
        '--workingDir', 's3://{s3}/temp/emr/{id}/workspace'.format(s3=s3_data,id=random_id)
    ]
    
    if job == 'generic':
        file = event['file']
        index = event['index']
    
        generic_job_params = [
            '--file', file,
            '--index', index
        ]
        step_args.extend(generic_job_params)
        step_name = 'Generic Job: {file} -> {index}'.format(file=file, index=index)
    
    step = {
        'Name': step_name,
        'ActionOnFailure': 'CONTINUE',
        'HadoopJarStep': {
            'Jar': 'command-runner.jar',
            'Args': step_args
        }
    }
    
    #use the event object
    my_tags = lambda_client.get_function(FunctionName='delegate')['Tags']
    print(my_tags)
    tags = []
    for key, value in my_tags.items():
        tags.append({'Key': key, 'Value': value})
    
    #TODO: filter for running clusters with the right name
    list_cluster_response = emr.list_clusters(ClusterStates=['STARTING', 'BOOTSTRAPPING', 'RUNNING', 'WAITING'])
    active_clusters = list_cluster_response['Clusters']
    if (active_clusters):
        cluster_id = active_clusters[0]['Id']
        return emr.add_job_flow_steps(JobFlowId=cluster_id, Steps=[step])
        
    return emr.run_job_flow(
        Name='data-processor',
        Applications = [{
            'Name': 'Spark'
        }],
        LogUri='s3://{s3}/temp/emr/{id}/logs'.format(s3=s3_data, id=random_id),
        ReleaseLabel='emr-5.30.1',
        Instances={
            'Ec2SubnetIds': [worker_subnet],
            'AdditionalSlaveSecurityGroups': [worker_sg],
            'AdditionalMasterSecurityGroups': [worker_sg],
            'InstanceGroups': [
                {
                    'Name': "Controller",
                    'Market': 'SPOT',
                    'InstanceRole': 'MASTER',
                    'InstanceType': 'm5.xlarge',
                    'InstanceCount': 1,
                },
                {
                    'Name': "Workers",
                    'Market': 'SPOT',
                    'InstanceRole': 'CORE',
                    'InstanceType': 'm5.xlarge',
                    'InstanceCount': 3,
                }
            ]
        },
        Steps=[step],
        VisibleToAllUsers=True,
        ServiceRole='EMR_DefaultRole',
        JobFlowRole='EMR_EC2_DefaultRole',
        Tags=tags
    )
```
