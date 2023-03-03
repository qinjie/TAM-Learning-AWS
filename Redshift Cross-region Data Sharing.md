# Redshift Cross-Region Data Sharing



1. From the producer cluster's Datashares tab, create a database connection to current cluster.

![image-20230224165027401](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224165027401.png)

2. Create a datashare in the producer cluster.

![image-20230224165251270](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224165251270.png)

![image-20230224165628927](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224165628927.png)

![image-20230224165421155](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224165421155.png)



![image-20230224172850977](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224172850977.png)

3. Check the consumer's cluster namespace, e.g. `208cd717-0950-455b-b77e-7f7a7986082a`.

![image-20230224172622757](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224172622757.png)

4. In the producter cluster's namespace, run following command to grant datashare to consumer cluster.

```sql
GRANT USAGE ON DATASHARE cluster_singapore TO NAMESPACE 'CONSUMER_CLUSTER_NAMESPACE';
```

![image-20230224172602238](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224172602238.png)

5. Take note of the **Producer namespace** of the newly created datashare. Note: If the datashare is shared, the status will show as "Shared".

![image-20230224170607948](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224170607948.png)

6. From the consumer cluster, create a database connection to current cluster (itself).

![image-20230224170000432](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224170000432.png)

7. The datashare from producer cluster is shown in the list. Create a database from it. 

![image-20230224173037042](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224173037042.png)

![image-20230224173136216](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224173136216.png)

8. Examine the share database in the consumer cluster. 

![image-20230224173230187](assets/Redshift%20Cross-region%20Data%20Sharing.assets/image-20230224173230187.png)

