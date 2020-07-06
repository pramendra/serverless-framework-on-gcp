# serverless-framework-on-gcp

## GCP for serverless framework

### Setup google project

#### login to gcp
```sh
gcloud auth login
```

#### verify the login 
```sh
gcloud auth list 
```
> notice the *star in front of the email

#### list the config
```sh
gcloud config list
```
> notice `project` and copy [PROJECT-NAME]

#### setup the proejct config
```sh
gcloud config set project [PROJECT-NAME]
```

### Enable API on GCP 
```sh
gcloud services enable deploymentmanager.googleapis.com cloudfunctions.googleapis.com storage-component.googleapis.com logging.googleapis.com
```
> Operation "operations/acf.46287898-0590-xyz-xyz-xyzxyzxyz" finished successfully.


### Create service account

#### Export constants
##### PROJECT=YOUR_PROJECT_NAME
PROJECT=[PROJECT-NAME]

##### Service account name
SERVICE_ACCOUNT=cicd-deploy

##### File to download authentication information 
KEY_FILE=~/.gcloud/sls_keyfile.json

#### Create Service account
```sh
gcloud iam service-accounts create ${SERVICE_ACCOUNT} --display-name "Cloud Functions deployment account for Serverless Framework"
```

#### Assign roles to Service account
##### roles/deploymentmanager.editor
```sh
gcloud projects add-iam-policy-binding ${PROJECT} \
--member=serviceAccount:${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com \
--role=roles/deploymentmanager.editor
```

##### roles/storage.admin
```sh
gcloud projects add-iam-policy-binding ${PROJECT} \
--member=serviceAccount:${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com \
--role=roles/storage.admin
```
##### roles/logging.admin
```sh
gcloud projects add-iam-policy-binding ${PROJECT} \
--member=serviceAccount:${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com \
--role=roles/logging.admin
```

##### roles/cloudfunctions.developer
```sh
gcloud projects add-iam-policy-binding ${PROJECT} \
--member=serviceAccount:${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com \
--role=roles/cloudfunctions.developer
```


### Generate and download Service account credentials
```sh
$ gcloud iam service-accounts keys create ${KEY_FILE} \
--iam-account ${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com
```

> created key [XYZ30XYZ1cXYZ08462XY4b775842] of type [json] as [~/.gcloud/sls_-_keyfile.json] for [cicd-deploy@linc-info.iam.gserviceaccount.com]


## setup serverless framework

### create serverless function
```sh
mkdir sls-gcp && $_
npx serverless create --template google-nodejs
```

### integrate env
```sh
npm i -D serverless-dotenv-plugin
```
#### create .env file with followng content
```
SLS_CREDENTIALS_PATH=~/.gcloud/sls_keyfile.json 
SLS_GCP_PROJECT=[PROJECT-NAME]
SLS_GCP_REGION=asia-northeast1
```

#### update serverless.yml 
```
provider:
  name: google
  stage: dev
  runtime: nodejs10
  region: ${env:SLS_GCP_REGION}
  project: ${env:SLS_GCP_PROJECT}
  credentials: ${env:SLS_CREDENTIALS_PATH}

plugins:
  - serverless-google-cloudfunctions
  - serverless-dotenv-plugin
```

## Deploy to gcp
### deploy
```
sls deploy
```

### remove deploy
```
sls remove
```


## Error 

### Error: Deployment failed: RESOURCE_ERROR
Reason: environment variable name GCP_PROJECT is reserved by the system: it cannot be set by users
Solution: change `GCP_REGION` -> `SLS_GCP_REGION`

Reason: this may happen if earlier deployed function is different region so you have to remove it using serverless cli
Solution:
```
sls remove
```

### Error: Deployment failed: RESOURCE_ERROR

```
Error: Forbidden
Your client does not have permission to get URL /sls-gcp-dev-first from this server.
```
when accessing https://asia-northeast1-[PROJECT-NAME].cloudfunctions.net/sls-gcp-dev-first
Reason: Deployed function is not public available yet ie add `allUsers` with following role `roles/cloudfunctions.invoker` to make function available public

Solution:
```
gcloud functions add-iam-policy-binding sls-gcp-dev-first \
--member="allUsers" \
--role="roles/cloudfunctions.invoker" \
--project [PROJECT-NAME] \
--region asia-northeast1
```


## Development

### change the node version 
```sh
nvm use
```
### Install dependencies 
```sh
npm i
```

