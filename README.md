# ReachRight 🎯

> AI-powered cold email generator. Upload your resume, describe who you want to reach, and get a personalised cold email with resume improvements in seconds.

**Application at:** [http://34.28.173.84](http://34.28.173.84)

**Demo video:** https://drive.google.com/file/d/1SV4dZxvN2laW9S9yIf2YgkLNcx4ZKytg/view?usp=drive_link

---

## What it does

ReachRight takes your resume and a description of your target (e.g. "ML engineering manager at an AI startup") and:

1. Searches LinkedIn via Tavily to find a real matching prospect
2. Extracts their name, role, and company
3. Routes to the right email style (Founder, Executive, or Recruiter) based on their profile
4. Generates a personalised cold email using Vertex AI, with a confidence score
5. Analyses your resume and gives tailored improvement suggestions for the target role

---



## Tech stack

| Layer | Technology |
|---|---|
| Orchestration | Langflow (No-Code Agent Builder) |
| LLM | Google Vertex AI (Gemini 2.5 Flash) |
| Prospect Search | Tavily Search API |
| Frontend | HTML/CSS/JS served via nginx |
| Infrastructure | Google Kubernetes Engine (GKE) |

---

## Architecture

```
Browser (http://34.28.173.84)
        ↓
nginx pod — serves the frontend HTML
        ↓
Langflow pod (http://35.224.85.248) — runs the AI flow
        ├── Tavily Search → finds LinkedIn prospect
        ├── Vertex AI → builds search query
        ├── Vertex AI → extracts prospect name/role
        ├── If-Else routing → Founder / Executive / Recruiter
        ├── Vertex AI → generates personalised email + confidence score
        └── Vertex AI → generates resume improvement suggestions
```

Both pods run on GKE and are exposed via LoadBalancer services.

---

## Setup & deployment

### Prerequisites

- Google Cloud account with billing enabled
- `gcloud` CLI installed and authenticated
- `kubectl` installed

### 1. Create GKE cluster

```bash
gcloud container clusters create reachright-cluster \
  --zone us-central1-a \
  --num-nodes 2 \
  --machine-type e2-standard-2

gcloud container clusters get-credentials reachright-cluster --zone us-central1-a
```

### 2. Deploy Langflow backend

```bash
kubectl apply -f langflow-deployment.yaml
kubectl get service langflow-service  # wait for EXTERNAL-IP
```

### 3. Import the flow

1. Open Langflow at the external IP in your browser
2. Import `ReachRight_final__without_API_keys_.json`
3. Add your API keys to the relevant nodes:
   - **Vertex AI nodes** — Google Cloud credentials / project ID
   - **Tavily Search API** — Tavily API key (free tier at [tavily.com](https://tavily.com))
4. Create a Langflow API key (Settings → API Keys)
5. Note your flow ID from the API access panel

### 4. Update and deploy the frontend

In `frontend.yaml`, update the following values in the JavaScript section:

```javascript
const FLOW_URL = 'http://<YOUR_LANGFLOW_IP>/api/v1/run/<YOUR_FLOW_ID>';
const API_KEY = '<YOUR_LANGFLOW_API_KEY>';
```

Also update the file upload URL:
```javascript
const uploadRes = await fetch('http://<YOUR_LANGFLOW_IP>/api/v2/files', {
```

Then deploy:

```bash
kubectl apply -f frontend.yaml
kubectl get service reachright-frontend-service  # wait for EXTERNAL-IP
```

### 5. Open the app

Navigate to the frontend's external IP in your browser.

---

## Usage

1. **Upload resume** — PDF, DOCX, or TXT
2. **Describe your target** — e.g. `"Hiring manager at a cybersecurity startup in NYC"`
3. **Click Generate Emails** — wait ~30-60 seconds
4. **Review the email** — confidence score shown at the bottom
5. **Accept or decline** — accept opens your mail app with subject + body pre-filled
6. **Check resume suggestions** — scroll down for AI-powered improvement tips

---

## Scaling

Since both components are Kubernetes deployments, scaling is a single command:

```bash
kubectl scale deployment langflow --replicas=3
```

GKE automatically load balances traffic across replicas.

---

## Notes

- API keys are **not included** in the Langflow JSON — you must add them after importing
- Files uploaded through the frontend are stored temporarily in the Langflow pod — they will be lost if the pod restarts
- The app is served over HTTP (not HTTPS) — clipboard API is unavailable in this mode; the "Open in Email App" button uses a `mailto:` fallback instead

---

## Team

| Name | Andrew ID |
|---|---|
| Nandini Chanda | nchanda |
| Serena Gomez | serenag |
| Ananya Gahalaut | agahala |
| Shashank Aluru | yaluru |

