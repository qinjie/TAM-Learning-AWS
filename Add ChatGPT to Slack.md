# Add ChatGPT to Slack



## Gather Tokens

### Setup Slack App

1. Login into Slack and and go to [Slack API website](https://api.slack.com/apps).

2. Click on Your Apps at top right corner and create a new app. 

   ![image-20230306085411462](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306085411462.png)

3. Select **from Scratch**. Give an app name and select the workspace which it will be used.

   ![image-20230306083707973](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306083707973.png)

4. From side menu, select **Settings > Socket Mode**. Enable socket mode, give the app token a name and generate it. Take note of the token string, which will be used as `slack-bot-token` in next session.

   ![image-20230306084521479](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306084521479.png)

5. From side menu, select **Features > OAuth & Permissions**. In the **Scopes** session, add following **Bot Token Scopes**: `app_mentions:read`, `channels:history`, `channels:read`, `chat:write`.

   ![image-20230306084016112](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306084016112.png)

6. From side menu, select **Features > Event Subscriptions**. Enable the Events. In **Subscribe to bot events**, add `app_mention`.

   ![image-20230306090437885](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306090437885.png)

7. From side menu, select **Install App**. Copy the Bot User OAuth Token, which will be used as `slack-app-token` in next session.

   ![image-20230306090035829](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306090035829.png)




### Get ChatGPT Token

1. Go to [ChatGPT API site](https://openai.com/blog/openai-api).
2. Click on Sign Up and login with your account.
3. Click on your Account Profile on top right corner and select **View API Keys**.
4. Create a new key, which will be used as `open-api-key` in next session.



## Setup Python Program

1. Store tokens in AWS Secrets Manager with a name `developer-me`.

   ![image-20230306091306168](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306091306168.png)

2. Create a python file `aws_secrets.py` and test.

   ```python
   from botocore.exceptions import ClientError
   import boto3
   import json
   
   SECRET_NAME = 'developer-me'
   REGION_NAME = 'ap-southeast-1'
   
   
   def get_secret():
       '''Get a secret json string from AWS Secret Manager and parse them into a dictionary'''
       # Create a Secrets Manager client
       session = boto3.session.Session()
       client = session.client(
           service_name='secretsmanager',
           region_name=REGION_NAME
       )
   
       try:
           get_secret_value_response = client.get_secret_value(
               SecretId=SECRET_NAME
           )
       except ClientError as e:
           # For a list of exceptions thrown, see
           # https://docs.aws.amazon.com/secretsmanager/latest/apireference/API_GetSecretValue.html
           raise e
   
       # Decrypts secret using the associated KMS key.
       response = get_secret_value_response['SecretString']
       secrets = json.loads(response)
   
       # Your code goes here.
       return secrets
   
   
   if __name__ == "__main__":
       tokens = get_secret()
       print(tokens)
   
   ```



## Add Bot to Slack

1. Create a slack channel in your workspace.

2. Integrate the app in the channel.

   ![image-20230306093829243](./assets/Add%20ChatGPT%20to%20Slack.assets/image-20230306093829243.png)
