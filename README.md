# aws-lambda
Lambda and Integrating with other AWS Service


## Lambda + Redshift Unload
```python
import boto3
import psycopg2
import datetime

def delegate(event, context):
    rs = boto3.client('redshift', region_name='us-west-2')
    cluster_creds = rs.get_cluster_credentials( DbUser='xxx',
                                                DbName='xxx',
                                                ClusterIdentifier='xxx',
                                                AutoCreate=False)
    print(cluster_creds)
    
    host = 'xxx.xxx.us-west-2.redshift.amazonaws.com'
    user = cluster_creds['DbUser']
    password = cluster_creds['DbPassword']
    con = psycopg2.connect(dbname='hhdata', host=host, port=5439, user=user, password=password)
    
    query = """
        unload ( 
            'select *
            from cand_profile_na
            where cand_id in (select xxx.xx_id from xxx)
            order by cand_id'
        )
        to 's3://xxx/temp/{id}/xxx'
        iam_role 'arn:aws:iam::xxx:role/xxx'
        format parquet
    """.format(id=datetime.datetime.now() )
    
    cur = con.cursor()
    cur.execute(query)
    
    cur.close()
    con.close()
    
    return 1
```
