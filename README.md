# Jan-Setu (जन-सेतू) | The People's Bridge 🇮🇳

![Project Status](https://img.shields.io/badge/Status-Hackathon_Submission-orange)
![Team](https://img.shields.io/badge/Team-Binary_Blitz-blue)
![Stack](https://img.shields.io/badge/Tech-AWS_Serverless_&_Bedrock-232F3E)

> **Voice-First Agentic AI Governance for the Last Mile.**

## 📄 Project Overview
**Jan-Setu** is an AI-powered super-app designed to bridge the digital divide for rural Indian citizens. By leveraging **Amazon Bedrock (Claude 3)** and a **Serverless Architecture**, it enables illiterate users to:
1.  **Discover Government Schemes** via voice.
2.  **File Legal Grievances** (PDF generation) automatically.
3.  **Access Markets** to sell produce.

---

## 🏗️ Architecture
The system follows a proprietary **Serverless Event-Driven Architecture**:
* **Frontend:** Flutter (Mobile) + Bhashini API (Voice).
* **Orchestrator:** AWS Lambda + Amazon Bedrock (Router).
* **Core Logic:** AWS Step Functions (Grievance Workflow).
* **Data:** Amazon DynamoDB (Single Table Design).

*(See `design.md` for full architectural details.)*

---

## 📂 Repository Structure
```bash
Jan-Setu/
├── app/                  # Flutter Mobile Application
├── backend/              # AWS SAM Template & Lambda Functions
├── docs/                 # Documentation & Presentation
│   ├── Binary_Blitz_Presentation.pdf  <-- VIEW PROJECT PPT HERE
│   ├── requirements.md
│   └── design.md
└── README.md
