# Manual Snapshot of OpenSearch without Fine-Grained Access Control

#opensearch, #snapshot

This article demonstrates how to use a Lambda function to take manual snapshot of a OpenSearch cluster which has **not** enabled its fine-grained access control.

### 1. Setup OpenSearch in VPC

1. Create a security group for the lambda function, e.g. `secgroup-lambda-elasticsearch`.

   * No need to add inbound rule.
   * Keep default outbound rule.

2. Create a new security group for the OpenSearch domain, e.g. <u>`secgroup-elasticsearch-default`</u>.

   * Add an inbound rule at TCP port 443 with source from security group `secgroup-lambda-elasticsearch`. 
   * Keep default outbound rule.

3. Create a new OpenSeach Cluster, e.g. <u>`mydomain`</u>. This may take a few minutes.

   * If it is <u>for testing purpose</u>, choose minimum resourse to save cost.

     * Development type = **Development and testing**.

     * **Auto-Tune** = Disable

     * Availbility Zones = **1-AZ** 

     * Instance Type = **t3.small.search**

     * Number of nodes = 1
   * Set Network to **VCP access**
     * Choose VPC, subnet.
     * Use the newly created security group <u>`secgroup-elasticsearch-default`</u>
   * Do **NOT** enable fine-grained access control.
   * For Access policy, select "Configure domain level access policy"
     * View policy in JSON format and update it by changing Effect from "Deny" to "Allow".
     * Ignore warning message.

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "*"
         },
         "Action": "es:*",
         "Resource": "arn:aws:es:ap-southeast-1:460453255610:domain/mydomain-2/*"
       }
     ]
   }
   ```

   * Leave other settings at default.

### 4. Deploy Lambda Function to Create OpenSearch Snapshot

1. Create a S3 bucket to store OpenSearch snapshots.

2. Create an IAM policy `elasticsearch-snapshot-policy`, which will be attached to an IAM role  for lambda function to work with OpenSearch service.

   * We will use `ElasticSearchSnapshotLambdaRole` as IAM role name.
   * Replace ACCOUNT_ID, IAM_ROLE_NAME, BUCKET_NAME accordingly.

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "VisualEditor0",
               "Effect": "Allow",
               "Action": [
                   "iam:PassRole"
               ],
               "Resource": [
                   "arn:aws:iam::ACCOUNT_ID:role/IAM_ROLE_NAME"
               ]
           },
           {
               "Sid": "VisualEditor1",
               "Effect": "Allow",
               "Action": [
                   "s3:ListBucket"
               ],
               "Resource": [
                   "arn:aws:s3:::BUCKET_NAME"
               ]
           },
           {
               "Sid": "VisualEditor2",
               "Effect": "Allow",
               "Action": [
                   "s3:PutObject",
                   "s3:GetObject",
                   "s3:DeleteObject"
               ],
               "Resource": "arn:aws:s3:::BUCKET_NAME/*"
           },
           {
               "Sid": "VisualEditor3",
               "Effect": "Allow",
               "Action": [
                   "es:ESHttpPut",
                   "es:ESHttpGet",
                   "es:ESHttpPost",
                 	"es:ESHttpDelete"
               ],
               "Resource": [
                   "arn:aws:es:REGION:ACCOUNT_ID:domain/*"
               ]
           }
       ]
   }
   ```

