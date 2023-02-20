# Setup Cloud9 on Existing Compute



You can setup Cloud9 using a new EC2 instance or an existing one. To use an existing compute instance, this instance need to have following software installed

* Python 2.7  (Other Python version will not work)
* Node.js (Use stable version v14)



## Setup Cloud9 on Existing Amazon Linux 2 Instance



### Prepare EC2 Instance

#### Install Python

1. Check whether **python** version 2.7 and **pip** version 2.7 are installed.

```bash
python --version
pip --version
```

2. If either of them is not installed, install them with following commands.

   ```bash
   sudo yum -y install python
   sudo yum -y install pip
   ```



#### Install Node

1. Check whether **node** is installed. If it is installed, check and take note of the path of **node**. You may skip this session if it is installed.

   ```bash
   node --version
   which node
   ```

2. Node Version Manager provides an easy way to manage and switch between multiple node versions. Install Node Version Manager.

   ```bash
   sudo yum install curl -y 
   curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash   
   source ~/.bashrc
   ```

3. You can check the list of node versions available.

   ```
   nvm list-remote
   ```

4. Install the stable version of node 14. Check and take note of the path of **node**.

   ```bash
   nvm install v14.21.2
   which node
   ```



#### Assign an Public/Elastic IP for the Instance

Cloud9 connects to EC2 instance through IP address. **Note:** If you prefer your EC2 instance to be within an intranet subnet, you may connect to it through a bastion host.

1. [Optional] You may want to assigned an elastic IP to your instance and use elastic IP in Cloud9 Environment configuration.



### Create Environment

1. Create a new Cloud9 environment. Choose "Existing compute".

2. Enter in User, Host and Port. Copy the SSH public key to clipboard. 

   * EC2 Instance need to open access at Port 22.

3. SSH into EC2 instance. Use Edit the `.ssh/authorized_keys` file and append the SSH key in clipboard.

   ```bash
   nano .ssh/authorized_keys
   ```

4. Expand the Additonal details. Put in the path to node.js binary.

5. Launch Cloud9 IDE.

6. Run the Cloud9 installer




