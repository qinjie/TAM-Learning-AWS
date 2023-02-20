# Setup Isengard with AWS CLI  


## A. Setup Isengard
1. Create an Isengard account for learning and development. 
   * https://w.amazon.com/bin/view/AWS_IT_Security/Isengard/Isengard_FAQ/#HAboutIsengard
   * Add a Console user
  
2. Go to Isengard > AWS Console Access (left pane). Search for your account.

   * https://isengard.amazon.com/console-access

   ![image-20221012180140369](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221012180140369.png)

   * Check the heart icon on the left to favorite it
   * Click on the account email address to access the account details. 

   ![image-20221012175954705](https://raw.githubusercontent.com/qinjie/picgo-images-2/master/img/image-20221012175954705.png)

   * Update Assume Role and Console Access timeout to max value, e.g. 720.

3. Bookmark the link which filters the profile by keyword. For example, https://isengard.amazon.com/console-access?filter=qinjie


## B. Setup mwinit with AWS CLI

We can use [ADA (Amazon Developer Account)](https://w.amazon.com/bin/view/DevAccount/Docs/) to refresh token automatically.

1. Perform Midway authentication in terminal using `mwinit`.
   * https://builderhub.corp.amazon.com/docs/builder-toolbox/user-guide/getting-started.html#midway-authentication

2. Install **Builder Toolbox** so that we can use `toolbox` in command line.
   * https://builderhub.corp.amazon.com/docs/builder-toolbox/user-guide/getting-started.html#install-toolbox-macos

3. Install **Amazon Developer Account (ADA) CLI** which allows access to developer repositories.
   * https://w.amazon.com/bin/view/DevAccount/Docs/

4. Open a terminal, login with 2FA using USB security key.

```
mwinit
```

5. In the same terminal, run following command which will refresh temporary credential from isengard

	* --account: isengard account id
	* --role: isengard account user role
	* --profile: (optional) update credential to a aws profile

```
ada credentials update --account=460453255610 --provider=isengard --role=Admin
```

6. You can save above command into a bash file, e.g. `refresh-isengard-credential.sh` and run it when you need to.

```
# Login 2FA using USB Key
mwinit
# Refresh credential from isengard into .aws/credential
ada credentials update --account=460453255610 --provider=isengard --role=Admin
```

7. Alternatively, you can add a profile in `.aws/config` which will fetch the credential from Isengard when command is run.

```
[profile isengard]
region = ap-southeast-1
output = json
credential_process=ada credentials print --account=460453255610 --provider=isengard --role=Admin
```

8. Test it with `aws s3 ls --profile isengard`

