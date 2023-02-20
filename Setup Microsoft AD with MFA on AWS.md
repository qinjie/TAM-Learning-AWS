# Setup Microsoft AD with MFA on AWS

This article demonstrates 

* how to create Microsoft Active Directory in AWS
* how to add MFA to Microsoft AD
* how to use it to authenticate AWS WorkSpaces



## 1. Create AWS Managed Microsoft AD

1. Go to **Directory Service** on AWS Console.

2. Go to **Active Directory > Directories** in left pane.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116140949869-20221118183709150.png" alt="image-20221116140949869" style="zoom: 67%;" />

3. Click on **Set up directory** button to create a new directory.

   * Select `AWS Managed Microsoft AD`.

   ![image-20221116141206415](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116141206415.png)

   * Choose an Edition.
   * Set the fully qualified domain name.
   * Set an password for `Admin` user. We will use this username `Admin` and password to join an EC2 instance to this domain.

   ![image-20221116141242190](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116141242190.png)

   ![image-20221116141415109](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116141415109.png)

   * Use default VPC.

   ![image-20221116141739164](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116141739164.png)

   * Click on **Create directory** button to create the directory.

4. It will take a while for the Directory to be ready. After it is ready, view its details.
5. In **Networking & security** tab, take note of its 2 IP addresses in **DNS address**.



## 2. Manage Microsoft AD Users

### Create a Windows EC2 Instance

1. Go to **EC2** in AWS Console. Click on **Launch Instances" button.

2. Launch a new `Windows` instance with `Microsoft Windows Server 2019 Base`.

   * Choose instance type `t3.small` or larger.
   * Pick an existing key pair or create a new one. This key pair will be used to decrypt initial Windows Administorator password.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221116143427065.png" alt="image-20221116143427065" style="zoom: 50%;" />

3. Leave other settings with their default values. Click on **Launch instance** button to launch instance.

 

### Login to Windows EC2 Instance

1. EC2 instance needs permission to access Active Directory. Create a new IAM role, e.g. `WindowsADRole`, with following 2 permissions.

   * AmazonSSMManagedInstanceCore
   * AmazonSSMDirectoryServiceAccess

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117133456033.png" alt="image-20221117133456033" style="zoom:67%;" />

2. Update the IAM Role of EC2 instance to the newly created role, e..g `WindowsADRole`. 

   * **Actions > Security > Modify IAM role**.

   ![image-20221117133143675](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117133143675.png)

3. Select the instance in AWS EC2 Console. Get Windows' login password. 

   * You need to use the key pair used during the EC2 instance provision. 

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117132503686.png" alt="image-20221117132503686" style="zoom:80%;" />

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117132750949.png" alt="image-20221117132750949" style="zoom:67%;" />

4. Using Microsoft Remote Desktop, login as `Administrator` and password.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117132923074.png" alt="image-20221117132923074" style="zoom:80%;" />

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221117132937372.png" alt="image-20221117132937372" style="zoom:80%;" />

### Join Windows EC2 Instance to the AD

1. After login into Windows, run following command in command prompt to open the TCP/IPv4 properties.

   ```
   %SystemRoot%\system32\control.exe ncpa.cpl
   ```

   * Right click on the network adapter, go to Properties.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118135628185.png" alt="image-20221118135628185" style="zoom: 50%;" />

   * Update DNS Server IP with IP of the Microsoft AD.

     <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118140008976.png" alt="image-20221118140008976" style="zoom: 50%;" />

2. Run following command to open **System Properties**.  

   ```
   %SystemRoot%\system32\control.exe sysdm.cpl
   ```

3. On **Computer Name** tab, click on **Change...** button. Enter fully-Aqualified name of the domain.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118141357738.png" alt="image-20221118141357738" style="zoom: 50%;" />

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118142042533.png" alt="image-20221118142042533" style="zoom: 33%;" />

4. Restart the computer for the changes to take effect.



### Install AD Tools

1. Start PowerShell in administrator mode. Run following command to install AD administration tools.

   ```
   Install-WindowsFeature RSAT-ADDS
   ```

