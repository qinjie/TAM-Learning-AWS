# Setup VPC Flowlog and Ingest into OpenSearch



Pre-requisite:

* Assume you already have an OpenSearch cluster ready to ingest the VPC flowlog.



## 1. Setup VPC Flowlog

1. Go to CloudWatch Console. Create a CloudWatch Log Group, e.g. `vpc-flowlog-myvpc`. 

2. Go to IAM Console. Create a new IAM Policy, e.g. `vpc-flowlog-policy`.

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
       {
           "Action": [
               "logs:CreateLogGroup",
               "logs:CreateLogStream",
               "logs:PutLogEvents",
               "logs:DescribeLogGroups",
               "logs:DescribeLogStreams"
           
           ],
           "Effect": "Allow",
           "Resource": "*"
       }
       ]
   }
   ```

3. Create an IAM Role with "Custom trust policy", e.g. `vpc-flowlog`, with following trust policy. Attach the newly created IAM policy.

   ```json
   {
   	"Version": "2012-10-17",
   	"Statement": [
   		{
   			"Effect": "Allow",
   			"Principal": {
   				"Service": "vpc-flow-logs.amazonaws.com"
   			},
   			"Action": "sts:AssumeRole"
   		}
   	]
   }
   ```

   

4. Go to VPC Console. Select the VPC. In its property pane, create a flow log.

   * Filter = `All`
   * Maximum aggregation interval = `1 minute`
   * Destination = `Send to CloudWatch Logs`
   * Destination log group = `vpc-flowlog-myvpc`
   * IAM roel = `vpc-flowlog`

5. Note: Need to make sure flow log's status remain as Active. If its status become `error xxx`, fix it and recreate the flowlog.



## 2. Ingest Flowlog into OpenSearch

1. Go to IAM Console. Create a new IAM role for AWS service > Lambda, e.g. `lambda-opensearch-cloudwatch-subscription`. 
   * Add permissions `AmazonOpenSearchServiceFullAccess` and `AWSLambdaVPCAccessExecutionRole`.
2. Go to CloudWatch. For the log group for our vpc flowlog, e.g. `vpc-flowlog-myvpc`, create a OpenSearch subscription filter.
   * Select account = `This account`
   * Amazon OpenSearch Service cluster = `mydomain-2`
   * Lambda IAM Execution Role = `lambda-open search-cloudwatch-subscription`
   * Log format = `Amazon VPC Flow Logs`
   * Subscription filter name = `{}`
3. Note: A lambda function will be created to pull logs from CloudWatch and push it into OpenSearch.
4. Note: Index sources from cloudwatch log come with name prefix `cwl-`.
5. Login into OpenSearch Dashboard. Go to *Stack Management > Index patterns > Create index pattern*. Create a new index pattern. 
   * index pattern name = `cwl-*`
   * Time field = `@timestamp`

6. On OpenSearch Dashboard, go to *Discover* and observe the logs in the newly created index pattern. 



