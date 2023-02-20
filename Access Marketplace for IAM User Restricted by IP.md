# Access Marketplace for IAM User Restricted by IP

#aws, #marketplace, #iam

## Background

You can setup IAM permissions such that users can only access AWS Console or services in either IP range, for example, by using following policy.

  ```json
  {
      "Version": "2012-10-17",
      "Statement": {
          "Effect": "Deny",
          "Action": "*",
          "Resource": "*",
          "Condition": {
              "NotIpAddress": {
                  "aws:SourceIp": [
                      "X.X.X.X/X"
                  ]
              },
              "Bool": {
                  "aws:ViaAWSService": "false"
              }
          }
      }
  }
  ```

But once you restrict the IP address, even user with administrator permission will encounter an error when they access AWS Marketplace.

<img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122201458632.png" alt="image-20221122201458632" style="zoom: 67%;" />

This article shows how to allow users to access Marketplace through assumed role.


## 1. Create an IAM Role

1. Create a new IAM Policy **"allow-license-manager"** which allows actions on `license-manager:*`.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "license-manager:*"
               ],
               "Resource": "*"
           }
       ]
   }
   ```

   

2. Create a new <u>IAM role</u> **"MarketplaceRole"**.

   * Select entity type **Custom trust policy**.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122201834474.png" alt="image-20221122201834474" style="zoom:67%;" />

   * Use following sample **Custom trust policy**, where `demo-user1` and `demo-user2` are 2 **existing** users which you would like to grant access to; `54.240.0.0/16` is the permitted IP address range to access AWS.

     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": {
                     "AWS": [
                         "arn:aws:iam::460453255610:user/demo-user1",
                         "arn:aws:iam::460453255610:user/demo-user2"
                     ]                    
                 },
                 "Action": "sts:AssumeRole"
             },
             {
                 "Effect": "Deny",
                 "Principal": {
                     "AWS": [
                         "arn:aws:iam::460453255610:user/demo-user1",
                         "arn:aws:iam::460453255610:user/demo-user2"
                     ]                    
                 },
                 "Action": "sts:AssumeRole",
                 "Condition": {
                     "NotIpAddress": {
                         "aws:SourceIp": "54.240.0.0/16"
                     }
                 }
             }
         ]
     }
     ```

   * Add permissions **AWSMarketplaceFullAccess** and **allow-license-manager**.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122203541146.png" alt="image-20221122203541146" style="zoom:80%;" />

   * Name it **"MarketplaceRole"**

3. Take note of this URL Link to switch roles in console.

   ![image-20221122205639313](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122205639313.png)

## 2. Grant User to Assume MarketplaceRole

1. Create another policy **"assume-marketplace-role"** for user to assume the newly created role `MarketplaceRole`. Replace role name accordingly if you different name for the role.

  ```json
  {
      "Version": "2012-10-17",
      "Statement": {
          "Effect": "Allow",
          "Action": "sts:AssumeRole",
          "Resource": "arn:aws:iam::460453255610:role/MarketplaceRole"
      }
  }
  ```

2. Add the newly created policy  **"assume-marketplace-role"** to the users which you put in the role **"MarketplaceRole"**, which are `demo-user1` and `demo-user2`. 


## 3. Test

1. Login using `demo-user1`.

2. Visit the URL, which you copied in section 1, to switch Role. 

3. Enter a Display Name

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122205830082.png" alt="image-20221122205830082" style="zoom:67%;" />

4. Now you can access to Marketplace.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122210217957.png" alt="image-20221122210217957" style="zoom:67%;" />

   

5. You can switch back when you are done.

   <img src="https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221122210250580.png" alt="image-20221122210250580" style="zoom:67%;" />
