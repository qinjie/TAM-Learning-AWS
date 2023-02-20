# Connect DBeaver to RDS and Redshift

**Tags:** 

* AWS
* DBeaver, RDS, Redshift, Bastion

**Pre-requists:**

* [Downlaod and install DBeaver](https://dbeaver.io/download/)



This article shows the steps to setup an AWS RDS cluster, a Redshift cluster, a bastion host, and how to configure DBeaver to connect to them through the bastion host. 

[DBeaver](https://dbeaver.io/) is a free multi-platform SQL client software application and a database administration tool. 

[Amazon RDS](https://aws.amazon.com/rds/) is a fully-managed relational database service by Amazon Web Services. It runs "in the cloud" designed to simplify the setup, operation, and scaling of a relational database for use in applications.

[Amazon Redshift](https://aws.amazon.com/redshift/) is a data warehouse product which forms part of the larger cloud-computing platform Amazon Web Services. It is built on top of technology from the massive parallel processing data warehouse company ParAccel, to handle large scale data sets and database migrations.

A [bastion host](https://en.wikipedia.org/wiki/Bastion_host) is a server whose purpose is to provide access to a private network from an external network, such as the Internet.



### 1. Bastion Host

#### Create Security Group

We will create a security groups for the bastion host, which should allows incoming traffic at port 22 and any outgoing traffic.

1. Login to AWS Console. Go to **EC2 service** > Network & Security > Security Groups.

2. Create a new security group
   * Security group name: <u>secgroup-bastion-host</u>
   * Description: <u>Security group for bastion host</u>
   * VPC: In the same VPC as RDS and Redshift cluster

3. Add an inbound rule to access SSH connection at port 22 from anywhere. 

   * NOTE: To better security, you may want to limit the source to your own IP. You may use https://www.whatismyip.com/ to find your current IP address.

   ![image-20221005145842550](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005145842550.png)

4. Leave outbound rules as it is.

   * By default, it allows all outbound traffic to anywhere.

#### Launch Bastion Host

1. Launch an EC2 instance. Give it a name e.g. <u>bastion-host</u>.

   **Application and OS Images:**

   * Choose <u>Amazon Linux 2 AMI</u>

   **Instance Type:**

   * Choose a small one, e.g. <u>t2.micro</u>

   **Key pair:**

   * Select an existing key pair

   **Network settings:**

   * Use existing security group and select the newly created `secgroup-bastion-host`

2. Leave other configurations at their default values. 
3. Launch the instance and take note of it's public IP address.

#### Test SSH Connection

Test connection to bastion host using terminal or any other SSH client, e.g. Termius.

* Use Terminal on Mac

  ```bash
  ssh -i <PEM_KEY_PATH> ec2-user@<BASTION_IP>
  ```



### 2. Setup RDS Cluster

#### Create Security Group

We will create a security groups for the RDS cluster, which should only allow incoming traffic at port 3306 from clients within the same VPC. It shall also allow any outgoing traffic.

1. Take note of the **PRIVATE IP address** of the bastion host.

2. Go to **EC2 service** > Network & Security > Security Groups.

3. Create a new security group

   * Security group name: <u>secgroup-rds-cluster</u>
   * Description: <u>Security group for rds cluster</u>
   * VPC: In the same VPC as RDS and Redshift cluster

4. Add an inbound rule to access SSH connection at **port 3306** from any instance in the same VPC.

   * Source is set to `172.31.0.0/16` because the private IP of the bastion host is `172.31.xx.xx`.

   ![image-20221005161204412](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005161204412.png)

5. Leave outbound rules as it is.

   * By default, it allows all outbound traffic to anywhere.

#### Create RDS Cluster

1. Go to RDS service in AWS Console and **create a database** in the same region as bastion host.

2. Update following settings:

   **Engine options & Templates:**

   * Choose <u>Standard create</u> as creation method.
   * Choose <u>Amazon Aurora</u> as engine type and <u>MySQL-Compatible</u> edition
   * Choose <u>Dev/Test</u> template

   **Settings:**

   * Set Master username and password.

   **Connectivity:**

   * Set <u>Public access</u> to <u>No</u>
   * Set <u>Existing VPC security groups</u> to <u>secgroup-rds-cluster</u>

3. Leave other settings with their default values.
4. Create the instance and wait for its status to become Available.

#### Test Connection from Bastion Host using MySQL Client

We will install MySQL client on bastion host to test connection to RDS cluster. Amazon documentation recommends MariaDB over MySQL client for Amazon Linux.

1. SSH into bastion host.

2. Install MariaDB client.

```bash
sudo yum -y update
curl -LsS -O https://downloads.mariadb.com/MariaDB/mariadb_repo_setup
sudo bash mariadb_repo_setup --os-type=rhel  --os-version=7 --mariadb-server-version=10.7
sudo rm -rf /var/cache/yum
sudo yum makecache
sudo yum -y install MariaDB-client
```

3. Connect to RDS using terminal.

```
mysql -h <ENDPOINT> -P 3306 -u <USER> -p
```

   ![image-20221005163254206](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005163254206.png)

4. Exit from MySQl command prompt

```bash
\q
```


### 3. Setup Redshift Cluster

#### Create Security Group

We will create a security groups for the Redshift cluster, which should only allow incoming traffic at port 5439 from clients within the same VPC. It shall also allow any outgoing traffic.

1. Take note of the **PRIVATE IP address** of the bastion host.

2. Go to **EC2 service** > Network & Security > Security Groups.

3. Create a new security group

   * Security group name: <u>secgroup-redshift-cluster</u>
   * Description: <u>Security group for redshift cluster</u>
   * VPC: In the same VPC as RDS and Redshift cluster

4. Add an inbound rule to access SSH connection at **port 5439** from any instance in the same VPC.

   * Source is set to `172.31.0.0/16` because the private IP of the bastion host is `172.31.xx.xx`.

   ![image-20221005163859212](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005163859212.png)

5. Leave outbound rules as it is.

   * By default, it allows all outbound traffic to anywhere.

#### Create Redshift Cluster

1. Go to Redshift service in AWS Console and **create a cluster** in the same region as bastion host.

2. Choose **Free tiral** which will use many default settings.

   **Database configurations:**

   * Set Admin username and password.

3. Leave other settings with their default values; Create the cluster; Wait for its status to become Available.

4. Check out the properties of the Redshift cluster.

   * Under <u>General Information</u>, take note of its <u>Endpoint</u> value.
   * Edit its <u>Network and security settings</u>, and change <u>VPC security groups</u> to <u>secgroup-redshift-cluster</u>

#### Test Connection from Bastion Host using psql Client

We will install [psql](https://www.postgresql.org/docs/current/app-psql.html), a terminal-based front-end to PostgreSQL, on bastion host to test connection to Redshift cluster.

1. SSH into bastion host.

2. Install psql client.

   ```bash
   sudo amazon-linux-extras install postgresql10
   ```

3. Connect to RDS using terminal.

   * NOTE: Remember to remove port and database from end of the Endpoint string.

   ```bash
   psql -h <ENDPOINT> -p 5439 -d <DATABASE_NAME> -U <USERID>
   ```

4. Exit from psql command prompt.

   ```bash
   \q
   ```

   ![image-20221005165445030](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005165445030.png)



### 4. Configure DBeaver

#### Connection for RDS

1. Start DBeaver.

2. Create a new connection and choose **MySQL**.

3. Go to SSH tab and create a new Profile.

   ![image-20221005170134689](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005170134689.png)

4. Create a network profile to use bastion host for SSH tunneling.

   * Set the public IP, username, and key for bastion host.
   * Click on <u>Test tunnel configuration</u> and make sure it is successful.

   ![image-20221005170341300](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005170341300.png)

5. Select the newly created profile.

   ![image-20221005170605013](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005170605013.png)

6. Set the server host to RDS endpoint; Set username and password.

7. Test the connection and make sure it is successful.

   ![image-20221005170805627](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005170805627.png)



#### Connection for Redshift

1. Create a new connection. Go to <u>SSH tab</u> and select <u>Profile Bastion</u>, which we created in previous section.

   ![image-20221005171801136](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005171801136.png)

2. In the <u>Main tab</u>, enter the settings with values from Redshift.

   * NOTE: Host string should not include port number and database name.

3. Click on Test Connection and make sure it is successful.

   ![image-20221005172040815](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221005172040815.png)