2. Start **Server Manager**. Click on **Add roles and features**.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118142729101.png" alt="image-20221118142729101" style="zoom:50%;" />

3. In the **Add Roles and Features Wizard** choose **Installation Type**, select **Role-based or feature-based installation**, and choose **Next**.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118142824108.png" alt="image-20221118142824108" style="zoom:50%;" />

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118142930258.png" alt="image-20221118142930258" style="zoom: 50%;" />

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118143103044.png" alt="image-20221118143103044" style="zoom:50%;" />

   

   ### Add a User to AD

   1. Logout of the computer as Administrator. 

   2. Login as the Admin (domain user). You need to update the login user in Remote Desktop.

      <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118143512346.png" alt="image-20221118143512346" style="zoom: 50%;" />

   

   3. Start the **Active Directory Users and Computers** tool in the **Administrative Tools** folder. Or run following command.

      ```
      %SystemRoot%\system32\dsa.msc
      ```

   4. Add a new user in the domain.

      <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118143724605.png" alt="image-20221118143724605" style="zoom: 50%;" />

      <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118143909459.png" alt="image-20221118143909459" style="zoom:50%;" />

      

## 3. Launch a Workspace using AWS Managed Microsoft AD

1. Register a directory to be used for Workspaces.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118151245225.png" alt="image-20221118151245225" style="zoom: 33%;" />

2. Following this article to create a workspaces for the AD user **user1**.

   * https://docs.aws.amazon.com/workspaces/latest/adminguide/launch-workspace-microsoft-ad.html



3. Launch Workspaces app and login with an AD user, e.g. `user1`. 



## 4. Integrate MFA Server with AD

WorkSpaces only works with MFA providers that support RADIUS. If you already have a RADIUS server ready, you can integrate it with AD. 

### Configure the Security Group of AD Controller

By default, the AWS security group for the Directory Services instance heavily restrict the inbound and outbound traffic. In order to enable RADIUS, you’ll need to add **inbound and outbound** **UDP 1812** to the RADIUS.

1. Take note of the Directory ID of your active directory. 

<img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118154056994.png" alt="image-20221118154056994" style="zoom: 80%;" />

2. Search for related Security group for AD Controller using directory ID.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118175542700.png" alt="image-20221118175542700" style="zoom:80%;" />

3. Edit the outbound rule for the security group of the Active Directory to allow UDP 1812 (or the Radius service port) for the destination IP (private IP) of your Radius Server. Or, you can allow all traffic if your use case lets you do so.![image-20221118180250604](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118180250604.png)

4. Leave its inbound rules unchanged.



### Configure AWS Security Group Rules of RADIUS

1. Edit the security group of RADIUS server. Add an **inbound** rule to allow **UDP 1812** from AD Controller by its security group.

   ![image-20221118182024634](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118182024634.png)

2. No change to its outbound rules.

   ![image-20221118182423415](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118182423415.png)



### Enable MFA for AD

1. In Directory Service and select the AD service.
2. Click on **Multi-Factor Authentication** tab, and click on **Enable Multi-Factor Authentication**.
   * Enter RADIUS Server private IP.
   * Enter a Shared Secret Code.
   * [Optional] Set Timeout = 20 seconds, Max Retries = 3.



### Test on WorkSpaces App

<img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118182502390.png" alt="image-20221118182502390" style="zoom:50%;" />



## Reference

* https://docs.aws.amazon.com/directoryservice/latest/admin-guide/simple_ad_install_ad_tools.html
* https://docs.aws.amazon.com/directoryservice/latest/admin-guide/simple_ad_join_windows_instance.html
* https://docs.aws.amazon.com/directoryservice/latest/admin-guide/simple_ad_manage_users_groups_create_user.html
* https://docs.aws.amazon.com/workspaces/latest/adminguide/launch-workspace-microsoft-ad.html#create-workspace-microsoft-ad
* https://thevirtualhorizon.com/2017/05/15/integrating-duo-mfa-and-amazon-workspaces/





