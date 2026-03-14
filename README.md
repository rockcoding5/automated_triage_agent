# 🛡️ Project: Automated Triage Agent
**GenAI Academy - Professional Project Submission**

## 📋 Project Scope
This project focuses on the development and deployment of an intelligent **Senior DevOps Triage Agent**. The core objective was to create a specialized AI tool capable of transforming raw, chaotic error logs into structured, actionable insights while maintaining strict operational guardrails.

### 🎯 What We Achieved:
* **Intelligent Log Summarization:** Leveraged Gemini 1.5 Flash to parse complex system errors into a standardized 3-point report (Category, Cause, Resolution).
* **Custom Persona & Guardrails:** Configured the agent to ignore non-technical queries (e.g., general knowledge or lifestyle questions), ensuring it remains a focused enterprise tool.
* **Cloud-Native Serverless Architecture:** Deployed via Google Cloud Run for automatic scaling and high availability.
* **Enterprise Identity Management:** Implemented a dedicated IAM Service Account with scoped permissions to Vertex AI, adhering to the **Principle of Least Privilege**.



---

## 🛠️ Technical Implementation

### 1. The Agent Logic (`agent.py`)
The agent is built using the **Google Agent Development Kit (ADK)**. It is programmed with a system instruction set that handles four critical states:
1.  **Greeting:** Identifies itself and its purpose immediately.
2.  **Triage Processing:** Extracts technical context from raw input.
3.  **Formatting:** Returns responses in clean, bolded Markdown.
4.  **Scope Guarding:** Rejects requests outside the DevOps triage domain.

### 2. Infrastructure Stack
* **Compute:** Google Cloud Run
* **AI Engine:** Vertex AI (Gemini 1.5 Flash)
* **Environment:** Python 3.11 / UV Virtual Environment
* **Identity:** Custom IAM Service Account (`triage-service-auth`)

---

## 🚀 Deployment Workflow
The agent was deployed using a single-shot `uvx` command to ensure the build environment was clean and consistent with production requirements:

```bash
uvx --from google-adk==1.14.0 \
adk deploy cloud_run \
  --project=$PROJECT_ID \
  --region=us-central1 \
  --service_name=automated-triage-agent \
  --app_name=triage_agent \
  --with_ui \
  . \
  -- \
  --service-account=$SERVICE_ACCOUNT \
  --set-env-vars="GOOGLE_GENAI_USE_VERTEXAI=1,MODEL=gemini-1.5-flash"