# Investigate Traffic Pass Through Transit Gateway



1. Choose 2 VPC, e.g. "DefaultVPC" and "MyVPC-VPC".

2. Create a Transit Gateway.

3. Attach it to the 2 VPC.

4. For each VPC, update its Route Table by adding a routing entry to point to Transit Gateway.

   * In MyVPC-VPC, point IPv4 CIDR of DefaultVPC to Transit Gateway.
   * In DefaultVPC, point IPv4 CIDR of MyVPC-VPC to Transit Gateway.

5. Enable flowlog of Transit Gateway.

6. Create a RDS in "DefaultVPC"

7. Create an EC2 instance as Bastion Host in "DefaultVPC". Create another EC2 instance "ubuntu" in "MyVPC-VPC".

8. Using terminal of bastion host, find out IP address of RDS endpoint.

   * For example, the IP address of this RDS is 172.31.33.125.

9. Using terminal of ubuntu EC2 instance, connect to RDS.

   * Permform this a few times to generate flowlog.

10. Create a Crawler in AWS Glue to crawl the flowlog in the S3 bucket.

11. Use Athena to query the table.

    ```
    select to_iso8601(from_unixtime(start)) AS start_time,
              to_iso8601(from_unixtime("end")) AS end_time,
              bytes
       from my_tgw_flowlog 
       where dstaddr = '172.31.33.125' or srcaddr = '172.31.33.125';
    ```

    