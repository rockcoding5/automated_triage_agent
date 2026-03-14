# Automated Triage Agent – Implementation Steps

## Step 1: Environment & API Setup

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

# Step 2: Create Project Files

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

# Step 3: Identity & IAM Permissions

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

# Step 4: Deployment

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

# Step 5: Testing Inputs

| # | User Input | Expected Response (Approximate) | Purpose |
| :--- | :--- | :--- | :--- |
| 1 | `"Hello"` | "Hello! I am the Automated Triage Agent. I help summarize logs and error text." | Greeting Validation |
| 2 | `"Tell me a joke."` | "I am specifically designed for triage summarization and cannot assist with other queries like telling jokes." | Guardrail Validation |
| 3 | `"ModuleNotFoundError: No module named 'pandas'"` | "**Category:** Dependency <br><br> **Cause:** The Python interpreter could not find the 'pandas' module because it is not installed in the current environment or is not accessible in the Python path. <br><br> **Resolution:** Install the missing 'pandas' package using a package manager, typically pip install pandas, within the active Python environment." | Logic/Lib Triage |
| 4 | `"401 Unauthorized: Invalid API Key"` | "**Category:** Authentication/Authorization <br><br> **Cause:** The API key provided with the request is either missing, invalid, expired, or does not have the necessary permissions to access the requested resource. <br><br> **Resolution:** Verify that the correct and valid API key is being used. Ensure it is included in the request as specified by the API documentation (e.g., in a header, query parameter, or request body). If necessary, generate a new API key from the service provider's dashboard." | Security Triage |
| 5 | `"504 Gateway Timeout"` | "**Category:** Networking/Server-side <br><br> **Cause:** The server acting as a gateway or proxy did not receive a timely response from an upstream server, indicating a delay or failure in communication between them. <br><br> **Resolution:** Investigate the health and performance of the upstream server, check its logs for errors or high load, verify network connectivity between the proxy and the upstream server, and consider adjusting timeout configurations on the proxy if the upstream server is expected to take longer to respond." | Web Error Triage |
| 6 | `"Error: ENOSPC: System limit reached"` | "**Category:** OS/Disk <br><br> **Cause:** Max inotify watches reached. <br><br> **Resolution:** Increase `fs.inotify.max_user_watches`." | OS Limit Triage |
| 7 | `"Fatal Error: Out of memory (allocated 20971520)"` | "**Category:** Resource Allocation <br><br> **Cause:** RAM limit exceeded. <br><br> **Resolution:** Increase server memory or optimize code." | Memory Triage |
| 8 | `"EADDRINUSE: address already in use :::8080"` | "**Category:** Network <br><br> **Cause:** Port 8080 is being used by another process. <br><br> **Resolution:** Kill the process on `8080` or change the app port." | Port Conflict Triage |
| 9 | `"EACCES: permission denied, open '/root/config'"` | "**Category:** Permissions <br><br> **Cause:** Current user lacks read/write access. <br><br> **Resolution:** Use `sudo` or adjust file permissions with `chmod`." | Permissions Triage |
| 10 | `"How is the weather today?"` | "I am specifically designed for triage summarization and cannot assist with other queries." | Guardrail Validation |

![Accessing the Agent](Accessing_the_Agent_ADK_Web.png)
