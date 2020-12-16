# serverless-yfm

## Create GitHub App

Notice that here we have variables for second Cloud Function: `APP_ID`, `INSTALLATION_ID`, `PRIVATE_KEY`.
  
- Open Github
- Click on your avatar
- Click 'Settings' -> 'Developer settings' -> 'Github App'
- Click 'New Github App'
  - Enter 'GitHub App name'
  - Enter default url for 'Homepage URL'
  - Check 'Expire user authorization tokens'
  - For 'Webhook' -> Uncheck 'Active'
  - Update 'Repository permissions' -> Checks: Read & Write
  - Choose 'Only on this account'
  - Click 'Create Github App'
- On 'General' tab
  - `APP_ID`: In 'About' -> App ID
  - `PRIVATE_KEY`: In 'Private keys' -> Generate private key
- Install App
 - Your App
 - Click 'Install App'
 - Click 'Install' -> Check 'Only Select repository' and select your repository -> Install
 - `INSTALLATION_ID`: Copy ID from url -> https://github.com/settings/installations/<installation_ID>


## Create 'Message Queue'

- Open Message Queue
- Click 'Create Queue'
- Fill data
  - Enter 'Name'
  - type 'Standart'
  - Click 'Create'


## Create 'Service Account'

Notice that here we have variables for first Cloud Function: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.

- Open your Catalog -> 'Service accounts'
- Click 'Create Service account'
  - Enter 'Name'
  - Add rols: `iam.serviceAccounts.user`, `serverless.functions.invoker`, `editor`
  - Click 'Create'
- Open new service account
    - Click 'Create new key'
    - Select 'Create static access key'
    - Add 'Description'
    - Click 'Create'
    - Save `AWS_ACCESS_KEY_ID`: Key ID
    - Save `AWS_SECRET_ACCESS_KEY`: Your secret key


## Create 'Could Function'
[Official documentation]

Create 2 functions:
- Enter name
- Click 'Create'
- Choose 'NodeJS'
- Click 'Next'
- Open 'Overview'
  - Check 'Public Function'
- Open 'Editor' tab.
  - Create 'index.js' and 'package.json'
  - Entry point -> index.handler
  - Choose your new 'Service Account'
  - Click 'Create version'

### First Cloud Function - wrapper
- Open'Editor' tab. And enter:
	- Memory: 128
	- Timeout: 3
    - Add 'Environment Variables' -> `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.
  - Open package.json and add code (below).
    ```
      {
        "name": "nanov94-wrapper",
        "version": "1.0.0",
        "dependencies": {
          "node-fetch": "2.6.1",
          "aws-sdk": "2.810.0"
        }
      }
    ```
  - Open index.js and add code (below). Copy 'URL' on tab 'Overview' from 'Message Queue'.
    ```
      const fetch = require('node-fetch');
      var AWS = require('aws-sdk');

      const queueUrl = <Message Queue URL>;

      module.exports.handler = async function (event, context) {
        const requestID = context.requestId;

        var ymq = new AWS.SQS({
          'region': 'ru-central1',
          'endpoint': 'https://message-queue.api.cloud.yandex.net',
        });

        params = {
          'QueueUrl': queueUrl,
          'MessageBody': JSON.stringify(event.body),
        }

        await ymq.sendMessage(params, function(err, data) {
          if (err) {
            console.log(`[${requestID}] Error has occurred.`, err);
          } else {
            console.log(`[${requestID}] sendMessage was finished successful.`, data.MessageId);
          }
        });

        return {
          statusCode: 202,
        };
      }
    ```

### Second Cloud Function with code from checkYfm.js
- Open'Editor' tab. And enter:
	- Memory: 2048
	- Timeout: 600
  - Open package.json and add code (below).
    ```
      {
        "name": "test-github-webhook",
        "version": "1.0.0",
        "dependencies": {
          "isomorphic-git": "^1.8.0",
          "@octokit/core": "^3.2.1",
          "@octokit/auth-app": "^2.10.1",
          "@doc-tools/docs": "^1.3.1"
        }
      }
    ```
  - Open index.js and add code from file ./scr/checkYfm.js
  - Add 'Environment Variables' -> `APP_ID`, `INSTALLATION_ID`, `PRIVATE_KEY`,
  `TOKEN` (Open GitHub -> 'Settings' -> 'Developer Settings' -> 'Personal access tokens'), `YCF_TMPFS` = 1204


## Create 'Trigger'
- Open 'Cloud Function' -> 'Triggers'
- Click 'Create trigger'
  - Enter 'Name'
  - Choose 'Type': 'Message Queue'
  - Choose your 'Message Queue'
  - Select 'Service account' which you use for message queue
  - Choose your 'Cloud Function'
  - Select 'Service account' which you use for cloud function


## Add Webhook to Git repo

- Open your repository
- 'Settings' -> 'Webhooks' -> 'Add Webhook'
  - Add Payload URL -> Copy 'Link for call' on tab 'Overview' from first Cloud Function (wrapper).
	- Content type: application/json
	- 'Which events would you like to trigger this webhook?' : Pull request
	- Click 'Add Webhook'


## Лицензии

© YANDEX LLC, 2018. Licensed under Creative Commons Attribution 4.0 International Public License. See [LICENSE](LICENSE) file for more details.

[Official documentation]: https://cloud.yandex.ru/docs/functions/quickstart/?utm_source=console&utm_medium=side-bar-left&utm_campaign=functions
