# List DMS Table Stats



1. List all DMS tasks.

```bash
aws dms describe-replication-tasks --query 'ReplicationTasks[*].[ReplicationTaskArn]' --output text
```

![image-20230301181012699](assets/List%20DMS%20Tasks%20Table%20Stats.assets/image-20230301181012699.png)

2. List a stats of a DMS task.

```
aws dms describe-table-statistics --replication-task-arn arn:aws:dms:ap-southeast-1:460453255610:task:5O56XPY4WBIPIHXGYAG7LYLC3TD3WMLCBPZPXZQ --query 'TableStatistics[*].[SchemaName,TableName, FullLoadRows, Inserts, Deletes, Updates, FullLoadStartTime]' --output text
```

![image-20230301181224753](assets/List%20DMS%20Tasks%20Table%20Stats.assets/image-20230301181224753.png)

3. Combine the 2 commands using `xargs -I {}`.

```bash
aws dms describe-replication-tasks --query 'ReplicationTasks[*].[ReplicationTaskArn]' --output text | xargs -I {} aws dms describe-table-statistics --replication-task-arn {} --query 'TableStatistics[*].[SchemaName,TableName, FullLoadRows, Inserts, Deletes, Updates, FullLoadStartTime]' --output text
```

![image-20230301181213579](assets/List%20DMS%20Tasks%20Table%20Stats.assets/image-20230301181213579.png)