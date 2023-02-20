# IAM Policy to Restrict Updating Route53 Recordsets



1. Create a public domain `qinjie.com` in Route53.
   * Add 2 TXT records `test.qinjie.com` and `banned.qinjie.com`

   ![image-20230214132849351](./IAM%20Policy%20to%20Restrict%20Updating%20Route53%20Recordsets.assets/image-20230214132849351.png)

2. Create another public domain `zhang.com` in Route53.

   * Add 2 TXT records, `test.zhang.com` and `banned.zhang.com`.

   ![image-20230214132912533](./IAM%20Policy%20to%20Restrict%20Updating%20Route53%20Recordsets.assets/image-20230214132912533.png)

3. Now we have 2 public domains for testing.

   ![image-20230214132936283](./IAM%20Policy%20to%20Restrict%20Updating%20Route53%20Recordsets.assets/image-20230214132936283.png)

   

4. Create an IAM Role `lambda_update_route53_records` for Lmabda with following policy, which allows role to update recordset `test.qinjie.com` in domain `qinjie.com`. Set the permissions (Inline Policy) as following, where `Z0573923n2WBO20BEZE484` is the Zone ID of `qinjie.com`.

     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": "route53:ChangeResourceRecordSets",
                 "Resource": "arn:aws:route53:::hostedzone/Z05739232WBO20BEZE484",
                 "Condition": {
                     "ForAllValues:StringEquals": {
                         "route53:ChangeResourceRecordSetsNormalizedRecordNames": [
                             "test.qinjie.com"
                         ],
                         "route53:ChangeResourceRecordSetsRecordTypes": [
                             "TXT"
                         ],
                         "route53:ChangeResourceRecordSetsActions": [
                             "CREATE",
                             "UPSERT",
                             "DELETE"
                         ]
                     }
                 }
             }
         ]
     }
     ```

6. Set its Trust Relationships.

```json
{
   "Version": "2012-10-17",
   "Statement": [
       {
           "Effect": "Allow",
           "Principal": {
               "Service": "lambda.amazonaws.com"
           },
           "Action": "sts:AssumeRole"
       }
   ]
}
```

7. Create a lambda function with following code. Set role to `lambda_update_route53_records `.

```python
import json
import boto3

def lambda_handler(event, context):

    client = boto3.client('route53')

    # TEST UPDATING OTHER RECORDSETS IN CORRECT DOMAIN

    zone_id = 'Z05739232WBO20BEZE484' # qinjie.com
    value = "aaaaa"
    record_names = ['test.qinjie.com', 'banned.qinjie.com']

    for record_name in record_names:
        print(f"Updating {record_name}")
        try:
            response = client.change_resource_record_sets(
                HostedZoneId=zone_id,
                ChangeBatch={
                    "Comment": f"Update TXT Fields {record_name}",
                    "Changes": [
                        {
                            "Action": "UPSERT",
                            "ResourceRecordSet": {
                                "Name": record_name,
                                "Type": "TXT",
                                "TTL": 180,
                                "ResourceRecords": [
                                    {
                                        "Value": f'"{value}"'
                                    },
                                ],
                            }
                        },
                    ]
                }
            )
            print(f"Success: {response}")
        except Exception as ex:
            print(f"Error: {ex}")
        print('+++++++++++++++++++++++++++++++++++++++')
    
    # TEST UPDATING RECORDSETS IN WRONG DOMAIN

    zone_id = 'Z04824712LR428SMYXA4A' # zhang.com
    value = 'aaaaa'
    record_names = ['test.zhang.com', 'banned.zhang.com']
    
    for record_name in record_names:
        print(f"Updating {record_name}")
        try:
            response = client.change_resource_record_sets(
                HostedZoneId=zone_id,
                ChangeBatch={
                    "Comment": f"Update TXT Fields {record_name}",
                    "Changes": [
                        {
                            "Action": "UPSERT",
                            "ResourceRecordSet": {
                                "Name": record_name,
                                "Type": "TXT",
                                "TTL": 180,
                                "ResourceRecords": [
                                    {
                                        "Value": f'"{value}"'
                                    },
                                ],
                            }
                        },
                    ]
                }
            )
            print(f"Success: {response}")
        except Exception as ex:
            print(f"Error: {ex}")
        print('+++++++++++++++++++++++++++++++++++++++')

    return {
        'Message': "DONE"
    }

```



8. Run the Lambd to test. Can see that only updating of first record set is successful. 

![image-20230214133509350](./IAM%20Policy%20to%20Restrict%20Updating%20Route53%20Recordsets.assets/image-20230214133509350.png)



#### References

1. https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/route53.html
2. https://stackoverflow.com/questions/47527575/aws-policy-allow-update-specific-record-in-route53-hosted-zone
3. https://aws.amazon.com/cn/about-aws/whats-new/2022/09/amazon-route-53-support-dns-resource-record-set-permissions/





