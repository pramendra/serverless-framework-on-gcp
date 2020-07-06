# Continuous integration and deployment via github actions


## Create github actions workflow 
```cat .github/workflows/branch.yml```

## Create Secretes on github 
secretes will be used during deploying cloud functions 
1. login to https://github.com/pramendra/serverless-framework-on-gcp/settings/secrets

2. add secretes

SLS_GCP_PROJECT: [PROJECT-NAME]
SLS_GCP_REGION: asia-northeast1
SLS_GCP_SA_EMAIL: ${SERVICE_ACCOUNT}@${PROJECT}.iam.gserviceaccount.com
SLS_GCP_SA_KEY: cat ~/.gcloud/sls_keyfile.json | base64
SERVICE_NAME: sls-gcp
SERVICE_NAME_FUNCTION: first