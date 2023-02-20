# Connect to OpenSearch Dashboard through Bastion Host



For security reason, it's common to deploy an OpenSearch domain in a private subnet. But this also restrict the access to the domain's dashboard which is useful for domain administration purpose. This article demonstrates how to connect to an OpenSearch domain's dashboard through a Bastion Host, which is deployed to a public subnet of the same VPC.



### 1. Setup Bastion Host

1. Create a new security group <u>secgroup-bastion-host</u>.

   * Add an inbound rule to allow TCP connection at port 22 from anywhere, or from your own IP CIDR. You may use https://checkip.amazonaws.com/ to find your current IP address.
   * Leave outbound rules as it is. By default, it allows all outbound traffic to anywhere.

2. Launch an EC2 instance as a <u>bastion-host</u>.

   * Application and OS Images: <u>Amazon Linux 2 AMI</u>

   * Instance Type: Choose a small one, e.g. <u>t2.micro</u>

   * Key pair: Select an existing key pair

   * Network settings: Choose VPC, a public subnet, and the newly created security group `secgroup-bastion-host`.
   * Leave other configurations at their default values. 

3. Launch the instance and take note of it's public IP address.

4. Test connection to bastion host using terminal or any other SSH client, e.g. Termius.

   * Using Terminal on Mac

     ```bash
     ssh -i <PEM_KEY_PATH> ec2-user@<BASTION_IP>
     ```




### 2. Setup OpenSearch in VPC

1. Create a new security group `secgroup-opensearch-default`.

   * Add an inbound rule to allow HTTPS connection (TCP at port 443) from the security group of bastion host, i.e. `secgroup-bastion-host`
   * Leave outbound rule which allows connections to anywhere. 

2. Create a new OpenSeach Cluster. 

   * For testing purpose, we choose minimum resourse to save cost.

     * Development type = **Development and testing**.

     * **Auto-Tune** = Disable

     * Availbility Zones = **1-AZ** 

     * Instance Type = **t3.small.search**

     * Number of nodes = 1

   * Set Network to **VCP access**
     * Choose the **same** VPC as the bastion host, and a private subnet.
     * Use the newly created security group <u>`secgroup-opensearch-default`</u>

   * Optionally, you may or may not want to **Enable fine-grained access control**.
     * If you enable the fine-grained access control, select <u>Create master user</u>
     
   * For Access policy, select "Configure domain level access policy"
     * View policy in JSON format and update it by changing Effect from "Deny" to "Allow".
     
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
     
     

3. After domain is Active, take note of its <u>Domain endpoint (VPC)</u>.
   * For domains within VPC, their endpoints have the format of `vpc-*-*.*.es.amazonaws.com`.



### 3. Port Forwarding

1. Run following command to forward local port 9200 to port 443 of domain endpoint through the bastion host.

   * DOMAIN_ENDPOINT: the domain endpoint of the opensearch cluster **without** `https://`
   * The command will not return any message. 

   ```sh
   ssh -i <PEM_KEY_PATH> ec2-user@<BASTION_IP> -N -L 9200:DOMAIN_ENDPOINT:443
   ```

![image-20221124152252286](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221124152252286.png)

2. On web browser, go to OpenSearch Dashboards URL, which is the `https://DOMAIN_ENDPOINT/_dashboards`. 

   * Login with username and password which we set during cluster creation.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221124152622226.png" alt="image-20221124152622226" style="zoom:67%;" />

#### [Alternative] Using SSH Profile

Alternatively, you can create a SSH profile so that you don't need to remember the port forwarding command.

1. Edit SSH config file.

   ```
   vim ~/.ssh/config
   ```

2. At following profile `estunnel` above existing profiles.

   ```
   ########################
   # ElasticSearch Tunnel
   Host estunnel
   HostName <BASTION_HOST_PUBLIC_IP>
   User ec2-user
   IdentitiesOnly yes
   IdentityFile <PATH_TO_PEM_KEY>
   LocalForward 9200 <DOMAIN_ENDPOINT>:443 
   ```

3. Run the SSH command with host ID, i.e. `estunnel`.

   ```bash
   ssh -N estunnel
   ```



### 4. Useful OpenSearch API

Here are some useful OpenSearch APIs.

```apl
 # List all snapshots in a repo
GET /_snapshot/my-repo/_all

# Get informatiopn of a snapshot
GET /_snapshot/my-repo/20221130-054427

# Get snapshot in progress of taking/restoring
GET /_snapshot/my-repo/_status

# List all indices
GET /_cat/indices?format=json

# Add a document in repo. Create index if not exists
POST veggies/_doc
{
  "name":"beet",
  "color":"red",
  "classification":"root"
}

# Open an index
POST /kibana_sample_data_ecommerce/_open

# Get info of an index
GET /kibana_sample_data_ecommerce
```

