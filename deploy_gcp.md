## How to deploy to Cloud Run on a new GCP project

Commands to run in cloud shell:

Clone repo and change to directory
git clone https://github.com/bilydr/gemini-app
cd gemini-app

Activate APIs
```
gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com artifactregistry.googleapis.com
```

Set env variables
```
PROJECT_ID=$(gcloud config get-value project)
REGION=us-south1
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```

more env variables
``` 
SERVICE_NAME='gemini-app-playground' # Name of your Cloud Run service.
AR_REPO='gemini-app-repo'            # Name of your repository in Artifact Registry
echo "SERVICE_NAME=${SERVICE_NAME}"
echo "AR_REPO=${AR_REPO}"
```

create an artifact repo and set up authentication
```
gcloud artifacts repositories create "$AR_REPO" --location="$REGION" --repository-format=Docker
gcloud auth configure-docker "$REGION-docker.pkg.dev"
```

build docker image
```
gcloud builds submit --tag "$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME"
```

deploy to cloud run
```
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$REGION \
  --platform=managed  \
  --project=$PROJECT_ID \
  --set-env-vars=PROJECT_ID=$PROJECT_ID,REGION=$REGION
  ```

