# 🛡️ Project: Automated Triage Agent
**GenAI Academy - Professional Project Submission**

## 🎯 Problem Statement
Build and deploy a *single AI agent* using *ADK and Gemini* that is hosted on *Cloud Run* and performs *one clearly defined task*. The agent must be callable via an HTTP endpoint and return a valid response for a given input.

This is a mini project focused on agent structure and deployment, showcasing a production-ready serverless AI workflow.

## 📋 Project Scope
This project focuses on the development and deployment of an intelligent **Senior DevOps Triage Agent**. The core objective was to create a specialized AI tool capable of transforming raw, chaotic error logs into structured, actionable insights while maintaining strict operational guardrails.

### 🎯 What We Achieved:
* **Single Capability Focus:** Specialized in **Classification and Summarization** of technical error logs.
* **Intelligent Log Analysis:** Leveraged **Gemini 1.5 Flash** to parse system errors into a standardized 3-point report (Category, Cause, Resolution).
* **Custom Persona & Guardrails:** Configured the agent to ignore non-technical queries, ensuring it remains a focused enterprise tool.
* **Cloud-Native Serverless Architecture:** Deployed via **Google Cloud Run** for automatic scaling and high availability.
* **Enterprise Identity Management:** Implemented a dedicated **IAM Service Account** with scoped permissions to Vertex AI, adhering to the Principle of Least Privilege.

---

| # | User Input | Expected Response (Approximate) | Purpose |
| :--- | :--- | :--- | :--- |
| 1 | `"Hello"` | "Hello! I am the Automated Triage Agent. I help summarize logs and error text." | Greeting Validation |
| 2 | `"Error: ENOSPC: System limit reached"` | "**Category:** OS/Disk <br><br> **Cause:** Max inotify watches reached. <br><br> **Resolution:** Increase `fs.inotify.max_user_watches`." | OS Limit Triage |
| 3 | `"Fatal Error: Out of memory (allocated 20971520)"` | "**Category:** Resource Allocation <br><br> **Cause:** RAM limit exceeded. <br><br> **Resolution:** Increase server memory or optimize code." | Memory Triage |
| 4 | `"EADDRINUSE: address already in use :::8080"` | "**Category:** Network <br><br> **Cause:** Port 8080 is being used by another process. <br><br> **Resolution:** Kill the process on `8080` or change the app port." | Port Conflict Triage |
| 5 | `"EACCES: permission denied, open '/root/config'"` | "**Category:** Permissions <br><br> **Cause:** Current user lacks read/write access. <br><br> **Resolution:** Use `sudo` or adjust file permissions with `chmod`." | Permissions Triage |
| 6 | `"ModuleNotFoundError: No module named 'pandas'"` | "**Category:** Dependency <br><br> **Cause:** Required library is missing from the environment. <br><br> **Resolution:** Run `pip install pandas`." | Logic/Lib Triage |
| 7 | `"504 Gateway Timeout"` | "**Category:** Network/Web <br><br> **Cause:** Upstream server failed to respond in time. <br><br> **Resolution:** Check server health or increase timeout settings." | Web Error Triage |
| 8 | `"401 Unauthorized: Invalid API Key"` | "**Category:** Security <br><br> **Cause:** Authentication credentials failed or are missing. <br><br> **Resolution:** Verify API keys and environment variables." | Security Triage |
| 9 | `"How is the weather today?"` | "I am specifically designed for triage summarization and cannot assist with other queries." | Guardrail Validation |
| 10 | `"Tell me a joke."` | "I am focused strictly on DevOps triage tasks and do not provide general entertainment." | Guardrail Validation |


---

## 🛠️ Technical Implementation

### 1. The Agent Logic (`agent.py`)
The agent is built using the **Google Agent Development Kit (ADK)**. It is programmed with a system instruction set that handles four critical states:
1.  **Greeting:** Identifies itself and its purpose immediately upon connection.
2.  **Triage Processing:** Extracts technical context and classifies raw input logs.
3.  **Formatting:** Returns responses in clean, structured Markdown (Bold headers and bullet points).
4.  **Scope Guarding:** Rejects requests outside the DevOps triage domain to maintain focus.

### 2. Infrastructure Stack
* **Compute:** Google Cloud Run (Exposed via HTTP Endpoint)
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