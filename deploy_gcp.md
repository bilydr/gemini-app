# Run on a new GCP project

## Prep

Clone repo and change to directory
```bash
git clone https://github.com/bilydr/gemini-app
cd gemini-app
```

Activate APIs
```bash
gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com artifactregistry.googleapis.com
```

Set some env variables
```bash
PROJECT_ID=$(gcloud config get-value project)
REGION=us-south1 # Region of your choice
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```

## To run locally

```bash
# set up virtual env
python3 -m venv gemini-streamlit
source gemini-streamlit/bin/activate

# install dependencies
pip install -r requirements.txt

# start the app
streamlit run app.py \
--browser.serverAddress=localhost \
--server.enableCORS=false \
--server.enableXsrfProtection=false \
--server.port 8080
```
## To deploy to cloud run
Set more env variables
```bash
SERVICE_NAME='gemini-app-playground' # Name of your Cloud Run service.
AR_REPO='gemini-app-repo'            # Name of your repository in Artifact Registry
echo "SERVICE_NAME=${SERVICE_NAME}"
echo "AR_REPO=${AR_REPO}"
```

Create an artifact repo and set up authentication
```bash
gcloud artifacts repositories create "$AR_REPO" --location="$REGION" --repository-format=Docker
gcloud auth configure-docker "$REGION-docker.pkg.dev"
```

Build docker image
```bash
gcloud builds submit --tag "$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME"
```

Deploy to cloud run
```bash
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$REGION \
  --platform=managed  \
  --project=$PROJECT_ID \
  --set-env-vars=PROJECT_ID=$PROJECT_ID,REGION=$REGION
  ```

