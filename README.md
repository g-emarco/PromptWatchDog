# 🐶 PromptWatchDog

![PromptWatchDog Banner](static/banner.png)

<div align="center">

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/)
[![GCP](https://img.shields.io/badge/Google_Cloud-4285F4?logo=google-cloud&logoColor=white)](https://cloud.google.com/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

**Real-time Insights metric powered by Gemini**

</div>

---

## 📖 Overview

**PromptWatchDog** is a serverless, event-driven framework designed to monitor data streams using natural language prompts. Instead of rigid code-based metrics, you define what you want to track in English, and Gemini interprets the data in real-time.

Key capabilities include:
- **Event Ingestion**: Listens to messages on Google Cloud Pub/Sub.
- **AI Analysis**: Uses **Gemini 1.5 Pro** to process and classify incoming data payloads according to your prompt.
- **Structured Insights**: Converts unstructured text/data into structured metrics logs.
- **Serverless Scale**: Deploys on Google Cloud Functions (Gen2) for auto-scaling capabilities.

## 🏗️ Project Structure

```bash
├── watchdog/
│   ├── main.py              # Main Cloud Function entry point & subscription logic
│   └── requirements.txt     # Python dependencies (LangChain, Vertex AI, etc.)
├── dashboard/           # Next.js UI for managing prompts (Local Only)
├── static/
│   └── banner.png           # Project assets
├── cloudbuild.yaml          # CI/CD configuration for Cloud Build
├── README.md                # Documentation
└── .env                     # Local environment configuration
```

## 🧩 Key Components

1.  **Message Bus (Pub/Sub)**: Acts as the buffer and trigger for the system. Messages published to the `messages` topic trigger the analysis.
2.  **Orchestrator (LangChain)**: Manages the interaction between the raw data and the LLM, ensuring prompts are formatted correctly.
3.  **Intelligence Engine (Gemini)**: The core "brain" that evaluates the data against the natural language metric definitions.
4.  **Observer (Cloud Monitoring)**: Captures the structured output from Gemini as log-based metrics for dashboarding.
5.  **Storage (Firestore)**: Stores the natural language prompts used by PromptWatchDog.

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.11+** installed locally.
- **Google Cloud Project** with billing enabled.
- **gcloud CLI** authenticated and configured.
- APIs Enabled: `vertexai`, `cloudfunctions`, `pubsub`, `run`, `cloudbuild`.

### Environment Setup

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/g-emarco/PromptWatchDog.git
    cd PromptWatchDog
    ```

2.  **Create a `.env` file**:
    ```bash
    touch .env
    ```
    Add the following variables:
    ```env
    GCP_PROJECT=your-project-id
    GOOGLE_APPLICATION_CREDENTIALS=path/to/key.json
    LOCAL=true
    ```

3.  **Install Dependencies**:
    ```bash
    pip install -r watchdog/requirements.txt
    ```

### Running Locally

To start the local watchdog process that emulates the behavior of the Cloud Function:

```bash
python watchdog/main.py
```

> **Note**: This will execute the `subscribe` function loop, pulling messages from the configured subscription if running in local simulation mode, or performing a single run depending on logic.

---

## ☁️ Deployment

You can deploy the solution directly to Google Cloud using the `gcloud` CLI.

### 1. Infrastructure Setup

Initialize the required Google Cloud resources.

**Enable APIs:**
```bash
gcloud services enable \
  cloudbuild.googleapis.com \
  run.googleapis.com \
  secretmanager.googleapis.com \
  artifactregistry.googleapis.com \
  vertexai.googleapis.com \
  firestore.googleapis.com
```

**Setup IAM:**
```bash
export PROJECT_ID=$(gcloud config get-value project)
export SA_NAME="vertex-ai-consumer"
export SA_EMAIL="$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com"

# Create Service Account
gcloud iam service-accounts create $SA_NAME --display-name="Vertex AI Consumer"

# Grant roles
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/run.invoker"
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/aiplatform.user"
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/pubsub.subscriber"
```

**Create Pub/Sub Topic:**
```bash
gcloud pubsub topics create messages
```

**Create Firestore Database:**
```bash
gcloud firestore databases create --database=watchdog-prompts --location=us-east1
```

### 2. Deploy Function

Deploy the code as a Gen2 Cloud Function.

```bash
gcloud functions deploy prompt-watchdog \
  --region=us-east1 \
  --source=./watchdog \
  --trigger-topic=messages \
  --runtime=python311 \
  --gen2 \
  --entry-point=subscribe \
  --service-account=$SA_EMAIL \
  --memory=8Gi \
  --cpu=8
```

---

## 🖥️ Dashboard

> **⚠️ CRITICAL WARNING**: The Next.js dashboard server (`dashboard/`) is designed to be run **LOCALLY ONLY**.

**DO NOT DEPLOY THE DASHBOARD TO THE CLOUD.**

The dashboard requires elevated permissions to:
*   Run commands to create custom log-based metrics.
*   Perform administrative operations in GCP.

Running this locally follows the **Principle of Least Privilege**, ensuring that these powerful permissions are restricted to your local authenticated session and not exposed via a specialized service account in a deployed environment.

### Running the Dashboard Locally

```bash
cd dashboard
npm install
npm run dev
```

---


## ⚠️ Disclaimer

This is not an officially supported Google product. This project is not eligible for the Google Open Source Software Vulnerability Rewards Program.


## 👤 Author

**Eden Marco** - *LLM Specialist @ Google Cloud*

[![LinkedIn](https://img.shields.io/badge/linkedin-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/eden-marco/)
[![X](https://img.shields.io/badge/X-%23000000.svg?style=for-the-badge&logo=X&logoColor=white)](https://twitter.com/EdenEmarco177)

---