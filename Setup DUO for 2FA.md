# Setup DUO for 2FA



![img](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/Picture1-11.png)


## 1. 安装和配置 Duo

1. 在 Duo 网站上[注册](https://signup.duo.com/) Duo 账户，然后[登录](https://admin.duosecurity.com/login?next=%2F)。

2. 在您的移动设备上安装 Duo 应用程序。当您从 Duo 收到文本或推送通知时，请按照说明对您的 Duo 账户进行身份验证。

3. 在您的 Duo Web 账户中，从左侧的导航窗格中选择**应用程序**。

4. 选择 **RADIUS** 进行安装。每个应用程序都有`Integration key`, `Secret key`, 和`API hostname`。

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221108101105514.png" alt="image-20221108101105514" style="zoom:67%;" />

5. 从导航窗格中选择**用户** > **添加用户**。

6. 在**用户名**中，输入您的最终用户的用户名。这些用户名必须与 Active Directory 用户的名称以及您的最终用户随后用来对其客户端 VPN 终端节点的连接进行身份验证的用户名一致。

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118182749198.png" alt="image-20221118182749198" style="zoom: 50%;" />

7. 选择每个用户并添加他们的电话号码。用户将收到一激活短信。点击短信内的链接将提示安装Duo Mobile并添加用户。



## 2. Deploy Radius Server

### Setup Radius Server

1. Create an Ubuntu instance. SSH into the server.

   ```bash
   sudo apt update
   sudo apt upgrade
   ```

2. Install dependencies

   ```bash
   sudo apt-get install build-essential libffi-dev perl zlib1g-dev
   ```

3. Download the most recent Authentication Proxy for Unix from https://dl.duosecurity.com/duoauthproxy-latest-src.tgz.

   ```
   wget --content-disposition https://dl.duosecurity.com/duoauthproxy-latest-src.tgz
   ```

4. Extract the Authentication Proxy files and build it as follows. Replace `duoauthproxy-XXX-src` with the actual file name.

   * The make process may take a while.

   ```bash
   ls -l
   tar xzf duoauthproxy-XXX-src.tgz
   cd duoauthproxy-XXXX-src
   make
   ```

5. Install the authentication proxy. 

   ```bash
   cd duoauthproxy-build
   sudo ./install
   ```



### Configure Radius Server

1. The Duo Authentication Proxy configuration file is `/opt/duoauthproxy/conf/authproxy.cfg`.

2. Edit the config file `authproxy.cfg`. 

   ```bash
   sudo vim /opt/duoauthproxy/conf/authproxy.cfg
   ```

3. Update the configuration as following with values from your DUO application.

   ```ini
   [duo_only_client]
    
   [radius_server_duo_only]
   ikey=<Integration Key>
   skey=<Secret Key>
   api_host=<API Hostname>.duosecurity.com
   failmode=safe
   radius_ip_1=<Directory Service IP 1>
   radius_secret_1=<Secret Key>
   radius_ip_2=<Directory Service IP 2>
   radius_secret_2=<Secret Key>
   port=1812
   client=duo_only_client
   ```

4. Here are the properties in above configuration.

   * When using the `[duo_only_client]` configuration, the Authentication Proxy will ignore primary credentials and perform Duo factor authentication only.

   | `ikey`            | Your integration key.                                        |
   | ----------------- | ------------------------------------------------------------ |
   | `skey`            | Your secret key.                                             |
   | `api_host`        | Your API hostname (api-XXXXXXXX.duosecurity.com).            |
   | `radius_ip_1`     | The IP address of your first AWS WorkSpaces Directory Controller. |
   | `radius_secret_1` | A secret to be shared between the proxy and your AWS WorkSpaces Directory. If you're on Windows and would like to encrypt this secret, see [Encrypting Passwords](https://duo.com/docs/authproxy-reference#encrypting-passwords) in the full Authentication Proxy documentation. |
   | `radius_ip_2`     | The IP address of your second AWS WorkSpaces Directory Controller. |
   | `radius_secret_2` | A secret to be shared between the proxy and your AWS WorkSpaces Directory. This secret should be the same as was used for `radius_secret_1`. If you're on Windows and would like to encrypt this secret, see [Encrypting Passwords](https://duo.com/docs/authproxy-reference#encrypting-passwords) in the full Authentication Proxy documentation. |
   | `client`          | The mechanism that the Authentication Proxy should use to perform primary authentication. This should correspond with a "client" section elsewhere in the config file.`duo_only_client`Do not perform primary authentication. Make sure you have a `[duo_only_client]` section configured.This parameter is optional if you only have one "client" section. If you have multiple, each "server" section should specify which "client" to use. |

### Start Proxy

1. Start proxy and make sure its status is running.

   ```bash
   sudo /opt/duoauthproxy/bin/authproxyctl restart
   sudo /opt/duoauthproxy/bin/authproxyctl status
   ```

2. Update security group of proxy server to allow inbound traffic at port 1812.



### Set Security Group for RADIUS Server

1. Allow **inbound connection** at **UDP 1812** from the security group of Active Directory Controller.

   ![image-20221118183154516](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221118183154516.png)

   

## 3. Update AWS Directory

1. In Workspaces > Directory, select the directory to be setup with 2FA.
2. Update the "Enable Multi-Factor Authentication" configuration.
   * Use private IP address of RADIUS server.

![RADIUS Multi-Factor Authentication](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/awsws-radius-server.png)




## 4. Reference

https://duo.com/docs/awsworkspaces

For Amazon Linux with Google Authenticator https://aws.amazon.com/blogs/desktop-and-application-streaming/integrating-freeradius-mfa-with-amazon-workspaces/



