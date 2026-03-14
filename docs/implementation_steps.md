# Automated Triage Agent – Implementation Steps

## Phase 1: Environment & API Setup

Run the following commands to prepare the Google Cloud environment.

### Set Project ID and Region

```bash
PROJECT_ID=$(gcloud config get-value project)
REGION=us-central1
```

### Enable Required APIs

```bash
gcloud services enable \
run.googleapis.com \
artifactregistry.googleapis.com \
cloudbuild.googleapis.com \
aiplatform.googleapis.com \
compute.googleapis.com
```

### Create Project Directory

```bash
cd ~ && mkdir automated_triage_agent && cd automated_triage_agent
```

### Setup Python Virtual Environment

```bash
uv venv
source .venv/bin/activate
```

---

# Phase 2: Create Project Files

## requirements.txt

```bash
cat <<EOF > requirements.txt
google-adk==1.14.0
python-dotenv==1.0.1
google-cloud-logging==3.11.0
EOF

uv pip install -r requirements.txt
```

## .env Configuration

```bash
SA_NAME=triage-service-auth

cat <<EOF > .env
PROJECT_ID=$PROJECT_ID
SERVICE_ACCOUNT=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
MODEL="gemini-1.5-flash"
GOOGLE_GENAI_USE_VERTEXAI=1
EOF
```

## **init**.py

```bash
echo "from . import agent" > __init__.py
```

---

# Phase 3: Identity & IAM Permissions

Create the service account.

```bash
gcloud iam service-accounts create triage-service-auth \
--display-name="Triage Project Service Account"
```

Grant Vertex AI access.

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member="serviceAccount:triage-service-auth@$PROJECT_ID.iam.gserviceaccount.com" \
--role="roles/aiplatform.user"
```

---

# Phase 4: Deployment

Deploy the agent to Cloud Run.

```bash
source .env

uvx --from google-adk==1.14.0 \
adk deploy cloud_run \
--project=$PROJECT_ID \
--region=$REGION \
--service_name=automated-triage-agent \
--app_name=triage_agent \
--with_ui \
. \
-- \
--service-account=$SERVICE_ACCOUNT \
--set-env-vars="GOOGLE_GENAI_USE_VERTEXAI=1,MODEL=gemini-1.5-flash"
```

---

# Phase 5: Testing Inputs

### Greeting

```
Hello
```

### File Watcher Error

```
Error: ENOSPC: System limit for number of file watchers reached
```

### Port Conflict

```
Error: listen EADDRINUSE: address already in use :::8080
```

### Permission Error

```
EACCES: permission denied
```