3. Create an IAM role `ElasticSearchSnapshotLambdaRole` with following trust relationship. 

   * Attach above policy `elasticsearch-snapshot-policy` and the AWS managed policy`AWSLambdaVPCAccessExecutionRole` to the role.

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "Service": [
                       "lambda.amazonaws.com",
                       "ssm.amazonaws.com",
                       "es.amazonaws.com"
                   ]
               },
               "Action": "sts:AssumeRole"
           }
       ]
   }
   ```

   

4. Create a Lambda function `SnapshotElasticSearch`.

   * Choose Python 3.9+ as Runtime.
   * Choose same VPC, Subnet as the OpenSearch domain. 
   * Use the Security Group `secgroup-lambda-elasticsearch` which is created in section 1. 
   * Use the newly created IAM role `ElasticSearchSnapshotLambdaRole`. 

5. Use following Python code.

   * Update <DOMAIN_ENDPOINT_WITH_HTTPS>, <BUCKET_NAME>, <AWS_REGION>, <ARN_OF_IAM_ROLE_OpensearchSnapshotLambdaRole> and <REPOSITORY_NAME> accordingly.

   ```python
   import json
   from typing import Dict, Tuple
   
   import boto3
   import requests
   from requests_aws4auth import AWS4Auth
   from datetime import datetime
   
   
   def lambda_handler(event, context):
       # Get region and credential
       service = 'es'
       session = boto3.session.Session()
       region = session.region_name
       credentials = session.get_credentials()
       awsauth = AWS4Auth(credentials.access_key, credentials.secret_key, region, service, session_token=credentials.token)
   
       # Settings
       host = '<DOMAIN_ENDPOINT_WITH_HTTPS>'
       bucket_name = '<BUCKET_NAME>'
       region = '<AWS_REGION>'
       role_arn = '<ARN_OF_IAM_ROLE_LAMBDA>'
       repo_name = '<REPOSITORY_NAME>'
   
       # # Register repository
       # register_repository(host, awsauth, repo_name, bucket_name, region, role_arn)
       #
       # # List all repositories
       # list_all_repositories(host, awsauth)
   
       # Create a snapshot
       snapshot_name = take_snapshot(host, awsauth, repo_name)
       print(snapshot_name)
   
       # # Get snapshot in-progress
       # get_snapshot_status(host, awsauth)
       #
       # # List all snapshots in all repository
       # snapshots = list_snapshots_in_repo(host, repo_name, awsauth)
       #
       # # Delete a snapshot by name
       # if len(snapshots) > 0:
       #     delete_one_snapshot(host, awsauth, repo_name, snapshot_name=snapshots[0].get('snapshot'))
       #
       # # List all snapshots in all repository after deletion
       # list_snapshots_in_repo(host, repo_name, awsauth)
   
       return {
           'statusCode': 200
       }
   
   
   def get_snapshot_status(host: str, awsauth: AWS4Auth, repo_name: str = None, snapshot_name: str = None):
       """
       Retrieves a detailed description of the current state for each shard participating in the snapshot.
       """
       if repo_name and snapshot_name:
           path = f'/_snapshot/{repo_name}/{snapshot_name}/_status'
       elif repo_name:
           path = f'/_snapshot/{repo_name}/_status'
       else:
           path = f'/_snapshot/_status'
   
       url = host + path
       r = requests.get(url, auth=awsauth)
       print(r.text)
   
   
   def list_snapshots_in_repo(host: str, repo_name: str, awsauth: AWS4Auth) -> Dict:
       """
       List all snapshots in a repository
       """
       path = f'/_snapshot/{repo_name}/_all'
       url = host + path
   
       r = requests.get(url, auth=awsauth)
       snapshots = r.json().get("snapshots", [])
       print(f'Snapshot count = {len(snapshots)}')
       print(r.text)
   
       return snapshots
   
   
   def list_all_repositories(host: str, awsauth: AWS4Auth):
       """
       List all repositories
       """
       path = '/_snapshot/_all'
       url = host + path
   
       r = requests.get(url, auth=awsauth)
       print(r.text)
   
   
   def register_repository(host: str, awsauth: AWS4Auth, repo_name: str, bucket_name: str, region: str, role_arn: str):
       """
       Register a snapshot repository
       """
       path = f'/_snapshot/{repo_name}'
       url = host + path
   
       payload = {
           "type": "s3",
           "settings": {
               "bucket": bucket_name,
               "region": region,
               "role_arn": role_arn
           }
       }
   
       headers = {"Content-Type": "application/json"}
       r = requests.put(url, auth=awsauth, json=payload, headers=headers)
       print(r.text)
   
   
   def take_snapshot(host: str, awsauth: AWS4Auth, repo_name: str, snapshot_name: str = None) -> str:
       """
       Take a snapshot in a repo. If snapshot_name is omitted, it will use current datetime string as name.
       Return snapshot name.
       """
       if snapshot_name is None:
           # Use current datetime as snapshot name
           now = datetime.now()
           snapshot_name = now.strftime("%Y%m%d-%H%M%S")
       path = f'/_snapshot/{repo_name}/{snapshot_name}'
       url = host + path
   
       r = requests.put(url, auth=awsauth)
       print(r.text)
   
       return snapshot_name
   
   
   def delete_one_snapshot(host: str, awsauth: AWS4Auth, repo_name: str, snapshot_name: str):
       """
       Deletes a snapshot.
       """
       path = f'/_snapshot/{repo_name}/{snapshot_name}'
       url = host + path
   
       r = requests.delete(url, auth=awsauth)
       print(r.text)
   
   
   def delete_one_repository(host: str, awsauth: AWS4Auth, repo_name: str):
       """
       Deletes a snapshot.
       """
       path = f'/_snapshot/{repo_name}'
       url = host + path
   
       r = requests.delete(url, auth=awsauth)
       print(r.text)
   
   ```



### 5. Test Lambda Function

We will update and run the lambda function to register snapshot repository, and then to take snapshot.

1. Update the `lambda_handler()` function of Lambda, which is to register a snapshot repository.

   ```python
   def lambda_handler(event, context):
       # Get region and credential
       service = 'es'
       session = boto3.session.Session()
       region = session.region_name
       credentials = session.get_credentials()
       awsauth = AWS4Auth(credentials.access_key, credentials.secret_key, region, service, session_token=credentials.token)
   
       # Settings
       host = '<DOMAIN_ENDPOINT_WITH_HTTPS>'
       bucket_name = '<BUCKET_NAME>'
       region = '<AWS_REGION>'
       role_arn = '<ARN_OF_IAM_ROLE_LAMBDA>'
       repo_name = '<REPOSITORY_NAME>'
   
       # Register repository
       register_repository(host, awsauth, repo_name, bucket_name, region, role_arn)
   
       # # Create a snapshot
       # snapshot_name = take_snapshot(host, awsauth, repo_name)
       # print(snapshot_name)
       
       # List all snapshots in all repository after deletion
       list_snapshots_in_repo(host, repo_name, awsauth)
       
       return { 'statusCode': 200 }
   ```

2. Run the lambda to register a snapshot repository. 

   * Make sure there is no error/warning in the execution output.

3. Update the `lambda_handler()` function to take snapshot.

   ```python
   def lambda_handler(event, context):
       # Get region and credential
       service = 'es'
       session = boto3.session.Session()
       region = session.region_name
       credentials = session.get_credentials()
       awsauth = AWS4Auth(credentials.access_key, credentials.secret_key, region, service, session_token=credentials.token)
   
       # Settings
       host = '<DOMAIN_ENDPOINT_WITH_HTTPS>'
       bucket_name = '<BUCKET_NAME>'
       region = '<AWS_REGION>'
       role_arn = '<ARN_OF_IAM_ROLE_LAMBDA>'
       repo_name = '<REPOSITORY_NAME>'
   
       # # Register repository
       # register_repository(host, awsauth, repo_name, bucket_name, region, role_arn)
   
       # Create a snapshot
       snapshot_name = take_snapshot(host, awsauth, repo_name)
       print(snapshot_name)
   
       # List all snapshots in all repository
       snapshots = list_snapshots_in_repo(host, repo_name, awsauth)
      
       return { 'statusCode': 200 }
   ```

4. Run the lambda function and make sure there is no error/warning message in the output.

5. Check the S3 bucket for the generated files.



### 6. Add EventBridge Trigger

We can use EventBridge to setup a daily schedule to run the Lambda function. 

1. Open the Lambda Function. Go to Configuration > Triggers > Add Trigger.
2. Choose **EventBridge (CloudWatch Events)**
   * Create a new rule
   * Choose Rule type = Schedule expression
   * Set the expression to a cron expression, e.g. cron(0 16 * * ? *)


### Reference

* https://docs.aws.amazon.com/opensearch-service/latest/developerguide/vpc.html#vpc-security

