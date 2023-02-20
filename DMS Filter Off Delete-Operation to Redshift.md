# DMS Filter Off Delete-Operation to Redshift

#dms, #redshift

## Background

DMS is a popular tool to migrate data from a source database to a target data store. When it connects between RDS and Redshift, it synchronises tables between them, i.e. when data in the source table is deleted, the data will be removed from target table too.

This may not always be the desired behavior. For example, you may need to clean the source table for better performance by deleting old data, but you would like to keep them in the redshift table for archiving and audit purpose.

By default, DMS doesn't provide a direct configuration to filter off delete operations. This article demonstrates how to use `operation_indicator` to flag records in target tables as inserted, updated, or deleted in the source table, thus records deleted from the source aren't deleted from the target. Instead, the target record is flagged with a user-provided value to indicate that it was deleted from the source.

![image-20230220143547671](./DMS%20Filter%20Off%20Delete-Operation%20to%20Redshift.assets/image-20230220143547671.png)

## Steps

1. Create a DMS instance. Take note of its security group ID.

2. Create a Redshift cluster. Modify its security group to allow inbound connection at port 5439 from security group of DMS instance.

3. Create a RDs instance. Modify its security group to allow inbound connection at port 3306 from security group of DMS instance.

4. If you dont have a test table, you may use following SQL to create a test table.

   ```sql
   CREATE TABLE `test` (
     `id` bigint(20) NOT NULL,
     `first_name` text,
     `last_name` text,
     `email` text,
     `gender` text,
     `ip_address` text,
     PRIMARY KEY (`id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
   ```

5. In DSM Console, create a source endpoint from an existing RDS database and a target endpoint to an existing Redshift database. Test both endpoints to make sure both have successful status. 

   ![image-20230220150928185](./DMS%20Filter%20Off%20Delete-Operation%20to%20Redshift.assets/image-20230220150928185.png)

6. Create a database migration task using above endpoints.

   * Choose `Migrate existing data and replicate ongoing changes`.
   * In **Table Mappings** section, choose **Editing mode** =  **"JSON editor"**. 
   * In the 1st rule, update **object-locator** to match the schema and table of source database.
   * Add another rule by copy and paste in the JSON object, which will add a column with name **operation**. The value in this column can be <u>D, U or I</u>, which represents the data in source table is <u>Deleted, Updated or Inserted</u> respectively. 

   ```json
   {
     "rules": [
       {
         "rule-type": "selection",
         "rule-id": "535677346",
         "rule-name": "535677346",
         "object-locator": {
           "schema-name": "dev",
           "table-name": "test"
         },
         "rule-action": "include",
         "filters": []
       },
       {
         "rule-type": "transformation",
         "rule-id": "2",
         "rule-name": "2",
         "rule-target": "column",
         "object-locator": {
           "schema-name": "%",
           "table-name": "%"
         },
         "rule-action": "add-column",
         "value": "Operation",
         "expression": "operation_indicator('D', 'U', 'I')",
         "data-type": {
           "type": "string",
           "length": 50
         }
       }
     ]
   }
   ```



## Testing

1. Connect to source database. Run following SQL statements to insert 3 records.

   ```
   insert into test values(1, 'alan', 'alan', 'alan@gmail.com', 'male', '1.0.0.1');
   insert into test values(2, 'bob', 'bob', 'bob@gmail.com', 'male', '2.0.0.1');
   insert into test values(3, 'charlie', 'charlie', 'charlie@gmail.com', 'male', '3.0.0.1');
   ```

2. In Redshift console, use Query Editor to examine the data in the target table.

   ```sql
   SELECT * FROM "dev"."dev"."test" order by id;
   ```

   ![image-20230220160952366](./DMS%20Filter%20Off%20Delete-Operation%20to%20Redshift.assets/image-20230220160952366.png)

3. In source datase, update the 2nd record (id = 2), and delete the 3rd record (id=3).

   ```sql
   update test set ip_address = '2.0.0.2' where id = 2;
   delete from test where id = 3;
   ```

4. In Redshift console, examine the data in target table. The `operation` status of 2nd and 3rd records have changed to `U` and `D` respectively, which represent `Updated` and `Deleted` respectively.

   ![image-20230220161353712](./DMS%20Filter%20Off%20Delete-Operation%20to%20Redshift.assets/image-20230220161353712.png)



## Conclustion

This article demonstrates how to filter Delete operations in a DMS task to preserve data in target Redshift table when they are deleted from source table.

 
