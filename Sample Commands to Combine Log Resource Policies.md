# Sample Commands to Combine Log Resource Policies

PutResourcePolicy 请求在目标的 CloudWatch Logs 添加对应的 "资源策略" (Resource Policy)。有可能你已经拥有 10 个 Resource Policy [3] 达到上限：    Resource policies - Up to 10 CloudWatch Logs resource policies "per Region" per account. This quota can't be changed.  



你跑一下这个命令，看看是不是已经有10个ResourcePolicy了 

```sh
aws logs describe-resource-policies > policies.json
```

NOTE：Resource policies for OpenSearch can be updated on AWS OpenSearch Service console.




## Steps

1. 修改 "OpenSearchService-bon-archive-Application-logs” 来包括 “Application-logs”，“Audit-logs”，“Index-logs”。
   
```sql
aws logs put-resource-policy --policy-name "OpenSearchService-bon-archive-Application-logs" --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"es.amazonaws.com\"},\"Action\":[\"logs:PutLogEvents\",\"logs:CreateLogStream\"],\"Resource\":\"arn:aws:logs:ap-southeast-1:267264053346:log-group:/aws/OpenSearchService/domains/bon-archive/*:*\"}]}"
```

2. 删除 “Audit-logs”，“Index-logs” 的策略。
   
```sql
aws logs delete-resource-policy --policy-name "OpenSearchService-bon-archive-Audit-logs"
aws logs delete-resource-policy --policy-name "OpenSearchService-bon-archive-Index-logs"
```

3. 修改 "OpenSearchService-bon-bigdatax-Application-logs” 来包括 ”Application-logs“，“Index-logs”。
   
```sql
aws logs put-resource-policy --policy-name "OpenSearchService-bon-bigdatax-Application-logs" --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"es.amazonaws.com\"},\"Action\":[\"logs:PutLogEvents\",\"logs:CreateLogStream\"],\"Resource\":\"arn:aws:logs:ap-southeast-1:267264053346:log-group:/aws/OpenSearchService/domains/bon-bigdatax/*:*\"}]}"
```

4. 删除“Index-logs”的缺略。
   
```sql
aws logs delete-resource-policy --policy-name "OpenSearchService-bon-bigdatax-Index-logs"
```

5. 修改“OpenSearchService-bon-biz-Application-logs”来包括“Application-logs”，“Audit-logs”，“Index-logs”。
   
```sql
aws logs put-resource-policy --policy-name "OpenSearchService-bon-biz-Application-logs" --policy-document "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"es.amazonaws.com\"},\"Action\":[\"logs:PutLogEvents\",\"logs:CreateLogStream\"],\"Resource\":\"arn:aws:logs:ap-southeast-1:267264053346:log-group:/aws/OpenSearchService/domains/bon-biz/*:*\"}]}"
```

6. 删除 “Audit-logs”，“Index-logs” 的策略。
   
```sql
aws logs delete-resource-policy --policy-name "OpenSearchService-bon-biz-Audit-logs"
aws logs delete-resource-policy --policy-name "OpenSearchService-bon-biz-Index-logs"
```



